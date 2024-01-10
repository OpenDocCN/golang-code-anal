# `grype\grype\search\language_test.go`

```
package search

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/google/uuid"  // 导入 uuid 包，用于生成唯一标识符
    "github.com/stretchr/testify/assert"  // 导入 assert 包，用于编写断言语句

    "github.com/anchore/grype/grype/match"  // 导入 match 包，用于匹配
    "github.com/anchore/grype/grype/pkg"  // 导入 pkg 包，用于处理包信息
    "github.com/anchore/grype/grype/version"  // 导入 version 包，用于处理版本信息
    "github.com/anchore/grype/grype/vulnerability"  // 导入 vulnerability 包，用于处理漏洞信息
    syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包，用于处理包信息
)

type mockLanguageProvider struct {
    data map[string]map[string][]vulnerability.Vulnerability  // 定义一个结构体 mockLanguageProvider，包含一个数据字段
}

func newMockProviderByLanguage() *mockLanguageProvider {
    pr := mockLanguageProvider{  // 创建 mockLanguageProvider 结构体实例
        data: make(map[string]map[string][]vulnerability.Vulnerability),  // 初始化 data 字段
    }
    pr.stub()  // 调用 stub 方法
    return &pr  // 返回结构体实例的指针
}

func (pr *mockLanguageProvider) stub() {
    pr.data["github:gem"] = map[string][]vulnerability.Vulnerability{  // 设置 data 字段的值
        // direct...
        "activerecord": {  // 设置 activerecord 的值
            {
                // make sure we find it with semVer constraint
                Constraint: version.MustGetConstraint("< 3.7.6", version.SemanticFormat),  // 设置 Constraint 字段的值
                ID:         "CVE-2017-fake-1",  // 设置 ID 字段的值
                Namespace:  "github:ruby",  // 设置 Namespace 字段的值
            },
            {
                Constraint: version.MustGetConstraint("< 3.7.4", version.GemFormat),  // 设置 Constraint 字段的值
                ID:         "CVE-2017-fake-2",  // 设置 ID 字段的值
                Namespace:  "github:ruby",  // 设置 Namespace 字段的值
            },
        },
        "nokogiri": {  // 设置 nokogiri 的值
            {
                // make sure we find it with gem version constraint
                Constraint: version.MustGetConstraint("< 1.7.6", version.GemFormat),  // 设置 Constraint 字段的值
                ID:         "CVE-2017-fake-1",  // 设置 ID 字段的值
                Namespace:  "github:ruby",  // 设置 Namespace 字段的值
            },
            {
                Constraint: version.MustGetConstraint("< 1.7.4", version.SemanticFormat),  // 设置 Constraint 字段的值
                ID:         "CVE-2017-fake-2",  // 设置 ID 字段的值
                Namespace:  "github:ruby",  // 设置 Namespace 字段的值
            },
        },
    }
}

func (pr *mockLanguageProvider) GetByLanguage(l syftPkg.Language, p pkg.Package) ([]vulnerability.Vulnerability, error) {
    # 如果语言不是 Ruby，则触发 panic，抛出错误
    if l != syftPkg.Ruby:
        panic(fmt.Errorf("test mock only supports ruby"))
    # 返回指定包的数据和空值
    return pr.data["github:gem"][p.Name], nil
}

// expectedMatch 函数返回一个匹配列表，包含了给定包和约束条件的匹配信息
func expectedMatch(p pkg.Package, constraint string) []match.Match {
    return []match.Match{
        {
            Vulnerability: vulnerability.Vulnerability{
                ID: "CVE-2017-fake-1",
            },
            Package: p,
            Details: []match.Detail{
                {
                    Type:       match.ExactDirectMatch,
                    Confidence: 1,
                    SearchedBy: map[string]interface{}{
                        "language":  "ruby",
                        "namespace": "github:ruby",
                        "package":   map[string]string{"name": p.Name, "version": p.Version},
                    },
                    Found: map[string]interface{}{
                        "versionConstraint": constraint,
                        "vulnerabilityID":   "CVE-2017-fake-1",
                    },
                    Matcher: match.RubyGemMatcher,
                },
            },
        },
    }
}

// TestFindMatchesByPackageLanguage 函数用于测试按包语言查找匹配项
func TestFindMatchesByPackageLanguage(t *testing.T) {
    cases := []struct {
        p          pkg.Package
        constraint string
    }{
        {
            constraint: "< 3.7.6 (semver)",
            p: pkg.Package{
                ID:       pkg.ID(uuid.NewString()),
                Name:     "activerecord",
                Version:  "3.7.5",
                Language: syftPkg.Ruby,
                Type:     syftPkg.GemPkg,
            },
        },
        {
            constraint: "< 1.7.6 (semver)",
            p: pkg.Package{
                ID:       pkg.ID(uuid.NewString()),
                Name:     "nokogiri",
                Version:  "1.7.5",
                Language: syftPkg.Ruby,
                Type:     syftPkg.GemPkg,
            },
        },
    }

    // 创建一个按语言分类的模拟提供程序
    store := newMockProviderByLanguage()
    # 遍历 cases 数组，使用下划线 _ 忽略索引，c 为当前元素
    for _, c := range cases:
        # 使用测试框架运行测试，测试名称为 c.p.Name
        t.Run(c.p.Name, func(t *testing.T):
            # 调用 ByPackageLanguage 函数，传入 store、nil、c.p、match.RubyGemMatcher 参数，获取 actual 和 err
            actual, err := ByPackageLanguage(store, nil, c.p, match.RubyGemMatcher)
            # 使用断言检查错误是否为空
            assert.NoError(t, err)
            # 使用自定义函数 assertMatchesUsingIDsForVulnerabilities 检查 actual 是否符合预期
            assertMatchesUsingIDsForVulnerabilities(t, expectedMatch(c.p, c.constraint), actual)
        )
    )
# 闭合前面的函数定义
```