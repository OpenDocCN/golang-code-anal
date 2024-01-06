# `grype\grype\matcher\matcher.go`

```
// 导入所需的包
package matcher

import (
	"github.com/anchore/grype/grype/distro"  // 导入 distro 包
	"github.com/anchore/grype/grype/match"   // 导入 match 包
	"github.com/anchore/grype/grype/pkg"     // 导入 pkg 包
	"github.com/anchore/grype/grype/vulnerability"  // 导入 vulnerability 包
	syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包并重命名为 syftPkg
)

// 定义 Matcher 接口
type Matcher interface {
	// 返回支持的包类型列表
	PackageTypes() []syftPkg.Type
	// 返回匹配器类型
	Type() match.MatcherType
	// 匹配漏洞
	Match(vulnerability.Provider, *distro.Distro, pkg.Package) ([]match.Match, error)
}
```