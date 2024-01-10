# `kubo\plugin\daemon.go`

```
// 导入 plugin 包
package plugin

// 导入 coreiface 包，该包提供了与核心接口相关的功能
import (
    coreiface "github.com/ipfs/kubo/core/coreiface"
)

// PluginDaemon 是一个守护程序插件的接口。这些插件将在守护程序上运行，并将被赋予对 CoreAPI 实现的访问权限。
type PluginDaemon interface {
    // PluginDaemon 接口继承自 Plugin 接口
    Plugin
    // Start 方法接收一个 CoreAPI 实现，并返回一个错误
    Start(coreiface.CoreAPI) error
}
```