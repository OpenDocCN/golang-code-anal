# `kubo\cmd\ipfs\util\ulimit_unix.go`

```go
// 根据操作系统构建条件编译，仅在 darwin、linux、netbsd、openbsd 下编译
// 导入 util 包
package util

// 导入 golang.org/x/sys/unix 包，并重命名为 unix
import (
    unix "golang.org/x/sys/unix"
)

// 初始化函数
func init() {
    // 设置支持文件描述符管理为 true
    supportsFDManagement = true
    // 设置获取文件描述符限制的函数为 unixGetLimit
    getLimit = unixGetLimit
    // 设置设置文件描述符限制的函数为 unixSetLimit
    setLimit = unixSetLimit
}

// 获取文件描述符限制的函数
func unixGetLimit() (uint64, uint64, error) {
    // 创建 unix.Rlimit 变量 rlimit
    rlimit := unix.Rlimit{}
    // 调用 unix.Getrlimit 获取文件描述符限制，并将结果赋值给 rlimit
    err := unix.Getrlimit(unix.RLIMIT_NOFILE, &rlimit)
    // 返回当前文件描述符限制的 soft 和 max 值，以及可能的错误
    return rlimit.Cur, rlimit.Max, err
}

// 设置文件描述符限制的函数
func unixSetLimit(soft uint64, max uint64) error {
    // 创建 unix.Rlimit 变量 rlimit，并设置其 soft 和 max 值
    rlimit := unix.Rlimit{
        Cur: soft,
        Max: max,
    }
    // 调用 unix.Setrlimit 设置文件描述符限制，并传入 rlimit
    return unix.Setrlimit(unix.RLIMIT_NOFILE, &rlimit)
}
```