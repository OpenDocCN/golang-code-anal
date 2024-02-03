# `v2ray-core\common\platform\ctlcmd\attr_windows.go`

```go
// +build windows
// 声明该文件只在 Windows 平台下编译

package ctlcmd
// 声明包名为 ctlcmd

import "syscall"
// 导入 syscall 包，用于访问操作系统底层接口

func getSysProcAttr() *syscall.SysProcAttr {
    // 定义一个返回 *syscall.SysProcAttr 类型的函数 getSysProcAttr
    return &syscall.SysProcAttr{
        HideWindow: true,
        // 返回一个隐藏窗口的系统进程属性
    }
}
```