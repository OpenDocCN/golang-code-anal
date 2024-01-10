# `trojan-go\tunnel\tproxy\getsockopt.go`

```
// 根据条件编译标记，仅在 Linux 平台且不是 386 架构下构建该包
// +build linux,!386

package tproxy

import (
    "syscall"
    "unsafe"
)

// 获取套接字选项的值
func getsockopt(fd int, level int, optname int, optval unsafe.Pointer, optlen *uint32) (err error) {
    // 调用系统调用获取套接字选项的值
    _, _, e := syscall.Syscall6(
        syscall.SYS_GETSOCKOPT, uintptr(fd), uintptr(level), uintptr(optname),
        uintptr(optval), uintptr(unsafe.Pointer(optlen)), 0)
    // 如果出现错误，返回错误信息
    if e != 0 {
        return e
    }
    return
}
```