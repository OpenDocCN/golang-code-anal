# `grype\grype\matcher\golang\matcher.go`

```
// 声明一个名为 golang 的包
package golang

// 导入所需的包
import (
	"strings" // 导入 strings 包
	"github.com/anchore/grype/grype/distro" // 导入 distro 包
	"github.com/anchore/grype/grype/match" // 导入 match 包
	"github.com/anchore/grype/grype/pkg" // 导入 pkg 包
	"github.com/anchore/grype/grype/search" // 导入 search 包
	"github.com/anchore/grype/grype/vulnerability" // 导入 vulnerability 包
	syftPkg "github.com/anchore/syft/syft/pkg" // 导入 syftPkg 包，并重命名为 syftPkg
)

// 定义 Matcher 结构体
type Matcher struct {
	cfg MatcherConfig // 包含 MatcherConfig 类型的字段 cfg
}

// 定义 MatcherConfig 结构体
type MatcherConfig struct {
	UseCPEs               bool // 布尔类型字段 UseCPEs
	AlwaysUseCPEForStdlib bool // 布尔类型字段 AlwaysUseCPEForStdlib
}
}

// 创建一个新的 Golang 匹配器，使用给定的配置
func NewGolangMatcher(cfg MatcherConfig) *Matcher {
	// 返回一个指向 Matcher 结构的指针，其中包含给定的配置
	return &Matcher{
		cfg: cfg,
	}
}

// 返回匹配器支持的包类型列表
func (m *Matcher) PackageTypes() []syftPkg.Type {
	return []syftPkg.Type{syftPkg.GoModulePkg}
}

// 返回匹配器的类型
func (m *Matcher) Type() match.MatcherType {
	return match.GoModuleMatcher
}

// 匹配给定的软件包，返回匹配结果
func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
	// 创建一个空的匹配结果列表
	matches := make([]match.Match, 0)

	// 初始化主模块名称为空字符串
	mainModule := ""
// 如果 p.Metadata 是 pkg.GolangBinMetadata 类型，则将其转换为 m，并将 m.MainModule 赋值给 mainModule
if m, ok := p.Metadata.(pkg.GolangBinMetadata); ok {
    mainModule = m.MainModule
}

// Golang 目前没有一种标准的方式将 vcs 版本信息编入编译后的二进制文件中：https://github.com/golang/go/issues/50603
// 主模块的当前版本信息不完整，导致多个 FP（假阳性）
// TODO: 在将来的 Go 版本中包含 vcs 信息时，删除这个排除
isNotCorrected := strings.HasPrefix(p.Version, "v0.0.0-") || strings.HasPrefix(p.Version, "(devel)")
if p.Name == mainModule && isNotCorrected {
    return matches, nil
}

// 设置搜索标准为通用标准
criteria := search.CommonCriteria
// 如果按照 CPE 搜索 p.Name，则将 search.ByCPE 添加到 criteria 中
if searchByCPE(p.Name, m.cfg) {
    criteria = append(criteria, search.ByCPE)
}

// 根据给定的标准 criteria 进行搜索
return search.ByCriteria(store, d, p, m.Type(), criteria...)
# 根据给定的名称和匹配器配置来搜索CPE（通用平台漏洞披露）信息
func searchByCPE(name string, cfg MatcherConfig) bool:
    # 如果配置中使用CPE，则返回true
    if cfg.UseCPEs:
        return true
    # 如果配置中始终使用CPE来搜索标准库，并且给定的名称是"stdlib"，则返回true
    return cfg.AlwaysUseCPEForStdlib && (name == "stdlib")
```