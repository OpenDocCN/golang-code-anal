# `grype\grype\matcher\apk\matcher.go`

```
// 导入所需的包
package apk

import (
	"fmt"

	"github.com/anchore/grype/grype/distro"
	"github.com/anchore/grype/grype/match"
	"github.com/anchore/grype/grype/pkg"
	"github.com/anchore/grype/grype/search"
	"github.com/anchore/grype/grype/version"
	"github.com/anchore/grype/grype/vulnerability"
	syftPkg "github.com/anchore/syft/syft/pkg"
)

// 定义 Matcher 结构体
type Matcher struct {
}

// 实现 PackageTypes 方法，返回支持的包类型
func (m *Matcher) PackageTypes() []syftPkg.Type {
	return []syftPkg.Type{syftPkg.ApkPkg}
}
// 返回匹配器的类型
func (m *Matcher) Type() match.MatcherType {
    return match.ApkMatcher
}

// 使用给定的存储、发行版和软件包进行匹配，返回匹配结果和可能的错误
func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    // 创建一个空的匹配结果数组
    var matches = make([]match.Match, 0)

    // 直接使用软件包进行匹配
    directMatches, err := m.findApkPackage(store, d, p)
    if err != nil {
        return nil, err
    }
    // 将直接匹配结果添加到匹配结果数组中
    matches = append(matches, directMatches...)

    // 间接使用软件包源进行匹配
    indirectMatches, err := m.matchBySourceIndirection(store, d, p)
    if err != nil {
        return nil, err
    }
// 将 indirectMatches 切片追加到 matches 切片中
matches = append(matches, indirectMatches...)

// 返回 matches 切片和空值
return matches, nil
}

// 查找与给定软件包名称和版本相关的 CPE 索引漏洞匹配
func (m *Matcher) cpeMatchesWithoutSecDBFixes(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
	// 查找特定软件包名称和版本的 CPE 索引漏洞匹配
	cpeMatches, err := search.ByPackageCPE(store, d, p, m.Type())
	if err != nil {
		return nil, err
	}

	// 通过匹配的漏洞创建一个以漏洞 ID 为键的映射
	cpeMatchesByID := matchesByID(cpeMatches)

	// 获取与给定发行版和软件包相关的 secDB 漏洞
	secDBVulnerabilities, err := store.GetByDistro(d, p)
	if err != nil {
		return nil, err
	}
	// 通过漏洞 ID 从 secDBVulnerabilities 中获取漏洞信息
	secDBVulnerabilitiesByID := vulnerabilitiesByID(secDBVulnerabilities)

	// 从软件包中获取版本信息
	verObj, err := version.NewVersionFromPkg(p)
	if err != nil {
		// 如果解析版本信息失败，则返回错误
		return nil, fmt.Errorf("matcher failed to parse version pkg='%s' ver='%s': %w", p.Name, p.Version, err)
	}

	// 创建一个空的最终匹配结果列表
	var finalCpeMatches []match.Match

// 遍历 cpeMatchesByID 中的每个 CVE
cveLoop:
	for id, cpeMatchesForID := range cpeMatchesByID {
		// 检查是否存在 secDBVulnerabilitiesByID 中的漏洞信息
		secDBVulnerabilitiesForID, exists := secDBVulnerabilitiesByID[id]
		if !exists {
			// 如果不存在于 secDBVulnerabilitiesByID 中，则将 CPE 记录添加到最终结果列表中
			finalCpeMatches = append(finalCpeMatches, cpeMatchesForID...)
			continue
		}

// 遍历安全数据库中与给定ID相关的漏洞条目
for _, vuln := range secDBVulnerabilitiesForID {
    // 检查是否存在修复版本的条目（应该始终存在）
    if len(vuln.Fix.Versions) == 0 {
        continue
    }

    // 检查当前软件包是否存在漏洞
    vulnerable, err := vuln.Constraint.Satisfied(verObj)
    if err != nil {
        return nil, err
    }

    if vulnerable {
        // 如果至少有一个漏洞条目，则将所有CPE记录添加到最终结果中
        finalCpeMatches = append(finalCpeMatches, cpeMatchesForID...)
        continue cveLoop
    }
}
// deduplicateMatches 函数用于去重匹配结果，从CPE源中添加额外的唯一匹配项，这些匹配项与SecDB匹配项不同
func deduplicateMatches(secDBMatches, cpeMatches []match.Match) (matches []match.Match) {
    // 通过 matchesByID 函数将 secDBMatches 和 cpeMatches 转换为以 ID 为键的 map
    secDBMatchesByID := matchesByID(secDBMatches)
    cpeMatchesByID := matchesByID(cpeMatches)
    // 遍历 cpeMatchesByID，找出在 secDBMatchesByID 中不存在的唯一 CPE 候选项
    for id, cpeMatchesForID := range cpeMatchesByID {
        // 在此之前，所有匹配项都已经经过验证，相对于漏洞源，它们都是在给定软件包版本中存在漏洞的
        // 现在我们将添加在 secDB 中未找到的唯一 CPE 候选项
        if _, exists := secDBMatchesByID[id]; !exists {
            // 将新的基于 CPE 的记录（例如 NVD）添加到匹配结果中，因为它在 secDB 中未找到
            matches = append(matches, cpeMatchesForID...)
        }
    }
    return matches
}

// matchesByID 函数用于将匹配结果转换为以 ID 为键的 map
func matchesByID(matches []match.Match) map[string][]match.Match {
    var results = make(map[string][]match.Match)
// 遍历匹配结果，将漏洞ID和匹配结果添加到结果map中
for _, secDBMatch := range matches {
    results[secDBMatch.Vulnerability.ID] = append(results[secDBMatch.Vulnerability.ID], secDBMatch)
}
// 返回结果map
return results
}

// 根据漏洞列表生成漏洞ID到漏洞列表的映射
func vulnerabilitiesByID(vulns []vulnerability.Vulnerability) map[string][]vulnerability.Vulnerability {
    var results = make(map[string][]vulnerability.Vulnerability)
    // 遍历漏洞列表，将漏洞ID和漏洞添加到结果map中
    for _, vuln := range vulns {
        results[vuln.ID] = append(results[vuln.ID], vuln)
    }
    // 返回结果map
    return results
}

// 查找APK包的安全数据库匹配
func (m *Matcher) findApkPackage(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    // 查找给定包名和版本的Alpine SecDB匹配
    secDBMatches, err := search.ByPackageDistro(store, d, p, m.Type())
    if err != nil {
        return nil, err
	}

	// 通过调用cpeMatchesWithoutSecDBFixes方法获取CPE匹配结果
	cpeMatches, err := m.cpeMatchesWithoutSecDBFixes(store, d, p)
	// 如果出现错误，返回空值和错误信息
	if err != nil {
		return nil, err
	}

	// 声明一个空的匹配结果数组
	var matches []match.Match

	// 将secDBMatches的匹配结果添加到matches数组中，作为权威数据源
	matches = append(matches, secDBMatches...)

	// 保留唯一的CPE匹配结果
	// 将secDBMatches和cpeMatches的匹配结果去重后添加到matches数组中
	matches = append(matches, deduplicateMatches(secDBMatches, cpeMatches)...)

	// 返回匹配结果数组和空值
	return matches, nil
}

// 根据源间接引用进行匹配
func (m *Matcher) matchBySourceIndirection(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
	// 声明一个空的匹配结果数组
	var matches []match.Match
# 遍历 pkg.UpstreamPackages(p) 中的间接包
for _, indirectPackage := range pkg.UpstreamPackages(p) {
    # 查找间接包的漏洞匹配
    indirectMatches, err := m.findApkPackage(store, d, indirectPackage)
    # 如果查找失败，返回错误
    if err != nil {
        return nil, fmt.Errorf("failed to find vulnerabilities for apk upstream source package: %w", err)
    }
    # 将间接包的漏洞匹配添加到总的匹配列表中
    matches = append(matches, indirectMatches...)
}

# 将匹配转换为间接匹配，以便基于 SBOM 中的包进行跟踪
match.ConvertToIndirectMatches(matches, p)

# 返回匹配列表和空错误
return matches, nil
```