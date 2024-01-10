# `grype\grype\deprecated.go`

```
package grype

import (
    "github.com/anchore/grype/grype/match"  // 导入 match 包
    "github.com/anchore/grype/grype/matcher"  // 导入 matcher 包
    "github.com/anchore/grype/grype/pkg"  // 导入 pkg 包
    "github.com/anchore/grype/grype/store"  // 导入 store 包
    "github.com/anchore/grype/internal/log"  // 导入 log 包
    "github.com/anchore/stereoscope/pkg/image"  // 导入 image 包
    "github.com/anchore/syft/syft/linux"  // 导入 linux 包
    "github.com/anchore/syft/syft/pkg/cataloger"  // 导入 cataloger 包
    "github.com/anchore/syft/syft/source"  // 导入 source 包
)

// TODO: deprecated, will remove before v1.0.0
func FindVulnerabilities(store store.Store, userImageStr string, scopeOpt source.Scope, registryOptions *image.RegistryOptions) (match.Matches, pkg.Context, []pkg.Package, error) {
    providerConfig := pkg.ProviderConfig{  // 创建 pkg.ProviderConfig 结构体
        SyftProviderConfig: pkg.SyftProviderConfig{  // 创建 pkg.SyftProviderConfig 结构体
            RegistryOptions:   registryOptions,  // 设置 RegistryOptions 字段值
            CatalogingOptions: cataloger.DefaultConfig(),  // 设置 CatalogingOptions 字段值为默认配置
        },
    }
    providerConfig.CatalogingOptions.Search.Scope = scopeOpt  // 设置 CatalogingOptions.Search.Scope 字段值为 scopeOpt

    packages, context, _, err := pkg.Provide(userImageStr, providerConfig)  // 调用 pkg.Provide 方法获取 packages, context, _, err
    if err != nil {  // 如果 err 不为空
        return match.Matches{}, pkg.Context{}, nil, err  // 返回空的 match.Matches 结构体，空的 pkg.Context 结构体，nil，和 err
    }

    matchers := matcher.NewDefaultMatchers(matcher.Config{})  // 创建默认配置的 matchers

    return FindVulnerabilitiesForPackage(store, context.Distro, matchers, packages), context, packages, nil  // 调用 FindVulnerabilitiesForPackage 方法并返回结果
}

// TODO: deprecated, will remove before v1.0.0
func FindVulnerabilitiesForPackage(store store.Store, d *linux.Release, matchers []matcher.Matcher, packages []pkg.Package) match.Matches {
    runner := VulnerabilityMatcher{  // 创建 VulnerabilityMatcher 结构体
        Store:          store,  // 设置 Store 字段值
        Matchers:       matchers,  // 设置 Matchers 字段值
        NormalizeByCVE: false,  // 设置 NormalizeByCVE 字段值为 false
    }

    actualResults, _, err := runner.FindMatches(packages, pkg.Context{  // 调用 runner.FindMatches 方法获取 actualResults, _, err
        Distro: d,  // 设置 pkg.Context 结构体的 Distro 字段值为 d
    })
    if err != nil || actualResults == nil {  // 如果 err 不为空或者 actualResults 为空
        log.WithFields("error", err).Error("unable to find vulnerabilities")  // 记录错误日志
        return match.NewMatches()  // 返回新的 match.Matches 结构体
    }
    return *actualResults  // 返回 actualResults 的值
}
```