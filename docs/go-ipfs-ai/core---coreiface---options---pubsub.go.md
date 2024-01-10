# `kubo\core\coreiface\options\pubsub.go`

```
package options

// 定义 PubSubPeersSettings 结构体，包含 Topic 字段
type PubSubPeersSettings struct {
    Topic string
}

// 定义 PubSubSubscribeSettings 结构体，包含 Discover 字段
type PubSubSubscribeSettings struct {
    Discover bool
}

// 定义 PubSubPeersOption 和 PubSubSubscribeOption 类型
type (
    PubSubPeersOption     func(*PubSubPeersSettings) error
    PubSubSubscribeOption func(*PubSubSubscribeSettings) error
)

// PubSubPeersOptions 函数，接收 PubSubPeersOption 类型的可变参数，返回 PubSubPeersSettings 和 error
func PubSubPeersOptions(opts ...PubSubPeersOption) (*PubSubPeersSettings, error) {
    // 初始化 options 结构体
    options := &PubSubPeersSettings{
        Topic: "",
    }

    // 遍历参数中的选项，对 options 进行设置
    for _, opt := range opts {
        err := opt(options)
        if err != nil {
            return nil, err
        }
    }
    return options, nil
}

// PubSubSubscribeOptions 函数，接收 PubSubSubscribeOption 类型的可变参数，返回 PubSubSubscribeSettings 和 error
func PubSubSubscribeOptions(opts ...PubSubSubscribeOption) (*PubSubSubscribeSettings, error) {
    // 初始化 options 结构体
    options := &PubSubSubscribeSettings{
        Discover: false,
    }

    // 遍历参数中的选项，对 options 进行设置
    for _, opt := range opts {
        err := opt(options)
        if err != nil {
            return nil, err
        }
    }
    return options, nil
}

// 定义 pubsubOpts 结构体
type pubsubOpts struct{}

// 定义 PubSub 结构体
var PubSub pubsubOpts

// Topic 方法，返回 PubSubPeersOption 类型的函数
func (pubsubOpts) Topic(topic string) PubSubPeersOption {
    return func(settings *PubSubPeersSettings) error {
        settings.Topic = topic
        return nil
    }
}

// Discover 方法，返回 PubSubSubscribeOption 类型的函数
func (pubsubOpts) Discover(discover bool) PubSubSubscribeOption {
    return func(settings *PubSubSubscribeSettings) error {
        settings.Discover = discover
        return nil
    }
}
```