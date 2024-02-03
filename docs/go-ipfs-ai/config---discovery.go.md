# `kubo\config\discovery.go`

```go
# 定义一个名为 Discovery 的结构体
type Discovery struct {
    MDNS MDNS
}

# 定义一个名为 MDNS 的结构体
type MDNS struct {
    Enabled bool
    # 标记：已废弃，不再可配置发现轮次间隔时间
    # 参考：https://github.com/ipfs/go-ipfs/pull/9048#discussion_r906814717
    Interval *OptionalInteger `json:",omitempty"`
}
```