# `trojan-go\log\simplelog\simplelog.go`

```
package simplelog

import (
    "io"  // 导入io包，用于进行输入输出操作
    golog "log"  // 导入log包，并起别名为golog
    "os"  // 导入os包，用于操作系统功能

    "github.com/p4gefau1t/trojan-go/log"  // 导入自定义的log包
)

func init() {
    log.RegisterLogger(&SimpleLogger{})  // 在程序初始化时注册SimpleLogger
}

type SimpleLogger struct {
    logLevel log.LogLevel  // 定义SimpleLogger结构体，包含logLevel字段
}

func (l *SimpleLogger) SetLogLevel(level log.LogLevel) {  // 设置日志级别的方法
    l.logLevel = level  // 将传入的日志级别赋值给logLevel字段
}

func (l *SimpleLogger) Fatal(v ...interface{}) {  // 记录严重错误的方法
    if l.logLevel <= log.FatalLevel {  // 如果日志级别小于等于FatalLevel
        golog.Fatal(v...)  // 调用golog的Fatal方法记录日志
    }
    os.Exit(1)  // 退出程序
}

func (l *SimpleLogger) Fatalf(format string, v ...interface{}) {  // 格式化记录严重错误的方法
    if l.logLevel <= log.FatalLevel {  // 如果日志级别小于等于FatalLevel
        golog.Fatalf(format, v...)  // 使用指定格式记录日志
    }
    os.Exit(1)  // 退出程序
}

// 以下方法作用类似，根据日志级别调用golog的不同方法记录日志
func (l *SimpleLogger) Error(v ...interface{}) {
    if l.logLevel <= log.ErrorLevel {
        golog.Println(v...)
    }
}

func (l *SimpleLogger) Errorf(format string, v ...interface{}) {
    if l.logLevel <= log.ErrorLevel {
        golog.Printf(format, v...)
    }
}

func (l *SimpleLogger) Warn(v ...interface{}) {
    if l.logLevel <= log.WarnLevel {
        golog.Println(v...)
    }
}

func (l *SimpleLogger) Warnf(format string, v ...interface{}) {
    if l.logLevel <= log.WarnLevel {
        golog.Printf(format, v...)
    }
}

func (l *SimpleLogger) Info(v ...interface{}) {
    if l.logLevel <= log.InfoLevel {
        golog.Println(v...)
    }
}

func (l *SimpleLogger) Infof(format string, v ...interface{}) {
    if l.logLevel <= log.InfoLevel {
        golog.Printf(format, v...)
    }
}

func (l *SimpleLogger) Debug(v ...interface{}) {
    if l.logLevel <= log.AllLevel {
        golog.Println(v...)
    }
}

func (l *SimpleLogger) Debugf(format string, v ...interface{}) {
    if l.logLevel <= log.AllLevel {
        golog.Printf(format, v...)
    }
}

func (l *SimpleLogger) Trace(v ...interface{}) {
    if l.logLevel <= log.AllLevel {
        golog.Println(v...)
    }
}

func (l *SimpleLogger) Tracef(format string, v ...interface{}) {
    if l.logLevel <= log.AllLevel {
        golog.Printf(format, v...)
    }
}

func (l *SimpleLogger) SetOutput(io.Writer) {  // 设置日志输出的方法
    // 这行代码是一个注释，实际上并没有执行任何操作
# 闭合前面的函数定义
```