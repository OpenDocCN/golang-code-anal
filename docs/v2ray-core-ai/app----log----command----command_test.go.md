# `v2ray-core\app\log\command\command_test.go`

```go
package command_test

import (
    "context"  // 导入上下文包
    "testing"  // 导入测试包

    "v2ray.com/core"  // 导入核心包
    "v2ray.com/core/app/dispatcher"  // 导入调度器应用包
    "v2ray.com/core/app/log"  // 导入日志应用包
    . "v2ray.com/core/app/log/command"  // 导入日志命令包，并使用点操作符简化调用
    "v2ray.com/core/app/proxyman"  // 导入代理管理应用包
    _ "v2ray.com/core/app/proxyman/inbound"  // 导入入站代理管理应用包，但不使用它
    _ "v2ray.com/core/app/proxyman/outbound"  // 导入出站代理管理应用包，但不使用它
    "v2ray.com/core/common"  // 导入通用包
    "v2ray.com/core/common/serial"  // 导入序列化包
)

func TestLoggerRestart(t *testing.T) {
    v, err := core.New(&core.Config{  // 创建核心对象，并传入配置
        App: []*serial.TypedMessage{  // 设置应用列表
            serial.ToTypedMessage(&log.Config{}),  // 将日志配置转换为类型化消息
            serial.ToTypedMessage(&dispatcher.Config{}),  // 将调度器配置转换为类型化消息
            serial.ToTypedMessage(&proxyman.InboundConfig{}),  // 将入站代理配置转换为类型化消息
            serial.ToTypedMessage(&proxyman.OutboundConfig{}),  // 将出站代理配置转换为类型化消息
        },
    })
    common.Must(err)  // 检查错误并处理
    common.Must(v.Start())  // 启动核心对象

    server := &LoggerServer{  // 创建日志服务器对象
        V: v,  // 设置日志服务器的核心对象
    }
    common.Must2(server.RestartLogger(context.Background(), &RestartLoggerRequest{}))  // 重启日志服务器，并传入上下文和重启请求
}
```