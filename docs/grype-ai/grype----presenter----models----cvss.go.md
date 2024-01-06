# `grype\grype\presenter\models\cvss.go`

```
// 导入名为 "vulnerability" 的包，该包中包含了 Metadata 结构体
import "github.com/anchore/grype/grype/vulnerability"

// 定义名为 Cvss 的结构体，包含了漏洞的 CVSS 信息
type Cvss struct {
	Source         string      `json:"source,omitempty"`  // 漏洞信息的来源
	Type           string      `json:"type,omitempty"`    // 漏洞类型
	Version        string      `json:"version"`           // CVSS 版本
	Vector         string      `json:"vector"`            // CVSS 向量
	Metrics        CvssMetrics `json:"metrics"`           // CVSS 评分指标
	VendorMetadata interface{} `json:"vendorMetadata"`    // 厂商元数据
}

// 定义名为 CvssMetrics 的结构体，包含了 CVSS 评分指标
type CvssMetrics struct {
	BaseScore           float64  `json:"baseScore"`                // 基础分数
	ExploitabilityScore *float64 `json:"exploitabilityScore,omitempty"`  // 可利用性分数
	ImpactScore         *float64 `json:"impactScore,omitempty"`    // 影响分数
}

// 定义名为 NewCVSS 的函数，接收一个指向 vulnerability.Metadata 结构体的指针，并返回 Cvss 结构体的切片
func NewCVSS(metadata *vulnerability.Metadata) []Cvss {
# 创建一个空的 Cvss 切片
cvss := make([]Cvss, 0)
# 遍历 metadata.Cvss 中的每个元素
for _, score := range metadata.Cvss {
    # 获取 score.VendorMetadata
    vendorMetadata := score.VendorMetadata
    # 如果 vendorMetadata 为空，则创建一个空的 map
    if vendorMetadata == nil {
        vendorMetadata = make(map[string]interface{})
    }
    # 将当前 score 的信息添加到 cvss 切片中
    cvss = append(cvss, Cvss{
        Source:  score.Source,
        Type:    score.Type,
        Version: score.Version,
        Vector:  score.Vector,
        Metrics: CvssMetrics{
            BaseScore:           score.Metrics.BaseScore,
            ExploitabilityScore: score.Metrics.ExploitabilityScore,
            ImpactScore:         score.Metrics.ImpactScore,
        },
        VendorMetadata: vendorMetadata,
    })
}
# 返回填充好数据的 cvss 切片
return cvss
这是一个代码块的结束符号，表示前面的函数或者循环的结束。
```