# `grype\grype\matcher\msrc\matcher.go`

```
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
	// 定义一个名为 Matcher 的结构体，用于进行匹配操作

	// 返回一个 syftPkg.Type 类型的切片，用于表示匹配的包类型
	// 这里看起来像是有一个特殊的包类型，但实际上这只是一个变通方法。MSRC 匹配是在 KB 补丁级别进行的，因此这将 KB 视为“包”，但它们并不是真正的包，它们是补丁
	return []syftPkg.Type{syftPkg.KbPkg}
}
// 返回匹配器的类型
func (m *Matcher) Type() match.MatcherType {
    return match.MsrcMatcher
}

// 根据给定的漏洞提供者、发行版和软件包，查找与MSFT版本匹配的KB
// “distro”保存有关Windows版本及其补丁（KB）的信息
func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    // 通过标准搜索条件查找与MSFT版本匹配的KB
    return search.ByCriteria(store, d, p, m.Type(), search.ByDistro)
}
```