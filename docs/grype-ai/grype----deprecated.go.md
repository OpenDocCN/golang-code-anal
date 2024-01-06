# `grype\grype\deprecated.go`

```
// 导入所需的包
package grype

import (
	"github.com/anchore/grype/grype/match"  // 导入匹配模块
	"github.com/anchore/grype/grype/matcher"  // 导入匹配器模块
	"github.com/anchore/grype/grype/pkg"  // 导入包模块
	"github.com/anchore/grype/grype/store"  // 导入存储模块
	"github.com/anchore/grype/internal/log"  // 导入日志模块
	"github.com/anchore/stereoscope/pkg/image"  // 导入图像模块
	"github.com/anchore/syft/syft/linux"  // 导入 Linux 模块
	"github.com/anchore/syft/syft/pkg/cataloger"  // 导入目录模块
	"github.com/anchore/syft/syft/source"  // 导入源模块
)

// TODO: deprecated, will remove before v1.0.0
// 查找漏洞函数，接收存储、用户图像字符串、范围选项和注册表选项作为参数，返回匹配、上下文、包列表和错误
func FindVulnerabilities(store store.Store, userImageStr string, scopeOpt source.Scope, registryOptions *image.RegistryOptions) (match.Matches, pkg.Context, []pkg.Package, error) {
	// 设置包提供者配置
	providerConfig := pkg.ProviderConfig{
		// 设置 Syft 提供者配置
		SyftProviderConfig: pkg.SyftProviderConfig{
			RegistryOptions:   registryOptions,  // 设置注册表选项
			CatalogingOptions: cataloger.DefaultConfig(),  // 设置目录选项为默认配置
		},
	}
	providerConfig.CatalogingOptions.Search.Scope = scopeOpt
    // 设置 providerConfig 的搜索范围为 scopeOpt

	packages, context, _, err := pkg.Provide(userImageStr, providerConfig)
    // 调用 pkg.Provide 方法获取 packages、context、错误信息
	if err != nil {
        // 如果出现错误，返回空的 match.Matches、pkg.Context、nil 和错误信息
		return match.Matches{}, pkg.Context{}, nil, err
	}

	matchers := matcher.NewDefaultMatchers(matcher.Config{})
    // 创建默认的匹配器

	return FindVulnerabilitiesForPackage(store, context.Distro, matchers, packages), context, packages, nil
    // 返回调用 FindVulnerabilitiesForPackage 方法的结果，以及 context、packages 和空值

}

// TODO: deprecated, will remove before v1.0.0
// TODO: 已弃用，在 v1.0.0 之前将删除
func FindVulnerabilitiesForPackage(store store.Store, d *linux.Release, matchers []matcher.Matcher, packages []pkg.Package) match.Matches {
    // 定义 FindVulnerabilitiesForPackage 方法，接受 store、d、matchers 和 packages 作为参数
	runner := VulnerabilityMatcher{
        // 创建 VulnerabilityMatcher 对象
		Store:          store,
        // 设置 Store 属性为传入的 store
		Matchers:       matchers,
        // 设置 Matchers 属性为传入的 matchers
		NormalizeByCVE: false,
        // 设置 NormalizeByCVE 属性为 false
	}

	// 调用runner的FindMatches方法，传入packages和pkg.Context对象，返回匹配结果、状态和错误
	actualResults, _, err := runner.FindMatches(packages, pkg.Context{
		Distro: d,
	})
	// 如果发生错误或者actualResults为空，则记录错误信息并返回空的匹配结果
	if err != nil || actualResults == nil {
		log.WithFields("error", err).Error("unable to find vulnerabilities")
		return match.NewMatches()
	}
	// 返回actualResults的值
	return *actualResults
}
```