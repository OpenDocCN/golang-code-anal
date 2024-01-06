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
var log = discard.New()  // 创建一个丢弃日志的单例对象

func Set(l logger.Logger) {
	// though the application will automatically have a redaction logger, library consumers may not be doing this.
	// for this reason we additionally ensure there is a redaction logger configured for any logger passed. The
	// source of truth for redaction values is still in the internal redact package. If the passed logger is already
	// redacted, then this is a no-op.
	// 设置日志对象，确保传递的任何日志记录器都配置了红action日志记录器。红action值的真实来源仍然在内部红action包中。如果传递的日志记录器已经被红action，则这是一个无操作。
// 从存储中获取日志记录器
store := red.Get()
// 如果存储不为空，则创建一个新的经过处理的日志记录器
if store != nil {
    l = redact.New(l, store)
}
// 将日志记录器赋值给全局变量log
log = l
}

// 获取日志记录器
func Get() logger.Logger {
    return log
}

// Errorf 使用格式化的模板字符串和模板参数记录错误日志级别
func Errorf(format string, args ...interface{}) {
    log.Errorf(format, args...)
}

// Error 在错误日志级别记录给定的参数
func Error(args ...interface{}) {
    log.Error(args...)
}
// Warnf函数接受一个格式化的模板字符串和警告日志级别的模板参数。
func Warnf(format string, args ...interface{}) {
    // 使用警告日志级别记录给定的格式化字符串和参数
    log.Warnf(format, args...)
}

// Warn函数以警告日志级别记录给定的参数。
func Warn(args ...interface{}) {
    // 使用警告日志级别记录给定的参数
    log.Warn(args...)
}

// Infof函数接受一个格式化的模板字符串和信息日志级别的模板参数。
func Infof(format string, args ...interface{}) {
    // 使用信息日志级别记录给定的格式化字符串和参数
    log.Infof(format, args...)
}

// Info函数以信息日志级别记录给定的参数。
func Info(args ...interface{}) {
    // 使用信息日志级别记录给定的参数
    log.Info(args...)
}
// Debugf函数接受一个格式化的模板字符串和调试日志级别的模板参数。
func Debugf(format string, args ...interface{}) {
    // 使用log包的Debugf函数记录调试级别的日志
    log.Debugf(format, args...)
}

// Debug函数以调试日志级别记录给定的参数。
func Debug(args ...interface{}) {
    // 使用log包的Debug函数记录调试级别的日志
    log.Debug(args...)
}

// Tracef函数接受一个格式化的模板字符串和跟踪日志级别的模板参数。
func Tracef(format string, args ...interface{}) {
    // 使用log包的Tracef函数记录跟踪级别的日志
    log.Tracef(format, args...)
}

// Trace函数以跟踪日志级别记录给定的参数。
func Trace(args ...interface{}) {
    // 使用log包的Trace函数记录跟踪级别的日志
    log.Trace(args...)
}
// 使用多个键值对字段返回一个消息记录器
func WithFields(fields ...interface{}) logger.MessageLogger {
    return log.WithFields(fields...)
}

// 使用硬编码的键值对返回一个新的记录器
func Nested(fields ...interface{}) logger.Logger {
    return log.Nested(fields...)
}
```