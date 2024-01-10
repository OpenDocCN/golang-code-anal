# `grype\grype\presenter\models\descriptor.go`

```
// 定义结构体 descriptor，描述了创建文档的信息以及周围的元数据
type descriptor struct {
    // 文档的名称
    Name                  string      `json:"name"`
    // 文档的版本
    Version               string      `json:"version"`
    // 文档的配置信息，可以为空
    Configuration         interface{} `json:"configuration,omitempty"`
    // 漏洞数据库的状态信息，可以为空
    VulnerabilityDBStatus interface{} `json:"db,omitempty"`
    // 时间戳，记录了文档创建的时间
    Timestamp             string      `json:"timestamp"`
}
```