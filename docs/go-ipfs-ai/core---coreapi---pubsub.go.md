# `kubo\core\coreapi\pubsub.go`

```go
package coreapi

import (
    "context"  // 导入上下文包，用于处理请求的上下文信息
    "errors"   // 导入错误包，用于处理错误信息

    coreiface "github.com/ipfs/kubo/core/coreiface"  // 导入核心接口包
    caopts "github.com/ipfs/kubo/core/coreiface/options"  // 导入核心接口选项包
    "github.com/ipfs/kubo/tracing"  // 导入追踪包
    pubsub "github.com/libp2p/go-libp2p-pubsub"  // 导入发布订阅包
    peer "github.com/libp2p/go-libp2p/core/peer"  // 导入对等节点包
    routing "github.com/libp2p/go-libp2p/core/routing"  // 导入路由包
    "go.opentelemetry.io/otel/attribute"  // 导入属性包
    "go.opentelemetry.io/otel/trace"  // 导入追踪包
)

type PubSubAPI CoreAPI  // 定义 PubSubAPI 结构体类型为 CoreAPI

type pubSubSubscription struct {  // 定义 pubSubSubscription 结构体
    subscription *pubsub.Subscription  // 包含一个 pubsub.Subscription 类型的指针字段
}

type pubSubMessage struct {  // 定义 pubSubMessage 结构体
    msg *pubsub.Message  // 包含一个 pubsub.Message 类型的指针字段
}

func (api *PubSubAPI) Ls(ctx context.Context) ([]string, error) {  // 定义 PubSubAPI 结构体的 Ls 方法，接收上下文参数，返回字符串切片和错误
    _, span := tracing.Span(ctx, "CoreAPI.PubSubAPI", "Ls")  // 调用追踪包的 Span 方法，记录追踪信息
    defer span.End()  // 在函数返回时结束追踪

    _, err := api.checkNode()  // 调用 PubSubAPI 结构体的 checkNode 方法，检查节点状态
    if err != nil {  // 如果出现错误
        return nil, err  // 返回空和错误
    }

    return api.pubSub.GetTopics(), nil  // 返回 PubSubAPI 结构体的 pubSub 对象的主题列表和空
}

func (api *PubSubAPI) Peers(ctx context.Context, opts ...caopts.PubSubPeersOption) ([]peer.ID, error) {  // 定义 PubSubAPI 结构体的 Peers 方法，接收上下文参数和选项参数，返回对等节点 ID 切片和错误
    _, span := tracing.Span(ctx, "CoreAPI.PubSubAPI", "Peers")  // 调用追踪包的 Span 方法，记录追踪信息
    defer span.End()  // 在函数返回时结束追踪

    _, err := api.checkNode()  // 调用 PubSubAPI 结构体的 checkNode 方法，检查节点状态
    if err != nil {  // 如果出现错误
        return nil, err  // 返回空和错误
    }

    settings, err := caopts.PubSubPeersOptions(opts...)  // 调用核心接口选项包的 PubSubPeersOptions 方法，获取选项参数
    if err != nil {  // 如果出现错误
        return nil, err  // 返回空和错误
    }

    span.SetAttributes(attribute.String("topic", settings.Topic))  // 设置追踪信息的属性

    return api.pubSub.ListPeers(settings.Topic), nil  // 返回 PubSubAPI 结构体的 pubSub 对象的指定主题的对等节点列表和空
}

func (api *PubSubAPI) Publish(ctx context.Context, topic string, data []byte) error {  // 定义 PubSubAPI 结构体的 Publish 方法，接收上下文参数、主题和数据，返回错误
    _, span := tracing.Span(ctx, "CoreAPI.PubSubAPI", "Publish", trace.WithAttributes(attribute.String("topic", topic)))  // 调用追踪包的 Span 方法，记录追踪信息
    defer span.End()  // 在函数返回时结束追踪

    _, err := api.checkNode()  // 调用 PubSubAPI 结构体的 checkNode 方法，检查节点状态
    if err != nil {  // 如果出现错误
        return err  // 返回错误
    }

    //nolint deprecated  // 忽略过时的警告
    return api.pubSub.Publish(topic, data)  // 调用 PubSubAPI 结构体的 pubSub 对象的 Publish 方法，发布指定主题的数据
}

func (api *PubSubAPI) Subscribe(ctx context.Context, topic string, opts ...caopts.PubSubSubscribeOption) (coreiface.PubSubSubscription, error) {  // 定义 PubSubAPI 结构体的 Subscribe 方法，接收上下文参数、主题和选项参数，返回发布订阅订阅对象和错误
    _, span := tracing.Span(ctx, "CoreAPI.PubSubAPI", "Subscribe", trace.WithAttributes(attribute.String("topic", topic)))  // 调用追踪包的 Span 方法，记录追踪信息
    // 结束跟踪 span
    defer span.End()

    // 解析选项以避免对无效选项引入静默失败。然而，我们目前没有任何用途。唯一的订阅选项，discovery，现在已经是一个空操作，因为它由 pubsub 本身处理。
    _, err := caopts.PubSubSubscribeOptions(opts...)
    if err != nil {
        return nil, err
    }

    // 检查节点状态
    _, err = api.checkNode()
    if err != nil {
        return nil, err
    }

    //nolint deprecated
    // 订阅指定主题的 pubsub 通道
    sub, err := api.pubSub.Subscribe(topic)
    if err != nil {
        return nil, err
    }

    // 返回 pubSubSubscription 对象和 nil 错误
    return &pubSubSubscription{sub}, nil
# 关闭 pubSubSubscription 对象
func (sub *pubSubSubscription) Close() error:
    # 取消订阅
    sub.subscription.Cancel()
    # 返回空错误
    return nil

# 获取下一条 pubsub 消息
func (sub *pubSubSubscription) Next(ctx context.Context) (coreiface.PubSubMessage, error):
    # 创建追踪 span
    ctx, span := tracing.Span(ctx, "CoreAPI.PubSubSubscription", "Next")
    # 结束追踪 span
    defer span.End()

    # 获取下一条消息
    msg, err := sub.subscription.Next(ctx)
    # 如果出现错误，返回空消息和错误
    if err != nil:
        return nil, err

    # 返回 pubSubMessage 对象
    return &pubSubMessage{msg}, nil

# 获取消息发送者的 peer.ID
func (msg *pubSubMessage) From() peer.ID:
    return peer.ID(msg.msg.From)

# 获取消息数据
func (msg *pubSubMessage) Data() []byte:
    return msg.msg.Data

# 获取消息序列号
func (msg *pubSubMessage) Seq() []byte:
    return msg.msg.Seqno

# 获取消息主题
func (msg *pubSubMessage) Topics() []string:
    # TODO: 处理下游变更，返回单个字符串
    if msg.msg.Topic == nil:
        return nil
    return []string{*msg.msg.Topic}
```