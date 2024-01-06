# `grype\grype\presenter\models\ignore_test.go`

```
package models

import (
	"testing"  // 导入测试包

	"github.com/google/go-cmp/cmp"  // 导入用于比较的包

	grypeDb "github.com/anchore/grype/grype/db/v5"  // 导入数据库包
	"github.com/anchore/grype/grype/match"  // 导入匹配包
)

func TestNewIgnoreRule(t *testing.T) {  // 定义测试函数
	cases := []struct {  // 定义测试用例
		name     string  // 测试用例名称
		input    match.IgnoreRule  // 输入的匹配忽略规则
		expected IgnoreRule  // 期望的忽略规则
	}{
		{
			name:  "no values",  // 测试用例名称
			input: match.IgnoreRule{},  // 输入的匹配忽略规则为空
# 定义一个期望的忽略规则，其中漏洞、修复状态和包名都为空
expected: IgnoreRule{
    Vulnerability: "",
    FixState:      "",
    Package:       nil,
},
# 定义一个只有漏洞字段的忽略规则
{
    name: "only vulnerability field",
    input: match.IgnoreRule{
        Vulnerability: "CVE-2020-1234",
    },
    expected: IgnoreRule{
        Vulnerability: "CVE-2020-1234",
    },
},
# 定义一个只有修复状态字段的忽略规则
{
    name: "only fix state field",
    input: match.IgnoreRule{
        FixState: string(grypeDb.NotFixedState),
    },
```
# 定义一个忽略规则，设置修复状态为未修复
expected: IgnoreRule{
    FixState: string(grypeDb.NotFixedState),
},

# 定义一个忽略规则，设置包的名称、版本、类型和位置
{
    name: "all package fields",
    input: match.IgnoreRule{
        Package: match.IgnoreRulePackage{
            Name:     "libc",
            Version:  "3.0.0",
            Type:     "rpm",
            Location: "/some/location",
        },
    },
    # 期望的忽略规则，设置包的名称、版本和类型
    expected: IgnoreRule{
        Package: &IgnoreRulePackage{
            Name:     "libc",
            Version:  "3.0.0",
            Type:     "rpm",
# 创建一个包含位置信息的匹配忽略规则
{
    name: "only one location field",  # 规则名称
    input: match.IgnoreRule{  # 输入的匹配忽略规则
        Location: "/some/location",  # 忽略的位置信息
    },
    expected: IgnoreRule{  # 期望的匹配忽略规则
        Location: &IgnoreRuleLocation{  # 忽略的位置信息
            Path: "/some/location",  # 路径信息
        },
    },
},
{
    name: "only one package field",  # 规则名称
    input: match.IgnoreRule{  # 输入的匹配忽略规则
        Package: match.IgnoreRulePackage{  # 忽略的包信息
            Type: "apk",  # 包类型
        },
    },
    expected: IgnoreRule{  # 期望的匹配忽略规则
        Package: &IgnoreRulePackage{  # 忽略的包信息
            Type: "apk",  # 包类型
        },
    },
},
{
    name: "all fields",  # 规则名称
    input: match.IgnoreRule{  # 输入的匹配忽略规则
# 定义漏洞编号为 "CVE-2020-1234"
Vulnerability: "CVE-2020-1234",
# 设置修复状态为未修复
FixState:      string(grypeDb.NotFixedState),
# 定义要忽略的软件包信息
Package: match.IgnoreRulePackage{
    # 软件包名称为 "libc"
    Name:     "libc",
    # 软件包版本为 "3.0.0"
    Version:  "3.0.0",
    # 软件包类型为 "rpm"
    Type:     "rpm",
    # 软件包所在位置为 "/some/location"
    Location: "/some/location",
},
# 定义期望的忽略规则
expected: IgnoreRule{
    # 期望的漏洞编号为 "CVE-2020-1234"
    Vulnerability: "CVE-2020-1234",
    # 期望的修复状态为 "not-fixed"
    FixState:      "not-fixed",
    # 期望的软件包信息
    Package: &IgnoreRulePackage{
        # 期望的软件包名称为 "libc"
        Name:     "libc",
        # 期望的软件包版本为 "3.0.0"
        Version:  "3.0.0",
        # 期望的软件包类型为 "rpm"
        Type:     "rpm",
        # 期望的软件包所在位置为 "/some/location"
        Location: "/some/location",
    },
},
	}

	// 遍历测试用例，依次执行测试
	for _, testCase := range cases {
		// 使用测试用例的名称创建子测试
		t.Run(testCase.name, func(t *testing.T) {
			// 调用被测试的函数，获取实际结果
			actual := newIgnoreRule(testCase.input)
			// 比较预期结果和实际结果，如果不一致则输出差异
			if diff := cmp.Diff(testCase.expected, actual); diff != "" {
				t.Errorf("(-expected +actual):\n%s", diff)
			}
		})
	}
}
```