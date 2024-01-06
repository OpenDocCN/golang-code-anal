# `grype\grype\match\ignore_test.go`

```
package match

import (
	"testing"  // 导入测试包

	"github.com/google/uuid"  // 导入 uuid 包
	"github.com/stretchr/testify/assert"  // 导入 testify 包

	grypeDb "github.com/anchore/grype/grype/db/v5"  // 导入 grypeDb 包
	"github.com/anchore/grype/grype/pkg"  // 导入 grype/pkg 包
	"github.com/anchore/grype/grype/vulnerability"  // 导入 vulnerability 包
	"github.com/anchore/syft/syft/file"  // 导入 file 包
	syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包
)

var (
	allMatches = []Match{  // 定义名为 allMatches 的 Match 切片
		{
			Vulnerability: vulnerability.Vulnerability{  // 定义名为 Vulnerability 的 vulnerability.Vulnerability 结构体
				ID:        "CVE-123",  // 设置 ID 字段值为 "CVE-123"
# 创建一个名为 "debian-vulns" 的命名空间，并设置漏洞修复状态为已修复
Namespace: "debian-vulns",
Fix: vulnerability.Fix{
    State: grypeDb.FixedState,
},

# 创建一个名为 "dive" 的软件包，设置其ID、名称、版本、类型和位置
Package: pkg.Package{
    ID:        pkg.ID(uuid.NewString()),
    Name:      "dive",
    Version:   "0.5.2",
    Type:      "deb",
    Locations: file.NewLocationSet(file.NewLocation("/path/that/has/dive")),
},

# 创建一个名为 "CVE-456" 的漏洞，并设置其命名空间为 "ruby-vulns"，修复状态为未修复
Vulnerability: vulnerability.Vulnerability{
    ID:        "CVE-456",
    Namespace: "ruby-vulns",
    Fix: vulnerability.Fix{
        State: grypeDb.NotFixedState,
    },
		},
		// 创建一个新的包对象
		Package: pkg.Package{
			// 生成一个新的唯一标识符
			ID:       pkg.ID(uuid.NewString()),
			// 设置包的名称
			Name:     "reach",
			// 设置包的版本号
			Version:  "100.0.50",
			// 设置包的语言类型为 Ruby
			Language: syftPkg.Ruby,
			// 设置包的类型为 GemPkg
			Type:     syftPkg.GemPkg,
			// 设置包的位置信息
			Locations: file.NewLocationSet(file.NewVirtualLocation("/real/path/with/reach",
				"/virtual/path/that/has/reach")),
		},
		// 创建一个新的漏洞对象
		{
			Vulnerability: vulnerability.Vulnerability{
				// 设置漏洞的ID
				ID:        "CVE-457",
				// 设置漏洞的命名空间
				Namespace: "ruby-vulns",
				// 设置漏洞的修复状态为 WontFixState
				Fix: vulnerability.Fix{
					State: grypeDb.WontFixState,
				},
			},
			// 创建一个新的包对象
			Package: pkg.Package{
# 创建一个唯一标识符
ID: pkg.ID(uuid.NewString()),
# 设置软件包的名称为"beach"
Name: "beach",
# 设置软件包的版本号为"100.0.51"
Version: "100.0.51",
# 设置软件包的语言为Ruby
Language: syftPkg.Ruby,
# 设置软件包的类型为GemPkg
Type: syftPkg.GemPkg,
# 设置软件包的位置为虚拟路径"/virtual/path/that/has/beach"对应的真实路径"/real/path/with/beach"
Locations: file.NewLocationSet(file.NewVirtualLocation("/real/path/with/beach", "/virtual/path/that/has/beach")),
# 设置漏洞的ID为"CVE-458"
ID: "CVE-458",
# 设置漏洞的命名空间为"ruby-vulns"
Namespace: "ruby-vulns",
# 设置漏洞的修复状态为未知状态
Fix: vulnerability.Fix{
    State: grypeDb.UnknownFixState,
},
# 创建一个唯一标识符
ID: pkg.ID(uuid.NewString()),
# 设置软件包的名称为"speach"
Name: "speach",
# 定义一个版本号为"100.0.52"的软件包，语言为Ruby，类型为GemPkg，位置为虚拟路径"/virtual/path/that/has/speach"
syftPkg := &Package{
    Name: "example-package",
    Version: "100.0.52",
    Language: syftPkg.Ruby,
    Type: syftPkg.GemPkg,
    Locations: file.NewLocationSet(file.NewVirtualLocation("/real/path/with/speach", "/virtual/path/that/has/speach")),
}

# 定义一个测试函数TestApplyIgnoreRules，用于测试应用忽略规则的情况
func TestApplyIgnoreRules(t *testing.T) {
    # 定义测试用例
    cases := []struct {
        name                     string
        allMatches               []Match
        ignoreRules              []IgnoreRule
        expectedRemainingMatches []Match
        expectedIgnoredMatches   []IgnoredMatch
    }{
        {
            name: "no ignore rules",
            # 其他测试用例的定义和期望结果
        },
    }
}
		// 定义变量allMatches，并赋值为allMatches
		allMatches:               allMatches,
		// 定义变量ignoreRules，并赋值为nil
		ignoreRules:              nil,
		// 定义变量expectedRemainingMatches，并赋值为allMatches
		expectedRemainingMatches: allMatches,
		// 定义变量expectedIgnoredMatches，并赋值为nil
		expectedIgnoredMatches:   nil,
	},
	// 定义测试用例名称为"no applicable ignore rules"
	{
		// 测试用例中的变量allMatches赋值为allMatches
		allMatches: allMatches,
		// 测试用例中的变量ignoreRules为包含三个IgnoreRule的数组
		ignoreRules: []IgnoreRule{
			// 第一个IgnoreRule的Vulnerability属性为"CVE-789"
			{
				Vulnerability: "CVE-789",
			},
			// 第二个IgnoreRule的Package属性为IgnoreRulePackage对象
			{
				Package: IgnoreRulePackage{
					Name:    "bashful",
					Version: "5",
					Type:    "npm",
				},
			},
			{
# 定义一个 IgnoreRulePackage 结构体，包含名称和版本信息
Package: IgnoreRulePackage{
    Name:    "reach",
    Version: "3000",
},
# 定义一个测试用例，名称为 "ignore all matches"
{
    name:       "ignore all matches",
    # 定义所有匹配项
    allMatches: allMatches,
    # 定义忽略规则列表
    ignoreRules: []IgnoreRule{
        # 定义一个忽略规则，指定漏洞为 "CVE-123"
        {
            Vulnerability: "CVE-123",
        },
        # 定义一个忽略规则，指定包的位置为 "/virtual/path/that/has/reach"
        {
            Package: IgnoreRulePackage{
                Location: "/virtual/path/that/has/reach",
            },
# 定义了一个包含多个嵌套结构的数据结构
# 包含了多个字段，如expectedRemainingMatches、expectedIgnoredMatches等
# expectedRemainingMatches字段包含了多个Match类型的元素
# expectedIgnoredMatches字段包含了多个IgnoredMatch类型的元素
# 每个IgnoredMatch类型的元素包含了Match类型的元素和AppliedIgnoreRules字段
# AppliedIgnoreRules字段包含了多个IgnoreRule类型的元素
# IgnoreRule类型的元素包含了Vulnerability和Package字段
# 设置位置为"/virtual/path/that/has/reach"
Location: "/virtual/path/that/has/reach",
# 设置所有匹配项
allMatches: allMatches,
# 设置忽略规则
ignoreRules: []IgnoreRule{
    # 设置忽略的漏洞为"CVE-456"
    {
        Vulnerability: "CVE-456",
    },
},
# 期望剩余匹配项为指定的匹配项
expectedRemainingMatches: []Match{
    allMatches[0],
    allMatches[2],
    allMatches[3],
},
# 定义了一个名为expectedIgnoredMatches的数组，用于存储被忽略的匹配项
expectedIgnoredMatches: []IgnoredMatch{
    # 定义了一个被忽略的匹配项，包括匹配的信息和应用的忽略规则
    {
        Match: allMatches[1], # 匹配的信息
        AppliedIgnoreRules: []IgnoreRule{ # 应用的忽略规则
            {
                Vulnerability: "CVE-456", # 漏洞编号
            },
        },
    },
},
# 定义了一个名为ignore matches without fix的测试用例，包括所有匹配项、忽略规则和期望的剩余匹配项
{
    name:       "ignore matches without fix", # 测试用例名称
    allMatches: allMatches, # 所有匹配项
    ignoreRules: []IgnoreRule{ # 忽略规则
        {FixState: string(grypeDb.NotFixedState)}, # 未修复状态
        {FixState: string(grypeDb.WontFixState)}, # 不会修复状态
        {FixState: string(grypeDb.UnknownFixState)}, # 未知修复状态
    },
    expectedRemainingMatches: []Match{ # 期望的剩余匹配项
# 定义一个包含所有匹配项的数组
allMatches[0],

# 定义期望被忽略的匹配项数组
expectedIgnoredMatches: []IgnoredMatch{
    # 定义一个被忽略的匹配项
    {
        # 匹配项为数组中的第二个元素
        Match: allMatches[1],
        # 应用的忽略规则数组
        AppliedIgnoreRules: []IgnoreRule{
            # 定义一个忽略规则，修复状态为"not-fixed"
            {
                FixState: "not-fixed",
            },
        },
    },
    {
        # 定义另一个被忽略的匹配项
        Match: allMatches[2],
        # 应用的忽略规则数组
        AppliedIgnoreRules: []IgnoreRule{
            # 定义一个忽略规则，修复状态为"wont-fix"
            {
                FixState: "wont-fix",
            },
        },
    },
    {
# 创建一个包含匹配和应用的忽略规则的结构体
Match: allMatches[3],
AppliedIgnoreRules: []IgnoreRule{
    # 创建一个忽略规则，修复状态为未知
    {
        FixState: "unknown",
    },
},
# 创建一个包含名称、所有匹配、忽略规则、预期剩余匹配和预期被忽略匹配的结构体
{
    name: "ignore matches on namespace",
    allMatches: allMatches,
    ignoreRules: []IgnoreRule{
        # 创建一个忽略规则，命名空间为"ruby-vulns"
        {Namespace: "ruby-vulns"},
    },
    expectedRemainingMatches: []Match{
        # 预期剩余匹配为allMatches中的第一个
        allMatches[0],
    },
    expectedIgnoredMatches: []IgnoredMatch{
        {
# 创建一个包含匹配和应用的忽略规则的结构体
{
    # 匹配的内容
    Match: allMatches[1],
    # 应用的忽略规则列表
    AppliedIgnoreRules: []IgnoreRule{
        # 创建一个忽略规则，指定其命名空间为"ruby-vulns"
        {
            Namespace: "ruby-vulns",
        },
    },
},
{
    # 匹配的内容
    Match: allMatches[2],
    # 应用的忽略规则列表
    AppliedIgnoreRules: []IgnoreRule{
        # 创建一个忽略规则，指定其命名空间为"ruby-vulns"
        {
            Namespace: "ruby-vulns",
        },
    },
},
{
    # 匹配的内容
    Match: allMatches[3],
    # 应用的忽略规则列表
    AppliedIgnoreRules: []IgnoreRule{
        # 创建一个忽略规则，指定其命名空间为"ruby-vulns"
# 定义一个测试用例，测试忽略特定语言的匹配项
{
    name:       "ignore matches on language",  # 测试用例名称
    allMatches: allMatches,  # 所有匹配项
    ignoreRules: [  # 忽略规则列表
        {
            Package: IgnoreRulePackage{  # 忽略规则的包信息
                Language: string(syftPkg.Ruby),  # 忽略的语言类型为 Ruby
            },
        },
    ],
    expectedRemainingMatches: [  # 预期剩余的匹配项
        allMatches[0],  # 第一个匹配项
    ],
    expectedIgnoredMatches: [  # 预期被忽略的匹配项
        {
            # 在这里应该有被忽略的匹配项的具体信息
        },
    ],
}
# 创建一个包含匹配和应用的忽略规则的结构体
Match: allMatches[1],
AppliedIgnoreRules: []IgnoreRule{
    # 创建一个忽略规则对象，指定语言为 Ruby
    {
        Package: IgnoreRulePackage{
            Language: string(syftPkg.Ruby),
        },
    },
},
# 创建另一个包含匹配和应用的忽略规则的结构体
{
    Match: allMatches[2],
    AppliedIgnoreRules: []IgnoreRule{
        # 创建一个忽略规则对象，指定语言为 Ruby
        {
            Package: IgnoreRulePackage{
                Language: string(syftPkg.Ruby),
            },
        },
    },
},
# 创建一个包含匹配项和应用的忽略规则的结构体
Match: allMatches[3],
AppliedIgnoreRules: []IgnoreRule{
    {
        Package: IgnoreRulePackage{
            Language: string(syftPkg.Ruby), # 设置语言为 Ruby
        },
    },
},

# 遍历测试用例
for _, testCase := range cases {
    # 对每个测试用例运行子测试
    t.Run(testCase.name, func(t *testing.T) {
        # 应用忽略规则到匹配项，获取剩余匹配项和被忽略的匹配项
        actualRemainingMatches, actualIgnoredMatches := ApplyIgnoreRules(sliceToMatches(testCase.allMatches), testCase.ignoreRules)

        # 断言剩余匹配项的顺序是否符合预期
        assertMatchOrder(t, testCase.expectedRemainingMatches, actualRemainingMatches.Sorted())
        # 断言被忽略的匹配项的顺序是否符合预期
        assertIgnoredMatchOrder(t, testCase.expectedIgnoredMatches, actualIgnoredMatches)
// 将切片转换为 Matches 结构
func sliceToMatches(s []Match) Matches {
	// 创建一个新的 Matches 结构
	matches := NewMatches()
	// 将切片中的 Match 对象添加到 matches 结构中
	matches.Add(s...)
	// 返回转换后的 Matches 结构
	return matches
}

// 定义一个示例的 Match 对象
var (
	exampleMatch = Match{
		// 定义漏洞的 ID
		Vulnerability: vulnerability.Vulnerability{
			ID: "CVE-2000-1234",
		},
		// 定义包的信息
		Package: pkg.Package{
			ID:      pkg.ID(uuid.NewString()),
			Name:    "a-pkg",
			Version: "1.0",
			// 定义文件的位置信息
			Locations: file.NewLocationSet(
# 创建一个文件对象，指定其物理路径
file.NewLocation("/some/path"),
# 创建一个虚拟文件对象，指定其虚拟路径和物理路径
file.NewVirtualLocation("/some/path", "/some/virtual/path"),
# 指定文件类型为rpm
Type: "rpm",
}

# 定义测试函数TestShouldIgnore
func TestShouldIgnore(t *testing.T) {
    # 定义测试用例
    cases := []struct {
        name     string
        match    Match
        rule     IgnoreRule
        expected bool
    }{
        {
            name:     "empty rule",
            match:    exampleMatch,
            rule:     IgnoreRule{},
            expected: false,
		},
		{
			# 定义规则名称为“rule applies via vulnerability ID”，匹配条件为exampleMatch，规则为忽略规则，忽略的漏洞ID为exampleMatch.Vulnerability.ID
			name:  "rule applies via vulnerability ID",
			match: exampleMatch,
			rule: IgnoreRule{
				Vulnerability: exampleMatch.Vulnerability.ID,
			},
			expected: true,
		},
		{
			# 定义规则名称为“rule applies via package name”，匹配条件为exampleMatch，规则为忽略规则，忽略的包名为exampleMatch.Package.Name
			name:  "rule applies via package name",
			match: exampleMatch,
			rule: IgnoreRule{
				Package: IgnoreRulePackage{
					Name: exampleMatch.Package.Name,
				},
			},
			expected: true,
		},
		{
# 定义测试用例名称为"rule applies via package version"
# 设置匹配条件为exampleMatch
# 创建忽略规则对象，设置忽略规则的包版本为exampleMatch.Package.Version
# 期望结果为true
{
    name:  "rule applies via package version",
    match: exampleMatch,
    rule: IgnoreRule{
        Package: IgnoreRulePackage{
            Version: exampleMatch.Package.Version,
        },
    },
    expected: true,
},

# 定义测试用例名称为"rule applies via package type"
# 设置匹配条件为exampleMatch
# 创建忽略规则对象，设置忽略规则的包类型为exampleMatch.Package.Type的字符串形式
# 期望结果为true
{
    name:  "rule applies via package type",
    match: exampleMatch,
    rule: IgnoreRule{
        Package: IgnoreRulePackage{
            Type: string(exampleMatch.Package.Type),
        },
    },
    expected: true,
},
# 定义测试用例名称为“rule applies via package location real path”，并设置匹配条件和期望结果
name:  "rule applies via package location real path",
match: exampleMatch,
rule: IgnoreRule{
    Package: IgnoreRulePackage{
        # 设置规则的包位置为实际路径
        Location: exampleMatch.Package.Locations.ToSlice()[0].RealPath,
    },
},
expected: true,
# 定义测试用例名称为“rule applies via package location virtual path”，并设置匹配条件和期望结果
name:  "rule applies via package location virtual path",
match: exampleMatch,
rule: IgnoreRule{
    Package: IgnoreRulePackage{
        # 设置规则的包位置为虚拟路径
        Location: exampleMatch.Package.Locations.ToSlice()[1].AccessPath,
    },
},
expected: true,
# 定义规则名称为"rule applies via package location glob"，匹配条件为exampleMatch，规则为忽略规则
{
    # 定义忽略规则的包信息
    Package: IgnoreRulePackage{
        # 包的位置匹配为"/some/**"
        Location: "/some/**",
    },
    # 期望结果为true
    expected: true,
},
# 定义规则名称为"rule applies via multiple fields"，匹配条件为exampleMatch，规则为忽略规则
{
    # 定义忽略规则的包信息
    Package: IgnoreRulePackage{
        # 包的类型为exampleMatch.Package.Type的字符串形式
        Type: string(exampleMatch.Package.Type),
    },
    # 期望结果为true
    expected: true,
},
# 定义测试用例
{
    # 测试用例名称
    name:  "rule doesn't apply despite some fields matching",
    # 匹配对象
    match: exampleMatch,
    # 忽略规则
    rule: IgnoreRule{
        # 漏洞ID
        Vulnerability: exampleMatch.Vulnerability.ID,
        # 包信息
        Package: IgnoreRulePackage{
            # 包名称
            Name:    "not-the-right-package",
            # 包版本
            Version: exampleMatch.Package.Version,
        },
    },
    # 期望结果
    expected: false,
}

# 遍历测试用例
for _, testCase := range cases:
    # 运行测试
    t.Run(testCase.name, func(t *testing.T):
        actual := shouldIgnore(testCase.match, testCase.rule)
        # 断言判断实际结果与期望结果是否一致
        assert.Equal(t, testCase.expected, actual)
    )
这是一个代码块的结束符号，表示前面的函数或者循环的结束。
```