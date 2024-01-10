# `v2ray-core\common\log\logger.go`

```
package log

import (
    "io" // 导入io包，用于实现输入输出接口
    "log" // 导入log包，用于日志记录
    "os" // 导入os包，用于操作系统功能
    "time" // 导入time包，用于时间相关功能

    "v2ray.com/core/common/platform" // 导入平台相关功能
    "v2ray.com/core/common/signal/done" // 导入信号处理相关功能
    "v2ray.com/core/common/signal/semaphore" // 导入信号量相关功能
)

// Writer is the interface for writing logs.
type Writer interface {
    Write(string) error // 定义写日志的接口
    io.Closer // 实现io.Closer接口
}

// WriterCreator is a function to create LogWriters.
type WriterCreator func() Writer // 定义创建日志写入器的函数类型

type generalLogger struct {
    creator WriterCreator // 日志写入器的创建函数
    buffer  chan Message // 消息缓冲通道
    access  *semaphore.Instance // 信号量实例
    done    *done.Instance // 完成实例
}

// NewLogger returns a generic log handler that can handle all type of messages.
func NewLogger(logWriterCreator WriterCreator) Handler {
    return &generalLogger{
        creator: logWriterCreator, // 设置日志写入器的创建函数
        buffer:  make(chan Message, 16), // 创建消息缓冲通道
        access:  semaphore.New(1), // 创建信号量实例
        done:    done.New(), // 创建完成实例
    }
}

func (l *generalLogger) run() {
    defer l.access.Signal() // 延迟释放信号量

    dataWritten := false // 数据是否已写入标志
    ticker := time.NewTicker(time.Minute) // 创建定时器
    defer ticker.Stop() // 延迟关闭定时器

    logger := l.creator() // 创建日志写入器
    if logger == nil { // 如果日志写入器为空
        return // 直接返回
    }
    defer logger.Close() // 延迟关闭日志写入器 // nolint: errcheck

    for {
        select {
        case <-l.done.Wait(): // 等待完成信号
            return // 直接返回
        case msg := <-l.buffer: // 从消息缓冲通道中读取消息
            logger.Write(msg.String() + platform.LineSeparator()) // 写入消息到日志写入器 // nolint: errcheck
            dataWritten = true // 设置数据已写入标志
        case <-ticker.C: // 定时器触发
            if !dataWritten { // 如果数据未写入
                return // 直接返回
            }
            dataWritten = false // 重置数据已写入标志
        }
    }
}

func (l *generalLogger) Handle(msg Message) {
    select {
    case l.buffer <- msg: // 将消息放入缓冲通道
    default:
    }

    select {
    case <-l.access.Wait(): // 等待信号量
        go l.run() // 启动日志处理
    default:
    }
}

func (l *generalLogger) Close() error {
    return l.done.Close() // 关闭完成实例
}

type consoleLogWriter struct {
    logger *log.Logger // 日志记录器
}

func (w *consoleLogWriter) Write(s string) error {
    w.logger.Print(s) // 打印日志
    return nil // 返回空错误
}

func (w *consoleLogWriter) Close() error {
    return nil // 返回空错误
}

type fileLogWriter struct {
    file   *os.File // 文件指针
    # 声明一个名为logger的log.Logger类型的变量
// Write 方法用于向文件日志写入字符串，并调用 logger 的 Print 方法
func (w *fileLogWriter) Write(s string) error {
    w.logger.Print(s)
    return nil
}

// Close 方法用于关闭文件日志
func (w *fileLogWriter) Close() error {
    return w.file.Close()
}

// CreateStdoutLogWriter 返回一个 LogWriterCreator，用于创建 stdout 的日志写入器
func CreateStdoutLogWriter() WriterCreator {
    return func() Writer {
        return &consoleLogWriter{
            logger: log.New(os.Stdout, "", log.Ldate|log.Ltime),
        }
    }
}

// CreateStderrLogWriter 返回一个 LogWriterCreator，用于创建 stderr 的日志写入器
func CreateStderrLogWriter() WriterCreator {
    return func() Writer {
        return &consoleLogWriter{
            logger: log.New(os.Stderr, "", log.Ldate|log.Ltime),
        }
    }
}

// CreateFileLogWriter 返回一个 LogWriterCreator，用于创建指定文件的日志写入器
func CreateFileLogWriter(path string) (WriterCreator, error) {
    file, err := os.OpenFile(path, os.O_APPEND|os.O_WRONLY|os.O_CREATE, 0600)
    if err != nil {
        return nil, err
    }
    file.Close()
    return func() Writer {
        file, err := os.OpenFile(path, os.O_APPEND|os.O_WRONLY|os.O_CREATE, 0600)
        if err != nil {
            return nil
        }
        return &fileLogWriter{
            file:   file,
            logger: log.New(file, "", log.Ldate|log.Ltime),
        }
    }, nil
}

// init 方法用于注册日志处理程序
func init() {
    RegisterHandler(NewLogger(CreateStdoutLogWriter()))
}
```