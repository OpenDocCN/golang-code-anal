# `grype\grype\presenter\table\presenter_test.go`

```
package table

import (
	"bytes"  // 导入 bytes 包，用于操作字节
	"testing"  // 导入 testing 包，用于编写测试函数

	"github.com/gkampitakis/go-snaps/snaps"  // 导入 go-snaps/snaps 包
	"github.com/go-test/deep"  // 导入 go-test/deep 包，用于深度比较
	"github.com/google/go-cmp/cmp"  // 导入 google/go-cmp/cmp 包，用于比较两个值
	"github.com/stretchr/testify/assert"  // 导入 testify/assert 包，用于编写断言
	"github.com/stretchr/testify/require"  // 导入 testify/require 包，用于编写测试函数中的必要条件

	"github.com/anchore/grype/grype/match"  // 导入 anchore/grype/grype/match 包
	"github.com/anchore/grype/grype/pkg"  // 导入 anchore/grype/grype/pkg 包
	"github.com/anchore/grype/grype/presenter/internal"  // 导入 anchore/grype/grype/presenter/internal 包
	"github.com/anchore/grype/grype/presenter/models"  // 导入 anchore/grype/grype/presenter/models 包
	"github.com/anchore/grype/grype/vulnerability"  // 导入 anchore/grype/grype/vulnerability 包
	syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 anchore/syft/syft/pkg 包，并重命名为 syftPkg
)
// 创建一个测试函数，用于测试创建行为
func TestCreateRow(t *testing.T) {
	// 创建一个名为pkg1的Package对象，包含ID、Name、Version和Type属性
	pkg1 := pkg.Package{
		ID:      "package-1-id",
		Name:    "package-1",
		Version: "1.0.1",
		Type:    syftPkg.DebPkg,
	}
	// 创建一个名为match1的Match对象，包含Vulnerability、Package和Details属性
	match1 := match.Match{
		// 包含Vulnerability对象，包含ID和Namespace属性
		Vulnerability: vulnerability.Vulnerability{
			ID:        "CVE-1999-0001",
			Namespace: "source-1",
		},
		// 包含Package对象，使用之前创建的pkg1对象
		Package: pkg1,
		// 包含Detail对象的数组，包含Type和Matcher属性
		Details: []match.Detail{
			{
				Type:    match.ExactDirectMatch,
				Matcher: match.DpkgMatcher,
			},
		},
	}
}
# 定义一个结构体切片，每个结构体包含名称、匹配、严重性后缀、预期错误和预期行
cases := []struct {
    name           string         // 名称
    match          match.Match    // 匹配对象
    severitySuffix string         // 严重性后缀
    expectedErr    error          // 预期错误
    expectedRow    []string       // 预期行
}{
    {
        name:           "create row for vulnerability",  // 漏洞的行
        match:          match1,                            // 匹配对象
        severitySuffix: "",                               // 严重性后缀为空
        expectedErr:    nil,                               // 预期错误为空
        expectedRow:    []string{match1.Package.Name, match1.Package.Version, "", string(match1.Package.Type), match1.Vulnerability.ID, "Low"},  // 预期行
    },
    {
        name:           "create row for suppressed vulnerability",  // 被抑制的漏洞的行
        match:          match1,                                       // 匹配对象
        severitySuffix: appendSuppressed,                            // 严重性后缀为appendSuppressed
        expectedErr:    nil,                                          // 预期错误为空
        expectedRow:    []string{match1.Package.Name, match1.Package.Version, "", string(match1.Package.Type), match1.Vulnerability.ID, "Low (suppressed)"},  // 预期行
    }
}
		},
	}

	for _, testCase := range cases {
		t.Run(testCase.name, func(t *testing.T) {
			// 对每个测试用例执行测试
			row, err := createRow(testCase.match, models.NewMetadataMock(), testCase.severitySuffix)

			// 断言测试结果是否符合预期
			assert.Equal(t, testCase.expectedErr, err)
			assert.Equal(t, testCase.expectedRow, row)
		})
	}
}

func TestTablePresenter(t *testing.T) {
	// 创建一个字节缓冲区
	var buffer bytes.Buffer
	// 生成分析所需的数据
	_, matches, packages, _, metadataProvider, _, _ := internal.GenerateAnalysis(t, internal.ImageSource)

	// 创建一个展示配置对象
	pb := models.PresenterConfig{
		Matches:          matches,
		Packages:         packages,
```
在这段代码中，主要是对测试用例进行执行和展示配置的创建，其中包括了对测试结果的断言。
		// 创建一个新的Presenter对象，传入metadataProvider和false作为参数
		pres := NewPresenter(pb, false)

		// 运行测试，测试不使用颜色的情况
		t.Run("no color", func(t *testing.T) {
			// 设置pres的withColor属性为true
			pres.withColor = true

			// 调用Present方法，将结果存储到buffer中
			err := pres.Present(&buffer)
			require.NoError(t, err)

			// 获取buffer中的字符串，与快照进行比较
			actual := buffer.String()
			snaps.MatchSnapshot(t, actual)
		})

		// 运行测试，测试使用颜色的情况
		t.Run("with color", func(t *testing.T) {
			// 设置pres的withColor属性为false
			pres.withColor = false

			// 调用Present方法，将结果存储到buffer中
			err := pres.Present(&buffer)
			require.NoError(t, err)
		// 将 buffer 中的内容转换为字符串
		actual := buffer.String()
		// 使用 snaps 包中的 MatchSnapshot 函数，比较 actual 和预期结果，输出测试结果
		snaps.MatchSnapshot(t, actual)
	})

	// TODO: add me back in when there is a JSON schema
	// 当有 JSON 模式时，将我添加回来
	// 使用 validateAgainstDbSchema 函数，验证 actual 是否符合数据库模式
	// validateAgainstDbSchema(t, string(actual))
}

func TestEmptyTablePresenter(t *testing.T) {
	// 预期没有输出

	// 创建一个字节缓冲区
	var buffer bytes.Buffer

	// 创建一个空的 matches 对象
	matches := match.NewMatches()

	// 创建一个 PresenterConfig 对象
	pb := models.PresenterConfig{
		Matches:          matches,
		Packages:         nil,
		MetadataProvider: nil,
	}

	// 创建一个新的Presenter对象，传入参数pb和false
	pres := NewPresenter(pb, false)

	// 运行Presenter对象的Present方法，传入参数&buffer
	err := pres.Present(&buffer)
	// 检查是否有错误发生
	require.NoError(t, err)

	// 将buffer转换为字符串
	actual := buffer.String()
	// 使用snaps.MatchSnapshot方法对actual进行快照测试
	snaps.MatchSnapshot(t, actual)
}

// 定义测试函数TestRemoveDuplicateRows
func TestRemoveDuplicateRows(t *testing.T) {
	// 创建一个包含多个子数组的二维数组data
	data := [][]string{
		{"1", "2", "3"},
		{"a", "b", "c"},
		{"1", "2", "3"},
		{"a", "b", "c"},
		{"1", "2", "3"},
		{"4", "5", "6"},
# 定义一个二维字符串数组，表示输入数据
data := [][]string{
	{"1", "2", "1"},
}

# 定义一个预期的二维字符串数组，表示去重后的数据
expected := [][]string{
	{"1", "2", "3"},
	{"a", "b", "c"},
	{"4", "5", "6"},
	{"1", "2", "1"},
}

# 调用 removeDuplicateRows 函数，去除输入数据中的重复行
actual := removeDuplicateRows(data)

# 检查预期结果和实际结果是否相等，如果不相等则输出差异
if diffs := deep.Equal(expected, actual); len(diffs) > 0 {
	t.Errorf("found diffs!")
	for _, d := range diffs {
		t.Errorf("   diff: %+v", d)
	}
}
# 定义一个测试函数，用于测试排序行的功能
func TestSortRows(t *testing.T) {
    # 定义一个包含多个字符串数组的数据
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

    # 定义一个期望的排序后的字符串数组数据
    expected := [][]string{
        {"a", "v0.1.0", "", "deb", "CVE-2019-9996", "Critical"},
        {"a", "v0.1.0", "", "deb", "CVE-2018-9996", "Critical"},
        {"a", "v0.2.0", "", "deb", "CVE-2010-9996", "High"},
        {"b", "v0.2.0", "", "deb", "CVE-2019-9996", "High"},
        {"b", "v0.2.0", "", "deb", "CVE-2010-9996", "Medium"},
        {"c", "v0.6.0", "", "node", "CVE-2013-9996", "Critical"},
        {"d", "v0.4.0", "", "node", "CVE-2011-9996", "Low"},
// 定义一个包含多个字符串的二维数组
data := [][]string{
	{"a", "v0.1.0", "", "node", "CVE-2012-9999", "Critical"},
	{"b", "v0.2.0", "", "node", "CVE-2012-9998", "High"},
	{"c", "v0.3.0", "", "node", "CVE-2012-9997", "Medium"},
	{"d", "v0.4.0", "", "node", "CVE-2012-9996", "Negligible"},
}

// 调用sortRows函数对data进行排序，并将结果赋值给actual变量
actual := sortRows(data)

// 使用cmp.Diff函数比较期望值和实际值，如果不相等则输出错误信息
if diff := cmp.Diff(expected, actual); diff != "" {
	t.Errorf("sortRows() mismatch (-want +got):\n%s", diff)
}

// 定义TestHidesIgnoredMatches测试函数
func TestHidesIgnoredMatches(t *testing.T) {
	// 初始化变量并调用GenerateAnalysisWithIgnoredMatches函数
	var buffer bytes.Buffer
	matches, ignoredMatches, packages, _, metadataProvider, _, _ := internal.GenerateAnalysisWithIgnoredMatches(t, internal.ImageSource)

	// 创建PresenterConfig结构体对象，并传入相关参数
	pb := models.PresenterConfig{
		Matches:          matches,
		IgnoredMatches:   ignoredMatches,
		Packages:         packages,
		MetadataProvider: metadataProvider,
	}
// 创建一个新的Presenter对象，传入参数pb和false
pres := NewPresenter(pb, false)

// 使用Presenter对象展示内容到buffer中
err := pres.Present(&buffer)
require.NoError(t, err)

// 将buffer中的内容转换为字符串
actual := buffer.String()

// 使用snaps.MatchSnapshot比较actual和预期结果，检查是否匹配
snaps.MatchSnapshot(t, actual)
}

// 测试展示被忽略的匹配项
func TestDisplaysIgnoredMatches(t *testing.T) {
	// 创建一个bytes.Buffer对象
	var buffer bytes.Buffer
	
	// 生成分析结果，获取匹配项、被忽略的匹配项、包信息、元数据提供者等
	matches, ignoredMatches, packages, _, metadataProvider, _, _ := internal.GenerateAnalysisWithIgnoredMatches(t, internal.ImageSource)

	// 创建一个PresenterConfig对象，传入匹配项、被忽略的匹配项、包信息、元数据提供者等参数
	pb := models.PresenterConfig{
		Matches:          matches,
		IgnoredMatches:   ignoredMatches,
		Packages:         packages,
		MetadataProvider: metadataProvider,
	}
```

# 创建一个新的Presenter对象，传入参数pb和true
pres := NewPresenter(pb, true)

# 调用Presenter对象的Present方法，将结果存储在buffer中
err := pres.Present(&buffer)
require.NoError(t, err)

# 将buffer中的内容转换为字符串存储在actual中
actual := buffer.String()

# 使用snaps包中的MatchSnapshot方法，比较actual和预期结果，如果不一致则会报错
snaps.MatchSnapshot(t, actual)
```