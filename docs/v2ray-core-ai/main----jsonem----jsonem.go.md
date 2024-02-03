# `v2ray-core\main\jsonem\jsonem.go`

```go
package jsonem

import (
    "io"

    "v2ray.com/core"  // 导入 v2ray 核心包
    "v2ray.com/core/common"  // 导入 v2ray 核心通用包
    "v2ray.com/core/common/cmdarg"  // 导入 v2ray 核心命令参数包
    "v2ray.com/core/infra/conf"  // 导入 v2ray 核心配置包
    "v2ray.com/core/infra/conf/serial"  // 导入 v2ray 核心配置序列化包
    "v2ray.com/core/main/confloader"  // 导入 v2ray 核心配置加载包
)

func init() {
    // 注册 JSON 格式的配置加载器
    common.Must(core.RegisterConfigLoader(&core.ConfigFormat{
        Name:      "JSON",  // 配置格式名称为 JSON
        Extension: []string{"json"},  // 文件扩展名为 json
        Loader: func(input interface{}) (*core.Config, error) {
            switch v := input.(type) {
            case cmdarg.Arg:  // 如果输入是命令参数
                cf := &conf.Config{}  // 创建配置对象
                for _, arg := range v {
                    newError("Reading config: ", arg).AtInfo().WriteToLog()  // 记录读取配置的日志
                    r, err := confloader.LoadConfig(arg)  // 加载配置
                    common.Must(err)  // 如果出错，抛出异常
                    c, err := serial.DecodeJSONConfig(r)  // 解析 JSON 格式的配置
                    common.Must(err)  // 如果出错，抛出异常
                    cf.Override(c, arg)  // 覆盖配置
                }
                return cf.Build()  // 构建配置
            case io.Reader:  // 如果输入是读取器
                return serial.LoadJSONConfig(v)  // 加载 JSON 格式的配置
            default:
                return nil, newError("unknow type")  // 如果输入类型未知，返回错误
            }
        },
    }))
}
```