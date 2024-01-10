# `kubo\config\plugins.go`

```
# 定义一个名为 Plugins 的结构体类型
type Plugins struct {
    # 插件名称到插件对象的映射
    Plugins map[string]Plugin
    # TODO: Loader Path? Leaving that out for now due to security concerns.
}

# 定义一个名为 Plugin 的结构体类型
type Plugin struct {
    # 插件是否被禁用
    Disabled bool
    # 插件的配置信息，类型为接口
    Config   interface{}
}
```