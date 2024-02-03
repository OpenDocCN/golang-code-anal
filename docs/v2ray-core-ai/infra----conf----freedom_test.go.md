# `v2ray-core\infra\conf\freedom_test.go`

```go
package conf_test

import (
    "testing"

    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol"
    . "v2ray.com/core/infra/conf"
    "v2ray.com/core/proxy/freedom"
)

func TestFreedomConfig(t *testing.T) {
    // 创建一个函数，返回一个可构建的对象
    creator := func() Buildable {
        return new(FreedomConfig)
    }

    // 运行多个测试用例
    runMultiTestCase(t, []TestCase{
        {
            // 输入的 JSON 字符串
            Input: `{
                "domainStrategy": "AsIs",
                "timeout": 10,
                "redirect": "127.0.0.1:3366",
                "userLevel": 1
            }`,
            // 解析 JSON 字符串的函数
            Parser: loadJSON(creator),
            // 期望输出的 freedom.Config 对象
            Output: &freedom.Config{
                DomainStrategy: freedom.Config_AS_IS,
                Timeout:        10,
                DestinationOverride: &freedom.DestinationOverride{
                    Server: &protocol.ServerEndpoint{
                        Address: &net.IPOrDomain{
                            Address: &net.IPOrDomain_Ip{
                                Ip: []byte{127, 0, 0, 1},
                            },
                        },
                        Port: 3366,
                    },
                },
                UserLevel: 1,
            },
        },
    })
}
```