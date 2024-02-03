# `kubo\plugin\plugin.go`

```go
// 定义一个名为 Environment 的结构体，用于传递给插件的环境信息
type Environment struct {
    // IPFS 仓库的路径
    Repo string

    // 插件的配置，如果在用户的 go-ipfs 配置文件的 Plugins.Plugins["plugin-name"].Config 字段中指定
    // 详细信息请参考 docs/plugins.md
    //
    // 这是一个类似 JSON 的任意对象，根据 https://golang.org/pkg/encoding/json/#Unmarshal 进行解组
    Config interface{}
}

// Plugin 是所有 go-ipfs 插件的基本接口
// 它将包含在不同插件的接口中
//
// 可选地，如果插件希望在卸载时进行终止步骤，插件可以实现 io.Closer
type Plugin interface {
    // Name 应该返回插件的唯一名称
    Name() string

    // Version 返回插件的当前版本
    Version() string

    // Init 在加载插件时调用一次
    // 插件将接收一个包含 IPFS 仓库路径和插件配置的环境
    Init(env *Environment) error
}
```