# `grype\grype\presenter\models\descriptor.go`

```
// 定义了一个结构体 descriptor，用于描述创建文档的信息以及周围的元数据
type descriptor struct {
    // 文档的名称
    Name string `json:"name"`
    // 文档的版本
    Version string `json:"version"`
    // 文档的配置信息，可以为空
    Configuration interface{} `json:"configuration,omitempty"`
    // 漏洞数据库的状态信息，可以为空
    VulnerabilityDBStatus interface{} `json:"db,omitempty"`
    // 时间戳，表示文档创建的时间
    Timestamp string `json:"timestamp"`
}
```