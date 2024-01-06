# `grype\grype\presenter\models\metadata_mock.go`

```
// 导入 anchore/grype/grype/vulnerability 包
import "github.com/anchore/grype/grype/vulnerability"

// 定义 MetadataMock 结构体，实现 vulnerability.MetadataProvider 接口，用于测试目的
type MetadataMock struct {
	store map[string]map[string]vulnerability.Metadata
}

// 定义 MockVendorMetadata 结构体，包含 BaseSeverity 和 Status 字段
type MockVendorMetadata struct {
	BaseSeverity string
	Status       string
}

// NewMetadataMock 返回一个 MetadataMock 实例
func NewMetadataMock() *MetadataMock {
	// 初始化 MetadataMock 结构体的 store 字段
	return &MetadataMock{
		store: map[string]map[string]vulnerability.Metadata{
# 定义一个名为 "CVE-1999-0001" 的漏洞对象，包含了来自 "source-1" 的信息
"CVE-1999-0001": {
    # "source-1" 的描述信息
    "source-1": {
        Description: "1999-01 description",
        # 漏洞严重程度为低
        Severity:    "Low",
        # 漏洞的 CVSS 评分信息
        Cvss: []vulnerability.Cvss{
            {
                # CVSS 评分的具体指标
                Metrics: vulnerability.CvssMetrics{
                    BaseScore: 4,
                },
                # CVSS 的向量信息
                Vector:  "another vector",
                # CVSS 的版本信息
                Version: "3.0",
            },
        },
    },
},
# 定义一个名为 "CVE-1999-0002" 的漏洞对象，包含了来自 "source-2" 的信息
"CVE-1999-0002": {
    # "source-2" 的描述信息
    "source-2": {
        Description: "1999-02 description",
        # 漏洞严重程度为严重
        Severity:    "Critical",
        # 漏洞的 CVSS 评分信息
        Cvss: []vulnerability.Cvss{
# 创建一个包含漏洞信息的数据结构
{
    # 创建一个包含 CVSS 指标的对象
    Metrics: vulnerability.NewCvssMetrics(
        1,  # 基础指标
        2,  # 严重性指标
        3,  # 可用性指标
    ),
    Vector:  "vector",  # 指定漏洞的向量
    Version: "2.0",  # 指定漏洞的版本
    VendorMetadata: MockVendorMetadata{  # 创建一个模拟的供应商元数据对象
        BaseSeverity: "Low",  # 指定基础严重性
        Status:       "verified",  # 指定状态
    },
},
# 创建一个包含漏洞信息的数据结构
},
"CVE-1999-0003": {  # 指定漏洞编号
    "source-1": {  # 指定来源
        Description: "1999-03 description",  # 指定描述
        Severity:    "High",  # 指定严重性
# 定义一个名为CVE-1999-0004的漏洞，包含了source-2的信息
"CVE-1999-0004": {
    "source-2": {
        # 描述漏洞的信息
        Description: "1999-04 description",
        # 漏洞的严重程度
        Severity:    "Critical",
        # 漏洞的CVSS评分
        Cvss: []vulnerability.Cvss{
            {
                # 定义CVSS评分的指标
                Metrics: vulnerability.NewCvssMetrics(
                    1,
                    2,
                    3,
                ),
                # CVSS评分的向量
                Vector:  "vector",
                # CVSS评分的版本
                Version: "2.0",
                # 漏洞的供应商元数据
                VendorMetadata: MockVendorMetadata{
                    BaseSeverity: "Low",
                    Status:       "verified",
                },
            },
// GetMetadata函数用于根据给定的id和recordSource返回漏洞元数据
func (m *MetadataMock) GetMetadata(id, namespace string) (*vulnerability.Metadata, error) {
    // 从存储中获取指定id和namespace的值
    value := m.store[id][namespace]
    // 返回获取的值作为指针类型的漏洞元数据，同时返回nil错误
    return &value, nil
}
```