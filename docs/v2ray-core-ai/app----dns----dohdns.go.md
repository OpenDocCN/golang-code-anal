# `v2ray-core\app\dns\dohdns.go`

```
// +build !confonly

package dns

import (
    "bytes" // 导入 bytes 包，用于操作二进制数据
    "context" // 导入 context 包，用于跟踪请求的上下文
    "fmt" // 导入 fmt 包，用于格式化输出
    "io" // 导入 io 包，用于进行 I/O 操作
    "io/ioutil" // 导入 ioutil 包，用于进行 I/O 操作
    "net/http" // 导入 http 包，用于进行 HTTP 请求
    "net/url" // 导入 url 包，用于解析和生成 URL
    "sync" // 导入 sync 包，用于实现同步原语
    "sync/atomic" // 导入 atomic 包，用于实现原子操作
    "time" // 导入 time 包，用于处理时间

    "golang.org/x/net/dns/dnsmessage" // 导入 DNS 消息格式相关的包
    "v2ray.com/core/common" // 导入 v2ray 核心通用包
    "v2ray.com/core/common/net" // 导入 v2ray 核心网络通用包
    "v2ray.com/core/common/protocol/dns" // 导入 v2ray 核心 DNS 协议包
    "v2ray.com/core/common/session" // 导入 v2ray 核心会话包
    "v2ray.com/core/common/signal/pubsub" // 导入 v2ray 核心信号发布订阅包
    "v2ray.com/core/common/task" // 导入 v2ray 核心任务包
    dns_feature "v2ray.com/core/features/dns" // 导入 v2ray 核心 DNS 功能包
    "v2ray.com/core/features/routing" // 导入 v2ray 核心路由功能包
    "v2ray.com/core/transport/internet" // 导入 v2ray 核心网络传输包
)

// DoHNameServer implemented DNS over HTTPS (RFC8484) Wire Format,
// which is compatible with traditional dns over udp(RFC1035),
// thus most of the DOH implementation is copied from udpns.go
type DoHNameServer struct {
    sync.RWMutex // 读写锁，用于保护共享资源
    ips        map[string]record // 存储 IP 地址和记录的映射关系
    pub        *pubsub.Service // 发布订阅服务
    cleanup    *task.Periodic // 定期清理任务
    reqID      uint32 // 请求 ID
    clientIP   net.IP // 客户端 IP 地址
    httpClient *http.Client // HTTP 客户端
    dohURL     string // DOH URL
    name       string // 名称
}

// NewDoHNameServer creates DOH client object for remote resolving
func NewDoHNameServer(url *url.URL, dispatcher routing.Dispatcher, clientIP net.IP) (*DoHNameServer, error) {
    // 创建远程解析的 DOH 客户端对象
    newError("DNS: created Remote DOH client for ", url.String()).AtInfo().WriteToLog()
    // 创建基础的 DOH 名称服务器
    s := baseDOHNameServer(url, "DOH", clientIP)

    // Dispatched connection will be closed (interrupted) after each request
    // This makes DOH inefficient without a keep-alived connection
    // See: core/app/proxyman/outbound/handler.go:113
    // Using mux (https request wrapped in a stream layer) improves the situation.
    // Recommend to use NewDoHLocalNameServer (DOHL:) if v2ray instance is running on
    //  a normal network eg. the server side of v2ray
    // 分发的连接将在每个请求后关闭（中断）
    // 这使得 DOH 在没有保持连接的情况下效率低下
    // 参见：core/app/proxyman/outbound/handler.go:113
    // 使用 mux（在流层中包装的 HTTPS 请求）可以改善这种情况
    // 如果 v2ray 实例在正常网络上运行，例如 v2ray 的服务器端，则建议使用 NewDoHLocalNameServer（DOHL：）
    # 创建一个自定义的 HTTP 传输对象
    tr := &http.Transport{
        # 设置最大空闲连接数
        MaxIdleConns:        30,
        # 设置空闲连接的超时时间
        IdleConnTimeout:     90 * time.Second,
        # 设置 TLS 握手的超时时间
        TLSHandshakeTimeout: 30 * time.Second,
        # 强制尝试使用 HTTP/2
        ForceAttemptHTTP2:   true,
        # 自定义 DialContext 函数，用于创建网络连接
        DialContext: func(ctx context.Context, network, addr string) (net.Conn, error) {
            # 解析网络地址
            dest, err := net.ParseDestination(network + ":" + addr)
            if err != nil {
                return nil, err
            }
            # 调度网络连接
            link, err := dispatcher.Dispatch(ctx, dest)
            if err != nil {
                return nil, err
            }
            # 创建新的网络连接
            return net.NewConnection(
                net.ConnectionInputMulti(link.Writer),
                net.ConnectionOutputMulti(link.Reader),
            ), nil
        },
    }

    # 创建一个使用自定义传输对象的 HTTP 客户端
    dispatchedClient := &http.Client{
        Transport: tr,
        # 设置超时时间
        Timeout:   60 * time.Second,
    }

    # 将创建的客户端赋值给 s.httpClient
    s.httpClient = dispatchedClient
    # 返回 s 和 nil
    return s, nil
// NewDoHLocalNameServer 创建本地解析的 DOH 客户端对象
func NewDoHLocalNameServer(url *url.URL, clientIP net.IP) *DoHNameServer {
    // 设置 URL 的协议为 https
    url.Scheme = "https"
    // 创建基础的 DOH 名称服务器对象
    s := baseDOHNameServer(url, "DOHL", clientIP)
    // 创建 HTTP 传输对象
    tr := &http.Transport{
        IdleConnTimeout:   90 * time.Second,  // 设置空闲连接超时时间
        ForceAttemptHTTP2: true,  // 强制使用 HTTP/2
        DialContext: func(ctx context.Context, network, addr string) (net.Conn, error) {
            // 解析目标地址
            dest, err := net.ParseDestination(network + ":" + addr)
            if err != nil {
                return nil, err
            }
            // 使用系统默认方式拨号
            conn, err := internet.DialSystem(ctx, dest, nil)
            if err != nil {
                return nil, err
            }
            return conn, nil
        },
    }
    // 设置 HTTP 客户端对象
    s.httpClient = &http.Client{
        Timeout:   time.Second * 180,  // 设置超时时间
        Transport: tr,  // 设置传输对象
    }
    // 记录日志
    newError("DNS: created Local DOH client for ", url.String()).AtInfo().WriteToLog()
    return s
}

// baseDOHNameServer 创建基础的 DOH 名称服务器对象
func baseDOHNameServer(url *url.URL, prefix string, clientIP net.IP) *DoHNameServer {
    // 创建 DOH 名称服务器对象
    s := &DoHNameServer{
        ips:      make(map[string]record),  // 初始化 IP 地址映射表
        clientIP: clientIP,  // 设置客户端 IP 地址
        pub:      pubsub.NewService(),  // 创建发布订阅服务
        name:     prefix + "//" + url.Host,  // 设置名称
        dohURL:   url.String(),  // 设置 DOH URL
    }
    // 设置清理任务
    s.cleanup = &task.Periodic{
        Interval: time.Minute,  // 设置执行间隔
        Execute:  s.Cleanup,  // 设置执行函数
    }
    return s
}

// Name 返回客户端名称
func (s *DoHNameServer) Name() string {
    return s.name
}

// Cleanup 清理缓存中过期的条目
func (s *DoHNameServer) Cleanup() error {
    now := time.Now()  // 获取当前时间
    s.Lock()  // 加锁
    defer s.Unlock()  // 解锁

    if len(s.ips) == 0 {
        return newError("nothing to do. stopping...")  // 如果 IP 地址映射表为空，则返回错误信息
    }
}
    # 遍历 s.ips 中的域名和记录
    for domain, record := range s.ips {
        # 如果 A 记录存在且过期时间在当前时间之前，则将其置空
        if record.A != nil && record.A.Expire.Before(now) {
            record.A = nil
        }
        # 如果 AAAA 记录存在且过期时间在当前时间之前，则将其置空
        if record.AAAA != nil && record.AAAA.Expire.Before(now) {
            record.AAAA = nil
        }

        # 如果 A 和 AAAA 记录都为空
        if record.A == nil && record.AAAA == nil {
            # 输出调试信息并删除该域名的记录
            newError(s.name, " cleanup ", domain).AtDebug().WriteToLog()
            delete(s.ips, domain)
        } else {
            # 否则更新 s.ips 中的记录
            s.ips[domain] = record
        }
    }

    # 如果 s.ips 中没有记录了，则重新创建一个空的记录映射
    if len(s.ips) == 0 {
        s.ips = make(map[string]record)
    }

    # 返回空
    return nil
// 更新 IP 地址记录，包括对应的域名请求和 IP 地址记录
func (s *DoHNameServer) updateIP(req *dnsRequest, ipRec *IPRecord) {
    // 计算请求处理时间
    elapsed := time.Since(req.start)

    // 加锁，确保并发安全
    s.Lock()
    // 获取对应域名的 IP 地址记录
    rec := s.ips[req.domain]
    updated := false

    // 根据请求类型进行不同的处理
    switch req.reqType {
    case dnsmessage.TypeA:
        // 如果新的 IP 地址记录比原记录更新，则更新记录
        if isNewer(rec.A, ipRec) {
            rec.A = ipRec
            updated = true
        }
    case dnsmessage.TypeAAAA:
        // 如果是 IPv6 地址，则更新记录
        addr := make([]net.Address, 0)
        for _, ip := range ipRec.IP {
            if len(ip.IP()) == net.IPv6len {
                addr = append(addr, ip)
            }
        }
        ipRec.IP = addr
        if isNewer(rec.AAAA, ipRec) {
            rec.AAAA = ipRec
            updated = true
        }
    }
    // 记录日志
    newError(s.name, " got answer: ", req.domain, " ", req.reqType, " -> ", ipRec.IP, " ", elapsed).AtInfo().WriteToLog()

    // 如果有更新，则更新 IP 地址记录
    if updated {
        s.ips[req.domain] = rec
    }
    // 根据请求类型发布消息
    switch req.reqType {
    case dnsmessage.TypeA:
        s.pub.Publish(req.domain+"4", nil)
    case dnsmessage.TypeAAAA:
        s.pub.Publish(req.domain+"6", nil)
    }
    // 解锁
    s.Unlock()
    // 启动清理任务
    common.Must(s.cleanup.Start())
}

// 生成新的请求 ID
func (s *DoHNameServer) newReqID() uint16 {
    return uint16(atomic.AddUint32(&s.reqID, 1))
}

// 发送查询请求
func (s *DoHNameServer) sendQuery(ctx context.Context, domain string, option IPOption) {
    // 记录日志
    newError(s.name, " querying: ", domain).AtInfo().WriteToLog(session.ExportIDToError(ctx))

    // 构建请求消息
    reqs := buildReqMsgs(domain, option, s.newReqID, genEDNS0Options(s.clientIP))

    // 设置超时时间
    var deadline time.Time
    if d, ok := ctx.Deadline(); ok {
        deadline = d
    } else {
        deadline = time.Now().Add(time.Second * 5)
    }
}
    // 遍历 reqs 切片中的每个请求
    for _, req := range reqs {
        // 使用匿名函数处理每个请求，生成新的上下文，使用相同的上下文可能导致所有请求在遇到错误时都被中止
        go func(r *dnsRequest) {
            // 为每个请求生成新的上下文，使用相同的上下文可能导致所有请求在遇到错误时都被中止
            dnsCtx := context.Background()

            // 保留内部 DNS 服务器请求的入站
            if inbound := session.InboundFromContext(ctx); inbound != nil {
                dnsCtx = session.ContextWithInbound(dnsCtx, inbound)
            }

            // 在上下文中设置内容信息
            dnsCtx = session.ContextWithContent(dnsCtx, &session.Content{
                Protocol:      "https",
                SkipRoutePick: true,
            })

            // 强制使用 mux 进行 DOH
            dnsCtx = session.ContextWithMuxPrefered(dnsCtx, true)

            // 设置上下文的截止时间，并返回取消函数
            var cancel context.CancelFunc
            dnsCtx, cancel = context.WithDeadline(dnsCtx, deadline)
            defer cancel()

            // 将 DNS 消息打包成字节流
            b, err := dns.PackMessage(r.msg)
            if err != nil {
                newError("failed to pack dns query").Base(err).AtError().WriteToLog()
                return
            }
            // 使用 DOH 进行 HTTPS 请求
            resp, err := s.dohHTTPSContext(dnsCtx, b.Bytes())
            if err != nil {
                newError("failed to retrieve response").Base(err).AtError().WriteToLog()
                return
            }
            // 解析响应
            rec, err := parseResponse(resp)
            if err != nil {
                newError("failed to handle DOH response").Base(err).AtError().WriteToLog()
                return
            }
            // 更新 IP 地址
            s.updateIP(r, rec)
        }(req)
    }
// 使用HTTPS协议进行DNS over HTTPS查询，传入上下文和数据，返回查询结果或错误
func (s *DoHNameServer) dohHTTPSContext(ctx context.Context, b []byte) ([]byte, error) {
    // 将数据封装成字节流
    body := bytes.NewBuffer(b)
    // 创建一个HTTP POST请求
    req, err := http.NewRequest("POST", s.dohURL, body)
    if err != nil {
        return nil, err
    }

    // 添加请求头信息
    req.Header.Add("Accept", "application/dns-message")
    req.Header.Add("Content-Type", "application/dns-message")

    // 发起HTTP请求并获取响应
    resp, err := s.httpClient.Do(req.WithContext(ctx))
    if err != nil {
        return nil, err
    }

    // 延迟关闭响应体
    defer resp.Body.Close()
    // 如果响应状态码不是200，丢弃响应体并返回错误
    if resp.StatusCode != http.StatusOK {
        io.Copy(ioutil.Discard, resp.Body) // flush resp.Body so that the conn is reusable
        return nil, fmt.Errorf("DOH server returned code %d", resp.StatusCode)
    }

    // 读取并返回响应体的全部内容
    return ioutil.ReadAll(resp.Body)
}

// 根据域名查找对应的IP地址，传入域名和IP选项，返回IP地址列表或错误
func (s *DoHNameServer) findIPsForDomain(domain string, option IPOption) ([]net.IP, error) {
    // 加读锁
    s.RLock()
    // 从缓存中查找域名对应的记录
    record, found := s.ips[domain]
    // 解锁
    s.RUnlock()

    // 如果未找到记录，返回错误
    if !found {
        return nil, errRecordNotFound
    }

    var ips []net.Address
    var lastErr error
    // 如果IPv6启用且记录中存在IPv6地址，则获取并添加到IP列表中
    if option.IPv6Enable && record.AAAA != nil && record.AAAA.RCode == dnsmessage.RCodeSuccess {
        aaaa, err := record.AAAA.getIPs()
        if err != nil {
            lastErr = err
        }
        ips = append(ips, aaaa...)
    }

    // 如果IPv4启用且记录中存在IPv4地址，则获取并添加到IP列表中
    if option.IPv4Enable && record.A != nil && record.A.RCode == dnsmessage.RCodeSuccess {
        a, err := record.A.getIPs()
        if err != nil {
            lastErr = err
        }
        ips = append(ips, a...)
    }

    // 如果IP列表不为空，转换为net.IP类型并返回
    if len(ips) > 0 {
        return toNetIP(ips), nil
    }

    // 如果有最后的错误，返回该错误
    if lastErr != nil {
        return nil, lastErr
    }

    // 如果IPv4或IPv6启用但记录中没有对应地址，返回空响应错误
    if (option.IPv4Enable && record.A != nil) || (option.IPv6Enable && record.AAAA != nil) {
        return nil, dns_feature.ErrEmptyResponse
    }

    // 如果记录中没有对应地址，返回记录未找到错误
    return nil, errRecordNotFound
}

// 从dns.Server->queryIPTimeout调用，根据域名查询IP地址，传入上下文、域名和IP选项，返回IP地址列表或错误
func (s *DoHNameServer) QueryIP(ctx context.Context, domain string, option IPOption) ([]net.IP, error) {
    // 将域名转换为全限定域名
    fqdn := Fqdn(domain);
    // 调用 findIPsForDomain 方法查找域名对应的 IP 地址
    ips, err := s.findIPsForDomain(fqdn, option)
    // 如果错误不是记录未找到，则记录缓存命中并返回结果
    if err != errRecordNotFound {
        newError(s.name, " cache HIT ", domain, " -> ", ips).Base(err).AtDebug().WriteToLog()
        return ips, err
    }

    // ipv4 和 ipv6 属于不同的订阅组
    var sub4, sub6 *pubsub.Subscriber
    // 如果启用 IPv4，则订阅对应的频道并在函数返回时关闭订阅
    if option.IPv4Enable {
        sub4 = s.pub.Subscribe(fqdn + "4")
        defer sub4.Close()
    }
    // 如果启用 IPv6，则订阅对应的频道并在函数返回时关闭订阅
    if option.IPv6Enable {
        sub6 = s.pub.Subscribe(fqdn + "6")
        defer sub6.Close()
    }
    // 创建一个通道用于通知查询完成
    done := make(chan interface{})
    // 启动一个 goroutine，等待订阅的消息或者上下文取消
    go func() {
        if sub4 != nil {
            select {
            case <-sub4.Wait():
            case <-ctx.Done():
            }
        }
        if sub6 != nil {
            select {
            case <-sub6.Wait():
            case <-ctx.Done():
            }
        }
        close(done)
    }()
    // 发送查询请求
    s.sendQuery(ctx, fqdn, option)

    // 循环等待查询结果或者上下文取消
    for {
        ips, err := s.findIPsForDomain(fqdn, option)
        if err != errRecordNotFound {
            return ips, err
        }

        select {
        case <-ctx.Done():
            return nil, ctx.Err()
        case <-done:
        }
    }
# 闭合前面的函数定义
```