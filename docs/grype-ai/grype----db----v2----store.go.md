# `grype\grype\db\v2\store.go`

```
# 定义了一个接口类型 Store，该接口包含了 StoreReader 和 StoreWriter 两个子接口
type Store interface {
    # 定义了一个子接口 StoreReader，该接口包含了 IDReader、VulnerabilityStoreReader 和 VulnerabilityMetadataStoreReader 三个子接口
    StoreReader
    # 定义了一个子接口 StoreWriter，该接口包含了 IDWriter、VulnerabilityStoreWriter 和 VulnerabilityMetadataStoreWriter 三个子接口，以及 Close 方法
    StoreWriter
}

# 定义了一个接口类型 StoreReader，该接口包含了 IDReader、VulnerabilityStoreReader 和 VulnerabilityMetadataStoreReader 三个子接口
type StoreReader interface {
    IDReader
    VulnerabilityStoreReader
    VulnerabilityMetadataStoreReader
}

# 定义了一个接口类型 StoreWriter，该接口包含了 IDWriter、VulnerabilityStoreWriter 和 VulnerabilityMetadataStoreWriter 三个子接口，以及 Close 方法
type StoreWriter interface {
    IDWriter
    VulnerabilityStoreWriter
    VulnerabilityMetadataStoreWriter
    Close()
}
```