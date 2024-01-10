# `v2ray-core\app\dns\dnscommon.go`

```
// +build !confonly

package dns

import (
    "encoding/binary" // 导入二进制编码包
    "time" // 导入时间包

    "golang.org/x/net/dns/dnsmessage" // 导入 DNS 消息包
    "v2ray.com/core/common" // 导入通用包
    "v2ray.com/core/common/errors" // 导入错误处理包
    "v2ray.com/core/common/net" // 导入网络包
    dns_feature "v2ray.com/core/features/dns" // 导入 DNS 功能包
)

// Fqdn normalize domain make sure it ends with '.'
// Fqdn 函数用于规范化域名，确保以 '.' 结尾
func Fqdn(domain string) string {
    if len(domain) > 0 && domain[len(domain)-1] == '.' {
        return domain
    }
    return domain + "."
}

type record struct {
    A    *IPRecord // A 记录
    AAAA *IPRecord // AAAA 记录
}

// IPRecord is a cacheable item for a resolved domain
// IPRecord 是一个可缓存的已解析域名项
type IPRecord struct {
    ReqID  uint16 // 请求 ID
    IP     []net.Address // IP 地址
    Expire time.Time // 过期时间
    RCode  dnsmessage.RCode // 响应码
}

func (r *IPRecord) getIPs() ([]net.Address, error) {
    if r == nil || r.Expire.Before(time.Now()) {
        return nil, errRecordNotFound
    }
    if r.RCode != dnsmessage.RCodeSuccess {
        return nil, dns_feature.RCodeError(r.RCode)
    }
    return r.IP, nil
}

func isNewer(baseRec *IPRecord, newRec *IPRecord) bool {
    if newRec == nil {
        return false
    }
    if baseRec == nil {
        return true
    }
    return baseRec.Expire.Before(newRec.Expire)
}

var (
    errRecordNotFound = errors.New("record not found") // 记录未找到错误
)

type dnsRequest struct {
    reqType dnsmessage.Type // 请求类型
    domain  string // 域名
    start   time.Time // 开始时间
    expire  time.Time // 过期时间
    msg     *dnsmessage.Message // DNS 消息
}

func genEDNS0Options(clientIP net.IP) *dnsmessage.Resource {
    if len(clientIP) == 0 {
        return nil
    }

    var netmask int // 网络掩码
    var family uint16 // 地址族

    if len(clientIP) == 4 {
        family = 1 // IPv4 地址族
        netmask = 24 // 24 位掩码，用于 IPv4
    } else {
        family = 2 // IPv6 地址族
        netmask = 96 // 96 位掩码，用于 IPv6
    }

    b := make([]byte, 4) // 创建一个长度为 4 的字节切片
    binary.BigEndian.PutUint16(b[0:], family) // 将地址族写入字节切片
    b[2] = byte(netmask) // 写入网络掩码
    b[3] = 0 // 设置为 0
    switch family {
    # 根据不同情况对客户端 IP 进行处理
    case 1:
        # 对 IPv4 地址进行掩码处理
        ip := clientIP.To4().Mask(net.CIDRMask(netmask, net.IPv4len*8))
        # 计算需要的长度，向上取整
        needLength := (netmask + 8 - 1) / 8 // division rounding up
        # 将处理后的 IP 地址添加到字节流中
        b = append(b, ip[:needLength]...)
    case 2:
        # 对 IPv6 地址进行掩码处理
        ip := clientIP.Mask(net.CIDRMask(netmask, net.IPv6len*8))
        # 计算需要的长度，向上取整
        needLength := (netmask + 8 - 1) / 8 // division rounding up
        # 将处理后的 IP 地址添加到字节流中
        b = append(b, ip[:needLength]...)

    # 定义 EDNS0SUBNET 常量
    const EDNS0SUBNET = 0x08

    # 创建一个 DNS 资源对象
    opt := new(dnsmessage.Resource)
    # 设置 EDNS0 参数
    common.Must(opt.Header.SetEDNS0(1350, 0xfe00, true))

    # 设置 DNS 选项
    opt.Body = &dnsmessage.OPTResource{
        Options: []dnsmessage.Option{
            {
                Code: EDNS0SUBNET,
                Data: b,
            },
        },
    }

    # 返回设置好的 DNS 选项
    return opt
// 根据给定的域名、选项、请求ID生成器和请求选项构建DNS请求消息数组
func buildReqMsgs(domain string, option IPOption, reqIDGen func() uint16, reqOpts *dnsmessage.Resource) []*dnsRequest {
    // 创建查询类型为A的DNS问题
    qA := dnsmessage.Question{
        Name:  dnsmessage.MustNewName(domain),
        Type:  dnsmessage.TypeA,
        Class: dnsmessage.ClassINET,
    }

    // 创建查询类型为AAAA的DNS问题
    qAAAA := dnsmessage.Question{
        Name:  dnsmessage.MustNewName(domain),
        Type:  dnsmessage.TypeAAAA,
        Class: dnsmessage.ClassINET,
    }

    // 初始化DNS请求数组
    var reqs []*dnsRequest
    // 获取当前时间
    now := time.Now()

    // 如果IPv4启用
    if option.IPv4Enable {
        // 创建新的DNS消息
        msg := new(dnsmessage.Message)
        // 设置消息头部ID
        msg.Header.ID = reqIDGen()
        // 设置递归期望为true
        msg.Header.RecursionDesired = true
        // 设置消息的问题部分为查询类型为A的问题
        msg.Questions = []dnsmessage.Question{qA}
        // 如果请求选项不为空，则添加到附加部分
        if reqOpts != nil {
            msg.Additionals = append(msg.Additionals, *reqOpts)
        }
        // 将DNS请求添加到请求数组中
        reqs = append(reqs, &dnsRequest{
            reqType: dnsmessage.TypeA,
            domain:  domain,
            start:   now,
            msg:     msg,
        })
    }

    // 如果IPv6启用
    if option.IPv6Enable {
        // 创建新的DNS消息
        msg := new(dnsmessage.Message)
        // 设置消息头部ID
        msg.Header.ID = reqIDGen()
        // 设置递归期望为true
        msg.Header.RecursionDesired = true
        // 设置消息的问题部分为查询类型为AAAA的问题
        msg.Questions = []dnsmessage.Question{qAAAA}
        // 如果请求选项不为空，则添加到附加部分
        if reqOpts != nil {
            msg.Additionals = append(msg.Additionals, *reqOpts)
        }
        // 将DNS请求添加到请求数组中
        reqs = append(reqs, &dnsRequest{
            reqType: dnsmessage.TypeAAAA,
            domain:  domain,
            start:   now,
            msg:     msg,
        })
    }

    // 返回DNS请求数组
    return reqs
}

// parseResponse 从返回的载荷中解析DNS答案
func parseResponse(payload []byte) (*IPRecord, error) {
    // 创建DNS消息解析器
    var parser dnsmessage.Parser
    // 开始解析载荷
    h, err := parser.Start(payload)
    // 如果解析失败，则返回错误
    if err != nil {
        return nil, newError("failed to parse DNS response").Base(err).AtWarning()
    }
    // 跳过所有问题部分
    if err := parser.SkipAllQuestions(); err != nil {
        return nil, newError("failed to skip questions in DNS response").Base(err).AtWarning()
    }

    // 获取当前时间
    now := time.Now()
    # 创建一个新的IPRecord结构体对象，并初始化其ReqID、RCode和Expire字段
    ipRecord := &IPRecord{
        ReqID:  h.ID,  # 将h.ID赋值给ipRecord的ReqID字段
        RCode:  h.RCode,  # 将h.RCode赋值给ipRecord的RCode字段
        Expire: now.Add(time.Second * 600),  # 将当前时间加上600秒后的时间赋值给ipRecord的Expire字段
    }
# 使用无限循环来处理 DNS 解析结果的每个回答部分
L:
    # 从解析器中获取回答部分的头部信息
    ah, err := parser.AnswerHeader()
    # 如果出现错误
    if err != nil:
        # 如果错误不是表示部分结束，则记录错误信息并终止循环
        if err != dnsmessage.ErrSectionDone:
            newError("failed to parse answer section for domain: ", ah.Name.String()).Base(err).WriteToLog()
        break
    # 获取 TTL 值，如果为 0 则设置为默认值 600
    ttl := ah.TTL
    if ttl == 0:
        ttl = 600
    # 计算记录过期时间
    expire := now.Add(time.Duration(ttl) * time.Second)
    # 如果记录的过期时间在新计算的过期时间之后，则更新过期时间
    if ipRecord.Expire.After(expire):
        ipRecord.Expire = expire
    # 根据回答类型进行不同的处理
    switch ah.Type:
        # 处理 A 类型记录
        case dnsmessage.TypeA:
            # 从解析器中获取 A 类型资源记录
            ans, err := parser.AResource()
            # 如果出现错误，则记录错误信息并终止循环
            if err != nil:
                newError("failed to parse A record for domain: ", ah.Name).Base(err).WriteToLog()
                break L
            # 将 IP 地址添加到记录中
            ipRecord.IP = append(ipRecord.IP, net.IPAddress(ans.A[:]))
        # 处理 AAAA 类型记录
        case dnsmessage.TypeAAAA:
            # 从解析器中获取 AAAA 类型资源记录
            ans, err := parser.AAAAResource()
            # 如果出现错误，则记录错误信息并终止循环
            if err != nil:
                newError("failed to parse A record for domain: ", ah.Name).Base(err).WriteToLog()
                break L
            # 将 IP 地址添加到记录中
            ipRecord.IP = append(ipRecord.IP, net.IPAddress(ans.AAAA[:]))
        # 处理其他类型记录
        default:
            # 跳过当前回答部分并记录错误信息
            if err := parser.SkipAnswer(); err != nil:
                newError("failed to skip answer").Base(err).WriteToLog()
                break L
            # 继续下一次循环
            continue
    # 返回 IP 记录和空错误
    return ipRecord, nil
```