# `grype\grype\matcher\dpkg\matcher.go`

```
package dpkg

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
    return []syftPkg.Type{syftPkg.DebPkg}
}

func (m *Matcher) Type() match.MatcherType {
    return match.DpkgMatcher
}

func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    matches := make([]match.Match, 0)

    // 通过源间接匹配包
    sourceMatches, err := m.matchUpstreamPackages(store, d, p)
    if err != nil {
        return nil, fmt.Errorf("failed to match by source indirection: %w", err)
    }
    matches = append(matches, sourceMatches...)

    // 通过精确包名匹配
    exactMatches, err := search.ByPackageDistro(store, d, p, m.Type())
    if err != nil {
        return nil, fmt.Errorf("failed to match by exact package name: %w", err)
    }
    matches = append(matches, exactMatches...)

    return matches, nil
}

func (m *Matcher) matchUpstreamPackages(store vulnerability.ProviderByDistro, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    var matches []match.Match

    // 遍历上游包并匹配漏洞
    for _, indirectPackage := range pkg.UpstreamPackages(p) {
        indirectMatches, err := search.ByPackageDistro(store, d, indirectPackage, m.Type())
        if err != nil {
            return nil, fmt.Errorf("failed to find vulnerabilities for dpkg upstream source package: %w", err)
        }
        matches = append(matches, indirectMatches...)
    }

    // 确保我们基于 SBOM 包跟踪匹配（而不是间接包）
    // 但是，我们也希望保留间接包以供将来参考
    match.ConvertToIndirectMatches(matches, p)

    return matches, nil
}
```