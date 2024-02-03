# `kubo\plugin\datastore.go`

```go
// 导入名为 "plugin" 的包
package plugin

// 导入名为 "fsrepo" 的包，该包位于 "github.com/ipfs/kubo/repo/fsrepo" 路径下
import (
    "github.com/ipfs/kubo/repo/fsrepo"
)

// PluginDatastore 是一个接口，可以实现不同数据存储的处理程序
type PluginDatastore interface {
    // 继承自 Plugin 接口
    Plugin

    // 返回数据存储类型的名称
    DatastoreTypeName() string
    // 返回数据存储配置解析器
    DatastoreConfigParser() fsrepo.ConfigFromMap
}
```