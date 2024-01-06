# `grype\grype\cpe\cpe_test.go`

```
package cpe
// 导入所需的包

import (
	"testing"
	// 导入测试包

	"github.com/sergi/go-diff/diffmatchpatch"
	// 导入用于比较差异的包

	"github.com/anchore/syft/syft/cpe"
	// 导入CPE包
)

func TestMatchWithoutVersion(t *testing.T) {
	// 定义测试函数
	tests := []struct {
		name       string
		// 测试用例名称
		compare    cpe.CPE
		// 要比较的CPE
		candidates []cpe.CPE
		// 候选CPE列表
		expected   []cpe.CPE
		// 期望的CPE列表
	}{
		{
			name:    "GoCase",
			// 测试用例名称
			compare: cpe.Must("cpe:2.3:*:python-requests:requests:2.3.0:*:*:*:*:python:*:*"),
			// 创建CPE对象
# 创建一个包含候选项和期望结果的测试用例
{
    name: "IgnoreVersion",  # 测试用例的名称
    compare: cpe.Must("cpe:2.3:*:name:name:3.2:*:*:*:*:java:*:*"),  # 设置比较对象
    candidates: [  # 候选项列表
        cpe.Must("cpe:2.3:*:name:name:3.2:*:*:*:*:java:*:*"),  # 候选项1
        cpe.Must("cpe:2.3:*:name:name:3.3:*:*:*:*:java:*:*"),  # 候选项2
        cpe.Must("cpe:2.3:*:name:name:5.5:*:*:*:*:java:*:*"),  # 候选项3
    ],
    expected: [  # 期望结果列表
        cpe.Must("cpe:2.3:*:name:name:3.2:*:*:*:*:java:*:*"),  # 期望结果1
        cpe.Must("cpe:2.3:*:name:name:3.3:*:*:*:*:java:*:*"),  # 期望结果2
        cpe.Must("cpe:2.3:*:name:name:5.5:*:*:*:*:java:*:*"),  # 期望结果3
    ],
}
		},
		{
			# 设置测试用例名称为"MatchByTargetSW"
			name:    "MatchByTargetSW",
			# 设置比较对象为指定的 CPE
			compare: cpe.Must("cpe:2.3:*:name:name:3.2:*:*:*:*:java:*:*"),
			# 设置候选对象列表
			candidates: []cpe.CPE{
				# 添加候选对象
				cpe.Must("cpe:2.3:*:name:name:3.2:*:*:*:*:java:*:*"),
				cpe.Must("cpe:2.3:*:name:name:3.2:*:*:*:*:maven:*:*"),
				cpe.Must("cpe:2.3:*:name:name:3.2:*:*:*:*:jenkins:*:*"),
				cpe.Must("cpe:2.3:*:name:name:3.2:*:*:*:*:cloudbees_jenkins:*:*"),
				cpe.Must("cpe:2.3:*:name:name:3.2:*:*:*:*:*:*:*"),
			},
			# 设置期望结果列表
			expected: []cpe.CPE{
				# 添加期望结果
				cpe.Must("cpe:2.3:*:name:name:3.2:*:*:*:*:java:*:*"),
				cpe.Must("cpe:2.3:*:name:name:3.2:*:*:*:*:*:*:*"),
			},
		},
		{
			# 设置测试用例名称为"MatchByName"
			name:    "MatchByName",
			# 设置比较对象为指定的 CPE
			compare: cpe.Must("cpe:2.3:*:name:name5:3.2:*:*:*:*:java:*:*"),
			# 设置候选对象列表
			candidates: []cpe.CPE{
# 创建CPE对象并添加到列表中
cpe.Must("cpe:2.3:*:name:name1:3.2:*:*:*:*:java:*:*"),
cpe.Must("cpe:2.3:*:name:name2:3.2:*:*:*:*:java:*:*"),
cpe.Must("cpe:2.3:*:name:name3:3.2:*:*:*:*:java:*:*"),
cpe.Must("cpe:2.3:*:name:name4:3.2:*:*:*:*:java:*:*"),
cpe.Must("cpe:2.3:*:name:name5:3.2:*:*:*:*:*:*:*"),
# 期望的CPE对象列表
expected: []cpe.CPE{
    cpe.Must("cpe:2.3:*:name:name5:3.2:*:*:*:*:*:*:*"),
},
# 创建CPE对象并添加到列表中
name:    "MatchByVendor",
compare: cpe.Must("cpe:2.3:*:name3:name:3.2:*:*:*:*:java:*:*"),
candidates: []cpe.CPE{
    cpe.Must("cpe:2.3:*:name1:name:3.2:*:*:*:*:java:*:*"),
    cpe.Must("cpe:2.3:*:name3:name:3.2:*:*:*:*:jaba-no-bother:*:*"),
    cpe.Must("cpe:2.3:*:name3:name:3.2:*:*:*:*:java:*:*"),
    cpe.Must("cpe:2.3:*:name4:name:3.2:*:*:*:*:java:*:*"),
    cpe.Must("cpe:2.3:*:name5:name:3.2:*:*:*:*:*:*:*"),
},
# 创建一个包含期望结果的 CPE 对象列表
expected: []cpe.CPE{
    # 创建一个 CPE 对象，表示任意版本的名为 name3 的软件，使用 Java 编程语言
    cpe.Must("cpe:2.3:*:name3:name:3.2:*:*:*:*:java:*:*"),
},
# 创建一个测试用例
{
    # 测试用例名称
    name:    "MatchAnyVendorOrTargetSW",
    # 要比较的 CPE 对象
    compare: cpe.Must("cpe:2.3:*:*:name:3.2:*:*:*:*:*:*:*"),
    # 候选的 CPE 对象列表
    candidates: []cpe.CPE{
        # 创建一个 CPE 对象，表示任意版本的名为 name1 的软件，使用 Java 编程语言
        cpe.Must("cpe:2.3:*:name1:name:3.2:*:*:*:*:java:*:*"),
        # 创建一个 CPE 对象，表示任意版本的名为 name3 的软件，不使用 Java 编程语言
        cpe.Must("cpe:2.3:*:name3:name:3.2:*:*:*:*:jaba-no-bother:*:*"),
        # 创建一个 CPE 对象，表示任意版本的名为 name3 的软件，使用 Java 编程语言
        cpe.Must("cpe:2.3:*:name3:name:3.2:*:*:*:*:java:*:*"),
        # 创建一个 CPE 对象，表示任意版本的名为 name4 的软件，使用 Java 编程语言
        cpe.Must("cpe:2.3:*:name4:name:3.2:*:*:*:*:java:*:*"),
        # 创建一个 CPE 对象，表示任意版本的名为 name5 的软件
        cpe.Must("cpe:2.3:*:name5:name:3.2:*:*:*:*:*:*:*"),
        # 创建一个 CPE 对象，表示任意版本的名为 name5 的软件，但不匹配任何条件
        cpe.Must("cpe:2.3:*:name5:NOMATCH:3.2:*:*:*:*:*:*:*"),
    },
    # 期望的结果列表
    expected: []cpe.CPE{
        # 创建一个 CPE 对象，表示任意版本的名为 name1 的软件，使用 Java 编程语言
        cpe.Must("cpe:2.3:*:name1:name:3.2:*:*:*:*:java:*:*"),
        # 创建一个 CPE 对象，表示任意版本的名为 name3 的软件，不使用 Java 编程语言
        cpe.Must("cpe:2.3:*:name3:name:3.2:*:*:*:*:jaba-no-bother:*:*"),
        # 创建一个 CPE 对象，表示任意版本的名为 name3 的软件，使用 Java 编程语言
        cpe.Must("cpe:2.3:*:name3:name:3.2:*:*:*:*:java:*:*"),
        # 创建一个 CPE 对象，表示任意版本的名为 name4 的软件，使用 Java 编程语言
        cpe.Must("cpe:2.3:*:name4:name:3.2:*:*:*:*:java:*:*"),
    },
},
# 创建一个包含测试用例的切片
tests := []struct {
	# 测试用例的名称
	name string
	# 用于比较的CPE
	compare string
	# 候选CPE列表
	candidates []string
	# 期望的匹配结果
	expected []CPE
}{
	{
		# 测试用例1
		name: "Test Case 1",
		compare: "cpe:2.3:*:name1:name:3.2:*:*:*:*:*:*:*",
		candidates: []string{
			"cpe:2.3:*:name2:name:3.2:*:*:*:*:*:*:*",
			"cpe:2.3:*:name3:name:3.2:*:*:*:*:*:*:*",
			"cpe:2.3:*:name4:name:3.2:*:*:*:*:*:*:*",
			"cpe:2.3:*:name5:name:3.2:*:*:*:*:*:*:*",
		},
		expected: []CPE{
			"cpe:2.3:*:name5:name:3.2:*:*:*:*:*:*:*",
		},
	},
}

# 遍历测试用例切片
for _, test := range tests {
	# 运行测试用例
	t.Run(test.name, func(t *testing.T) {
		# 获取实际的匹配结果
		actual := MatchWithoutVersion(test.compare, test.candidates)

		# 检查实际结果和期望结果的长度是否相等
		if len(actual) != len(test.expected) {
			# 如果长度不相等，打印出所有实际结果，并报告长度不相等的错误
			for _, e := range actual {
				t.Errorf("   unexpected entry: %+v", e.BindToFmtString())
			}
			t.Fatalf("unexpected number of entries: %d", len(actual))
		}

		# 遍历实际结果和期望结果，比较它们的格式化字符串表示
		for idx, a := range actual {
			e := test.expected[idx]
			if a.BindToFmtString() != e.BindToFmtString() {
				# 如果格式化字符串不相等，创建一个差异比较对象
				dmp := diffmatchpatch.New()
# 使用比较算法计算两个字符串之间的差异，并返回差异结果
diffs := dmp.DiffMain(a.BindToFmtString(), e.BindToFmtString(), true)
# 如果存在不匹配的条目，则输出错误信息，包括索引、期望值、实际值和差异
t.Errorf("mismatched entries @ %d:\n\texpected:%+v\n\t  actual:%+v\n\t    diff:%+v\n", idx, e.BindToFmtString(), a.BindToFmtString(), dmp.DiffPrettyText(diffs))
```