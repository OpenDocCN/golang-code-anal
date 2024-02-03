# `grype\grype\search\language.go`

```go
package search

import (
    "fmt"

    "github.com/anchore/grype/grype/distro"
    "github.com/anchore/grype/grype/match"
    "github.com/anchore/grype/grype/pkg"
    "github.com/anchore/grype/grype/version"
    "github.com/anchore/grype/grype/vulnerability"
)

func ByPackageLanguage(store vulnerability.ProviderByLanguage, d *distro.Distro, p pkg.Package, upstreamMatcher match.MatcherType) ([]match.Match, error) {
    // 从包对象中创建版本对象
    verObj, err := version.NewVersionFromPkg(p)
    if err != nil {
        return nil, fmt.Errorf("matcher failed to parse version pkg=%q ver=%q: %w", p.Name, p.Version, err)
    }

    // 根据包的语言和包对象获取所有相关漏洞
    allPkgVulns, err := store.GetByLanguage(p.Language, p)
    if err != nil {
        return nil, fmt.Errorf("matcher failed to fetch language=%q pkg=%q: %w", p.Language, p.Name, err)
    }

    // 仅保留与当前包相关的漏洞
    applicableVulns, err := onlyQualifiedPackages(d, p, allPkgVulns)
    if err != nil {
        return nil, fmt.Errorf("unable to filter language-related vulnerabilities: %w", err)
    }

    // TODO: 将此部分迁移到一个限定器中并移除
    // 仅保留与当前版本相关的漏洞
    applicableVulns, err = onlyVulnerableVersions(verObj, applicableVulns)
    if err != nil {
        return nil, fmt.Errorf("unable to filter language-related vulnerabilities: %w", err)
    }

    var matches []match.Match
    # 遍历适用的漏洞列表
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
                    Type:       match.ExactDirectMatch,
                    # 置信度为1.0，暂时硬编码
                    Confidence: 1.0, // TODO: this is hard coded for now
                    # 匹配器为upstreamMatcher
                    Matcher:    upstreamMatcher,
                    # 搜索条件
                    SearchedBy: map[string]interface{}{
                        "language":  string(p.Language),
                        "namespace": vuln.Namespace,
                        "package": map[string]string{
                            "name":    p.Name,
                            "version": p.Version,
                        },
                    },
                    # 发现的漏洞信息
                    Found: map[string]interface{}{
                        "vulnerabilityID":   vuln.ID,
                        "versionConstraint": vuln.Constraint.String(),
                    },
                },
            },
        })
    }

    # 返回匹配结果和错误信息
    return matches, err
# 闭合前面的函数定义
```