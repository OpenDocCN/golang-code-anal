# `kubo\plugin\daemoninternal.go`

```go
# 导入名为 "core" 的包，该包位于 "github.com/ipfs/kubo" 下
import "github.com/ipfs/kubo/core"

# 定义一个名为 PluginDaemonInternal 的接口，用于守护程序插件
# 这些插件将在守护程序上运行，并将直接访问 IpfsNode
# 注意：PluginDaemonInternal 被认为是内部的，对其 API 的稳定性不做任何保证
# 如果可能的话，使用 PluginAPI 替代
type PluginDaemonInternal interface {
    # 继承 Plugin 接口
    Plugin
    # 定义一个名为 Start 的方法，接受一个名为 IpfsNode 的参数，并返回一个错误
    Start(*core.IpfsNode) error
}
```