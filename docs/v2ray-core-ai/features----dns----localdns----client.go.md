# `v2ray-core\features\dns\localdns\client.go`

```go
// 导入本地DNS包
package localdns

// 导入v2ray.com/core/common/net包
import (
    "v2ray.com/core/common/net"
    "v2ray.com/core/features/dns"
)

// Client是dns.Client的实现，用于查询本地DNS。
type Client struct{}

// Type实现common.HasType接口。
func (*Client) Type() interface{} {
    return dns.ClientType()
}

// Start实现common.Runnable接口。
func (*Client) Start() error { return nil }

// Close实现common.Closable接口。
func (*Client) Close() error { return nil }

// LookupIP实现Client接口。
func (*Client) LookupIP(host string) ([]net.IP, error) {
    // 查询主机的IP地址
    ips, err := net.LookupIP(host)
    if err != nil {
        return nil, err
    }
    parsedIPs := make([]net.IP, 0, len(ips))
    for _, ip := range ips {
        parsed := net.IPAddress(ip)
        if parsed != nil {
            parsedIPs = append(parsedIPs, parsed.IP())
        }
    }
    if len(parsedIPs) == 0 {
        return nil, dns.ErrEmptyResponse
    }
    return parsedIPs, nil
}

// LookupIPv4实现IPv4Lookup接口。
func (c *Client) LookupIPv4(host string) ([]net.IP, error) {
    ips, err := c.LookupIP(host)
    if err != nil {
        return nil, err
    }
    ipv4 := make([]net.IP, 0, len(ips))
    for _, ip := range ips {
        if len(ip) == net.IPv4len {
            ipv4 = append(ipv4, ip)
        }
    }
    if len(ipv4) == 0 {
        return nil, dns.ErrEmptyResponse
    }
    return ipv4, nil
}

// LookupIPv6实现IPv6Lookup接口。
func (c *Client) LookupIPv6(host string) ([]net.IP, error) {
    ips, err := c.LookupIP(host)
    if err != nil {
        return nil, err
    }
    ipv6 := make([]net.IP, 0, len(ips))
    for _, ip := range ips {
        if len(ip) == net.IPv6len {
            ipv6 = append(ipv6, ip)
        }
    }
    if len(ipv6) == 0 {
        return nil, dns.ErrEmptyResponse
    }
    return ipv6, nil
}

// New创建一个新的dns.Client，用于查询本地DNS。
func New() *Client {
    return &Client{}
}
```