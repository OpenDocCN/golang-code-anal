# `v2ray-core\testing\mocks\proxy.go`

```go
// 由 MockGen 生成的代码。请勿编辑。
// 来源：v2ray.com/core/proxy (接口：Inbound,Outbound)

// 包 mocks 是一个生成的 GoMock 包。
package mocks

import (
    context "context"
    gomock "github.com/golang/mock/gomock"
    reflect "reflect"
    net "v2ray.com/core/common/net"
    routing "v2ray.com/core/features/routing"
    transport "v2ray.com/core/transport"
    internet "v2ray.com/core/transport/internet"
)

// ProxyInbound 是 Inbound 接口的模拟
type ProxyInbound struct {
    ctrl     *gomock.Controller
    recorder *ProxyInboundMockRecorder
}

// ProxyInboundMockRecorder 是 ProxyInbound 的模拟记录器
type ProxyInboundMockRecorder struct {
    mock *ProxyInbound
}

// NewProxyInbound 创建一个新的模拟实例
func NewProxyInbound(ctrl *gomock.Controller) *ProxyInbound {
    mock := &ProxyInbound{ctrl: ctrl}
    mock.recorder = &ProxyInboundMockRecorder{mock}
    return mock
}

// EXPECT 返回一个对象，允许调用者指示预期的使用
func (m *ProxyInbound) EXPECT() *ProxyInboundMockRecorder {
    return m.recorder
}

// Network 模拟基本方法
func (m *ProxyInbound) Network() []net.Network {
    m.ctrl.T.Helper()
    ret := m.ctrl.Call(m, "Network")
    ret0, _ := ret[0].([]net.Network)
    return ret0
}

// Network 指示对 Network 的预期调用
func (mr *ProxyInboundMockRecorder) Network() *gomock.Call {
    mr.mock.ctrl.T.Helper()
    return mr.mock.ctrl.RecordCallWithMethodType(mr.mock, "Network", reflect.TypeOf((*ProxyInbound)(nil).Network))
}

// Process 模拟基本方法
func (m *ProxyInbound) Process(arg0 context.Context, arg1 net.Network, arg2 internet.Connection, arg3 routing.Dispatcher) error {
    m.ctrl.T.Helper()
    ret := m.ctrl.Call(m, "Process", arg0, arg1, arg2, arg3)
    ret0, _ := ret[0].(error)
    return ret0
}

// Process 指示对 Process 的预期调用
func (mr *ProxyInboundMockRecorder) Process(arg0, arg1, arg2, arg3 interface{}) *gomock.Call {
    # 标记当前测试辅助函数
    mr.mock.ctrl.T.Helper()
    # 使用指定的方法类型记录调用信息，并返回记录结果
    return mr.mock.ctrl.RecordCallWithMethodType(mr.mock, "Process", reflect.TypeOf((*ProxyInbound)(nil).Process), arg0, arg1, arg2, arg3)
// ProxyOutbound 是 Outbound 接口的模拟
type ProxyOutbound struct {
    ctrl     *gomock.Controller  // 控制器
    recorder *ProxyOutboundMockRecorder  // 模拟记录器
}

// ProxyOutboundMockRecorder 是 ProxyOutbound 的模拟记录器
type ProxyOutboundMockRecorder struct {
    mock *ProxyOutbound
}

// NewProxyOutbound 创建一个新的模拟实例
func NewProxyOutbound(ctrl *gomock.Controller) *ProxyOutbound {
    mock := &ProxyOutbound{ctrl: ctrl}
    mock.recorder = &ProxyOutboundMockRecorder{mock}
    return mock
}

// EXPECT 返回一个对象，允许调用者指示预期的使用
func (m *ProxyOutbound) EXPECT() *ProxyOutboundMockRecorder {
    return m.recorder
}

// Process 模拟基本方法
func (m *ProxyOutbound) Process(arg0 context.Context, arg1 *transport.Link, arg2 internet.Dialer) error {
    m.ctrl.T.Helper()
    ret := m.ctrl.Call(m, "Process", arg0, arg1, arg2)
    ret0, _ := ret[0].(error)
    return ret0
}

// Process 表示对 Process 的预期调用
func (mr *ProxyOutboundMockRecorder) Process(arg0, arg1, arg2 interface{}) *gomock.Call {
    mr.mock.ctrl.T.Helper()
    return mr.mock.ctrl.RecordCallWithMethodType(mr.mock, "Process", reflect.TypeOf((*ProxyOutbound)(nil).Process), arg0, arg1, arg2)
}
```