# `grype\grype\pkg\provider.go`

```
package pkg

import (
	"errors"  // 导入 errors 包，用于处理错误
	"fmt"  // 导入 fmt 包，用于格式化输出

	"github.com/bmatcuk/doublestar/v2"  // 导入 doublestar 包，用于处理文件路径匹配

	"github.com/anchore/syft/syft/file"  // 导入 file 包，用于处理文件
	"github.com/anchore/syft/syft/sbom"  // 导入 sbom 包，用于处理软件构建物料清单
)

var errDoesNotProvide = fmt.Errorf("cannot provide packages from the given source")  // 定义一个错误变量

// Provide a set of packages and context metadata describing where they were sourced from.
func Provide(userInput string, config ProviderConfig) ([]Package, Context, *sbom.SBOM, error) {
	// 调用 syftSBOMProvider 函数获取软件包、上下文元数据和软件构建物料清单
	packages, ctx, s, err := syftSBOMProvider(userInput, config)
	// 如果错误不是 errDoesNotProvide，则执行以下逻辑
	if !errors.Is(err, errDoesNotProvide) {
		// 如果排除列表不为空，则过滤软件包
		if len(config.Exclusions) > 0 {
			packages, err = filterPackageExclusions(packages, config.Exclusions)
		// 如果发生错误，返回空值、上下文、状态和错误
		if err != nil {
			return nil, ctx, s, err
		}
	}
	// 返回包列表、上下文、状态和错误
	return packages, ctx, s, err
}

// 使用用户提供的输入获取包，如果出现错误且不是 errDoesNotProvide，则返回包、空上下文、状态和错误
packages, err = purlProvider(userInput)
if !errors.Is(err, errDoesNotProvide) {
	// 如果错误不是 errDoesNotProvide，则返回包、空上下文、状态和错误
	return packages, Context{}, s, err
}

// 使用用户提供的输入和配置获取包
return syftProvider(userInput, config)
}

// 根据一组排除表达式过滤提供的包列表。排除表达式允许使用通配符。只有当所有位置都匹配提供的排除表达式时，包才会被排除。
func filterPackageExclusions(packages []Package, exclusions []string) ([]Package, error) {
	var out []Package
# 遍历包列表中的每个包
	for _, pkg := range packages:
		# 默认情况下包是要包含的
		includePackage := true
		# 获取包的所有位置
		locations := pkg.Locations.ToSlice()
		# 如果包含有位置信息
		if len(locations) > 0:
			# 默认情况下不包含该包
			includePackage = false
			# 对于每个位置，检查是否需要排除该包
		location:
			for _, location := range locations:
				# 对于每个位置，检查是否需要排除该包
				for _, exclusion := range exclusions:
					# 检查位置是否匹配排除规则
					match, err := locationMatches(location, exclusion)
					# 如果出现错误，返回错误
					if err != nil:
						return nil, err
					# 如果位置匹配排除规则，继续下一个位置的检查
					if match:
						continue location
				# 如果位置没有匹配任何排除规则，包含该包
				includePackage = true
				break
// 检查位置的真实路径和虚拟路径是否与排除参数匹配
// 排除参数允许使用通配符表达式，如`/usr/**`或`**/*.json`。如果排除参数是无效模式，则返回错误；否则，返回的布尔值表示匹配。
func locationMatches(location file.Location, exclusion string) (bool, error) {
    // 使用排除参数检查真实路径是否匹配
    matchesRealPath, err := doublestar.Match(exclusion, location.RealPath)
    if err != nil {
        return false, err
    }
    // 使用排除参数检查虚拟路径是否匹配
    matchesVirtualPath, err := doublestar.Match(exclusion, location.AccessPath)
    if err != nil {
        return false, err
    }
# 返回 matchesRealPath 或 matchesVirtualPath，以及一个空的错误
} 
return matchesRealPath || matchesVirtualPath, nil
```