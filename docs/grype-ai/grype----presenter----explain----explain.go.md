# `grype\grype\presenter\explain\explain.go`

```go
package explain

import (
    _ "embed" // 导入 embed 包，但不使用它，用于静态文件嵌入
    "fmt" // 导入 fmt 包，用于格式化输出
    "io" // 导入 io 包，用于 I/O 操作
    "sort" // 导入 sort 包，用于排序
    "strings" // 导入 strings 包，用于字符串操作
    "text/template" // 导入 text/template 包，用于文本模板操作

    "github.com/anchore/grype/grype/match" // 导入 match 包，用于匹配
    "github.com/anchore/grype/grype/presenter/models" // 导入 models 包，用于展示模型
    "github.com/anchore/syft/syft/file" // 导入 file 包，用于文件操作
)

//go:embed explain_cve.tmpl
var explainTemplate string // 嵌入的漏洞解释模板

type VulnerabilityExplainer interface {
    ExplainByID(IDs []string) error // 根据漏洞 ID 解释漏洞
    ExplainBySeverity(severity string) error // 根据严重程度解释漏洞
    ExplainAll() error // 解释所有漏洞
}

type ViewModel struct {
    PrimaryVulnerability   models.VulnerabilityMetadata // 主要漏洞信息
    RelatedVulnerabilities []models.VulnerabilityMetadata // 相关漏洞信息
    MatchedPackages        []*explainedPackage // 匹配的软件包信息
    URLs                   []string // URL 列表
}

type viewModelBuilder struct {
    PrimaryMatch   models.Match // 主要匹配
    RelatedMatches []models.Match // 相关匹配
    requestedIDs   []string // 用户请求解释的漏洞 ID
}

type Findings map[string]ViewModel // 漏洞解释结果

type explainedPackage struct {
    PURL                string // 软件包 URL
    Name                string // 软件包名称
    Version             string // 软件包版本
    MatchedOnID         string // 匹配的漏洞 ID
    MatchedOnNamespace  string // 匹配的命名空间
    IndirectExplanation string // 间接解释
    DirectExplanation   string // 直接解释
    CPEExplanation      string // CPE 解释
    Locations           []explainedEvidence // 位置信息
    displayPriority     int // 显示优先级
}

type explainedEvidence struct {
    Location     string // 位置
    ArtifactID   string // 资源 ID
    ViaVulnID    string // 通过漏洞 ID
    ViaNamespace string // 通过命名空间
}

type vulnerabilityExplainer struct {
    w   io.Writer // 写入器
    doc *models.Document // 文档
}

func NewVulnerabilityExplainer(w io.Writer, doc *models.Document) VulnerabilityExplainer {
    return &vulnerabilityExplainer{
        w:   w,
        doc: doc,
    }
}

var funcs = template.FuncMap{
    "trim": strings.TrimSpace, // 注册模板函数 trim
}

func (e *vulnerabilityExplainer) ExplainByID(ids []string) error {
    findings, err := Doc(e.doc, ids) // 根据漏洞 ID 解释漏洞
    # 如果错误不为空，则返回错误
    if err != nil:
        return err
    # 创建模板对象，并使用给定的函数映射和解析模板
    t := template.Must(template.New("explanation").Funcs(funcs).Parse(explainTemplate))
    # 遍历给定的 id 数组
    for _, id := range ids:
        # 获取对应 id 的 finding
        finding, ok := findings[id]
        # 如果找不到对应 id 的 finding，则继续下一次循环
        if !ok:
            continue
        # 使用模板对象执行 finding，并将结果写入到输出流中
        if err := t.Execute(e.w, finding); err != nil:
            # 如果执行模板时出错，则返回带有错误信息的新错误
            return fmt.Errorf("unable to execute template: %w", err)
    # 所有操作执行成功，返回空错误
    return nil
// ExplainBySeverity 根据严重程度解释漏洞，但是这里只是返回一个未实现的错误
func (e *vulnerabilityExplainer) ExplainBySeverity(_ string) error {
    return fmt.Errorf("not implemented")
}

// ExplainAll 解释所有漏洞
func (e *vulnerabilityExplainer) ExplainAll() error {
    // 从文档中获取所有的发现
    findings, err := Doc(e.doc, nil)
    if err != nil {
        return err
    }
    // 创建模板并执行，将结果写入 e.w
    t := template.Must(template.New("explanation").Funcs(funcs).Parse(explainTemplate))
    return t.Execute(e.w, findings)
}

// Doc 从文档中获取发现
func Doc(doc *models.Document, requestedIDs []string) (Findings, error) {
    result := make(Findings)
    builders := make(map[string]*viewModelBuilder)
    // 遍历文档中的匹配项
    for _, m := range doc.Matches {
        key := m.Vulnerability.ID
        existing, ok := builders[key]
        if !ok {
            existing = newBuilder(requestedIDs)
            builders[m.Vulnerability.ID] = existing
        }
        existing.WithMatch(m, requestedIDs)
    }
    // 遍历文档中的匹配项，处理相关漏洞
    for _, m := range doc.Matches {
        for _, related := range m.RelatedVulnerabilities {
            key := related.ID
            existing, ok := builders[key]
            if !ok {
                existing = newBuilder(requestedIDs)
                builders[key] = existing
            }
            existing.WithMatch(m, requestedIDs)
        }
    }
    // 将构建的结果存入 result
    for k, v := range builders {
        result[k] = v.Build()
    }
    return result, nil
}

// newBuilder 创建一个新的 viewModelBuilder
func newBuilder(requestedIDs []string) *viewModelBuilder {
    return &viewModelBuilder{
        requestedIDs: requestedIDs,
    }
}

// WithMatch 向构建器中添加一个匹配项
// 接受足够的信息来确定匹配项是主要匹配还是相关匹配
func (b *viewModelBuilder) WithMatch(m models.Match, userRequestedIDs []string) {
    if b.isPrimaryAdd(m, userRequestedIDs) {
        // 如果主要匹配已存在，则将当前主要匹配降级为相关匹配
        if b.PrimaryMatch.Vulnerability.ID != "" {
            b.WithRelatedMatch(b.PrimaryMatch)
        }
        b.WithPrimaryMatch(m)
    } else {
        b.WithRelatedMatch(m)
    }
}
// 检查候选匹配是否为主要添加的匹配
func (b *viewModelBuilder) isPrimaryAdd(candidate models.Match, userRequestedIDs []string) bool {
    // 如果主要匹配的漏洞ID为空，则返回true
    if b.PrimaryMatch.Vulnerability.ID == "" {
        return true
    }

    // 检查用户请求的ID是否存在
    idWasRequested := false
    for _, id := range userRequestedIDs {
        if candidate.Vulnerability.ID == id {
            idWasRequested = true
            break
        }
    }
    // 如果用户没有请求此ID，则不是主要匹配
    if !idWasRequested && len(userRequestedIDs) > 0 {
        return false
    }

    // NVD CPEs是漏洞的规范ID，如果用户询问了CVE-YYYY-ID类型的编号，并且我们有来自NVD的记录，则将其视为主要记录
    if candidate.Vulnerability.Namespace == "nvd:cpe" {
        return true
    }

    // 如果用户没有请求特定的ID，或者候选匹配有用户请求的ID
    for _, related := range b.PrimaryMatch.RelatedVulnerabilities {
        if related.ID == candidate.Vulnerability.ID {
            return true
        }
    }
    return false
}

// 设置主要匹配
func (b *viewModelBuilder) WithPrimaryMatch(m models.Match) *viewModelBuilder {
    b.PrimaryMatch = m
    return b
}

// 设置相关匹配
func (b *viewModelBuilder) WithRelatedMatch(m models.Match) *viewModelBuilder {
    b.RelatedMatches = append(b.RelatedMatches, m)
    return b
}

// 构建视图模型
func (b *viewModelBuilder) Build() ViewModel {
    // 解释包装并排序证据
    explainedPackages := groupAndSortEvidence(append(b.RelatedMatches, b.PrimaryMatch))

    var relatedVulnerabilities []models.VulnerabilityMetadata
    dedupeRelatedVulnerabilities := make(map[string]models.VulnerabilityMetadata)
    var sortDedupedRelatedVulnerabilities []string
}
    // 遍历主要匹配和相关匹配的漏洞列表
    for _, m := range append(b.RelatedMatches, b.PrimaryMatch) {
        // 生成漏洞的唯一标识作为键
        key := fmt.Sprintf("%s:%s", m.Vulnerability.Namespace, m.Vulnerability.ID)
        // 将漏洞信息存入去重后的相关漏洞字典中
        dedupeRelatedVulnerabilities[key] = m.Vulnerability.VulnerabilityMetadata
        // 遍历相关漏洞列表
        for _, r := range m.RelatedVulnerabilities {
            // 生成相关漏洞的唯一标识作为键
            key := fmt.Sprintf("%s:%s", r.Namespace, r.ID)
            // 将相关漏洞信息存入去重后的相关漏洞字典中
            dedupeRelatedVulnerabilities[key] = r
        }
    }

    // 从相关漏洞字典中删除主要漏洞，以避免重复列出
    primary := b.primaryVulnerability()
    delete(dedupeRelatedVulnerabilities, fmt.Sprintf("%s:%s", primary.Namespace, primary.ID))
    // 将去重后的相关漏洞字典的键转换为列表
    for k := range dedupeRelatedVulnerabilities {
        sortDedupedRelatedVulnerabilities = append(sortDedupedRelatedVulnerabilities, k)
    }
    // 对去重后的相关漏洞列表进行排序
    sort.Strings(sortDedupedRelatedVulnerabilities)
    // 将去重后的相关漏洞按顺序存入相关漏洞列表
    for _, k := range sortDedupedRelatedVulnerabilities {
        relatedVulnerabilities = append(relatedVulnerabilities, dedupeRelatedVulnerabilities[k])
    }

    // 返回视图模型对象
    return ViewModel{
        PrimaryVulnerability:   primary,
        RelatedVulnerabilities: relatedVulnerabilities,
        MatchedPackages:        explainedPackages,
        URLs:                   b.dedupeAndSortURLs(primary),
    }
// 返回主要漏洞的元数据
func (b *viewModelBuilder) primaryVulnerability() models.VulnerabilityMetadata {
    var primaryVulnerability models.VulnerabilityMetadata
    // 遍历相关匹配和主要匹配
    for _, m := range append(b.RelatedMatches, b.PrimaryMatch) {
        // 遍历相关漏洞和漏洞元数据
        for _, r := range append(m.RelatedVulnerabilities, m.Vulnerability.VulnerabilityMetadata) {
            // 如果漏洞 ID 和命名空间符合条件，则将其设置为主要漏洞
            if r.ID == b.PrimaryMatch.Vulnerability.ID && r.Namespace == "nvd:cpe" {
                primaryVulnerability = r
            }
        }
    }
    // 如果主要漏洞的 ID 为空，则将其设置为主要匹配的漏洞元数据
    if primaryVulnerability.ID == "" {
        primaryVulnerability = b.PrimaryMatch.Vulnerability.VulnerabilityMetadata
    }
    return primaryVulnerability
}

// nolint:funlen
// 对证据进行分组和排序
func groupAndSortEvidence(matches []models.Match) []*explainedPackage {
    // 创建映射，用于存储 ID 到匹配详情的映射关系
    idsToMatchDetails := make(map[string]*explainedPackage)
    // 创建排序的 ID 列表
    var sortIDs []string
    // 遍历映射
    for k, v := range idsToMatchDetails {
        // 将 ID 添加到排序列表中
        sortIDs = append(sortIDs, k)
        // 创建用于去重的位置映射
        dedupeLocations := make(map[string]explainedEvidence)
        // 遍历位置列表，去重并存储到 uniqueLocations 中
        for _, l := range v.Locations {
            dedupeLocations[l.Location] = l
        }
        var uniqueLocations []explainedEvidence
        for _, l := range dedupeLocations {
            uniqueLocations = append(uniqueLocations, l)
        }
        // 对 uniqueLocations 进行排序
        sort.Slice(uniqueLocations, func(i, j int) bool {
            if uniqueLocations[i].ViaNamespace == uniqueLocations[j].ViaNamespace {
                return uniqueLocations[i].Location < uniqueLocations[j].Location
            }
            return uniqueLocations[i].ViaNamespace < uniqueLocations[j].ViaNamespace
        })
        // 更新匹配详情的位置列表为排序后的 uniqueLocations
        v.Locations = uniqueLocations
    }

    // 对 sortIDs 进行排序
    sort.Slice(sortIDs, func(i, j int) bool {
        return explainedPackageIsLess(idsToMatchDetails[sortIDs[i]], idsToMatchDetails[sortIDs[j]])
    })
    // 创建解释包列表
    var explainedPackages []*explainedPackage
    // 根据排序后的 ID 列表，将匹配详情添加到解释包列表中
    for _, k := range sortIDs {
        explainedPackages = append(explainedPackages, idsToMatchDetails[k])
    }
    return explainedPackages
}

// 比较两个解释包的大小
func explainedPackageIsLess(i, j *explainedPackage) bool {
    # 如果 i 的显示优先级不等于 j 的显示优先级
    if i.displayPriority != j.displayPriority {
        # 返回 i 的显示优先级是否大于 j 的显示优先级
        return i.displayPriority > j.displayPriority
    }
    # 如果 i 的名称小于 j 的名称
    return i.Name < j.Name
// 解释比赛细节，返回字符串
func explainMatchDetail(m models.Match, index int) string {
    // 如果比赛细节的长度小于等于索引，则返回空字符串
    if len(m.MatchDetails) <= index {
        return ""
    }
    // 获取比赛细节中索引位置的数据
    md := m.MatchDetails[index]
    explanation := ""
    // 根据不同的类型进行不同的解释
    switch md.Type {
    case string(match.CPEMatch):
        explanation = formatCPEExplanation(m)
    case string(match.ExactIndirectMatch):
        sourceName, sourceVersion := sourcePackageNameAndVersion(md)
        explanation = fmt.Sprintf("Indirect match; this CVE is reported against %s (version %s), the %s of this %s package.", sourceName, sourceVersion, nameForUpstream(string(m.Artifact.Type)), m.Artifact.Type)
    case string(match.ExactDirectMatch):
        explanation = fmt.Sprintf("Direct match (package name, version, and ecosystem) against %s (version %s).", m.Artifact.Name, m.Artifact.Version)
    }
    return explanation
}

// dedupeAndSortURLs返回一个DataSource字段的切片，去重并排序
// NVD和GHSA URL受到特殊对待；如果存在，则返回第一个和第二个
// 其余的按字符串排序
func (b *viewModelBuilder) dedupeAndSortURLs(primaryVulnerability models.VulnerabilityMetadata) []string {
    showFirst := primaryVulnerability.DataSource
    var URLs []string
    URLs = append(URLs, b.PrimaryMatch.Vulnerability.DataSource)
    for _, v := range b.PrimaryMatch.RelatedVulnerabilities {
        URLs = append(URLs, v.DataSource)
    }
    for _, m := range b.RelatedMatches {
        URLs = append(URLs, m.Vulnerability.DataSource)
        for _, v := range m.RelatedVulnerabilities {
            URLs = append(URLs, v.DataSource)
        }
    }
    var result []string
    deduplicate := make(map[string]bool)
    result = append(result, showFirst)
    deduplicate[showFirst] = true
    nvdURL := ""
    ghsaURL := ""
    # 遍历URLs列表中的每个URL
    for _, u := range URLs:
        # 如果URL以"https://nvd.nist.gov/vuln/detail"开头，则将其赋值给nvdURL
        if strings.HasPrefix(u, "https://nvd.nist.gov/vuln/detail"):
            nvdURL = u
        # 如果URL以"https://github.com/advisories"开头，则将其赋值给ghsaURL
        if strings.HasPrefix(u, "https://github.com/advisories"):
            ghsaURL = u
    # 如果nvdURL不为空且不等于showFirst，则将其添加到结果列表中，并在deduplicate字典中标记为true
    if nvdURL != "" && nvdURL != showFirst:
        result = append(result, nvdURL)
        deduplicate[nvdURL] = true
    # 如果ghsaURL不为空且不等于showFirst，则将其添加到结果列表中，并在deduplicate字典中标记为true
    if ghsaURL != "" && ghsaURL != showFirst:
        result = append(result, ghsaURL)
        deduplicate[ghsaURL] = true

    # 再次遍历URLs列表中的每个URL
    for _, u := range URLs:
        # 如果URL不在deduplicate字典中，则将其添加到结果列表中，并在deduplicate字典中标记为true
        if _, ok := deduplicate[u]; !ok:
            result = append(result, u)
            deduplicate[u] = true
    # 返回结果列表
    return result
# 解释位置信息，返回解释后的证据
func explainLocation(match models.Match, location file.Coordinates) explainedEvidence {
    # 获取真实路径
    path := location.RealPath
    # 如果匹配的文件元数据是 Java 类型，并且包含虚拟路径信息，则使用虚拟路径
    if javaMeta, ok := match.Artifact.Metadata.(map[string]any); ok {
        if virtPath, ok := javaMeta["virtualPath"].(string); ok {
            path = virtPath
        }
    }
    # 返回解释后的证据
    return explainedEvidence{
        Location:     path,
        ArtifactID:   match.Artifact.ID,
        ViaVulnID:    match.Vulnerability.ID,
        ViaNamespace: match.Vulnerability.Namespace,
    }
}

# 格式化 CPE 解释
func formatCPEExplanation(m models.Match) string {
    # 获取匹配细节中的搜索方式
    searchedBy := m.MatchDetails[0].SearchedBy
    # 如果搜索方式是 map 类型，则获取其中的 CPE 信息
    if mapResult, ok := searchedBy.(map[string]interface{}); ok {
        if cpes, ok := mapResult["cpes"]; ok {
            if cpeSlice, ok := cpes.([]interface{}); ok {
                # 如果 CPE 列表不为空，则返回格式化后的 CPE 匹配信息
                if len(cpeSlice) > 0 {
                    return fmt.Sprintf("CPE match on `%s`.", cpeSlice[0])
                }
            }
        }
    }
    # 如果没有匹配到 CPE 信息，则返回空字符串
    return ""
}

# 获取源包的名称和版本信息
func sourcePackageNameAndVersion(md models.MatchDetails) (string, string) {
    var name string
    var version string
    # 如果搜索方式是 map 类型，则获取其中的源包名称和版本信息
    if mapResult, ok := md.SearchedBy.(map[string]interface{}); ok {
        if sourcePackage, ok := mapResult["package"]; ok {
            if sourceMap, ok := sourcePackage.(map[string]interface{}); ok {
                if maybeName, ok := sourceMap["name"]; ok {
                    name, _ = maybeName.(string)
                }
                if maybeVersion, ok := sourceMap["version"]; ok {
                    version, _ = maybeVersion.(string)
                }
            }
        }
    }
    # 返回源包名称和版本信息
    return name, version
}

# 根据类型返回上游名称
func nameForUpstream(typ string) string {
    # 根据类型返回对应的上游名称
    switch typ {
    case "deb":
        return "origin"
    case "rpm":
        return "source RPM"
    }
    # 默认返回 upstream
    return "upstream"
}
```