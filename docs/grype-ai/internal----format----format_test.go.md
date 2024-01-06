# `grype\internal\format\format_test.go`

```
package format  // 声明代码所属的包

import (
	"testing"  // 导入测试包
	"github.com/stretchr/testify/assert"  // 导入断言包
)

func TestParse(t *testing.T) {  // 定义测试函数
	cases := []struct {  // 定义测试用例
		input    string  // 输入字符串
		expected Format  // 期望的格式
	}{
		{  // 第一个测试用例
			"",  // 空字符串作为输入
			TableFormat,  // 期望的格式为表格格式
		},
		{  // 第二个测试用例
			"table",  // "table" 字符串作为输入
			TableFormat,  // 期望的格式为表格格式
# 定义测试用例数组，每个测试用例包含输入和期望输出
cases := []struct {
    input    string  // 输入字符串
    expected Format  // 期望的输出格式
}{
    {
        "xml",        // 输入为 "xml"
        XMLFormat,    // 期望输出为 XML 格式
    },
    {
        "json",       // 输入为 "json"
        JSONFormat,   // 期望输出为 JSON 格式
    },
    {
        "booboodepoopoo",  // 输入为 "booboodepoopoo"
        UnknownFormat,     // 期望输出为未知格式
    },
}

# 遍历测试用例数组，对每个测试用例进行测试
for _, tc := range cases {
    # 使用 t.Run 方法运行测试用例，并在测试失败时输出错误信息
    t.Run(tc.input, func(t *testing.T) {
        actual := Parse(tc.input)  // 调用 Parse 方法获取实际输出
        assert.Equal(t, tc.expected, actual, "unexpected result for input %q", tc.input)  // 断言实际输出与期望输出是否相等
    })
}
```