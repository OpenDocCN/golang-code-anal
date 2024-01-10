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
    tests := []struct {  // 定义测试用例结构体切片
        name     string  // 测试用例名称
        fixture  string  // 测试用例的数据文件路径
        excludes []string  // 排除的文件路径列表
        expected []string  // 期望的结果列表
    }{
        {
            name:     "exclude everything",  // 测试用例名称
            fixture:  "test-fixtures/syft-spring.json",  // 测试数据文件路径
            excludes: []string{"**"},  // 排除所有文件
            expected: []string{},  // 期望结果为空列表
        },
        {
            name:     "exclude specific real path match",  // 测试用例名称
            fixture:  "test-fixtures/syft-spring.json",  // 测试数据文件路径
            excludes: []string{"**/tomcat*.jar"},  // 排除特定文件路径
            expected: []string{"charsets"},  // 期望结果包含指定文件
        },
        {
            name:     "include everything with no match",  // 测试用例名称
            fixture:  "test-fixtures/syft-spring.json",  // 测试数据文件路径
            excludes: []string{"**/asdf*.jar"},  // 排除特定文件路径
            expected: []string{"charsets", "tomcat-embed-el"},  // 期望结果包含指定文件
        },
        {
            name:     "include everything with no excludes",  // 测试用例名称
            fixture:  "test-fixtures/syft-spring.json",  // 测试数据文件路径
            excludes: []string{},  // 不排除任何文件
            expected: []string{"charsets", "tomcat-embed-el"},  // 期望结果包含指定文件
        },
    }

    for _, test := range tests {  // 遍历测试用例
        t.Run(test.name, func(t *testing.T) {  // 运行测试用例
            cfg := ProviderConfig{  // 创建提供程序配置
                SyftProviderConfig: SyftProviderConfig{  // 创建 Syft 提供程序配置
                    Exclusions:        test.excludes,  // 设置排除列表
                    CatalogingOptions: cataloger.DefaultConfig(),  // 设置目录选项
                },
            }
            pkgs, _, _, _ := Provide(test.fixture, cfg)  // 调用 Provide 函数获取结果

            var pkgNames []string  // 创建包名称列表

            for _, pkg := range pkgs {  // 遍历包列表
                pkgNames = append(pkgNames, pkg.Name)  // 将包名称添加到列表中
            }

            assert.ElementsMatch(t, pkgNames, test.expected)  // 使用断言检查结果是否符合预期
        })
    }
}

func TestSyftLocationExcludes(t *testing.T) {
    # 定义测试用例的结构体数组，包括名称、固定值、排除项和期望结果
    tests := []struct {
        name     string
        fixture  string
        excludes []string
        expected []string
    }{
        # 第一个测试用例：排除所有内容
        {
            name:     "exclude everything",
            fixture:  "image-simple",
            excludes: []string{"**"},
            expected: []string{},
        },
        # 第二个测试用例：排除特定的真实路径匹配
        {
            name:     "exclude specific real path match",
            fixture:  "image-simple",
            excludes: []string{"**/nested/package.json"},
            expected: []string{"top-level-package"},
        },
        # 第三个测试用例：包含所有内容，没有匹配项
        {
            name:     "include everything with no match",
            fixture:  "image-simple",
            excludes: []string{"**/asdf*.json"},
            expected: []string{"nested-package", "top-level-package"},
        },
        # 第四个测试用例：包含所有内容，没有排除项
        {
            name:     "include everything with no excludes",
            fixture:  "image-simple",
            excludes: []string{},
            expected: []string{"nested-package", "top-level-package"},
        },
    }

    # 遍历测试用例数组
    for _, test := range tests {
        # 运行测试用例
        t.Run(test.name, func(t *testing.T) {
            # 获取测试用例的固定值图像路径
            userInput := imagetest.GetFixtureImageTarPath(t, test.fixture)
            # 配置提供者的配置
            cfg := ProviderConfig{
                SyftProviderConfig: SyftProviderConfig{
                    Exclusions:        test.excludes,
                    CatalogingOptions: cataloger.DefaultConfig(),
                },
            }
            # 调用 Provide 函数，获取软件包、图像摘要、图像配置和错误信息
            pkgs, _, _, err := Provide(userInput, cfg)

            # 断言没有错误发生
            assert.NoErrorf(t, err, "error calling Provide function")

            # 创建软件包名称的字符串数组
            var pkgNames []string

            # 遍历软件包数组，将软件包名称添加到 pkgNames 数组中
            for _, pkg := range pkgs {
                pkgNames = append(pkgNames, pkg.Name)
            }

            # 断言软件包名称数组和期望结果数组相匹配
            assert.ElementsMatch(t, pkgNames, test.expected)
        })
    }
# 测试函数，用于测试过滤包排除的功能
func Test_filterPackageExclusions(t *testing.T) {
    # 定义测试用例
    tests := []struct {
        name       string
        locations  [][]string
        exclusions []string
        expected   int
    }{
        {
            name:       "exclude nothing",
            locations:  [][]string{{"/foo", "/bar"}, {"/foo", "/bar"}},
            exclusions: []string{"/asdf/**"},
            expected:   2,
        },
        {
            name:       "exclude everything",
            locations:  [][]string{{"/foo", "/bar"}, {"/foo", "/bar"}},
            exclusions: []string{"**"},
            expected:   0,
        },
        {
            name:       "exclude based on all location match",
            locations:  [][]string{{"/foo1", "/bar1"}, {"/foo2", "/bar2"}},
            exclusions: []string{"/foo2", "/bar2"},
            expected:   1,
        },
        {
            name:       "don't exclude with single location match",
            locations:  [][]string{{"/foo1", "/bar1"}, {"/foo2", "/bar2"}},
            exclusions: []string{"/foo1", "/foo2"},
            expected:   2,
        },
    }

    # 遍历测试用例
    for _, test := range tests {
        # 运行单个测试用例
        t.Run(test.name, func(t *testing.T) {
            # 初始化包列表
            var packages []Package
            # 遍历测试用例中的位置
            for _, pkg := range test.locations {
                # 创建文件位置集合
                locations := file.NewLocationSet()
                # 遍历位置列表，添加虚拟位置
                for _, l := range pkg {
                    locations.Add(
                        file.NewVirtualLocation(l, l),
                    )
                }
                # 将包添加到包列表中
                packages = append(packages, Package{Locations: locations})
            }
            # 过滤包排除，并返回过滤后的结果和错误
            filtered, err := filterPackageExclusions(packages, test.exclusions)

            # 断言没有错误发生
            assert.NoError(t, err)
            # 断言过滤后的结果长度符合预期
            assert.Len(t, filtered, test.expected)
        })
    }
}

# 测试函数，用于测试位置匹配功能
func Test_matchesLocation(t *testing.T) {
    # 定义测试用例
    tests := []struct {
        name        string
        realPath    string
        virtualPath string
        match       string
        expected    bool
    # 测试用例列表，包含了不同情况下的测试数据
    {
        # 测试名称，表示不匹配真实路径
        name:        "doesn't match real",
        # 真实路径
        realPath:    "/asdf",
        # 虚拟路径
        virtualPath: "",
        # 匹配规则
        match:       "/usr",
        # 期望结果
        expected:    false,
    },
    {
        # 测试名称，表示不匹配虚拟路径
        name:        "doesn't match virtual",
        # 真实路径
        realPath:    "",
        # 虚拟路径
        virtualPath: "/asdf",
        # 匹配规则
        match:       "/usr",
        # 期望结果
        expected:    false,
    },
    {
        # 测试名称，表示匹配真实路径
        name:        "does match real",
        # 真实路径
        realPath:    "/usr/foo",
        # 虚拟路径
        virtualPath: "",
        # 匹配规则
        match:       "/usr/**",
        # 期望结果
        expected:    true,
    },
    {
        # 测试名称，表示匹配虚拟路径
        name:        "does match virtual",
        # 真实路径
        realPath:    "",
        # 虚拟路径
        virtualPath: "/usr/bar/oof.txt",
        # 匹配规则
        match:       "/usr/**",
        # 期望结果
        expected:    true,
    },
    {
        # 测试名称，表示匹配文件
        name:        "does match file",
        # 真实路径
        realPath:    "",
        # 虚拟路径
        virtualPath: "/usr/bar/oof.txt",
        # 匹配规则
        match:       "**/*.txt",
        # 期望结果
        expected:    true,
    },
}

# 遍历测试用例列表
for _, test := range tests:
    # 运行单个测试用例
    t.Run(test.name, func(t *testing.T) {
        # 调用函数进行匹配测试
        matches, err := locationMatches(file.NewVirtualLocation(test.realPath, test.virtualPath), test.match)
        # 断言，验证结果是否符合预期
        assert.NoError(t, err)
        assert.Equal(t, test.expected, matches)
    })
# 闭合前面的函数定义
```