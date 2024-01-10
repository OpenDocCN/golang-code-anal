# `grype\grype\matcher\java\matcher.go`

```
package java
# 导入所需的包

import (
    "fmt"
    "net/http"

    "github.com/anchore/grype/grype/distro"
    "github.com/anchore/grype/grype/match"
    "github.com/anchore/grype/grype/pkg"
    "github.com/anchore/grype/grype/search"
    "github.com/anchore/grype/grype/vulnerability"
    "github.com/anchore/grype/internal/log"
    syftPkg "github.com/anchore/syft/syft/pkg"
)
# 导入所需的包

const (
    sha1Query = `1:"%s"`
)
# 定义常量 sha1Query

type Matcher struct {
    MavenSearcher
    cfg MatcherConfig
}
# 定义 Matcher 结构体

type ExternalSearchConfig struct {
    SearchMavenUpstream bool
    MavenBaseURL        string
}
# 定义 ExternalSearchConfig 结构体

type MatcherConfig struct {
    ExternalSearchConfig
    UseCPEs bool
}
# 定义 MatcherConfig 结构体

func NewJavaMatcher(cfg MatcherConfig) *Matcher {
    return &Matcher{
        cfg: cfg,
        MavenSearcher: &mavenSearch{
            client:  http.DefaultClient,
            baseURL: cfg.MavenBaseURL,
        },
    }
}
# 定义 NewJavaMatcher 函数，返回 Matcher 结构体指针

func (m *Matcher) PackageTypes() []syftPkg.Type {
    return []syftPkg.Type{syftPkg.JavaPkg, syftPkg.JenkinsPluginPkg}
}
# 定义 PackageTypes 方法，返回 syftPkg.Type 切片

func (m *Matcher) Type() match.MatcherType {
    return match.JavaMatcher
}
# 定义 Type 方法，返回 match.MatcherType

func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    var matches []match.Match
    if m.cfg.SearchMavenUpstream {
        upstreamMatches, err := m.matchUpstreamMavenPackages(store, d, p)
        if err != nil {
            log.Debugf("failed to match against upstream data for %s: %v", p.Name, err)
        } else {
            matches = append(matches, upstreamMatches...)
        }
    }
    criteria := search.CommonCriteria
    if m.cfg.UseCPEs {
        criteria = append(criteria, search.ByCPE)
    }
    criteriaMatches, err := search.ByCriteria(store, d, p, m.Type(), criteria...)
    if err != nil {
        return nil, fmt.Errorf("failed to match by exact package: %w", err)
    }

    matches = append(matches, criteriaMatches...)
    return matches, nil
}
# 定义 Match 方法，用于匹配漏洞
func (m *Matcher) matchUpstreamMavenPackages(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    var matches []match.Match  // 声明一个空的匹配结果数组

    if metadata, ok := p.Metadata.(pkg.JavaMetadata); ok {  // 检查包的元数据是否为 JavaMetadata 类型
        for _, digest := range metadata.ArchiveDigests {  // 遍历元数据中的存档摘要
            if digest.Algorithm == "sha1" {  // 检查摘要算法是否为 SHA1
                indirectPackage, err := m.GetMavenPackageBySha(digest.Value)  // 通过 SHA1 值获取 Maven 包
                if err != nil {
                    return nil, err  // 如果出现错误，返回空结果和错误信息
                }
                indirectMatches, err := search.ByPackageLanguage(store, d, *indirectPackage, m.Type())  // 通过包语言搜索间接匹配
                if err != nil {
                    return nil, err  // 如果出现错误，返回空结果和错误信息
                }
                matches = append(matches, indirectMatches...)  // 将间接匹配结果追加到总匹配结果数组中
            }
        }
    }

    match.ConvertToIndirectMatches(matches, p)  // 将匹配结果转换为间接匹配结果

    return matches, nil  // 返回最终的匹配结果数组和空错误信息
}
```