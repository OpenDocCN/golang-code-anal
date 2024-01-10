# `v2ray-core\testing\mocks\log.go`

```
// 由 MockGen 生成的代码。请勿编辑。
// 来源：v2ray.com/core/common/log（接口：Handler）

// 包 mocks 是一个生成的 GoMock 包。
package mocks

import (
    gomock "github.com/golang/mock/gomock"
    reflect "reflect"
    log "v2ray.com/core/common/log"
)

// LogHandler 是 Handler 接口的模拟
type LogHandler struct {
    ctrl     *gomock.Controller
    recorder *LogHandlerMockRecorder
}

// LogHandlerMockRecorder 是 LogHandler 的模拟记录器
type LogHandlerMockRecorder struct {
    mock *LogHandler
}

// NewLogHandler 创建一个新的模拟实例
func NewLogHandler(ctrl *gomock.Controller) *LogHandler {
    mock := &LogHandler{ctrl: ctrl}
    mock.recorder = &LogHandlerMockRecorder{mock}
    return mock
}

// EXPECT 返回一个对象，允许调用者指示预期的使用
func (m *LogHandler) EXPECT() *LogHandlerMockRecorder {
    return m.recorder
}

// Handle 模拟基本方法
func (m *LogHandler) Handle(arg0 log.Message) {
    m.ctrl.T.Helper()
    m.ctrl.Call(m, "Handle", arg0)
}

// Handle 表示对 Handle 的预期调用
func (mr *LogHandlerMockRecorder) Handle(arg0 interface{}) *gomock.Call {
    mr.mock.ctrl.T.Helper()
    return mr.mock.ctrl.RecordCallWithMethodType(mr.mock, "Handle", reflect.TypeOf((*LogHandler)(nil).Handle), arg0)
}
```