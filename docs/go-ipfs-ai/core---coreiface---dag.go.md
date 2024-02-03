# `kubo\core\coreiface\dag.go`

```go
// 声明 iface 包
package iface

// 导入 ipld 包，并重命名为ipld
import (
    ipld "github.com/ipfs/go-ipld-format"
)

// 定义 APIDagService 接口，继承自 ipld.DAGService 接口
type APIDagService interface {
    ipld.DAGService  // 继承自 ipld.DAGService 接口

    // 定义 Pinning 方法，返回一个特殊的 NodeAdder，用于递归固定添加的节点
    Pinning() ipld.NodeAdder
}
```