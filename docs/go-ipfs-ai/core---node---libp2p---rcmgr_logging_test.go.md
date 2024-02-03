# `kubo\core\node\libp2p\rcmgr_logging_test.go`

```go
package libp2p

import (
    "context"  // 导入上下文包，用于控制程序的执行流程
    "testing"  // 导入测试包，用于编写测试函数
    "time"     // 导入时间包，用于处理时间相关的操作

    "github.com/benbjohnson/clock"  // 导入第三方库，用于模拟时间
    "github.com/libp2p/go-libp2p/core/network"  // 导入网络核心包
    rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager"  // 导入资源管理包
    ma "github.com/multiformats/go-multiaddr"  // 导入多地址格式包
    "github.com/stretchr/testify/require"  // 导入断言包，用于测试时的断言
    "go.uber.org/zap"  // 导入日志库
    "go.uber.org/zap/zaptest/observer"  // 导入日志观察者包
)

func TestLoggingResourceManager(t *testing.T) {
    clock := clock.NewMock()  // 创建一个模拟时钟对象
    orig := rcmgr.DefaultLimits.AutoScale()  // 获取默认资源限制
    limits := orig.ToPartialLimitConfig()  // 将默认资源限制转换为部分限制配置
    limits.System.Conns = 1  // 设置系统连接数为1
    limits.System.ConnsInbound = 1  // 设置系统入站连接数为1
    limits.System.ConnsOutbound = 1  // 设置系统出站连接数为1
    limiter := rcmgr.NewFixedLimiter(limits.Build(orig))  // 创建一个固定限制器
    rm, err := rcmgr.NewResourceManager(limiter)  // 创建资源管理器
    if err != nil {
        t.Fatal(err)  // 如果出现错误，测试失败
    }

    oCore, oLogs := observer.New(zap.WarnLevel)  // 创建一个日志观察者
    oLogger := zap.New(oCore)  // 创建一个日志记录器
    lrm := &loggingResourceManager{  // 创建日志资源管理器对象
        clock:       clock,  // 设置时钟
        logger:      oLogger.Sugar(),  // 设置日志记录器
        delegate:    rm,  // 设置委托资源管理器
        logInterval: 1 * time.Second,  // 设置日志间隔为1秒
    }

    // 2 of these should result in resource limit exceeded errors and subsequent log messages
    for i := 0; i < 3; i++ {  // 循环3次
        _, _ = lrm.OpenConnection(network.DirInbound, false, ma.StringCast("/ip4/127.0.0.1/tcp/1234"))  // 打开连接
    }

    // run the logger which will write an entry for those errors
    ctx, cancel := context.WithCancel(context.Background())  // 创建一个带有取消功能的上下文
    defer cancel()  // 延迟取消上下文
    lrm.start(ctx)  // 启动日志资源管理器
    clock.Add(3 * time.Second)  // 模拟时间流逝3秒

    timer := time.NewTimer(1 * time.Second)  // 创建一个1秒的定时器
    for {
        select {
        case <-timer.C:  // 定时器触发
            t.Fatalf("expected logs never arrived")  // 测试失败，预期的日志未到达
        default:
            if oLogs.Len() == 0 {  // 如果日志长度为0
                continue  // 继续循环
            }
            require.Equal(t, "Protected from exceeding resource limits 2 times.  libp2p message: \"system: cannot reserve inbound connection: resource limit exceeded\".", oLogs.All()[0].Message)  // 断言日志内容
            return  // 返回
        }
    }
}
```