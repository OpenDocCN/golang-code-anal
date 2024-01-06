# `grype\grype\presenter\models\match.go`

```
package models

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"sort" // 导入 sort 包，用于对数据进行排序

	"github.com/anchore/grype/grype/match" // 导入 match 包
	"github.com/anchore/grype/grype/pkg"   // 导入 pkg 包
	"github.com/anchore/grype/grype/vulnerability" // 导入 vulnerability 包
)

// Match 是报告的 JSON 数组中的单个条目
type Match struct {
	Vulnerability          Vulnerability           `json:"vulnerability"` // 漏洞信息
	RelatedVulnerabilities []VulnerabilityMetadata `json:"relatedVulnerabilities"` // 相关漏洞信息
	MatchDetails           []MatchDetails          `json:"matchDetails"` // 匹配细节
	Artifact               Package                 `json:"artifact"` // 软件包信息
}

// MatchDetails 包含所有指示结果匹配方式的数据
// 定义 MatchDetails 结构体，包含了匹配的类型、匹配器、搜索属性和匹配的结果
type MatchDetails struct {
	Type       string      `json:"type"`
	Matcher    string      `json:"matcher"`
	SearchedBy interface{} `json:"searchedBy"` // 指示匹配是如何进行的
	Found      interface{} `json:"found"`      // 指示匹配的具体结果
}

// 创建新的匹配对象，包括匹配信息、软件包信息和漏洞元数据提供者
func newMatch(m match.Match, p pkg.Package, metadataProvider vulnerability.MetadataProvider) (*Match, error) {
	// 初始化相关漏洞元数据的切片
	relatedVulnerabilities := make([]VulnerabilityMetadata, 0)
	// 遍历相关漏洞列表，获取相关漏洞的元数据
	for _, r := range m.Vulnerability.RelatedVulnerabilities {
		relatedMetadata, err := metadataProvider.GetMetadata(r.ID, r.Namespace)
		// 如果获取失败，返回错误信息
		if err != nil {
			return nil, fmt.Errorf("unable to fetch related vuln=%q metadata: %+v", r, err)
		}
		// 如果获取成功，将相关漏洞的元数据添加到切片中
		if relatedMetadata != nil {
			relatedVulnerabilities = append(relatedVulnerabilities, NewVulnerabilityMetadata(r.ID, r.Namespace, relatedMetadata))
		}
	}

	// 获取当前漏洞的元数据
	metadata, err := metadataProvider.GetMetadata(m.Vulnerability.ID, m.Vulnerability.Namespace)
// 如果发生错误，则返回空值和错误信息
if err != nil {
    return nil, fmt.Errorf("unable to fetch vuln=%q metadata: %+v", m.Vulnerability.ID, err)
}

// 创建一个与 m.Details 长度相同的 MatchDetails 切片
details := make([]MatchDetails, len(m.Details))
// 遍历 m.Details，将每个元素赋值给 details 中对应的元素
for idx, d := range m.Details {
    details[idx] = MatchDetails{
        Type:       string(d.Type),
        Matcher:    string(d.Matcher),
        SearchedBy: d.SearchedBy,
        Found:      d.Found,
    }
}

// 返回一个指向 Match 结构体的指针，包括漏洞、元数据、软件包和相关漏洞的详细信息
return &Match{
    Vulnerability:          NewVulnerability(m.Vulnerability, metadata),
    Artifact:               newPackage(p),
    RelatedVulnerabilities: relatedVulnerabilities,
    MatchDetails:           details,
}, nil
// 定义一个接口类型，表示实现了 sort.Interface 接口的 MatchSort 类型
var _ sort.Interface = (*MatchSort)(nil)

// 定义 MatchSort 结构体，用于排序
type MatchSort []Match

// Len 返回集合中的元素数量
func (m MatchSort) Len() int {
	return len(m)
}

// Less 报告索引为 i 的元素是否应在索引为 j 的元素之前排序
// 排序应该在展示器之间保持一致：名称、版本、类型、严重程度、漏洞
func (m MatchSort) Less(i, j int) bool {
	matchI := m[i]
	matchJ := m[j]
if matchI.Artifact.Name == matchJ.Artifact.Name {
		if matchI.Artifact.Version == matchJ.Artifact.Version {
			if matchI.Artifact.Type == matchJ.Artifact.Type {
				if SeverityScore(matchI.Vulnerability.Severity) == SeverityScore(matchJ.Vulnerability.Severity) {
// 比较两个匹配对象的漏洞ID，返回比较结果
func (m MatchSort) Less(i, j int) bool {
    return matchI.Vulnerability.ID > matchJ.Vulnerability.ID
}

// 如果两个匹配对象的漏洞ID相同，则比较它们的漏洞严重程度，返回比较结果
func (m MatchSort) Less(i, j int) bool {
    return SeverityScore(matchI.Vulnerability.Severity) > SeverityScore(matchJ.Vulnerability.Severity)
}

// 如果两个匹配对象的漏洞ID和严重程度都相同，则比较它们的软件类型，返回比较结果
func (m MatchSort) Less(i, j int) bool {
    return matchI.Artifact.Type < matchJ.Artifact.Type
}

// 如果两个匹配对象的漏洞ID、严重程度和软件类型都相同，则比较它们的软件版本，返回比较结果
func (m MatchSort) Less(i, j int) bool {
    return matchI.Artifact.Version < matchJ.Artifact.Version
}

// 如果两个匹配对象的漏洞ID、严重程度、软件类型和版本都相同，则比较它们的软件名称，返回比较结果
func (m MatchSort) Less(i, j int) bool {
    return matchI.Artifact.Name < matchJ.Artifact.Name
}

// 交换匹配对象列表中索引为i和j的元素
func (m MatchSort) Swap(i, j int) {
    m[i], m[j] = m[j], m[i]
}
```