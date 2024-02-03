# `v2ray-core\app\dns\udpns.go`

```go
// +build !confonly
// 设置构建标签，表示此代码不仅仅是配置文件

package dns
// 导入所需的包
import (
    "context"
    "strings"
    "sync"
    "sync/atomic"
    "time"

    "golang.org/x/net/dns/dnsmessage"
    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol/dns"
    udp_proto "v2ray.com/core/common/protocol/udp"
    "v2ray.com/core/common/session"
    "v2ray.com/core/common/signal/pubsub"
    "v2ray.com/core/common/task"
    dns_feature "v2ray.com/core/features/dns"
    "v2ray.com/core/features/routing"
    "v2ray.com/core/transport/internet/udp"
)

// ClassicNameServer 结构体
type ClassicNameServer struct {
    sync.RWMutex
    name      string
    address   net.Destination
    ips       map[string]record
    requests  map[uint16]dnsRequest
    pub       *pubsub.Service
    udpServer *udp.Dispatcher
    cleanup   *task.Periodic
    reqID     uint32
    clientIP  net.IP
}

// NewClassicNameServer 创建 ClassicNameServer 实例
func NewClassicNameServer(address net.Destination, dispatcher routing.Dispatcher, clientIP net.IP) *ClassicNameServer {

    // 如果地址端口未指定，则默认为 53
    if address.Port == 0 {
        address.Port = net.Port(53)
    }

    // 初始化 ClassicNameServer 实例
    s := &ClassicNameServer{
        address:  address,
        ips:      make(map[string]record),
        requests: make(map[uint16]dnsRequest),
        clientIP: clientIP,
        pub:      pubsub.NewService(),
        name:     strings.ToUpper(address.String()),
    }
    // 设置定时清理任务
    s.cleanup = &task.Periodic{
        Interval: time.Minute,
        Execute:  s.Cleanup,
    }
    // 创建 UDP 调度器
    s.udpServer = udp.NewDispatcher(dispatcher, s.HandleResponse)
    newError("DNS: created udp client inited for ", address.NetAddr()).AtInfo().WriteToLog()
    return s
}

// ClassicNameServer 的 Name 方法
func (s *ClassicNameServer) Name() string {
    return s.name
}

// ClassicNameServer 的 Cleanup 方法
func (s *ClassicNameServer) Cleanup() error {
    now := time.Now()
    s.Lock()
    defer s.Unlock()

    // 如果 ips 和 requests 都为空，则停止清理任务
    if len(s.ips) == 0 && len(s.requests) == 0 {
        return newError(s.name, " nothing to do. stopping...")
    }
    # 遍历 s.ips 中的域名和记录
    for domain, record := range s.ips {
        # 如果 A 记录存在且过期时间在当前时间之前，则将 A 记录置空
        if record.A != nil && record.A.Expire.Before(now) {
            record.A = nil
        }
        # 如果 AAAA 记录存在且过期时间在当前时间之前，则将 AAAA 记录置空
        if record.AAAA != nil && record.AAAA.Expire.Before(now) {
            record.AAAA = nil
        }

        # 如果 A 记录和 AAAA 记录都为空，则从 s.ips 中删除该域名
        if record.A == nil && record.AAAA == nil {
            delete(s.ips, domain)
        } else {
            # 否则更新 s.ips 中的记录
            s.ips[domain] = record
        }
    }

    # 如果 s.ips 中没有记录，则重新创建一个空的记录映射
    if len(s.ips) == 0 {
        s.ips = make(map[string]record)
    }

    # 遍历 s.requests 中的请求 ID 和请求
    for id, req := range s.requests {
        # 如果请求的过期时间在当前时间之前，则从 s.requests 中删除该请求
        if req.expire.Before(now) {
            delete(s.requests, id)
        }
    }

    # 如果 s.requests 中没有请求，则重新创建一个空的请求映射
    if len(s.requests) == 0 {
        s.requests = make(map[uint16]dnsRequest)
    }

    # 返回空值
    return nil
func (s *ClassicNameServer) HandleResponse(ctx context.Context, packet *udp_proto.Packet) {
    // 解析响应数据包，获取解析结果和可能的错误
    ipRec, err := parseResponse(packet.Payload.Bytes())
    // 如果解析出错，记录错误信息并返回
    if err != nil {
        newError(s.name, " fail to parse responded DNS udp").AtError().WriteToLog()
        return
    }

    // 加锁，确保并发安全
    s.Lock()
    id := ipRec.ReqID
    req, ok := s.requests[id]
    // 如果找到对应的请求，删除该请求
    if ok {
        delete(s.requests, id)
    }
    s.Unlock()
    // 如果未找到对应的请求，记录错误信息并返回
    if !ok {
        newError(s.name, " cannot find the pending request").AtError().WriteToLog()
        return
    }

    var rec record
    // 根据请求类型，将解析结果存入对应的记录字段
    switch req.reqType {
    case dnsmessage.TypeA:
        rec.A = ipRec
    case dnsmessage.TypeAAAA:
        rec.AAAA = ipRec
    }

    // 计算请求处理时间，并记录处理结果
    elapsed := time.Since(req.start)
    newError(s.name, " got answer: ", req.domain, " ", req.reqType, " -> ", ipRec.IP, " ", elapsed).AtInfo().WriteToLog()
    // 如果域名不为空且解析结果不为空，更新IP记录
    if len(req.domain) > 0 && (rec.A != nil || rec.AAAA != nil) {
        s.updateIP(req.domain, rec)
    }
}

func (s *ClassicNameServer) updateIP(domain string, newRec record) {
    // 加锁，确保并发安全
    s.Lock()

    // 记录调试信息，表示正在更新指定域名的IP记录
    newError(s.name, " updating IP records for domain:", domain).AtDebug().WriteToLog()
    rec := s.ips[domain]

    updated := false
    // 如果新的A记录更新时间更晚，更新A记录并标记为已更新
    if isNewer(rec.A, newRec.A) {
        rec.A = newRec.A
        updated = true
    }
    // 如果新的AAAA记录更新时间更晚，更新AAAA记录并标记为已更新
    if isNewer(rec.AAAA, newRec.AAAA) {
        rec.AAAA = newRec.AAAA
        updated = true
    }

    // 如果有更新，将更新后的记录存入IP记录中
    if updated {
        s.ips[domain] = rec
    }
    // 如果有新的A记录，发布对应的域名+4的消息
    if newRec.A != nil {
        s.pub.Publish(domain+"4", nil)
    }
    // 如果有新的AAAA记录，发布对应的域名+6的消息
    if newRec.AAAA != nil {
        s.pub.Publish(domain+"6", nil)
    }
    // 解锁
    s.Unlock()
    // 启动清理任务
    common.Must(s.cleanup.Start())
}

func (s *ClassicNameServer) newReqID() uint16 {
    // 生成新的请求ID
    return uint16(atomic.AddUint32(&s.reqID, 1))
}

func (s *ClassicNameServer) addPendingRequest(req *dnsRequest) {
    // 加锁，确保并发安全
    s.Lock()
    defer s.Unlock()

    id := req.msg.ID
    // 设置请求的过期时间，并将请求存入pending请求列表中
    req.expire = time.Now().Add(time.Second * 8)
    s.requests[id] = *req
}
func (s *ClassicNameServer) sendQuery(ctx context.Context, domain string, option IPOption) {
    // 在调试级别记录查询 DNS 的日志
    newError(s.name, " querying DNS for: ", domain).AtDebug().WriteToLog(session.ExportIDToError(ctx))

    // 构建 DNS 请求消息
    reqs := buildReqMsgs(domain, option, s.newReqID, genEDNS0Options(s.clientIP))

    // 遍历请求消息列表
    for _, req := range reqs {
        // 将请求消息添加到待处理请求列表
        s.addPendingRequest(req)
        // 打包 DNS 消息
        b, _ := dns.PackMessage(req.msg)
        // 创建 UDP 上下文
        udpCtx := context.Background()
        // 如果上下文中存在入站信息，则将其添加到 UDP 上下文中
        if inbound := session.InboundFromContext(ctx); inbound != nil {
            udpCtx = session.ContextWithInbound(udpCtx, inbound)
        }
        // 将协议类型添加到 UDP 上下文中
        udpCtx = session.ContextWithContent(udpCtx, &session.Content{
            Protocol: "dns",
        })
        // 分发 UDP 上下文和数据包到指定地址
        s.udpServer.Dispatch(udpCtx, s.address, b)
    }
}

func (s *ClassicNameServer) findIPsForDomain(domain string, option IPOption) ([]net.IP, error) {
    // 读取锁
    s.RLock()
    // 从 IP 地址映射中查找指定域名的记录
    record, found := s.ips[domain]
    // 释放锁
    s.RUnlock()

    // 如果未找到记录，则返回错误
    if !found {
        return nil, errRecordNotFound
    }

    var ips []net.Address
    var lastErr error
    // 如果 IPv4 启用，则获取 IPv4 地址
    if option.IPv4Enable {
        a, err := record.A.getIPs()
        if err != nil {
            lastErr = err
        }
        ips = append(ips, a...)
    }

    // 如果 IPv6 启用，则获取 IPv6 地址
    if option.IPv6Enable {
        aaaa, err := record.AAAA.getIPs()
        if err != nil {
            lastErr = err
        }
        ips = append(ips, aaaa...)
    }

    // 如果存在 IP 地址，则转换为 net.IP 类型并返回
    if len(ips) > 0 {
        return toNetIP(ips), nil
    }

    // 如果存在最后的错误，则返回该错误
    if lastErr != nil {
        return nil, lastErr
    }

    // 如果 IP 地址为空，则返回空响应错误
    return nil, dns_feature.ErrEmptyResponse
}

func (s *ClassicNameServer) QueryIP(ctx context.Context, domain string, option IPOption) ([]net.IP, error) {
    // 获取完全限定域名
    fqdn := Fqdn(domain)

    // 查找指定域名的 IP 地址
    ips, err := s.findIPsForDomain(fqdn, option)
    // 如果未找到记录，则返回缓存命中日志和错误
    if err != errRecordNotFound {
        newError(s.name, " cache HIT ", domain, " -> ", ips).Base(err).AtDebug().WriteToLog()
        return ips, err
    }

    // IPv4 和 IPv6 属于不同的订阅组
    var sub4, sub6 *pubsub.Subscriber
}
    # 如果 IPv4Enable 选项为真，则订阅 IPv4 主题，并在函数返回时关闭订阅
    if option.IPv4Enable {
        sub4 = s.pub.Subscribe(fqdn + "4")
        defer sub4.Close()
    }
    # 如果 IPv6Enable 选项为真，则订阅 IPv6 主题，并在函数返回时关闭订阅
    if option.IPv6Enable {
        sub6 = s.pub.Subscribe(fqdn + "6")
        defer sub6.Close()
    }
    # 创建一个通道用于通知后台 goroutine 完成
    done := make(chan interface{})
    # 启动后台 goroutine
    go func() {
        # 如果 sub4 不为空，则等待其完成或者上下文被取消
        if sub4 != nil {
            select {
            case <-sub4.Wait():
            case <-ctx.Done():
            }
        }
        # 如果 sub6 不为空，则等待其完成或者上下文被取消
        if sub6 != nil {
            select {
            case <-sub6.Wait():
            case <-ctx.Done():
            }
        }
        # 关闭通知通道
        close(done)
    }()
    # 发送查询请求
    s.sendQuery(ctx, fqdn, option)

    # 循环直到找到域名对应的 IP 地址或者上下文被取消
    for {
        # 查找域名对应的 IP 地址
        ips, err := s.findIPsForDomain(fqdn, option)
        # 如果出现错误且不是记录未找到错误，则返回错误
        if err != errRecordNotFound {
            return ips, err
        }

        # 等待上下文被取消或者后台 goroutine 完成
        select {
        case <-ctx.Done():
            return nil, ctx.Err()
        case <-done:
        }
    }
# 闭合前面的函数定义
```