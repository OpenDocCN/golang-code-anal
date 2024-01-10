# `v2ray-core\infra\conf\loader.go`

```
package conf

import (
    "encoding/json"  // 导入 JSON 包
    "strings"        // 导入字符串处理包
)

type ConfigCreator func() interface{}  // 定义一个函数类型 ConfigCreator，用于创建配置对象

type ConfigCreatorCache map[string]ConfigCreator  // 定义一个映射类型 ConfigCreatorCache，用于存储配置创建函数

func (v ConfigCreatorCache) RegisterCreator(id string, creator ConfigCreator) error {
    if _, found := v[id]; found {  // 如果已经存在相同 id 的配置创建函数，则返回错误
        return newError(id, " already registered.").AtError()
    }

    v[id] = creator  // 将配置创建函数注册到缓存中
    return nil
}

func (v ConfigCreatorCache) CreateConfig(id string) (interface{}, error) {
    creator, found := v[id]  // 从缓存中获取指定 id 的配置创建函数
    if !found {  // 如果未找到对应的配置创建函数，则返回错误
        return nil, newError("unknown config id: ", id)
    }
    return creator(), nil  // 调用配置创建函数创建配置对象
}

type JSONConfigLoader struct {
    cache     ConfigCreatorCache  // 配置创建函数缓存
    idKey     string  // ID 键名
    configKey string  // 配置键名
}

func NewJSONConfigLoader(cache ConfigCreatorCache, idKey string, configKey string) *JSONConfigLoader {
    return &JSONConfigLoader{  // 创建并返回一个 JSONConfigLoader 对象
        idKey:     idKey,  // 设置 ID 键名
        configKey: configKey,  // 设置配置键名
        cache:     cache,  // 设置配置创建函数缓存
    }
}

func (v *JSONConfigLoader) LoadWithID(raw []byte, id string) (interface{}, error) {
    id = strings.ToLower(id)  // 将 ID 转换为小写
    config, err := v.cache.CreateConfig(id)  // 根据 ID 从缓存中创建配置对象
    if err != nil {  // 如果创建配置对象时发生错误，则返回错误
        return nil, err
    }
    if err := json.Unmarshal(raw, config); err != nil {  // 将原始数据解析为配置对象
        return nil, err
    }
    return config, nil  // 返回配置对象
}

func (v *JSONConfigLoader) Load(raw []byte) (interface{}, string, error) {
    var obj map[string]json.RawMessage  // 定义一个 map 用于存储 JSON 数据
    if err := json.Unmarshal(raw, &obj); err != nil {  // 解析原始数据为 JSON 对象
        return nil, "", err
    }
    rawID, found := obj[v.idKey]  // 获取 ID 键对应的原始数据
    if !found {  // 如果未找到 ID 键，则返回错误
        return nil, "", newError(v.idKey, " not found in JSON context").AtError()
    }
    var id string
    if err := json.Unmarshal(rawID, &id); err != nil {  // 解析 ID 原始数据为字符串
        return nil, "", err
    }
    rawConfig := json.RawMessage(raw)  // 将原始数据转换为 JSON 原始消息
    if len(v.configKey) > 0 {  // 如果配置键名不为空
        configValue, found := obj[v.configKey]  // 获取配置键对应的值
        if found {  // 如果找到配置键对应的值
            rawConfig = configValue  // 使用配置值作为配置原始消息
        } else {
            // 默认为空的 JSON 对象
            rawConfig = json.RawMessage([]byte("{}"))  // 默认使用空的 JSON 对象作为配置原始消息
        }
    }
    // 使用给定的 ID 加载配置文件，返回配置对象和错误信息
    config, err := v.LoadWithID([]byte(rawConfig), id)
    // 如果加载过程中出现错误，返回空对象、ID 和错误信息
    if err != nil {
        return nil, id, err
    }
    // 如果加载成功，返回配置对象、ID 和空错误信息
    return config, id, nil
# 闭合前面的函数定义
```