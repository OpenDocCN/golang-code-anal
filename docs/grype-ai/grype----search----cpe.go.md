# `grype\grype\search\cpe.go`

```
package search

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"sort"  // 导入 sort 包，用于对数据进行排序
	"strings"  // 导入 strings 包，用于处理字符串

	"github.com/facebookincubator/nvdtools/wfn"  // 导入 wfn 包
	"github.com/scylladb/go-set/strset"  // 导入 strset 包，用于操作字符串集合

	"github.com/anchore/grype/grype/distro"  // 导入 distro 包
	"github.com/anchore/grype/grype/match"  // 导入 match 包
	"github.com/anchore/grype/grype/pkg"  // 导入 pkg 包
	"github.com/anchore/grype/grype/version"  // 导入 version 包
	"github.com/anchore/grype/grype/vulnerability"  // 导入 vulnerability 包
	"github.com/anchore/syft/syft/cpe"  // 导入 cpe 包
	syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包，并重命名为 syftPkg

)

type CPEPackageParameter struct {
// Name字段表示名称，使用json标签指定在JSON序列化和反序列化时的字段名
Name    string `json:"name"`

// Version字段表示版本，使用json标签指定在JSON序列化和反序列化时的字段名
Version string `json:"version"`
}

// CPEParameters结构体表示CPE参数，包含命名空间、CPE列表和CPE包参数
type CPEParameters struct {
    // Namespace字段表示命名空间，使用json标签指定在JSON序列化和反序列化时的字段名
    Namespace string   `json:"namespace"`
    // CPEs字段表示CPE列表，使用json标签指定在JSON序列化和反序列化时的字段名
    CPEs      []string `json:"cpes"`
    // Package字段表示CPE包参数
    Package   CPEPackageParameter
}

// Merge方法用于合并两个CPEParameters对象
func (i *CPEParameters) Merge(other CPEParameters) error {
    // 检查命名空间是否匹配，不匹配则返回错误
    if i.Namespace != other.Namespace {
        return fmt.Errorf("namespaces do not match")
    }

    // 使用strset.New创建现有CPE列表和新CPE列表的集合
    existingCPEs := strset.New(i.CPEs...)
    newCPEs := strset.New(other.CPEs...)
    // 合并两个集合并排序
    mergedCPEs := strset.Union(existingCPEs, newCPEs).List()
    sort.Strings(mergedCPEs)
    // 更新当前对象的CPE列表为合并后的列表
    i.CPEs = mergedCPEs
}
	return nil
}
// 定义 CPEResult 结构体，包含漏洞ID、版本约束和CPE列表
type CPEResult struct {
	VulnerabilityID   string   `json:"vulnerabilityID"`
	VersionConstraint string   `json:"versionConstraint"`
	CPEs              []string `json:"cpes"`
}

// 定义 Equals 方法，用于比较两个 CPEResult 对象是否相等
func (h CPEResult) Equals(other CPEResult) bool {
	// 检查版本约束是否相等，如果不相等则返回false
	if h.VersionConstraint != other.VersionConstraint {
		return false
	}

	// 检查CPE列表长度是否相等，如果不相等则返回false
	if len(h.CPEs) != len(other.CPEs) {
		return false
	}

	// 遍历CPE列表，逐个比较对应位置的CPE是否相等
	for i := range h.CPEs {
		if h.CPEs[i] != other.CPEs[i] {
// 返回 false
return false
// 如果版本号包含 -r，则清理 alpine 包版本，以便与 CPE 版本比较逻辑正确比较
// alpine 版本后缀为 -r{buildindex}；但是，如果保持不变，CPE 比较逻辑将错误地将其视为预发布版本。
// 实际上，我们只想将 1.2.3-r21 视为等同于 1.2.3，以便基于 CPE 的匹配目的，因为 alpine 修复应该过滤掉任何后续构建修复了 1.2.3 中的漏洞的情况
func alpineCPEComparableVersion(version string) string {
    // 使用 -r 分割版本号
    components := strings.Split(version, "-r")
    // 默认使用原始版本号
    cpeComparableVersion := version

    // 如果分割后有两部分，则使用第一部分作为比较版本号
    if len(components) == 2 {
        cpeComparableVersion = components[0]
    }
	return cpeComparableVersion
}

// ByPackageCPE retrieves all vulnerabilities that match the generated CPE
func ByPackageCPE(store vulnerability.ProviderByCPE, d *distro.Distro, p pkg.Package, upstreamMatcher match.MatcherType) ([]match.Match, error) {
	// we attempt to merge match details within the same matcher when searching by CPEs, in this way there are fewer duplicated match
	// objects (and fewer duplicated match details).
	// 创建一个空的匹配详情的映射，用于存储匹配的指纹和匹配对象
	matchesByFingerprint := make(map[match.Fingerprint]match.Match)
	// 遍历包的CPE列表
	for _, c := range p.CPEs {
		// 优先使用CPE版本，如果未指定，则使用包的版本
		searchVersion := c.Version

		// 如果包类型为Apk包，则使用特定的方法转换版本号为可比较的版本
		if p.Type == syftPkg.ApkPkg {
			searchVersion = alpineCPEComparableVersion(searchVersion)
		}

		// 如果搜索版本为NA或Any，则使用包的版本
		if searchVersion == wfn.NA || searchVersion == wfn.Any {
			searchVersion = p.Version
		}
		// 创建一个新的版本对象，用于比较版本号
		verObj, err := version.NewVersion(searchVersion, version.FormatFromPkgType(p.Type))
		// 如果发生错误，返回错误信息，包括包名和版本号
		if err != nil {
			return nil, fmt.Errorf("matcher failed to parse version pkg=%q ver=%q: %w", p.Name, p.Version, err)
		}

		// 在数据库中查找给定CPE的所有漏洞记录（不包括版本比较）
		allPkgVulns, err := store.GetByCPE(c)
		if err != nil {
			return nil, fmt.Errorf("matcher failed to fetch by CPE pkg=%q: %w", p.Name, err)
		}

		// 仅获取符合条件的漏洞记录
		applicableVulns, err := onlyQualifiedPackages(d, p, allPkgVulns)
		if err != nil {
			return nil, fmt.Errorf("unable to filter cpe-related vulnerabilities: %w", err)
		}

		// TODO: 将此部分移植到一个限定符中，并删除
		// 仅获取有漏洞的版本
		applicableVulns, err = onlyVulnerableVersions(verObj, applicableVulns)
		if err != nil {
			return nil, fmt.Errorf("unable to filter cpe-related vulnerabilities: %w", err)
		}
# 将只有漏洞的目标添加到适用漏洞列表中
applicableVulns = onlyVulnerableTargets(p, applicableVulns)

# 对于找到的每个漏洞记录，检查版本约束。如果相对于当前的CPE（或软件包）的版本信息满足约束，则该软件包是有漏洞的。
for _, vuln := range applicableVulns:
    addNewMatch(matchesByFingerprint, vuln, p, *verObj, upstreamMatcher, c)

# 返回匹配结果和空值
return toMatches(matchesByFingerprint), nil

# 创建候选匹配对象
candidateMatch := match.Match{
    Vulnerability: vuln,
    Package: p,
}
// 检查是否存在相同指纹的匹配，如果存在则使用已存在的匹配
if existingMatch, exists := matchesByFingerprint[candidateMatch.Fingerprint()]; exists {
    candidateMatch = existingMatch
}

// 添加匹配的详细信息
candidateMatch.Details = addMatchDetails(candidateMatch.Details,
    match.Detail{
        Type:       match.CPEMatch, // 匹配类型为CPEMatch
        Confidence: 0.9, // 置信度为0.9，暂时硬编码
        Matcher:    upstreamMatcher, // 匹配器为upstreamMatcher
        SearchedBy: CPEParameters{ // 使用CPE参数进行搜索
            Namespace: vuln.Namespace, // 命名空间为vuln的命名空间
            CPEs: []string{
                searchedByCPE.BindToFmtString(), // 将searchedByCPE绑定为格式化字符串
            },
            Package: CPEPackageParameter{ // CPE包参数
                Name:    p.Name, // 包名为p的名称
                Version: p.Version, // 版本为p的版本
            },
        },
        Found: CPEResult{ // 找到的CPE结果
            // 其他详细信息
        },
    })
# 将漏洞ID、版本约束和CPEs添加到匹配项中
VulnerabilityID:   vuln.ID,
VersionConstraint: vuln.Constraint.String(),
CPEs:              cpesToString(filterCPEsByVersion(searchVersion, vuln.CPEs)),
```

```
# 将候选匹配的指纹和匹配项添加到匹配项的映射中
matchesByFingerprint[candidateMatch.Fingerprint()] = candidateMatch
```

```
# 将新的匹配项细节添加到现有的匹配项细节中
func addMatchDetails(existingDetails []match.Detail, newDetails match.Detail) []match.Detail {
    # 将新的匹配项细节转换为CPEResult类型，如果转换失败则返回现有的匹配项细节
    newFound, ok := newDetails.Found.(CPEResult)
    if !ok {
        return existingDetails
    }

    # 将新的匹配项细节中的搜索参数转换为CPEParameters类型，如果转换失败则返回现有的匹配项细节
    newSearchedBy, ok := newDetails.SearchedBy.(CPEParameters)
    if !ok {
        return existingDetails
    }
# 遍历 existingDetails 列表，获取索引和元素值
for idx, detail := range existingDetails:
    # 尝试将 detail.Found 转换为 CPEResult 类型，如果转换成功，found 为转换后的值，ok 为 true
    found, ok := detail.Found.(CPEResult)
    # 如果转换失败，继续下一次循环
    if !ok:
        continue

    # 尝试将 detail.SearchedBy 转换为 CPEParameters 类型，如果转换成功，searchedBy 为转换后的值，ok 为 true
    searchedBy, ok := detail.SearchedBy.(CPEParameters)
    # 如果转换失败，继续下一次循环
    if !ok:
        continue

    # 如果 found 与 newFound 不相等，继续下一次循环
    if !found.Equals(newFound):
        continue

    # 将 newSearchedBy 合并到 searchedBy 中
    err := searchedBy.Merge(newSearchedBy)
    # 如果合并过程中出现错误，继续下一次循环
    if err != nil:
        continue
		// 将搜索到的信息更新到现有细节中的SearchedBy字段
		existingDetails[idx].SearchedBy = searchedBy
		// 返回更新后的现有细节
		return existingDetails
	}

	// 无法与其他条目合并，将新细节追加到末尾
	existingDetails = append(existingDetails, newDetails)
	// 返回更新后的现有细节
	return existingDetails
}

// 根据包版本过滤CPEs
func filterCPEsByVersion(pkgVersion version.Version, allCPEs []cpe.CPE) (matchedCPEs []cpe.CPE) {
	// 遍历所有的CPEs
	for _, c := range allCPEs {
		// 如果CPE的版本是任意或者不适用，则将其加入匹配的CPEs中
		if c.Version == wfn.Any || c.Version == wfn.NA {
			matchedCPEs = append(matchedCPEs, c)
			continue
		}

		// 尝试获取版本约束
		constraint, err := version.GetConstraint(c.Version, version.UnknownFormat)
		// 如果无法获取版本约束，则不过滤掉该CPE
		if err != nil {
			matchedCPEs = append(matchedCPEs, c)
// 继续循环下一个元素
continue
// 使用约束条件检查包版本是否满足要求
satisfied, err := constraint.Satisfied(&pkgVersion)
// 如果无法检查版本满足性，或者满足要求，则不过滤掉CPE
if err != nil || satisfied {
    matchedCPEs = append(matchedCPEs, c)
    continue
}
// 返回匹配的CPE列表
return matchedCPEs
// 将匹配结果按指纹转换为匹配列表
func toMatches(matchesByFingerprint map[match.Fingerprint]match.Match) (matches []match.Match) {
    for _, m := range matchesByFingerprint {
        matches = append(matches, m)
    }
    // 按元素排序匹配列表
    sort.Sort(match.ByElements(matches))
    return matches
}
// cpesToString接收一个或多个CPE，并将它们转换为字符串
func cpesToString(cpes []cpe.CPE) []string {
    // 创建一个与CPE数量相同的字符串切片
    var strs = make([]string, len(cpes))
    // 遍历CPE切片，将每个CPE转换为格式化字符串并存储在对应位置
    for idx, c := range cpes {
        strs[idx] = c.BindToFmtString()
    }
    // 对字符串切片进行排序
    sort.Strings(strs)
    // 返回结果字符串切片
    return strs
}
```