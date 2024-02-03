# `kubo\core\coreiface\pubsub.go`

```go
package iface

import (
    "context"  // 上下文包，用于处理请求的取消、超时等
    "io"  // 输入输出包，提供了接口和函数用于读写数据

    "github.com/ipfs/kubo/core/coreiface/options"  // 导入第三方包

    "github.com/libp2p/go-libp2p/core/peer"  // 导入第三方包
)

// PubSubSubscription is an active PubSub subscription
type PubSubSubscription interface {
    io.Closer  // 实现io.Closer接口，用于关闭资源

    // Next return the next incoming message
    Next(context.Context) (PubSubMessage, error)  // 返回下一个传入的消息
}

// PubSubMessage is a single PubSub message
type PubSubMessage interface {
    // From returns id of a peer from which the message has arrived
    From() peer.ID  // 返回消息来源的对等节点ID

    // Data returns the message body
    Data() []byte  // 返回消息体

    // Seq returns message identifier
    Seq() []byte  // 返回消息标识符

    // Topics returns list of topics this message was set to
    Topics() []string  // 返回消息所属的主题列表
}

// PubSubAPI specifies the interface to PubSub
type PubSubAPI interface {
    // Ls lists subscribed topics by name
    Ls(context.Context) ([]string, error)  // 列出订阅的主题名称

    // Peers list peers we are currently pubsubbing with
    Peers(context.Context, ...options.PubSubPeersOption) ([]peer.ID, error)  // 列出当前正在进行pubsub的对等节点

    // Publish a message to a given pubsub topic
    Publish(context.Context, string, []byte) error  // 向指定的pubsub主题发布消息

    // Subscribe to messages on a given topic
    Subscribe(context.Context, string, ...options.PubSubSubscribeOption) (PubSubSubscription, error)  // 订阅指定主题的消息
}
```