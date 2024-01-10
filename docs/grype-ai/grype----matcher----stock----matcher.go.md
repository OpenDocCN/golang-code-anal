# `grype\grype\matcher\stock\matcher.go`

```
package stock

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

func NewStockMatcher(cfg MatcherConfig) *Matcher {
    return &Matcher{
        cfg: cfg,  // 返回一个新的 Matcher 对象，初始化 cfg 字段
    }
}

func (m *Matcher) PackageTypes() []syftPkg.Type {
    return nil  // 返回空的 syftPkg.Type 切片
}

func (m *Matcher) Type() match.MatcherType {
    return match.StockMatcher  // 返回 match.StockMatcher
}

func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    criteria := search.CommonCriteria  // 初始化 criteria 变量为 search.CommonCriteria
    if m.cfg.UseCPEs {  // 如果配置中使用 CPEs
        criteria = append(criteria, search.ByCPE)  // 将 search.ByCPE 添加到 criteria 中
    }
    return search.ByCriteria(store, d, p, m.Type(), criteria...)  // 返回根据条件搜索的结果
}
```