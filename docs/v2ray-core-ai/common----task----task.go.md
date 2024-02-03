# `v2ray-core\common\task\task.go`

```go
// 定义一个函数 OnSuccess，该函数接受两个参数 f 和 g，分别为执行函数，当 f 返回 nil 时执行 g
func OnSuccess(f func() error, g func() error) func() error {
    // 返回一个函数，该函数执行 f，如果 f 返回错误则直接返回错误，否则执行 g 并返回其结果
    return func() error {
        if err := f(); err != nil {
            return err
        }
        return g()
    }
}

// 定义一个函数 Run，该函数接受一个上下文和一系列任务函数，以并行方式执行这些任务，返回遇到的第一个错误，如果所有任务都通过则返回 nil
func Run(ctx context.Context, tasks ...func() error) error {
    // 获取任务数量
    n := len(tasks)
    // 创建一个信号量，初始值为任务数量
    s := semaphore.New(n)
    // 创建一个通道，用于存放已完成的任务的错误信息
    done := make(chan error, 1)

    // 遍历任务列表
    for _, task := range tasks {
        // 从信号量中获取一个信号
        <-s.Wait()
        // 启动一个 goroutine 执行任务函数
        go func(f func() error) {
            // 执行任务函数并获取返回的错误
            err := f()
            // 如果任务函数执行成功，释放一个信号并返回
            if err == nil {
                s.Signal()
                return
            }
            // 如果任务函数执行失败，尝试将错误信息发送到通道中
            select {
            case done <- err:
            default:
            }
        }(task)
    }

    // 循环检查任务执行结果
    for i := 0; i < n; i++ {
        select {
        // 如果有任务执行失败，则返回错误信息
        case err := <-done:
            return err
        // 如果上下文被取消，则返回上下文的错误信息
        case <-ctx.Done():
            return ctx.Err()
        // 如果有任务执行完成，则继续循环
        case <-s.Wait():
        }
    }

    // 所有任务执行成功，返回 nil
    return nil
}
```