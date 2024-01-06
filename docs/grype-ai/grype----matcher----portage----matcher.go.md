# `grype\grype\matcher\portage\matcher.go`

```
package portage

import (
	"fmt"  // 导入 fmt 包，用于格式化输出

	"github.com/anchore/grype/grype/distro"  // 导入 distro 包
	"github.com/anchore/grype/grype/match"  // 导入 match 包
	"github.com/anchore/grype/grype/pkg"  // 导入 pkg 包
	"github.com/anchore/grype/grype/search"  // 导入 search 包
	"github.com/anchore/grype/grype/vulnerability"  // 导入 vulnerability 包
	syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包，并重命名为 syftPkg
)

type Matcher struct {  // 定义 Matcher 结构体
}

func (m *Matcher) PackageTypes() []syftPkg.Type {  // 定义 Matcher 结构体的 PackageTypes 方法
	return []syftPkg.Type{syftPkg.PortagePkg}  // 返回 PortagePkg 类型的包
}
# 返回匹配器的类型
func (m *Matcher) Type() match.MatcherType {
    return match.PortageMatcher
}

# 根据包的发行版和包信息，在提供的漏洞提供者中搜索匹配项
func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    # 使用包的发行版和包信息在漏洞提供者中搜索匹配项
    matches, err := search.ByPackageDistro(store, d, p, m.Type())
    # 如果发生错误，返回错误信息
    if err != nil {
        return nil, fmt.Errorf("failed to find vulnerabilities: %w", err)
    }

    # 返回匹配项
    return matches, nil
}
```