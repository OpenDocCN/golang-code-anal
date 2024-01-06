# `grype\grype\pkg\rpm_metadata.go`

```
# 定义了一个名为 RpmMetadata 的结构体，用于存储 RPM 包的元数据信息
package pkg

# 定义了 RpmMetadata 结构体的字段
type RpmMetadata struct {
    # RPM 包的 Epoch 字段，指向一个整数指针，使用 json 标签指定在 JSON 序列化时的字段名为 "epoch"
    Epoch           *int   `json:"epoch"`
    # RPM 包的 ModularityLabel 字段，存储模块化标签信息，使用 json 标签指定在 JSON 序列化时的字段名为 "modularityLabel"
    ModularityLabel string `json:"modularityLabel"`
}
```