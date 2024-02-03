# `grype\grype\presenter\sarif\presenter.go`

```go
package sarif

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"  // 导入 io 包，用于输入输出操作
    "path/filepath"  // 导入 filepath 包，用于处理文件路径
    "strings"  // 导入 strings 包，用于处理字符串

    "github.com/owenrumney/go-sarif/sarif"  // 导入第三方库

    "github.com/anchore/clio"  // 导入 anchore/clio 包
    v5 "github.com/anchore/grype/grype/db/v5"  // 导入 anchore/grype/grype/db/v5 包
    "github.com/anchore/grype/grype/match"  // 导入 anchore/grype/grype/match 包
    "github.com/anchore/grype/grype/pkg"  // 导入 anchore/grype/grype/pkg 包
    "github.com/anchore/grype/grype/presenter/models"  // 导入 anchore/grype/grype/presenter/models 包
    "github.com/anchore/grype/grype/vulnerability"  // 导入 anchore/grype/grype/vulnerability 包
    "github.com/anchore/syft/syft/file"  // 导入 anchore/syft/syft/file 包
    "github.com/anchore/syft/syft/source"  // 导入 anchore/syft/syft/source 包
)

// Presenter holds the data for generating a report and implements the presenter.Presenter interface
type Presenter struct {
    id               clio.Identification  // 定义 clio.Identification 类型的 id 字段
    results          match.Matches  // 定义 match.Matches 类型的 results 字段
    packages         []pkg.Package  // 定义 pkg.Package 类型的切片 packages 字段
    src              *source.Description  // 定义 *source.Description 类型的 src 字段
    metadataProvider vulnerability.MetadataProvider  // 定义 vulnerability.MetadataProvider 类型的 metadataProvider 字段
}

// NewPresenter is a *Presenter constructor
func NewPresenter(pb models.PresenterConfig) *Presenter {
    return &Presenter{  // 返回一个 Presenter 对象
        id:               pb.ID,  // 初始化 id 字段
        results:          pb.Matches,  // 初始化 results 字段
        packages:         pb.Packages,  // 初始化 packages 字段
        metadataProvider: pb.MetadataProvider,  // 初始化 metadataProvider 字段
        src:              pb.Context.Source,  // 初始化 src 字段
    }
}

// Present creates a SARIF-based report
func (pres *Presenter) Present(output io.Writer) error {
    doc, err := pres.toSarifReport()  // 调用 toSarifReport 方法生成 SARIF 报告
    if err != nil {
        return err  // 如果生成报告出错，返回错误
    }
    err = doc.PrettyWrite(output)  // 将报告写入输出流
    return err  // 返回可能的错误
}

// toSarifReport outputs a sarif report object
func (pres *Presenter) toSarifReport() (*sarif.Report, error) {
    doc, err := sarif.New(sarif.Version210)  // 创建一个 SARIF 报告对象
    if err != nil {
        return nil, err  // 如果创建报告对象出错，返回错误
    }

    v := pres.id.Version  // 获取版本号
    if v == "[not provided]" || v == "" {
        // Need a semver to pass the MS SARIF validator
        v = "0.0.0-dev"  // 如果版本号未提供，则设置为 "0.0.0-dev"
    }
    # 向 SARIF 文档中添加运行信息
    doc.AddRun(&sarif.Run{
        # 设置工具信息
        Tool: sarif.Tool{
            # 设置驱动信息
            Driver: &sarif.ToolComponent{
                # 设置工具名称为 pres.id.Name
                Name:           pres.id.Name,
                # 设置工具版本为 sp(v)
                Version:        sp(v),
                # 设置工具信息链接为 "https://github.com/anchore/grype"
                InformationURI: sp("https://github.com/anchore/grype"),
                # 设置工具规则为 pres.sarifRules()
                Rules:          pres.sarifRules(),
            },
        },
        # 设置运行结果为 pres.sarifResults()
        Results: pres.sarifResults(),
    })

    # 返回 SARIF 文档和空错误
    return doc, nil
// sarifRules 生成要包含在此运行中的规则集
func (pres *Presenter) sarifRules() (out []*sarif.ReportingDescriptor) {
    // 如果结果计数大于0
    if pres.results.Count() > 0 {
        // 创建规则ID的映射
        ruleIDs := map[string]bool{}

        // 遍历排序后的结果
        for _, m := range pres.results.Sorted() {
            // 获取规则ID
            ruleID := pres.ruleID(m)
            // 如果规则ID已存在，则跳过
            if ruleIDs[ruleID] {
                continue
            }

            ruleIDs[ruleID] = true

            // 可能完全没有任何链接
            link := m.Vulnerability.ID
            meta := pres.metadata(m)
            if meta != nil {
                switch {
                case meta.DataSource != "":
                    link = fmt.Sprintf("[%s](%s)", meta.ID, meta.DataSource)
                case len(meta.URLs) > 0:
                    link = fmt.Sprintf("[%s](%s)", meta.ID, meta.URLs[0])
                }
            }

            // 添加规则到输出列表
            out = append(out, &sarif.ReportingDescriptor{
                ID:      ruleID,
                Name:    sp(ruleName(m)),
                HelpURI: sp("https://github.com/anchore/grype"),
                // SARIF报告的标题
                ShortDescription: &sarif.MultiformatMessageString{
                    Text: sp(pres.shortDescription(m)),
                },
                // SARIF报告的副标题
                FullDescription: &sarif.MultiformatMessageString{
                    Text: sp(pres.subtitle(m)),
                },
                Help: pres.helpText(m, link),
                Properties: sarif.Properties{
                    // 对于GitHub reportingDescriptor对象：
                    // https://docs.github.com/en/code-security/code-scanning/integrating-with-code-scanning/sarif-support-for-code-scanning#reportingdescriptor-object
                    "security-severity": pres.securitySeverityValue(m),
                },
            })
        }
    }
}
    # 返回变量 out 的值
    return out
// 为给定匹配创建一个唯一的规则ID
func (pres *Presenter) ruleID(m match.Match) string {
    // 如果我们支持配置，可能希望允许添加另一个限定符，以便如果在多个容器上运行多个漏洞扫描，我们可以为每个识别出唯一的规则
    return fmt.Sprintf("%s-%s", m.Vulnerability.ID, m.Package.Name)
}

// 获取规则的帮助文本，这在GitHub上显示在漏洞列表中的标题中
func (pres *Presenter) helpText(m match.Match, link string) *sarif.MultiformatMessageString {
    // 我们不一定需要在这里添加位置，可能会有多个引用相同的漏洞
    // 例如，如果在图像中找到多个引用，则我们可以添加一些受影响位置的列表，但如果特定分支有多个漏洞扫描，这可能会变得更复杂
    text := fmt.Sprintf("Vulnerability %s\nSeverity: %s\nPackage: %s\nVersion: %s\nFix Version: %s\nType: %s\nLocation: %s\nData Namespace: %s\nLink: %s",
        m.Vulnerability.ID, pres.severityText(m), m.Package.Name, m.Package.Version, fixVersions(m), m.Package.Type, pres.packagePath(m.Package), m.Vulnerability.Namespace, link,
    )
    markdown := fmt.Sprintf(
        "**Vulnerability %s**\n"+
            "| Severity | Package | Version | Fix Version | Type | Location | Data Namespace | Link |\n"+
            "| --- | --- | --- | --- | --- | --- | --- | --- |\n"+
            "| %s  | %s  | %s  | %s  | %s  | %s  | %s  | %s  |\n",
        m.Vulnerability.ID, pres.severityText(m), m.Package.Name, m.Package.Version, fixVersions(m), m.Package.Type, pres.packagePath(m.Package), m.Vulnerability.Namespace, link,
    )
    return &sarif.MultiformatMessageString{
        Text:     &text,
        Markdown: &markdown,
    }
}

// packagePath尝试获取包相对于“扫描根目录”的相对路径
// packagePath 返回给定包的路径
func (pres *Presenter) packagePath(p pkg.Package) string {
    // 将位置转换为切片
    locations := p.Locations.ToSlice()
    // 如果位置切片不为空，返回第一个位置的路径
    if len(locations) > 0 {
        return pres.locationPath(locations[0])
    }
    // 否则返回输入路径
    return pres.inputPath()
}

// inputPath 根据输入返回更友好的相对路径或绝对路径，不以 . 或 ./ 开头
func (pres *Presenter) inputPath() string {
    // 如果源为空，返回空字符串
    if pres.src == nil {
        return ""
    }
    var inputPath string
    // 根据源的元数据类型设置输入路径
    switch m := pres.src.Metadata.(type) {
    case source.FileSourceMetadata:
        inputPath = m.Path
    case source.DirectorySourceMetadata:
        inputPath = m.Path
    default:
        return ""
    }
    // 去除路径中的 ./ 前缀
    inputPath = strings.TrimPrefix(inputPath, "./")
    // 如果输入路径为 .，返回空字符串
    if inputPath == "." {
        return ""
    }
    return inputPath
}

// locationPath 返回相对于当前工作目录的位置路径
func (pres *Presenter) locationPath(l file.Location) string {
    // 获取位置的路径
    path := l.Path()
    // 获取输入路径
    in := pres.inputPath()
    // 去除路径中的 ./ 前缀
    path = strings.TrimPrefix(path, "./")
    // 如果源不为空
    if pres.src != nil {
        // 判断源的元数据类型
        _, ok := pres.src.Metadata.(source.DirectorySourceMetadata)
        if ok {
            // 如果路径是绝对路径或输入路径为空，返回路径
            if filepath.IsAbs(path) || in == "" {
                return path
            }
            // 如果路径不是绝对路径，返回相对于当前工作目录的路径
            return fmt.Sprintf("%s/%s", in, path)
        }
    }
    return path
}

// locations 函数的注释
func (pres *Presenter) locations(m match.Match) []*sarif.Location {
    // 获取包的路径
    physicalLocation := pres.packagePath(m.Package)
    // 初始化逻辑位置切片
    var logicalLocations []*sarif.LogicalLocation
    // 根据源的元数据类型设置逻辑位置
    switch metadata := pres.src.Metadata.(type) {
    # 如果是立体镜像源元数据
    case source.StereoscopeImageSourceMetadata:
        # 获取用户输入的图像
        img := metadata.UserInput
        # 获取包的位置并转换为切片
        locations := m.Package.Locations.ToSlice()
        # 遍历位置切片
        for _, l := range locations:
            # 去除路径前缀并存储到trimmedPath
            trimmedPath := strings.TrimPrefix(pres.locationPath(l), "/")
            # 将逻辑位置添加到逻辑位置切片
            logicalLocations = append(logicalLocations, &sarif.LogicalLocation{
                FullyQualifiedName: sp(fmt.Sprintf("%s@%s:/%s", img, l.FileSystemID, trimmedPath)),
                Name:               sp(l.RealPath),
            })
        }

        # 这是一个用于在GitHub上显示结果的hack，因为它需要位置的相对路径
        # 但我们实际上不会有任何关于在文件系统上使用哪个Dockerfile构建图像的信息
        # TODO：我们可以添加配置来指定前缀，用户可能希望指定图像名称和架构
        # 例如，在多个漏洞扫描的情况下
        physicalLocation = fmt.Sprintf("image/%s", physicalLocation)
    # 如果是文件源元数据
    case source.FileSourceMetadata:
        # 获取包的位置并转换为切片
        locations := m.Package.Locations.ToSlice()
        # 遍历位置切片
        for _, l := range locations:
            # 将逻辑位置添加到逻辑位置切片
            logicalLocations = append(logicalLocations, &sarif.LogicalLocation{
                FullyQualifiedName: sp(fmt.Sprintf("%s:/%s", metadata.Path, pres.locationPath(l))),
                Name:               sp(l.RealPath),
            })
    # 如果是目录源元数据
    case source.DirectorySourceMetadata:
        # 目录方案已经处理过，如果需要，已经添加了输入
    }
    # 返回一个空的sarif.Location列表
    return []*sarif.Location{
        {
            # 设置物理位置信息
            PhysicalLocation: &sarif.PhysicalLocation{
                # 设置文件位置的URI
                ArtifactLocation: &sarif.ArtifactLocation{
                    URI: sp(physicalLocation),
                },
                # 设置区域信息，暂时使用1行1列
                # TODO 当grype开始报告行号时，需要更新此信息
                Region: &sarif.Region{
                    StartLine:   ip(1),
                    StartColumn: ip(1),
                    EndLine:     ip(1),
                    EndColumn:   ip(1),
                },
            },
            # 设置逻辑位置信息
            LogicalLocations: logicalLocations,
        },
    }
// severityText 提供了匹配的严重级别的文本表示
func (pres *Presenter) severityText(m match.Match) string {
    // 获取匹配的元数据
    meta := pres.metadata(m)
    // 如果元数据不为空
    if meta != nil {
        // 解析严重级别
        severity := vulnerability.ParseSeverity(meta.Severity)
        // 根据严重级别返回相应的文本表示
        switch severity {
        case vulnerability.CriticalSeverity:
            return "critical"
        case vulnerability.HighSeverity:
            return "high"
        case vulnerability.MediumSeverity:
            return "medium"
        }
    }
    // 如果没有匹配的严重级别，则返回"low"
    return "low"
}

// cvssScore 尝试获取我们的漏洞数据中最佳的 CVSS 分数
func (pres *Presenter) cvssScore(v vulnerability.Vulnerability) float64 {
    var all []*vulnerability.Metadata

    // 获取漏洞的元数据
    meta, err := pres.metadataProvider.GetMetadata(v.ID, v.Namespace)
    // 如果没有错误并且元数据不为空
    if err == nil && meta != nil {
        // 将元数据添加到all切片中
        all = append(all, meta)
    }

    // 遍历相关的漏洞
    for _, related := range v.RelatedVulnerabilities {
        // 获取相关漏洞的元数据
        meta, err = pres.metadataProvider.GetMetadata(related.ID, related.Namespace)
        // 如果没有错误并且元数据不为空
        if err == nil && meta != nil {
            // 将元数据添加到all切片中
            all = append(all, meta)
        }
    }

    // 初始化分数为-1.0
    score := -1.0

    // 首先检查供应商特定的条目
    for _, m := range all {
        // 如果命名空间是 "nvd:cpe"，则继续下一次循环
        if m.Namespace == "nvd:cpe" {
            continue
        }
        // 遍历 CVSS 数据
        for _, cvss := range m.Cvss {
            // 如果基础分数大于当前分数，则更新分数
            if cvss.Metrics.BaseScore > score {
                score = cvss.Metrics.BaseScore
            }
        }
    }

    // 如果分数大于0，则返回分数
    if score > 0 {
        return score
    }

    // 接下来，检查 nvd 条目
    for _, m := range all {
        // 遍历 CVSS 数据
        for _, cvss := range m.Cvss {
            // 如果基础分数大于当前分数，则更新分数
            if cvss.Metrics.BaseScore > score {
                score = cvss.Metrics.BaseScore
            }
        }
    }

    // 返回分数
    return score
}

// securitySeverityValue GitHub security-severity 属性使用数值严重性值来确定是否为关键、高等；这将我们的漏洞转换为范围内的值
func (pres *Presenter) securitySeverityValue(m match.Match) string {
    // 获取漏洞的元数据
    meta := pres.metadata(m)
    // 如果存在元数据
    if meta != nil {
        // 获取漏洞的 CVSS 分数
        score := pres.cvssScore(m.Vulnerability)
        // 如果分数大于 0，则返回分数
        if score > 0 {
            return fmt.Sprintf("%.1f", score)
        }
        // 解析漏洞的严重程度
        severity := vulnerability.ParseSeverity(meta.Severity)
        // 根据严重程度返回相应的分数
        switch severity {
        case vulnerability.CriticalSeverity:
            return "9.0"
        case vulnerability.HighSeverity:
            return "7.0"
        case vulnerability.MediumSeverity:
            return "4.0"
        case vulnerability.LowSeverity:
            return "1.0"
        }
    }
    // 如果没有元数据，则返回默认分数
    return "0.0"
// metadata 返回与提供程序匹配的 *vulnerability.Metadata，如果未找到或出现错误，则返回 nil
func (pres *Presenter) metadata(m match.Match) *vulnerability.Metadata {
    // 从 metadataProvider 获取匹配的漏洞元数据，如果未找到则返回 nil
    meta, _ := pres.metadataProvider.GetMetadata(m.Vulnerability.ID, m.Vulnerability.Namespace)
    return meta
}

// subtitle 为给定的匹配生成副标题
func (pres *Presenter) subtitle(m match.Match) string {
    // 获取匹配的元数据
    meta := pres.metadata(m)
    if meta != nil {
        subtitle := meta.Description
        if subtitle != "" {
            return subtitle
        }
    }

    // 获取修复版本信息
    fixVersion := fixVersions(m)
    if fixVersion != "" {
        return fmt.Sprintf("Version %s is affected with an available fix in versions %s", m.Package.Version, fixVersion)
    }

    return fmt.Sprintf("Version %s is affected with no fixes reported yet.", m.Package.Version)
}

// fixVersions 返回修复版本信息
func fixVersions(m match.Match) string {
    // 如果漏洞修复状态为已修复，并且修复版本数量大于 0，则返回修复版本列表
    if m.Vulnerability.Fix.State == v5.FixedState && len(m.Vulnerability.Fix.Versions) > 0 {
        return strings.Join(m.Vulnerability.Fix.Versions, ",")
    }
    return ""
}

// shortDescription 返回简短描述
func (pres *Presenter) shortDescription(m match.Match) string {
    return fmt.Sprintf("%s %s vulnerability for %s package", m.Vulnerability.ID, pres.severityText(m), m.Package.Name)
}

// sarifResults 返回 SARIF 结果数组
func (pres *Presenter) sarifResults() []*sarif.Result {
    out := make([]*sarif.Result, 0) // 确保至少有一个空数组
    for _, m := range pres.results.Sorted() {
        out = append(out, &sarif.Result{
            RuleID:  sp(pres.ruleID(m)),
            Message: pres.resultMessage(m),
            // 根据 SARIF 规范，我认为我们应该使用 AnalysisTarget.URI 来指示逻辑文件，比如 "Dockerfile"，但 GitHub 与此不兼容
            // FIXME github "requires" partialFingerprints
            // PartialFingerprints: ???
            Locations: pres.locations(m),
        })
    }
    return out
}

// ip 根据提供的值返回一个 int 指针
// 返回一个指向整数的指针
func ip(i int) *int {
    return &i
}

// 根据提供的值返回一个字符串指针
func sp(sarif string) *string {
    return &sarif
}

// 根据匹配结果生成消息
func (pres *Presenter) resultMessage(m match.Match) sarif.Message {
    // 获取包路径
    path := pres.packagePath(m.Package)
    // 格式化消息
    message := fmt.Sprintf("The path %s reports %s at version %s ", path, m.Package.Name, m.Package.Version)

    // 判断是否为目录源
    if _, ok := pres.src.Metadata.(source.DirectorySourceMetadata); ok {
        // 如果是目录源，则添加相应信息
        message = fmt.Sprintf("%s which would result in a vulnerable (%s) package installed", message, m.Package.Type)
    } else {
        // 如果不是目录源，则添加相应信息
        message = fmt.Sprintf("%s which is a vulnerable (%s) package installed in the container", message, m.Package.Type)
    }

    // 返回消息对象
    return sarif.Message{
        Text: &message,
    }
}

// 根据匹配结果生成规则名称
func ruleName(m match.Match) string {
    if len(m.Details) > 0 {
        d := m.Details[0]
        buf := strings.Builder{}
        for _, segment := range []string{string(d.Matcher), string(d.Type)} {
            for _, part := range strings.Split(segment, "-") {
                buf.WriteString(strings.ToUpper(part[:1]))
                buf.WriteString(part[1:])
            }
        }
        return buf.String()
    }
    return m.Vulnerability.ID
}
```