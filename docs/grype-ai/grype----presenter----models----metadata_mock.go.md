# `grype\grype\presenter\models\metadata_mock.go`

```
package models

import "github.com/anchore/grype/grype/vulnerability"

// 定义一个接口类型的变量，用于实现漏洞元数据提供者的行为
var _ vulnerability.MetadataProvider = (*MetadataMock)(nil)

// MetadataMock 为测试目的提供了漏洞元数据提供者所需的行为
type MetadataMock struct {
    store map[string]map[string]vulnerability.Metadata
}

// MockVendorMetadata 定义了一个模拟供应商元数据的结构
type MockVendorMetadata struct {
    BaseSeverity string
    Status       string
}

// NewMetadataMock 返回一个 MetadataMock 的新实例
func NewMetadataMock() *MetadataMock {
    // 返回一个空的 MetadataMock 实例
    return &MetadataMock{}
}

// GetMetadata 根据给定的 id 和 namespace 返回漏洞元数据
func (m *MetadataMock) GetMetadata(id, namespace string) (*vulnerability.Metadata, error) {
    // 获取存储中对应 id 和 namespace 的漏洞元数据
    value := m.store[id][namespace]
    // 返回漏洞元数据和 nil 错误
    return &value, nil
}
```