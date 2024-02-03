# `kubo\cmd\ipfs\util\signal.go`

```go
//go:build !wasm
// +build !wasm

// 定义 util 包，用于提供一些工具函数和结构体
package util

import (
    "context"   // 导入 context 包，用于处理上下文
    "fmt"       // 导入 fmt 包，用于格式化输出
    "io"        // 导入 io 包，用于进行输入输出操作
    "os"        // 导入 os 包，提供对操作系统功能的访问
    "os/signal" // 导入 os/signal 包，用于处理信号
    "sync"      // 导入 sync 包，提供同步原语
    "syscall"   // 导入 syscall 包，提供对底层系统调用的访问
)

// IntrHandler 结构体，用于设置可以通过 io.Closer 接口清晰关闭的中断处理程序
type IntrHandler struct {
    closing chan struct{} // 用于关闭中断处理程序的通道
    wg      sync.WaitGroup // 用于等待所有 goroutine 完成的 WaitGroup
}

// NewIntrHandler 函数，用于创建一个新的 IntrHandler 结构体实例
func NewIntrHandler() *IntrHandler {
    return &IntrHandler{closing: make(chan struct{})} // 返回一个带有初始化通道的 IntrHandler 实例
}

// Close 方法，用于关闭中断处理程序
func (ih *IntrHandler) Close() error {
    close(ih.closing) // 关闭中断处理程序的通道
    ih.wg.Wait()      // 等待所有 goroutine 完成
    return nil        // 返回 nil 表示关闭成功
}

// Handle 方法，用于处理给定的信号，并在每次捕获信号时调用处理程序回调函数
func (ih *IntrHandler) Handle(handler func(count int, ih *IntrHandler), sigs ...os.Signal) {
    notify := make(chan os.Signal, 1) // 创建一个用于接收信号的通道
    signal.Notify(notify, sigs...)     // 通知接收指定的信号
    ih.wg.Add(1)                       // 增加 WaitGroup 的计数
    go func() {
        defer ih.wg.Done() // 在函数退出时减少 WaitGroup 的计数
        defer signal.Stop(notify) // 停止接收指定的信号

        count := 0
        for {
            select {
            case <-ih.closing: // 如果收到关闭信号，则退出循环
                return
            case <-notify: // 如果收到信号，则调用处理程序回调函数
                count++
                handler(count, ih)
            }
        }
    }()
}

// SetupInterruptHandler 函数，用于设置中断处理程序
func SetupInterruptHandler(ctx context.Context) (io.Closer, context.Context) {
    intrh := NewIntrHandler() // 创建一个新的中断处理程序实例
    ctx, cancelFunc := context.WithCancel(ctx) // 创建一个带有取消函数的上下文

    handlerFunc := func(count int, ih *IntrHandler) {
        switch count {
        case 1:
            fmt.Println() // 防止终端中出现未终止的 ^C 字符

            ih.wg.Add(1) // 增加 WaitGroup 的计数
            go func() {
                defer ih.wg.Done() // 在函数退出时减少 WaitGroup 的计数
                cancelFunc() // 调用取消函数
            }()

        default:
            fmt.Println("Received another interrupt before graceful shutdown, terminating...")
            os.Exit(-1) // 接收到另一个中断时，终止程序
        }
    }
    # 将 handlerFunc 函数注册为处理 SIGHUP、SIGINT 和 SIGTERM 信号的处理程序
    intrh.Handle(handlerFunc, syscall.SIGHUP, syscall.SIGINT, syscall.SIGTERM)
    
    # 返回注册的信号处理程序和上下文对象
    return intrh, ctx
# 闭合前面的函数定义
```