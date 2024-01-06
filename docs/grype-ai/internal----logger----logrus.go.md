# `grype\internal\logger\logrus.go`

```
package logger
// 声明一个名为logger的包

import (
	"fmt"
	"io"
	"io/fs"
	"os"

	"github.com/sirupsen/logrus"
	prefixed "github.com/x-cray/logrus-prefixed-formatter"
)
// 导入所需的包

const defaultLogFilePermissions fs.FileMode = 0644
// 声明一个常量defaultLogFilePermissions，表示默认的日志文件权限为0644

type LogrusConfig struct {
	EnableConsole bool
	EnableFile    bool
	Structured    bool
	Level         logrus.Level
	FileLocation  string
}
// 声明一个结构体LogrusConfig，包含日志配置的各个属性
}

// LogrusLogger 结构体定义，包含配置信息、日志记录器和输出流
type LogrusLogger struct {
	Config LogrusConfig
	Logger *logrus.Logger
	Output io.Writer
}

// LogrusNestedLogger 结构体定义，包含日志记录器
type LogrusNestedLogger struct {
	Logger *logrus.Entry
}

// NewLogrusLogger 函数，根据配置信息创建并返回 LogrusLogger 对象
func NewLogrusLogger(cfg LogrusConfig) *LogrusLogger {
    // 创建一个新的日志记录器
    appLogger := logrus.New()

    var output io.Writer
    // 根据配置信息选择输出方式
    switch {
    case cfg.EnableConsole && cfg.EnableFile:
        // 打开或创建日志文件
        logFile, err := os.OpenFile(cfg.FileLocation, os.O_WRONLY|os.O_CREATE, defaultLogFilePermissions)
        if err != nil {
// 如果配置中启用了日志文件，并且未启用控制台输出，则将日志输出到日志文件和标准错误流
panic(fmt.Errorf("unable to setup log file: %w", err)) // 如果设置日志文件失败，则抛出错误
output = io.MultiWriter(os.Stderr, logFile) // 将日志输出到标准错误流和日志文件

// 如果配置中启用了控制台输出，则将日志输出到标准错误流
output = os.Stderr

// 如果配置中启用了日志文件，则将日志输出到指定的日志文件
logFile, err := os.OpenFile(cfg.FileLocation, os.O_WRONLY|os.O_CREATE, defaultLogFilePermissions) // 打开或创建日志文件
if err != nil {
    panic(fmt.Errorf("unable to setup log file: %w", err)) // 如果设置日志文件失败，则抛出错误
}
output = logFile // 将日志输出到日志文件

// 如果未启用任何日志输出，则将日志输出丢弃
output = io.Discard

appLogger.SetOutput(output) // 设置日志输出目标
appLogger.SetLevel(cfg.Level) // 设置日志级别

// 如果配置中启用了结构化日志，则设置日志格式为 JSON 格式
appLogger.SetFormatter(&logrus.JSONFormatter{
# 设置时间戳的格式为 "年-月-日 时:分:秒"
TimestampFormat:   "2006-01-02 15:04:05",
# 不禁用时间戳
DisableTimestamp:  false,
# 不禁用 HTML 转义
DisableHTMLEscape: false,
# 不使用漂亮的打印格式
PrettyPrint:       false,
# 如果条件成立，使用指定的格式化器
if condition {
    appLogger.SetFormatter(&prefixed.TextFormatter{
        # 设置时间戳的格式为 "年-月-日 时:分:秒"
        TimestampFormat: "2006-01-02 15:04:05",
        # 强制使用颜色
        ForceColors:     true,
        # 强制格式化
        ForceFormatting: true,
    })
} 
# 如果条件不成立，使用另一种格式化器
else {
    appLogger.SetFormatter(&prefixed.TextFormatter{
        # 设置时间戳的格式为 "年-月-日 时:分:秒"
        TimestampFormat: "2006-01-02 15:04:05",
        # 强制使用颜色
        ForceColors:     true,
        # 强制格式化
        ForceFormatting: true,
    })
}
# 返回 LogrusLogger 对象，其中包含配置、日志记录器和输出
return &LogrusLogger{
    Config: cfg,
    Logger: appLogger,
    Output: output,
}
# 定义一个方法，用于输出调试级别的日志，接受格式化字符串和参数
func (l *LogrusLogger) Debugf(format string, args ...interface{}) {
    # 调用Logrus库中的Debugf方法，将格式化字符串和参数传递给它
    l.Logger.Debugf(format, args...)
}

# 定义一个方法，用于输出信息级别的日志，接受格式化字符串和参数
func (l *LogrusLogger) Infof(format string, args ...interface{}) {
    # 调用Logrus库中的Infof方法，将格式化字符串和参数传递给它
    l.Logger.Infof(format, args...)
}

# 定义一个方法，用于输出警告级别的日志，接受格式化字符串和参数
func (l *LogrusLogger) Warnf(format string, args ...interface{}) {
    # 调用Logrus库中的Warnf方法，将格式化字符串和参数传递给它
    l.Logger.Warnf(format, args...)
}

# 定义一个方法，用于输出错误级别的日志，接受格式化字符串和参数
func (l *LogrusLogger) Errorf(format string, args ...interface{}) {
    # 调用Logrus库中的Errorf方法，将格式化字符串和参数传递给它
    l.Logger.Errorf(format, args...)
}

# 定义一个方法，用于输出调试级别的日志，接受参数
func (l *LogrusLogger) Debug(args ...interface{}) {
    # 调用Logrus库中的Debug方法，将参数传递给它
    l.Logger.Debug(args...)
}
# 定义 LogrusLogger 结构体的 Info 方法，用于记录信息级别的日志
func (l *LogrusLogger) Info(args ...interface{}) {
    # 调用 LogrusLogger 结构体中的 Logger 对象的 Info 方法，记录信息级别的日志
    l.Logger.Info(args...)
}

# 定义 LogrusLogger 结构体的 Warn 方法，用于记录警告级别的日志
func (l *LogrusLogger) Warn(args ...interface{}) {
    # 调用 LogrusLogger 结构体中的 Logger 对象的 Warn 方法，记录警告级别的日志
    l.Logger.Warn(args...)
}

# 定义 LogrusLogger 结构体的 Error 方法，用于记录错误级别的日志
func (l *LogrusLogger) Error(args ...interface{}) {
    # 调用 LogrusLogger 结构体中的 Logger 对象的 Error 方法，记录错误级别的日志
    l.Logger.Error(args...)
}

# 定义 LogrusNestedLogger 结构体的 Debugf 方法，用于以指定格式记录调试级别的日志
func (l *LogrusNestedLogger) Debugf(format string, args ...interface{}) {
    # 调用 LogrusNestedLogger 结构体中的 Logger 对象的 Debugf 方法，以指定格式记录调试级别的日志
    l.Logger.Debugf(format, args...)
}

# 定义 LogrusNestedLogger 结构体的 Infof 方法，用于以指定格式记录信息级别的日志
func (l *LogrusNestedLogger) Infof(format string, args ...interface{}) {
    # 调用 LogrusNestedLogger 结构体中的 Logger 对象的 Infof 方法，以指定格式记录信息级别的日志
    l.Logger.Infof(format, args...)
}
# 使用格式化字符串和参数记录警告级别的日志
func (l *LogrusNestedLogger) Warnf(format string, args ...interface{}) {
    l.Logger.Warnf(format, args...)
}

# 使用格式化字符串和参数记录错误级别的日志
func (l *LogrusNestedLogger) Errorf(format string, args ...interface{}) {
    l.Logger.Errorf(format, args...)
}

# 记录调试级别的日志
func (l *LogrusNestedLogger) Debug(args ...interface{}) {
    l.Logger.Debug(args...)
}

# 记录信息级别的日志
func (l *LogrusNestedLogger) Info(args ...interface{}) {
    l.Logger.Info(args...)
}

# 记录警告级别的日志
func (l *LogrusNestedLogger) Warn(args ...interface{}) {
    l.Logger.Warn(args...)
}
# 定义 LogrusNestedLogger 结构体的 Error 方法，接收任意数量的参数
func (l *LogrusNestedLogger) Error(args ...interface{}) {
    # 调用 Logger 结构体的 Error 方法，传入参数
    l.Logger.Error(args...)
}
```