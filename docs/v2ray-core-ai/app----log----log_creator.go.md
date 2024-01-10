# `v2ray-core\app\log\log_creator.go`

```
// +build !confonly
// 设置构建标签，表示该文件不仅仅是配置文件

package log
// 导入所需的包
import (
    "v2ray.com/core/common"
    "v2ray.com/core/common/log"
)

// 定义日志处理器创建选项结构体
type HandlerCreatorOptions struct {
    Path string
}

// 定义日志处理器创建函数类型
type HandlerCreator func(LogType, HandlerCreatorOptions) (log.Handler, error)

// 创建日志处理器映射
var (
    handlerCreatorMap = make(map[LogType]HandlerCreator)
)

// 注册日志处理器创建函数
func RegisterHandlerCreator(logType LogType, f HandlerCreator) error {
    if f == nil {
        return newError("nil HandlerCreator")
    }

    handlerCreatorMap[logType] = f
    return nil
}

// 创建日志处理器
func createHandler(logType LogType, options HandlerCreatorOptions) (log.Handler, error) {
    creator, found := handlerCreatorMap[logType]
    if !found {
        return nil, newError("unable to create log handler for ", logType)
    }
    return creator(logType, options)
}

// 初始化函数
func init() {
    // 注册控制台日志处理器创建函数
    common.Must(RegisterHandlerCreator(LogType_Console, func(lt LogType, options HandlerCreatorOptions) (log.Handler, error) {
        return log.NewLogger(log.CreateStdoutLogWriter()), nil
    }))

    // 注册文件日志处理器创建函数
    common.Must(RegisterHandlerCreator(LogType_File, func(lt LogType, options HandlerCreatorOptions) (log.Handler, error) {
        creator, err := log.CreateFileLogWriter(options.Path)
        if err != nil {
            return nil, err
        }
        return log.NewLogger(creator), nil
    }))

    // 注册空日志处理器创建函数
    common.Must(RegisterHandlerCreator(LogType_None, func(lt LogType, options HandlerCreatorOptions) (log.Handler, error) {
        return nil, nil
    }))
}
```