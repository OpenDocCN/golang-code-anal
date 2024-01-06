# `grype\grype\presenter\explain\explain.go`

```
// 导入所需的包
package explain

import (
	_ "embed" // 使用 _ 表示只导入包，但不使用包内的变量或函数
	"fmt" // 导入 fmt 包，用于格式化输出
	"io" // 导入 io 包，用于输入输出操作
	"sort" // 导入 sort 包，用于排序
	"strings" // 导入 strings 包，用于字符串操作
	"text/template" // 导入 text/template 包，用于文本模板操作

	"github.com/anchore/grype/grype/match" // 导入 match 包，用于匹配
	"github.com/anchore/grype/grype/presenter/models" // 导入 models 包，用于展示模型
	"github.com/anchore/syft/syft/file" // 导入 file 包，用于文件操作
)

// 使用 embed 指令将 explain_cve.tmpl 文件内容嵌入到二进制文件中
// 这样在运行时可以直接访问文件内容，而不需要额外的文件读取操作
// 这里的 explainTemplate 是嵌入文件的内容
// 类型为 string
// 该变量用于存储模板内容
// 在后续的代码中可以直接使用该变量来访问模板内容
// 例如：template.New("explain_cve").Parse(explainTemplate)
var explainTemplate string

// 定义一个接口类型 VulnerabilityExplainer
// 该接口包含一个方法 ExplainByID
// 该方法接收一个字符串数组作为参数，返回一个错误
type VulnerabilityExplainer interface {
	ExplainByID(IDs []string) error
// 根据严重程度解释漏洞
ExplainBySeverity(severity string) error
// 解释所有漏洞
ExplainAll() error
}

// 视图模型，包含主要漏洞、相关漏洞、匹配的软件包、URLs
type ViewModel struct {
    PrimaryVulnerability   models.VulnerabilityMetadata
    RelatedVulnerabilities []models.VulnerabilityMetadata
    MatchedPackages        []*explainedPackage // 我认为这里需要一个软件包到解释证据的映射
    URLs                   []string
}

// 视图模型构建器，包含主要匹配、相关匹配、请求的漏洞ID
type viewModelBuilder struct {
    PrimaryMatch   models.Match // 似乎是我们试图解释的匹配
    RelatedMatches []models.Match
    requestedIDs   []string // 用户请求解释的漏洞ID
}

// 发现结果的映射，键为字符串，值为视图模型
type Findings map[string]ViewModel

// 解释的软件包
type explainedPackage struct {
```
// 以上是对给定代码的注释，解释了每个类型和方法的作用。
// 定义结构体 vulnerabilityExplainer，用于存储漏洞解释信息
type vulnerabilityExplainer struct {
    PURL                string  // 漏洞的 PURL
    Name                string  // 漏洞的名称
    Version             string  // 漏洞的版本
    MatchedOnID         string  // 匹配的 ID
    MatchedOnNamespace  string  // 匹配的命名空间
    IndirectExplanation string  // 间接解释
    DirectExplanation   string  // 直接解释
    CPEExplanation      string  // CPE 解释
    Locations           []explainedEvidence  // 存储解释的证据
    displayPriority     int // 显示优先级，直接匹配优先显示
}

// 定义结构体 explainedEvidence，用于存储解释的证据
type explainedEvidence struct {
    Location     string  // 位置
    ArtifactID   string  // 艺术品 ID
    ViaVulnID    string  // 通过漏洞 ID
    ViaNamespace string  // 通过命名空间
}
// 定义一个接口类型VulnerabilityExplainer，包含一个io.Writer和一个models.Document类型的指针
type VulnerabilityExplainer interface {
	ExplainByID(ids []string) error
}

// 创建一个新的VulnerabilityExplainer对象，传入io.Writer和models.Document类型的指针
func NewVulnerabilityExplainer(w io.Writer, doc *models.Document) VulnerabilityExplainer {
	return &vulnerabilityExplainer{
		w:   w,
		doc: doc,
	}
}

// 定义一个模板函数映射，包含一个名为"trim"的函数，用于去除字符串两端的空白字符
var funcs = template.FuncMap{
	"trim": strings.TrimSpace,
}

// 实现VulnerabilityExplainer接口的ExplainByID方法，根据给定的ids解释漏洞
func (e *vulnerabilityExplainer) ExplainByID(ids []string) error {
	// 调用Doc函数获取漏洞信息，存储在findings中
	findings, err := Doc(e.doc, ids)
	if err != nil {
		return err
	}
// 使用模板解析器创建一个模板对象，并将自定义的函数 funcs 注册到模板中，然后解析 explainTemplate 模板
t := template.Must(template.New("explanation").Funcs(funcs).Parse(explainTemplate))

// 遍历 ids 数组中的每个 id
for _, id := range ids {
    // 从 findings 中根据 id 获取对应的 finding
    finding, ok := findings[id]
    // 如果找不到对应的 finding，则继续下一次循环
    if !ok {
        continue
    }
    // 使用模板对象 t 执行模板，并将 finding 作为数据传入，将结果写入 e.w
    if err := t.Execute(e.w, finding); err != nil {
        // 如果执行模板过程中出现错误，则返回错误信息
        return fmt.Errorf("unable to execute template: %w", err)
    }
}
// 执行完毕，返回 nil
return nil
}

// 未实现的方法，返回错误信息
func (e *vulnerabilityExplainer) ExplainBySeverity(_ string) error {
    return fmt.Errorf("not implemented")
}

// 解释所有漏洞的方法
func (e *vulnerabilityExplainer) ExplainAll() error {
    // 使用 Doc 方法获取漏洞信息
    findings, err := Doc(e.doc, nil)
    // 如果获取漏洞信息过程中出现错误，则返回错误信息
    if err != nil {
		return err
	# 如果有错误发生，直接返回错误
	}
	t := template.Must(template.New("explanation").Funcs(funcs).Parse(explainTemplate))
	# 创建一个模板并解析模板字符串，如果出错则会panic
	return t.Execute(e.w, findings)
	# 使用模板将结果写入到给定的writer中

func Doc(doc *models.Document, requestedIDs []string) (Findings, error) {
	# 定义一个函数，接受一个Document类型的参数和一个字符串数组参数，返回Findings类型和error类型
	result := make(Findings)
	# 创建一个Findings类型的map
	builders := make(map[string]*viewModelBuilder)
	# 创建一个viewModelBuilder类型的map
	for _, m := range doc.Matches {
		# 遍历doc.Matches数组
		key := m.Vulnerability.ID
		# 获取Vulnerability的ID作为键
		existing, ok := builders[key]
		# 判断是否存在对应的viewModelBuilder
		if !ok {
			# 如果不存在，则创建一个新的viewModelBuilder
			existing = newBuilder(requestedIDs)
			builders[m.Vulnerability.ID] = existing
		}
		existing.WithMatch(m, requestedIDs)
		# 将匹配结果添加到对应的viewModelBuilder中
	}
	for _, m := range doc.Matches {
		# 再次遍历doc.Matches数组
# 遍历相关漏洞列表
for _, related := range m.RelatedVulnerabilities {
    # 获取相关漏洞的ID作为键
    key := related.ID
    # 检查是否存在对应ID的构建器
    existing, ok := builders[key]
    # 如果不存在对应ID的构建器，则创建一个新的构建器并添加到builders中
    if !ok {
        existing = newBuilder(requestedIDs)
        builders[key] = existing
    }
    # 使用相关漏洞和请求的ID更新构建器
    existing.WithMatch(m, requestedIDs)
}
# 遍历构建器列表，构建结果并添加到结果字典中
for k, v := range builders {
    result[k] = v.Build()
}
# 返回结果字典和空错误
return result, nil
}

# 创建一个新的视图模型构建器
func newBuilder(requestedIDs []string) *viewModelBuilder {
    return &viewModelBuilder{
        requestedIDs: requestedIDs,
    }
}
}

// WithMatch向构建器添加一个匹配项，接受足够的信息来确定匹配项是主要匹配还是相关匹配
func (b *viewModelBuilder) WithMatch(m models.Match, userRequestedIDs []string) {
	// 如果是主要匹配项，则将当前主要匹配项降级为相关匹配项（如果存在），然后添加新的主要匹配项
	if b.isPrimaryAdd(m, userRequestedIDs) {
		if b.PrimaryMatch.Vulnerability.ID != "" {
			b.WithRelatedMatch(b.PrimaryMatch)
		}
		b.WithPrimaryMatch(m)
	} else {
		b.WithRelatedMatch(m)
	}
}

// 判断是否是主要匹配项
func (b *viewModelBuilder) isPrimaryAdd(candidate models.Match, userRequestedIDs []string) bool {
	// 如果当前主要匹配项的漏洞ID为空，则返回true，表示是主要匹配项
	if b.PrimaryMatch.Vulnerability.ID == "" {
		return true
	}

	// 初始化一个标志，用于标记用户是否请求了特定的ID
	idWasRequested := false
	// 遍历用户请求的ID列表
	for _, id := range userRequestedIDs {
		// 如果候选漏洞的ID与用户请求的ID相匹配，则将标志设置为true，并跳出循环
		if candidate.Vulnerability.ID == id {
			idWasRequested = true
			break
		}
	}
	// 如果用户没有请求这个ID，并且用户请求的ID列表长度大于0，则返回false
	// 表示这个ID不是主要的ID
	if !idWasRequested && len(userRequestedIDs) > 0 {
		return false
	}
	// NVD CPEs是漏洞的一种规范ID，所以如果用户请求了CVE-YYYY-ID类型的ID
	// 并且我们有来自NVD的记录，则将其视为主要记录
	if candidate.Vulnerability.Namespace == "nvd:cpe" {
		return true
	}
	// 如果用户没有请求特定的ID，或者候选漏洞有一个用户请求的ID
	// 则继续处理与主要匹配相关的漏洞
	for _, related := range b.PrimaryMatch.RelatedVulnerabilities {
		# 如果相关的ID与候选漏洞的ID相等，则返回true
		if related.ID == candidate.Vulnerability.ID {
			return true
		}
	}
	# 如果没有找到相关的ID与候选漏洞的ID相等的情况，则返回false
	return false
}

# 将主要匹配添加到视图模型构建器中
func (b *viewModelBuilder) WithPrimaryMatch(m models.Match) *viewModelBuilder {
	b.PrimaryMatch = m
	return b
}

# 将相关匹配添加到视图模型构建器中
func (b *viewModelBuilder) WithRelatedMatch(m models.Match) *viewModelBuilder {
	b.RelatedMatches = append(b.RelatedMatches, m)
	return b
}

# 构建视图模型
func (b *viewModelBuilder) Build() ViewModel {
	# 将相关匹配和主要匹配合并，并进行分组和排序
	explainedPackages := groupAndSortEvidence(append(b.RelatedMatches, b.PrimaryMatch))
// 定义相关漏洞的数组
var relatedVulnerabilities []models.VulnerabilityMetadata
// 创建用于去重的相关漏洞映射
dedupeRelatedVulnerabilities := make(map[string]models.VulnerabilityMetadata)
// 创建用于排序的去重相关漏洞的数组
var sortDedupedRelatedVulnerabilities []string
// 遍历主要匹配和相关匹配的漏洞
for _, m := range append(b.RelatedMatches, b.PrimaryMatch) {
    // 生成漏洞的唯一键
    key := fmt.Sprintf("%s:%s", m.Vulnerability.Namespace, m.Vulnerability.ID)
    // 将漏洞信息存入去重映射
    dedupeRelatedVulnerabilities[key] = m.Vulnerability.VulnerabilityMetadata
    // 遍历相关漏洞
    for _, r := range m.RelatedVulnerabilities {
        // 生成相关漏洞的唯一键
        key := fmt.Sprintf("%s:%s", r.Namespace, r.ID)
        // 将相关漏洞信息存入去重映射
        dedupeRelatedVulnerabilities[key] = r
    }
}

// 从相关漏洞中删除主要漏洞，以避免重复列出
primary := b.primaryVulnerability()
delete(dedupeRelatedVulnerabilities, fmt.Sprintf("%s:%s", primary.Namespace, primary.ID))
// 将去重后的相关漏洞键存入排序数组
for k := range dedupeRelatedVulnerabilities {
    sortDedupedRelatedVulnerabilities = append(sortDedupedRelatedVulnerabilities, k)
}
// 对去重后的相关漏洞键进行排序
sort.Strings(sortDedupedRelatedVulnerabilities)
// 遍历排序后的去重相关漏洞键
for _, k := range sortDedupedRelatedVulnerabilities {
# 将dedupeRelatedVulnerabilities中的元素添加到relatedVulnerabilities切片中
relatedVulnerabilities = append(relatedVulnerabilities, dedupeRelatedVulnerabilities[k])

# 返回一个包含ViewModel结构体的对象
return ViewModel{
    # 设置PrimaryVulnerability字段为primary
    PrimaryVulnerability:   primary,
    # 设置RelatedVulnerabilities字段为relatedVulnerabilities
    RelatedVulnerabilities: relatedVulnerabilities,
    # 设置MatchedPackages字段为explainedPackages
    MatchedPackages:        explainedPackages,
    # 设置URLs字段为b.dedupeAndSortURLs(primary)的返回值
    URLs:                   b.dedupeAndSortURLs(primary),
}

# 定义一个方法primaryVulnerability，返回一个VulnerabilityMetadata类型的对象
func (b *viewModelBuilder) primaryVulnerability() models.VulnerabilityMetadata {
    # 定义一个VulnerabilityMetadata类型的变量primaryVulnerability
    var primaryVulnerability models.VulnerabilityMetadata
    # 遍历RelatedMatches和PrimaryMatch切片中的元素
    for _, m := range append(b.RelatedMatches, b.PrimaryMatch) {
        # 遍历每个元素的RelatedVulnerabilities和Vulnerability.VulnerabilityMetadata字段
        for _, r := range append(m.RelatedVulnerabilities, m.Vulnerability.VulnerabilityMetadata) {
            # 如果r的ID等于b.PrimaryMatch.Vulnerability.ID并且r的Namespace等于"nvd:cpe"，则将r赋值给primaryVulnerability
            if r.ID == b.PrimaryMatch.Vulnerability.ID && r.Namespace == "nvd:cpe" {
                primaryVulnerability = r
            }
        }
    }
}
// 如果主要漏洞的ID为空，则将主要漏洞设置为匹配的漏洞元数据
if primaryVulnerability.ID == "" {
    primaryVulnerability = b.PrimaryMatch.Vulnerability.VulnerabilityMetadata
}
// 返回主要漏洞
return primaryVulnerability
}

// nolint:funlen
// 对匹配进行分组和排序，返回解释的包列表
func groupAndSortEvidence(matches []models.Match) []*explainedPackage {
    // 创建一个映射，将ID与匹配细节关联起来
    idsToMatchDetails := make(map[string]*explainedPackage)
    // 遍历匹配列表
    for _, m := range matches {
        // 获取匹配的ID作为键
        key := m.Artifact.ID
        // 创建一个新的解释位置列表
        var newLocations []explainedEvidence
        // 遍历匹配的位置列表，解释每个位置
        for _, l := range m.Artifact.Locations {
            newLocations = append(newLocations, explainLocation(m, l))
        }
        // 初始化直接解释、间接解释、CPE解释和匹配类型优先级
        var directExplanation string
        var indirectExplanation string
        var cpeExplanation string
        var matchTypePriority int
        // 遍历匹配细节列表
        for i, md := range m.MatchDetails {
// 调用 explainMatchDetail 函数解释匹配细节，并将结果赋值给 explanation
explanation := explainMatchDetail(m, i)
// 如果 explanation 不为空
if explanation != "" {
    // 根据匹配类型进行不同的处理
    switch md.Type {
    // 如果是 CPE 匹配
    case string(match.CPEMatch):
        // 格式化 CPE 匹配的解释
        cpeExplanation = fmt.Sprintf("%s:%s %s", m.Vulnerability.Namespace, m.Vulnerability.ID, explanation)
        // 设置匹配类型优先级为 1，CPE 匹配是一种直接匹配
        matchTypePriority = 1
    // 如果是精确的间接匹配
    case string(match.ExactIndirectMatch):
        // 格式化间接匹配的解释
        indirectExplanation = fmt.Sprintf("%s:%s %s", m.Vulnerability.Namespace, m.Vulnerability.ID, explanation)
        // 设置匹配类型优先级为 0，显示间接匹配在直接匹配之后
        matchTypePriority = 0
    // 如果是精确的直接匹配
    case string(match.ExactDirectMatch):
        // 格式化直接匹配的解释
        directExplanation = fmt.Sprintf("%s:%s %s", m.Vulnerability.Namespace, m.Vulnerability.ID, explanation)
        // 设置匹配类型优先级为 2，精确的直接匹配是高置信度的直接匹配，优先显示
        matchTypePriority = 2
    }
}
// 根据 key 从 idsToMatchDetails 中获取值并赋给 e，如果不存在则创建一个新的 explainedPackage
e, ok := idsToMatchDetails[key]
if !ok {
    e = &explainedPackage{
        PURL:                m.Artifact.PURL,
        Name:                m.Artifact.Name,
# 设置版本号为 m.Artifact.Version
Version:             m.Artifact.Version,
# 设置匹配到的漏洞ID为 m.Vulnerability.ID
MatchedOnID:         m.Vulnerability.ID,
# 设置匹配到的漏洞命名空间为 m.Vulnerability.Namespace
MatchedOnNamespace:  m.Vulnerability.Namespace,
# 设置直接解释为 directExplanation
DirectExplanation:   directExplanation,
# 设置间接解释为 indirectExplanation
IndirectExplanation: indirectExplanation,
# 设置CPE解释为 cpeExplanation
CPEExplanation:      cpeExplanation,
# 设置位置为 newLocations
Locations:           newLocations,
# 设置显示优先级为 matchTypePriority
displayPriority:     matchTypePriority,
# 如果 key 不存在于 idsToMatchDetails 中，则创建新的匹配详情对象
idsToMatchDetails[key] = e
# 如果 key 存在于 idsToMatchDetails 中，则更新匹配详情对象
e.Locations = append(e.Locations, newLocations...)
# 如果 CPEExplanation 为空，则设置为 cpeExplanation
if e.CPEExplanation == "" {
    e.CPEExplanation = cpeExplanation
}
# 如果 IndirectExplanation 为空，则设置为 indirectExplanation
if e.IndirectExplanation == "" {
    e.IndirectExplanation = indirectExplanation
}
# 增加显示优先级
e.displayPriority += matchTypePriority
	}
	// 声明一个字符串切片用于存储排序后的ID
	var sortIDs []string
	// 遍历idsToMatchDetails中的键值对
	for k, v := range idsToMatchDetails {
		// 将键添加到sortIDs切片中
		sortIDs = append(sortIDs, k)
		// 创建一个空的地点去重映射
		dedupeLocations := make(map[string]explainedEvidence)
		// 遍历v.Locations中的地点，将其添加到地点去重映射中
		for _, l := range v.Locations {
			dedupeLocations[l.Location] = l
		}
		// 声明一个空的唯一地点切片
		var uniqueLocations []explainedEvidence
		// 遍历地点去重映射，将其添加到唯一地点切片中
		for _, l := range dedupeLocations {
			uniqueLocations = append(uniqueLocations, l)
		}
		// 对唯一地点切片进行排序
		sort.Slice(uniqueLocations, func(i, j int) bool {
			if uniqueLocations[i].ViaNamespace == uniqueLocations[j].ViaNamespace {
				return uniqueLocations[i].Location < uniqueLocations[j].Location
			}
			return uniqueLocations[i].ViaNamespace < uniqueLocations[j].ViaNamespace
		})
		// 更新v.Locations为排序后的唯一地点切片
		v.Locations = uniqueLocations
	}
# 使用sort.Slice对sortIDs进行排序，排序规则为explainedPackageIsLess函数的返回值
sort.Slice(sortIDs, func(i, j int) bool {
    return explainedPackageIsLess(idsToMatchDetails[sortIDs[i]], idsToMatchDetails[sortIDs[j]])
})

# 创建一个空的explainedPackages切片
var explainedPackages []*explainedPackage

# 遍历sortIDs，将idsToMatchDetails中对应的值添加到explainedPackages切片中
for _, k := range sortIDs {
    explainedPackages = append(explainedPackages, idsToMatchDetails[k])
}

# 返回explainedPackages切片
return explainedPackages
}

# 判断两个explainedPackage对象的优先级，如果优先级不同则按照优先级排序，否则按照名称排序
func explainedPackageIsLess(i, j *explainedPackage) bool {
    if i.displayPriority != j.displayPriority {
        return i.displayPriority > j.displayPriority
    }
    return i.Name < j.Name
}

# 根据索引获取匹配细节，如果索引超出范围则返回空字符串
func explainMatchDetail(m models.Match, index int) string {
    if len(m.MatchDetails) <= index {
		// 如果没有匹配的类型，则返回空字符串
		return ""
	}
	// 获取匹配详情
	md := m.MatchDetails[index]
	// 初始化解释字符串
	explanation := ""
	// 根据匹配类型进行不同的处理
	switch md.Type {
	case string(match.CPEMatch):
		// 如果是 CPE 匹配类型，则调用 formatCPEExplanation 函数生成解释
		explanation = formatCPEExplanation(m)
	case string(match.ExactIndirectMatch):
		// 如果是精确间接匹配类型，则获取源包名称和版本
		sourceName, sourceVersion := sourcePackageNameAndVersion(md)
	case string(match.ExactDirectMatch):
		// 如果是精确直接匹配类型，则生成相应的解释
		explanation = fmt.Sprintf("Direct match (package name, version, and ecosystem) against %s (version %s).", m.Artifact.Name, m.Artifact.Version)
	}
	// 返回解释字符串
	return explanation
}

// dedupeAndSortURLs 返回一个去重且排序后的 DataSource 字段的切片
// NVD 和 GHSA URL 会被特殊处理，如果存在则会排在前面
// 其他 URL 则按照字符串排序
func (b *viewModelBuilder) dedupeAndSortURLs(primaryVulnerability models.VulnerabilityMetadata) []string {
	// 获取主要漏洞的数据源
	showFirst := primaryVulnerability.DataSource
# 定义一个字符串数组用于存储 URL
var URLs []string
# 将主要匹配漏洞的数据源添加到 URL 数组中
URLs = append(URLs, b.PrimaryMatch.Vulnerability.DataSource)
# 遍历主要匹配漏洞的相关漏洞，将数据源添加到 URL 数组中
for _, v := range b.PrimaryMatch.RelatedVulnerabilities {
    URLs = append(URLs, v.DataSource)
}
# 遍历相关匹配，将漏洞的数据源添加到 URL 数组中
for _, m := range b.RelatedMatches {
    URLs = append(URLs, m.Vulnerability.DataSource)
    # 遍历相关匹配的相关漏洞，将数据源添加到 URL 数组中
    for _, v := range m.RelatedVulnerabilities {
        URLs = append(URLs, v.DataSource)
    }
}
# 定义一个字符串数组用于存储结果
var result []string
# 定义一个布尔类型的 map 用于去重
deduplicate := make(map[string]bool)
# 将 showFirst 添加到结果数组中
result = append(result, showFirst)
# 将 showFirst 添加到去重 map 中
deduplicate[showFirst] = true
# 初始化 nvdURL 和 ghsaURL
nvdURL := ""
ghsaURL := ""
# 遍历 URL 数组
for _, u := range URLs:
    # 如果 URL 以 "https://nvd.nist.gov/vuln/detail" 开头，则将其赋值给 nvdURL
    if strings.HasPrefix(u, "https://nvd.nist.gov/vuln/detail") {
        nvdURL = u
		}
		// 如果 URL 以 "https://github.com/advisories" 开头，则将其赋值给 ghsaURL
		if strings.HasPrefix(u, "https://github.com/advisories") {
			ghsaURL = u
		}
	}
	// 如果 nvdURL 不为空且不等于 showFirst，则将其添加到结果列表中，并标记为已处理
	if nvdURL != "" && nvdURL != showFirst {
		result = append(result, nvdURL)
		deduplicate[nvdURL] = true
	}
	// 如果 ghsaURL 不为空且不等于 showFirst，则将其添加到结果列表中，并标记为已处理
	if ghsaURL != "" && ghsaURL != showFirst {
		result = append(result, ghsaURL)
		deduplicate[ghsaURL] = true
	}

	// 遍历 URLs 列表，如果 URL 不在 deduplicate 中，则将其添加到结果列表中，并标记为已处理
	for _, u := range URLs {
		if _, ok := deduplicate[u]; !ok {
			result = append(result, u)
			deduplicate[u] = true
		}
	}
// 返回结果
return result
}

// 解释匹配的位置信息
func explainLocation(match models.Match, location file.Coordinates) explainedEvidence {
	// 获取真实路径
	path := location.RealPath
	// 如果匹配的文件元数据是 Java 类型，并且包含虚拟路径信息，则使用虚拟路径
	if javaMeta, ok := match.Artifact.Metadata.(map[string]any); ok {
		if virtPath, ok := javaMeta["virtualPath"].(string); ok {
			path = virtPath
		}
	}
	// 返回解释后的证据
	return explainedEvidence{
		Location:     path,
		ArtifactID:   match.Artifact.ID,
		ViaVulnID:    match.Vulnerability.ID,
		ViaNamespace: match.Vulnerability.Namespace,
	}
}

// 格式化 CPE 解释
func formatCPEExplanation(m models.Match) string {
	// 获取匹配的搜索方式
	searchedBy := m.MatchDetails[0].SearchedBy
// 如果 searchedBy 是 map 类型，则进入条件判断
if mapResult, ok := searchedBy.(map[string]interface{}); ok {
    // 如果 mapResult 中包含名为 "cpes" 的键，则进入条件判断
    if cpes, ok := mapResult["cpes"]; ok {
        // 如果 cpes 是 []interface{} 类型，则进入条件判断
        if cpeSlice, ok := cpes.([]interface{}); ok {
            // 如果 cpeSlice 长度大于 0，则返回第一个元素的格式化字符串
            if len(cpeSlice) > 0 {
                return fmt.Sprintf("CPE match on `%s`.", cpeSlice[0])
            }
        }
    }
}
// 如果以上条件都不满足，则返回空字符串

// 根据传入的 MatchDetails 结构体获取源包的名称和版本
func sourcePackageNameAndVersion(md models.MatchDetails) (string, string) {
    var name string
    var version string
    // 如果 SearchedBy 是 map 类型，则进入条件判断
    if mapResult, ok := md.SearchedBy.(map[string]interface{}); ok {
        // 如果 mapResult 中包含名为 "package" 的键，则进入条件判断
        if sourcePackage, ok := mapResult["package"]; ok {
            // 如果 sourcePackage 是 map 类型，则进入条件判断
            if sourceMap, ok := sourcePackage.(map[string]interface{}); ok {
                // 如果 sourceMap 中包含名为 "name" 的键，则获取其值赋给 name
                if maybeName, ok := sourceMap["name"]; ok {
                    name, _ = maybeName.(string)
// 如果sourceMap中存在"version"键，则将其值转换为string类型并赋值给version变量
if maybeVersion, ok := sourceMap["version"]; ok {
    version, _ = maybeVersion.(string)
}

// 根据不同的typ类型返回对应的name
func nameForUpstream(typ string) string {
    switch typ {
    case "deb":
        return "origin"
    case "rpm":
        return "source RPM"
    }
    return "upstream"
}
```