# `grype\grype\match\matches_test.go`

```go
package match

import (
    "testing"

    "github.com/google/uuid"  // 导入 Google 的 UUID 包
    "github.com/stretchr/testify/assert"  // 导入 testify 的 assert 包
    "github.com/stretchr/testify/require"  // 导入 testify 的 require 包

    "github.com/anchore/grype/grype/pkg"  // 导入 grype 的 pkg 包
    "github.com/anchore/grype/grype/vulnerability"  // 导入 grype 的 vulnerability 包
    syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syft 的 pkg 包并重命名为 syftPkg
)

func TestMatchesSortMixedDimensions(t *testing.T) {
    first := Match{  // 创建 Match 结构体实例 first
        Vulnerability: vulnerability.Vulnerability{  // 设置 Vulnerability 字段
            ID: "CVE-2020-0010",  // 设置 ID 字段
        },
        Package: pkg.Package{  // 设置 Package 字段
            ID:      pkg.ID(uuid.NewString()),  // 设置 ID 字段为新生成的 UUID 字符串
            Name:    "package-b",  // 设置 Name 字段
            Version: "1.0.0",  // 设置 Version 字段
            Type:    syftPkg.RpmPkg,  // 设置 Type 字段为 syftPkg 中的 RpmPkg
        },
    }
    second := Match{  // 创建 Match 结构体实例 second
        Vulnerability: vulnerability.Vulnerability{  // 设置 Vulnerability 字段
            ID: "CVE-2020-0020",  // 设置 ID 字段
        },
        Package: pkg.Package{  // 设置 Package 字段
            ID:      pkg.ID(uuid.NewString()),  // 设置 ID 字段为新生成的 UUID 字符串
            Name:    "package-a",  // 设置 Name 字段
            Version: "1.0.0",  // 设置 Version 字段
            Type:    syftPkg.NpmPkg,  // 设置 Type 字段为 syftPkg 中的 NpmPkg
        },
    }
    third := Match{  // 创建 Match 结构体实例 third
        Vulnerability: vulnerability.Vulnerability{  // 设置 Vulnerability 字段
            ID: "CVE-2020-0020",  // 设置 ID 字段
        },
        Package: pkg.Package{  // 设置 Package 字段
            ID:      pkg.ID(uuid.NewString()),  // 设置 ID 字段为新生成的 UUID 字符串
            Name:    "package-a",  // 设置 Name 字段
            Version: "2.0.0",  // 设置 Version 字段
            Type:    syftPkg.RpmPkg,  // 设置 Type 字段为 syftPkg 中的 RpmPkg
        },
    }
    fourth := Match{  // 创建 Match 结构体实例 fourth
        Vulnerability: vulnerability.Vulnerability{  // 设置 Vulnerability 字段
            ID: "CVE-2020-0020",  // 设置 ID 字段
        },
        Package: pkg.Package{  // 设置 Package 字段
            ID:      pkg.ID(uuid.NewString()),  // 设置 ID 字段为新生成的 UUID 字符串
            Name:    "package-c",  // 设置 Name 字段
            Version: "3.0.0",  // 设置 Version 字段
            Type:    syftPkg.ApkPkg,  // 设置 Type 字段为 syftPkg 中的 ApkPkg
        },
    }
    fifth := Match{  // 创建 Match 结构体实例 fifth
        Vulnerability: vulnerability.Vulnerability{  // 设置 Vulnerability 字段
            ID: "CVE-2020-0020",  // 设置 ID 字段
        },
        Package: pkg.Package{  // 设置 Package 字段
            ID:      pkg.ID(uuid.NewString()),  // 设置 ID 字段为新生成的 UUID 字符串
            Name:    "package-d",  // 设置 Name 字段
            Version: "2.0.0",  // 设置 Version 字段
            Type:    syftPkg.RpmPkg,  // 设置 Type 字段为 syftPkg 中的 RpmPkg
        },
    }
    // 创建一个包含 Match 结构的切片 input，按照指定顺序排列
    input := []Match{
        // 重新排列漏洞ID、软件包名称、软件包版本和软件包类型
        fifth, third, first, second, fourth,
    }
    // 使用 input 切片创建一个新的 Matches 对象
    matches := NewMatches(input...)

    // 断言 matches.Sorted() 方法返回的排序后的结果与指定顺序一致
    assertMatchOrder(t, []Match{first, second, third, fourth, fifth}, matches.Sorted())
func TestMatchesSortByVulnerability(t *testing.T) {
    // 创建第一个匹配对象
    first := Match{
        Vulnerability: vulnerability.Vulnerability{
            ID: "CVE-2020-0010",
        },
        Package: pkg.Package{
            ID:      pkg.ID(uuid.NewString()),
            Name:    "package-b",
            Version: "1.0.0",
            Type:    syftPkg.RpmPkg,
        },
    }
    // 创建第二个匹配对象
    second := Match{
        Vulnerability: vulnerability.Vulnerability{
            ID: "CVE-2020-0020",
        },
        Package: pkg.Package{
            ID:      pkg.ID(uuid.NewString()),
            Name:    "package-b",
            Version: "1.0.0",
            Type:    syftPkg.RpmPkg,
        },
    }

    // 创建匹配对象数组
    input := []Match{
        second, first,
    }
    // 创建匹配对象集合
    matches := NewMatches(input...)

    // 断言匹配对象集合按照漏洞排序后的顺序
    assertMatchOrder(t, []Match{first, second}, matches.Sorted())

}

func TestMatches_AllByPkgID(t *testing.T) {
    // 创建第一个匹配对象
    first := Match{
        Vulnerability: vulnerability.Vulnerability{
            ID: "CVE-2020-0010",
        },
        Package: pkg.Package{
            ID:      pkg.ID(uuid.NewString()),
            Name:    "package-b",
            Version: "1.0.0",
            Type:    syftPkg.RpmPkg,
        },
    }
    // 创建第二个匹配对象
    second := Match{
        Vulnerability: vulnerability.Vulnerability{
            ID: "CVE-2020-0010",
        },
        Package: pkg.Package{
            ID:      pkg.ID(uuid.NewString()),
            Name:    "package-c",
            Version: "1.0.0",
            Type:    syftPkg.RpmPkg,
        },
    }

    // 创建匹配对象数组
    input := []Match{
        second, first,
    }
    // 创建匹配对象集合
    matches := NewMatches(input...)

    // 期望的匹配对象按照包ID分组的结果
    expected := map[pkg.ID][]Match{
        first.Package.ID: {
            first,
        },
        second.Package.ID: {
            second,
        },
    }

    // 断言匹配对象集合按照包ID分组后的结果
    assert.Equal(t, expected, matches.AllByPkgID())

}

func TestMatchesSortByPackage(t *testing.T) {
    # 创建第一个匹配对象，包含漏洞和软件包信息
    first := Match{
        Vulnerability: vulnerability.Vulnerability{
            ID: "CVE-2020-0010",
        },
        Package: pkg.Package{
            ID:      pkg.ID(uuid.NewString()),
            Name:    "package-b",
            Version: "1.0.0",
            Type:    syftPkg.RpmPkg,
        },
    }
    # 创建第二个匹配对象，包含漏洞和软件包信息
    second := Match{
        Vulnerability: vulnerability.Vulnerability{
            ID: "CVE-2020-0010",
        },
        Package: pkg.Package{
            ID:      pkg.ID(uuid.NewString()),
            Name:    "package-c",
            Version: "1.0.0",
            Type:    syftPkg.RpmPkg,
        },
    }

    # 创建匹配对象数组
    input := []Match{
        second, first,
    }
    # 使用匹配对象数组创建匹配对象集合
    matches := NewMatches(input...)

    # 断言匹配对象集合的顺序是否符合预期
    assertMatchOrder(t, []Match{first, second}, matches.Sorted())
func TestMatchesSortByPackageVersion(t *testing.T) {
    // 创建第一个匹配对象
    first := Match{
        Vulnerability: vulnerability.Vulnerability{
            ID: "CVE-2020-0010",
        },
        Package: pkg.Package{
            ID:      pkg.ID(uuid.NewString()),
            Name:    "package-b",
            Version: "1.0.0",
            Type:    syftPkg.RpmPkg,
        },
    }
    // 创建第二个匹配对象
    second := Match{
        Vulnerability: vulnerability.Vulnerability{
            ID: "CVE-2020-0010",
        },
        Package: pkg.Package{
            ID:      pkg.ID(uuid.NewString()),
            Name:    "package-b",
            Version: "2.0.0",
            Type:    syftPkg.RpmPkg,
        },
    }

    // 创建匹配对象数组
    input := []Match{
        second, first,
    }
    // 创建匹配对象集合
    matches := NewMatches(input...)

    // 断言匹配对象集合按照包版本排序后的顺序
    assertMatchOrder(t, []Match{first, second}, matches.Sorted())

}

func TestMatchesSortByPackageType(t *testing.T) {
    // 创建第一个匹配对象
    first := Match{
        Vulnerability: vulnerability.Vulnerability{
            ID: "CVE-2020-0010",
        },
        Package: pkg.Package{
            ID:      pkg.ID(uuid.NewString()),
            Name:    "package-b",
            Version: "1.0.0",
            Type:    syftPkg.ApkPkg,
        },
    }
    // 创建第二个匹配对象
    second := Match{
        Vulnerability: vulnerability.Vulnerability{
            ID: "CVE-2020-0010",
        },
        Package: pkg.Package{
            ID:      pkg.ID(uuid.NewString()),
            Name:    "package-b",
            Version: "1.0.0",
            Type:    syftPkg.RpmPkg,
        },
    }

    // 创建匹配对象数组
    input := []Match{
        second, first,
    }
    // 创建匹配对象集合
    matches := NewMatches(input...)

    // 断言匹配对象集合按照包类型排序后的顺序
    assertMatchOrder(t, []Match{first, second}, matches.Sorted())

}

func assertMatchOrder(t *testing.T, expected, actual []Match) {
    // 创建期望结果的包名数组
    var expectedStr []string
    for _, e := range expected {
        expectedStr = append(expectedStr, e.Package.Name)
    }

    // 创建实际结果的包名数组
    var actualStr []string
    for _, a := range actual {
        actualStr = append(actualStr, a.Package.Name)
    }
}
    // 使用断言来比较两个字符串是否相等，如果不相等则会触发测试失败
    require.Equal(t, expectedStr, actualStr)
    
    // 使用断言来比较两个值是否相等，如果不相等则会触发测试失败
    assert.Equal(t, expected, actual)
func assertIgnoredMatchOrder(t *testing.T, expected, actual []IgnoredMatch) {
    // 定义一个函数，用于断言忽略匹配的顺序是否符合预期
    var expectedStr []string
    // 遍历预期结果，将包名添加到字符串数组中
    for _, e := range expected {
        expectedStr = append(expectedStr, e.Package.Name)
    }

    var actualStr []string
    // 遍历实际结果，将包名添加到字符串数组中
    for _, a := range actual {
        actualStr = append(actualStr, a.Package.Name)
    }

    // 使用 require.Equal 函数断言预期结果和实际结果是否相等
    require.Equal(t, expectedStr, actualStr)

    // 使用 assert.Equal 函数断言预期结果和实际结果是否相等
    assert.Equal(t, expected, actual)
}

func TestMatches_Diff(t *testing.T) {
    a := Match{
        Vulnerability: vulnerability.Vulnerability{
            ID:        "vuln-a",
            Namespace: "name-a",
        },
        Package: pkg.Package{
            ID: "package-a",
        },
    }

    b := Match{
        Vulnerability: vulnerability.Vulnerability{
            ID:        "vuln-b",
            Namespace: "name-b",
        },
        Package: pkg.Package{
            ID: "package-b",
        },
    }

    c := Match{
        Vulnerability: vulnerability.Vulnerability{
            ID:        "vuln-c",
            Namespace: "name-c",
        },
        Package: pkg.Package{
            ID: "package-c",
        },
    }

    tests := []struct {
        name    string
        subject Matches
        other   Matches
        want    Matches
    }{
        {
            name:    "no diff",
            subject: NewMatches(a, b, c),
            other:   NewMatches(a, b, c),
            want:    newMatches(),
        },
        {
            name:    "extra items in subject",
            subject: NewMatches(a, b, c),
            other:   NewMatches(a, b),
            want:    NewMatches(c),
        },
        {
            // this demonstrates that this is not meant to implement a symmetric diff
            name:    "extra items in other (results in no diff)",
            subject: NewMatches(a, b),
            other:   NewMatches(a, b, c),
            want:    NewMatches(),
        },
    }
}
    }
    // 遍历测试用例数组
    for _, tt := range tests {
        // 使用测试名称创建子测试，并运行
        t.Run(tt.name, func(t *testing.T) {
            // 断言两个值是否相等，并在不相等时输出自定义的错误信息
            assert.Equalf(t, &tt.want, tt.subject.Diff(tt.other), "Diff(%v)", tt.other)
        })
    }
# 闭合前面的函数定义
```