# `grype\grype\matcher\portage\matcher.go`

```
package portage

import (
    "fmt"

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
    // 返回 Portage 包类型
    return []syftPkg.Type{syftPkg.PortagePkg}
}

func (m *Matcher) Type() match.MatcherType {
    // 返回 Portage 匹配器类型
    return match.PortageMatcher
}

func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    // 通过包、发行版和匹配器类型搜索漏洞匹配
    matches, err := search.ByPackageDistro(store, d, p, m.Type())
    if err != nil {
        // 如果搜索失败，则返回错误
        return nil, fmt.Errorf("failed to find vulnerabilities: %w", err)
    }

    // 返回匹配结果
    return matches, nil
}
```