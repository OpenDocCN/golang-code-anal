# `v2ray-core\infra\conf\dns_proxy_test.go`

```go
package conf_test

import (
    "testing"

    "v2ray.com/core/common/net"
    . "v2ray.com/core/infra/conf"  // 导入 v2ray 核心配置包
    "v2ray.com/core/proxy/dns"     // 导入 v2ray DNS 代理包
)

func TestDnsProxyConfig(t *testing.T) {
    creator := func() Buildable {  // 创建一个返回可构建对象的函数
        return new(DnsOutboundConfig)  // 返回一个新的 DNS 出站配置对象
    }

    runMultiTestCase(t, []TestCase{  // 运行多个测试用例
        {
            Input: `{
                "address": "8.8.8.8",
                "port": 53,
                "network": "tcp"
            }`,  // 输入的 JSON 字符串
            Parser: loadJSON(creator),  // 使用 loadJSON 函数解析 JSON 字符串
            Output: &dns.Config{  // 期望的输出结果
                Server: &net.Endpoint{  // DNS 服务器的网络端点
                    Network: net.Network_TCP,  // 网络类型为 TCP
                    Address: net.NewIPOrDomain(net.IPAddress([]byte{8, 8, 8, 8})),  // 设置 IP 地址
                    Port:    53,  // 设置端口号
                },
            },
        },
    })
}
```