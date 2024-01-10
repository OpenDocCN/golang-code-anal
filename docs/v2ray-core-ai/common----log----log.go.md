# `v2ray-core\common\log\log.go`

```
// 定义日志包，导入路径为 "v2ray.com/core/common/log"
package log // import "v2ray.com/core/common/log"

import (
    "sync" // 导入同步包

    "v2ray.com/core/common/serial" // 导入序列化包
)

// Message 是所有日志消息的接口
type Message interface {
    String() string
}

// Handler 是日志处理程序的接口
type Handler interface {
    Handle(msg Message)
}

// GeneralMessage 是一个通用的日志消息，可以包含各种内容
type GeneralMessage struct {
    Severity Severity // 严重程度
    Content  interface{} // 内容
}

// String 实现了 Message 接口
func (m *GeneralMessage) String() string {
    return serial.Concat("[", m.Severity, "] ", m.Content) // 返回拼接后的字符串
}

// Record 将消息写入日志流
func Record(msg Message) {
    logHandler.Handle(msg) // 调用日志处理程序的 Handle 方法
}

var (
    logHandler syncHandler // 日志处理程序
)

// RegisterHandler 注册一个新的处理程序作为当前的日志处理程序。之前注册的处理程序将被丢弃。
func RegisterHandler(handler Handler) {
    if handler == nil {
        panic("Log handler is nil") // 如果处理程序为空，抛出异常
    }
    logHandler.Set(handler) // 设置日志处理程序
}

type syncHandler struct {
    sync.RWMutex // 读写锁
    Handler      // 处理程序
}

func (h *syncHandler) Handle(msg Message) {
    h.RLock() // 读锁
    defer h.RUnlock() // 解锁

    if h.Handler != nil {
        h.Handler.Handle(msg) // 如果处理程序不为空，调用其 Handle 方法
    }
}

func (h *syncHandler) Set(handler Handler) {
    h.Lock() // 写锁
    defer h.Unlock() // 解锁

    h.Handler = handler // 设置处理程序
}
```