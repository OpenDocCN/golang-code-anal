# `trojan-go\option\option.go`

```
package option

import "github.com/p4gefau1t/trojan-go/common"

// 定义处理器接口
type Handler interface {
    Name() string   // 返回处理器名称
    Handle() error  // 处理器处理逻辑
    Priority() int  // 返回处理器优先级
}

// 存储处理器的映射
var handlers = make(map[string]Handler)

// 注册处理器
func RegisterHandler(h Handler) {
    handlers[h.Name()] = h  // 将处理器添加到映射中
}

// 弹出优先级最高的处理器
func PopOptionHandler() (Handler, error) {
    var maxHandler Handler = nil  // 初始化最高优先级处理器为nil
    for _, h := range handlers {  // 遍历所有处理器
        if maxHandler == nil || maxHandler.Priority() < h.Priority() {  // 如果当前处理器优先级高于最高优先级处理器
            maxHandler = h  // 更新最高优先级处理器
        }
    }
    if maxHandler == nil {  // 如果最高优先级处理器仍为nil
        return nil, common.NewError("no option left")  // 返回错误
    }
    delete(handlers, maxHandler.Name())  // 从映射中删除最高优先级处理器
    return maxHandler, nil  // 返回最高优先级处理器
}
```