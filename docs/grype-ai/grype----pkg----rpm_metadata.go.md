# `grype\grype\pkg\rpm_metadata.go`

```
# 定义了一个名为 RpmMetadata 的结构体，用于存储 RPM 包的元数据信息
package pkg

# 定义了 RpmMetadata 结构体的字段 Epoch，类型为指向整数的指针，并使用 json 标签指定 JSON 序列化时的字段名为 "epoch"
type RpmMetadata struct {
    Epoch           *int   `json:"epoch"`
    # 定义了 RpmMetadata 结构体的字段 ModularityLabel，类型为字符串，并使用 json 标签指定 JSON 序列化时的字段名为 "modularityLabel"
    ModularityLabel string `json:"modularityLabel"`
}
```