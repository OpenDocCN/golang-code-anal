# `grype\grype\matcher\rpm\matcher.go`

```
package rpm

import (
    "fmt"
    "strings"

    "github.com/anchore/grype/grype/distro"
    "github.com/anchore/grype/grype/match"
    "github.com/anchore/grype/grype/pkg"
    "github.com/anchore/grype/grype/search"
    "github.com/anchore/grype/grype/vulnerability"
    syftPkg "github.com/anchore/syft/syft/pkg"
)

type Matcher struct {
}

func (m *Matcher) PackageTypes() []syftPkg.Type {
    return []syftPkg.Type{syftPkg.RpmPkg}
}

func (m *Matcher) Type() match.MatcherType {
    return match.RpmMatcher
}

//nolint:funlen
func (m *Matcher) Match(store vulnerability.Provider, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    matches := make([]match.Match, 0)

    // let's match with a synthetic package that doesn't exist. We will create a new
    // package that matches the same name and version as what is contained in the
    // "sourceRPM" field.

    // Regarding RPM epoch and comparisons... RedHat is explicit that when an RPM
    // epoch is not specified that it should be assumed to be zero (see
    // https://github.com/rpm-software-management/rpm/issues/450). This comment from
    // RedHat is applicable for a project that has elected to not use epoch and has
    // not changed their version scheme at all --therefore it is safe to assume that
    // the epoch (though not specified) is 0. However, in cases where there may be a
    // non-zero epoch and it has been omitted from the version string it is NOT safe
    // to assume an epoch of 0... as this could lead to misleading comparison
    // results.

    // For example, take the perl-Errno package:
    //        name:         perl-Errno
    //        version:    0:1.28-419.el8_4.1
    //        sourceRPM:    perl-5.26.3-419.el8_4.1.src.rpm

    // Say we have a vulnerability with the following information (note this is
    // against the SOURCE package "perl", not the target package, "perl-Errno"):
    //         ID:                    CVE-2020-10543
    //        Package Name:        perl
    //        Version constraint:    < 4:5.26.3-419.el8
    // 版本约束： < 4:5.26.3-419.el8

    // Note that the vulnerability information has complete knowledge about the
    // version and it's lineage (epoch + version), however, the source package
    // information for perl-Errno does not include any information about epoch. With
    // the rule from RedHat we should assume a 0 epoch and make the comparison:
    // 注意，漏洞信息完全了解版本及其血统（epoch + version），但是 perl-Errno 的源包信息不包含任何有关 epoch 的信息。根据 RedHat 的规则，我们应该假设为 0 epoch 并进行比较：

    //        0:5.26.3-419.el8 < 4:5.26.3-419.el8 = true! ... therefore we are vulnerable since epoch 0 < 4.
    //                                                  ... this is an INVALID comparison!
    // 0:5.26.3-419.el8 < 4:5.26.3-419.el8 = true！... 因此我们是脆弱的，因为 epoch 0 < 4。
    //                                                  ... 这是一个无效的比较！

    // The problem with this is that sourceRPMs tend to not specify epoch even though
    // there may be a non-zero epoch for that package! This is important. The "more
    // correct" thing to do in this case is to drop the epoch:
    // 这样做的问题在于，sourceRPMs 倾向于不指定 epoch，即使对于该软件包可能存在非零的 epoch！这很重要。在这种情况下，“更正确”的做法是删除 epoch：

    //        5.26.3-419.el8 < 5.26.3-419.el8 = false!    ... these are the SAME VERSION
    // 5.26.3-419.el8 < 5.26.3-419.el8 = false！... 这些是相同的版本

    // There is still a problem with this approach: it essentially makes an
    // assumption that a missing epoch really is the SAME epoch to the other version
    // being compared (in our example, no perl epoch on one side means we should
    // really assume an epoch of 4 on the other side). This could still lead to
    // problems since an epoch delimits potentially non-comparable version lineages.
    // 这种方法仍然存在问题：它基本上假设缺失的 epoch 确实是与另一个版本进行比较的相同 epoch（在我们的例子中，一侧没有 perl epoch 意味着我们应该真的假设另一侧的 epoch 为 4）。这仍然可能导致问题，因为 epoch 可能限制了潜在的不可比较的版本血统。

    sourceMatches, err := m.matchUpstreamPackages(store, d, p)
    if err != nil {
        return nil, fmt.Errorf("failed to match by source indirection: %w", err)
    }
    matches = append(matches, sourceMatches...)
    // 让我们与给定的软件包进行匹配（直接匹配）。

    // Regarding RPM epochs... we know that the package and vulnerability will have
    // well specified epochs since both are sourced from either the RPMDB directly or
    // the upstream RedHat vulnerability data. Note: this is very much UNLIKE our
    // matching on a source package above where the epoch could be dropped in the
    // context of a comparison.
    // 关于 RPM epochs... 我们知道软件包和漏洞将具有明确定义的 epochs，因为两者都是直接从 RPMDB 或上游 RedHat 漏洞数据获取的。注意：这与我们上面对源包进行匹配的情况非常不同，在那里 epoch 可能在比较的上下文中被删除。
    // 参考数据。这意味着任何缺失的时代都可以假定为零，
    // 因为它属于“项目选择不对第一个版本方案进行时代”而不是其他情况。

    // 准确匹配一个软件包时，我们应该明确指定时代（因为下游版本比较逻辑在上述提到的情况下会在比较时去掉时代 -- 基本上是针对源 RPM 情况）。为了做到这一点，我们用显式的 0 填充软件包版本中缺失的时代值。

    // 进行准确匹配，如果出错则返回错误
    exactMatches, err := m.matchPackage(store, d, p)
    if err != nil {
        return nil, fmt.Errorf("failed to match by exact package name: %w", err)
    }

    // 将准确匹配的结果追加到匹配列表中
    matches = append(matches, exactMatches...)

    // 返回匹配结果和空错误
    return matches, nil
// 匹配上游软件包，返回匹配结果和错误信息
func (m *Matcher) matchUpstreamPackages(store vulnerability.ProviderByDistro, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    var matches []match.Match

    // 遍历上游软件包列表
    for _, indirectPackage := range pkg.UpstreamPackages(p) {
        // 根据上游软件包和发行版查找漏洞
        indirectMatches, err := search.ByPackageDistro(store, d, indirectPackage, m.Type())
        if err != nil {
            return nil, fmt.Errorf("failed to find vulnerabilities for rpm upstream source package: %w", err)
        }
        // 将匹配结果添加到匹配列表中
        matches = append(matches, indirectMatches...)
    }

    // 确保跟踪匹配结果是基于 SBOM 中的软件包而不是间接软件包
    // 匹配详情已经包含了用于匹配的具体间接软件包信息
    match.ConvertToIndirectMatches(matches, p)

    return matches, nil
}

// 匹配软件包，返回匹配结果和错误信息
func (m *Matcher) matchPackage(store vulnerability.ProviderByDistro, d *distro.Distro, p pkg.Package) ([]match.Match, error) {
    // 确保版本号始终包含指定的 epoch
    originalPkg := p

    p.Version = addZeroEpicIfApplicable(p.Version)

    // 根据软件包和发行版查找漏洞
    matches, err := search.ByPackageDistro(store, d, p, m.Type())
    if err != nil {
        return nil, fmt.Errorf("failed to find vulnerabilities by dpkg source indirection: %w", err)
    }

    // 确保跟踪匹配结果是基于 SBOM 中的软件包而不是修改后的软件包
    for idx := range matches {
        matches[idx].Package = originalPkg
    }

    return matches, nil
}

// 如果适用，添加 epoch 为 0
func addZeroEpicIfApplicable(version string) string {
    if strings.Contains(version, ":") {
        return version
    }
    return "0:" + version
}
```