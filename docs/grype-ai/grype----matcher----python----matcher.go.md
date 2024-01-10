# `grype\grype\matcher\python\matcher.go`

```
package python

import (
    "github.com/anchore/grype/grype/distro"  // 导入 distro 包
    "github.com/anchore/grype/grype/match"   // 导入 match 包
    "github.com/anchore/grype/grype/pkg"      // 导入 pkg 包
    "github.com/anchore/grype/grype/search"   // 导入 search 包
    "github.com/anchore/grype/grype/vulnerability"  // 导入 vulnerability 包
    syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包并重命名为 syftPkg
)

type Matcher struct {
    cfg MatcherConfig  // 定义 Matcher 结构体，包含 cfg 字段
}

type MatcherConfig struct {
    UseCPEs bool  // 定义 MatcherConfig 结构体，包含 UseCPEs 字段
}

func NewPythonMatcher(cfg MatcherConfig) *Matcher {
    return &Matcher{
        cfg: cfg,  // 返回一个新的 PythonMatcher 对象，包含传入的配置
    }
}

func (m *Matcher) PackageTypes() []syftPkg.Type {
    return []syftPkg.Type{syftPkg.PythonPkg}  // 返回 Python 包的类型
}

func (m *Matcher) Type() match.MatcherType {
    return match.PythonMatcher  // 返回 Python 匹配器的类型
}

func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    criteria := search.CommonCriteria  // 设置搜索的通用标准
    if m.cfg.UseCPEs {
        criteria = append(criteria, search.ByCPE)  // 如果使用 CPEs，则添加 CPE 标准
    }
    return search.ByCriteria(store, d, p, m.Type(), criteria...)  // 根据标准进行搜索匹配
}
```