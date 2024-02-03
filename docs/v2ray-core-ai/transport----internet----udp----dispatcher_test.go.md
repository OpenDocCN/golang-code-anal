# `v2ray-core\transport\internet\udp\dispatcher_test.go`

```go
package udp_test

import (
    "context"  // 引入上下文包
    "sync/atomic"  // 引入原子操作包
    "testing"  // 引入测试包
    "time"  // 引入时间包

    "v2ray.com/core/common"  // 引入通用包
    "v2ray.com/core/common/buf"  // 引入缓冲区包
    "v2ray.com/core/common/net"  // 引入网络包
    "v2ray.com/core/common/protocol/udp"  // 引入 UDP 协议包
    "v2ray.com/core/features/routing"  // 引入路由功能包
    "v2ray.com/core/transport"  // 引入传输包
    . "v2ray.com/core/transport/internet/udp"  // 引入 UDP 传输包
    "v2ray.com/core/transport/pipe"  // 引入管道包
)

type TestDispatcher struct {
    OnDispatch func(ctx context.Context, dest net.Destination) (*transport.Link, error)  // 定义测试分发器结构体
}

func (d *TestDispatcher) Dispatch(ctx context.Context, dest net.Destination) (*transport.Link, error) {  // 分发器的分发方法
    return d.OnDispatch(ctx, dest)  // 返回分发结果
}

func (d *TestDispatcher) Start() error {  // 分发器的启动方法
    return nil  // 返回空值
}

func (d *TestDispatcher) Close() error {  // 分发器的关闭方法
    return nil  // 返回空值
}

func (*TestDispatcher) Type() interface{} {  // 分发器的类型方法
    return routing.DispatcherType()  // 返回路由分发器类型
}

func TestSameDestinationDispatching(t *testing.T) {  // 测试相同目的地分发
    ctx, cancel := context.WithCancel(context.Background())  // 创建上下文和取消函数
    uplinkReader, uplinkWriter := pipe.New(pipe.WithSizeLimit(1024))  // 创建上行读写管道
    downlinkReader, downlinkWriter := pipe.New(pipe.WithSizeLimit(1024))  // 创建下行读写管道

    go func() {  // 启动协程
        for {  // 无限循环
            data, err := uplinkReader.ReadMultiBuffer()  // 从上行读取多缓冲数据
            if err != nil {  // 如果有错误
                break  // 退出循环
            }
            err = downlinkWriter.WriteMultiBuffer(data)  // 向下行写入多缓冲数据
            common.Must(err)  // 必须处理错误
        }
    }()

    var count uint32  // 定义计数变量
    td := &TestDispatcher{  // 创建测试分发器对象
        OnDispatch: func(ctx context.Context, dest net.Destination) (*transport.Link, error) {  // 分发器的分发方法
            atomic.AddUint32(&count, 1)  // 原子操作增加计数
            return &transport.Link{Reader: downlinkReader, Writer: uplinkWriter}, nil  // 返回读写链接
        },
    }
    dest := net.UDPDestination(net.LocalHostIP, 53)  // 定义 UDP 目的地

    b := buf.New()  // 创建新的缓冲区
    b.WriteString("abcd")  // 向缓冲区写入字符串

    var msgCount uint32  // 定义消息计数变量
    dispatcher := NewDispatcher(td, func(ctx context.Context, packet *udp.Packet) {  // 创建新的分发器
        atomic.AddUint32(&msgCount, 1)  // 原子操作增加消息计数
    })

    dispatcher.Dispatch(ctx, dest, b)  // 分发消息
    for i := 0; i < 5; i++ {  // 循环5次
        dispatcher.Dispatch(ctx, dest, b)  // 分发消息
    }
}
    # 暂停执行当前 goroutine 一秒钟
    time.Sleep(time.Second)
    # 取消操作

    # 如果 count 不等于 1，则输出错误信息
    if count != 1 {
        t.Error("count: ", count)
    }
    # 使用原子操作加载 msgCount 的值，如果不等于 6，则输出错误信息
    if v := atomic.LoadUint32(&msgCount); v != 6 {
        t.Error("msgCount: ", v)
    }
# 闭合前面的函数定义
```