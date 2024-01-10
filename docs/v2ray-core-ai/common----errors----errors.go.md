# `v2ray-core\common\errors\errors.go`

```
// Package errors is a drop-in replacement for Golang lib 'errors'.
package errors // import "v2ray.com/core/common/errors"

import (
    "os" // 导入操作系统相关的包
    "reflect" // 导入反射相关的包
    "strings" // 导入字符串处理相关的包

    "v2ray.com/core/common/log" // 导入日志相关的包
    "v2ray.com/core/common/serial" // 导入序列化相关的包
)

type hasInnerError interface {
    // Inner returns the underlying error of this one.
    Inner() error // 定义接口方法，返回当前错误的底层错误
}

type hasSeverity interface {
    Severity() log.Severity // 定义接口方法，返回日志的严重程度
}

// Error is an error object with underlying error.
type Error struct {
    pathObj  interface{} // 错误路径对象
    prefix   []interface{} // 错误前缀
    message  []interface{} // 错误消息
    inner    error // 内部错误
    severity log.Severity // 错误严重程度
}

func (err *Error) WithPathObj(obj interface{}) *Error {
    err.pathObj = obj
    return err
}

func (err *Error) pkgPath() string {
    if err.pathObj == nil {
        return ""
    }
    return reflect.TypeOf(err.pathObj).PkgPath() // 返回路径对象的包路径
}

// Error implements error.Error().
func (err *Error) Error() string {
    builder := strings.Builder{} // 创建字符串构建器
    for _, prefix := range err.prefix {
        builder.WriteByte('[')
        builder.WriteString(serial.ToString(prefix)) // 将前缀转换为字符串并添加到构建器中
        builder.WriteString("] ")
    }

    path := err.pkgPath() // 获取路径对象的包路径
    if len(path) > 0 {
        builder.WriteString(path) // 将包路径添加到构建器中
        builder.WriteString(": ")
    }

    msg := serial.Concat(err.message...) // 将错误消息连接为字符串
    builder.WriteString(msg) // 将错误消息添加到构建器中

    if err.inner != nil {
        builder.WriteString(" > ")
        builder.WriteString(err.inner.Error()) // 将内部错误消息添加到构建器中
    }

    return builder.String() // 返回构建器中的字符串
}

// Inner implements hasInnerError.Inner()
func (err *Error) Inner() error {
    if err.inner == nil {
        return nil
    }
    return err.inner // 返回内部错误
}

func (err *Error) Base(e error) *Error {
    err.inner = e // 设置内部错误
    return err
}

func (err *Error) atSeverity(s log.Severity) *Error {
    err.severity = s // 设置错误严重程度
    return err
}

func (err *Error) Severity() log.Severity {
    if err.inner == nil {
        return err.severity // 返回错误严重程度
    }
    # 检查错误对象是否实现了hasSeverity接口，并将其赋值给s，同时判断是否成功
    if s, ok := err.inner.(hasSeverity); ok:
        # 获取实现hasSeverity接口的对象的严重程度
        as := s.Severity()
        # 如果实现hasSeverity接口的对象的严重程度小于当前错误对象的严重程度，则返回实现hasSeverity接口的对象的严重程度
        if as < err.severity:
            return as
    # 返回当前错误对象的严重程度
    return err.severity
// AtDebug将错误的严重性设置为调试级别。
func (err *Error) AtDebug() *Error {
    return err.atSeverity(log.Severity_Debug)
}

// AtInfo将错误的严重性设置为信息级别。
func (err *Error) AtInfo() *Error {
    return err.atSeverity(log.Severity_Info)
}

// AtWarning将错误的严重性设置为警告级别。
func (err *Error) AtWarning() *Error {
    return err.atSeverity(log.Severity_Warning)
}

// AtError将错误的严重性设置为错误级别。
func (err *Error) AtError() *Error {
    return err.atSeverity(log.Severity_Error)
}

// String返回此错误的字符串表示形式。
func (err *Error) String() string {
    return err.Error()
}

// WriteToLog将当前错误写入日志。
func (err *Error) WriteToLog(opts ...ExportOption) {
    var holder ExportOptionHolder

    for _, opt := range opts {
        opt(&holder)
    }

    if holder.SessionID > 0 {
        err.prefix = append(err.prefix, holder.SessionID)
    }

    log.Record(&log.GeneralMessage{
        Severity: GetSeverity(err),
        Content:  err,
    })
}

type ExportOptionHolder struct {
    SessionID uint32
}

type ExportOption func(*ExportOptionHolder)

// New返回一个新的错误对象，其中的消息由给定的参数形成。
func New(msg ...interface{}) *Error {
    return &Error{
        message:  msg,
        severity: log.Severity_Info,
    }
}

// Cause返回此错误的根本原因。
func Cause(err error) error {
    if err == nil {
        return nil
    }
L:
    for {
        switch inner := err.(type) {
        case hasInnerError:
            if inner.Inner() == nil {
                break L
            }
            err = inner.Inner()
        case *os.PathError:
            if inner.Err == nil {
                break L
            }
            err = inner.Err
        case *os.SyscallError:
            if inner.Err == nil {
                break L
            }
            err = inner.Err
        default:
            break L
        }
    }
    return err
}
// GetSeverity函数返回错误的实际严重程度，包括内部错误。
func GetSeverity(err error) log.Severity {
    // 检查错误是否实现了hasSeverity接口
    if s, ok := err.(hasSeverity); ok {
        // 如果是，返回其严重程度
        return s.Severity()
    }
    // 如果不是，返回Info级别的严重程度
    return log.Severity_Info
}
```