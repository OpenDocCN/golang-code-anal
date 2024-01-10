# `grype\grype\presenter\models\match.go`

```
package models

import (
    "fmt"
    "sort"

    "github.com/anchore/grype/grype/match"
    "github.com/anchore/grype/grype/pkg"
    "github.com/anchore/grype/grype/vulnerability"
)

// Match is a single item for the JSON array reported
type Match struct {
    Vulnerability          Vulnerability           `json:"vulnerability"` // 表示漏洞信息
    RelatedVulnerabilities []VulnerabilityMetadata `json:"relatedVulnerabilities"` // 相关漏洞信息
    MatchDetails           []MatchDetails          `json:"matchDetails"` // 匹配细节
    Artifact               Package                 `json:"artifact"` // 软件包信息
}

// MatchDetails contains all data that indicates how the result match was found
type MatchDetails struct {
    Type       string      `json:"type"` // 匹配类型
    Matcher    string      `json:"matcher"` // 匹配器
    SearchedBy interface{} `json:"searchedBy"` // 被搜索的特定属性（除了软件包名称和版本）--表示匹配是如何进行的
    Found      interface{} `json:"found"`      // 漏洞对象上被匹配的特定属性--表示匹配的是什么/在哪里
}

func newMatch(m match.Match, p pkg.Package, metadataProvider vulnerability.MetadataProvider) (*Match, error) {
    relatedVulnerabilities := make([]VulnerabilityMetadata, 0)
    for _, r := range m.Vulnerability.RelatedVulnerabilities {
        relatedMetadata, err := metadataProvider.GetMetadata(r.ID, r.Namespace)
        if err != nil {
            return nil, fmt.Errorf("unable to fetch related vuln=%q metadata: %+v", r, err)
        }
        if relatedMetadata != nil {
            relatedVulnerabilities = append(relatedVulnerabilities, NewVulnerabilityMetadata(r.ID, r.Namespace, relatedMetadata))
        }
    }

    metadata, err := metadataProvider.GetMetadata(m.Vulnerability.ID, m.Vulnerability.Namespace)
    if err != nil {
        return nil, fmt.Errorf("unable to fetch vuln=%q metadata: %+v", m.Vulnerability.ID, err)
    }
    // 创建一个与 m.Details 长度相同的 MatchDetails 切片
    details := make([]MatchDetails, len(m.Details))
    // 遍历 m.Details，将每个元素转换为 MatchDetails 结构体并存储到 details 切片中
    for idx, d := range m.Details {
        details[idx] = MatchDetails{
            Type:       string(d.Type),
            Matcher:    string(d.Matcher),
            SearchedBy: d.SearchedBy,
            Found:      d.Found,
        }
    }

    // 返回一个指向 Match 结构体的指针，包含了新创建的 Vulnerability、Artifact、RelatedVulnerabilities 和 details
    return &Match{
        Vulnerability:          NewVulnerability(m.Vulnerability, metadata),
        Artifact:               newPackage(p),
        RelatedVulnerabilities: relatedVulnerabilities,
        MatchDetails:           details,
    }, nil
# 声明 MatchSort 类型实现 sort.Interface 接口
var _ sort.Interface = (*MatchSort)(nil)

# 定义 MatchSort 结构体
type MatchSort []Match

# 返回集合中的元素数量
func (m MatchSort) Len() int {
    return len(m)
}

# 比较索引为 i 和 j 的元素，确定它们的排序顺序
# 排序应该在展示器之间保持一致：名称、版本、类型、严重程度、漏洞
func (m MatchSort) Less(i, j int) bool {
    matchI := m[i]
    matchJ := m[j]
    if matchI.Artifact.Name == matchJ.Artifact.Name {
        if matchI.Artifact.Version == matchJ.Artifact.Version {
            if matchI.Artifact.Type == matchJ.Artifact.Type {
                if SeverityScore(matchI.Vulnerability.Severity) == SeverityScore(matchJ.Vulnerability.Severity) {
                    return matchI.Vulnerability.ID > matchJ.Vulnerability.ID
                }
                return SeverityScore(matchI.Vulnerability.Severity) > SeverityScore(matchJ.Vulnerability.Severity)
            }
            return matchI.Artifact.Type < matchJ.Artifact.Type
        }
        return matchI.Artifact.Version < matchJ.Artifact.Version
    }
    return matchI.Artifact.Name < matchJ.Artifact.Name
}

# 交换索引 i 和 j 处的元素
func (m MatchSort) Swap(i, j int) {
    m[i], m[j] = m[j], m[i]
}
```