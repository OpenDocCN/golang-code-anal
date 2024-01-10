# `kubo\core\coreiface\tests\pubsub.go`

```
package tests

import (
    "context"  // 上下文包，用于管理请求的生命周期
    "testing"  // 测试包，用于编写测试函数
    "time"     // 时间包，用于处理时间相关操作

    iface "github.com/ipfs/kubo/core/coreiface"  // 导入自定义接口包
    "github.com/ipfs/kubo/core/coreiface/options"  // 导入自定义接口选项包
)

func (tp *TestSuite) TestPubSub(t *testing.T) {
    tp.hasApi(t, func(api iface.CoreAPI) error {  // 调用自定义函数，检查是否存在指定的 API
        if api.PubSub() == nil {  // 如果 PubSub API 不存在
            return errAPINotImplemented  // 返回未实现错误
        }
        return nil  // 返回空值
    })

    t.Run("TestBasicPubSub", tp.TestBasicPubSub)  // 运行基本 PubSub 测试
}

func (tp *TestSuite) TestBasicPubSub(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())  // 创建带有取消函数的上下文
    defer cancel()  // 延迟调用取消函数

    apis, err := tp.MakeAPISwarm(t, ctx, 2)  // 创建 API 群集
    if err != nil {  // 如果出现错误
        t.Fatal(err)  // 终止测试并输出错误信息
    }

    sub, err := apis[0].PubSub().Subscribe(ctx, "testch")  // 使用 API 订阅指定主题
    if err != nil {  // 如果出现错误
        t.Fatal(err)  // 终止测试并输出错误信息
    }

    done := make(chan struct{})  // 创建一个结构体通道
    go func() {  // 启动协程
        defer close(done)  // 延迟关闭通道

        ticker := time.NewTicker(100 * time.Millisecond)  // 创建定时器
        defer ticker.Stop()  // 延迟停止定时器

        for {
            err := apis[1].PubSub().Publish(ctx, "testch", []byte("hello world"))  // 使用 API 发布消息到指定主题
            switch err {  // 开始错误判断
            case nil:  // 如果没有错误
            case context.Canceled:  // 如果上下文已取消
                return  // 返回
            default:  // 其他情况
                t.Error(err)  // 输出错误信息
                cancel()  // 取消上下文
                return  // 返回
            }
            select {  // 开始选择
            case <-ticker.C:  // 如果定时器触发
            case <-ctx.Done():  // 如果上下文已完成
                return  // 返回
            }
        }
    }()

    // 等待发送者完成后再返回
    // 否则，我们可能会得到随机错误，因为发布会失败。
    defer func() {  // 延迟函数
        cancel()  // 取消上下文
        <-done  // 从通道中接收数据
    }()

    m, err := sub.Next(ctx)  // 获取下一条消息
    if err != nil {  // 如果出现错误
        t.Fatal(err)  // 终止测试并输出错误信息
    }

    if string(m.Data()) != "hello world" {  // 如果消息数据不是 "hello world"
        t.Errorf("got invalid data: %s", string(m.Data()))  // 输出无效数据错误信息
    }

    self1, err := apis[1].Key().Self(ctx)  // 获取节点自身信息
    if err != nil {  // 如果出现错误
        t.Fatal(err)  // 终止测试并输出错误信息
    }

    if m.From() != self1.ID() {  // 如果消息发送者不是节点自身
        t.Errorf("m.From didn't match")  // 输出发送者不匹配错误信息
    }

    peers, err := apis[1].PubSub().Peers(ctx, options.PubSub.Topic("testch"))  // 获取指定主题的对等节点
    # 如果发生错误，则输出错误信息并终止测试
    if err != nil:
        t.Fatal(err)

    # 如果对等节点数量不等于1，则输出错误信息并终止测试
    if len(peers) != 1:
        t.Fatalf("got incorrect number of peers: %d", len(peers))

    # 获取第一个 API 的自身信息
    self0, err := apis[0].Key().Self(ctx)
    if err != nil:
        t.Fatal(err)

    # 如果第一个对等节点的 ID 不等于自身 ID，则输出错误信息
    if peers[0] != self0.ID():
        t.Errorf("peer didn't match")

    # 获取第二个 API 的指定主题的对等节点列表
    peers, err = apis[1].PubSub().Peers(ctx, options.PubSub.Topic("nottestch"))
    if err != nil:
        t.Fatal(err)

    # 如果对等节点数量不等于0，则输出错误信息并终止测试
    if len(peers) != 0:
        t.Fatalf("got incorrect number of peers: %d", len(peers))

    # 获取第一个 API 的所有主题列表
    topics, err = apis[0].PubSub().Ls(ctx)
    if err != nil:
        t.Fatal(err)

    # 如果主题数量不等于1，则输出错误信息并终止测试
    if len(topics) != 1:
        t.Fatalf("got incorrect number of topics: %d", len(peers))

    # 如果第一个主题不等于"testch"，则输出错误信息
    if topics[0] != "testch":
        t.Errorf("topic didn't match")

    # 获取第二个 API 的所有主题列表
    topics, err = apis[1].PubSub().Ls(ctx)
    if err != nil:
        t.Fatal(err)

    # 如果主题数量不等于0，则输出错误信息并终止测试
    if len(topics) != 0:
        t.Fatalf("got incorrect number of topics: %d", len(peers))
# 闭合前面的函数定义
```