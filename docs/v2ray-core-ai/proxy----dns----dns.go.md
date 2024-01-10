# `v2ray-core\proxy\dns\dns.go`

```
// +build !confonly

package dns

import (
    "context"  // 导入上下文包，用于处理请求上下文
    "io"  // 导入输入输出包，用于处理输入输出流
    "sync"  // 导入同步包，用于实现并发控制

    "golang.org/x/net/dns/dnsmessage"  // 导入 DNS 消息包，用于处理 DNS 消息

    "v2ray.com/core"  // 导入 V2Ray 核心包
    "v2ray.com/core/common"  // 导入 V2Ray 公共包
    "v2ray.com/core/common/buf"  // 导入 V2Ray 缓冲区包
    "v2ray.com/core/common/net"  // 导入 V2Ray 网络包
    dns_proto "v2ray.com/core/common/protocol/dns"  // 导入 V2Ray DNS 协议包
    "v2ray.com/core/common/session"  // 导入 V2Ray 会话包
    "v2ray.com/core/common/task"  // 导入 V2Ray 任务包
    "v2ray.com/core/features/dns"  // 导入 V2Ray DNS 功能包
    "v2ray.com/core/transport"  // 导入 V2Ray 传输包
    "v2ray.com/core/transport/internet"  // 导入 V2Ray 互联网传输包
)

func init() {
    // 注册 DNS 配置
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        h := new(Handler)
        // 要求上下文中存在 DNS 客户端，初始化 DNS 处理器
        if err := core.RequireFeatures(ctx, func(dnsClient dns.Client) error {
            return h.Init(config.(*Config), dnsClient)
        }); err != nil {
            return nil, err
        }
        return h, nil
    }))
}

type ownLinkVerifier interface {
    IsOwnLink(ctx context.Context) bool
}

type Handler struct {
    ipv4Lookup      dns.IPv4Lookup  // IPv4 查询
    ipv6Lookup      dns.IPv6Lookup  // IPv6 查询
    ownLinkVerifier ownLinkVerifier  // 自有链接验证器
    server          net.Destination  // 服务器目标地址
}

func (h *Handler) Init(config *Config, dnsClient dns.Client) error {
    ipv4lookup, ok := dnsClient.(dns.IPv4Lookup)
    if !ok {
        return newError("dns.Client doesn't implement IPv4Lookup")
    }
    h.ipv4Lookup = ipv4lookup  // 初始化 IPv4 查询

    ipv6lookup, ok := dnsClient.(dns.IPv6Lookup)
    if !ok {
        return newError("dns.Client doesn't implement IPv6Lookup")
    }
    h.ipv6Lookup = ipv6lookup  // 初始化 IPv6 查询

    if v, ok := dnsClient.(ownLinkVerifier); ok {
        h.ownLinkVerifier = v  // 初始化自有链接验证器
    }

    if config.Server != nil {
        h.server = config.Server.AsDestination()  // 初始化服务器目标地址
    }
    return nil
}

func (h *Handler) isOwnLink(ctx context.Context) bool {
    return h.ownLinkVerifier != nil && h.ownLinkVerifier.IsOwnLink(ctx)  // 判断是否为自有链接
}

func parseIPQuery(b []byte) (r bool, domain string, id uint16, qType dnsmessage.Type) {
    var parser dnsmessage.Parser  // 创建 DNS 消息解析器
    header, err := parser.Start(b)  // 解析 DNS 消息头
    # 如果发生错误，则记录错误信息并返回
    if err != nil:
        newError("parser start").Base(err).WriteToLog()
        return

    # 获取报文头部的 ID
    id = header.ID
    # 解析报文中的问题部分
    q, err := parser.Question()
    # 如果解析出错，则记录错误信息并返回
    if err != nil:
        newError("question").Base(err).WriteToLog()
        return
    # 获取问题部分的类型
    qType = q.Type
    # 如果问题类型不是 A 类型和 AAAA 类型，则直接返回
    if qType != dnsmessage.TypeA && qType != dnsmessage.TypeAAAA:
        return

    # 获取域名字符串
    domain = q.Name.String()
    # 设置返回值为 true
    r = true
    # 返回结果
    return
// Process 实现了 proxy.Outbound 接口
func (h *Handler) Process(ctx context.Context, link *transport.Link, d internet.Dialer) error {
    // 从上下文中获取出站代理信息
    outbound := session.OutboundFromContext(ctx)
    // 如果出站代理信息为空或目标地址无效，则返回错误
    if outbound == nil || !outbound.Target.IsValid() {
        return newError("invalid outbound")
    }

    // 获取出站代理的网络类型
    srcNetwork := outbound.Target.Network

    // 将目标地址设置为出站代理的目标地址
    dest := outbound.Target
    // 如果服务器的网络类型不为未知，则将目标地址的网络类型设置为服务器的网络类型
    if h.server.Network != net.Network_Unknown {
        dest.Network = h.server.Network
    }
    // 如果服务器的地址不为空，则将目标地址的地址设置为服务器的地址
    if h.server.Address != nil {
        dest.Address = h.server.Address
    }
    // 如果服务器的端口不为0，则将目标地址的端口设置为服务器的端口
    if h.server.Port != 0 {
        dest.Port = h.server.Port
    }

    // 记录处理 DNS 流量到目标地址的日志
    newError("handling DNS traffic to ", dest).WriteToLog(session.ExportIDToError(ctx))

    // 创建出站连接对象
    conn := &outboundConn{
        dialer: func() (internet.Connection, error) {
            return d.Dial(ctx, dest)
        },
        connReady: make(chan struct{}, 1),
    }

    // 创建 DNS 消息读取器和写入器
    var reader dns_proto.MessageReader
    var writer dns_proto.MessageWriter
    // 如果出站代理的网络类型为 TCP，则使用 TCPReader 和 TCPWriter
    if srcNetwork == net.Network_TCP {
        reader = dns_proto.NewTCPReader(link.Reader)
        writer = &dns_proto.TCPWriter{
            Writer: link.Writer,
        }
    } else {
        // 否则使用 UDPReader 和 UDPWriter
        reader = &dns_proto.UDPReader{
            Reader: link.Reader,
        }
        writer = &dns_proto.UDPWriter{
            Writer: link.Writer,
        }
    }

    // 创建出站连接的 DNS 消息读取器和写入器
    var connReader dns_proto.MessageReader
    var connWriter dns_proto.MessageWriter
    // 如果目标地址的网络类型为 TCP，则使用 TCPReader 和 TCPWriter
    if dest.Network == net.Network_TCP {
        connReader = dns_proto.NewTCPReader(buf.NewReader(conn))
        connWriter = &dns_proto.TCPWriter{
            Writer: buf.NewWriter(conn),
        }
    } else {
        // 否则使用 UDPReader 和 UDPWriter
        connReader = &dns_proto.UDPReader{
            Reader: buf.NewPacketReader(conn),
        }
        connWriter = &dns_proto.UDPWriter{
            Writer: buf.NewWriter(conn),
        }
    }
}
    # 定义一个匿名函数，用于处理请求
    request := func() error {
        # 延迟关闭连接，确保在函数返回时关闭连接
        defer conn.Close()

        # 无限循环，处理请求
        for {
            # 读取消息
            b, err := reader.ReadMessage()
            # 如果读取到文件末尾，返回空
            if err == io.EOF {
                return nil
            }

            # 如果读取出错，返回错误
            if err != nil {
                return err
            }

            # 如果不是本地链接，解析 IP 查询
            if !h.isOwnLink(ctx) {
                isIPQuery, domain, id, qType := parseIPQuery(b.Bytes())
                # 如果是 IP 查询，开启协程处理
                if isIPQuery {
                    go h.handleIPQuery(id, qType, domain, writer)
                    continue
                }
            }

            # 如果不是 IP 查询，将消息写入连接
            if err := connWriter.WriteMessage(b); err != nil {
                return err
            }
        }
    }

    # 定义一个匿名函数，用于处理响应
    response := func() error {
        # 无限循环，处理响应
        for {
            # 读取消息
            b, err := connReader.ReadMessage()
            # 如果读取到文件末尾，返回空
            if err == io.EOF {
                return nil
            }

            # 如果读取出错，返回错误
            if err != nil {
                return err
            }

            # 将消息写入连接
            if err := writer.WriteMessage(b); err != nil {
                return err
            }
        }
    }

    # 运行任务，处理请求和响应
    if err := task.Run(ctx, request, response); err != nil {
        # 如果出错，返回连接结束的错误
        return newError("connection ends").Base(err)
    }

    # 没有错误，返回空
    return nil
// 处理 IP 查询的方法，根据查询类型和域名返回对应的 IP 地址
func (h *Handler) handleIPQuery(id uint16, qType dnsmessage.Type, domain string, writer dns_proto.MessageWriter) {
    var ips []net.IP  // 定义存储 IP 地址的变量
    var err error  // 定义存储错误信息的变量

    switch qType {  // 根据查询类型进行不同的处理
    case dnsmessage.TypeA:  // 如果查询类型为 A 记录
        ips, err = h.ipv4Lookup.LookupIPv4(domain)  // 调用 IPv4 查询方法获取对应的 IP 地址
    case dnsmessage.TypeAAAA:  // 如果查询类型为 AAAA 记录
        ips, err = h.ipv6Lookup.LookupIPv6(domain)  // 调用 IPv6 查询方法获取对应的 IP 地址
    }

    rcode := dns.RCodeFromError(err)  // 根据错误信息获取响应码
    if rcode == 0 && len(ips) == 0 && err != dns.ErrEmptyResponse {  // 如果响应码为 0 且 IP 地址列表为空且错误不是空响应错误
        newError("ip query").Base(err).WriteToLog()  // 记录错误信息并写入日志
        return  // 结束方法
    }

    b := buf.New()  // 创建新的缓冲区
    rawBytes := b.Extend(buf.Size)  // 扩展缓冲区大小
    builder := dnsmessage.NewBuilder(rawBytes[:0], dnsmessage.Header{  // 创建 DNS 消息构建器
        ID:                 id,  // 设置消息头部的 ID
        RCode:              dnsmessage.RCode(rcode),  // 设置消息头部的响应码
        RecursionAvailable: true,  // 设置消息头部的递归可用标志
        RecursionDesired:   true,  // 设置消息头部的递归期望标志
        Response:           true,  // 设置消息头部的响应标志
        Authoritative:      true,  // 设置消息头部的授权标志
    })
    builder.EnableCompression()  // 启用消息压缩
    common.Must(builder.StartQuestions())  // 开始添加问题部分
    common.Must(builder.Question(dnsmessage.Question{  // 添加问题
        Name:  dnsmessage.MustNewName(domain),  // 设置问题的域名
        Class: dnsmessage.ClassINET,  // 设置问题的类别为 INET
        Type:  qType,  // 设置问题的类型为查询类型
    }))
    common.Must(builder.StartAnswers())  // 开始添加回答部分

    rHeader := dnsmessage.ResourceHeader{Name: dnsmessage.MustNewName(domain), Class: dnsmessage.ClassINET, TTL: 600}  // 创建资源头部
    for _, ip := range ips {  // 遍历 IP 地址列表
        if len(ip) == net.IPv4len {  // 如果 IP 地址长度为 IPv4 长度
            var r dnsmessage.AResource  // 创建 A 记录资源
            copy(r.A[:], ip)  // 复制 IP 地址到资源中
            common.Must(builder.AResource(rHeader, r))  // 添加 A 记录资源到消息中
        } else {  // 如果 IP 地址长度不为 IPv4 长度
            var r dnsmessage.AAAAResource  // 创建 AAAA 记录资源
            copy(r.AAAA[:], ip)  // 复制 IP 地址到资源中
            common.Must(builder.AAAAResource(rHeader, r))  // 添加 AAAA 记录资源到消息中
        }
    }
    msgBytes, err := builder.Finish()  // 完成消息构建并获取消息字节
    if err != nil {  // 如果有错误发生
        newError("pack message").Base(err).WriteToLog()  // 记录错误信息并写入日志
        b.Release()  // 释放缓冲区
        return  // 结束方法
    }
    b.Resize(0, int32(len(msgBytes)))  // 调整缓冲区大小

    if err := writer.WriteMessage(b); err != nil {  // 如果写入消息时发生错误
        newError("write IP answer").Base(err).WriteToLog()  // 记录错误信息并写入日志
    }
}

type outboundConn struct {
    access sync.Mutex  // 定义同步锁
    # 定义一个名为dialer的函数，该函数返回一个internet.Connection类型的对象和一个错误
    dialer func() (internet.Connection, error)
    
    # 定义一个名为conn的net.Conn类型的变量
    conn      net.Conn
    # 定义一个名为connReady的通道，用于通知连接已经准备就绪
    connReady chan struct{}
# 拨号连接
func (c *outboundConn) dial() error:
    # 使用拨号器建立连接
    conn, err := c.dialer()
    # 如果出现错误，返回错误
    if err != nil:
        return err
    # 将连接赋值给对象的连接属性
    c.conn = conn
    # 向连接就绪通道发送信号
    c.connReady <- struct{}{}
    # 返回空值
    return nil

# 写入数据
func (c *outboundConn) Write(b []byte) (int, error):
    # 加锁
    c.access.Lock()
    # 如果连接为空
    if c.conn == nil:
        # 尝试拨号连接，如果失败则解锁并记录错误
        if err := c.dial(); err != nil:
            c.access.Unlock()
            newError("failed to dial outbound connection").Base(err).AtWarning().WriteToLog()
            return len(b), nil
    # 解锁
    c.access.Unlock()
    # 调用连接的写入方法
    return c.conn.Write(b)

# 读取数据
func (c *outboundConn) Read(b []byte) (int, error):
    # 定义连接变量
    var conn net.Conn
    # 加锁
    c.access.Lock()
    # 将连接赋值给变量
    conn = c.conn
    # 解锁
    c.access.Unlock()
    # 如果连接为空
    if conn == nil:
        # 从连接就绪通道读取数据，如果通道关闭则返回 EOF
        _, open := <-c.connReady
        if !open:
            return 0, io.EOF
        # 重新赋值连接
        conn = c.conn
    # 调用连接的读取方法
    return conn.Read(b)

# 关闭连接
func (c *outboundConn) Close() error:
    # 加锁
    c.access.Lock()
    # 关闭连接就绪通道
    close(c.connReady)
    # 如果连接不为空，关闭连接
    if c.conn != nil:
        c.conn.Close()
    # 解锁
    c.access.Unlock()
    # 返回空值
    return nil
```