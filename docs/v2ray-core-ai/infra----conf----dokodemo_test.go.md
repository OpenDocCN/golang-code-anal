# `v2ray-core\infra\conf\dokodemo_test.go`

```go
package conf_test

import (
    "testing"

    "v2ray.com/core/common/net"
    . "v2ray.com/core/infra/conf"
    "v2ray.com/core/proxy/dokodemo"
)

func TestDokodemoConfig(t *testing.T) {
    // 创建一个函数，返回一个可构建的对象
    creator := func() Buildable {
        return new(DokodemoConfig)
    }

    // 运行多个测试用例
    runMultiTestCase(t, []TestCase{
        {
            // 输入的 JSON 字符串
            Input: `{
                "address": "8.8.8.8",
                "port": 53,
                "network": "tcp",
                "timeout": 10,
                "followRedirect": true,
                "userLevel": 1
            }`,
            // 解析 JSON 字符串并构建对象
            Parser: loadJSON(creator),
            // 期望的输出对象
            Output: &dokodemo.Config{
                Address: &net.IPOrDomain{
                    Address: &net.IPOrDomain_Ip{
                        Ip: []byte{8, 8, 8, 8},
                    },
                },
                Port:           53,
                Networks:       []net.Network{net.Network_TCP},
                Timeout:        10,
                FollowRedirect: true,
                UserLevel:      1,
            },
        },
    })
}
```