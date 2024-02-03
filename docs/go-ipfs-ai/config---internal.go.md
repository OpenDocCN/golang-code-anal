# `kubo\config\internal.go`

```go
package config

type Internal struct {
    // 所有标记为omitempty，因为我们期望对Internal的所有子组件进行更改
    Bitswap                     *InternalBitswap  `json:",omitempty"`  // 内部位交换的配置
    UnixFSShardingSizeThreshold *OptionalString   `json:",omitempty"`  // UnixFS分片大小阈值
    Libp2pForceReachability     *OptionalString   `json:",omitempty"`  // 强制Libp2p可达性
    BackupBootstrapInterval     *OptionalDuration `json:",omitempty"`  // 备份引导间隔
}

type InternalBitswap struct {
    TaskWorkerCount             OptionalInteger   // 任务工作器数量
    EngineBlockstoreWorkerCount OptionalInteger   // 引擎块存储工作器数量
    EngineTaskWorkerCount       OptionalInteger   // 引擎任务工作器数量
    MaxOutstandingBytesPerPeer  OptionalInteger   // 每个对等节点的最大未完成字节数
    ProviderSearchDelay         OptionalDuration  // 提供者搜索延迟
}
```