# `grype\grype\search\only_vulnerable_targets_test.go`

```go
package search

import (
    "testing"  // 导入测试包
    "github.com/stretchr/testify/assert"  // 导入断言包
)

func Test_isUnknownTarget(t *testing.T) {  // 定义测试函数
    tests := []struct {  // 定义测试用例结构体切片
        name     string  // 测试用例名称
        targetSW string  // 目标软件名称
        expected bool    // 期望结果
    }{
        {name: "supported syft language", targetSW: "python", expected: false},  // 第一个测试用例
        {name: "supported non-syft language CPE component", targetSW: "wordpress", expected: false},  // 第二个测试用例
        {name: "unknown component", targetSW: "abc", expected: true},  // 第三个测试用例
    }

    for _, test := range tests {  // 遍历测试用例
        t.Run(test.name, func(t *testing.T) {  // 运行测试用例
            u := isUnknownTarget(test.targetSW)  // 调用被测函数
            assert.Equal(t, test.expected, u)  // 使用断言判断测试结果
        })
    }
}
```