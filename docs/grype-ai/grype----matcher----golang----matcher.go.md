# `grype\grype\matcher\golang\matcher.go`

```
package golang

import (
    "strings" // 导入 strings 包

    "github.com/anchore/grype/grype/distro" // 导入 distro 包
    "github.com/anchore/grype/grype/match" // 导入 match 包
    "github.com/anchore/grype/grype/pkg" // 导入 pkg 包
    "github.com/anchore/grype/grype/search" // 导入 search 包
    "github.com/anchore/grype/grype/vulnerability" // 导入 vulnerability 包
    syftPkg "github.com/anchore/syft/syft/pkg" // 导入 syftPkg 包
)

type Matcher struct {
    cfg MatcherConfig // 定义 Matcher 结构体
}

type MatcherConfig struct {
    UseCPEs               bool // 定义 UseCPEs 字段
    AlwaysUseCPEForStdlib bool // 定义 AlwaysUseCPEForStdlib 字段
}

func NewGolangMatcher(cfg MatcherConfig) *Matcher {
    return &Matcher{
        cfg: cfg, // 返回 Matcher 结构体指针
    }
}

func (m *Matcher) PackageTypes() []syftPkg.Type {
    return []syftPkg.Type{syftPkg.GoModulePkg} // 返回 GoModulePkg 类型的切片
}

func (m *Matcher) Type() match.MatcherType {
    return match.GoModuleMatcher // 返回 GoModuleMatcher 类型
}

func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    matches := make([]match.Match, 0) // 创建空的 match.Match 类型切片

    mainModule := "" // 初始化 mainModule 变量为空字符串
    if m, ok := p.Metadata.(pkg.GolangBinMetadata); ok { // 判断 p.Metadata 是否为 pkg.GolangBinMetadata 类型
        mainModule = m.MainModule // 将 m.MainModule 赋值给 mainModule
    }

    // Golang 目前没有一种标准的方式将 vcs 版本合并到编译后的二进制文件中：https://github.com/golang/go/issues/50603
    // 当前主模块的版本信息不完整，导致多个 FP
    // TODO: 在将来的 Go 版本中包含 vcs 信息时，删除此排除
    isNotCorrected := strings.HasPrefix(p.Version, "v0.0.0-") || strings.HasPrefix(p.Version, "(devel)") // 判断 p.Version 是否以指定前缀开头
    if p.Name == mainModule && isNotCorrected { // 判断 p.Name 是否等于 mainModule 并且 isNotCorrected 为真
        return matches, nil // 返回空的 match.Match 类型切片和 nil
    }

    criteria := search.CommonCriteria // 初始化 criteria 为 CommonCriteria
    if searchByCPE(p.Name, m.cfg) { // 调用 searchByCPE 函数判断是否使用 CPE
        criteria = append(criteria, search.ByCPE) // 将 ByCPE 添加到 criteria 中
    }

    return search.ByCriteria(store, d, p, m.Type(), criteria...) // 调用 ByCriteria 函数进行搜索
}

func searchByCPE(name string, cfg MatcherConfig) bool {
    if cfg.UseCPEs { // 判断是否使用 CPE
        return true // 返回 true
    }

    return cfg.AlwaysUseCPEForStdlib && (name == "stdlib") // 返回 AlwaysUseCPEForStdlib 是否为真并且 name 是否为 "stdlib"
}
```