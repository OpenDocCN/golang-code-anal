# `grype\grype\presenter\models\ignore_test.go`

```go
package models

import (
    "testing"  // 导入测试包

    "github.com/google/go-cmp/cmp"  // 导入用于比较的包

    grypeDb "github.com/anchore/grype/grype/db/v5"  // 导入数据库包
    "github.com/anchore/grype/grype/match"  // 导入匹配包
)

func TestNewIgnoreRule(t *testing.T) {
    cases := []struct {  // 定义测试用例结构
        name     string
        input    match.IgnoreRule
        expected IgnoreRule
    }

    for _, testCase := range cases {  // 遍历测试用例
        t.Run(testCase.name, func(t *testing.T) {  // 运行测试用例
            actual := newIgnoreRule(testCase.input)  // 调用函数获取实际结果
            if diff := cmp.Diff(testCase.expected, actual); diff != "" {  // 比较期望结果和实际结果
                t.Errorf("(-expected +actual):\n%s", diff)  // 输出比较结果的差异
            }
        })
    }
}
```