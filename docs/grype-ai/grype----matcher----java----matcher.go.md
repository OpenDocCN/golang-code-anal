# `grype\grype\matcher\java\matcher.go`

```
package java
// 导入所需的包

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
// 导入所需的包

const (
	sha1Query = `1:"%s"`
)
// 定义常量 sha1Query

type Matcher struct {
```
// 定义结构体 Matcher
// 定义 MavenSearcher 结构体
MavenSearcher
// 定义 MatcherConfig 结构体
cfg MatcherConfig
}

// 定义 ExternalSearchConfig 结构体
type ExternalSearchConfig struct {
	// 是否搜索 Maven 上游
	SearchMavenUpstream bool
	// Maven 基础 URL
	MavenBaseURL        string
}

// 定义 MatcherConfig 结构体，包含 ExternalSearchConfig 和 UseCPEs 字段
type MatcherConfig struct {
	ExternalSearchConfig
	// 是否使用 CPEs
	UseCPEs bool
}

// 创建新的 JavaMatcher 对象，传入 MatcherConfig 配置
func NewJavaMatcher(cfg MatcherConfig) *Matcher {
	return &Matcher{
		// 设置 MatcherConfig 配置
		cfg: cfg,
		// 创建 mavenSearch 对象，设置 http 客户端和 Maven 基础 URL
		MavenSearcher: &mavenSearch{
			client:  http.DefaultClient,
			baseURL: cfg.MavenBaseURL,
		},
	}
}

func (m *Matcher) PackageTypes() []syftPkg.Type {
	// 返回匹配器支持的包类型列表
	return []syftPkg.Type{syftPkg.JavaPkg, syftPkg.JenkinsPluginPkg}
}

func (m *Matcher) Type() match.MatcherType {
	// 返回匹配器的类型
	return match.JavaMatcher
}

func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
	// 初始化匹配结果列表
	var matches []match.Match
	// 如果配置允许搜索 Maven 上游包，则进行匹配
	if m.cfg.SearchMavenUpstream {
		// 匹配上游 Maven 包
		upstreamMatches, err := m.matchUpstreamMavenPackages(store, d, p)
		// 如果匹配出错，则记录日志
		if err != nil {
			log.Debugf("failed to match against upstream data for %s: %v", p.Name, err)
		} else {
			// 将匹配结果添加到匹配列表中
			matches = append(matches, upstreamMatches...)
		}
	}
	// 获取通用的搜索条件
	criteria := search.CommonCriteria
	// 如果配置中使用了CPEs，则添加CPE作为搜索条件
	if m.cfg.UseCPEs {
		criteria = append(criteria, search.ByCPE)
	}
	// 根据给定的条件进行匹配
	criteriaMatches, err := search.ByCriteria(store, d, p, m.Type(), criteria...)
	if err != nil {
		// 如果匹配失败，返回错误信息
		return nil, fmt.Errorf("failed to match by exact package: %w", err)
	}

	// 将匹配结果添加到matches中
	matches = append(matches, criteriaMatches...)
	// 返回匹配结果和无错误
	return matches, nil
}

// 匹配上游Maven包
func (m *Matcher) matchUpstreamMavenPackages(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
	var matches []match.Match

	// 如果包的元数据是JavaMetadata类型
	if metadata, ok := p.Metadata.(pkg.JavaMetadata); ok {
		// 遍历元数据中的存档摘要
		for _, digest := range metadata.ArchiveDigests {
# 如果摘要算法是 sha1
if digest.Algorithm == "sha1" {
    # 通过 sha1 值获取 Maven 包
    indirectPackage, err := m.GetMavenPackageBySha(digest.Value)
    # 如果获取失败，返回错误
    if err != nil {
        return nil, err
    }
    # 通过包和语言搜索间接匹配
    indirectMatches, err := search.ByPackageLanguage(store, d, *indirectPackage, m.Type())
    # 如果搜索失败，返回错误
    if err != nil {
        return nil, err
    }
    # 将间接匹配结果追加到匹配列表中
    matches = append(matches, indirectMatches...)
}
# 将匹配结果转换为间接匹配
match.ConvertToIndirectMatches(matches, p)
# 返回匹配结果和空错误
return matches, nil
```