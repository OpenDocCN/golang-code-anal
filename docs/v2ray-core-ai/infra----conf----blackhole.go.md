# `v2ray-core\infra\conf\blackhole.go`

```go
package conf

import (
    "encoding/json"  // 导入处理 JSON 数据的包
    "github.com/golang/protobuf/proto"  // 导入处理 Protocol Buffers 数据的包
    "v2ray.com/core/common/serial"  // 导入处理序列化数据的包
    "v2ray.com/core/proxy/blackhole"  // 导入黑洞代理相关的包
)

type NoneResponse struct{}  // 定义 NoneResponse 结构体

func (*NoneResponse) Build() (proto.Message, error) {  // NoneResponse 结构体实现 Build 方法
    return new(blackhole.NoneResponse), nil  // 返回一个新的黑洞代理 NoneResponse 对象
}

type HttpResponse struct{}  // 定义 HttpResponse 结构体

func (*HttpResponse) Build() (proto.Message, error) {  // HttpResponse 结构体实现 Build 方法
    return new(blackhole.HTTPResponse), nil  // 返回一个新的黑洞代理 HTTPResponse 对象
}

type BlackholeConfig struct {  // 定义 BlackholeConfig 结构体
    Response json.RawMessage `json:"response"`  // 定义 Response 字段，用于存储 JSON 格式的原始消息
}

func (v *BlackholeConfig) Build() (proto.Message, error) {  // BlackholeConfig 结构体实现 Build 方法
    config := new(blackhole.Config)  // 创建一个新的黑洞代理配置对象
    if v.Response != nil {  // 如果 Response 字段不为空
        response, _, err := configLoader.Load(v.Response)  // 使用 configLoader 加载 Response 字段的数据
        if err != nil {  // 如果加载出错
            return nil, newError("Config: Failed to parse Blackhole response config.").Base(err)  // 返回错误信息
        }
        responseSettings, err := response.(Buildable).Build()  // 调用 Buildable 接口的 Build 方法
        if err != nil {  // 如果调用出错
            return nil, err  // 返回错误信息
        }
        config.Response = serial.ToTypedMessage(responseSettings)  // 将 responseSettings 转换为序列化的消息并赋值给 config.Response
    }

    return config, nil  // 返回配置对象和空错误
}

var (
    configLoader = NewJSONConfigLoader(  // 创建一个新的 JSON 配置加载器
        ConfigCreatorCache{  // 配置创建器缓存
            "none": func() interface{} { return new(NoneResponse) },  // "none" 类型对应的配置创建函数
            "http": func() interface{} { return new(HttpResponse) },  // "http" 类型对应的配置创建函数
        },
        "type",  // 配置类型字段名
        "")  // 配置类型字段的默认值
)
```