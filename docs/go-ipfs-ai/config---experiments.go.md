# `kubo\config\experiments.go`

```go
package config

// Experiments 结构体定义了一系列实验性功能的开关
type Experiments struct {
    FilestoreEnabled              bool
    UrlstoreEnabled               bool
    ShardingEnabled               bool `json:",omitempty"` // 由 autosharding 废弃：https://github.com/ipfs/kubo/pull/8527
    Libp2pStreamMounting          bool
    P2pHttpProxy                  bool //nolint
    StrategicProviding            bool
    OptimisticProvide             bool
    OptimisticProvideJobsPoolSize int
    GatewayOverLibp2p             bool `json:",omitempty"`

    GraphsyncEnabled     graphsyncEnabled                 `json:",omitempty"` // 图形同步功能是否启用
    AcceleratedDHTClient experimentalAcceleratedDHTClient `json:",omitempty"` // 加速的 DHT 客户端实验功能
}
```