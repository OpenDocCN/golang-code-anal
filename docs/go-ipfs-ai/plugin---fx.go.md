# `kubo\plugin\fx.go`

```
// 导入 plugin 包
package plugin

// 导入所需的包
import (
    "github.com/ipfs/kubo/core"  // 导入 core 包
    "go.uber.org/fx"  // 导入 fx 包
)

// PluginFx 可用于自定义在初始化 go-ipfs 应用程序时传递给 fx 选项。
//
// 这是一种侵入性的方法，依赖于内部细节，比如依赖图的结构，
// 因此在发布之间可能会发生破坏性的更改。
// 因此建议尽可能保持简单，并坚持使用 fx.Replace() 或 fx.Decorate() 覆盖接口。
//
// 返回的选项成为传递给 fx 的完整选项数组。
// 通常，您会希望将附加选项追加到 NodeInfo.FXOptions 并返回它。
type PluginFx interface {
    Plugin  // 实现 Plugin 接口
    Options(core.FXNodeInfo) ([]fx.Option, error)  // 返回 fx 选项的数组
}
```