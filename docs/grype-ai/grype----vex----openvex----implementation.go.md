# `grype\grype\vex\openvex\implementation.go`

```
package openvex
# 声明包名为 openvex

import (
	"errors"
	"fmt"
	"net/url"
	"strings"

	"github.com/google/go-containerregistry/pkg/name"
	openvex "github.com/openvex/go-vex/pkg/vex"

	"github.com/anchore/grype/grype/match"
	"github.com/anchore/grype/grype/pkg"
	"github.com/anchore/packageurl-go"
	"github.com/anchore/syft/syft/source"
)
# 导入所需的包

type Processor struct{}
# 定义 Processor 结构体

func New() *Processor {
# 定义 New 函数，返回一个 Processor 类型的指针
// 返回一个空的 Processor 结构体实例
return &Processor{}
}

// Match 结构体用于捕获导致漏洞匹配的条件
type Match struct {
	Statement openvex.Statement
}

// SearchedBy 结构体用于捕获用于搜索 VEX 数据的参数
type SearchedBy struct {
	Vulnerability string
	Product       string
	Subcomponents []string
}

// augmentStatuses 是用于增强结果的 VEX 状态列表
var augmentStatuses = []openvex.Status{
	openvex.StatusAffected,
	openvex.StatusUnderInvestigation,
}
// filterStatuses are the VEX statuses that filter matched to the ignore list
// 定义了需要被忽略的 VEX 状态列表

var ignoreStatuses = []openvex.Status{
	openvex.StatusNotAffected,
	openvex.StatusFixed,
}
// 初始化了需要被忽略的 VEX 状态列表，包括未受影响和已修复状态

// ReadVexDocuments reads and merges VEX documents
// ReadVexDocuments 函数用于读取和合并 VEX 文档
func (ovm *Processor) ReadVexDocuments(docs []string) (interface{}, error) {
	// Combine all VEX documents into a single VEX document
	// 将所有的 VEX 文档合并成一个单独的 VEX 文档
	vexdata, err := openvex.MergeFiles(docs)
	// 调用 openvex.MergeFiles 函数合并文档
	if err != nil {
		return nil, fmt.Errorf("merging vex documents: %w", err)
	}
	// 如果合并过程中出现错误，则返回错误信息

	return vexdata, nil
	// 返回合并后的 VEX 文档
}

// productIdentifiersFromContext reads the package context and returns software
// identifiers identifying the scanned image.
// productIdentifiersFromContext 函数用于读取软件包上下文，并返回识别扫描图像的软件标识符
// 从包上下文中获取产品标识符
func productIdentifiersFromContext(pkgContext *pkg.Context) ([]string, error) {
	// 根据包上下文中的源元数据类型进行不同的处理
	switch v := pkgContext.Source.Metadata.(type) {
	case source.StereoscopeImageSourceMetadata:
		// TODO(puerco): 在这里我们可以创建更广泛的定义。这实际上
		// 添加了多架构镜像和运行 grype 的操作系统的镜像。我们
		// 可以生成更多的标识符以更好地匹配。
		return identifiersFromDigests(v.RepoDigests), nil
	default:
		// 目前不支持的源类型
		return nil, errors.New("source type not supported for VEX")
	}
}

// 从摘要中获取标识符
func identifiersFromDigests(digests []string) []string {
	identifiers := []string{}

	// 遍历摘要列表，将每个摘要作为标识符添加到标识符列表中
	for _, d := range digests {
		// 第一个标识符是原始镜像引用：
		identifiers = append(identifiers, d)
	}
	// 返回标识符列表
	return identifiers
}
		// 如果不是一个图片引用，则跳过
		ref, err := name.ParseReference(d)  // 解析引用
		if err != nil {  // 如果解析出错，则继续下一个循环
			continue
		}

		var digestString, repoURL string  // 定义变量
		shaString := ref.Identifier()  // 获取引用的标识符

		// 如果不是一个摘要，则无法形成 purl，所以跳过
		if !strings.HasPrefix(shaString, "sha256:") {  // 如果标识符不是以 "sha256:" 开头，则跳过
			continue
		}

		digestString = url.QueryEscape(shaString)  // 对标识符进行 URL 编码

		pts := strings.Split(ref.Context().RepositoryStr(), "/")  // 根据斜杠分割仓库字符串
		name := pts[len(pts)-1]  // 获取仓库名称
		repoURL = strings.TrimSuffix(  // 获取仓库 URL
			ref.Context().RegistryStr()+"/"+ref.Context().RepositoryStr(),  // 拼接仓库 URL
		// 使用格式化字符串创建一个以斜杠开头的字符串
		fmt.Sprintf("/%s", name),
	)

	// 创建一个空的字符串到字符串的映射
	qMap := map[string]string{}

	// 如果 repoURL 不为空，则将 "repository_url" 映射到 repoURL
	if repoURL != "" {
		qMap["repository_url"] = repoURL
	}

	// 从映射创建一个 packageurl.Qualifiers 对象
	qs := packageurl.QualifiersFromMap(qMap)

	// 创建一个新的 PackageURL 对象，并将其字符串形式添加到 identifiers 列表中
	identifiers = append(identifiers, packageurl.NewPackageURL(
		"oci", "", name, digestString, qs, "",
	).String())

	// 在标识符列表中添加一个哈希，以防有人想要使用图像摘要的值
	identifiers = append(identifiers, strings.TrimPrefix(shaString, "sha256:"))
}
// 返回标识符列表
return identifiers
// subcomponentIdentifiersFromMatch函数从匹配的包中返回标识符列表。
func subcomponentIdentifiersFromMatch(m *match.Match) []string {
    ret := []string{}
    // 如果匹配的包的PURL不为空，则将其添加到返回列表中
    if m.Package.PURL != "" {
        ret = append(ret, m.Package.PURL)
    }

    // TODO(puerco): 在openvex/go-vex中实现CPE匹配
    /*
        for _, c := range m.Package.CPEs {
            ret = append(ret, c.String())
        }
    */
    return ret
}

// FilterMatches函数接受一组扫描结果，并将VEX数据中标记为fixed或not_affected的结果移动到忽略列表中。
func (ovm *Processor) FilterMatches(
# 定义一个函数，接受多个参数，并返回匹配结果、被忽略的匹配结果和可能出现的错误
func processMatches(
    docRaw interface{},  # 原始文档数据
    ignoreRules []match.IgnoreRule,  # 忽略规则列表
    pkgContext *pkg.Context,  # 包上下文
    matches *match.Matches,  # 匹配结果
    ignoredMatches []match.IgnoredMatch,  # 被忽略的匹配结果
) (*match.Matches, []match.IgnoredMatch, error) {  # 返回匹配结果、被忽略的匹配结果和可能出现的错误
    # 将原始文档数据转换为 openvex.VEX 类型
    doc, ok := docRaw.(*openvex.VEX)
    if !ok {
        return nil, nil, errors.New("unable to cast vex document as openvex")
    }

    # 创建一个新的匹配结果对象
    remainingMatches := match.NewMatches()

    # 从包上下文中获取产品标识符
    products, err := productIdentifiersFromContext(pkgContext)
    if err != nil {
        return nil, nil, fmt.Errorf("reading product identifiers from context: %w", err)
    }

    # 循环遍历 grype 的匹配结果
    sorted := matches.Sorted()
    for i := range sorted {
		// 声明一个指向 openvex.Statement 类型的变量
		var statement *openvex.Statement
		// 从匹配结果中获取子组件标识符
		subcmp := subcomponentIdentifiersFromMatch(&sorted[i])

		// 遍历产品的不同名称
		for _, product := range products {
			// 如果存在匹配的语句，则将其赋值给 statement，并跳出循环
			if matchingStatements := doc.Matches(sorted[i].Vulnerability.ID, product, subcmp); len(matchingStatements) != 0 {
				statement = &matchingStatements[0]
				break
			}
		}

		// 如果没有关于此匹配组件的数据，则继续下一个循环
		if statement == nil {
			remainingMatches.Add(sorted[i])
			continue
		}

		// 获取匹配规则
		rule := matchingRule(ignoreRules, sorted[i], statement, ignoreStatuses)
		// 如果规则为空，则将当前匹配结果添加到 remainingMatches 中
		if rule == nil {
			remainingMatches.Add(sorted[i])
		// 如果语句为空，则继续下一次循环
		continue
	}

	// 过滤只适用于未受影响和已修复状态的语句
	if statement.Status != openvex.StatusNotAffected && statement.Status != openvex.StatusFixed {
		remainingMatches.Add(sorted[i])
		// 继续下一次循环
		continue
	}

	// 将匹配的忽略规则添加到忽略匹配列表中
	ignoredMatches = append(ignoredMatches, match.IgnoredMatch{
		Match:              sorted[i],
		AppliedIgnoreRules: []match.IgnoreRule{*rule},
	})
}
// 返回剩余匹配项、被忽略的匹配项和空指针
return &remainingMatches, ignoredMatches, nil
}

// matchingRule 遍历一组忽略规则，并返回与语句和匹配项匹配的第一个规则。如果没有匹配项，则返回空指针。
func matchingRule(ignoreRules []match.IgnoreRule, m match.Match, statement *openvex.Statement, allowedStatuses []openvex.Status) *match.IgnoreRule {
// 创建一个新的匹配对象
ms := match.NewMatches()
// 将给定的匹配对象添加到匹配集合中
ms.Add(m)

// 创建一个空的字符串到结构体的映射
revStatuses := map[string]struct{}{}
// 遍历允许的状态列表，将状态转换为字符串并添加到映射中
for _, s := range allowedStatuses {
	revStatuses[string(s)] = struct{}{}
}

// 遍历忽略规则列表
for _, rule := range ignoreRules {
	// 如果规则除了 VEX 语句外还有其他条件，检查它是否适用于当前匹配
	if rule.HasConditions() {
		r := rule
		r.VexStatus = ""
		// 应用忽略规则到匹配集合，如果没有被忽略则继续下一个规则
		if _, ignored := match.ApplyIgnoreRules(ms, []match.IgnoreRule{r}); len(ignored) == 0 {
			continue
		}
	}

	// 如果语句中的状态与规则中的状态不同
		// 如果语句的状态与规则的 VexStatus 不匹配，则跳过
		if string(statement.Status) != rule.VexStatus {
			continue
		}

		// 如果规则有除了允许的状态之外的其他状态，跳过：
		if len(revStatuses) > 0 && rule.VexStatus != "" {
			if _, ok := revStatuses[rule.VexStatus]; !ok {
				continue
			}
		}

		// 如果规则适用于 VEX 证明，它需要与语句匹配，注意证明只适用于 not_affected 状态：
		if statement.Status == openvex.StatusNotAffected && rule.VexJustification != "" &&
			rule.VexJustification != string(statement.Justification) {
			continue
		}

		// 如果规则中的漏洞为空，这意味着我们将遵守
// 如果规则中的漏洞信息为空，则返回该规则
if rule.Vulnerability == "" {
    return &rule
}

// 如果规则中的漏洞信息与语句中的漏洞信息匹配，则返回该规则
if statement.Vulnerability.Matches(rule.Vulnerability) {
    return &rule
}
}
return nil
}

// AugmentMatches 函数用于在加载的 VEX 文档中找到受影响的 VEX 产品的匹配数据时，将结果添加到 match.Matches 数组中。
// 当在 ignoreRules 列表中找不到先前的数据时，匹配数据将从 ignore 列表中移除或合成。
func (ovm *Processor) AugmentMatches(
docRaw interface{}, ignoreRules []match.IgnoreRule, pkgContext *pkg.Context, remainingMatches *match.Matches, ignoredMatches []match.IgnoredMatch,
) (*match.Matches, []match.IgnoredMatch, error) {
// 将 docRaw 转换为 openvex.VEX 类型的 doc，如果转换失败则返回错误
doc, ok := docRaw.(*openvex.VEX)
if !ok {
    return nil, nil, errors.New("unable to cast vex document as openvex")
}

// 创建一个空的 additionalIgnoredMatches 数组
additionalIgnoredMatches := []match.IgnoredMatch{}

// 从 pkgContext 中读取产品标识符，如果出现错误则返回错误
products, err := productIdentifiersFromContext(pkgContext)
if err != nil {
    return nil, nil, fmt.Errorf("reading product identifiers from context: %w", err)
}

// 遍历 ignoredMatches 数组
for i := range ignoredMatches {
    // 声明 statement 和 searchedBy 变量
    var statement *openvex.Statement
    var searchedBy *SearchedBy
    // 从 ignoredMatches[i].Match 中获取子组件标识符
    subcmp := subcomponentIdentifiersFromMatch(&ignoredMatches[i].Match)

    // 遍历产品的不同名称，查看是否与语句数据匹配
		// 遍历产品列表
		for _, product := range products {
			// 检查文档中是否存在与忽略匹配的漏洞ID、产品和子组件匹配的语句
			if matchingStatements := doc.Matches(ignoredMatches[i].Vulnerability.ID, product, subcmp); len(matchingStatements) != 0 {
				// 如果存在匹配的语句，且第一个语句的状态不是“已受影响”或“正在调查中”，则跳出循环
				if matchingStatements[0].Status != openvex.StatusAffected &&
					matchingStatements[0].Status != openvex.StatusUnderInvestigation {
					break
				}
				// 将匹配的语句赋值给statement
				statement = &matchingStatements[0]
				// 设置searchedBy对象的属性
				searchedBy = &SearchedBy{
					Vulnerability: ignoredMatches[i].Vulnerability.ID,
					Product:       product,
					Subcomponents: subcmp,
				}
				// 跳出循环
				break
			}
		}

		// 如果statement为空，表示没有关于这个匹配组件的数据，将忽略匹配添加到additionalIgnoredMatches中，然后继续下一个循环
		if statement == nil {
			additionalIgnoredMatches = append(additionalIgnoredMatches, ignoredMatches[i])
			continue
		// 只有在配置了要增强的规则时才匹配
		rule := matchingRule(ignoreRules, ignoredMatches[i].Match, statement, augmentStatuses)
		// 如果没有匹配到规则，则将该匹配添加到额外忽略的匹配列表中并继续下一个循环
		if rule == nil {
			additionalIgnoredMatches = append(additionalIgnoredMatches, ignoredMatches[i])
			continue
		}

		// 创建新的匹配对象，并添加到详情列表中
		newMatch := ignoredMatches[i].Match
		newMatch.Details = append(newMatch.Details, match.Detail{
			Type:       match.ExactDirectMatch,
			SearchedBy: searchedBy,
			Found: Match{
				Statement: *statement,
			},
			Matcher: match.OpenVexMatcher,
		})

		// 将新的匹配对象添加到剩余匹配列表中
		remainingMatches.Add(newMatch)
# 返回 remainingMatches, additionalIgnoredMatches, 和 nil
```