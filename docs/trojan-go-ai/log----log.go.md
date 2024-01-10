# `trojan-go\log\log.go`

```
package log

import (
    "io"  // 导入io包，用于处理输入输出
    "os"  // 导入os包，用于操作系统功能
)

// LogLevel how much log to dump
// 0: ALL; 1: INFO; 2: WARN; 3: ERROR; 4: FATAL; 5: OFF
type LogLevel int  // 定义日志级别类型为整数

const (
    AllLevel   LogLevel = 0  // 定义日志级别常量
    InfoLevel  LogLevel = 1  // 定义日志级别常量
    WarnLevel  LogLevel = 2  // 定义日志级别常量
    ErrorLevel LogLevel = 3  // 定义日志级别常量
    FatalLevel LogLevel = 4  // 定义日志级别常量
    OffLevel   LogLevel = 5  // 定义日志级别常量
)

type Logger interface {  // 定义Logger接口
    Fatal(v ...interface{})  // 定义接口方法
    Fatalf(format string, v ...interface{})  // 定义接口方法
    Error(v ...interface{})  // 定义接口方法
    Errorf(format string, v ...interface{})  // 定义接口方法
    Warn(v ...interface{})  // 定义接口方法
    Warnf(format string, v ...interface{})  // 定义接口方法
    Info(v ...interface{})  // 定义接口方法
    Infof(format string, v ...interface{})  // 定义接口方法
    Debug(v ...interface{})  // 定义接口方法
    Debugf(format string, v ...interface{})  // 定义接口方法
    Trace(v ...interface{})  // 定义接口方法
    Tracef(format string, v ...interface{})  // 定义接口方法
    SetLogLevel(level LogLevel)  // 定义接口方法
    SetOutput(io.Writer)  // 定义接口方法
}

var logger Logger = &EmptyLogger{}  // 定义并初始化logger变量为EmptyLogger类型的实例

type EmptyLogger struct{}  // 定义EmptyLogger结构体

func (l *EmptyLogger) SetLogLevel(LogLevel) {}  // 实现EmptyLogger结构体的SetLogLevel方法

func (l *EmptyLogger) Fatal(v ...interface{}) { os.Exit(1) }  // 实现EmptyLogger结构体的Fatal方法

func (l *EmptyLogger) Fatalf(format string, v ...interface{}) { os.Exit(1) }  // 实现EmptyLogger结构体的Fatalf方法

func (l *EmptyLogger) Error(v ...interface{}) {}  // 实现EmptyLogger结构体的Error方法

func (l *EmptyLogger) Errorf(format string, v ...interface{}) {}  // 实现EmptyLogger结构体的Errorf方法

func (l *EmptyLogger) Warn(v ...interface{}) {}  // 实现EmptyLogger结构体的Warn方法

func (l *EmptyLogger) Warnf(format string, v ...interface{}) {}  // 实现EmptyLogger结构体的Warnf方法

func (l *EmptyLogger) Info(v ...interface{}) {}  // 实现EmptyLogger结构体的Info方法

func (l *EmptyLogger) Infof(format string, v ...interface{}) {}  // 实现EmptyLogger结构体的Infof方法

func (l *EmptyLogger) Debug(v ...interface{}) {}  // 实现EmptyLogger结构体的Debug方法

func (l *EmptyLogger) Debugf(format string, v ...interface{}) {}  // 实现EmptyLogger结构体的Debugf方法

func (l *EmptyLogger) Trace(v ...interface{}) {}  // 实现EmptyLogger结构体的Trace方法

func (l *EmptyLogger) Tracef(format string, v ...interface{}) {}  // 实现EmptyLogger结构体的Tracef方法

func (l *EmptyLogger) SetOutput(w io.Writer) {}  // 实现EmptyLogger结构体的SetOutput方法

func Error(v ...interface{}) {  // 定义Error函数
    logger.Error(v...)  // 调用logger的Error方法
}

func Errorf(format string, v ...interface{}) {  // 定义Errorf函数
    logger.Errorf(format, v...)  // 调用logger的Errorf方法
}

func Warn(v ...interface{}) {  // 定义Warn函数
    logger.Warn(v...)  // 调用logger的Warn方法
}

func Warnf(format string, v ...interface{}) {  // 定义Warnf函数
    logger.Warnf(format, v...)  // 调用logger的Warnf方法
}
# 打印信息日志，接受可变数量的参数
func Info(v ...interface{}) {
    logger.Info(v...)
}

# 打印格式化信息日志，接受格式化字符串和可变数量的参数
func Infof(format string, v ...interface{}) {
    logger.Infof(format, v...)
}

# 打印调试日志，接受可变数量的参数
func Debug(v ...interface{}) {
    logger.Debug(v...)
}

# 打印格式化调试日志，接受格式化字符串和可变数量的参数
func Debugf(format string, v ...interface{}) {
    logger.Debugf(format, v...)
}

# 打印跟踪日志，接受可变数量的参数
func Trace(v ...interface{}) {
    logger.Trace(v...)
}

# 打印格式化跟踪日志，接受格式化字符串和可变数量的参数
func Tracef(format string, v ...interface{}) {
    logger.Tracef(format, v...)
}

# 打印致命错误日志，接受可变数量的参数
func Fatal(v ...interface{}) {
    logger.Fatal(v...)
}

# 打印格式化致命错误日志，接受格式化字符串和可变数量的参数
func Fatalf(format string, v ...interface{}) {
    logger.Fatalf(format, v...)
}

# 设置日志级别
func SetLogLevel(level LogLevel) {
    logger.SetLogLevel(level)
}

# 设置日志输出目标
func SetOutput(w io.Writer) {
    logger.SetOutput(w)
}

# 注册日志记录器
func RegisterLogger(l Logger) {
    logger = l
}
```