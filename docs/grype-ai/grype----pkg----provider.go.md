# `grype\grype\pkg\provider.go`

```
package pkg

import (
    "errors"  // 导入 errors 包，用于处理错误
    "fmt"     // 导入 fmt 包，用于格式化输出

    "github.com/bmatcuk/doublestar/v2"  // 导入 doublestar 包，用于处理 glob 匹配

    "github.com/anchore/syft/syft/file"  // 导入 file 包，用于处理文件
    "github.com/anchore/syft/syft/sbom"  // 导入 sbom 包，用于处理 SBOM
)

var errDoesNotProvide = fmt.Errorf("cannot provide packages from the given source")  // 定义一个错误变量

// Provide a set of packages and context metadata describing where they were sourced from.
func Provide(userInput string, config ProviderConfig) ([]Package, Context, *sbom.SBOM, error) {
    packages, ctx, s, err := syftSBOMProvider(userInput, config)  // 调用 syftSBOMProvider 函数获取包和上下文信息
    if !errors.Is(err, errDoesNotProvide) {  // 如果错误不是 errDoesNotProvide
        if len(config.Exclusions) > 0 {  // 如果排除列表不为空
            packages, err = filterPackageExclusions(packages, config.Exclusions)  // 调用 filterPackageExclusions 函数过滤包
            if err != nil {  // 如果出现错误
                return nil, ctx, s, err  // 返回空值和错误
            }
        }
        return packages, ctx, s, err  // 返回包、上下文和错误
    }

    packages, err = purlProvider(userInput)  // 调用 purlProvider 函数获取包
    if !errors.Is(err, errDoesNotProvide) {  // 如果错误不是 errDoesNotProvide
        return packages, Context{}, s, err  // 返回包、空上下文和错误
    }

    return syftProvider(userInput, config)  // 返回 syftProvider 函数的结果
}

// This will filter the provided packages list based on a set of exclusion expressions. Globs
// are allowed for the exclusions. A package will be *excluded* only if *all locations* match
// one of the provided exclusions.
func filterPackageExclusions(packages []Package, exclusions []string) ([]Package, error) {
    var out []Package  // 定义一个空的包列表
    # 遍历包列表
    for _, pkg := range packages:
        # 默认包含该包
        includePackage := true
        # 获取包的所有位置
        locations := pkg.Locations.ToSlice()
        # 如果有位置信息
        if len(locations) > 0:
            # 默认不包含该包
            includePackage = false
            # 对每个位置进行遍历
            // require ALL locations to be excluded for the package to be excluded
        location:
            for _, location := range locations:
                # 对每个位置和排除列表进行匹配
                for _, exclusion := range exclusions:
                    # 判断位置是否匹配排除列表
                    match, err := locationMatches(location, exclusion)
                    # 如果出现错误，返回错误
                    if err != nil:
                        return nil, err
                    # 如果匹配，跳过当前位置
                    if match {
                        continue location
                    }
                # 如果位置没有匹配任何排除项，包含该包
                includePackage = true
                break
        # 如果包需要包含
        if includePackage:
            # 将包添加到输出列表
            out = append(out, pkg)
    # 返回输出列表和空错误
    return out, nil
// 测试位置的 RealPath 和 VirtualPath 是否与排除参数匹配。
// 排除参数允许使用类似 `/usr/**` 或 `**/*.json` 的通配符表达式。如果排除参数是无效的模式，则返回错误；否则，返回的布尔值表示是否匹配。
func locationMatches(location file.Location, exclusion string) (bool, error) {
    // 使用 doublestar 包中的 Match 函数，检查 RealPath 是否与排除参数匹配
    matchesRealPath, err := doublestar.Match(exclusion, location.RealPath)
    if err != nil {
        return false, err
    }
    // 使用 doublestar 包中的 Match 函数，检查 VirtualPath 是否与排除参数匹配
    matchesVirtualPath, err := doublestar.Match(exclusion, location.AccessPath)
    if err != nil {
        return false, err
    }
    // 返回 RealPath 或 VirtualPath 是否与排除参数匹配的结果
    return matchesRealPath || matchesVirtualPath, nil
}
```