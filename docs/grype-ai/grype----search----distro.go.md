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
    // 如果发行版为空，则返回空结果
    if d == nil {
        return nil, nil
    }

    // 从包中创建版本对象
    verObj, err := version.NewVersionFromPkg(p)
    if err != nil {
        return nil, fmt.Errorf("matcher failed to parse version pkg=%q ver=%q: %w", p.Name, p.Version, err)
    }

    // 获取指定发行版和包的所有漏洞
    allPkgVulns, err := store.GetByDistro(d, p)
    if err != nil {
        return nil, fmt.Errorf("matcher failed to fetch distro=%q pkg=%q: %w", d, p.Name, err)
    }

    // 过滤出适用于指定包和发行版的漏洞
    applicableVulns, err := onlyQualifiedPackages(d, p, allPkgVulns)
    if err != nil {
        return nil, fmt.Errorf("unable to filter distro-related vulnerabilities: %w", err)
    }

    // TODO: 将此部分迁移到限定符，并删除
    // 过滤出适用于指定版本的漏洞
    applicableVulns, err = onlyVulnerableVersions(verObj, applicableVulns)
    if err != nil {
        return nil, fmt.Errorf("unable to filter distro-related vulnerabilities: %w", err)
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
                    Type:    match.ExactDirectMatch,
                    # 匹配器为上游匹配器
                    Matcher: upstreamMatcher,
                    # 搜索条件
                    SearchedBy: map[string]interface{}{
                        "distro": map[string]string{
                            "type":    d.Type.String(),
                            "version": d.RawVersion,
                        },
                        # 为什么包含包信息？给定的包可能是另一个已安装在系统上的包的源包。这样可以清楚地表明搜索时使用了什么。
                        "package": map[string]string{
                            "name":    p.Name,
                            "version": p.Version,
                        },
                        "namespace": vuln.Namespace,
                    },
                    # 发现的漏洞信息
                    Found: map[string]interface{}{
                        "vulnerabilityID":   vuln.ID,
                        "versionConstraint": vuln.Constraint.String(),
                    },
                    # 置信度为1.0，暂时硬编码
                    Confidence: 1.0, // TODO: this is hard coded for now
                },
            },
        })
    }

    # 返回匹配结果和可能的错误
    return matches, err
# 闭合前面的函数定义
```