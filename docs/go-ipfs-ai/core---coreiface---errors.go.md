# `kubo\core\coreiface\errors.go`

```go
# 定义错误变量包
package iface

# 导入 errors 包
import "errors"

# 定义错误变量
var (
    # 表示 DAG 节点是一个目录
    ErrIsDir = errors.New("this dag node is a directory")
    # 表示 DAG 节点不是一个普通文件
    ErrNotFile = errors.New("this dag node is not a regular file")
    # 表示必须在在线模式下运行该操作，尝试先运行 'ipfs daemon'
    ErrOffline = errors.New("this action must be run in online mode, try running 'ipfs daemon' first")
    # 表示不支持的操作
    ErrNotSupported = errors.New("operation not supported")
)
```