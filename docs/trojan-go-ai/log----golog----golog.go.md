# `trojan-go\log\golog\golog.go`

```go
// 引入必要的包
package golog

import (
    "fmt"  // 格式化输出
    "io"  // 输入输出
    "os"  // 操作系统功能
    "path/filepath"  // 文件路径操作
    "runtime"  // 运行时信息
    "sync"  // 同步功能
    "sync/atomic"  // 原子操作
    "time"  // 时间操作

    terminal "golang.org/x/term"  // 终端操作

    "github.com/p4gefau1t/trojan-go/log"  // 引入日志包
    "github.com/p4gefau1t/trojan-go/log/golog/colorful"  // 引入彩色日志包
)

// 初始化函数，在包被加载时自动执行
func init() {
    log.RegisterLogger(New(os.Stdout))  // 注册日志记录器
}

// FdWriter 接口扩展了现有的 io.Writer，支持文件描述符功能
type FdWriter interface {
    io.Writer  // 继承 io.Writer 接口
    Fd() uintptr  // 获取文件描述符
}

// Logger 结构定义了单个日志记录器的基础存储
type Logger struct {
    mu        sync.RWMutex  // 读写锁
    color     bool  // 是否启用彩色输出
    out       io.Writer  // 输出目标
    debug     bool  // 是否启用调试模式
    timestamp bool  // 是否显示时间戳
    quiet     bool  // 是否安静模式
    buf       colorful.ColorBuffer  // 彩色缓冲区
    logLevel  int32  // 日志级别
}

// Prefix 结构定义了普通和彩色字节
type Prefix struct {
    Plain []byte  // 普通字节
    Color []byte  // 彩色字节
    File  bool  // 是否显示文件信息
}

var (
    // 普通前缀模板
    plainFatal = []byte("[FATAL] ")
    plainError = []byte("[ERROR] ")
    plainWarn  = []byte("[WARN]  ")
    plainInfo  = []byte("[INFO]  ")
    plainDebug = []byte("[DEBUG] ")
    plainTrace = []byte("[TRACE] ")

    // FatalPrefix 显示致命错误前缀
    FatalPrefix = Prefix{
        Plain: plainFatal,
        Color: colorful.Red(plainFatal),
        File:  true,
    }

    // ErrorPrefix 显示错误前缀
    ErrorPrefix = Prefix{
        Plain: plainError,
        Color: colorful.Red(plainError),
        File:  true,
    }

    // WarnPrefix 显示警告前缀
    WarnPrefix = Prefix{
        Plain: plainWarn,
        Color: colorful.Orange(plainWarn),
    }

    // InfoPrefix 显示信息前缀
    InfoPrefix = Prefix{
        Plain: plainInfo,
        Color: colorful.Green(plainInfo),
    }

    // DebugPrefix 显示调试前缀
    DebugPrefix = Prefix{
        Plain: plainDebug,
        Color: colorful.Purple(plainDebug),
        File:  true,
    }

    // TracePrefix 显示跟踪前缀
    # 创建一个名为TracePrefix的变量，其值为Prefix结构体
    TracePrefix = Prefix{
        # 在Prefix结构体中的Plain字段赋值为plainTrace变量的值
        Plain: plainTrace,
        # 在Prefix结构体中的Color字段赋值为使用colorful包中的Cyan函数对plainTrace变量的值进行着色后的结果
        Color: colorful.Cyan(plainTrace),
    }
// New函数返回一个具有预定义写入输出和自动检测终端颜色支持的新Logger实例
func New(out FdWriter) *Logger {
    return &Logger{
        color:     terminal.IsTerminal(int(out.Fd())), // 检测终端是否支持颜色
        out:       out, // 设置输出
        timestamp: true, // 设置时间戳为true
    }
}

// SetLogLevel设置日志级别
func (l *Logger) SetLogLevel(level log.LogLevel) {
    l.mu.Lock() // 加锁
    defer l.mu.Unlock() // 解锁
    atomic.StoreInt32(&l.logLevel, int32(level)) // 存储日志级别
}

// SetOutput设置输出
func (l *Logger) SetOutput(w io.Writer) {
    l.mu.Lock() // 加锁
    defer l.mu.Unlock() // 解锁
    l.color = false // 关闭颜色
    if fdw, ok := w.(FdWriter); ok {
        l.color = terminal.IsTerminal(int(fdw.Fd())) // 检测终端是否支持颜色
    }
    l.out = w // 设置输出
}

// WithColor显式打开日志的彩色特性
func (l *Logger) WithColor() *Logger {
    l.mu.Lock() // 加锁
    defer l.mu.Unlock() // 解锁
    l.color = true // 打开颜色
    return l
}

// WithoutColor显式关闭日志的彩色特性
func (l *Logger) WithoutColor() *Logger {
    l.mu.Lock() // 加锁
    defer l.mu.Unlock() // 解锁
    l.color = false // 关闭颜色
    return l
}

// WithDebug打开日志的调试输出以显示调试和跟踪级别
func (l *Logger) WithDebug() *Logger {
    l.mu.Lock() // 加锁
    defer l.mu.Unlock() // 解锁
    l.debug = true // 打开调试
    return l
}

// WithoutDebug关闭日志的调试输出
func (l *Logger) WithoutDebug() *Logger {
    l.mu.Lock() // 加锁
    defer l.mu.Unlock() // 解锁
    l.debug = false // 关闭调试
    return l
}

// IsDebug检查调试输出的状态
func (l *Logger) IsDebug() bool {
    l.mu.RLock() // 加读锁
    defer l.mu.RUnlock() // 解读锁
    return l.debug // 返回调试状态
}

// WithTimestamp打开日志的时间戳输出
func (l *Logger) WithTimestamp() *Logger {
    l.mu.Lock() // 加锁
    defer l.mu.Unlock() // 解锁
    l.timestamp = true // 打开时间戳
    return l
}

// WithoutTimestamp关闭日志的时间戳输出
func (l *Logger) WithoutTimestamp() *Logger {
    l.mu.Lock() // 加锁
    defer l.mu.Unlock() // 解锁
    l.timestamp = false // 关闭时间戳
    return l
}

// Quiet关闭所有日志输出
func (l *Logger) Quiet() *Logger {
    l.mu.Lock() // 加锁
    defer l.mu.Unlock() // 解锁
    l.quiet = true // 设置为安静模式
}
    # 返回变量 l 的值
    return l
// 关闭所有日志输出
func (l *Logger) NoQuiet() *Logger {
    // 加锁，确保线程安全
    l.mu.Lock()
    // 在函数返回时解锁
    defer l.mu.Unlock()
    // 设置 quiet 属性为 false
    l.quiet = false
    // 返回 Logger 对象
    return l
}

// 检查 quiet 状态
func (l *Logger) IsQuiet() bool {
    // 加读锁，确保线程安全
    l.mu.RLock()
    // 在函数返回时解锁
    defer l.mu.RUnlock()
    // 返回 quiet 属性的值
    return l.quiet
}

// 输出实际值
func (l *Logger) Output(depth int, prefix Prefix, data string) error {
    // 检查是否需要安静输出，尝试返回无错误并保持安静
    if l.IsQuiet() {
        return nil
    }
    // 获取当前时间
    now := time.Now()
    // 临时存储文件和行信息
    var file string
    var line int
    var fn string
    // 检查是否需要在文件日志中包含指定的前缀
    if prefix.File {
        var ok bool
        var pc uintptr

        // 获取调用者的文件名和行号
        if pc, file, line, ok = runtime.Caller(depth + 2); !ok {
            file = "<unknown file>"
            fn = "<unknown function>"
            line = 0
        } else {
            file = filepath.Base(file)
            fn = runtime.FuncForPC(pc).Name()
        }
    }
    // 获取对共享缓冲区的独占访问
    l.mu.Lock()
    // 在函数返回时解锁
    defer l.mu.Unlock()
    // 重置缓冲区，使其从开头开始
    l.buf.Reset()
    // 将前缀写入缓冲区
    if l.color {
        l.buf.Append(prefix.Color)
    } else {
        l.buf.Append(prefix.Plain)
    }
    // 检查日志是否需要时间戳
    // 如果日志有时间戳
    if l.timestamp {
        // 如果启用了颜色，打印时间戳颜色
        if l.color {
            l.buf.Blue()
        }
        // 打印日期和时间
        year, month, day := now.Date()
        l.buf.AppendInt(year, 4)
        l.buf.AppendByte('/')
        l.buf.AppendInt(int(month), 2)
        l.buf.AppendByte('/')
        l.buf.AppendInt(day, 2)
        l.buf.AppendByte(' ')
        hour, min, sec := now.Clock()
        l.buf.AppendInt(hour, 2)
        l.buf.AppendByte(':')
        l.buf.AppendInt(min, 2)
        l.buf.AppendByte(':')
        l.buf.AppendInt(sec, 2)
        l.buf.AppendByte(' ')
        // 如果启用了颜色，打印重置颜色
        if l.color {
            l.buf.Off()
        }
    }
    // 如果启用了文件名和行号前缀
    if prefix.File {
        // 如果启用了颜色，打印颜色开始
        if l.color {
            l.buf.Orange()
        }
        // 打印文件名和行号
        l.buf.Append([]byte(fn))
        l.buf.AppendByte(':')
        l.buf.Append([]byte(file))
        l.buf.AppendByte(':')
        l.buf.AppendInt(line, 0)
        l.buf.AppendByte(' ')
        // 打印颜色结束
        if l.color {
            l.buf.Off()
        }
    }
    // 打印来自调用者的实际字符串数据
    l.buf.Append([]byte(data))
    // 如果数据为空或者最后一个字符不是换行符，添加换行符
    if len(data) == 0 || data[len(data)-1] != '\n' {
        l.buf.AppendByte('\n')
    }
    // 刷新缓冲区到输出
    _, err := l.out.Write(l.buf.Buffer)
    return err
// Fatal 打印致命错误信息到输出并以状态 1 退出应用程序
func (l *Logger) Fatal(v ...interface{}) {
    // 如果日志级别小于等于 4，则输出致命错误信息
    if atomic.LoadInt32(&l.logLevel) <= 4 {
        l.Output(1, FatalPrefix, fmt.Sprintln(v...))
    }
    // 退出应用程序
    os.Exit(1)
}

// Fatalf 打印格式化的致命错误信息到输出并以状态 1 退出应用程序
func (l *Logger) Fatalf(format string, v ...interface{}) {
    // 如果日志级别小于等于 4，则输出格式化的致命错误信息
    if atomic.LoadInt32(&l.logLevel) <= 4 {
        l.Output(1, FatalPrefix, fmt.Sprintf(format, v...))
    }
    // 退出应用程序
    os.Exit(1)
}

// Error 打印错误信息到输出
func (l *Logger) Error(v ...interface{}) {
    // 如果日志级别小于等于 3，则输出错误信息
    if atomic.LoadInt32(&l.logLevel) <= 3 {
        l.Output(1, ErrorPrefix, fmt.Sprintln(v...))
    }
}

// Errorf 打印格式化的错误信息到输出
func (l *Logger) Errorf(format string, v ...interface{}) {
    // 如果日志级别小于等于 3，则输出格式化的错误信息
    if atomic.LoadInt32(&l.logLevel) <= 3 {
        l.Output(1, ErrorPrefix, fmt.Sprintf(format, v...))
    }
}

// Warn 打印警告信息到输出
func (l *Logger) Warn(v ...interface{}) {
    // 如果日志级别小于等于 2，则输出警告信息
    if atomic.LoadInt32(&l.logLevel) <= 2 {
        l.Output(1, WarnPrefix, fmt.Sprintln(v...))
    }
}

// Warnf 打印格式化的警告信息到输出
func (l *Logger) Warnf(format string, v ...interface{}) {
    // 如果日志级别小于等于 2，则输出格式化的警告信息
    if atomic.LoadInt32(&l.logLevel) <= 2 {
        l.Output(1, WarnPrefix, fmt.Sprintf(format, v...))
    }
}

// Info 打印信息性消息到输出
func (l *Logger) Info(v ...interface{}) {
    // 如果日志级别小于等于 1，则输出信息性消息
    if atomic.LoadInt32(&l.logLevel) <= 1 {
        l.Output(1, InfoPrefix, fmt.Sprintln(v...))
    }
}

// Infof 打印格式化的信息性消息到输出
func (l *Logger) Infof(format string, v ...interface{}) {
    // 如果日志级别小于等于 1，则输出格式化的信息性消息
    if atomic.LoadInt32(&l.logLevel) <= 1 {
        l.Output(1, InfoPrefix, fmt.Sprintf(format, v...))
    }
}

// Debug 如果启用了调试输出，则打印调试消息到输出
func (l *Logger) Debug(v ...interface{}) {
    // 如果日志级别为 0，则输出调试消息
    if atomic.LoadInt32(&l.logLevel) == 0 {
        l.Output(1, DebugPrefix, fmt.Sprintln(v...))
    }
}
// 如果调试输出被启用，则将格式化的调试消息打印到输出
func (l *Logger) Debugf(format string, v ...interface{}) {
    // 检查日志级别是否为调试模式
    if atomic.LoadInt32(&l.logLevel) == 0 {
        // 调用Output方法，输出调试消息
        l.Output(1, DebugPrefix, fmt.Sprintf(format, v...))
    }
}

// 如果调试输出被启用，则将跟踪消息打印到输出
func (l *Logger) Trace(v ...interface{}) {
    // 检查日志级别是否为调试模式
    if atomic.LoadInt32(&l.logLevel) == 0 {
        // 调用Output方法，输出跟踪消息
        l.Output(1, TracePrefix, fmt.Sprintln(v...))
    }
}

// 如果调试输出被启用，则将格式化的跟踪消息打印到输出
func (l *Logger) Tracef(format string, v ...interface{}) {
    // 检查日志级别是否为调试模式
    if atomic.LoadInt32(&l.logLevel) == 0 {
        // 调用Output方法，输出格式化的跟踪消息
        l.Output(1, TracePrefix, fmt.Sprintf(format, v...))
    }
}
```