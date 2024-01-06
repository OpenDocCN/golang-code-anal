# `grype\grype\presenter\sarif\presenter.go`

```
// 导入 sarif 包
package sarif

// 导入 fmt、io、filepath 和 strings 包
import (
	"fmt"
	"io"
	"path/filepath"
	"strings"
	
	// 导入 go-sarif/sarif 包
	"github.com/owenrumney/go-sarif/sarif"
	
	// 导入 anchore/clio 包
	"github.com/anchore/clio"
	// 导入 anchore/grype/grype/db/v5 包
	v5 "github.com/anchore/grype/grype/db/v5"
	// 导入 anchore/grype/grype/match 包
	"github.com/anchore/grype/grype/match"
	// 导入 anchore/grype/grype/pkg 包
	"github.com/anchore/grype/grype/pkg"
	// 导入 anchore/grype/grype/presenter/models 包
	"github.com/anchore/grype/grype/presenter/models"
	// 导入 anchore/grype/grype/vulnerability 包
	"github.com/anchore/grype/grype/vulnerability"
	// 导入 anchore/syft/syft/file 包
	"github.com/anchore/syft/syft/file"
	// 导入 anchore/syft/syft/source 包
	"github.com/anchore/syft/syft/source"
)
// Presenter 结构体用于保存生成报告所需的数据，并实现了 presenter.Presenter 接口
type Presenter struct {
	id               clio.Identification  // 用于标识报告的 ID
	results          match.Matches        // 匹配结果
	packages         []pkg.Package         // 包列表
	src              *source.Description   // 源描述
	metadataProvider vulnerability.MetadataProvider  // 漏洞元数据提供者
}

// NewPresenter 是 Presenter 的构造函数，返回一个 *Presenter 指针
func NewPresenter(pb models.PresenterConfig) *Presenter {
	// 使用传入的 PresenterConfig 创建一个 Presenter 对象并返回其指针
	return &Presenter{
		id:               pb.ID,  // 初始化 ID
		results:          pb.Matches,  // 初始化匹配结果
		packages:         pb.Packages,  // 初始化包列表
		metadataProvider: pb.MetadataProvider,  // 初始化漏洞元数据提供者
		src:              pb.Context.Source,  // 初始化源描述
	}
}
// Present函数创建一个基于SARIF的报告
func (pres *Presenter) Present(output io.Writer) error {
    // 将数据转换为SARIF报告
    doc, err := pres.toSarifReport()
    if err != nil {
        return err
    }
    // 将SARIF报告以美观的方式写入输出流
    err = doc.PrettyWrite(output)
    return err
}

// toSarifReport函数输出一个sarif报告对象
func (pres *Presenter) toSarifReport() (*sarif.Report, error) {
    // 创建一个SARIF报告对象
    doc, err := sarif.New(sarif.Version210)
    if err != nil {
        return nil, err
    }

    // 获取版本号
    v := pres.id.Version
    if v == "[not provided]" || v == "" {
        // 需要一个语义版本号才能通过MS SARIF验证器
		v = "0.0.0-dev"  // 设置变量 v 的值为 "0.0.0-dev"

	}

	doc.AddRun(&sarif.Run{  // 向 SARIF 文档中添加运行信息
		Tool: sarif.Tool{  // 设置工具信息
			Driver: &sarif.ToolComponent{  // 设置驱动信息
				Name:           pres.id.Name,  // 设置驱动名称为 pres.id.Name
				Version:        sp(v),  // 设置驱动版本为 v 的值
				InformationURI: sp("https://github.com/anchore/grype"),  // 设置信息链接
				Rules:          pres.sarifRules(),  // 设置规则信息
			},
		},
		Results: pres.sarifResults(),  // 设置运行结果信息
	})

	return doc, nil  // 返回 SARIF 文档和空值
}

// sarifRules generates the set of rules to include in this run
func (pres *Presenter) sarifRules() (out []*sarif.ReportingDescriptor) {  // 生成要包含在此运行中的规则集合
	// 检查结果集中是否有数据
	if pres.results.Count() > 0 {
		// 创建一个规则ID的布尔值映射
		ruleIDs := map[string]bool{}

		// 遍历结果集中的数据
		for _, m := range pres.results.Sorted() {
			// 获取规则ID
			ruleID := pres.ruleID(m)
			// 如果规则ID已经存在于映射中，则跳过
			if ruleIDs[ruleID] {
				// 在这里，我们只输出有关漏洞的信息，而不是匹配它们的位置
				continue
			}

			// 将规则ID添加到映射中
			ruleIDs[ruleID] = true

			// 获取漏洞的链接和元数据
			link := m.Vulnerability.ID
			meta := pres.metadata(m)
			if meta != nil {
				// 根据元数据的不同情况处理链接
				switch {
				case meta.DataSource != "":
					link = fmt.Sprintf("[%s](%s)", meta.ID, meta.DataSource)
				case len(meta.URLs) > 0:
				link = fmt.Sprintf("[%s](%s)", meta.ID, meta.URLs[0])
				// 根据 meta.ID 和 meta.URLs[0] 格式化生成链接字符串

			}

			out = append(out, &sarif.ReportingDescriptor{
				ID:      ruleID,
				Name:    sp(ruleName(m)),
				HelpURI: sp("https://github.com/anchore/grype"),
				// SARIF 报告的标题
				ShortDescription: &sarif.MultiformatMessageString{
					Text: sp(pres.shortDescription(m)),
				},
				// SARIF 报告的副标题
				FullDescription: &sarif.MultiformatMessageString{
					Text: sp(pres.subtitle(m)),
				},
				Help: pres.helpText(m, link),
				Properties: sarif.Properties{
					// 用于 GitHub reportingDescriptor 对象的属性
					// 参考：https://docs.github.com/en/code-security/code-scanning/integrating-with-code-scanning/sarif-support-for-code-scanning#reportingdescriptor-object
// security-severity: 为给定匹配项创建安全严重性值
func (pres *Presenter) securitySeverity(m match.Match) *sarif.MultiformatMessageString {
	// 创建包含安全严重性值的消息字符串
	return &sarif.MultiformatMessageString{
		Text: fmt.Sprintf("%d", pres.securitySeverityValue(m)),
	}
}

// ruleID: 为给定匹配项创建唯一的规则ID
func (pres *Presenter) ruleID(m match.Match) string {
	// TODO 如果我们支持配置，我们可能希望允许添加另一个限定符，以便如果在多个容器上运行多个漏洞扫描，我们可以为每个识别出唯一的规则
	return fmt.Sprintf("%s-%s", m.Vulnerability.ID, m.Package.Name)
}

// helpText: 获取规则的帮助文本，这在GitHub上点击漏洞列表中的标题时显示
func (pres *Presenter) helpText(m match.Match, link string) *sarif.MultiformatMessageString {
	// TODO 我们不一定需要在这里添加位置，可能有多个引用相同的漏洞
	// 如果在图像中找到多个引用，我们可以添加一些受影响位置的列表，例如，但如果特定分支有多个漏洞扫描，这可能会变得更复杂
// 根据给定的漏洞信息和包信息生成文本信息
text := fmt.Sprintf("Vulnerability %s\nSeverity: %s\nPackage: %s\nVersion: %s\nFix Version: %s\nType: %s\nLocation: %s\nData Namespace: %s\nLink: %s",
	m.Vulnerability.ID, pres.severityText(m), m.Package.Name, m.Package.Version, fixVersions(m), m.Package.Type, pres.packagePath(m.Package), m.Vulnerability.Namespace, link,
)
// 根据给定的漏洞信息和包信息生成 Markdown 格式的信息
markdown := fmt.Sprintf(
	"**Vulnerability %s**\n"+
	"| Severity | Package | Version | Fix Version | Type | Location | Data Namespace | Link |\n"+
	"| --- | --- | --- | --- | --- | --- | --- | --- |\n"+
	"| %s  | %s  | %s  | %s  | %s  | %s  | %s  | %s  |\n",
	m.Vulnerability.ID, pres.severityText(m), m.Package.Name, m.Package.Version, fixVersions(m), m.Package.Type, pres.packagePath(m.Package), m.Vulnerability.Namespace, link,
)
// 返回包含文本和 Markdown 格式信息的 MultiformatMessageString 结构体指针
return &sarif.MultiformatMessageString{
	Text:     &text,
	Markdown: &markdown,
}
}

// packagePath 尝试获取包相对于“扫描根目录”的相对路径
func (pres *Presenter) packagePath(p pkg.Package) string {
	// 将包的位置转换为切片
	locations := p.Locations.ToSlice()
	// 如果位置切片长度大于 0
	if len(locations) > 0 {
		// 返回给定位置的路径
		return pres.locationPath(locations[0])
	}
	// 返回输入路径
	return pres.inputPath()
}

// inputPath 根据输入返回更友好的相对路径或绝对路径，不以.或./开头
func (pres *Presenter) inputPath() string {
	// 如果源为空，则返回空字符串
	if pres.src == nil {
		return ""
	}
	var inputPath string
	// 根据源的类型设置输入路径
	switch m := pres.src.Metadata.(type) {
	case source.FileSourceMetadata:
		inputPath = m.Path
	case source.DirectorySourceMetadata:
		inputPath = m.Path
	default:
		return ""
	}
	// 去除路径中的./前缀
	inputPath = strings.TrimPrefix(inputPath, "./")
// 如果输入路径为当前目录，则返回空字符串
if inputPath == "." {
    return ""
}
// 返回输入路径
return inputPath
}

// locationPath 返回相对于当前工作目录的位置路径
func (pres *Presenter) locationPath(l file.Location) string {
    // 获取文件的路径
    path := l.Path()
    // 获取输入路径
    in := pres.inputPath()
    // 去掉路径中的 "./"
    path = strings.TrimPrefix(path, "./")
    // 如果存在源文件，并且源文件的元数据是目录源元数据
    if pres.src != nil {
        _, ok := pres.src.Metadata.(source.DirectorySourceMetadata)
        if ok {
            // 如果路径是绝对路径，或者输入路径为空，则返回路径
            if filepath.IsAbs(path) || in == "" {
                return path
            }
            // 如果路径不是绝对路径，则返回相对于当前工作目录的路径
            return fmt.Sprintf("%s/%s", in, path)
		}
	}

	return path
}
// 返回路径

// locations the locations array is a single "physical" location with potentially multiple logical locations
// locations 函数返回一个包含多个逻辑位置的单个“物理”位置
func (pres *Presenter) locations(m match.Match) []*sarif.Location {
	// 获取匹配项所在包的物理位置
	physicalLocation := pres.packagePath(m.Package)

	var logicalLocations []*sarif.LogicalLocation

	// 根据不同的源类型，处理逻辑位置
	switch metadata := pres.src.Metadata.(type) {
	case source.StereoscopeImageSourceMetadata:
		// 获取用户输入的图像信息
		img := metadata.UserInput
		// 将位置转换为切片
		locations := m.Package.Locations.ToSlice()
		// 遍历位置
		for _, l := range locations {
			// 去除路径前缀中的斜杠
			trimmedPath := strings.TrimPrefix(pres.locationPath(l), "/")
			// 添加逻辑位置到逻辑位置数组
			logicalLocations = append(logicalLocations, &sarif.LogicalLocation{
				// 构建完全限定名
				FullyQualifiedName: sp(fmt.Sprintf("%s@%s:/%s", img, l.FileSystemID, trimmedPath)),
// 对象为文件源元数据时，遍历包含的位置信息，将每个位置信息转换为逻辑位置对象并添加到逻辑位置列表中
// 逻辑位置对象包括完全限定名称和名称
// 完全限定名称由路径和位置的真实路径组成
// 名称为位置的真实路径
// 该操作用于处理文件源元数据的位置信息
// 位置信息可能包含文件路径和真实路径
// 位置信息将被转换为逻辑位置对象并添加到逻辑位置列表中
// 逻辑位置对象包括完全限定名称和名称
// 完全限定名称由路径和位置的真实路径组成
// 名称为位置的真实路径
// 该操作用于处理文件源元数据的位置信息
// 位置信息可能包含文件路径和真实路径
// 位置信息将被转换为逻辑位置对象并添加到逻辑位置列表中
// 逻辑位置对象包括完全限定名称和名称
// 完全限定名称由路径和位置的真实路径组成
// 名称为位置的真实路径
// 该操作用于处理文件源元数据的位置信息
// 位置信息可能包含文件路径和真实路径
// 位置信息将被转换为逻辑位置对象并添加到逻辑位置列表中
// 逻辑位置对象包括完全限定名称和名称
// 完全限定名称由路径和位置的真实路径组成
// 名称为位置的真实路径
// 该操作用于处理文件源元数据的位置信息
// 位置信息可能包含文件路径和真实路径
// 位置信息将被转换为逻辑位置对象并添加到逻辑位置列表中
// 逻辑位置对象包括完全限定名称和名称
// 完全限定名称由路径和位置的真实路径组成
// 名称为位置的真实路径
// 该操作用于处理文件源元数据的位置信息
// 位置信息可能包含文件路径和真实路径
// 位置信息将被转换为逻辑位置对象并添加到逻辑位置列表中
// 逻辑位置对象包括完全限定名称和名称
// 完全限定名称由路径和位置的真实路径组成
// 名称为位置的真实路径
// 该操作用于处理文件源元数据的位置信息
// 位置信息可能包含文件路径和真实路径
// 位置信息将被转换为逻辑位置对象并添加到逻辑位置列表中
// 逻辑位置对象包括完全限定名称和名称
// 完全限定名称由路径和位置的真实路径组成
// 名称为位置的真实路径
// 该操作用于处理文件源元数据的位置信息
// 位置信息可能包含文件路径和真实路径
// 位置信息将被转换为逻辑位置对象并添加到逻辑位置列表中
// 逻辑位置对象包括完全限定名称和名称
// 完全限定名称由路径和位置的真实路径组成
// 名称为位置的真实路径
// 该操作用于处理文件源元数据的位置信息
// 位置信息可能包含文件路径和真实路径
// 位置信息将被转换为逻辑位置对象并添加到逻辑位置列表中
// 逻辑位置对象包括完全限定名称和名称
// 完全限定名称由路径和位置的真实路径组成
// 名称为位置的真实路径
// 该操作用于处理文件源元数据的位置信息
// 位置信息可能包含文件路径和真实路径
// 位置信息将被转换为逻辑位置对象并添加到逻辑位置列表中
// 逻辑位置对象包括完全限定名称和名称
// 完全限定名称由路径和位置的真实路径组成
// 名称为位置的真实路径
// 该操作用于处理文件源元数据的位置信息
// 位置信息可能包含文件路径和真实路径
// 位置信息将被转换为逻辑位置对象并添加到逻辑位置列表中
// 逻辑位置对象包括完全限定名称和名称
// 完全限定名称由路径和位置的真实路径组成
// 名称为位置的真实路径
// 该操作用于处理文件源元数据的位置信息
// 位置信息可能包含文件路径和真实路径
// 位置信息将被转换为逻辑位置对象并添加到逻辑位置列表中
// 逻辑位置对象包括完全限定名称和名称
// 完全限定名称由路径和位置的真实路径组成
// 名称为位置的真实路径
// 该操作用于处理文件源元数据的位置信息
// 位置信息可能包含文件路径和真实路径
// 位置信息将被转换为逻辑位置对象并添加到逻辑位置列表中
// 逻辑位置对象包括完全限定名称和名称
// 完全限定名称由路径和位置的真实路径组成
// 名称为位置的真实路径
// 该操作用于处理文件源元数据的位置信息
// 位置信息可能包含文件路径和真实路径
// 位置信息将被转换为逻辑位置对象并添加到逻辑位置列表中
// 逻辑位置对象包拼完全限定名称和名称
// 完全限定名称由路径和位置的真实路径组成
// 名称为位置的真实路径
// 该操作用于处理文件源元数据的位置信息
// 位置信息可能包含文件路径和真实路径
// 位置信息将被转换为逻辑位置对象并添加到逻辑位置列表中
# 返回一个包含sarif.Location对象的空列表
return []*sarif.Location{
	{
		# 设置物理位置的URI
		PhysicalLocation: &sarif.PhysicalLocation{
			ArtifactLocation: &sarif.ArtifactLocation{
				URI: sp(physicalLocation),
			},
			# 设置区域的起始和结束行、列
			Region: &sarif.Region{
				StartLine:   ip(1),
				StartColumn: ip(1),
				EndLine:     ip(1),
				EndColumn:   ip(1),
			},
		},
		# 设置逻辑位置
		LogicalLocations: logicalLocations,
	},
}
// severityText函数根据匹配的严重程度级别提供匹配的文本表示
func (pres *Presenter) severityText(m match.Match) string {
    // 获取匹配的元数据
    meta := pres.metadata(m)
    // 如果元数据不为空
    if meta != nil {
        // 解析严重程度级别
        severity := vulnerability.ParseSeverity(meta.Severity)
        // 根据严重程度级别返回相应的文本表示
        switch severity {
        case vulnerability.CriticalSeverity:
            return "critical"
        case vulnerability.HighSeverity:
            return "high"
        case vulnerability.MediumSeverity:
            return "medium"
        }
    }
    // 如果没有匹配的严重程度级别，则返回"low"
    return "low"
}

// cvssScore函数尝试获取我们的漏洞数据包含的最佳CVSS分数
func (pres *Presenter) cvssScore(v vulnerability.Vulnerability) float64 {
    // 创建一个空的元数据切片
    var all []*vulnerability.Metadata
// 从metadataProvider获取指定ID和Namespace的元数据，存储在meta中，如果没有错误并且meta不为空，则将其添加到all切片中
meta, err := pres.metadataProvider.GetMetadata(v.ID, v.Namespace)
if err == nil && meta != nil {
    all = append(all, meta)
}

// 遍历v.RelatedVulnerabilities中的每个元素，获取其相关的元数据，如果没有错误并且meta不为空，则将其添加到all切片中
for _, related := range v.RelatedVulnerabilities {
    meta, err = pres.metadataProvider.GetMetadata(related.ID, related.Namespace)
    if err == nil && meta != nil {
        all = append(all, meta)
    }
}

// 初始化score为-1.0
score := -1.0

// 首先检查vendor-specific条目，如果Namespace为"nvd:cpe"则跳过当前循环
for _, m := range all {
    if m.Namespace == "nvd:cpe" {
        continue
    }
}
# 遍历 m.Cvss 列表中的每个元素，将其赋值给变量 cvss
for _, cvss := range m.Cvss:
    # 如果 cvss.Metrics.BaseScore 大于 score，则将 score 更新为 cvss.Metrics.BaseScore
    if cvss.Metrics.BaseScore > score:
        score = cvss.Metrics.BaseScore

# 如果 score 大于 0，则返回 score
if score > 0:
    return score

# 接下来，检查所有条目中的 nvd 条目
# 遍历 all 列表中的每个元素，将其赋值给变量 m
for _, m := range all:
    # 遍历 m.Cvss 列表中的每个元素，将其赋值给变量 cvss
    for _, cvss := range m.Cvss:
        # 如果 cvss.Metrics.BaseScore 大于 score，则将 score 更新为 cvss.Metrics.BaseScore
        if cvss.Metrics.BaseScore > score:
            score = cvss.Metrics.BaseScore
// 返回安全性评估值
func (pres *Presenter) securitySeverityValue(m match.Match) string {
    // 获取匹配项的元数据
    meta := pres.metadata(m)
    if meta != nil {
        // 如果存在 CVSS 分数，则直接返回
        score := pres.cvssScore(m.Vulnerability)
        if score > 0 {
            return fmt.Sprintf("%.1f", score)
        }
        // 解析漏洞严重程度
        severity := vulnerability.ParseSeverity(meta.Severity)
        switch severity {
        case vulnerability.CriticalSeverity:
            return "9.0"
        case vulnerability.HighSeverity:
            return "7.0"
        case vulnerability.MediumSeverity:
            // 返回中等级别的安全性评估值
// 返回字符串"4.0"
		return "4.0"
// 根据漏洞严重程度返回相应的字符串
		case vulnerability.LowSeverity:
// 返回字符串"1.0"
			return "1.0"
		}
	}
// 返回匹配的漏洞元数据，如果未找到或出现错误则返回nil
func (pres *Presenter) metadata(m match.Match) *vulnerability.Metadata {
// 从提供程序获取匹配的漏洞元数据，如果未找到则返回nil
	meta, _ := pres.metadataProvider.GetMetadata(m.Vulnerability.ID, m.Vulnerability.Namespace)
// 返回元数据
	return meta
}

// 为给定的匹配生成副标题
func (pres *Presenter) subtitle(m match.Match) string {
// 获取匹配的漏洞元数据
	meta := pres.metadata(m)
// 如果元数据不为空，则生成副标题
	if meta != nil {
// 使用元数据的描述生成副标题
		subtitle := meta.Description
// 如果副标题不为空，则返回副标题
		if subtitle != "" {
// 返回匹配的漏洞的简要描述
func (pres *Presenter) shortDescription(m match.Match) string {
	// 如果漏洞有子标题，则返回子标题
	if subtitle := m.Vulnerability.Subtitle; subtitle != "" {
		return subtitle
	}

	// 获取修复版本信息
	fixVersion := fixVersions(m)
	// 如果有可用的修复版本，则返回包含修复版本信息的字符串
	if fixVersion != "" {
		return fmt.Sprintf("Version %s is affected with an available fix in versions %s", m.Package.Version, fixVersion)
	}

	// 如果没有可用的修复版本，则返回未报告修复的字符串
	return fmt.Sprintf("Version %s is affected with no fixes reported yet.", m.Package.Version)
}

// 获取修复版本信息
func fixVersions(m match.Match) string {
	// 如果漏洞的修复状态为已修复，并且修复版本列表不为空，则返回修复版本列表的字符串
	if m.Vulnerability.Fix.State == v5.FixedState && len(m.Vulnerability.Fix.Versions) > 0 {
		return strings.Join(m.Vulnerability.Fix.Versions, ",")
	}
	// 如果修复状态不为已修复或修复版本列表为空，则返回空字符串
	return ""
}
// 根据给定的信息格式化输出字符串，包括漏洞ID、严重程度和软件包名称
return fmt.Sprintf("%s %s vulnerability for %s package", m.Vulnerability.ID, pres.severityText(m), m.Package.Name)
}

// 生成 SARIF 格式的扫描结果
func (pres *Presenter) sarifResults() []*sarif.Result {
    out := make([]*sarif.Result, 0) // 确保至少有一个空数组
    for _, m := range pres.results.Sorted() {
        out = append(out, &sarif.Result{
            RuleID:  sp(pres.ruleID(m)),
            Message: pres.resultMessage(m),
            // 根据 SARIF 规范，应该使用 AnalysisTarget.URI 来指示逻辑文件，如 "Dockerfile"，但 GitHub 与此不兼容
            // FIXME github "requires" partialFingerprints
            // PartialFingerprints: ???
            Locations: pres.locations(m),
        })
    }
    return out
}

// 根据提供的值返回一个整数指针
// ip 返回一个指向整数的指针
func ip(i int) *int {
	return &i
}

// sp 根据提供的值返回一个字符串指针
func sp(sarif string) *string {
	return &sarif
}

// resultMessage 根据匹配结果生成消息
func (pres *Presenter) resultMessage(m match.Match) sarif.Message {
    // 获取包路径
	path := pres.packagePath(m.Package)
    // 根据匹配结果生成消息
	message := fmt.Sprintf("The path %s reports %s at version %s ", path, m.Package.Name, m.Package.Version)

    // 如果源数据是目录源元数据
	if _, ok := pres.src.Metadata.(source.DirectorySourceMetadata); ok {
		// 如果是目录源元数据，则生成相应消息
		message = fmt.Sprintf("%s which would result in a vulnerable (%s) package installed", message, m.Package.Type)
	} else {
		// 如果不是目录源元数据，则生成相应消息
		message = fmt.Sprintf("%s which is a vulnerable (%s) package installed in the container", message, m.Package.Type)
	}

    // 返回生成的消息
	return sarif.Message{
// 创建一个新的结构体，包含一个名为message的文本字段
type Rule struct {
    Text *message
}

// 根据匹配对象返回规则名称
func ruleName(m match.Match) string {
    // 如果匹配对象的详情长度大于0
    if len(m.Details) > 0 {
        // 获取第一个详情对象
        d := m.Details[0]
        // 创建一个字符串构建器
        buf := strings.Builder{}
        // 遍历详情对象的匹配器和类型
        for _, segment := range []string{string(d.Matcher), string(d.Type)} {
            // 遍历分割后的字符串
            for _, part := range strings.Split(segment, "-") {
                // 将每个部分的首字母大写并拼接到构建器中
                buf.WriteString(strings.ToUpper(part[:1]))
                buf.WriteString(part[1:])
            }
        }
        // 返回构建器中的字符串作为规则名称
        return buf.String()
    }
    // 如果没有详情对象，则返回漏洞ID作为规则名称
    return m.Vulnerability.ID
}
```