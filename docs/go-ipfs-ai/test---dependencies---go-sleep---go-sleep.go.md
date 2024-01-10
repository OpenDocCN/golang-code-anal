# `kubo\test\dependencies\go-sleep\go-sleep.go`

```
package main

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "os"   // 导入 os 包，用于访问操作系统功能
    "time" // 导入 time 包，用于处理时间
)

func main() {
    if len(os.Args) != 2 {  // 检查命令行参数数量是否为 2
        usageError()  // 如果不是，则调用 usageError 函数
    }
    d, err := time.ParseDuration(os.Args[1])  // 将命令行参数解析为时间间隔
    if err != nil {  // 如果解析出错
        fmt.Fprintf(os.Stderr, "Could not parse duration: %s\n", err)  // 格式化输出错误信息
        usageError()  // 调用 usageError 函数
    }

    time.Sleep(d)  // 休眠指定的时间间隔
}

func usageError() {
    fmt.Fprintf(os.Stderr, "Usage: %s <duration>\n", os.Args[0])  // 格式化输出使用说明
    fmt.Fprintln(os.Stderr, `Valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h".`)  // 输出有效的时间单位
    fmt.Fprintln(os.Stderr, "See https://godoc.org/time#ParseDuration for more.")  // 输出更多信息的链接
    os.Exit(-1)  // 退出程序并返回错误码 -1
}
```