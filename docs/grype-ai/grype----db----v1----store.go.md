# `grype\grype\db\v1\store.go`

```
# 定义一个接口类型 Store，该接口包含 StoreReader 和 StoreWriter 两个子接口
type Store interface {
    # 定义 StoreReader 接口，包含 IDReader、VulnerabilityStoreReader 和 VulnerabilityMetadataStoreReader 三个子接口
    StoreReader
    # 定义 StoreWriter 接口，包含 IDWriter、VulnerabilityStoreWriter 和 VulnerabilityMetadataStoreWriter 三个子接口，以及 Close 方法
    StoreWriter
}

# 定义 StoreReader 接口，包含 IDReader、VulnerabilityStoreReader 和 VulnerabilityMetadataStoreReader 三个子接口
type StoreReader interface {
    IDReader
    VulnerabilityStoreReader
    VulnerabilityMetadataStoreReader
}

# 定义 StoreWriter 接口，包含 IDWriter、VulnerabilityStoreWriter 和 VulnerabilityMetadataStoreWriter 三个子接口，以及 Close 方法
type StoreWriter interface {
    IDWriter
    VulnerabilityStoreWriter
    VulnerabilityMetadataStoreWriter
    Close()
}
```