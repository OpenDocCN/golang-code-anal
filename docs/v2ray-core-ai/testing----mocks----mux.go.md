# `v2ray-core\testing\mocks\mux.go`

```
// 由 MockGen 生成的代码。请勿编辑。
// 来源：v2ray.com/core/common/mux (接口：ClientWorkerFactory)

// 包 mocks 是一个生成的 GoMock 包。
package mocks

import (
    gomock "github.com/golang/mock/gomock"  // 导入 gomock 包
    reflect "reflect"  // 导入 reflect 包
    mux "v2ray.com/core/common/mux"  // 导入 mux 包
)

// MuxClientWorkerFactory 是 ClientWorkerFactory 接口的模拟
type MuxClientWorkerFactory struct {
    ctrl     *gomock.Controller  // gomock 控制器
    recorder *MuxClientWorkerFactoryMockRecorder  // MuxClientWorkerFactory 的模拟记录器
}

// MuxClientWorkerFactoryMockRecorder 是 MuxClientWorkerFactory 的模拟记录器
type MuxClientWorkerFactoryMockRecorder struct {
    mock *MuxClientWorkerFactory  // MuxClientWorkerFactory 的模拟
}

// NewMuxClientWorkerFactory 创建一个新的模拟实例
func NewMuxClientWorkerFactory(ctrl *gomock.Controller) *MuxClientWorkerFactory {
    mock := &MuxClientWorkerFactory{ctrl: ctrl}  // 创建 MuxClientWorkerFactory 实例
    mock.recorder = &MuxClientWorkerFactoryMockRecorder{mock}  // 创建 MuxClientWorkerFactoryMockRecorder 实例
    return mock
}

// EXPECT 返回一个对象，允许调用者指示预期的使用
func (m *MuxClientWorkerFactory) EXPECT() *MuxClientWorkerFactoryMockRecorder {
    return m.recorder
}

// Create 模拟基本方法
func (m *MuxClientWorkerFactory) Create() (*mux.ClientWorker, error) {
    m.ctrl.T.Helper()  // 标记当前测试辅助函数
    ret := m.ctrl.Call(m, "Create")  // 调用控制器的 Call 方法
    ret0, _ := ret[0].(*mux.ClientWorker)  // 断言返回值的类型
    ret1, _ := ret[1].(error)  // 断言返回值的类型
    return ret0, ret1  // 返回模拟的结果
}

// Create 表示对 Create 的预期调用
func (mr *MuxClientWorkerFactoryMockRecorder) Create() *gomock.Call {
    mr.mock.ctrl.T.Helper()  // 标记当前测试辅助函数
    return mr.mock.ctrl.RecordCallWithMethodType(mr.mock, "Create", reflect.TypeOf((*MuxClientWorkerFactory)(nil).Create))  // 记录对 Create 方法的调用
}
```