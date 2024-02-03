# `v2ray-core\infra\conf\log.go`

```go
package conf

import (
    "strings"  // 导入 strings 包，用于处理字符串
    "v2ray.com/core/app/log"  // 导入日志相关的包
    clog "v2ray.com/core/common/log"  // 导入通用日志包
)

func DefaultLogConfig() *log.Config {
    return &log.Config{  // 返回一个默认的日志配置对象
        AccessLogType: log.LogType_None,  // 访问日志类型为 None
        ErrorLogType:  log.LogType_Console,  // 错误日志类型为 Console
        ErrorLogLevel: clog.Severity_Warning,  // 错误日志级别为 Warning
    }
}

type LogConfig struct {
    AccessLog string `json:"access"`  // 访问日志路径
    ErrorLog  string `json:"error"`  // 错误日志路径
    LogLevel  string `json:"loglevel"`  // 日志级别
}

func (v *LogConfig) Build() *log.Config {
    if v == nil {  // 如果日志配置对象为空
        return nil  // 返回空
    }
    config := &log.Config{  // 创建一个日志配置对象
        ErrorLogType:  log.LogType_Console,  // 错误日志类型为 Console
        AccessLogType: log.LogType_Console,  // 访问日志类型为 Console
    }

    if v.AccessLog == "none" {  // 如果访问日志路径为 none
        config.AccessLogType = log.LogType_None  // 访问日志类型为 None
    } else if len(v.AccessLog) > 0 {  // 如果访问日志路径不为空
        config.AccessLogPath = v.AccessLog  // 设置访问日志路径
        config.AccessLogType = log.LogType_File  // 访问日志类型为 File
    }
    if v.ErrorLog == "none" {  // 如果错误日志路径为 none
        config.ErrorLogType = log.LogType_None  // 错误日志类型为 None
    } else if len(v.ErrorLog) > 0 {  // 如果错误日志路径不为空
        config.ErrorLogPath = v.ErrorLog  // 设置错误日志路径
        config.ErrorLogType = log.LogType_File  // 错误日志类型为 File
    }

    level := strings.ToLower(v.LogLevel)  // 将日志级别转换为小写
    switch level {  // 根据日志级别进行判断
    case "debug":
        config.ErrorLogLevel = clog.Severity_Debug  // 错误日志级别为 Debug
    case "info":
        config.ErrorLogLevel = clog.Severity_Info  // 错误日志级别为 Info
    case "error":
        config.ErrorLogLevel = clog.Severity_Error  // 错误日志级别为 Error
    case "none":
        config.ErrorLogType = log.LogType_None  // 错误日志类型为 None
        config.AccessLogType = log.LogType_None  // 访问日志类型为 None
    default:
        config.ErrorLogLevel = clog.Severity_Warning  // 默认错误日志级别为 Warning
    }
    return config  // 返回日志配置对象
}
```