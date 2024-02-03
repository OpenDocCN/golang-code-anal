# `trojan-go\main.go`

```go
// 主程序入口
func main() {
    // 解析命令行参数
    flag.Parse()
    // 无限循环，直到成功处理选项或发生错误
    for {
        // 从选项处理器中弹出一个选项处理器
        h, err := option.PopOptionHandler()
        // 如果发生错误，记录错误信息并退出程序
        if err != nil {
            log.Fatal("invalid options")
        }
        // 处理选项
        err = h.Handle()
        // 如果处理成功，跳出循环
        if err == nil {
            break
        }
    }
}
```