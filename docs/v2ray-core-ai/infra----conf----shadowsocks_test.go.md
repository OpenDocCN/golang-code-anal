# `v2ray-core\infra\conf\shadowsocks_test.go`

```
package conf_test

import (
    "testing"

    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol"
    "v2ray.com/core/common/serial"
    . "v2ray.com/core/infra/conf"
    "v2ray.com/core/proxy/shadowsocks"
)

func TestShadowsocksServerConfigParsing(t *testing.T) {
    // 创建一个 Buildable 接口的实例，类型为 ShadowsocksServerConfig
    creator := func() Buildable {
        return new(ShadowsocksServerConfig)
    }

    // 运行多个测试用例
    runMultiTestCase(t, []TestCase{
        {
            // 输入的 JSON 字符串
            Input: `{
                "method": "aes-128-cfb",
                "password": "v2ray-password"
            }`,
            // 解析 JSON 字符串的函数
            Parser: loadJSON(creator),
            // 预期的输出
            Output: &shadowsocks.ServerConfig{
                User: &protocol.User{
                    // 用户账户的信息
                    Account: serial.ToTypedMessage(&shadowsocks.Account{
                        // 加密方式为 AES-128-CFB
                        CipherType: shadowsocks.CipherType_AES_128_CFB,
                        // 密码为 v2ray-password
                        Password:   "v2ray-password",
                    }),
                },
                // 网络类型为 TCP
                Network: []net.Network{net.Network_TCP},
            },
        },
    })
}
```