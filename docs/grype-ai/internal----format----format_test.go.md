# `grype\internal\format\format_test.go`

```go
package format

import (
    "testing"  // 导入测试包
    "github.com/stretchr/testify/assert"  // 导入断言包
)

func TestParse(t *testing.T) {  // 定义测试函数
    cases := []struct {  // 定义测试用例结构体切片
        input    string  // 输入字符串
        expected Format  // 期望的格式
    }{
        {  // 第一个测试用例
            "",  // 空字符串
            TableFormat,  // 期望的格式为表格格式
        },
        {  // 第二个测试用例
            "table",  // "table"字符串
            TableFormat,  // 期望的格式为表格格式
        },
        {  // 第三个测试用例
            "jSOn",  // "jSOn"字符串
            JSONFormat,  // 期望的格式为JSON格式
        },
        {  // 第四个测试用例
            "booboodepoopoo",  // "booboodepoopoo"字符串
            UnknownFormat,  // 期望的格式为未知格式
        },
    }

    for _, tc := range cases {  // 遍历测试用例
        t.Run(tc.input, func(t *testing.T) {  // 运行测试用例
            actual := Parse(tc.input)  // 调用Parse函数获取实际结果
            assert.Equal(t, tc.expected, actual, "unexpected result for input %q", tc.input)  // 使用断言比较实际结果和期望结果
        })
    }
}
```