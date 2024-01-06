# `grype\grype\version\rpm_constraint_test.go`

```
package version

import (
	"testing"

	"github.com/stretchr/testify/assert"
)

func TestVersionRpmConstraint(t *testing.T) {
	tests := []testCase{
		// 定义测试用例数组

		// empty values
		{version: "2.3.1", constraint: "", satisfied: true},
		// 测试空约束条件下的情况，期望结果为满足

		// trivial compound conditions
		{version: "2.3.1", constraint: "> 1.0.0, < 2.0.0", satisfied: false},
		{version: "1.3.1", constraint: "> 1.0.0, < 2.0.0", satisfied: true},
		{version: "2.0.0", constraint: "> 1.0.0, <= 2.0.0", satisfied: true},
		{version: "2.0.0", constraint: "> 1.0.0, < 2.0.0", satisfied: false},
		{version: "1.0.0", constraint: ">= 1.0.0, < 2.0.0", satisfied: true},
		{version: "1.0.0", constraint: "> 1.0.0, < 2.0.0", satisfied: false},
		{version: "0.9.0", constraint: "> 1.0.0, < 2.0.0", satisfied: false},
		// 测试各种复杂约束条件下的情况，包括满足和不满足的情况
// 对给定的版本和约束条件进行判断，返回是否满足的布尔值
{version: "1.5.0", constraint: "> 0.1.0, < 0.5.0 || > 1.0.0, < 2.0.0", satisfied: true},
{version: "0.2.0", constraint: "> 0.1.0, < 0.5.0 || > 1.0.0, < 2.0.0", satisfied: true},
{version: "0.0.1", constraint: "> 0.1.0, < 0.5.0 || > 1.0.0, < 2.0.0", satisfied: false},
{version: "0.6.0", constraint: "> 0.1.0, < 0.5.0 || > 1.0.0, < 2.0.0", satisfied: false},
{version: "2.5.0", constraint: "> 0.1.0, < 0.5.0 || > 1.0.0, < 2.0.0", satisfied: false},
// 一些简单的情景
{version: "2.3.1", constraint: "< 2.0.0", satisfied: false},
{version: "2.3.1", constraint: "< 2.0", satisfied: false},
{version: "2.3.1", constraint: "< 2", satisfied: false},
{version: "2.3.1", constraint: "< 2.3", satisfied: false},
{version: "2.3.1", constraint: "< 2.3.1", satisfied: false},
{version: "2.3.1", constraint: "< 2.3.2", satisfied: true},
{version: "2.3.1", constraint: "< 2.4", satisfied: true},
{version: "2.3.1", constraint: "< 3", satisfied: true},
{version: "2.3.1", constraint: "< 3.0", satisfied: true},
{version: "2.3.1", constraint: "< 3.0.0", satisfied: true},
// epoch
{version: "1:0", constraint: "< 0:1", satisfied: false},
{version: "2:4.19.01-1.el7_5", constraint: "< 2:4.19.1-1.el7_5", satisfied: false},
{version: "2:4.19.01-1.el7_5", constraint: "<= 2:4.19.1-1.el7_5", satisfied: true},
		// 版本比较结果，包括版本号、约束条件和是否满足约束条件
		{version: "0:4.19.1-1.el7_5", constraint: "< 2:4.19.1-1.el7_5", satisfied: true},  // 满足约束条件
		{version: "11:4.19.0-1.el7_5", constraint: "< 12:4.19.0-1.el7", satisfied: true},  // 满足约束条件
		{version: "13:4.19.0-1.el7_5", constraint: "< 12:4.19.0-1.el7", satisfied: false},  // 不满足约束条件
		// 回归：https://github.com/anchore/grype/issues/316
		{version: "1.5.4-2.el7_9", constraint: "< 0:1.5.4-2.el7_9", satisfied: false},  // 不满足约束条件
		{version: "1.5.4-2.el7", constraint: "< 0:1.5.4-2.el7_9", satisfied: true},  // 满足约束条件
		// 非标准的 epoch 处理。在只有一个方向上比较 epoch 时，它们都会被忽略
		{version: "1:0", constraint: "< 1", satisfied: true},  // 满足约束条件
		{version: "0:0", constraint: "< 0", satisfied: false},  // 不满足约束条件
		{version: "0:0", constraint: "= 0", satisfied: true},  // 满足约束条件
		{version: "0", constraint: "= 0:0", satisfied: true},  // 满足约束条件
		{version: "1.0", constraint: "< 2:1.0", satisfied: false},  // 不满足约束条件
		{version: "1.0", constraint: "<= 2:1.0", satisfied: true},  // 满足约束条件
		{version: "1:2", constraint: "< 1", satisfied: false},  // 不满足约束条件
		{version: "1:2", constraint: "> 1", satisfied: true},  // 满足约束条件
		{version: "2:4.19.01-1.el7_5", constraint: "< 4.19.1-1.el7_5", satisfied: false},  // 不满足约束条件
		{version: "2:4.19.01-1.el7_5", constraint: "<= 4.19.1-1.el7_5", satisfied: true},  // 满足约束条件
		{version: "4.19.01-1.el7_5", constraint: "< 2:4.19.1-1.el7_5", satisfied: false},  // 不满足约束条件
		{version: "4.19.0-1.el7_5", constraint: "< 12:4.19.0-1.el7", satisfied: false},  // 不满足约束条件
		{version: "4.19.0-1.el7_5", constraint: "<= 12:4.19.0-1.el7", satisfied: false},  // 不满足约束条件
# 定义一个包含版本信息和约束条件的测试数据列表
tests := []struct {
    version    string  // 版本信息
    constraint string  // 约束条件
    satisfied  bool    // 是否满足约束条件
}{
    {version: "3:4.19.0-1.el7_5", constraint: "< 4.21.0-1.el7", satisfied: true},  // 版本满足约束条件
    {version: "4:1.2.3-3-el7_5", constraint: "< 1.2.3-el7_5~snapshot1", satisfied: false},  // 版本不满足约束条件
}

# 遍历测试数据列表
for _, test := range tests {
    # 对每个测试数据运行子测试
    t.Run(test.tName(), func(t *testing.T) {
        # 将约束条件转换为 RPM 约束对象
        constraint, err := newRpmConstraint(test.constraint)
        assert.NoError(t, err, "unexpected error from newRpmConstraint: %v", err)  // 断言没有错误发生

        # 断言版本满足约束条件
        test.assertVersionConstraint(t, RpmFormat, constraint)
    })
}
```