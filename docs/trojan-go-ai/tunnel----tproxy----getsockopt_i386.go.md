# `trojan-go\tunnel\tproxy\getsockopt_i386.go`

```
// 根据条件编译标记，当操作系统为 Linux 且架构为 386 时，才会编译该文件
// +build linux,386

// 定义包名为 tproxy
package tproxy

// 导入系统调用包
import (
    "syscall"
    "unsafe"
)

// 定义常量 GETSOCKOPT 为 15
const GETSOCKOPT = 15

// 定义函数 getsockopt，用于获取套接字选项
func getsockopt(fd int, level int, optname int, optval unsafe.Pointer, optlen *uint32) (err error) {
    // 调用系统调用获取套接字选项
    _, _, e := syscall.Syscall6(
        GETSOCKOPT, uintptr(fd), uintptr(level), uintptr(optname),
        uintptr(optval), uintptr(unsafe.Pointer(optlen)), 0)
    // 如果返回值不为 0，则返回错误
    if e != 0 {
        return e
    }
    // 返回空错误表示成功
    return
}
```