# `v2ray-core\infra\conf\vmess_test.go`

```
package conf_test

import (
    "testing"

    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol"
    "v2ray.com/core/common/serial"
    . "v2ray.com/core/infra/conf"  // 导入 v2ray 核心配置包
    "v2ray.com/core/proxy/vmess"   // 导入 VMess 代理包
    "v2ray.com/core/proxy/vmess/inbound"  // 导入 VMess 入站代理包
    "v2ray.com/core/proxy/vmess/outbound"  // 导入 VMess 出站代理包
)

func TestVMessOutbound(t *testing.T) {
    // 创建一个函数，返回一个可构建的对象
    creator := func() Buildable {
        return new(VMessOutboundConfig)  // 返回一个新的 VMess 出站配置对象
    }
    # 运行多个测试用例，传入测试对象和测试用例数组
    runMultiTestCase(t, []TestCase{
        {
            # 输入 JSON 格式的字符串
            Input: `{
                "vnext": [{
                    "address": "127.0.0.1",
                    "port": 80,
                    "users": [
                        {
                            "id": "e641f5ad-9397-41e3-bf1a-e8740dfed019",
                            "email": "love@v2ray.com",
                            "level": 255
                        }
                    ]
                }]
            }`,
            # 解析 JSON 字符串的解析器
            Parser: loadJSON(creator),
            # 期望输出的配置对象
            Output: &outbound.Config{
                Receiver: []*protocol.ServerEndpoint{
                    {
                        # 服务器地址
                        Address: &net.IPOrDomain{
                            Address: &net.IPOrDomain_Ip{
                                Ip: []byte{127, 0, 0, 1},
                            },
                        },
                        # 服务器端口
                        Port: 80,
                        # 用户信息
                        User: []*protocol.User{
                            {
                                # 用户邮箱
                                Email: "love@v2ray.com",
                                # 用户等级
                                Level: 255,
                                # 用户账户信息
                                Account: serial.ToTypedMessage(&vmess.Account{
                                    # 用户 ID
                                    Id:      "e641f5ad-9397-41e3-bf1a-e8740dfed019",
                                    # 用户替换 ID
                                    AlterId: 0,
                                    # 安全配置
                                    SecuritySettings: &protocol.SecurityConfig{
                                        # 安全类型
                                        Type: protocol.SecurityType_AUTO,
                                    },
                                }),
                            },
                        },
                    },
                },
            },
        },
    })
// 测试 VMessInbound 函数
func TestVMessInbound(t *testing.T) {
    // 创建一个 Buildable 对象的函数
    creator := func() Buildable {
        return new(VMessInboundConfig)
    }

    // 运行多个测试用例
    runMultiTestCase(t, []TestCase{
        {
            // 输入的 JSON 字符串
            Input: `{
                "clients": [
                    {
                        "id": "27848739-7e62-4138-9fd3-098a63964b6b",
                        "level": 0,
                        "alterId": 16,
                        "email": "love@v2ray.com",
                        "security": "aes-128-gcm"
                    }
                ],
                "default": {
                    "level": 0,
                    "alterId": 32
                },
                "detour": {
                    "to": "tag_to_detour"
                },
                "disableInsecureEncryption": true
            }`,
            // 解析 JSON 的函数
            Parser: loadJSON(creator),
            // 期望的输出结果
            Output: &inbound.Config{
                User: []*protocol.User{
                    {
                        Level: 0,
                        Email: "love@v2ray.com",
                        Account: serial.ToTypedMessage(&vmess.Account{
                            Id:      "27848739-7e62-4138-9fd3-098a63964b6b",
                            AlterId: 16,
                            SecuritySettings: &protocol.SecurityConfig{
                                Type: protocol.SecurityType_AES128_GCM,
                            },
                        }),
                    },
                },
                Default: &inbound.DefaultConfig{
                    Level:   0,
                    AlterId: 32,
                },
                Detour: &inbound.DetourConfig{
                    To: "tag_to_detour",
                },
                SecureEncryptionOnly: true,
            },
        },
    })
}
```