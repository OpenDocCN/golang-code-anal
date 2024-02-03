# `grype\grype\matcher\apk\matcher.go`

```go
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

type Matcher struct {
}

func (m *Matcher) PackageTypes() []syftPkg.Type {
    return []syftPkg.Type{syftPkg.ApkPkg}
}

func (m *Matcher) Type() match.MatcherType {
    return match.ApkMatcher
}

func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    var matches = make([]match.Match, 0)

    // direct matches with package
    directMatches, err := m.findApkPackage(store, d, p)  // 查找与包直接匹配的漏洞
    if err != nil {
        return nil, err
    }
    matches = append(matches, directMatches...)  // 将直接匹配的漏洞添加到匹配列表中

    // indirect matches with package source
    indirectMatches, err := m.matchBySourceIndirection(store, d, p)  // 通过包源间接匹配漏洞
    if err != nil {
        return nil, err
    }
    matches = append(matches, indirectMatches...)  // 将间接匹配的漏洞添加到匹配列表中

    return matches, nil  // 返回匹配列表
}

func (m *Matcher) cpeMatchesWithoutSecDBFixes(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    // find CPE-indexed vulnerability matches specific to the given package name and version
    cpeMatches, err := search.ByPackageCPE(store, d, p, m.Type())  // 查找特定包名称和版本的CPE索引漏洞匹配
    if err != nil {
        return nil, err
    }

    cpeMatchesByID := matchesByID(cpeMatches)  // 根据ID将CPE匹配组织成映射

    // remove cpe matches where there is an entry in the secDB for the particular package-vulnerability pairing, and the
    // installed package version is >= the fixed in version for the secDB record.
    secDBVulnerabilities, err := store.GetByDistro(d, p)  // 从数据库中获取特定发行版和包的漏洞信息
    if err != nil {
        return nil, err
    }

    secDBVulnerabilitiesByID := vulnerabilitiesByID(secDBVulnerabilities)  // 根据ID将secDB漏洞组织成映射

    verObj, err := version.NewVersionFromPkg(p)  // 从包中创建版本对象
    # 如果错误不为空，则返回空值和格式化的错误信息
    if err != nil:
        return nil, fmt.Errorf("matcher failed to parse version pkg='%s' ver='%s': %w", p.Name, p.Version, err)
    
    # 声明一个空的 match.Match 类型的切片
    var finalCpeMatches []match.Match
# 循环遍历 cpeMatchesByID 中的每个 ID 和对应的 CPE 匹配
cveLoop:
    for id, cpeMatchesForID := range cpeMatchesByID {
        # 检查是否存在与该 ID（CVE）对应的 secDB 条目
        secDBVulnerabilitiesForID, exists := secDBVulnerabilitiesByID[id]
        if !exists:
            # 在 secdb 中不存在该条目，因此应将 CPE 记录添加到最终结果中
            finalCpeMatches = append(finalCpeMatches, cpeMatchesForID...)
            继续下一次循环
            continue
        # 存在 secdb 条目...
        for _, vuln := range secDBVulnerabilitiesForID:
            # ...是否存在修复条目？（应该始终存在）
            if len(vuln.Fix.Versions) == 0:
                继续下一次循环
                continue
            # ...当前软件包是否存在漏洞？
            vulnerable, err := vuln.Constraint.Satisfied(verObj)
            if err != nil:
                返回空值和错误
                return nil, err
            如果存在漏洞
            if vulnerable:
                # 如果至少有一个存在漏洞的条目，则将所有 CPE 记录添加到最终结果中，并继续 cveLoop 循环
                finalCpeMatches = append(finalCpeMatches, cpeMatchesForID...)
                继续 cveLoop 循环
                continue cveLoop
        返回最终的 CPE 匹配结果和空值
    }
}

# 从 secDB 匹配和 cpe 匹配中去重
func deduplicateMatches(secDBMatches, cpeMatches []match.Match) (matches []match.Match):
    # 从 secDB 匹配中按 ID 分组
    secDBMatchesByID := matchesByID(secDBMatches)
    # 从 cpe 匹配中按 ID 分组
    cpeMatchesByID := matchesByID(cpeMatches)
    # 遍历 cpeMatchesByID 中的每个 ID 和对应的 CPE 匹配
    for id, cpeMatchesForID := range cpeMatchesByID:
        # 到这一步，所有匹配都已经验证为相对于漏洞源在给定软件包版本中存在漏洞
        # 现在我们将添加在 secdb 中未找到的唯一 CPE 候选项
        如果 _, exists := secDBMatchesByID[id]; !exists:
            # 添加新的基于 CPE 的记录（例如 NVD），因为它在 secDB 中未找到
            matches = append(matches, cpeMatchesForID...)
    返回匹配结果
    return matches
func matchesByID(matches []match.Match) map[string][]match.Match {
    // 创建一个空的结果字典，用于存储匹配结果
    var results = make(map[string][]match.Match)
    // 遍历匹配结果列表
    for _, secDBMatch := range matches {
        // 将匹配结果按照漏洞ID存储到结果字典中
        results[secDBMatch.Vulnerability.ID] = append(results[secDBMatch.Vulnerability.ID], secDBMatch)
    }
    // 返回结果字典
    return results
}

func vulnerabilitiesByID(vulns []vulnerability.Vulnerability) map[string][]vulnerability.Vulnerability {
    // 创建一个空的结果字典，用于存储漏洞信息
    var results = make(map[string][]vulnerability.Vulnerability)
    // 遍历漏洞列表
    for _, vuln := range vulns {
        // 将漏洞信息按照漏洞ID存储到结果字典中
        results[vuln.ID] = append(results[vuln.ID], vuln)
    }
    // 返回结果字典
    return results
}

func (m *Matcher) findApkPackage(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    // 查找给定包名和版本的 Alpine SecDB 匹配项
    secDBMatches, err := search.ByPackageDistro(store, d, p, m.Type())
    if err != nil {
        return nil, err
    }

    cpeMatches, err := m.cpeMatchesWithoutSecDBFixes(store, d, p)
    if err != nil {
        return nil, err
    }

    var matches []match.Match

    // 保留所有的 secdb 匹配项，因为这是一个权威来源
    matches = append(matches, secDBMatches...)

    // 仅保留唯一的 CPE 匹配项
    matches = append(matches, deduplicateMatches(secDBMatches, cpeMatches)...)
    
    // 返回匹配结果列表
    return matches, nil
}

func (m *Matcher) matchBySourceIndirection(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    var matches []match.Match

    // 遍历上游包列表
    for _, indirectPackage := range pkg.UpstreamPackages(p) {
        // 查找上游包的匹配项
        indirectMatches, err := m.findApkPackage(store, d, indirectPackage)
        if err != nil {
            return nil, fmt.Errorf("failed to find vulnerabilities for apk upstream source package: %w", err)
        }
        // 将上游包的匹配项添加到结果列表中
        matches = append(matches, indirectMatches...)
    }

    // 确保我们跟踪的匹配项是基于 SBOM 中的包（而不是间接包）
    // 将匹配结果转换为间接匹配，并保留间接匹配以备将来参考
    match.ConvertToIndirectMatches(matches, p)
    // 返回匹配结果和空指针
    return matches, nil
# 闭合前面的函数定义
```