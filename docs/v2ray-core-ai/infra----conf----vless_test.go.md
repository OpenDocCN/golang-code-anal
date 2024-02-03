# `v2ray-core\infra\conf\vless_test.go`

```go
package conf_test

import (
    "testing"

    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol"
    "v2ray.com/core/common/serial"
    . "v2ray.com/core/infra/conf"
    "v2ray.com/core/proxy/vless"
    "v2ray.com/core/proxy/vless/inbound"
    "v2ray.com/core/proxy/vless/outbound"
)

func TestVLessOutbound(t *testing.T) {
    // 创建一个函数，返回一个可构建的对象
    creator := func() Buildable {
        return new(VLessOutboundConfig)
    }

    // 运行多个测试用例
    runMultiTestCase(t, []TestCase{
        {
            // 输入的 JSON 字符串
            Input: `{
                "vnext": [{
                    "address": "example.com",
                    "port": 443,
                    "users": [
                        {
                            "id": "27848739-7e62-4138-9fd3-098a63964b6b",
                            "flow": "xtls-rprx-origin-udp443",
                            "encryption": "none",
                            "level": 0
                        }
                    ]
                }]
            }`,
            // 解析 JSON 字符串的函数
            Parser: loadJSON(creator),
            // 期望输出的配置对象
            Output: &outbound.Config{
                Vnext: []*protocol.ServerEndpoint{
                    {
                        Address: &net.IPOrDomain{
                            Address: &net.IPOrDomain_Domain{
                                Domain: "example.com",
                            },
                        },
                        Port: 443,
                        User: []*protocol.User{
                            {
                                Account: serial.ToTypedMessage(&vless.Account{
                                    Id:         "27848739-7e62-4138-9fd3-098a63964b6b",
                                    Flow:       "xtls-rprx-origin-udp443",
                                    Encryption: "none",
                                }),
                                Level: 0,
                            },
                        },
                    },
                },
            },
        },
    })
}

func TestVLessInbound(t *testing.T) {
    # 创建一个名为creator的变量，其值为一个函数，该函数返回一个VLessInboundConfig类型的实例
    creator := func() Buildable {
        return new(VLessInboundConfig)
    }
    # 结束函数定义
    })
# 闭合前面的函数定义
```