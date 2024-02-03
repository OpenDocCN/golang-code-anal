# `v2ray-core\infra\conf\socks_test.go`

```go
package conf_test

import (
    "testing"

    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol"
    "v2ray.com/core/common/serial"
    . "v2ray.com/core/infra/conf"
    "v2ray.com/core/proxy/socks"
)

func TestSocksInboundConfig(t *testing.T) {
    // 创建一个构建函数，返回 SocksServerConfig 对象
    creator := func() Buildable {
        return new(SocksServerConfig)
    }

    // 运行多个测试用例
    runMultiTestCase(t, []TestCase{
        {
            // 输入的 JSON 字符串
            Input: `{
                "auth": "password",
                "accounts": [
                    {
                        "user": "my-username",
                        "pass": "my-password"
                    }
                ],
                "udp": false,
                "ip": "127.0.0.1",
                "timeout": 5,
                "userLevel": 1
            }`,
            // 解析 JSON 字符串并创建对象
            Parser: loadJSON(creator),
            // 期望的输出对象
            Output: &socks.ServerConfig{
                AuthType: socks.AuthType_PASSWORD,
                Accounts: map[string]string{
                    "my-username": "my-password",
                },
                UdpEnabled: false,
                Address: &net.IPOrDomain{
                    Address: &net.IPOrDomain_Ip{
                        Ip: []byte{127, 0, 0, 1},
                    },
                },
                Timeout:   5,
                UserLevel: 1,
            },
        },
    })
}

func TestSocksOutboundConfig(t *testing.T) {
    // 创建一个构建函数，返回 SocksClientConfig 对象
    creator := func() Buildable {
        return new(SocksClientConfig)
    }
    # 运行多个测试用例，传入测试对象和测试用例数组
    runMultiTestCase(t, []TestCase{
        {
            # 输入 JSON 字符串
            Input: `{
                "servers": [{
                    "address": "127.0.0.1",
                    "port": 1234,
                    "users": [
                        {"user": "test user", "pass": "test pass", "email": "test@email.com"}
                    ]
                }]
            }`,
            # 解析 JSON 字符串，创建对应的对象
            Parser: loadJSON(creator),
            # 期望输出的客户端配置对象
            Output: &socks.ClientConfig{
                # 服务器端点数组
                Server: []*protocol.ServerEndpoint{
                    {
                        # 服务器地址
                        Address: &net.IPOrDomain{
                            Address: &net.IPOrDomain_Ip{
                                # IP 地址
                                Ip: []byte{127, 0, 0, 1},
                            },
                        },
                        # 服务器端口
                        Port: 1234,
                        # 用户数组
                        User: []*protocol.User{
                            {
                                # 用户邮箱
                                Email: "test@email.com",
                                # 用户账户对象
                                Account: serial.ToTypedMessage(&socks.Account{
                                    # 用户名
                                    Username: "test user",
                                    # 密码
                                    Password: "test pass",
                                }),
                            },
                        },
                    },
                },
            },
        },
    })
# 闭合前面的函数定义
```