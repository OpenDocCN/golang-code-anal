# `grype\grype\search\language_test.go`

```
package search

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"testing"  // 导入 testing 包，用于编写测试函数

	"github.com/google/uuid"  // 导入 uuid 包，用于生成和解析 UUID
	"github.com/stretchr/testify/assert"  // 导入 testify 包，用于编写断言语句

	"github.com/anchore/grype/grype/match"  // 导入 match 包，用于匹配
	"github.com/anchore/grype/grype/pkg"  // 导入 pkg 包，用于处理软件包
	"github.com/anchore/grype/grype/version"  // 导入 version 包，用于处理版本信息
	"github.com/anchore/grype/grype/vulnerability"  // 导入 vulnerability 包，用于处理漏洞信息
	syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包，用于处理软件包信息
)

type mockLanguageProvider struct {
	data map[string]map[string][]vulnerability.Vulnerability  // 定义一个结构体，包含一个 map 类型的字段
}
// 创建一个新的模拟语言提供者对象
func newMockProviderByLanguage() *mockLanguageProvider {
	// 初始化一个空的数据字典
	pr := mockLanguageProvider{
		data: make(map[string]map[string][]vulnerability.Vulnerability),
	}
	// 调用 stub 方法填充数据
	pr.stub()
	// 返回模拟语言提供者对象的指针
	return &pr
}

// 模拟语言提供者的填充数据方法
func (pr *mockLanguageProvider) stub() {
	// 填充数据字典，以 github:gem 为键，值为一个包含漏洞信息的字典
	pr.data["github:gem"] = map[string][]vulnerability.Vulnerability{
		// 直接依赖的漏洞信息
		"activerecord": {
			{
				// 确保我们可以通过 semVer 约束找到它
				Constraint: version.MustGetConstraint("< 3.7.6", version.SemanticFormat),
				ID:         "CVE-2017-fake-1",
				Namespace:  "github:ruby",
			},
			{
				Constraint: version.MustGetConstraint("< 3.7.4", version.GemFormat),
// 创建一个名为nokogiri的map，其中包含两个元素
"nokogiri": {
    // 第一个元素的约束条件为小于1.7.6的gem版本，ID为"CVE-2017-fake-1"，命名空间为"github:ruby"
    {
        Constraint: version.MustGetConstraint("< 1.7.6", version.GemFormat),
        ID:         "CVE-2017-fake-1",
        Namespace:  "github:ruby",
    },
    // 第二个元素的约束条件为小于1.7.4的语义版本格式，ID为"CVE-2017-fake-2"，命名空间为"github:ruby"
    {
        Constraint: version.MustGetConstraint("< 1.7.4", version.SemanticFormat),
        ID:         "CVE-2017-fake-2",
        Namespace:  "github:ruby",
    },
}
```
这段代码创建了一个名为nokogiri的map，其中包含两个元素，每个元素都包含约束条件、ID和命名空间。
// 根据语言和包名获取对应的漏洞信息
func (pr *mockLanguageProvider) GetByLanguage(l syftPkg.Language, p pkg.Package) ([]vulnerability.Vulnerability, error) {
	// 如果语言不是Ruby，则抛出错误
	if l != syftPkg.Ruby {
		panic(fmt.Errorf("test mock only supports ruby"))
	}
	// 返回对应包名的漏洞信息
	return pr.data["github:gem"][p.Name], nil
}

// 期望匹配的漏洞信息
func expectedMatch(p pkg.Package, constraint string) []match.Match {
	// 返回匹配的漏洞信息
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
// 设置命名空间为github:ruby，包含name和version两个属性的map
"namespace": "github:ruby",
"package":   map[string]string{"name": p.Name, "version": p.Version},

// 设置Found字段，包含versionConstraint和vulnerabilityID两个属性
Found: map[string]interface{}{
	"versionConstraint": constraint,
	"vulnerabilityID":   "CVE-2017-fake-1",
},

// 设置Matcher字段为RubyGemMatcher
Matcher: match.RubyGemMatcher,

// 定义测试函数TestFindMatchesByPackageLanguage
func TestFindMatchesByPackageLanguage(t *testing.T) {
	cases := []struct {
		p          pkg.Package
		constraint string
	}{
		{
# 定义约束条件为小于 3.7.6 的语义版本号
constraint: "< 3.7.6 (semver)",
# 定义包对象，包括 ID、名称、版本、语言和类型
p: pkg.Package{
    ID:       pkg.ID(uuid.NewString()),  # 生成新的唯一标识符作为包的 ID
    Name:     "activerecord",  # 包的名称为 activerecord
    Version:  "3.7.5",  # 包的版本号为 3.7.5
    Language: syftPkg.Ruby,  # 包的语言为 Ruby
    Type:     syftPkg.GemPkg,  # 包的类型为 GemPkg
},
# 定义约束条件为小于 1.7.6 的语义版本号
constraint: "< 1.7.6 (semver)",
# 定义包对象，包括 ID、名称、版本、语言和类型
p: pkg.Package{
    ID:       pkg.ID(uuid.NewString()),  # 生成新的唯一标识符作为包的 ID
    Name:     "nokogiri",  # 包的名称为 nokogiri
    Version:  "1.7.5",  # 包的版本号为 1.7.5
    Language: syftPkg.Ruby,  # 包的语言为 Ruby
    Type:     syftPkg.GemPkg,  # 包的类型为 GemPkg
},
# 创建一个新的模拟语言提供程序
store := newMockProviderByLanguage()

# 遍历测试用例
for _, c := range cases {
    # 对每个测试用例运行子测试
    t.Run(c.p.Name, func(t *testing.T) {
        # 调用 ByPackageLanguage 函数，获取实际结果和错误
        actual, err := ByPackageLanguage(store, nil, c.p, match.RubyGemMatcher)
        # 断言错误为空
        assert.NoError(t, err)
        # 使用漏洞的 ID 匹配断言实际结果
        assertMatchesUsingIDsForVulnerabilities(t, expectedMatch(c.p, c.constraint), actual)
    })
}
```