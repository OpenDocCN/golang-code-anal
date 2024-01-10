# `grype\grype\presenter\table\presenter_test.go`

```
package table

import (
    "bytes" // 导入 bytes 包，用于操作字节
    "testing" // 导入 testing 包，用于编写测试函数

    "github.com/gkampitakis/go-snaps/snaps" // 导入外部包

    "github.com/go-test/deep" // 导入外部包，用于深度比较
    "github.com/google/go-cmp/cmp" // 导入外部包，用于比较两个值
    "github.com/stretchr/testify/assert" // 导入外部包，用于编写断言
    "github.com/stretchr/testify/require" // 导入外部包，用于编写测试函数

    "github.com/anchore/grype/grype/match" // 导入外部包
    "github.com/anchore/grype/grype/pkg" // 导入外部包
    "github.com/anchore/grype/grype/presenter/internal" // 导入外部包
    "github.com/anchore/grype/grype/presenter/models" // 导入外部包
    "github.com/anchore/grype/grype/vulnerability" // 导入外部包
    syftPkg "github.com/anchore/syft/syft/pkg" // 导入外部包，并重命名为 syftPkg
)

func TestCreateRow(t *testing.T) {
    pkg1 := pkg.Package{ // 创建一个 pkg.Package 结构体实例
        ID:      "package-1-id", // 设置 ID 字段值
        Name:    "package-1", // 设置 Name 字段值
        Version: "1.0.1", // 设置 Version 字段值
        Type:    syftPkg.DebPkg, // 设置 Type 字段值为 syftPkg.DebPkg
    }
    match1 := match.Match{ // 创建一个 match.Match 结构体实例
        Vulnerability: vulnerability.Vulnerability{ // 设置 Vulnerability 字段值为 vulnerability.Vulnerability 结构体实例
            ID:        "CVE-1999-0001", // 设置 ID 字段值
            Namespace: "source-1", // 设置 Namespace 字段值
        },
        Package: pkg1, // 设置 Package 字段值为 pkg1
        Details: []match.Detail{ // 设置 Details 字段值为 match.Detail 结构体切片
            {
                Type:    match.ExactDirectMatch, // 设置 Type 字段值为 match.ExactDirectMatch
                Matcher: match.DpkgMatcher, // 设置 Matcher 字段值为 match.DpkgMatcher
            },
        },
    }
    cases := []struct { // 创建一个匿名结构体切片
        name           string // 定义 name 字段为字符串类型
        match          match.Match // 定义 match 字段为 match.Match 类型
        severitySuffix string // 定义 severitySuffix 字段为字符串类型
        expectedErr    error // 定义 expectedErr 字段为 error 类型
        expectedRow    []string // 定义 expectedRow 字段为字符串切片
    }{
        {
            name:           "create row for vulnerability", // 设置 name 字段值
            match:          match1, // 设置 match 字段值为 match1
            severitySuffix: "", // 设置 severitySuffix 字段值为空字符串
            expectedErr:    nil, // 设置 expectedErr 字段值为 nil
            expectedRow:    []string{match1.Package.Name, match1.Package.Version, "", string(match1.Package.Type), match1.Vulnerability.ID, "Low"}, // 设置 expectedRow 字段值为字符串切片
        },
        {
            name:           "create row for suppressed vulnerability", // 设置 name 字段值
            match:          match1, // 设置 match 字段值为 match1
            severitySuffix: appendSuppressed, // 设置 severitySuffix 字段值为 appendSuppressed
            expectedErr:    nil, // 设置 expectedErr 字段值为 nil
            expectedRow:    []string{match1.Package.Name, match1.Package.Version, "", string(match1.Package.Type), match1.Vulnerability.ID, "Low (suppressed)"}, // 设置 expectedRow 字段值为字符串切片
        },
    }
    # 遍历测试用例数组
    for _, testCase := range cases {
        # 使用测试用例的名称创建子测试
        t.Run(testCase.name, func(t *testing.T) {
            # 调用 createRow 函数，传入测试用例的匹配条件、模拟的元数据对象和严重性后缀
            row, err := createRow(testCase.match, models.NewMetadataMock(), testCase.severitySuffix)

            # 断言：验证返回的错误是否符合预期
            assert.Equal(t, testCase.expectedErr, err)
            # 断言：验证返回的行数据是否符合预期
            assert.Equal(t, testCase.expectedRow, row)
        })
    }
func TestTablePresenter(t *testing.T) {
    // 创建一个字节缓冲区
    var buffer bytes.Buffer
    // 调用GenerateAnalysis函数，获取返回的多个变量
    _, matches, packages, _, metadataProvider, _, _ := internal.GenerateAnalysis(t, internal.ImageSource)

    // 创建一个PresenterConfig结构体
    pb := models.PresenterConfig{
        Matches:          matches,
        Packages:         packages,
        MetadataProvider: metadataProvider,
    }

    // 根据PresenterConfig创建一个Presenter对象
    pres := NewPresenter(pb, false)

    // 运行测试用例"no color"
    t.Run("no color", func(t *testing.T) {
        // 设置withColor字段为true
        pres.withColor = true

        // 调用Present方法，将结果写入buffer
        err := pres.Present(&buffer)
        require.NoError(t, err)

        // 将buffer内容转换为字符串
        actual := buffer.String()
        // 使用MatchSnapshot函数进行断言
        snaps.MatchSnapshot(t, actual)
    })

    // 运行测试用例"with color"
    t.Run("with color", func(t *testing.T) {
        // 设置withColor字段为false
        pres.withColor = false

        // 调用Present方法，将结果写入buffer
        err := pres.Present(&buffer)
        require.NoError(t, err)

        // 将buffer内容转换为字符串
        actual := buffer.String()
        // 使用MatchSnapshot函数进行断言
        snaps.MatchSnapshot(t, actual)
    })

    // TODO: add me back in when there is a JSON schema
    // validateAgainstDbSchema(t, string(actual))
}

func TestEmptyTablePresenter(t *testing.T) {
    // 期望没有输出

    // 创建一个字节缓冲区
    var buffer bytes.Buffer

    // 创建一个空的Matches对象
    matches := match.NewMatches()

    // 创建一个PresenterConfig结构体
    pb := models.PresenterConfig{
        Matches:          matches,
        Packages:         nil,
        MetadataProvider: nil,
    }

    // 根据PresenterConfig创建一个Presenter对象
    pres := NewPresenter(pb, false)

    // 运行Presenter的Present方法，将结果写入buffer
    err := pres.Present(&buffer)
    require.NoError(t, err)

    // 将buffer内容转换为字符串
    actual := buffer.String()
    // 使用MatchSnapshot函数进行断言
    snaps.MatchSnapshot(t, actual)
}

func TestRemoveDuplicateRows(t *testing.T) {
    // 创建一个二维字符串数组
    data := [][]string{
        {"1", "2", "3"},
        {"a", "b", "c"},
        {"1", "2", "3"},
        {"a", "b", "c"},
        {"1", "2", "3"},
        {"4", "5", "6"},
        {"1", "2", "1"},
    }

    // 创建一个期望的二维字符串数组
    expected := [][]string{
        {"1", "2", "3"},
        {"a", "b", "c"},
        {"4", "5", "6"},
        {"1", "2", "1"},
    }

    // 调用removeDuplicateRows函数，传入data数组，返回去重后的数组
    actual := removeDuplicateRows(data)
}
    # 使用深度比较函数 deep.Equal 比较 expected 和 actual，如果有差异则返回差异列表
    if diffs := deep.Equal(expected, actual); len(diffs) > 0:
        # 如果有差异，则输出错误信息
        t.Errorf("found diffs!")
        # 遍历差异列表，输出每个差异的详细信息
        for _, d := range diffs:
            t.Errorf("   diff: %+v", d)
# 测试排序行的函数
func TestSortRows(t *testing.T):
    # 定义测试数据
    data := [][]string{
        {"a", "v0.1.0", "", "deb", "CVE-2019-9996", "Critical"},
        {"a", "v0.1.0", "", "deb", "CVE-2018-9996", "Critical"},
        {"a", "v0.2.0", "", "deb", "CVE-2010-9996", "High"},
        {"b", "v0.2.0", "", "deb", "CVE-2010-9996", "Medium"},
        {"b", "v0.2.0", "", "deb", "CVE-2019-9996", "High"},
        {"d", "v0.4.0", "", "node", "CVE-2011-9996", "Low"},
        {"d", "v0.4.0", "", "node", "CVE-2012-9996", "Negligible"},
        {"c", "v0.6.0", "", "node", "CVE-2013-9996", "Critical"},
    }
    # 定义预期结果
    expected := [][]string{
        {"a", "v0.1.0", "", "deb", "CVE-2019-9996", "Critical"},
        {"a", "v0.1.0", "", "deb", "CVE-2018-9996", "Critical"},
        {"a", "v0.2.0", "", "deb", "CVE-2010-9996", "High"},
        {"b", "v0.2.0", "", "deb", "CVE-2019-9996", "High"},
        {"b", "v0.2.0", "", "deb", "CVE-2010-9996", "Medium"},
        {"c", "v0.6.0", "", "node", "CVE-2013-9996", "Critical"},
        {"d", "v0.4.0", "", "node", "CVE-2011-9996", "Low"},
        {"d", "v0.4.0", "", "node", "CVE-2012-9996", "Negligible"},
    }
    # 调用排序函数
    actual := sortRows(data)
    # 检查实际结果与预期结果是否一致
    if diff := cmp.Diff(expected, actual); diff != "":
        t.Errorf("sortRows() mismatch (-want +got):\n%s", diff)

# 测试隐藏被忽略的匹配项的函数
func TestHidesIgnoredMatches(t *testing.T):
    # 定义变量
    var buffer bytes.Buffer
    matches, ignoredMatches, packages, _, metadataProvider, _, _ := internal.GenerateAnalysisWithIgnoredMatches(t, internal.ImageSource)
    # 创建 PresenterConfig 对象
    pb := models.PresenterConfig{
        Matches:          matches,
        IgnoredMatches:   ignoredMatches,
        Packages:         packages,
        MetadataProvider: metadataProvider,
    }
    # 创建 Presenter 对象
    pres := NewPresenter(pb, false)
    # 调用 Present 方法
    err := pres.Present(&buffer)
    require.NoError(t, err)
    # 获取实际结果
    actual := buffer.String()
    # 使用 snaps 包进行快照测试
    snaps.MatchSnapshot(t, actual)

# 测试显示被忽略的匹配项的函数
func TestDisplaysIgnoredMatches(t *testing.T):
    # 定义变量
    var buffer bytes.Buffer
    # 调用 internal.GenerateAnalysisWithIgnoredMatches 函数生成匹配结果、被忽略的匹配结果、包信息、元数据提供者等变量
    matches, ignoredMatches, packages, _, metadataProvider, _, _ := internal.GenerateAnalysisWithIgnoredMatches(t, internal.ImageSource)
    
    # 创建 PresenterConfig 结构体对象 pb，包含匹配结果、被忽略的匹配结果、包信息、元数据提供者等字段
    pb := models.PresenterConfig{
        Matches:          matches,
        IgnoredMatches:   ignoredMatches,
        Packages:         packages,
        MetadataProvider: metadataProvider,
    }
    
    # 根据 PresenterConfig 对象 pb 创建新的 Presenter 对象 pres
    pres := NewPresenter(pb, true)
    
    # 调用 Presenter 对象 pres 的 Present 方法，将结果写入 buffer
    err := pres.Present(&buffer)
    require.NoError(t, err)
    
    # 将 buffer 中的内容转换为字符串，存入 actual 变量
    actual := buffer.String()
    
    # 使用 snaps.MatchSnapshot 方法对 actual 进行快照测试
    snaps.MatchSnapshot(t, actual)
# 闭合前面的函数定义
```