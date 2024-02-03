# `grype\grype\search\cpe.go`

```go
package search

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "sort" // 导入 sort 包，用于对切片进行排序
    "strings" // 导入 strings 包，用于处理字符串

    "github.com/facebookincubator/nvdtools/wfn" // 导入第三方包 wfn
    "github.com/scylladb/go-set/strset" // 导入第三方包 strset

    "github.com/anchore/grype/grype/distro" // 导入 anchore/grype/grype/distro 包
    "github.com/anchore/grype/grype/match" // 导入 anchore/grype/grype/match 包
    "github.com/anchore/grype/grype/pkg" // 导入 anchore/grype/grype/pkg 包
    "github.com/anchore/grype/grype/version" // 导入 anchore/grype/grype/version 包
    "github.com/anchore/grype/grype/vulnerability" // 导入 anchore/grype/grype/vulnerability 包
    "github.com/anchore/syft/syft/cpe" // 导入 anchore/syft/syft/cpe 包
    syftPkg "github.com/anchore/syft/syft/pkg" // 导入 anchore/syft/syft/pkg 包
)

// 定义 CPEPackageParameter 结构体
type CPEPackageParameter struct {
    Name    string `json:"name"` // 结构体字段 Name，用于表示包名
    Version string `json:"version"` // 结构体字段 Version，用于表示包版本
}

// 定义 CPEParameters 结构体
type CPEParameters struct {
    Namespace string   `json:"namespace"` // 结构体字段 Namespace，用于表示命名空间
    CPEs      []string `json:"cpes"` // 结构体字段 CPEs，用于表示 CPE 列表
    Package   CPEPackageParameter // 结构体字段 Package，用于表示 CPEPackageParameter 结构体
}

// 定义 CPEParameters 结构体的 Merge 方法
func (i *CPEParameters) Merge(other CPEParameters) error {
    if i.Namespace != other.Namespace { // 如果命名空间不匹配
        return fmt.Errorf("namespaces do not match") // 返回错误信息
    }

    existingCPEs := strset.New(i.CPEs...) // 使用 i.CPEs 创建 strset 集合
    newCPEs := strset.New(other.CPEs...) // 使用 other.CPEs 创建 strset 集合
    mergedCPEs := strset.Union(existingCPEs, newCPEs).List() // 合并两个集合并转换为列表
    sort.Strings(mergedCPEs) // 对列表进行排序
    i.CPEs = mergedCPEs // 将合并后的列表赋值给 i.CPEs
    return nil // 返回空值
}

// 定义 CPEResult 结构体
type CPEResult struct {
    VulnerabilityID   string   `json:"vulnerabilityID"` // 结构体字段 VulnerabilityID，用于表示漏洞 ID
    VersionConstraint string   `json:"versionConstraint"` // 结构体字段 VersionConstraint，用于表示版本约束
    CPEs              []string `json:"cpes"` // 结构体字段 CPEs，用于表示 CPE 列表
}

// 定义 CPEResult 结构体的 Equals 方法
func (h CPEResult) Equals(other CPEResult) bool {
    if h.VersionConstraint != other.VersionConstraint { // 如果版本约束不匹配
        return false // 返回 false
    }

    if len(h.CPEs) != len(other.CPEs) { // 如果 CPE 列表长度不匹配
        return false // 返回 false
    }

    for i := range h.CPEs { // 遍历 CPE 列表
        if h.CPEs[i] != other.CPEs[i] { // 如果对应位置的 CPE 不匹配
            return false // 返回 false
        }
    }

    return true // 返回 true
}

// 定义 alpineCPEComparableVersion 函数
func alpineCPEComparableVersion(version string) string {
    // 清理 alpine 包版本，以便与 CPE 版本比较逻辑正确比较
    // alpine 版本后缀为 -r{buildindex}；但是，如果保持不变，CPE 比较逻辑将错误地将其视为预发布版本。
    // 实际上，我们只想将 1.2.3-r21 视为等效的
    // 根据CPE匹配的目的，将版本号按照"-r"进行分割
    components := strings.Split(version, "-r")
    // 初始化CPE可比较版本号为原始版本号
    cpeComparableVersion := version
    
    // 如果分割后的组件长度为2，说明版本号中包含"-r"，则将CPE可比较版本号设置为分割后的第一个组件
    if len(components) == 2 {
        cpeComparableVersion = components[0]
    }
    
    // 返回CPE可比较版本号
    return cpeComparableVersion
// 通过包的CPE检索与生成的CPE匹配的所有漏洞
func ByPackageCPE(store vulnerability.ProviderByCPE, d *distro.Distro, p pkg.Package, upstreamMatcher match.MatcherType) ([]match.Match, error) {
    // 当通过CPE搜索时，我们尝试在相同的匹配器中合并匹配细节，以便减少重复的匹配对象（和重复的匹配细节）。
    matchesByFingerprint := make(map[match.Fingerprint]match.Match)
    // 遍历 p.CPEs 列表中的每个元素，使用下划线 _ 忽略索引值，c 为当前元素
    for _, c := range p.CPEs {
        // 优先使用 CPE 的版本，如果未指定则使用包的版本
        searchVersion := c.Version

        // 如果包类型为 syftPkg.ApkPkg，则将 searchVersion 转换为 Alpine 包可比较的版本
        if p.Type == syftPkg.ApkPkg {
            searchVersion = alpineCPEComparableVersion(searchVersion)
        }

        // 如果 searchVersion 为 "NA" 或 "ANY"，则将其替换为包的版本 p.Version
        if searchVersion == wfn.NA || searchVersion == wfn.Any {
            searchVersion = p.Version
        }
        // 使用版本信息创建 Version 对象
        verObj, err := version.NewVersion(searchVersion, version.FormatFromPkgType(p.Type))
        // 如果创建 Version 对象时发生错误，则返回错误信息
        if err != nil {
            return nil, fmt.Errorf("matcher failed to parse version pkg=%q ver=%q: %w", p.Name, p.Version, err)
        }

        // 从数据库中获取给定 CPE 的所有漏洞记录（不包括版本比较）
        allPkgVulns, err := store.GetByCPE(c)
        // 如果获取过程中发生错误，则返回错误信息
        if err != nil {
            return nil, fmt.Errorf("matcher failed to fetch by CPE pkg=%q: %w", p.Name, err)
        }

        // 仅保留符合条件的漏洞记录
        applicableVulns, err := onlyQualifiedPackages(d, p, allPkgVulns)
        // 如果过滤过程中发生错误，则返回错误信息
        if err != nil {
            return nil, fmt.Errorf("unable to filter cpe-related vulnerabilities: %w", err)
        }

        // TODO: 将此部分移植到限定符中，并删除
        // 仅保留符合版本条件的漏洞记录
        applicableVulns, err = onlyVulnerableVersions(verObj, applicableVulns)
        // 如果过滤过程中发生错误，则返回错误信息
        if err != nil {
            return nil, fmt.Errorf("unable to filter cpe-related vulnerabilities: %w", err)
        }

        // 仅保留符合目标条件的漏洞记录
        applicableVulns = onlyVulnerableTargets(p, applicableVulns)

        // 遍历所有符合条件的漏洞记录，检查版本约束条件，如果满足则将该包标记为有漏洞
        for _, vuln := range applicableVulns {
            addNewMatch(matchesByFingerprint, vuln, p, *verObj, upstreamMatcher, c)
        }
    }

    // 将匹配结果转换为 Matches 对象，并返回
    return toMatches(matchesByFingerprint), nil
# 添加新的匹配项到指纹到匹配项的映射中
func addNewMatch(matchesByFingerprint map[match.Fingerprint]match.Match, vuln vulnerability.Vulnerability, p pkg.Package, searchVersion version.Version, upstreamMatcher match.MatcherType, searchedByCPE cpe.CPE) {
    # 创建一个候选匹配项
    candidateMatch := match.Match{
        Vulnerability: vuln,
        Package:       p,
    }

    # 检查候选匹配项是否已存在，如果存在则使用已存在的匹配项
    if existingMatch, exists := matchesByFingerprint[candidateMatch.Fingerprint()]; exists {
        candidateMatch = existingMatch
    }

    # 添加匹配项的详细信息
    candidateMatch.Details = addMatchDetails(candidateMatch.Details,
        match.Detail{
            Type:       match.CPEMatch,
            Confidence: 0.9, // TODO: this is hard coded for now
            Matcher:    upstreamMatcher,
            SearchedBy: CPEParameters{
                Namespace: vuln.Namespace,
                CPEs: []string{
                    searchedByCPE.BindToFmtString(),
                },
                Package: CPEPackageParameter{
                    Name:    p.Name,
                    Version: p.Version,
                },
            },
            Found: CPEResult{
                VulnerabilityID:   vuln.ID,
                VersionConstraint: vuln.Constraint.String(),
                CPEs:              cpesToString(filterCPEsByVersion(searchVersion, vuln.CPEs)),
            },
        },
    )

    # 更新指纹到匹配项的映射
    matchesByFingerprint[candidateMatch.Fingerprint()] = candidateMatch
}

# 添加新的匹配项的详细信息到已存在的详细信息列表中
func addMatchDetails(existingDetails []match.Detail, newDetails match.Detail) []match.Detail {
    # 检查新的详细信息中的 Found 字段是否为 CPEResult 类型
    newFound, ok := newDetails.Found.(CPEResult)
    if !ok {
        return existingDetails
    }

    # 检查新的详细信息中的 SearchedBy 字段是否为 CPEParameters 类型
    newSearchedBy, ok := newDetails.SearchedBy.(CPEParameters)
    if !ok {
        return existingDetails
    }
    // 遍历 existingDetails 数组，获取索引和元素值
    for idx, detail := range existingDetails {
        // 尝试将 detail.Found 转换为 CPEResult 类型，如果失败则跳过本次循环
        found, ok := detail.Found.(CPEResult)
        if !ok {
            continue
        }

        // 尝试将 detail.SearchedBy 转换为 CPEParameters 类型，如果失败则跳过本次循环
        searchedBy, ok := detail.SearchedBy.(CPEParameters)
        if !ok {
            continue
        }

        // 如果 found 与 newFound 不相等，则跳过本次循环
        if !found.Equals(newFound) {
            continue
        }

        // 尝试将 newSearchedBy 合并到 searchedBy 中，如果失败则跳过本次循环
        err := searchedBy.Merge(newSearchedBy)
        if err != nil {
            continue
        }

        // 将合并后的 searchedBy 赋值给 existingDetails 中对应索引的元素
        existingDetails[idx].SearchedBy = searchedBy
        // 返回更新后的 existingDetails
        return existingDetails
    }

    // 无法与其他条目合并，将新条目追加到数组末尾
    existingDetails = append(existingDetails, newDetails)
    // 返回更新后的 existingDetails
    return existingDetails
# 根据给定的包版本和所有的CPE列表，筛选出符合条件的CPE列表并返回
func filterCPEsByVersion(pkgVersion version.Version, allCPEs []cpe.CPE) (matchedCPEs []cpe.CPE) {
    # 遍历所有的CPE列表
    for _, c := range allCPEs {
        # 如果CPE的版本是任意或者不适用，则将其添加到匹配的CPE列表中并继续下一个循环
        if c.Version == wfn.Any || c.Version == wfn.NA {
            matchedCPEs = append(matchedCPEs, c)
            continue
        }

        # 尝试获取CPE版本的约束条件，如果失败则不过滤该CPE
        constraint, err := version.GetConstraint(c.Version, version.UnknownFormat)
        if err != nil {
            matchedCPEs = append(matchedCPEs, c)
            continue
        }

        # 检查包版本是否满足CPE的约束条件，如果无法检查或者满足则不过滤该CPE
        satisfied, err := constraint.Satisfied(&pkgVersion)
        if err != nil || satisfied {
            matchedCPEs = append(matchedCPEs, c)
            continue
        }
    }
    return matchedCPEs
}

# 将匹配的指纹映射转换为匹配列表，并按元素排序后返回
func toMatches(matchesByFingerprint map[match.Fingerprint]match.Match) (matches []match.Match) {
    # 遍历匹配的指纹映射并将其转换为匹配列表
    for _, m := range matchesByFingerprint {
        matches = append(matches, m)
    }
    # 按元素排序匹配列表
    sort.Sort(match.ByElements(matches))
    return matches
}

# 将CPE列表转换为字符串列表并返回
# cpesToString接收一个或多个CPE并将它们转换为字符串
func cpesToString(cpes []cpe.CPE) []string {
    # 创建一个与CPE列表长度相同的字符串列表
    var strs = make([]string, len(cpes))
    # 遍历CPE列表并将每个CPE转换为格式化字符串
    for idx, c := range cpes {
        strs[idx] = c.BindToFmtString()
    }
    # 对字符串列表进行排序
    sort.Strings(strs)
    return strs
}
```