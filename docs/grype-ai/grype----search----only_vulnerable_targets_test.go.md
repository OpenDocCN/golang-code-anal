# `grype\grype\search\only_vulnerable_targets_test.go`

```
package search

import (
	"testing"  // 导入测试包
	"github.com/stretchr/testify/assert"  // 导入断言包
)

func Test_isUnknownTarget(t *testing.T) {  // 定义测试函数
	tests := []struct {  // 定义测试用例
		name     string  // 测试用例名称
		targetSW string  // 目标软件名称
		expected bool    // 期望结果
	}{
		{name: "supported syft language", targetSW: "python", expected: false},  // 测试用例1
		{name: "supported non-syft language CPE component", targetSW: "wordpress", expected: false},  // 测试用例2
		{name: "unknown component", targetSW: "abc", expected: true},  // 测试用例3
	}

	for _, test := range tests {  // 遍历测试用例
# 对测试用例进行运行，并使用断言检查结果
t.Run(test.name, func(t *testing.T) {
    # 检查测试目标是否为未知目标软件
    u := isUnknownTarget(test.targetSW)
    # 使用断言检查实际结果与期望结果是否相等
    assert.Equal(t, test.expected, u)
})
```