# `v2ray-core\infra\conf\http_test.go`

```
package conf_test

import (
    "testing"

    . "v2ray.com/core/infra/conf"  // 导入 v2ray 核心配置包
    "v2ray.com/core/proxy/http"    // 导入 v2ray 核心 HTTP 代理包
)

func TestHttpServerConfig(t *testing.T) {
    creator := func() Buildable {  // 创建一个构建函数
        return new(HttpServerConfig)  // 返回一个新的 HTTP 服务器配置对象
    }

    runMultiTestCase(t, []TestCase{  // 运行多个测试用例
        {
            Input: `{
                "timeout": 10,  // 设置超时时间为 10
                "accounts": [  // 设置账户信息
                    {
                        "user": "my-username",  // 设置用户名
                        "pass": "my-password"   // 设置密码
                    }
                ],
                "allowTransparent": true,  // 允许透明代理
                "userLevel": 1  // 设置用户级别为 1
            }`,
            Parser: loadJSON(creator),  // 使用 JSON 加载配置并解析
            Output: &http.ServerConfig{  // HTTP 服务器配置对象
                Accounts: map[string]string{  // 设置账户映射
                    "my-username": "my-password",  // 用户名和密码映射
                },
                AllowTransparent: true,  // 允许透明代理
                UserLevel:        1,  // 用户级别为 1
                Timeout:          10,  // 超时时间为 10
            },
        },
    })
}
```