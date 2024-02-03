# `trojan-go\api\api.go`

```go
package api

import (
    "context"  // 导入 context 包，用于处理上下文
    "github.com/p4gefau1t/trojan-go/log"  // 导入日志包
    "github.com/p4gefau1t/trojan-go/statistic"  // 导入统计包
)

type Handler func(ctx context.Context, auth statistic.Authenticator) error  // 定义 Handler 类型，接受上下文和认证器，返回错误

var handlers = make(map[string]Handler)  // 创建一个映射，用于存储处理器

func RegisterHandler(name string, handler Handler) {  // 注册处理器函数，接受处理器名称和处理器
    handlers[name] = handler  // 将处理器存储到映射中
}

func RunService(ctx context.Context, name string, auth statistic.Authenticator) error {  // 运行服务函数，接受上下文、处理器名称和认证器，返回错误
    if h, ok := handlers[name]; ok {  // 检查处理器是否存在
        log.Debug("api handler found", name)  // 记录日志，表示找到处理器
        return h(ctx, auth)  // 调用处理器处理请求
    }
    log.Debug("api handler not found", name)  // 记录日志，表示未找到处理器
    return nil  // 返回空值
}
```