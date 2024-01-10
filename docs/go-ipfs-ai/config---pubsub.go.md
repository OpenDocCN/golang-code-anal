# `kubo\config\pubsub.go`

```
package config

const (
    // LastSeenMessagesStrategy 是一种策略，根据最后一次看到 Pubsub 消息的时间来计算 TTL 倒计时。
    // 这意味着如果接收到消息，然后在指定的 TTL 窗口内再次看到消息，它将在 TTL 倒计时从最后一次看到消息的时间到期之前不会被发送。
    LastSeenMessagesStrategy = "last-seen"

    // FirstSeenMessagesStrategy 是一种策略，根据 Pubsub 消息首次被看到的时间来计算 TTL 倒计时。
    // 这意味着如果接收到消息，然后在指定的 TTL 窗口内再次看到消息，它将不会被发送。
    FirstSeenMessagesStrategy = "first-seen"

    // DefaultSeenMessagesStrategy 是如果没有指定 Pubsub.SeenMessagesStrategy，则默认使用的策略。
    DefaultSeenMessagesStrategy = LastSeenMessagesStrategy
)

type PubsubConfig struct {
    // Router 可以是 floodsub（传统）或 gossipsub（新的并且向后兼容）。
    Router string

    // DisableSigning 禁用消息签名。消息签名默认情况下是 *启用* 的。
    DisableSigning bool

    // Enable pubsub (--enable-pubsub-experiment)
    Enabled Flag `json:",omitempty"`

    // SeenMessagesTTL 是一个控制在其中重复消息将被识别并且不会被发送的时间窗口的值。
    SeenMessagesTTL *OptionalDuration `json:",omitempty"`

    // SeenMessagesStrategy 是一个设置，确定如何计算用于去重消息的生存时间（TTL）倒计时。
    SeenMessagesStrategy *OptionalString `json:",omitempty"`
}
```