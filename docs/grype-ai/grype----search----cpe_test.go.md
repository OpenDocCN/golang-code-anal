# `grype\grype\search\cpe_test.go`

```
// 导入测试包
import (
	"testing"

	// 导入用于比较的包
	"github.com/google/go-cmp/cmp"
	// 导入用于生成 UUID 的包
	"github.com/google/uuid"
	// 导入断言包
	"github.com/stretchr/testify/assert"
	// 导入断言包的 require 函数
	"github.com/stretchr/testify/require"

	// 导入 grype 的数据库包
	"github.com/anchore/grype/grype/db"
	// 导入 grype v5 版本的数据库包
	grypeDB "github.com/anchore/grype/grype/db/v5"
	// 导入 grype 的匹配包
	"github.com/anchore/grype/grype/match"
	// 导入 grype 的包信息包
	"github.com/anchore/grype/grype/pkg"
	// 导入 grype 的版本包
	"github.com/anchore/grype/grype/version"
	// 导入 grype 的漏洞包
	"github.com/anchore/grype/grype/vulnerability"
	// 导入 syft 的 CPE 包
	"github.com/anchore/syft/syft/cpe"
	// 导入 syft 的包信息包
	syftPkg "github.com/anchore/syft/syft/pkg"
)
// 定义一个实现VulnerabilityStoreReader接口的mockVulnStore结构体
var _ grypeDB.VulnerabilityStoreReader = (*mockVulnStore)(nil)

type mockVulnStore struct {
    data map[string]map[string][]grypeDB.Vulnerability
}

// 实现VulnerabilityStoreReader接口的GetVulnerability方法
func (pr *mockVulnStore) GetVulnerability(namespace, id string) ([]grypeDB.Vulnerability, error) {
    //TODO implement me
    // 抛出一个错误，表示该方法需要被实现
    panic("implement me")
}

// 创建一个新的mockVulnStore对象
func newMockStore() *mockVulnStore {
    // 初始化mockVulnStore对象，并为其data字段分配内存
    pr := mockVulnStore{
        data: make(map[string]map[string][]grypeDB.Vulnerability),
    }
    // 调用stub方法为mockVulnStore对象添加数据
    pr.stub()
    // 返回mockVulnStore对象的指针
    return &pr
}

// 为mockVulnStore对象添加数据的方法
func (pr *mockVulnStore) stub() {
    # 将漏洞信息添加到 pr.data 字典中的 "nvd:cpe" 键下，使用 map 类型存储
    pr.data["nvd:cpe"] = map[string][]grypeDB.Vulnerability{
        # 为 "activerecord" 包添加漏洞信息
        "activerecord": {
            {
                # 漏洞所属包的名称
                PackageName:       "activerecord",
                # 版本约束条件
                VersionConstraint: "< 3.7.6",
                # 版本格式
                VersionFormat:     version.SemanticFormat.String(),
                # 漏洞ID
                ID:                "CVE-2017-fake-1",
                # 相关的 CPE（通用产品和版本标识符）列表
                CPEs: []string{
                    "cpe:2.3:*:activerecord:activerecord:*:*:*:*:*:rails:*:*",
                },
                # 命名空间
                Namespace: "nvd:cpe",
            },
            {
                # 漏洞所属包的名称
                PackageName:       "activerecord",
                # 版本约束条件
                VersionConstraint: "< 3.7.4",
                # 版本格式
                VersionFormat:     version.SemanticFormat.String(),
                # 漏洞ID
                ID:                "CVE-2017-fake-2",
                # 相关的 CPE（通用产品和版本标识符）列表
                CPEs: []string{
                    "cpe:2.3:*:activerecord:activerecord:*:*:*:*:*:ruby:*:*",
                },
# 定义了一个名为 "nvd:cpe" 的命名空间下的一组数据
{
    PackageName:       "activerecord",  # 包名为 activerecord
    VersionConstraint: "= 4.0.1",  # 版本约束为 4.0.1
    VersionFormat:     version.GemFormat.String(),  # 版本格式为 GemFormat
    ID:                "CVE-2017-fake-3",  # ID 为 CVE-2017-fake-3
    CPEs: []string{  # CPEs 列表
        "cpe:2.3:*:activerecord:activerecord:4.0.1:*:*:*:*:*:*:*",  # CPE 格式的字符串
    },
    Namespace: "nvd:cpe",  # 命名空间为 "nvd:cpe"
},
# 定义了一个名为 "awesome" 的命名空间下的一组数据
{
    PackageName:       "awesome",  # 包名为 awesome
    VersionConstraint: "< 98SP3",  # 版本约束为小于 98SP3
    VersionFormat:     version.UnknownFormat.String(),  # 版本格式为 UnknownFormat
    ID:                "CVE-2017-fake-4",  # ID 为 CVE-2017-fake-4
    CPEs: []string{  # CPEs 列表
# 定义一个包含漏洞信息的数据结构
{
    PackageName:       "awesome",  # 漏洞所属的软件包名称
    VersionConstraint: "< 4.0",    # 软件包的版本约束
    VersionFormat:     version.UnknownFormat.String(),  # 版本格式
    ID:                "CVE-2017-fake-5",  # 漏洞的唯一标识符
    CPEs: [  # 与漏洞相关的 CPE（通用平台标识符）列表
        "cpe:2.3:*:awesome:awesome:*:*:*:*:*:*:*:*",
    ],
    Namespace: "nvd:cpe",  # 命名空间
},
# 定义另一个包含漏洞信息的数据结构
{
    PackageName:       "multiple",  # 漏洞所属的软件包名称
    VersionConstraint: "< 4.0",    # 软件包的版本约束
    VersionFormat:     version.UnknownFormat.String(),  # 版本格式
    ID:                "CVE-2017-fake-5",  # 漏洞的唯一标识符
    CPEs: [  # 与漏洞相关的 CPE（通用平台标识符）列表
        "cpe:2.3:*:multiple:multiple:*:*:*:*:*:*:*:*",
        "cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*",
        "cpe:2.3:*:multiple:multiple:2.0:*:*:*:*:*:*:*",
        "cpe:2.3:*:multiple:multiple:3.0:*:*:*:*:*:*:*",
    ],
    Namespace: "nvd:cpe",  # 命名空间
},
# 定义一个名为"funfun"的字典，包含软件包名称、版本约束、版本格式、ID、CPEs和命名空间等信息
"funfun": {
    {
        PackageName:       "funfun",
        VersionConstraint: "= 5.2.1",
        VersionFormat:     version.UnknownFormat.String(),
        ID:                "CVE-2017-fake-6",
        CPEs: []string{
            "cpe:2.3:*:funfun:funfun:5.2.1:*:*:*:*:python:*:*",
            "cpe:2.3:*:funfun:funfun:*:*:*:*:*:python:*:*",
        },
        Namespace: "nvd:cpe",
    },
},

# 定义一个名为"sw"的字典，包含软件包名称、版本约束、版本格式、ID和CPEs等信息
"sw": {
    {
        PackageName:       "sw",
        VersionConstraint: "< 1.0",
        VersionFormat:     version.UnknownFormat.String(),
        ID:                "CVE-2017-fake-7",
        CPEs: []string{
# 定义一个名为 "sw" 的软件包，版本号为任意，使用通配符 "*" 表示
"cpe:2.3:*:sw:sw:*:*:*:*:*:puppet:*:*",

# 定义一个名为 "handlebars" 的软件包，版本号小于 4.7.7，版本格式未知
"handlebars": {
    PackageName:       "handlebars",
    VersionConstraint: "< 4.7.7",
    VersionFormat:     version.UnknownFormat.String(),
    ID:                "CVE-2021-23369",
    CPEs: []string{
        # 定义一个名为 "handlebars" 的软件包，版本号为任意，使用通配符 "*" 表示
        "cpe:2.3:a:handlebarsjs:handlebars:*:*:*:*:*:node.js:*:*",
    },
    Namespace: "nvd:cpe",
},
// SearchForVulnerabilities 根据命名空间和包名搜索漏洞
func (pr *mockVulnStore) SearchForVulnerabilities(namespace, pkg string) ([]grypeDB.Vulnerability, error) {
	// 返回指定命名空间和包名的漏洞数据
	return pr.data[namespace][pkg], nil
}

// GetAllVulnerabilities 获取所有漏洞
func (pr *mockVulnStore) GetAllVulnerabilities() (*[]grypeDB.Vulnerability, error) {
	// 返回所有漏洞数据
	return nil, nil
}

// GetVulnerabilityNamespaces 获取漏洞命名空间
func (pr *mockVulnStore) GetVulnerabilityNamespaces() ([]string, error) {
	// 获取所有命名空间的键值
	keys := make([]string, 0, len(pr.data))
	for k := range pr.data {
		keys = append(keys, k)
	}

	return keys, nil
}

// TestFindMatchesByPackageCPE 根据包的CPE测试匹配
func TestFindMatchesByPackageCPE(t *testing.T) {
	// 使用RubyGemMatcher进行包匹配
	matcher := match.RubyGemMatcher
	tests := []struct {
		name     string  // 定义一个字符串类型的变量name
		p        pkg.Package  // 定义一个pkg.Package类型的变量p
		expected []match.Match  // 定义一个match.Match类型的切片变量expected
	}{
		{
			name: "match from range",  // 设置name字段的值为"match from range"
			p: pkg.Package{  // 设置p字段的值为pkg.Package结构体的实例
				CPEs: []cpe.CPE{  // 设置CPEs字段的值为cpe.CPE类型的切片
					cpe.Must("cpe:2.3:*:activerecord:activerecord:3.7.5:rando1:*:ra:*:ruby:*:*"),  // 设置切片中的第一个元素
					cpe.Must("cpe:2.3:*:activerecord:activerecord:3.7.5:rando4:*:re:*:rails:*:*"),  // 设置切片中的第二个元素
				},
				Name:     "activerecord",  // 设置Name字段的值为"activerecord"
				Version:  "3.7.5",  // 设置Version字段的值为"3.7.5"
				Language: syftPkg.Ruby,  // 设置Language字段的值为syftPkg.Ruby
				Type:     syftPkg.GemPkg,  // 设置Type字段的值为syftPkg.GemPkg
			},
			expected: []match.Match{  // 设置expected字段的值为match.Match类型的切片
				{  // 切片中的第一个元素
					Vulnerability: vulnerability.Vulnerability{  // 设置Vulnerability字段的值为vulnerability.Vulnerability结构体的实例
# 定义一个名为 "CVE-2017-fake-1" 的漏洞
ID: "CVE-2017-fake-1",
},
Package: pkg.Package{
    # 定义包的CPE（通用平台表达式）列表
    CPEs: []cpe.CPE{
        cpe.Must("cpe:2.3:*:activerecord:activerecord:3.7.5:rando1:*:ra:*:ruby:*:*"),
        cpe.Must("cpe:2.3:*:activerecord:activerecord:3.7.5:rando4:*:re:*:rails:*:*"),
    },
    # 定义包的名称
    Name:     "activerecord",
    # 定义包的版本
    Version:  "3.7.5",
    # 定义包的语言
    Language: syftPkg.Ruby,
    # 定义包的类型
    Type:     syftPkg.GemPkg,
},
Details: []match.Detail{
    {
        # 定义匹配类型为CPE匹配
        Type:       match.CPEMatch,
        # 定义匹配的置信度
        Confidence: 0.9,
        # 定义通过CPE参数搜索到的信息
        SearchedBy: CPEParameters{
            Namespace: "nvd:cpe",
            # 定义用于搜索的CPE列表
            CPEs:      []string{"cpe:2.3:*:activerecord:activerecord:3.7.5:rando4:*:re:*:rails:*:*"},
            # 定义用于搜索的包参数
            Package: CPEPackageParameter{
# 定义一个名为 "activerecord" 的包，版本为 "3.7.5"
Name:    "activerecord",
Version: "3.7.5",
# 定义一个名为 "activerecord" 的 CPE（通用平台标识符）结果
Found: CPEResult{
    # 包含的 CPE 列表
    CPEs:              []string{"cpe:2.3:*:activerecord:activerecord:*:*:*:*:*:rails:*:*"},
    # 版本约束为小于 3.7.6 的语义版本
    VersionConstraint: "< 3.7.6 (semver)",
    # 漏洞 ID 为 "CVE-2017-fake-1"
    VulnerabilityID:   "CVE-2017-fake-1",
},
# 匹配器为 matcher
Matcher: matcher,
# 定义一个名为 "multiple matches" 的包
{
    name: "multiple matches",
    # 包含的 CPE 列表
    p: pkg.Package{
        CPEs: []cpe.CPE{
            # 定义一个 CPE 为 "cpe:2.3:*:activerecord:activerecord:3.7.3:rando1:*:ra:*:ruby:*:*"
            cpe.Must("cpe:2.3:*:activerecord:activerecord:3.7.3:rando1:*:ra:*:ruby:*:*"),
# 创建一个包含漏洞信息的结构体
Vulnerability: vulnerability.Vulnerability{
    # 指定漏洞的ID
    ID: "CVE-2017-fake-1",
},
# 创建一个包含软件包信息的结构体
Package: pkg.Package{
    # 指定软件包的CPEs（通用产品和版本标识符）
    CPEs: []cpe.CPE{
        # 创建一个CPE对象，指定软件包的通用产品和版本标识符
        cpe.Must("cpe:2.3:*:activerecord:activerecord:3.7.3:rando1:*:ra:*:ruby:*:*"),
        cpe.Must("cpe:2.3:*:activerecord:activerecord:3.7.3:rando4:*:re:*:rails:*:*"),
    },
    # 指定软件包的名称
    Name:     "activerecord",
    # 指定软件包的版本
    Version:  "3.7.3",
```

# 定义了一个语言为Ruby的包，并指定了类型为GemPkg
Language: syftPkg.Ruby,
Type:     syftPkg.GemPkg,

# 包的详细信息，包括匹配类型、置信度、搜索参数和找到的结果
Details: []match.Detail{
    {
        Type:       match.CPEMatch,
        Confidence: 0.9,
        SearchedBy: CPEParameters{
            CPEs: []string{
                "cpe:2.3:*:activerecord:activerecord:3.7.3:rando4:*:re:*:rails:*:*",
            },
            Namespace: "nvd:cpe",
            Package: CPEPackageParameter{
                Name:    "activerecord",
                Version: "3.7.3",
            },
        },
        Found: CPEResult{
            CPEs:              []string{"cpe:2.3:*:activerecord:activerecord:*:*:*:*:*:rails:*:*"},
# 设置版本约束为小于 3.7.6 的语义版本
VersionConstraint: "< 3.7.6 (semver)",
# 设置漏洞 ID
VulnerabilityID:   "CVE-2017-fake-1",
# 设置匹配器
Matcher: matcher,
# 设置漏洞信息
Vulnerability: vulnerability.Vulnerability{
    ID: "CVE-2017-fake-2",
},
# 设置包信息
Package: pkg.Package{
    # 设置包的 CPE
    CPEs: []cpe.CPE{
        cpe.Must("cpe:2.3:*:activerecord:activerecord:3.7.3:rando1:*:ra:*:ruby:*:*"),
        cpe.Must("cpe:2.3:*:activerecord:activerecord:3.7.3:rando4:*:re:*:rails:*:*"),
    },
    # 设置包的名称
    Name:     "activerecord",
    # 设置包的版本
    Version:  "3.7.3",
    # 设置包的语言
    Language: syftPkg.Ruby,
# 定义一个类型为 syftPkg.GemPkg 的结构体
Type:     syftPkg.GemPkg,

# 定义一个包含匹配细节的数组
Details: []match.Detail{
    {
        # 指定匹配类型为 CPEMatch
        Type:       match.CPEMatch,
        # 设置匹配的置信度为 0.9
        Confidence: 0.9,
        # 搜索参数为 CPEParameters 结构体
        SearchedBy: CPEParameters{
            # 指定搜索的 CPEs
            CPEs:      []string{"cpe:2.3:*:activerecord:activerecord:3.7.3:rando1:*:ra:*:ruby:*:*"},
            # 指定命名空间为 "nvd:cpe"
            Namespace: "nvd:cpe",
            # 指定包参数为 CPEPackageParameter 结构体
            Package: CPEPackageParameter{
                # 指定包名为 "activerecord"
                Name:    "activerecord",
                # 指定版本为 "3.7.3"
                Version: "3.7.3",
            },
        },
        # 匹配结果为 CPEResult 结构体
        Found: CPEResult{
            # 匹配到的 CPEs
            CPEs:              []string{"cpe:2.3:*:activerecord:activerecord:*:*:*:*:*:ruby:*:*"},
            # 指定版本约束为 "< 3.7.4 (semver)"
            VersionConstraint: "< 3.7.4 (semver)",
            # 指定漏洞 ID 为 "CVE-2017-fake-2"
            VulnerabilityID:   "CVE-2017-fake-2",
        },
    },
# 创建一个名为Matcher的变量，并将其赋值为matcher
Matcher: matcher,
# 创建一个名为p的变量，并将其赋值为一个包含特定属性的Package对象
p: pkg.Package{
    # 将CPEs属性赋值为一个包含特定属性的CPE对象的数组
    CPEs: []cpe.CPE{
        # 使用cpe.Must方法创建一个特定的CPE对象并添加到数组中
        cpe.Must("cpe:2.3:*:*:activerecord:4.0.1:*:*:*:*:*:*:*"),
    },
    # 设置Name属性为"activerecord"
    Name:     "activerecord",
    # 设置Version属性为"4.0.1"
    Version:  "4.0.1",
    # 设置Language属性为syftPkg.Ruby
    Language: syftPkg.Ruby,
    # 设置Type属性为syftPkg.GemPkg
    Type:     syftPkg.GemPkg,
},
# 设置name属性为"exact match"
name: "exact match",
# 设置expected属性为一个包含特定属性的Match对象的数组
expected: []match.Match{
    {
# 创建一个漏洞对象，包括漏洞ID
Vulnerability: vulnerability.Vulnerability{
    ID: "CVE-2017-fake-3",
},
# 创建一个包对象，包括CPE（通用平台表达式）列表、名称、版本、语言和类型
Package: pkg.Package{
    CPEs: []cpe.CPE{
        cpe.Must("cpe:2.3:*:*:activerecord:4.0.1:*:*:*:*:*:*:*"),
    },
    Name:     "activerecord",
    Version:  "4.0.1",
    Language: syftPkg.Ruby,
    Type:     syftPkg.GemPkg,
},
# 创建一个匹配详情列表，包括匹配类型、置信度和搜索参数
Details: []match.Detail{
    {
        Type:       match.CPEMatch,
        Confidence: 0.9,
        SearchedBy: CPEParameters{
            CPEs:      []string{"cpe:2.3:*:*:activerecord:4.0.1:*:*:*:*:*:*:*"},
            Namespace: "nvd:cpe",
            Package: CPEPackageParameter{
# 定义一个包含软件名称和版本的结构体
{
    Name:    "activerecord",  # 软件名称为 activerecord
    Version: "4.0.1",  # 软件版本为 4.0.1
},
# 定义一个包含 CPE 信息的结构体
Found: CPEResult{
    CPEs:              []string{"cpe:2.3:*:activerecord:activerecord:4.0.1:*:*:*:*:*:*:*"},  # CPEs 列表包含一个 CPE 信息
    VersionConstraint: "= 4.0.1 (semver)",  # 版本约束为 = 4.0.1 (semver)
    VulnerabilityID:   "CVE-2017-fake-3",  # 漏洞 ID 为 CVE-2017-fake-3
},
Matcher: matcher,  # 使用 matcher 进行匹配
# 定义一个包含软件名称和版本的结构体
{
    name: "no match",  # 软件名称为 no match
    p: pkg.Package{  # 定义一个包含软件信息的结构体
        ID:       pkg.ID(uuid.NewString()),  # 生成一个新的唯一 ID
        Name:     "couldntgetthisrightcouldyou",  # 软件名称为 couldntgetthisrightcouldyou
# 定义一个包的版本、语言和类型
Version:  "4.0.1",
Language: syftPkg.Ruby,
Type:     syftPkg.GemPkg,
},
# 期望的匹配结果为空列表
expected: []match.Match{},
},
{
# 定义一个模糊版本匹配
name: "fuzzy version match",
# 定义一个包的信息和CPE
p: pkg.Package{
CPEs: []cpe.CPE{
cpe.Must("cpe:2.3:*:awesome:awesome:98SE1:rando1:*:ra:*:dunno:*:*"),
},
Name:    "awesome",
Version: "98SE1",
},
# 期望的匹配结果
expected: []match.Match{
{
# 定义一个漏洞的信息
Vulnerability: vulnerability.Vulnerability{
ID: "CVE-2017-fake-4",
					},
					Package: pkg.Package{
						CPEs: []cpe.CPE{
							cpe.Must("cpe:2.3:*:awesome:awesome:98SE1:rando1:*:ra:*:dunno:*:*"),
						},
						Name:    "awesome",
						Version: "98SE1",
					},
                    # 定义一个包含详细信息的数组
					Details: []match.Detail{
                        # 定义一个包含匹配类型、置信度和搜索参数的结构体
						{
							Type:       match.CPEMatch,
							Confidence: 0.9,
							SearchedBy: CPEParameters{
                                # 定义一个包含CPE、命名空间和包参数的结构体
								CPEs:      []string{"cpe:2.3:*:awesome:awesome:98SE1:rando1:*:ra:*:dunno:*:*"},
								Namespace: "nvd:cpe",
								Package: CPEPackageParameter{
									Name:    "awesome",
									Version: "98SE1",
								},
# 定义一个结构体，包含了一组CPE信息和匹配器
Found: CPEResult{
    # CPEs字段存储了一个或多个CPE标识
    CPEs: []string{"cpe:2.3:*:awesome:awesome:*:*:*:*:*:*:*"},
    # VersionConstraint字段存储了版本约束条件
    VersionConstraint: "< 98SP3 (unknown)",
    # VulnerabilityID字段存储了漏洞ID
    VulnerabilityID:   "CVE-2017-fake-4",
},
# Matcher字段存储了匹配器信息
Matcher: matcher,
# 结构体结束
# 定义一个语言为Ruby、类型为GemPkg的包
{
    Language: syftPkg.Ruby,
    Type:     syftPkg.GemPkg,
},

# 期望的匹配结果
expected: []match.Match{
    {
        # 漏洞信息
        Vulnerability: vulnerability.Vulnerability{
            ID: "CVE-2017-fake-5",
        },
        # 包信息
        Package: pkg.Package{
            # 包的CPE（通用平台标识符）
            CPEs: []cpe.CPE{
                cpe.Must("cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*"),
            },
            Name:     "multiple",
            Version:  "1.0",
            Language: syftPkg.Ruby,
            Type:     syftPkg.GemPkg,
        },
        # 匹配的详细信息
        Details: []match.Detail{
# 定义一个匹配类型为CPEMatch的对象
{
    Type:       match.CPEMatch,  # 设置匹配类型为CPEMatch
    Confidence: 0.9,  # 设置匹配的置信度为0.9
    SearchedBy: CPEParameters{  # 设置搜索参数为CPEParameters结构体
        CPEs:      []string{"cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*"},  # 设置CPEs参数为包含一个CPE字符串的数组
        Namespace: "nvd:cpe",  # 设置命名空间为nvd:cpe
        Package: CPEPackageParameter{  # 设置包参数为CPEPackageParameter结构体
            Name:    "multiple",  # 设置包名称为multiple
            Version: "1.0",  # 设置包版本为1.0
        },
    },
    Found: CPEResult{  # 设置匹配结果为CPEResult结构体
        CPEs: []string{  # 设置匹配到的CPE字符串数组
            "cpe:2.3:*:multiple:multiple:*:*:*:*:*:*:*:*",
            "cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*",
        },
        VersionConstraint: "< 4.0 (unknown)",  # 设置版本约束为小于4.0（未知）
        VulnerabilityID:   "CVE-2017-fake-5",  # 设置漏洞ID为CVE-2017-fake-5
    },
    Matcher: matcher,  # 设置匹配器为matcher
}
# 创建一个名为 "filtered out match due to target_sw mismatch" 的测试用例
{
    # 创建一个名为 "p" 的包对象
    p: pkg.Package{
        # 设置包对象的 CPEs 属性为一个包含一个 CPE 对象的列表
        CPEs: []cpe.CPE{
            # 创建一个 CPE 对象
            cpe.Must("cpe:2.3:*:funfun:funfun:*:*:*:*:*:*:*:*"),
        },
        # 设置包对象的 Name 属性为 "funfun"
        Name:     "funfun",
        # 设置包对象的 Version 属性为 "5.2.1"
        Version:  "5.2.1",
        # 设置包对象的 Language 属性为 syftPkg.Rust
        Language: syftPkg.Rust,
        # 设置包对象的 Type 属性为 syftPkg.RustPkg
        Type:     syftPkg.RustPkg,
    },
    # 设置期望的结果为一个空的匹配列表
    expected: []match.Match{},
},
{
    # 创建一个名为 "target_sw mismatch with unsupported target_sw" 的测试用例
			# 创建一个名为 p 的 pkg.Package 结构体对象
			p: pkg.Package{
				# 设置 CPEs 字段为包含一个 cpe.CPE 对象的数组
				CPEs: []cpe.CPE{
					# 使用 cpe.Must 函数创建一个 cpe.CPE 对象
					cpe.Must("cpe:2.3:*:sw:sw:*:*:*:*:*:*:*:*"),
				},
				# 设置 Name 字段为 "sw"
				Name:     "sw",
				# 设置 Version 字段为 "0.1"
				Version:  "0.1",
				# 设置 Language 字段为 syftPkg.Erlang
				Language: syftPkg.Erlang,
				# 设置 Type 字段为 syftPkg.HexPkg
				Type:     syftPkg.HexPkg,
			},
			# 创建一个名为 expected 的 match.Match 结构体对象
			expected: []match.Match{
				{
					# 设置 Vulnerability 字段为一个 vulnerability.Vulnerability 对象
					Vulnerability: vulnerability.Vulnerability{
						# 设置 ID 字段为 "CVE-2017-fake-7"
						ID: "CVE-2017-fake-7",
					},
					# 设置 Package 字段为一个 pkg.Package 对象
					Package: pkg.Package{
						# 设置 CPEs 字段为包含一个 cpe.CPE 对象的数组
						CPEs: []cpe.CPE{
							# 使用 cpe.Must 函数创建一个 cpe.CPE 对象
							cpe.Must("cpe:2.3:*:sw:sw:*:*:*:*:*:*:*:*"),
						},
						# 设置 Name 字段为 "sw"
						Name:     "sw",
						# 设置 Version 字段为 "0.1"
						Version:  "0.1",
# 定义语言为 Erlang，类型为 HexPkg 的软件包
Language: syftPkg.Erlang,
Type:     syftPkg.HexPkg,
},
Details: []match.Detail{
    # 定义匹配的细节
    {
        Type:       match.CPEMatch,
        Confidence: 0.9,
        SearchedBy: CPEParameters{
            # 使用 CPE 参数进行搜索
            CPEs:      []string{"cpe:2.3:*:sw:sw:*:*:*:*:*:*:*:*"},
            Namespace: "nvd:cpe",
            Package: CPEPackageParameter{
                Name:    "sw",
                Version: "0.1",
            },
        },
        Found: CPEResult{
            # 找到的 CPE 结果
            CPEs: []string{
                "cpe:2.3:*:sw:sw:*:*:*:*:*:puppet:*:*",
            },
            VersionConstraint: "< 1.0 (unknown)",
# 定义一个漏洞的ID为"CVE-2017-fake-7"的结构体
VulnerabilityID:   "CVE-2017-fake-7",
},
# 定义一个匹配器为matcher的结构体
Matcher: matcher,
},
},
},
},
},
{
# 定义一个名称为"match included even though multiple cpes are mismatch"的结构体
name: "match included even though multiple cpes are mismatch",
# 定义一个包含多个CPE的包结构体
p: pkg.Package{
# 定义一个CPE列表
CPEs: []cpe.CPE{
# 添加一个CPE为"cpe:2.3:*:funfun:funfun:*:*:*:*:*:rust:*:*"
cpe.Must("cpe:2.3:*:funfun:funfun:*:*:*:*:*:rust:*:*"),
# 添加一个CPE为"cpe:2.3:*:funfun:funfun:*:*:*:*:*:rails:*:*"
cpe.Must("cpe:2.3:*:funfun:funfun:*:*:*:*:*:rails:*:*"),
# 添加一个CPE为"cpe:2.3:*:funfun:funfun:*:*:*:*:*:ruby:*:*"
cpe.Must("cpe:2.3:*:funfun:funfun:*:*:*:*:*:ruby:*:*"),
# 添加一个CPE为"cpe:2.3:*:funfun:funfun:*:*:*:*:*:python:*:*"
cpe.Must("cpe:2.3:*:funfun:funfun:*:*:*:*:*:python:*:*"),
},
# 定义包的名称为"funfun"
Name:     "funfun",
# 定义包的版本为"5.2.1"
Version:  "5.2.1",
# 定义包的语言为Python
Language: syftPkg.Python,
// 定义一个类型为 syftPkg.PythonPkg 的变量
Type:     syftPkg.PythonPkg,

// 定义一个期望的匹配结果，包含漏洞信息和软件包信息
expected: []match.Match{
    {
        // 漏洞信息
        Vulnerability: vulnerability.Vulnerability{
            ID: "CVE-2017-fake-6",
        },
        // 软件包信息
        Package: pkg.Package{
            // 软件包的 CPE（通用产品和版本）列表
            CPEs: []cpe.CPE{
                cpe.Must("cpe:2.3:*:funfun:funfun:*:*:*:*:*:rust:*:*"),
                cpe.Must("cpe:2.3:*:funfun:funfun:*:*:*:*:*:rails:*:*"),
                cpe.Must("cpe:2.3:*:funfun:funfun:*:*:*:*:*:ruby:*:*"),
                cpe.Must("cpe:2.3:*:funfun:funfun:*:*:*:*:*:python:*:*"),
            },
            // 软件包名称
            Name:     "funfun",
            // 软件包版本
            Version:  "5.2.1",
            // 软件包语言
            Language: syftPkg.Python,
            // 软件包类型
            Type:     syftPkg.PythonPkg,
        },
        // 匹配详情
        Details: []match.Detail{
# 定义一个匹配类型为CPEMatch的对象
{
    Type:       match.CPEMatch,  # 设置匹配类型为CPEMatch
    Confidence: 0.9,  # 设置匹配的置信度为0.9
    SearchedBy: CPEParameters{  # 设置搜索参数
        CPEs:      []string{"cpe:2.3:*:funfun:funfun:*:*:*:*:*:python:*:*"},  # 设置CPEs列表
        Namespace: "nvd:cpe",  # 设置命名空间为nvd:cpe
        Package: CPEPackageParameter{  # 设置包参数
            Name:    "funfun",  # 设置包名称
            Version: "5.2.1",  # 设置包版本
        },
    },
    Found: CPEResult{  # 设置匹配结果
        CPEs: []string{  # 设置匹配到的CPEs列表
            "cpe:2.3:*:funfun:funfun:*:*:*:*:*:python:*:*",
            "cpe:2.3:*:funfun:funfun:5.2.1:*:*:*:*:python:*:*",
        },
        VersionConstraint: "= 5.2.1 (unknown)",  # 设置版本约束
        VulnerabilityID:   "CVE-2017-fake-6",  # 设置漏洞ID
    },
    Matcher: matcher,  # 设置匹配器
}
# 定义一个测试用例，确保目标软件包不匹配不适用于 Java 软件包
{
    # 定义软件包的属性
    name: "Ensure target_sw mismatch does not apply to java packages",
    p: pkg.Package{
        # 定义软件包的 CPE
        CPEs: []cpe.CPE{
            cpe.Must("cpe:2.3:a:handlebarsjs:handlebars:*:*:*:*:*:*:*:*"),
        },
        # 定义软件包的名称、版本、语言和类型
        Name:     "handlebars",
        Version:  "0.1",
        Language: syftPkg.Java,
        Type:     syftPkg.JavaPkg,
    },
    # 定义预期的匹配结果
    expected: []match.Match{
        {
            # 定义漏洞的属性
            Vulnerability: vulnerability.Vulnerability{
                ID: "CVE-2021-23369",
					},
					Package: pkg.Package{
						CPEs: []cpe.CPE{
							cpe.Must("cpe:2.3:a:handlebarsjs:handlebars:*:*:*:*:*:*:*:*"),  # 定义一个CPE对象
						},
						Name:     "handlebars",  # 设置包的名称
						Version:  "0.1",  # 设置包的版本
						Language: syftPkg.Java,  # 设置包的语言为Java
						Type:     syftPkg.JavaPkg,  # 设置包的类型为Java包
					},
					Details: []match.Detail{  # 定义一个match.Detail对象的数组
						{
							Type:       match.CPEMatch,  # 设置匹配类型为CPEMatch
							Confidence: 0.9,  # 设置匹配的置信度为0.9
							SearchedBy: CPEParameters{  # 定义一个CPEParameters对象
								CPEs:      []string{"cpe:2.3:a:handlebarsjs:handlebars:*:*:*:*:*:*:*:*"},  # 设置CPE参数
								Namespace: "nvd:cpe",  # 设置命名空间
								Package: CPEPackageParameter{  # 定义一个CPEPackageParameter对象
									Name:    "handlebars",  # 设置包的名称
									Version: "0.1",  # 设置包的版本
# 定义一个结构体字段，包含了一些漏洞信息
Found: CPEResult{
    # 漏洞相关的CPE（通用平台表达式）列表
    CPEs: []string{
        "cpe:2.3:a:handlebarsjs:handlebars:*:*:*:*:*:node.js:*:*",
    },
    # 版本约束
    VersionConstraint: "< 4.7.7 (unknown)",
    # 漏洞ID
    VulnerabilityID:   "CVE-2021-23369",
},
# 匹配器
Matcher: matcher,
				},
				// 定义包的名称为"handlebars"
				Name:     "handlebars",
				// 定义包的版本为"0.1"
				Version:  "0.1",
				// 定义包的语言为Java
				Language: syftPkg.Java,
				// 定义包的类型为Jenkins插件包
				Type:     syftPkg.JenkinsPluginPkg,
			},
			// 定义期望的匹配结果
			expected: []match.Match{
				{
					// 定义漏洞ID为"CVE-2021-23369"
					Vulnerability: vulnerability.Vulnerability{
						ID: "CVE-2021-23369",
					},
					// 定义包的信息
					Package: pkg.Package{
						// 定义包的CPE（通用平台表达式）
						CPEs: []cpe.CPE{
							// 定义CPE为"cpe:2.3:a:handlebarsjs:handlebars:*:*:*:*:*:*:*:*"
							cpe.Must("cpe:2.3:a:handlebarsjs:handlebars:*:*:*:*:*:*:*:*"),
						},
						// 定义包的名称为"handlebars"
						Name:     "handlebars",
						// 定义包的版本为"0.1"
						Version:  "0.1",
						// 定义包的语言为Java
						Language: syftPkg.Java,
						// 定义包的类型为Jenkins插件包
						Type:     syftPkg.JenkinsPluginPkg,
					},
// 定义一个包含匹配细节的数组
Details: []match.Detail{
    // 定义一个CPE匹配的细节
    {
        Type:       match.CPEMatch, // 匹配类型为CPE
        Confidence: 0.9, // 匹配的置信度为0.9
        SearchedBy: CPEParameters{ // 使用CPE参数进行搜索
            CPEs:      []string{"cpe:2.3:a:handlebarsjs:handlebars:*:*:*:*:*:*:*:*"}, // 搜索的CPE列表
            Namespace: "nvd:cpe", // CPE的命名空间
            Package: CPEPackageParameter{ // CPE的包参数
                Name:    "handlebars", // 包名为handlebars
                Version: "0.1", // 版本为0.1
            },
        },
        Found: CPEResult{ // 找到的CPE结果
            CPEs: []string{ // 找到的CPE列表
                "cpe:2.3:a:handlebarsjs:handlebars:*:*:*:*:*:node.js:*:*",
            },
            VersionConstraint: "< 4.7.7 (unknown)", // 版本约束为小于4.7.7（未知）
            VulnerabilityID:   "CVE-2021-23369", // 漏洞ID为CVE-2021-23369
        },
        Matcher: matcher, // 匹配器为matcher
    },
# 遍历测试用例
for _, test := range tests:
    # 在测试中运行指定名称的测试用例
    t.Run(test.name, func(t *testing.T) {
        # 创建漏洞提供者对象，并使用模拟存储
        p, err := db.NewVulnerabilityProvider(newMockStore())
        require.NoError(t, err)
        # 使用指定的包名和匹配器获取实际结果
        actual, err := ByPackageCPE(p, nil, test.p, matcher)
        assert.NoError(t, err)
        # 断言实际结果与期望结果匹配
        assertMatchesUsingIDsForVulnerabilities(t, test.expected, actual)
        # 遍历期望结果，比较详细信息
        for idx, e := range test.expected:
            if d := cmp.Diff(e.Details, actual[idx].Details); d != "":
                t.Errorf("unexpected match details (-want +got):\n%s", d)
        }
    })
	}
}

func TestFilterCPEsByVersion(t *testing.T) {
	// 定义测试用例
	tests := []struct {
		name              string
		version           string
		vulnerabilityCPEs []string
		expected          []string
	}{
		// 第一个测试用例
		{
			name:    "filter out by simple version",
			version: "1.0",
			vulnerabilityCPEs: []string{
				// 漏洞 CPE 列表
				"cpe:2.3:*:multiple:multiple:*:*:*:*:*:*:*:*",
				"cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*",
				"cpe:2.3:*:multiple:multiple:2.0:*:*:*:*:*:*:*",
			},
			// 期望结果
			expected: []string{
				"cpe:2.3:*:multiple:multiple:*:*:*:*:*:*:*:*",
		// 定义测试用例
		tests := []struct {
			name               string
			version            string
			vulnerabilityCPEs  []string
		}{
			{
				name: "Test Case 1",
				version: "1.0",
				vulnerabilityCPEs: []string{
					"cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*",
				},
			},
		}

		// 遍历测试用例
		for _, test := range tests {
			t.Run(test.name, func(t *testing.T) {
				// 将漏洞CPE字符串转换为CPE对象
				vulnerabilityCPEs := make([]cpe.CPE, len(test.vulnerabilityCPEs))
				for idx, c := range test.vulnerabilityCPEs {
					vulnerabilityCPEs[idx] = cpe.Must(c)
				}

				// 创建版本对象
				versionObj, err := version.NewVersion(test.version, version.UnknownFormat)
				if err != nil {
					t.Fatalf("unable to get version: %+v", err)
				}

				// 运行测试主体函数
				actual := filterCPEsByVersion(*versionObj, vulnerabilityCPEs)
// 将CPE对象格式化为字符串
actualStrs := make([]string, len(actual)) // 创建一个字符串切片，长度为actual的长度
for idx, a := range actual { // 遍历actual切片
    actualStrs[idx] = a.BindToFmtString() // 将actual中的每个元素格式化为字符串并存储到actualStrs中
}

assert.ElementsMatch(t, test.expected, actualStrs) // 断言actualStrs与test.expected是否匹配

// 测试添加匹配细节
func TestAddMatchDetails(t *testing.T) {
    tests := []struct { // 定义测试用例
        name     string
        existing []match.Detail
        new      match.Detail
        expected []match.Detail
    }{
        { // 测试用例1
# 定义测试用例名称为“append new entry -- found not equal”
name: "append new entry -- found not equal",
# 定义已存在的匹配细节列表
existing: []match.Detail{
    # 定义已存在的匹配细节
    {
        # 定义通过CPE参数搜索
        SearchedBy: CPEParameters{
            Namespace: "nvd:cpe",
            CPEs: []string{
                "cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*",
            },
        },
        # 定义找到的CPE结果
        Found: CPEResult{
            VersionConstraint: "< 2.0 (unknown)",
            CPEs: []string{
                "cpe:2.3:*:multiple:multiple:*:*:*:*:*:*:*:*",
            },
        },
    },
},
# 定义新的匹配细节
new: match.Detail{
    # 定义通过CPE参数搜索
    SearchedBy: CPEParameters{
        Namespace: "nvd:cpe",
# 定义一个结构体，包含了搜索到的CPE信息和期望的CPE信息
expected: []match.Detail{
    # 第一个搜索到的CPE信息
    {
        # 通过CPE参数进行搜索
        SearchedBy: CPEParameters{
            # 指定命名空间
            Namespace: "nvd:cpe",
            # 指定CPE列表
            CPEs: []string{
                "cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*",
            },
        },
        # 搜索到的CPE结果
        Found: CPEResult{
            # 指定版本约束
            VersionConstraint: "< 2.0 (unknown)",
            # 搜索到的CPE列表
            CPEs: []string{
                "totally-different-match",
            },
        },
    },
    # 可能还有其他搜索到的CPE信息
}
# 定义一个版本约束为小于2.0的未知版本
VersionConstraint: "< 2.0 (unknown)",
# 定义一个CPEs列表，包含一个CPE字符串
CPEs: []string{
    "cpe:2.3:*:multiple:multiple:*:*:*:*:*:*:*:*",
},
# 定义一个SearchedBy结构体，包含Namespace和CPEs字段
{
    SearchedBy: CPEParameters{
        # 指定命名空间为nvd:cpe
        Namespace: "nvd:cpe",
        # 定义一个CPEs列表，包含一个CPE字符串
        CPEs: []string{
            "totally-different-search",
        },
    },
    # 定义一个Found结构体，包含VersionConstraint和CPEs字段
    Found: CPEResult{
        # 定义一个版本约束为小于2.0的未知版本
        VersionConstraint: "< 2.0 (unknown)",
        # 定义一个CPEs列表，包含一个CPE字符串
        CPEs: []string{
            "totally-different-match",
        },
    },
},
		},
		},
		{
			# 定义测试用例名称
			name: "append new entry -- searchedBy merge fails",
			# 定义已存在的匹配细节列表
			existing: []match.Detail{
				# 定义匹配细节对象
				{
					# 定义被搜索的CPE参数
					SearchedBy: CPEParameters{
						Namespace: "nvd:cpe",
						CPEs: []string{
							"cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*",
						},
					},
					# 定义找到的CPE结果
					Found: CPEResult{
						VersionConstraint: "< 2.0 (unknown)",
						CPEs: []string{
							"cpe:2.3:*:multiple:multiple:*:*:*:*:*:*:*:*",
						},
					},
				},
			},
			# 创建一个新的 match.Detail 对象
			new: match.Detail{
				# 设置搜索参数，包括命名空间和 CPEs 列表
				SearchedBy: CPEParameters{
					Namespace: "totally-different",
					CPEs: []string{
						"cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*",
					},
				},
				# 设置找到的结果，包括版本约束和 CPEs 列表
				Found: CPEResult{
					VersionConstraint: "< 2.0 (unknown)",
					CPEs: []string{
						"cpe:2.3:*:multiple:multiple:*:*:*:*:*:*:*:*",
					},
				},
			},
			# 设置期望的结果列表
			expected: []match.Detail{
				# 创建一个新的 match.Detail 对象
				{
					# 设置搜索参数，包括命名空间和 CPEs 列表
					SearchedBy: CPEParameters{
						Namespace: "nvd:cpe",
						CPEs: []string{
							"cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*",
# 定义一个包含多个元素的数组
{
    # 定义一个包含多个元素的数组
    SearchedBy: CPEParameters{
        # 设置命名空间为"totally-different"，并定义包含多个元素的数组
        Namespace: "totally-different",
        CPEs: []string{
            # 添加一个CPE字符串到数组中
            "cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*",
        },
    },
    Found: CPEResult{
        # 设置版本约束为"< 2.0 (unknown)"
        VersionConstraint: "< 2.0 (unknown)",
        CPEs: []string{
            # 添加一个CPE字符串到数组中
            "cpe:2.3:*:multiple:multiple:*:*:*:*:*:*:*:*",
        },
    },
},
# 定义测试用例名称为"merge with exiting entry"
{
    # 定义已存在的匹配细节列表
    existing: [
        {
            # 定义已搜索的CPE参数
            SearchedBy: CPEParameters{
                Namespace: "nvd:cpe",
                CPEs: [
                    "cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*",
                ],
            },
            # 定义找到的CPE结果
            Found: CPEResult{
                VersionConstraint: "< 2.0 (unknown)",
                CPEs: [
                    "cpe:2.3:*:multiple:multiple:*:*:*:*:*:*:*:*",
                ],
            }
        }
    ]
}
# 定义了一个新的 match.Detail 结构体，包含了 SearchedBy 和 Found 两个字段
new: match.Detail{
    # 设置了 SearchedBy 字段的值，包括 Namespace 和 CPEs 两个字段
    SearchedBy: CPEParameters{
        Namespace: "nvd:cpe",
        CPEs: []string{
            "totally-different-search",
        },
    },
    # 设置了 Found 字段的值，包括 VersionConstraint 和 CPEs 两个字段
    Found: CPEResult{
        VersionConstraint: "< 2.0 (unknown)",
        CPEs: []string{
            "cpe:2.3:*:multiple:multiple:*:*:*:*:*:*:*:*",
        },
    },
},
# 定义了一个期望的 match.Detail 结构体数组，包含了 SearchedBy 和 Found 两个字段
expected: []match.Detail{
    {
        # 设置了 SearchedBy 字段的值，包括 Namespace 和 CPEs 两个字段
        SearchedBy: CPEParameters{
# 设置 Namespace 为 "nvd:cpe"
Namespace: "nvd:cpe",
# 设置 CPEs 列表，包含两个 CPE 字符串
CPEs: []string{
    "cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*",  # 第一个 CPE 字符串
    "totally-different-search",  # 第二个 CPE 字符串
},
# 设置 Found 结构体的 VersionConstraint 字段和 CPEs 列表
Found: CPEResult{
    VersionConstraint: "< 2.0 (unknown)",  # VersionConstraint 字段
    CPEs: []string{
        "cpe:2.3:*:multiple:multiple:*:*:*:*:*:*:*:*",  # CPEs 列表中的 CPE 字符串
    },
},
# 结束当前 Detail 结构体
},
# 结束当前匹配的 Detail 列表
},
# 结束当前测试用例
},
# 开始下一个测试用例
{
name: "no addition - bad new searchedBy type",
# 设置 existing 列表，包含一个 Detail 结构体
existing: []match.Detail{
    {
        # 设置 SearchedBy 字段为 CPEParameters 结构体
        SearchedBy: CPEParameters{
# 设置 Namespace 为 "nvd:cpe"
Namespace: "nvd:cpe",
# 设置 CPEs 列表，包含一个 CPE 字符串
CPEs: []string{
    "cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*",
},
# 设置 Found 结构体，包含 VersionConstraint 和 CPEs 列表
Found: CPEResult{
    # 设置 VersionConstraint 为 "< 2.0 (unknown)"
    VersionConstraint: "< 2.0 (unknown)",
    # 设置 CPEs 列表，包含一个 CPE 字符串
    CPEs: []string{
        "cpe:2.3:*:multiple:multiple:*:*:*:*:*:*:*:*",
    },
},
# 设置 new 结构体，包含 SearchedBy 和 Found 字段
new: match.Detail{
    # 设置 SearchedBy 为 "something else!"
    SearchedBy: "something else!",
    # 设置 Found 结构体，包含 VersionConstraint 和 CPEs 列表
    Found: CPEResult{
        # 设置 VersionConstraint 为 "< 2.0 (unknown)"
        VersionConstraint: "< 2.0 (unknown)",
        # 设置 CPEs 列表，包含一个 CPE 字符串
        CPEs: []string{
            "cpe:2.3:*:multiple:multiple:*:*:*:*:*:*:*:*",
        },
    },
},
		},
	},
	expected: []match.Detail{  # 定义一个期望的匹配结果列表
		{  # 定义一个匹配结果的详细信息
			SearchedBy: CPEParameters{  # 搜索时使用的CPE参数
				Namespace: "nvd:cpe",  # CPE参数的命名空间
				CPEs: []string{  # CPE参数的具体值列表
					"cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*",
				},
			},
			Found: CPEResult{  # 找到的CPE结果
				VersionConstraint: "< 2.0 (unknown)",  # 版本约束
				CPEs: []string{  # 找到的CPE值列表
					"cpe:2.3:*:multiple:multiple:*:*:*:*:*:*:*:*",
				},
			},
		},
	},
},
{  # 另一个匹配结果的详细信息
# 定义一个名为 "no addition - bad new found type" 的测试用例
name: "no addition - bad new found type",
# 定义一个空的已存在的匹配细节列表
existing: []match.Detail{
    # 定义一个匹配细节对象
    {
        # 定义搜索参数
        SearchedBy: CPEParameters{
            Namespace: "nvd:cpe",
            CPEs: []string{
                "cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*",
            },
        },
        # 定义找到的结果
        Found: CPEResult{
            VersionConstraint: "< 2.0 (unknown)",
            CPEs: []string{
                "cpe:2.3:*:multiple:multiple:*:*:*:*:*:*:*:*",
            },
        },
    },
},
# 定义一个新的匹配细节对象
new: match.Detail{
    # 定义搜索参数
    SearchedBy: CPEParameters{
        Namespace: "nvd:cpe",
// 定义一个结构体，包含了CPEs和Found两个字段
CPEs: []string{
    "cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*",
},
// 定义了一个Found字段，值为"something-else!"
Found: "something-else!",
// 定义了一个expected字段，值为包含了SearchedBy和Found两个字段的结构体切片
expected: []match.Detail{
    {
        // 定义了一个SearchedBy字段，值为包含了Namespace和CPEs两个字段的结构体
        SearchedBy: CPEParameters{
            Namespace: "nvd:cpe",
            CPEs: []string{
                "cpe:2.3:*:multiple:multiple:1.0:*:*:*:*:*:*:*",
            },
        },
        // 定义了一个Found字段，值为包含了VersionConstraint和CPEs两个字段的结构体
        Found: CPEResult{
            VersionConstraint: "< 2.0 (unknown)",
            CPEs: []string{
                "cpe:2.3:*:multiple:multiple:*:*:*:*:*:*:*:*",
            },
        },
    },
}
# 定义测试函数，用于测试 addMatchDetails 函数
func TestAddMatchDetails(t *testing.T) {
	# 定义测试用例
	tests := []struct {
		name     string       # 测试用例名称
		existing CPEResult    # 已存在的 CPE 结果
		new      CPEResult    # 新的 CPE 结果
		expected CPEResult    # 期望的结果
	}{
		{
			# 测试用例1
			name: "Test case 1",
			existing: CPEResult{
				# 已存在的 CPE 结果的属性
			},
			new: CPEResult{
				# 新的 CPE 结果的属性
			},
			expected: CPEResult{
				# 期望的结果的属性
			},
		},
		# 其他测试用例...
	}

	# 遍历测试用例
	for _, test := range tests {
		# 运行单个测试用例
		t.Run(test.name, func(t *testing.T) {
			# 断言测试结果是否符合预期
			assert.Equal(t, test.expected, addMatchDetails(test.existing, test.new))
		})
	}
}

# 定义测试函数，用于测试 CPESearchHit 结构体的 Equals 方法
func TestCPESearchHit_Equals(t *testing.T) {
	# 定义测试用例
	tests := []struct {
		name     string       # 测试用例名称
		current  CPEResult    # 当前的 CPE 结果
		other    CPEResult    # 其他的 CPE 结果
		expected bool         # 期望的结果
	}{
		{
			# 测试用例1
		},
		# 其他测试用例...
	}
}
# 定义测试用例名称为"different version constraint"
name: "different version constraint",
# 定义当前版本约束为"current-constraint"，包含的CPEs为"a-cpe"
current: CPEResult{
    VersionConstraint: "current-constraint",
    CPEs: []string{
        "a-cpe",
    },
},
# 定义其他版本约束为"different-constraint"，包含的CPEs为"a-cpe"
other: CPEResult{
    VersionConstraint: "different-constraint",
    CPEs: []string{
        "a-cpe",
    },
},
# 定义期望结果为false
expected: false,
# 下面是另一个测试用例，名称为"different number of CPEs"，省略其余部分
# 定义测试用例
{
    name: "different CPE value",  # 测试用例名称
    current: CPEResult{  # 当前 CPE 结果
        VersionConstraint: "current-constraint",  # 当前版本约束
        CPEs: []string{  # 当前 CPE 列表
            "a-cpe",  # CPE 值
        },
    },
    other: CPEResult{  # 其他 CPE 结果
        VersionConstraint: "current-constraint",  # 其他版本约束
        CPEs: []string{  # 其他 CPE 列表
            "a-cpe",  # CPE 值
            "b-cpe",  # 另一个 CPE 值
        },
    },
    expected: false,  # 期望结果为假
},
# 定义一个测试用例，包括当前和其他的 CPE 结果，以及预期的结果
{
    name: "does not match",
    # 当前 CPE 结果
    current: CPEResult{
        VersionConstraint: "current-constraint",
        CPEs: []string{
            "a-cpe",
        },
    },
    # 其他 CPE 结果
    other: CPEResult{
        VersionConstraint: "current-constraint",
        CPEs: []string{
            "b-cpe",
        },
    },
    # 预期结果为 false
    expected: false,
},
{
    name: "matches",
    # 当前 CPE 结果
    current: CPEResult{
        VersionConstraint: "current-constraint",
        CPEs: []string{
            "a-cpe",
        },
    },
    # 其他 CPE 结果
    other: CPEResult{
        VersionConstraint: "current-constraint",
        CPEs: []string{
            "a-cpe",
        },
    },
    # 预期结果为 true
    expected: true,
},
# 定义测试用例结构体数组
tests := []struct {
	// 测试用例名称
	name string
	// 当前对象
	current Object
	// 其他对象
	other Object
	// 期望结果
	expected bool
}

# 遍历测试用例数组
for _, test := range tests {
	# 运行单个测试用例
	t.Run(test.name, func(t *testing.T) {
		# 断言当前对象是否等于其他对象，判断是否符合期望结果
		assert.Equal(t, test.expected, test.current.Equals(test.other))
	})
}
```