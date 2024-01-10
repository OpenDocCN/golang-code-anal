# `grype\grype\vex\openvex\implementation.go`

```
package openvex

import (
    "errors"  // 导入 errors 包，用于处理错误
    "fmt"  // 导入 fmt 包，用于格式化输出
    "net/url"  // 导入 net/url 包，用于 URL 相关操作
    "strings"  // 导入 strings 包，用于字符串操作

    "github.com/google/go-containerregistry/pkg/name"  // 导入 name 包，用于容器镜像名称操作
    openvex "github.com/openvex/go-vex/pkg/vex"  // 导入 openvex 包

    "github.com/anchore/grype/grype/match"  // 导入 match 包，用于匹配
    "github.com/anchore/grype/grype/pkg"  // 导入 pkg 包
    "github.com/anchore/packageurl-go"  // 导入 packageurl-go 包，用于处理 package URL
    "github.com/anchore/syft/syft/source"  // 导入 source 包，用于处理源数据
)

type Processor struct{}  // 定义 Processor 结构体

func New() *Processor {  // 定义 New 函数，返回 Processor 结构体指针
    return &Processor{}
}

// Match captures the criteria that caused a vulnerability to match
type Match struct {  // 定义 Match 结构体
    Statement openvex.Statement  // 包含 openvex.Statement 类型的字段
}

// SearchedBy captures the prameters used to search through the VEX data
type SearchedBy struct {  // 定义 SearchedBy 结构体
    Vulnerability string  // 字符串类型的字段
    Product       string  // 字符串类型的字段
    Subcomponents []string  // 字符串数组类型的字段
}

// augmentStatuses are the VEX statuses that augment results
var augmentStatuses = []openvex.Status{  // 定义 augmentStatuses 变量，包含 openvex.Status 类型的数组
    openvex.StatusAffected,  // 添加 openvex.StatusAffected 到数组中
    openvex.StatusUnderInvestigation,  // 添加 openvex.StatusUnderInvestigation 到数组中
}

// filterStatuses are the VEX statuses that filter matched to the ignore list
var ignoreStatuses = []openvex.Status{  // 定义 ignoreStatuses 变量，包含 openvex.Status 类型的数组
    openvex.StatusNotAffected,  // 添加 openvex.StatusNotAffected 到数组中
    openvex.StatusFixed,  // 添加 openvex.StatusFixed 到数组中
}

// ReadVexDocuments reads and merges VEX documents
func (ovm *Processor) ReadVexDocuments(docs []string) (interface{}, error) {  // 定义 ReadVexDocuments 方法，接收字符串数组参数，返回接口类型和错误
    // Combine all VEX documents into a single VEX document
    vexdata, err := openvex.MergeFiles(docs)  // 调用 openvex.MergeFiles 方法，将所有 VEX 文档合并成一个
    if err != nil {  // 如果有错误发生
        return nil, fmt.Errorf("merging vex documents: %w", err)  // 返回错误信息
    }

    return vexdata, nil  // 返回合并后的 VEX 文档和空错误
}

// productIdentifiersFromContext reads the package context and returns software
// identifiers identifying the scanned image.
func productIdentifiersFromContext(pkgContext *pkg.Context) ([]string, error) {  // 定义 productIdentifiersFromContext 方法，接收 pkg.Context 类型参数，返回字符串数组和错误
    switch v := pkgContext.Source.Metadata.(type) {  // 根据 pkgContext.Source.Metadata 的类型进行判断
    # 如果是StereoscopeImageSourceMetadata类型的数据
    case source.StereoscopeImageSourceMetadata:
        # TODO(puerco): 在这里我们可以创建一个更广泛的定义。这实际上
        # 添加了多架构镜像和运行grype的操作系统的镜像。我们
        # 可以生成更多的标识符以更好地匹配。
        # 从RepoDigests中获取标识符，然后返回
        return identifiersFromDigests(v.RepoDigests), nil
    # 如果不是StereoscopeImageSourceMetadata类型的数据
    default:
        # 暂时失败
        return nil, errors.New("source type not supported for VEX")
    }
}
// 从摘要中获取标识符列表
func identifiersFromDigests(digests []string) []string {
    identifiers := []string{}

    for _, d := range digests {
        // 第一个标识符是原始图像引用：
        identifiers = append(identifiers, d)

        // 不是图像引用，跳过
        ref, err := name.ParseReference(d)
        if err != nil {
            continue
        }

        var digestString, repoURL string
        shaString := ref.Identifier()

        // 如果不是摘要，我们无法形成 purl，所以跳过
        if !strings.HasPrefix(shaString, "sha256:") {
            continue
        }

        digestString = url.QueryEscape(shaString)

        pts := strings.Split(ref.Context().RepositoryStr(), "/")
        name := pts[len(pts)-1]
        repoURL = strings.TrimSuffix(
            ref.Context().RegistryStr()+"/"+ref.Context().RepositoryStr(),
            fmt.Sprintf("/%s", name),
        )

        qMap := map[string]string{}

        if repoURL != "" {
            qMap["repository_url"] = repoURL
        }
        qs := packageurl.QualifiersFromMap(qMap)
        identifiers = append(identifiers, packageurl.NewPackageURL(
            "oci", "", name, digestString, qs, "",
        ).String())

        // 在标识符列表中添加一个哈希，以防人们想要使用图像摘要的值
        identifiers = append(identifiers, strings.TrimPrefix(shaString, "sha256:"))
    }
    return identifiers
}

// 从匹配中返回包的子组件标识符列表
func subcomponentIdentifiersFromMatch(m *match.Match) []string {
    ret := []string{}
    if m.Package.PURL != "" {
        ret = append(ret, m.Package.PURL)
    }

    // TODO(puerco): 在 openvex/go-vex 中实现 CPE 匹配
    /*
        for _, c := range m.Package.CPEs {
            ret = append(ret, c.String())
        }
    */
    return ret
}
// FilterMatches函数接受一组扫描结果，并将VEX数据中标记为fixed或not_affected的结果移动到ignoredMatches列表中。
func (ovm *Processor) FilterMatches(
    docRaw interface{}, ignoreRules []match.IgnoreRule, pkgContext *pkg.Context, matches *match.Matches, ignoredMatches []match.IgnoredMatch,
) (*match.Matches, []match.IgnoredMatch, error) {
    // 将docRaw转换为*openvex.VEX类型的doc
    doc, ok := docRaw.(*openvex.VEX)
    if !ok {
        return nil, nil, errors.New("unable to cast vex document as openvex")
    }

    // 创建一个新的Matches对象用于存储剩余的匹配结果
    remainingMatches := match.NewMatches()

    // 从pkgContext中获取产品标识符
    products, err := productIdentifiersFromContext(pkgContext)
    if err != nil {
        return nil, nil, fmt.Errorf("reading product identifiers from context: %w", err)
    }

    // TODO(alex): 是否应该将vex忽略规则应用于已经忽略的匹配？
    // 这样最终用户就可以看到所有匹配被忽略的原因，以防多个原因适用

    // 现在，让我们遍历grype的匹配结果
    sorted := matches.Sorted()
}
    # 遍历已排序的匹配结果
    for i := range sorted:
        # 声明一个指向 openvex.Statement 类型的变量
        var statement *openvex.Statement
        # 从匹配结果中获取子组件标识符
        subcmp := subcomponentIdentifiersFromMatch(&sorted[i])

        # 遍历产品的不同名称
        for _, product := range products:
            # 如果匹配到的语句不为空，则将第一个匹配的语句赋值给 statement，并跳出循环
            if matchingStatements := doc.Matches(sorted[i].Vulnerability.ID, product, subcmp); len(matchingStatements) != 0:
                statement = &matchingStatements[0]
                break
            # 如果没有关于此匹配组件的数据，则继续下一个循环
        # 如果 statement 为空，则将当前匹配结果添加到 remainingMatches 中，并继续下一个循环
        if statement == nil:
            remainingMatches.Add(sorted[i])
            continue

        # 根据匹配规则获取规则
        rule := matchingRule(ignoreRules, sorted[i], statement, ignoreStatuses)
        # 如果规则为空，则将当前匹配结果添加到 remainingMatches 中，并继续下一个循环
        if rule == nil:
            remainingMatches.Add(sorted[i])
            continue

        # 只对未受影响和已修复状态进行过滤
        if statement.Status != openvex.StatusNotAffected && statement.Status != openvex.StatusFixed:
            remainingMatches.Add(sorted[i])
            continue

        # 将忽略的匹配结果添加到 ignoredMatches 中
        ignoredMatches = append(ignoredMatches, match.IgnoredMatch{
            Match:              sorted[i],
            AppliedIgnoreRules: []match.IgnoreRule{*rule},
        })
    # 返回 remainingMatches、ignoredMatches 和空值
    return &remainingMatches, ignoredMatches, nil
// matchingRule函数循环遍历一组忽略规则，并返回第一个与语句和匹配项匹配的规则。如果没有匹配项，则返回nil。
func matchingRule(ignoreRules []match.IgnoreRule, m match.Match, statement *openvex.Statement, allowedStatuses []openvex.Status) *match.IgnoreRule {
    // 创建一个新的匹配对象
    ms := match.NewMatches()
    // 将匹配项添加到匹配对象中
    ms.Add(m)

    // 创建一个反向状态映射，用于存储允许的状态
    revStatuses := map[string]struct{}{}
    // 遍历允许的状态列表，将状态存储到反向状态映射中
    for _, s := range allowedStatuses {
        revStatuses[string(s)] = struct{}{}
    }
}
    // 遍历忽略规则列表
    for _, rule := range ignoreRules {
        // 如果规则除了 VEX 语句之外还有更多条件，检查是否适用于当前匹配
        if rule.HasConditions() {
            // 复制规则以避免修改原始规则
            r := rule
            // 清空 VexStatus 字段
            r.VexStatus = ""
            // 应用忽略规则到当前匹配，检查是否被忽略
            if _, ignored := match.ApplyIgnoreRules(ms, []match.IgnoreRule{r}); len(ignored) == 0 {
                // 如果没有被忽略，则继续下一个规则
                continue
            }
        }

        // 如果语句中的状态与规则中的 VexStatus 不同，则不适用
        if string(statement.Status) != rule.VexStatus {
            continue
        }

        // 如果规则有除了允许的状态之外的语句，则跳过
        if len(revStatuses) > 0 && rule.VexStatus != "" {
            if _, ok := revStatuses[rule.VexStatus]; !ok {
                continue
            }
        }

        // 如果规则适用于 VEX 证明，它需要与语句匹配，注意证明只适用于 not_affected 状态
        if statement.Status == openvex.StatusNotAffected && rule.VexJustification != "" &&
            rule.VexJustification != string(statement.Justification) {
            continue
        }

        // 如果规则中的漏洞为空，表示我们将接受任何状态和任何漏洞
        if rule.Vulnerability == "" {
            // 返回该规则
            return &rule
        }

        // 如果设置了漏洞，规则适用于语句和规则中的相同漏洞
        if statement.Vulnerability.Matches(rule.Vulnerability) {
            // 返回该规则
            return &rule
        }
    }
    // 如果没有匹配的规则，则返回空
    return nil
// AugmentMatches函数用于在加载的VEX文档中找到受影响的VEX产品的匹配数据时，将结果添加到match.Matches数组中。
// 当找不到先前的数据时，匹配将从忽略列表中移除或合成。
func (ovm *Processor) AugmentMatches(
    docRaw interface{}, ignoreRules []match.IgnoreRule, pkgContext *pkg.Context, remainingMatches *match.Matches, ignoredMatches []match.IgnoredMatch,
) (*match.Matches, []match.IgnoredMatch, error) {
    // 将docRaw转换为openvex.VEX类型，如果无法转换，则返回错误
    doc, ok := docRaw.(*openvex.VEX)
    if !ok {
        return nil, nil, errors.New("unable to cast vex document as openvex")
    }

    // 初始化additionalIgnoredMatches数组
    additionalIgnoredMatches := []match.IgnoredMatch{}

    // 从pkgContext中获取产品标识符
    products, err := productIdentifiersFromContext(pkgContext)
    if err != nil {
        return nil, nil, fmt.Errorf("reading product identifiers from context: %w", err)
    }

    // 现在，让我们遍历grype的匹配结果
    // 遍历忽略的匹配列表
    for i := range ignoredMatches {
        // 声明语句和搜索方式的指针变量
        var statement *openvex.Statement
        var searchedBy *SearchedBy
        // 从匹配中获取子组件标识符
        subcmp := subcomponentIdentifiersFromMatch(&ignoredMatches[i].Match)

        // 遍历产品的不同名称，查看是否与语句数据匹配
        for _, product := range products {
            // 如果匹配到了语句，则获取匹配的语句列表
            if matchingStatements := doc.Matches(ignoredMatches[i].Vulnerability.ID, product, subcmp); len(matchingStatements) != 0 {
                // 如果匹配到的语句状态不是受影响或者正在调查中，则跳出循环
                if matchingStatements[0].Status != openvex.StatusAffected &&
                    matchingStatements[0].Status != openvex.StatusUnderInvestigation {
                    break
                }
                // 将匹配到的语句赋值给 statement
                statement = &matchingStatements[0]
                // 设置搜索方式的值
                searchedBy = &SearchedBy{
                    Vulnerability: ignoredMatches[i].Vulnerability.ID,
                    Product:       product,
                    Subcomponents: subcmp,
                }
                break
            }
        }

        // 如果没有关于此匹配组件的数据，则继续下一个循环
        if statement == nil {
            additionalIgnoredMatches = append(additionalIgnoredMatches, ignoredMatches[i])
            continue
        }

        // 只有在配置了用于增强的规则时才匹配
        rule := matchingRule(ignoreRules, ignoredMatches[i].Match, statement, augmentStatuses)
        if rule == nil {
            additionalIgnoredMatches = append(additionalIgnoredMatches, ignoredMatches[i])
            continue
        }

        // 创建新的匹配，并添加到剩余匹配列表中
        newMatch := ignoredMatches[i].Match
        newMatch.Details = append(newMatch.Details, match.Detail{
            Type:       match.ExactDirectMatch,
            SearchedBy: searchedBy,
            Found: Match{
                Statement: *statement,
            },
            Matcher: match.OpenVexMatcher,
        })

        remainingMatches.Add(newMatch)
    }

    // 返回剩余匹配列表、额外忽略的匹配列表和空值
    return remainingMatches, additionalIgnoredMatches, nil
# 闭合前面的函数定义
```