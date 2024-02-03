# `grype\internal\logger\logrus.go`

```go
package logger

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"   // 导入 io 包，用于进行输入输出操作
    "io/fs"  // 导入 io/fs 包，用于文件系统相关操作
    "os"   // 导入 os 包，用于操作系统相关功能

    "github.com/sirupsen/logrus"  // 导入 logrus 包，用于日志记录
    prefixed "github.com/x-cray/logrus-prefixed-formatter"  // 导入 logrus-prefixed-formatter 包，用于日志格式化
)

const defaultLogFilePermissions fs.FileMode = 0644  // 定义默认的日志文件权限

type LogrusConfig struct {
    EnableConsole bool  // 是否启用控制台输出
    EnableFile    bool  // 是否启用文件输出
    Structured    bool  // 是否使用结构化日志
    Level         logrus.Level  // 日志级别
    FileLocation  string  // 日志文件位置
}

type LogrusLogger struct {
    Config LogrusConfig  // 日志配置
    Logger *logrus.Logger  // logrus 日志记录器
    Output io.Writer  // 输出流
}

type LogrusNestedLogger struct {
    Logger *logrus.Entry  // logrus 日志记录器
}

func NewLogrusLogger(cfg LogrusConfig) *LogrusLogger {
    appLogger := logrus.New()  // 创建一个新的 logrus 日志记录器

    var output io.Writer  // 声明一个输出流变量
    switch {
    case cfg.EnableConsole && cfg.EnableFile:  // 如果同时启用控制台和文件输出
        logFile, err := os.OpenFile(cfg.FileLocation, os.O_WRONLY|os.O_CREATE, defaultLogFilePermissions)  // 打开或创建日志文件
        if err != nil {
            panic(fmt.Errorf("unable to setup log file: %w", err))  // 如果出错，抛出异常
        }
        output = io.MultiWriter(os.Stderr, logFile)  // 输出流为标准错误和日志文件
    case cfg.EnableConsole:  // 如果只启用控制台输出
        output = os.Stderr  // 输出流为标准错误
    case cfg.EnableFile:  // 如果只启用文件输出
        logFile, err := os.OpenFile(cfg.FileLocation, os.O_WRONLY|os.O_CREATE, defaultLogFilePermissions)  // 打开或创建日志文件
        if err != nil {
            panic(fmt.Errorf("unable to setup log file: %w", err))  // 如果出错，抛出异常
        }
        output = logFile  // 输出流为日志文件
    default:  // 如果都不启用
        output = io.Discard  // 输出流为空
    }

    appLogger.SetOutput(output)  // 设置日志记录器的输出流
    appLogger.SetLevel(cfg.Level)  // 设置日志记录器的日志级别

    if cfg.Structured {  // 如果使用结构化日志
        appLogger.SetFormatter(&logrus.JSONFormatter{  // 设置日志记录器的格式化器为 JSON 格式
            TimestampFormat:   "2006-01-02 15:04:05",  // 时间戳格式
            DisableTimestamp:  false,  // 不禁用时间戳
            DisableHTMLEscape: false,  // 不禁用 HTML 转义
            PrettyPrint:       false,  // 不美化输出
        })
    } else {  // 如果不使用结构化日志
        appLogger.SetFormatter(&prefixed.TextFormatter{  // 设置日志记录器的格式化器为文本格式
            TimestampFormat: "2006-01-02 15:04:05",  // 时间戳格式
            ForceColors:     true,  // 强制使用颜色
            ForceFormatting: true,  // 强制格式化
        })
    }

    return &LogrusLogger{  // 返回 LogrusLogger 对象
        Config: cfg,  // 配置信息
        Logger: appLogger,  // logrus 日志记录器
        Output: output,  // 输出流
    }
}
# 定义 LogrusLogger 结构体的 Debugf 方法，接收格式化字符串和参数，调用 Logger 结构体的 Debugf 方法
func (l *LogrusLogger) Debugf(format string, args ...interface{}) {
    l.Logger.Debugf(format, args...)
}

# 定义 LogrusLogger 结构体的 Infof 方法，接收格式化字符串和参数，调用 Logger 结构体的 Infof 方法
func (l *LogrusLogger) Infof(format string, args ...interface{}) {
    l.Logger.Infof(format, args...)
}

# 定义 LogrusLogger 结构体的 Warnf 方法，接收格式化字符串和参数，调用 Logger 结构体的 Warnf 方法
func (l *LogrusLogger) Warnf(format string, args ...interface{}) {
    l.Logger.Warnf(format, args...)
}

# 定义 LogrusLogger 结构体的 Errorf 方法，接收格式化字符串和参数，调用 Logger 结构体的 Errorf 方法
func (l *LogrusLogger) Errorf(format string, args ...interface{}) {
    l.Logger.Errorf(format, args...)
}

# 定义 LogrusLogger 结构体的 Debug 方法，接收参数，调用 Logger 结构体的 Debug 方法
func (l *LogrusLogger) Debug(args ...interface{}) {
    l.Logger.Debug(args...)
}

# 定义 LogrusLogger 结构体的 Info 方法，接收参数，调用 Logger 结构体的 Info 方法
func (l *LogrusLogger) Info(args ...interface{}) {
    l.Logger.Info(args...)
}

# 定义 LogrusLogger 结构体的 Warn 方法，接收参数，调用 Logger 结构体的 Warn 方法
func (l *LogrusLogger) Warn(args ...interface{}) {
    l.Logger.Warn(args...)
}

# 定义 LogrusLogger 结构体的 Error 方法，接收参数，调用 Logger 结构体的 Error 方法
func (l *LogrusLogger) Error(args ...interface{}) {
    l.Logger.Error(args...)
}

# 定义 LogrusNestedLogger 结构体的 Debugf 方法，接收格式化字符串和参数，调用 Logger 结构体的 Debugf 方法
func (l *LogrusNestedLogger) Debugf(format string, args ...interface{}) {
    l.Logger.Debugf(format, args...)
}

# 定义 LogrusNestedLogger 结构体的 Infof 方法，接收格式化字符串和参数，调用 Logger 结构体的 Infof 方法
func (l *LogrusNestedLogger) Infof(format string, args ...interface{}) {
    l.Logger.Infof(format, args...)
}

# 定义 LogrusNestedLogger 结构体的 Warnf 方法，接收格式化字符串和参数，调用 Logger 结构体的 Warnf 方法
func (l *LogrusNestedLogger) Warnf(format string, args ...interface{}) {
    l.Logger.Warnf(format, args...)
}

# 定义 LogrusNestedLogger 结构体的 Errorf 方法，接收格式化字符串和参数，调用 Logger 结构体的 Errorf 方法
func (l *LogrusNestedLogger) Errorf(format string, args ...interface{}) {
    l.Logger.Errorf(format, args...)
}

# 定义 LogrusNestedLogger 结构体的 Debug 方法，接收参数，调用 Logger 结构体的 Debug 方法
func (l *LogrusNestedLogger) Debug(args ...interface{}) {
    l.Logger.Debug(args...)
}

# 定义 LogrusNestedLogger 结构体的 Info 方法，接收参数，调用 Logger 结构体的 Info 方法
func (l *LogrusNestedLogger) Info(args ...interface{}) {
    l.Logger.Info(args...)
}

# 定义 LogrusNestedLogger 结构体的 Warn 方法，接收参数，调用 Logger 结构体的 Warn 方法
func (l *LogrusNestedLogger) Warn(args ...interface{}) {
    l.Logger.Warn(args...)
}

# 定义 LogrusNestedLogger 结构体的 Error 方法，接收参数，调用 Logger 结构体的 Error 方法
func (l *LogrusNestedLogger) Error(args ...interface{}) {
    l.Logger.Error(args...)
}
```