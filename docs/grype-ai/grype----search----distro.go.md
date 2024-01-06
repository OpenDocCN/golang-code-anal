# `grype\grype\search\distro.go`

```
package search

import (
	"fmt"

	"github.com/anchore/grype/grype/distro"
	"github.com/anchore/grype/grype/match"
	"github.com/anchore/grype/grype/pkg"
	"github.com/anchore/grype/grype/version"
	"github.com/anchore/grype/grype/vulnerability"
)

// ByPackageDistro 根据包和发行版进行搜索
func ByPackageDistro(store vulnerability.ProviderByDistro, d *distro.Distro, p pkg.Package, upstreamMatcher match.MatcherType) ([]match.Match, error) {
	// 如果发行版为空，返回空结果
	if d == nil {
		return nil, nil
	}

	// 从包中创建版本对象
	verObj, err := version.NewVersionFromPkg(p)
	if err != nil {
		// 如果创建版本对象失败，返回错误
		return nil, fmt.Errorf("matcher failed to parse version pkg=%q ver=%q: %w", p.Name, p.Version, err)
	}

	// 通过发行版和软件包名称获取所有的漏洞信息
	allPkgVulns, err := store.GetByDistro(d, p)
	if err != nil {
		// 如果获取失败，返回错误信息
		return nil, fmt.Errorf("matcher failed to fetch distro=%q pkg=%q: %w", d, p.Name, err)
	}

	// 通过发行版和软件包名称筛选出适用的漏洞信息
	applicableVulns, err := onlyQualifiedPackages(d, p, allPkgVulns)
	if err != nil {
		// 如果筛选失败，返回错误信息
		return nil, fmt.Errorf("unable to filter distro-related vulnerabilities: %w", err)
	}

	// TODO: 将此部分移植到一个限定器中，并删除
	// 通过版本对象和适用的漏洞信息筛选出受影响的版本
	applicableVulns, err = onlyVulnerableVersions(verObj, applicableVulns)
	if err != nil {
		// 如果筛选失败，返回错误信息
		return nil, fmt.Errorf("unable to filter distro-related vulnerabilities: %w", err)
	}

	// 创建一个匹配数组
	var matches []match.Match
	// 遍历适用的漏洞信息
	for _, vuln := range applicableVulns {
		# 将匹配结果添加到匹配列表中
		matches = append(matches, match.Match{
			# 漏洞信息
			Vulnerability: vuln,
			# 包信息
			Package:       p,
			# 匹配的详细信息
			Details: []match.Detail{
				{
					# 匹配类型为精确直接匹配
					Type:    match.ExactDirectMatch,
					# 使用的匹配器
					Matcher: upstreamMatcher,
					# 通过哪些信息进行搜索
					SearchedBy: map[string]interface{}{
						# 操作系统类型和版本
						"distro": map[string]string{
							"type":    d.Type.String(),
							"version": d.RawVersion,
						},
						# 为什么包括包信息？给定的包可能是另一个已安装在系统上的包的源包。这样可以清楚地表明搜索时使用了什么信息。
						"package": map[string]string{
							"name":    p.Name,
							"version": p.Version,
						},
						# 漏洞的命名空间
						"namespace": vuln.Namespace,
					},
					Found: map[string]interface{}{
						// 设置漏洞ID和版本约束的键值对
						"vulnerabilityID":   vuln.ID,
						"versionConstraint": vuln.Constraint.String(),
					},
					Confidence: 1.0, // 设置匹配的置信度，暂时是硬编码的值
				},
			},
		})
	}

	// 返回匹配结果和可能的错误
	return matches, err
}
```