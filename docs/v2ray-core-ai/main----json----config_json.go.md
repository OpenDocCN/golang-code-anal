# `v2ray-core\main\json\config_json.go`

```
package json

// 生成错误代码
//go:generate go run v2ray.com/core/common/errors/errorgen

import (
    "io"

    "v2ray.com/core"
    "v2ray.com/core/common"
    "v2ray.com/core/common/cmdarg"
    "v2ray.com/core/infra/conf/serial"
    "v2ray.com/core/main/confloader"
)

// 初始化函数
func init() {
    // 注册配置加载器
    common.Must(core.RegisterConfigLoader(&core.ConfigFormat{
        Name:      "JSON",
        Extension: []string{"json"},
        Loader: func(input interface{}) (*core.Config, error) {
            // 根据输入类型执行不同的加载操作
            switch v := input.(type) {
            // 如果是命令行参数类型
            case cmdarg.Arg:
                // 加载扩展配置
                r, err := confloader.LoadExtConfig(v)
                if err != nil {
                    return nil, newError("failed to execute v2ctl to convert config file.").Base(err).AtWarning()
                }
                return core.LoadConfig("protobuf", "", r)
            // 如果是输入流类型
            case io.Reader:
                // 加载 JSON 格式的配置
                return serial.LoadJSONConfig(v)
            // 其他类型
            default:
                return nil, newError("unknow type")
            }
        },
    }))
}
```