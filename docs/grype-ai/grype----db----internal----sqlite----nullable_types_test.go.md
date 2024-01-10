# `grype\grype\db\internal\sqlite\nullable_types_test.go`

```
// 导入 sqlite 包
package sqlite

// 导入测试包
import (
    "testing"

    // 导入断言包
    "github.com/stretchr/testify/assert"
)

// 定义 ToNullString 函数的测试用例
func TestToNullString(t *testing.T) {
    // 定义测试用例
    tests := []struct {
        name     string      // 测试用例名称
        input    any         // 输入值
        expected NullString  // 期望输出
    }

    // 遍历测试用例
    for _, test := range tests {
        // 运行单个测试用例
        t.Run(test.name, func(t *testing.T) {
            // 调用 ToNullString 函数，获取结果
            result := ToNullString(test.input)
            // 使用断言比较结果和期望值
            assert.Equal(t, test.expected, result)
        })
    }
}
```