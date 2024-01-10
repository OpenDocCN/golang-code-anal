# `kubo\cmd\ipfs\util\ui_windows.go`

```
package util

import "golang.org/x/sys/windows"

// 判断当前程序是否在 GUI 界面中运行
func InsideGUI() bool {
    // 创建一个用于存储控制台屏幕缓冲区信息的结构体指针
    conhostInfo := &windows.ConsoleScreenBufferInfo{}
    // 获取标准输出的控制台屏幕缓冲区信息，并存储到 conhostInfo 中
    if err := windows.GetConsoleScreenBufferInfo(windows.Stdout, conhostInfo); err != nil {
        // 如果获取失败，则返回 false
        return false
    }

    // 如果控制台光标位置的 X 坐标和 Y 坐标都为 0
    if (conhostInfo.CursorPosition.X | conhostInfo.CursorPosition.Y) == 0 {
        // 控制台光标在执行之前没有移动
        // 高概率下我们不在终端中
        return true
    }

    // 否则返回 false
    return false
}
```