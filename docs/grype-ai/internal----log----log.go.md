# `grype\internal\log\log.go`

```
/*
Package log contains the singleton object and helper functions for facilitating logging within the syft library.
*/
package log

import (
    "github.com/anchore/go-logger"  // 导入日志库
    "github.com/anchore/go-logger/adapter/discard"  // 导入日志适配器，用于丢弃日志
    "github.com/anchore/go-logger/adapter/redact"  // 导入日志适配器，用于日志内容的红action
    red "github.com/anchore/grype/internal/redact"  // 导入红action包
)

// log is the singleton used to facilitate logging internally within
var log = discard.New()  // 创建一个日志对象并初始化为丢弃日志的适配器

func Set(l logger.Logger) {
    // though the application will automatically have a redaction logger, library consumers may not be doing this.
    // for this reason we additionally ensure there is a redaction logger configured for any logger passed. The
    // source of truth for redaction values is still in the internal redact package. If the passed logger is already
    // redacted, then this is a no-op.
    store := red.Get()  // 获取红action存储
    if store != nil {  // 如果存储不为空
        l = redact.New(l, store)  // 创建一个新的红action日志对象
    }
    log = l  // 将传入的日志对象赋值给全局日志对象
}

func Get() logger.Logger {
    return log  // 返回全局日志对象
}

// Errorf takes a formatted template string and template arguments for the error logging level.
func Errorf(format string, args ...interface{}) {
    log.Errorf(format, args...)  // 使用错误日志级别记录格式化的字符串和参数
}

// Error logs the given arguments at the error logging level.
func Error(args ...interface{}) {
    log.Error(args...)  // 使用错误日志级别记录给定的参数
}

// Warnf takes a formatted template string and template arguments for the warning logging level.
func Warnf(format string, args ...interface{}) {
    log.Warnf(format, args...)  // 使用警告日志级别记录格式化的字符串和参数
}

// Warn logs the given arguments at the warning logging level.
func Warn(args ...interface{}) {
    log.Warn(args...)  // 使用警告日志级别记录给定的参数
}

// Infof takes a formatted template string and template arguments for the info logging level.
func Infof(format string, args ...interface{}) {
    log.Infof(format, args...)  // 使用信息日志级别记录格式化的字符串和参数
}

// Info logs the given arguments at the info logging level.
func Info(args ...interface{}) {
    log.Info(args...)  // 使用信息日志级别记录给定的参数
}

// Debugf takes a formatted template string and template arguments for the debug logging level.
// 使用给定的格式字符串和参数，在调试日志级别记录日志
func Debugf(format string, args ...interface{}) {
    log.Debugf(format, args...)
}

// 在调试日志级别记录给定的参数
func Debug(args ...interface{}) {
    log.Debug(args...)
}

// 使用格式化的模板字符串和模板参数，在跟踪日志级别记录日志
func Tracef(format string, args ...interface{}) {
    log.Tracef(format, args...)
}

// 在跟踪日志级别记录给定的参数
func Trace(args ...interface{}) {
    log.Trace(args...)
}

// 返回一个带有多个键值字段的消息记录器
func WithFields(fields ...interface{}) logger.MessageLogger {
    return log.WithFields(fields...)
}

// 返回一个带有硬编码的键值对的新记录器
func Nested(fields ...interface{}) logger.Logger {
    return log.Nested(fields...)
}
```