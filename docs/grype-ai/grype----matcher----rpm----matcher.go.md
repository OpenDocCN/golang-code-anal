# `grype\grype\matcher\rpm\matcher.go`

```
package rpm

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"strings"  // 导入 strings 包，用于处理字符串

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
	return []syftPkg.Type{syftPkg.RpmPkg}  // 返回 RPM 包类型
}
// 返回匹配器的类型
func (m *Matcher) Type() match.MatcherType {
	return match.RpmMatcher
}

//nolint:funlen
// 匹配漏洞
func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
	// 创建一个空的匹配结果数组
	matches := make([]match.Match, 0)

	// 创建一个合成的包，用于匹配与“sourceRPM”字段中相同名称和版本的包
	// 关于 RPM epoch 和比较... RedHat 明确指出，当未指定 RPM epoch 时，应假定为零（参见 https://github.com/rpm-software-management/rpm/issues/450）。这个来自 RedHat 的评论适用于选择不使用 epoch 并且完全没有更改其版本方案的项目--因此可以安全地假定 epoch（虽然未指定）为 0。然而，在可能存在非零 epoch 并且已从版本字符串中省略的情况下，就不安全了
// 假设一个纪元为0...，因为这可能会导致误导性的比较结果。

// 例如，以perl-Errno软件包为例：
//		名称：		perl-Errno
//		版本：		0:1.28-419.el8_4.1
//		sourceRPM：	perl-5.26.3-419.el8_4.1.src.rpm

// 假设我们有一个以下信息的漏洞（注意这是针对源软件包“perl”，而不是目标软件包“perl-Errno”）：
//		ID：					CVE-2020-10543
//		软件包名称：		perl
//		版本约束：		< 4:5.26.3-419.el8

// 请注意，漏洞信息完全了解版本及其血统（纪元+版本），然而，perl-Errno的源软件包信息不包括任何关于纪元的信息。根据RedHat的规则，我们应该假设纪元为0并进行比较：

//		0:5.26.3-419.el8 < 4:5.26.3-419.el8 = true！... 因此我们是有漏洞的，因为纪元0 < 4。
// 这是一个无效的比较！

// 这个问题在于，sourceRPMs往往不指定epoch，即使对于该软件包可能存在非零的epoch！这很重要。在这种情况下“更正确”的做法是去掉epoch：

// 5.26.3-419.el8 < 5.26.3-419.el8 = false！... 这些是相同的版本

// 这种方法仍然存在问题：它基本上假设缺少epoch实际上是与另一个版本相同的epoch（在我们的例子中，一侧没有perl epoch意味着我们应该真的假设另一侧的epoch是4）。这仍然可能导致问题，因为epoch界定了潜在的不可比较的版本谱系。

// 通过调用matchUpstreamPackages方法匹配上游软件包
sourceMatches, err := m.matchUpstreamPackages(store, d, p)
if err != nil {
    return nil, fmt.Errorf("failed to match by source indirection: %w", err)
}
// 将匹配结果追加到matches切片中
matches = append(matches, sourceMatches...)
// 让我们与给定的软件包进行直接匹配。

// 关于 RPM epochs... 我们知道软件包和漏洞将具有明确定义的 epochs，因为两者都是从 RPMDB 直接或上游 RedHat 漏洞数据获取的。注意：这与我们在上面匹配源软件包时非常不同，因为在参考数据中可能会丢弃 epoch。这意味着任何缺失的 epoch 可以假定为零，因为它属于“项目选择在第一个版本方案中不使用 epoch”的情况，而不属于其他任何情况。

// 准确匹配软件包的原因是我们应该明确关于 epoch（因为下游版本比较逻辑在上述情况下会在比较时去掉 epoch -- 基本上是针对源 RPM 的情况）。为了做到这一点，我们用显式的 0 填充软件包版本中缺失的 epoch 值。

exactMatches, err := m.matchPackage(store, d, p)
if err != nil {
    return nil, fmt.Errorf("failed to match by exact package name: %w", err)
}
// 将 exactMatches 切片中的元素追加到 matches 切片中
matches = append(matches, exactMatches...)
// 返回 matches 切片和空指针
return matches, nil
}

// 匹配上游软件包
func (m *Matcher) matchUpstreamPackages(store vulnerability.ProviderByDistro, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
	// 创建一个空的 matches 切片
	var matches []match.Match

	// 遍历 pkg.UpstreamPackages(p) 返回的切片中的元素
	for _, indirectPackage := range pkg.UpstreamPackages(p) {
		// 使用 search.ByPackageDistro 方法查找漏洞
		indirectMatches, err := search.ByPackageDistro(store, d, indirectPackage, m.Type())
		// 如果发生错误，返回空指针和错误信息
		if err != nil {
			return nil, fmt.Errorf("failed to find vulnerabilities for rpm upstream source package: %w", err)
		}
		// 将 indirectMatches 切片中的元素追加到 matches 切片中
		matches = append(matches, indirectMatches...)
	}

	// 确保我们基于 SBOM 中的软件包进行匹配（而不是间接软件包）。
	// 匹配详细信息已经包含了用于进行匹配的具体间接软件包信息。
	// 将 matches 切片中的匹配转换为间接匹配，基于 SBOM 中的软件包
	match.ConvertToIndirectMatches(matches, p)
```

// matchPackage 方法用于匹配给定包的漏洞信息
func (m *Matcher) matchPackage(store vulnerability.ProviderByDistro, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
	// 确保版本号始终包含一个epoch
	originalPkg := p

	p.Version = addZeroEpicIfApplicable(p.Version)

	// 通过包和发行版信息搜索漏洞匹配
	matches, err := search.ByPackageDistro(store, d, p, m.Type())
	if err != nil {
		return nil, fmt.Errorf("failed to find vulnerabilities by dpkg source indirection: %w", err)
	}

	// 确保跟踪匹配是基于SBOM中的包而不是修改后的包
	for idx := range matches {
		matches[idx].Package = originalPkg
	}
}
# 返回匹配结果和空值
return matches, nil

# 如果版本号中包含冒号，则直接返回版本号
# 否则在版本号前加上"0:"，然后返回
func addZeroEpicIfApplicable(version string) string {
    if strings.Contains(version, ":") {
        return version
    }
    return "0:" + version
}
```