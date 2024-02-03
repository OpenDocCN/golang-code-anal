# `kubo\core\node\libp2p\fd\sys_unix.go`

```go
// 根据条件编译指令，当操作系统为 Linux 或 Darwin 时，进行编译
// +build linux darwin

// 定义包名为 fd
package fd

// 导入 golang.org/x/sys/unix 包
import (
    "golang.org/x/sys/unix"
)

// 定义函数 GetNumFDs，返回当前进程打开的文件描述符数量
func GetNumFDs() int {
    // 定义变量 l 为 unix.Rlimit 类型
    var l unix.Rlimit
    // 获取当前进程的文件描述符限制，存储在 l 中
    if err := unix.Getrlimit(unix.RLIMIT_NOFILE, &l); err != nil {
        // 如果获取失败，返回 0
        return 0
    }
    // 返回当前进程打开的文件描述符数量
    return int(l.Cur)
}
```