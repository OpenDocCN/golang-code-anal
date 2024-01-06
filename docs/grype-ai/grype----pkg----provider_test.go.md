# `grype\grype\pkg\provider_test.go`

```
package pkg

import (
	"testing"  // 导入测试包
	"github.com/stretchr/testify/assert"  // 导入断言包

	"github.com/anchore/stereoscope/pkg/imagetest"  // 导入图像测试包
	"github.com/anchore/syft/syft/file"  // 导入文件包
	"github.com/anchore/syft/syft/pkg/cataloger"  // 导入目录包
)

func TestProviderLocationExcludes(t *testing.T) {
	tests := []struct {
		name     string  // 测试用例名称
		fixture  string  // 测试用例的数据文件
		excludes []string  // 排除的内容
		expected []string  // 期望的结果
	}{
		{  // 测试用例1
# 定义测试用例，包括名称、测试数据、排除规则和期望结果
{
    name:     "exclude everything",
    fixture:  "test-fixtures/syft-spring.json",
    excludes: []string{"**"},
    expected: []string{},
},
{
    name:     "exclude specific real path match",
    fixture:  "test-fixtures/syft-spring.json",
    excludes: []string{"**/tomcat*.jar"},
    expected: []string{"charsets"},
},
{
    name:     "include everything with no match",
    fixture:  "test-fixtures/syft-spring.json",
    excludes: []string{"**/asdf*.jar"},
    expected: []string{"charsets", "tomcat-embed-el"},
},
{
    name:     "include everything with no excludes",
    fixture:  "test-fixtures/syft-spring.json",
}
# 初始化测试数据
tests := []struct {
	name     string
	fixture  string
	excludes []string
	expected []string{"charsets", "tomcat-embed-el"},
}

# 遍历测试数据
for _, test := range tests {
	# 在测试中运行代码
	t.Run(test.name, func(t *testing.T) {
		# 配置提供程序的配置
		cfg := ProviderConfig{
			SyftProviderConfig: SyftProviderConfig{
				Exclusions:        test.excludes,  # 设置排除的文件/目录列表
				CatalogingOptions: cataloger.DefaultConfig(),  # 设置目录选项为默认配置
			},
		}
		# 调用 Provide 函数获取包信息
		pkgs, _, _, _ := Provide(test.fixture, cfg)

		# 初始化包名列表
		var pkgNames []string

		# 遍历获取的包信息，将包名添加到包名列表中
		for _, pkg := range pkgs {
			pkgNames = append(pkgNames, pkg.Name)
		}
	}
}
// 断言测试结果中的元素与期望结果中的元素匹配
assert.ElementsMatch(t, pkgNames, test.expected)
```

```go
// 定义测试函数 TestSyftLocationExcludes
func TestSyftLocationExcludes(t *testing.T) {
	// 定义测试用例
	tests := []struct {
		name     string
		fixture  string
		excludes []string
		expected []string
	}{
		{
			name:     "exclude everything",
			fixture:  "image-simple",
			excludes: []string{"**"},
			expected: []string{},
		},
		{
```
```go
// 第一个测试用例的注释
// 测试名称为 "exclude everything"，使用 fixture "image-simple"，排除所有内容，期望结果为空数组
# 定义测试用例，测试排除特定真实路径匹配的情况
name:     "exclude specific real path match",
fixture:  "image-simple",
excludes: []string{"**/nested/package.json"},
expected: []string{"top-level-package"},
},
# 定义测试用例，测试没有匹配的情况下包含所有内容
name:     "include everything with no match",
fixture:  "image-simple",
excludes: []string{"**/asdf*.json"},
expected: []string{"nested-package", "top-level-package"},
},
# 定义测试用例，测试没有排除任何内容时包含所有内容
name:     "include everything with no excludes",
fixture:  "image-simple",
excludes: []string{},
expected: []string{"nested-package", "top-level-package"},
}

# 遍历测试用例
for _, test := range tests {
# 对每个测试用例运行测试
t.Run(test.name, func(t *testing.T) {
    # 获取测试用例的图像文件路径作为用户输入
    userInput := imagetest.GetFixtureImageTarPath(t, test.fixture)
    # 创建提供程序配置
    cfg := ProviderConfig{
        SyftProviderConfig: SyftProviderConfig{
            # 设置排除项
            Exclusions:        test.excludes,
            # 设置目录选项为默认配置
            CatalogingOptions: cataloger.DefaultConfig(),
        },
    }
    # 调用 Provide 函数提供图像文件的软件包信息
    pkgs, _, _, err := Provide(userInput, cfg)

    # 断言是否没有错误发生
    assert.NoErrorf(t, err, "error calling Provide function")

    # 创建一个空的软件包名称列表
    var pkgNames []string

    # 遍历提供的软件包列表，将软件包名称添加到列表中
    for _, pkg := range pkgs {
        pkgNames = append(pkgNames, pkg.Name)
    }

    # 断言提供的软件包名称列表是否与预期的列表匹配
    assert.ElementsMatch(t, pkgNames, test.expected)
})
	}
}

func Test_filterPackageExclusions(t *testing.T) {
	tests := []struct {
		name       string
		locations  [][]string
		exclusions []string
		expected   int
	}{
		// 定义测试用例1：不排除任何内容
		{
			name:       "exclude nothing",
			locations:  [][]string{{"/foo", "/bar"}, {"/foo", "/bar"}},
			exclusions: []string{"/asdf/**"},
			expected:   2,
		},
		// 定义测试用例2：排除所有内容
		{
			name:       "exclude everything",
			locations:  [][]string{{"/foo", "/bar"}, {"/foo", "/bar"}},
			exclusions: []string{"**"},
# 定义测试用例
tests := []struct {
    name       string   // 测试用例名称
    locations  [][]string   // 文件位置列表
    exclusions []string   // 需要排除的文件位置
    expected   int   // 期望的结果
}{
    {
        name:       "exclude based on single location match",   // 测试用例名称
        locations:  [][]string{{"/foo1", "/bar1"}, {"/foo2", "/bar2"}},   // 文件位置列表
        exclusions: []string{"/foo1", "/bar1"},   // 需要排除的文件位置
        expected:   0,   // 期望的结果
    },
    {
        name:       "exclude based on all location match",   // 测试用例名称
        locations:  [][]string{{"/foo1", "/bar1"}, {"/foo2", "/bar2"}},   // 文件位置列表
        exclusions: []string{"/foo2", "/bar2"},   // 需要排除的文件位置
        expected:   1,   // 期望的结果
    },
    {
        name:       "don't exclude with single location match",   // 测试用例名称
        locations:  [][]string{{"/foo1", "/bar1"}, {"/foo2", "/bar2"}},   // 文件位置列表
        exclusions: []string{"/foo1", "/foo2"},   // 需要排除的文件位置
        expected:   2,   // 期望的结果
    },
}

# 遍历测试用例
for _, test := range tests {
    # 运行测试用例
    t.Run(test.name, func(t *testing.T) {
        var packages []Package
        # 遍历文件位置列表
        for _, pkg := range test.locations {
# 创建一个文件位置集合
locations := file.NewLocationSet()

# 遍历 pkg 切片中的每个元素
for _, l := range pkg {
    # 将虚拟位置对象添加到位置集合中
    locations.Add(
        file.NewVirtualLocation(l, l),
    )
}

# 将包含位置集合的 Package 对象添加到 packages 切片中
packages = append(packages, Package{Locations: locations})
```

```
# 使用 filterPackageExclusions 函数过滤掉指定的包，并返回过滤后的结果
filtered, err := filterPackageExclusions(packages, test.exclusions)

# 断言过滤操作是否出错
assert.NoError(t, err)

# 断言过滤后的结果长度是否符合预期
assert.Len(t, filtered, test.expected)
```

```
# 定义测试函数 Test_matchesLocation
func Test_matchesLocation(t *testing.T) {
    # 定义测试用例切片
    tests := []struct {
        name        string
        realPath    string
        ...
    }{
        ...
    }
}
		// 定义虚拟路径
		virtualPath string
		// 定义匹配字符串
		match       string
		// 定义预期结果
		expected    bool
	}{
		// 第一个测试用例：路径不匹配实际路径
		{
			name:        "doesn't match real",
			realPath:    "/asdf",
			virtualPath: "",
			match:       "/usr",
			expected:    false,
		},
		// 第二个测试用例：路径不匹配虚拟路径
		{
			name:        "doesn't match virtual",
			realPath:    "",
			virtualPath: "/asdf",
			match:       "/usr",
			expected:    false,
		},
		// 第三个测试用例：路径匹配实际路径
# 定义一个测试用例数组，每个测试用例包含以下字段：realPath（真实路径）、virtualPath（虚拟路径）、match（匹配规则）、expected（预期结果）
[
    {
        name:        "does match real",
        realPath:    "/usr/foo",
        virtualPath: "",
        match:       "/usr/**",
        expected:    true,
    },
    {
        name:        "does match virtual",
        realPath:    "",
        virtualPath: "/usr/bar/oof.txt",
        match:       "/usr/**",
        expected:    true,
    },
    {
        name:        "does match file",
        realPath:    "",
        virtualPath: "/usr/bar/oof.txt",
        match:       "**/*.txt",
        expected:    true,
    },
]
# 遍历测试用例列表
for _, test := range tests:
    # 对每个测试用例运行子测试
    t.Run(test.name, func(t *testing.T):
        # 调用 locationMatches 函数，检查虚拟路径和匹配模式是否匹配
        matches, err := locationMatches(file.NewVirtualLocation(test.realPath, test.virtualPath), test.match)
        # 断言错误为空
        assert.NoError(t, err)
        # 断言实际匹配结果与预期匹配结果相等
        assert.Equal(t, test.expected, matches)
    )
```