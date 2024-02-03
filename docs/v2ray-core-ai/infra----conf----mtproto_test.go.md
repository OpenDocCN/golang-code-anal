# `v2ray-core\infra\conf\mtproto_test.go`

```go
package conf_test

import (
    "testing"

    "v2ray.com/core/common/protocol"
    "v2ray.com/core/common/serial"
    . "v2ray.com/core/infra/conf"
    "v2ray.com/core/proxy/mtproto"
)

func TestMTProtoServerConfig(t *testing.T) {
    // 创建一个 Buildable 接口的实例，类型为 MTProtoServerConfig
    creator := func() Buildable {
        return new(MTProtoServerConfig)
    }

    // 运行多个测试用例
    runMultiTestCase(t, []TestCase{
        {
            // 输入的 JSON 字符串
            Input: `{
                "users": [{
                    "email": "love@v2ray.com",
                    "level": 1,
                    "secret": "b0cbcef5a486d9636472ac27f8e11a9d"
                }]
            }`,
            // 解析 JSON 字符串的函数
            Parser: loadJSON(creator),
            // 期望的输出
            Output: &mtproto.ServerConfig{
                User: []*protocol.User{
                    {
                        // 用户邮箱
                        Email: "love@v2ray.com",
                        // 用户等级
                        Level: 1,
                        // 用户账户信息
                        Account: serial.ToTypedMessage(&mtproto.Account{
                            // 用户密钥的字节数组
                            Secret: []byte{176, 203, 206, 245, 164, 134, 217, 99, 100, 114, 172, 39, 248, 225, 26, 157},
                        }),
                    },
                },
            },
        },
    })
}
```