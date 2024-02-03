# `v2ray-core\app\dns\nameserver.go`

```go
// +build !confonly
// 标记此文件不仅仅是配置文件

package dns
// 导入所需的包
import (
    "context"
    "v2ray.com/core/common/net"
    "v2ray.com/core/features/dns/localdns"
)

// IPOption 是 IP 查询选项的对象
type IPOption struct {
    IPv4Enable bool
    IPv6Enable bool
}

// Client 是 DNS 客户端的接口
type Client interface {
    // 返回客户端的名称
    Name() string

    // QueryIP 向配置的服务器发送 IP 查询
    QueryIP(ctx context.Context, domain string, option IPOption) ([]net.IP, error)
}

type localNameServer struct {
    client *localdns.Client
}

func (s *localNameServer) QueryIP(ctx context.Context, domain string, option IPOption) ([]net.IP, error) {
    if option.IPv4Enable && option.IPv6Enable {
        return s.client.LookupIP(domain)
    }

    if option.IPv4Enable {
        return s.client.LookupIPv4(domain)
    }

    if option.IPv6Enable {
        return s.client.LookupIPv6(domain)
    }

    return nil, newError("neither IPv4 nor IPv6 is enabled")
}

func (s *localNameServer) Name() string {
    return "localhost"
}

func NewLocalNameServer() *localNameServer {
    newError("DNS: created localhost client").AtInfo().WriteToLog()
    return &localNameServer{
        client: localdns.New(),
    }
}
```