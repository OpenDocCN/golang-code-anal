# `grype\grype\matcher\javascript\matcher.go`

```go
package javascript

import (
    "github.com/anchore/grype/grype/distro"  // 导入 distro 包
    "github.com/anchore/grype/grype/match"   // 导入 match 包
    "github.com/anchore/grype/grype/pkg"     // 导入 pkg 包
    "github.com/anchore/grype/grype/search"  // 导入 search 包
    "github.com/anchore/grype/grype/vulnerability"  // 导入 vulnerability 包
    syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包
)

type Matcher struct {
    cfg MatcherConfig  // 定义 Matcher 结构体，包含 cfg 字段
}

type MatcherConfig struct {
    UseCPEs bool  // 定义 MatcherConfig 结构体，包含 UseCPEs 字段
}

func NewJavascriptMatcher(cfg MatcherConfig) *Matcher {
    return &Matcher{
        cfg: cfg,  // 返回一个新的 Matcher 对象，包含传入的配置
    }
}

func (m *Matcher) PackageTypes() []syftPkg.Type {
    return []syftPkg.Type{syftPkg.NpmPkg}  // 返回一个包含 NpmPkg 类型的 syftPkg.Type 切片
}

func (m *Matcher) Type() match.MatcherType {
    return match.JavascriptMatcher  // 返回 JavascriptMatcher 类型的 match.MatcherType
}

func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    criteria := search.CommonCriteria  // 设置 criteria 变量为 CommonCriteria
    if m.cfg.UseCPEs {
        criteria = append(criteria, search.ByCPE)  // 如果配置中使用 CPEs，则将 ByCPE 添加到 criteria 中
    }
    return search.ByCriteria(store, d, p, m.Type(), criteria...)  // 返回根据条件搜索的结果
}
```