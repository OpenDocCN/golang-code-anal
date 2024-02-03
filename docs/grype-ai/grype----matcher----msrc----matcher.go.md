# `grype\grype\matcher\msrc\matcher.go`

```go
package msrc

import (
    "github.com/anchore/grype/grype/distro"
    "github.com/anchore/grype/grype/match"
    "github.com/anchore/grype/grype/pkg"
    "github.com/anchore/grype/grype/search"
    "github.com/anchore/grype/grype/vulnerability"
    syftPkg "github.com/anchore/syft/syft/pkg"
)

type Matcher struct {
}

func (m *Matcher) PackageTypes() []syftPkg.Type {
    // 返回一个特定类型的包列表，这里是为了解决一个问题，MSRC 匹配是在 KB 补丁级别进行的，
    // 所以这里将 KB 视为“包”，但它们实际上不是包，而是补丁
    return []syftPkg.Type{syftPkg.KbPkg}
}

func (m *Matcher) Type() match.MatcherType {
    // 返回匹配器的类型，这里是 MSRC 匹配器
    return match.MsrcMatcher
}

func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    // 查找与包中给定的 MSFT 版本和版本号匹配的 KB 补丁
    // “distro” 包含有关 Windows 版本及其补丁（KB）的信息
    return search.ByCriteria(store, d, p, m.Type(), search.ByDistro)
}
```