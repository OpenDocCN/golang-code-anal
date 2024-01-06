# `grype\grype\match\explicit_ignores_test.go`

```
package match

import (
	"testing"  // 导入测试包
	"github.com/stretchr/testify/assert"  // 导入断言包

	"github.com/anchore/grype/grype/pkg"  // 导入包
	"github.com/anchore/grype/grype/vulnerability"  // 导入包
	syftPkg "github.com/anchore/syft/syft/pkg"  // 导入包，重命名为syftPkg
)

// 定义一个mockExclusionProvider结构体
type mockExclusionProvider struct {
	data map[string][]IgnoreRule  // 结构体包含一个map类型的data字段
}

// 创建一个新的mockExclusionProvider对象
func newMockExclusionProvider() *mockExclusionProvider {
	// 初始化一个mockExclusionProvider对象
	d := mockExclusionProvider{
		data: make(map[string][]IgnoreRule),  // 使用make函数创建一个map
	}
// 调用 stub 方法，并返回 d 对象
d.stub()
return &d
}

// 定义 mockExclusionProvider 结构体的 stub 方法
func (d *mockExclusionProvider) stub() {
}

// 定义 mockExclusionProvider 结构体的 GetRules 方法，根据 vulnerabilityID 获取对应的 IgnoreRule 列表
func (d *mockExclusionProvider) GetRules(vulnerabilityID string) ([]IgnoreRule, error) {
	return d.data[vulnerabilityID], nil
}

// 定义测试函数 Test_ApplyExplicitIgnoreRules
func Test_ApplyExplicitIgnoreRules(t *testing.T) {
	// 定义 cvePkg 结构体，包含 cve 和 pkg 字段
	type cvePkg struct {
		cve string
		pkg string
	}
	// 定义测试用例列表，包含 name、typ、matches 字段
	tests := []struct {
		name     string
		typ      syftPkg.Type
		matches  []cvePkg
		expected []string  // 期望的结果字符串数组
		ignored  []string  // 忽略的结果字符串数组
	}{
		// 一些明确与log4j相关的数据:
		// "CVE-2021-44228", "CVE-2021-45046", "GHSA-jfh8-c2jp-5v3q", "GHSA-7rjr-3q55-vv33",
		// "log4j-api", "log4j-slf4j-impl", "log4j-to-slf4j", "log4j-1.2-api",
		{
			name: "keeps non-matching packages",  // 保留不匹配的软件包
			typ:  "java-archive",  // 类型为Java存档
			matches: []cvePkg{  // 匹配的CVE软件包数组
				{"CVE-2021-44228", "log4j-core"},  // CVE-2021-44228与log4j-core匹配
				{"CVE-2021-43452", "foo-tool"},  // CVE-2021-43452与foo-tool匹配
			},
			expected: []string{"log4j-core", "foo-tool"},  // 期望的结果为log4j-core和foo-tool
		},
		{
			name: "keeps non-matching CVEs",  // 保留不匹配的CVE
			typ:  "java-archive",  // 类型为Java存档
			matches: []cvePkg{  // 匹配的CVE软件包数组
				{"CVE-2021-428", "log4j-api"},  // CVE-2021-428与log4j-api匹配
# 定义测试用例，检查过滤功能是否按预期工作
{
    # 测试用例名称
    name: "filters only matching CVE and package",
    # 资源类型
    typ:  "java-archive",
    # 匹配的CVE和软件包
    matches: [
        {"CVE-2021-44228", "log4j-api"},
        {"CVE-2021-44228", "log4j-core"},
    ],
    # 预期结果：符合条件的软件包
    expected: ["log4j-core"],
    # 忽略的软件包
    ignored: ["log4j-api"],
},
{
    name: "filters all matching CVEs and packages",
    typ:  "java-archive",
    matches: [
        {"GHSA-jfh8-c2jp-5v3q", "log4j-api"},
        {"GHSA-jfh8-c2jp-5v3q", "log4j-slf4j-impl"},
    ],
		},
		// 期望结果为空字符串切片
		expected: []string{},
		// 忽略的包名为"log4j-api"和"log4j-slf4j-impl"
		ignored:  []string{"log4j-api", "log4j-slf4j-impl"},
	},
	// 过滤无效的 CVE 对于 protobuf Go 模块
	{
		name: "filters invalid CVEs for protobuf Go module",
		typ:  "go-module",
		// 匹配的 CVE 包含"google.golang.org/protobuf"的两个 CVE
		matches: []cvePkg{
			{"CVE-2015-5237", "google.golang.org/protobuf"},
			{"CVE-2021-22570", "google.golang.org/protobuf"},
		},
		// 期望结果为空字符串切片
		expected: []string{},
		// 忽略的包名为"google.golang.org/protobuf"和"google.golang.org/protobuf"
		ignored:  []string{"google.golang.org/protobuf", "google.golang.org/protobuf"},
	},
	// 保留有效的 CVE 对于 protobuf Go 模块
	{
		name: "keeps valid CVEs for protobuf Go module",
		typ:  "go-module",
		// 匹配的 CVE 包含"google.golang.org/protobuf"的一个 CVE
		matches: []cvePkg{
			{"CVE-1998-99999", "google.golang.org/protobuf"},
		},
		// 定义一个测试用例切片
		tests := []struct {
			name    string
			typ     pkg.Type
			matches []cp
		}{
			// 测试用例1
			{
				name: "test1",
				typ:  pkg.Type("test"),
				// 匹配项为空
				matches: []cp{},
			},
		}

		// 创建一个模拟的排除提供程序
		p := newMockExclusionProvider()

		// 遍历测试用例
		for _, test := range tests {
			// 在测试中创建匹配项
			matches := NewMatches()

			// 遍历匹配项
			for _, cp := range test.matches {
				// 添加匹配项
				matches.Add(Match{
					// 设置包信息
					Package: pkg.Package{
						ID:   pkg.ID(cp.pkg),
						Name: cp.pkg,
						Type: test.typ,
					},
					// 设置漏洞信息
					Vulnerability: vulnerability.Vulnerability{
						ID: cp.cve,
			},
		})
	}

	filtered, ignores := ApplyExplicitIgnoreRules(p, matches)
	// 应用显式忽略规则，返回过滤后的结果和被忽略的结果

	var found []string
	// 创建一个字符串数组用于存储找到的结果
	for match := range filtered.Enumerate() {
		// 遍历过滤后的结果，将匹配的包名添加到数组中
		found = append(found, match.Package.Name)
	}

	assert.ElementsMatch(t, test.expected, found)
	// 使用断言检查找到的结果是否符合预期

	if len(test.ignored) > 0 {
		// 如果预期有被忽略的结果
		var ignored []string
		// 创建一个字符串数组用于存储被忽略的结果
		for _, i := range ignores {
			// 遍历被忽略的结果，将包名添加到数组中
			ignored = append(ignored, i.Package.Name)
		}
		assert.ElementsMatch(t, test.ignored, ignored)
		// 使用断言检查被忽略的结果是否符合预期
	} else {
# 使用断言检查忽略列表是否为空
assert.Empty(t, ignores)
```