# `v2ray-core\infra\conf\blackhole_test.go`

```go
package conf_test

import (
    "testing"  // 导入测试包
    "v2ray.com/core/common/serial"  // 导入序列化包
    . "v2ray.com/core/infra/conf"  // 导入配置包，使用 . 表示在后续代码中可以省略包名
    "v2ray.com/core/proxy/blackhole"  // 导入黑洞代理包
)

func TestHTTPResponseJSON(t *testing.T) {
    // 创建一个函数，返回一个可构建的对象
    creator := func() Buildable {
        return new(BlackholeConfig)  // 返回一个黑洞配置对象
    }

    // 运行多个测试用例
    runMultiTestCase(t, []TestCase{
        {
            // 输入的 JSON 字符串
            Input: `{
                "response": {
                    "type": "http"
                }
            }`,
            // 解析 JSON 字符串的函数
            Parser: loadJSON(creator),
            // 期望输出的黑洞配置对象
            Output: &blackhole.Config{
                Response: serial.ToTypedMessage(&blackhole.HTTPResponse{}),  // 设置响应字段为 HTTP 响应对象
            },
        },
        {
            // 空的输入 JSON 字符串
            Input:  `{}`,
            // 解析 JSON 字符串的函数
            Parser: loadJSON(creator),
            // 期望输出的空的黑洞配置对象
            Output: &blackhole.Config{},
        },
    })
}
```