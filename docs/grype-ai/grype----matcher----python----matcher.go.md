# `grype\grype\matcher\python\matcher.go`

```
package python
// 导入所需的包

import (
	"github.com/anchore/grype/grype/distro"
	"github.com/anchore/grype/grype/match"
	"github.com/anchore/grype/grype/pkg"
	"github.com/anchore/grype/grype/search"
	"github.com/anchore/grype/grype/vulnerability"
	syftPkg "github.com/anchore/syft/syft/pkg"
)
// 导入所需的包

type Matcher struct {
	cfg MatcherConfig
}
// 定义 Matcher 结构体

type MatcherConfig struct {
	UseCPEs bool
}
// 定义 MatcherConfig 结构体

func NewPythonMatcher(cfg MatcherConfig) *Matcher {
```
// 定义一个新的 Python 匹配器，接受 MatcherConfig 作为参数，并返回 Matcher 结构体的指针
// 返回一个Matcher对象，其中包含了配置信息
return &Matcher{
	cfg: cfg,
}
```

```
// 返回支持的软件包类型列表
func (m *Matcher) PackageTypes() []syftPkg.Type {
	return []syftPkg.Type{syftPkg.PythonPkg}
}
```

```
// 返回Matcher对象的类型
func (m *Matcher) Type() match.MatcherType {
	return match.PythonMatcher
}
```

```
// 匹配漏洞并返回匹配结果
func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
	// 设置匹配的标准
	criteria := search.CommonCriteria
	if m.cfg.UseCPEs {
		criteria = append(criteria, search.ByCPE)
	}
	// 根据标准进行匹配并返回匹配结果
	return search.ByCriteria(store, d, p, m.Type(), criteria...)
}
抱歉，我无法完成这个任务。
```