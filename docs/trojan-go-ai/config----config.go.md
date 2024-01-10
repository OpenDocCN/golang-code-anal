# `trojan-go\config\config.go`

```
// 导入必要的包
package config

import (
    "context"
    "encoding/json"
    "gopkg.in/yaml.v3"
)

// 创建一个全局的 map，用于存储配置创建函数
var creators = make(map[string]Creator)

// 定义一个函数类型 Creator，用于创建模块的默认配置结构
type Creator func() interface{}

// 注册配置创建函数，将配置结构注册到全局 map 中
func RegisterConfigCreator(name string, creator Creator) {
    // 修改配置名称，添加后缀"_CONFIG"
    name += "_CONFIG"
    // 将配置名称和创建函数存储到全局 map 中
    creators[name] = creator
}

// 解析 JSON 数据，返回配置信息和可能的错误
func parseJSON(data []byte) (map[string]interface{}, error) {
    // 创建一个空的结果 map
    result := make(map[string]interface{})
    // 遍历全局 map 中的配置名称和创建函数
    for name, creator := range creators {
        // 调用创建函数，获取配置结构
        config := creator()
        // 解析 JSON 数据到配置结构中
        if err := json.Unmarshal(data, config); err != nil {
            return nil, err
        }
        // 将配置结构存储到结果 map 中
        result[name] = config
    }
    return result, nil
}

// 解析 YAML 数据，返回配置信息和可能的错误
func parseYAML(data []byte) (map[string]interface{}, error) {
    // 创建一个空的结果 map
    result := make(map[string]interface{})
    // 遍历全局 map 中的配置名称和创建函数
    for name, creator := range creators {
        // 调用创建函数，获取配置结构
        config := creator()
        // 解析 YAML 数据到配置结构中
        if err := yaml.Unmarshal(data, config); err != nil {
            return nil, err
        }
        // 将配置结构存储到结果 map 中
        result[name] = config
    }
    return result, nil
}

// 使用 JSON 数据更新上下文，返回更新后的上下文和可能的错误
func WithJSONConfig(ctx context.Context, data []byte) (context.Context, error) {
    var configs map[string]interface{}
    var err error
    // 解析 JSON 数据
    configs, err = parseJSON(data)
    if err != nil {
        return ctx, err
    }
    // 将配置信息存储到上下文中
    for name, config := range configs {
        ctx = context.WithValue(ctx, name, config)
    }
    return ctx, nil
}

// 使用 YAML 数据更新上下文，返回更新后的上下文和可能的错误
func WithYAMLConfig(ctx context.Context, data []byte) (context.Context, error) {
    var configs map[string]interface{}
    var err error
    // 解析 YAML 数据
    configs, err = parseYAML(data)
    if err != nil {
        return ctx, err
    }
    // 将配置信息存储到上下文中
    for name, config := range configs {
        ctx = context.WithValue(ctx, name, config)
    }
    return ctx, nil
}

// 使用配置信息更新上下文，返回更新后的上下文
func WithConfig(ctx context.Context, name string, cfg interface{}) context.Context {
    // 修改配置名称，添加后缀"_CONFIG"
    name += "_CONFIG"
    // 将配置信息存储到上下文中
    return context.WithValue(ctx, name, cfg)
}

// 从上下文中提取配置信息
# 从上下文中获取指定名称的值
func FromContext(ctx context.Context, name string) interface{}:
    # 返回上下文中指定名称的值
    return ctx.Value(name + "_CONFIG")
```