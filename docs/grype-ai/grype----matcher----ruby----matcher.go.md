# `grype\grype\matcher\ruby\matcher.go`

```
package ruby
# 导入所需的包

import (
	"github.com/anchore/grype/grype/distro"
	"github.com/anchore/grype/grype/match"
	"github.com/anchore/grype/grype/pkg"
	"github.com/anchore/grype/grype/search"
	"github.com/anchore/grype/grype/vulnerability"
	syftPkg "github.com/anchore/syft/syft/pkg"
)
# 导入所需的包

type Matcher struct {
	cfg MatcherConfig
}
# 定义 Matcher 结构体

type MatcherConfig struct {
	UseCPEs bool
}
# 定义 MatcherConfig 结构体

func NewRubyMatcher(cfg MatcherConfig) *Matcher {
# 定义 NewRubyMatcher 函数，接受 MatcherConfig 参数，返回 Matcher 指针
# 返回一个Matcher对象，其中包含了配置信息
return &Matcher{
    cfg: cfg,
}

# 返回支持的软件包类型列表
func (m *Matcher) PackageTypes() []syftPkg.Type {
    return []syftPkg.Type{syftPkg.GemPkg}
}

# 返回Matcher的类型
func (m *Matcher) Type() match.MatcherType {
    return match.RubyGemMatcher
}

# 匹配漏洞，返回匹配结果和可能的错误
func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    criteria := search.CommonCriteria
    # 如果配置使用CPEs，则添加ByCPE到匹配标准中
    if m.cfg.UseCPEs {
        criteria = append(criteria, search.ByCPE)
    }
    # 根据匹配标准进行匹配，返回匹配结果和可能的错误
    return search.ByCriteria(store, d, p, m.Type(), criteria...)
}
抱歉，我无法为您提供代码注释，因为您没有提供需要注释的代码。如果您有任何代码需要解释，请提供给我，我将竭诚为您服务。
```