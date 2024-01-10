# `grype\grype\matcher\matcher.go`

```
package matcher

import (
    "github.com/anchore/grype/grype/distro"  // 导入 distro 包
    "github.com/anchore/grype/grype/match"   // 导入 match 包
    "github.com/anchore/grype/grype/pkg"     // 导入 pkg 包
    "github.com/anchore/grype/grype/vulnerability"  // 导入 vulnerability 包
    syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包，并重命名为 syftPkg
)

type Matcher interface {
    PackageTypes() []syftPkg.Type  // 定义接口方法 PackageTypes，返回 syftPkg.Type 切片
    Type() match.MatcherType  // 定义接口方法 Type，返回 match.MatcherType
    Match(vulnerability.Provider, *distro.Distro, pkg.Package) ([]match.Match, error)  // 定义接口方法 Match，接收 vulnerability.Provider、*distro.Distro、pkg.Package 作为参数，返回 match.Match 切片和 error
}
```