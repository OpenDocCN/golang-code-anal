# `kubo\config\ipns.go`

```go
# 定义一个名为Ipns的结构体，用于存储IPNS配置信息
type Ipns struct {
    RepublishPeriod string  # 存储IPNS记录重新发布的周期
    RecordLifetime  string  # 存储IPNS记录的生命周期

    ResolveCacheSize int  # 存储解析缓存的大小

    // Enable namesys pubsub (--enable-namesys-pubsub)
    UsePubsub Flag `json:",omitempty"`  # 存储是否启用namesys pubsub的标志，使用json标签omitempty表示在json序列化时如果为空则忽略
}
```