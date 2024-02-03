# `v2ray-core\infra\conf\reverse_test.go`

```go
package conf_test

import (
    "testing"

    "v2ray.com/core/app/reverse"
    "v2ray.com/core/infra/conf"
)

func TestReverseConfig(t *testing.T) {
    // 创建一个函数，返回一个可构建的配置对象
    creator := func() conf.Buildable {
        return new(conf.ReverseConfig)
    }

    // 运行多个测试用例
    runMultiTestCase(t, []TestCase{
        {
            // 输入的 JSON 字符串
            Input: `{
                "bridges": [{
                    "tag": "test",
                    "domain": "test.v2ray.com"
                }]
            }`,
            // 解析 JSON 字符串的函数
            Parser: loadJSON(creator),
            // 期望的输出
            Output: &reverse.Config{
                BridgeConfig: []*reverse.BridgeConfig{
                    {Tag: "test", Domain: "test.v2ray.com"},
                },
            },
        },
        {
            // 输入的 JSON 字符串
            Input: `{
                "portals": [{
                    "tag": "test",
                    "domain": "test.v2ray.com"
                }]
            }`,
            // 解析 JSON 字符串的函数
            Parser: loadJSON(creator),
            // 期望的输出
            Output: &reverse.Config{
                PortalConfig: []*reverse.PortalConfig{
                    {Tag: "test", Domain: "test.v2ray.com"},
                },
            },
        },
    })
}
```