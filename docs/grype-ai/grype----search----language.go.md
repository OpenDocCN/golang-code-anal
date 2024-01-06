# `grype\grype\search\language.go`

```
package search

import (
	"fmt"  // 导入 fmt 包，用于格式化输出

	"github.com/anchore/grype/grype/distro"  // 导入 distro 包，用于处理操作系统发行版信息
	"github.com/anchore/grype/grype/match"  // 导入 match 包，用于匹配漏洞
	"github.com/anchore/grype/grype/pkg"  // 导入 pkg 包，用于处理软件包信息
	"github.com/anchore/grype/grype/version"  // 导入 version 包，用于处理软件版本信息
	"github.com/anchore/grype/grype/vulnerability"  // 导入 vulnerability 包，用于处理漏洞信息
)

// 根据软件包语言和操作系统发行版获取匹配的漏洞信息
func ByPackageLanguage(store vulnerability.ProviderByLanguage, d *distro.Distro, p pkg.Package, upstreamMatcher match.MatcherType) ([]match.Match, error) {
	// 从软件包信息中创建版本对象
	verObj, err := version.NewVersionFromPkg(p)
	if err != nil {
		// 如果创建版本对象失败，则返回错误信息
		return nil, fmt.Errorf("matcher failed to parse version pkg=%q ver=%q: %w", p.Name, p.Version, err)
	}

	// 根据软件包语言和软件包信息获取所有的漏洞信息
	allPkgVulns, err := store.GetByLanguage(p.Language, p)
	if err != nil {
	// 如果匹配器无法获取语言和包的信息，则返回错误
		return nil, fmt.Errorf("matcher failed to fetch language=%q pkg=%q: %w", p.Language, p.Name, err)
	}

	// 从所有包的漏洞中筛选出符合条件的漏洞
	applicableVulns, err := onlyQualifiedPackages(d, p, allPkgVulns)
	if err != nil {
		// 如果筛选失败，则返回错误
		return nil, fmt.Errorf("unable to filter language-related vulnerabilities: %w", err)
	}

	// TODO: 将此部分迁移到限定符，并删除
	// 仅保留与版本相关的漏洞，并更新applicableVulns
	applicableVulns, err = onlyVulnerableVersions(verObj, applicableVulns)
	if err != nil {
		// 如果筛选失败，则返回错误
		return nil, fmt.Errorf("unable to filter language-related vulnerabilities: %w", err)
	}

	// 创建一个空的匹配列表
	var matches []match.Match
	// 遍历符合条件的漏洞
	for _, vuln := range applicableVulns {
		// 将匹配结果添加到匹配列表中
		matches = append(matches, match.Match{
			Vulnerability: vuln,
			Package:       p,
// 创建一个空的匹配详情列表
Details: []match.Detail{
    // 创建一个具体的匹配详情
    {
        // 设置匹配类型为精确直接匹配
        Type:       match.ExactDirectMatch,
        // 设置匹配的置信度为1.0，暂时是硬编码的
        Confidence: 1.0, // TODO: this is hard coded for now
        // 设置匹配器为upstreamMatcher
        Matcher:    upstreamMatcher,
        // 设置搜索条件，包括语言、命名空间和包的名称和版本
        SearchedBy: map[string]interface{}{
            "language":  string(p.Language),
            "namespace": vuln.Namespace,
            "package": map[string]string{
                "name":    p.Name,
                "version": p.Version,
            },
        },
        // 设置找到的漏洞信息，包括漏洞ID和版本约束
        Found: map[string]interface{}{
            "vulnerabilityID":   vuln.ID,
            "versionConstraint": vuln.Constraint.String(),
        },
    },
},
# 返回匹配结果和错误信息
return matches, err
```