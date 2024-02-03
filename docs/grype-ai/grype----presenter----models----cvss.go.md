# `grype\grype\presenter\models\cvss.go`

```go
package models

import "github.com/anchore/grype/grype/vulnerability"

type Cvss struct {
    Source         string      `json:"source,omitempty"`  // 漏洞评分来源
    Type           string      `json:"type,omitempty"`    // 漏洞评分类型
    Version        string      `json:"version"`          // 漏洞评分版本
    Vector         string      `json:"vector"`           // 漏洞评分向量
    Metrics        CvssMetrics `json:"metrics"`          // 漏洞评分指标
    VendorMetadata interface{} `json:"vendorMetadata"`   // 供应商元数据
}

type CvssMetrics struct {
    BaseScore           float64  `json:"baseScore"`                // 基础分数
    ExploitabilityScore *float64 `json:"exploitabilityScore,omitempty"`  // 可利用性分数
    ImpactScore         *float64 `json:"impactScore,omitempty"`        // 影响分数
}

func NewCVSS(metadata *vulnerability.Metadata) []Cvss {
    cvss := make([]Cvss, 0)  // 创建空的漏洞评分数组
    for _, score := range metadata.Cvss {  // 遍历元数据中的漏洞评分
        vendorMetadata := score.VendorMetadata  // 获取供应商元数据
        if vendorMetadata == nil {  // 如果供应商元数据为空
            vendorMetadata = make(map[string]interface{})  // 创建一个空的供应商元数据映射
        }
        // 将漏洞评分信息添加到漏洞评分数组中
        cvss = append(cvss, Cvss{
            Source:  score.Source,  // 设置来源
            Type:    score.Type,    // 设置类型
            Version: score.Version,  // 设置版本
            Vector:  score.Vector,   // 设置向量
            Metrics: CvssMetrics{    // 设置指标
                BaseScore:           score.Metrics.BaseScore,  // 设置基础分数
                ExploitabilityScore: score.Metrics.ExploitabilityScore,  // 设置可利用性分数
                ImpactScore:         score.Metrics.ImpactScore,  // 设置影响分数
            },
            VendorMetadata: vendorMetadata,  // 设置供应商元数据
        })
    }
    return cvss  // 返回漏洞评分数组
}
```