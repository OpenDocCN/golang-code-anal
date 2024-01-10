# `v2ray-core\common\type.go`

```
// common 包中包含了一些通用的函数和类型
package common

import (
    "context"   // 导入 context 包，用于处理上下文
    "reflect"   // 导入 reflect 包，用于获取类型信息
)

// ConfigCreator 是一个根据配置创建对象的函数类型
type ConfigCreator func(ctx context.Context, config interface{}) (interface{}, error)

var (
    typeCreatorRegistry = make(map[reflect.Type]ConfigCreator)   // 创建一个类型到 ConfigCreator 函数的映射
)

// RegisterConfig 注册一个全局的配置创建函数。配置可以是 nil，但必须有一个类型。
func RegisterConfig(config interface{}, configCreator ConfigCreator) error {
    configType := reflect.TypeOf(config)   // 获取配置的类型信息
    if _, found := typeCreatorRegistry[configType]; found {   // 检查类型是否已经注册
        return newError(configType.Name() + " is already registered").AtError()   // 返回已注册的错误信息
    }
    typeCreatorRegistry[configType] = configCreator   // 将配置类型和创建函数注册到映射中
    return nil   // 返回空错误
}

// CreateObject 根据配置创建一个对象。配置类型必须通过 RegisterConfig() 注册。
func CreateObject(ctx context.Context, config interface{}) (interface{}, error) {
    configType := reflect.TypeOf(config)   // 获取配置的类型信息
    creator, found := typeCreatorRegistry[configType]   // 从映射中查找配置类型对应的创建函数
    if !found {   // 如果未找到对应的创建函数
        return nil, newError(configType.String() + " is not registered").AtError()   // 返回未注册的错误信息
    }
    return creator(ctx, config)   // 调用创建函数创建对象
}
```