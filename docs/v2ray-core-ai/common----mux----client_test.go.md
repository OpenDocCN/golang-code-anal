# `v2ray-core\common\mux\client_test.go`

```go
package mux_test

import (
    "context"  // 引入上下文包
    "testing"  // 引入测试包
    "time"     // 引入时间包

    "github.com/golang/mock/gomock"  // 引入 mock 包
    "v2ray.com/core/common"          // 引入 common 包
    "v2ray.com/core/common/errors"   // 引入 errors 包
    "v2ray.com/core/common/mux"      // 引入 mux 包
    "v2ray.com/core/common/net"      // 引入 net 包
    "v2ray.com/core/common/session"  // 引入 session 包
    "v2ray.com/core/testing/mocks"   // 引入 mocks 包
    "v2ray.com/core/transport"       // 引入 transport 包
    "v2ray.com/core/transport/pipe"  // 引入 pipe 包
)

func TestIncrementalPickerFailure(t *testing.T) {
    mockCtl := gomock.NewController(t)  // 创建 mock 控制器
    defer mockCtl.Finish()  // 在函数返回时关闭 mock 控制器

    mockWorkerFactory := mocks.NewMuxClientWorkerFactory(mockCtl)  // 创建 mock 工厂
    mockWorkerFactory.EXPECT().Create().Return(nil, errors.New("test"))  // 期望 mock 工厂创建并返回错误

    picker := mux.IncrementalWorkerPicker{  // 创建增量工作者选择器
        Factory: mockWorkerFactory,  // 使用 mock 工厂
    }

    _, err := picker.PickAvailable()  // 选择可用工作者
    if err == nil {  // 如果没有错误
        t.Error("expected error, but nil")  // 输出错误信息
    }
}

func TestClientWorkerEOF(t *testing.T) {
    reader, writer := pipe.New(pipe.WithoutSizeLimit())  // 创建管道读写器
    common.Must(writer.Close())  // 确保写入器关闭

    worker, err := mux.NewClientWorker(transport.Link{Reader: reader, Writer: writer}, mux.ClientStrategy{})  // 创建客户端工作者
    common.Must(err)  // 确保没有错误发生

    time.Sleep(time.Millisecond * 500)  // 休眠 500 毫秒

    f := worker.Dispatch(context.Background(), nil)  // 在后台上下文中分发工作者
    if f {  // 如果分发成功
        t.Error("expected failed dispatching, but actually not")  // 输出错误信息
    }
}

func TestClientWorkerClose(t *testing.T) {
    mockCtl := gomock.NewController(t)  // 创建 mock 控制器
    defer mockCtl.Finish()  // 在函数返回时关闭 mock 控制器

    r1, w1 := pipe.New(pipe.WithoutSizeLimit())  // 创建管道读写器
    worker1, err := mux.NewClientWorker(transport.Link{  // 创建客户端工作者
        Reader: r1,  // 使用读取器 r1
        Writer: w1,  // 使用写入器 w1
    }, mux.ClientStrategy{  // 使用客户端策略
        MaxConcurrency: 4,  // 最大并发数为 4
        MaxConnection:  4,  // 最大连接数为 4
    })
    common.Must(err)  // 确保没有错误发生

    r2, w2 := pipe.New(pipe.WithoutSizeLimit())  // 创建管道读写器
    worker2, err := mux.NewClientWorker(transport.Link{  // 创建客户端工作者
        Reader: r2,  // 使用读取器 r2
        Writer: w2,  // 使用写入器 w2
    }, mux.ClientStrategy{  // 使用客户端策略
        MaxConcurrency: 4,  // 最大并发数为 4
        MaxConnection:  4,  // 最大连接数为 4
    })
    common.Must(err)  // 确保没有错误发生

    factory := mocks.NewMuxClientWorkerFactory(mockCtl)  // 创建 mux 客户端工作者工厂
    # 按顺序设置期望的调用，分别返回 worker1 和 worker2
    gomock.InOrder(
        factory.EXPECT().Create().Return(worker1, nil),
        factory.EXPECT().Create().Return(worker2, nil),
    )

    # 创建一个 IncrementalWorkerPicker 对象，使用给定的工厂
    picker := &mux.IncrementalWorkerPicker{
        Factory: factory,
    }
    # 创建一个 ClientManager 对象，使用上面创建的 picker
    manager := &mux.ClientManager{
        Picker: picker,
    }

    # 创建一个无大小限制的管道
    tr1, tw1 := pipe.New(pipe.WithoutSizeLimit())
    # 创建一个带有目标地址的上下文
    ctx1 := session.ContextWithOutbound(context.Background(), &session.Outbound{
        Target: net.TCPDestination(net.DomainAddress("www.v2ray.com"), 80),
    })
    # 使用 manager 分发传输链接
    common.Must(manager.Dispatch(ctx1, &transport.Link{
        Reader: tr1,
        Writer: tw1,
    }))
    # 延迟关闭写入端
    defer tw1.Close()

    # 强制关闭 w1
    common.Must(w1.Close())

    # 等待 500 毫秒
    time.Sleep(time.Millisecond * 500)
    # 如果 worker1 没有关闭，则输出错误信息
    if !worker1.Closed() {
        t.Error("worker1 is not finished")
    }

    # 创建一个无大小限制的管道
    tr2, tw2 := pipe.New(pipe.WithoutSizeLimit())
    # 创建一个带有目标地址的上下文
    ctx2 := session.ContextWithOutbound(context.Background(), &session.Outbound{
        Target: net.TCPDestination(net.DomainAddress("www.v2ray.com"), 80),
    })
    # 使用 manager 分发传输链接
    common.Must(manager.Dispatch(ctx2, &transport.Link{
        Reader: tr2,
        Writer: tw2,
    }))
    # 延迟关闭写入端
    defer tw2.Close()

    # 强制关闭 w2
    common.Must(w2.Close())
# 闭合前面的函数定义
```