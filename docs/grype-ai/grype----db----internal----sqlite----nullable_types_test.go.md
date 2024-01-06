# `grype\grype\db\internal\sqlite\nullable_types_test.go`

```
package sqlite

import (
	"testing"  // 导入测试包
	"github.com/stretchr/testify/assert"  // 导入断言包
)

func TestToNullString(t *testing.T) {
	tests := []struct {  // 定义测试用例结构体
		name     string  // 测试用例名称
		input    any  // 输入参数
		expected NullString  // 期望输出
	}{
		{
			name:     "Nil input",  // 测试用例1：空输入
			input:    nil,  // 输入为空
			expected: NullString{},  // 期望输出为空字符串
		},
		{
# 定义测试用例，包括名称、输入和期望输出
{
    name:     "String null",  # 测试用例名称
    input:    "null",  # 输入值为 null
    expected: NullString{},  # 期望输出为空字符串
},
{
    name:     "Other string",  # 测试用例名称
    input:    "Hello there {}",  # 输入值为 "Hello there {}"
    expected: NewNullString("Hello there {}", true),  # 期望输出为包含输入值的新字符串对象
},
{
    name: "Single struct with all fields populated",  # 测试用例名称
    input: struct {  # 输入值为包含各种类型字段的结构体
        Boolean     bool   `json:"boolean"`  # 布尔类型字段
        String      string `json:"string"`  # 字符串类型字段
        Integer     int    `json:"integer"`  # 整数类型字段
        InnerStruct struct {  # 嵌套结构体
            StringList []string `json:"string_list"`  # 字符串列表类型字段
        } `json:"inner_struct"`
    }{
        Boolean: true,  # 布尔类型字段值为 true
			// 设置一个空字符串
			String:  "{}",
			// 设置一个整数
			Integer: 1034,
			// 设置一个内部结构体，包含一个字符串列表
			InnerStruct: struct {
				StringList []string `json:"string_list"`
			}{
				// 设置字符串列表的值
				StringList: []string{"a", "b", "c"},
			},
		},
		// 期望的结果，包含一个空字符串、一个整数和一个内部结构体
		expected: NewNullString(`{"boolean":true,"string":"{}","integer":1034,"inner_struct":{"string_list":["a","b","c"]}}`, true),
	},
	{
		// 测试用例名称
		name: "Single struct with one field populated",
		// 输入的结构体，包含布尔值、字符串、整数和内部结构体
		input: struct {
			Boolean     bool   `json:"boolean"`
			String      string `json:"string"`
			Integer     int    `json:"integer"`
			InnerStruct struct {
				StringList []string `json:"string_list"`
			} `json:"inner_struct"`
		}{
		{
			name: "Single struct with all fields populated",
			// 定义一个结构体，包含布尔型、字符串、整型和内部结构体字段
			input: struct {
				Boolean     bool   `json:"boolean,omitempty"` // 定义布尔型字段，并指定在 JSON 中如果为空则忽略
				String      string `json:"string,omitempty"`  // 定义字符串字段，并指定在 JSON 中如果为空则忽略
				Integer     int    `json:"integer,omitempty"` // 定义整型字段，并指定在 JSON 中如果为空则忽略
				InnerStruct struct { // 定义内部结构体字段
					StringList []string `json:"string_list,omitempty"` // 定义字符串数组字段，并指定在 JSON 中如果为空则忽略
				} `json:"inner_struct,omitempty"` // 指定内部结构体在 JSON 中的字段名，并指定如果为空则忽略
			}{
				Boolean: true, // 初始化布尔型字段为 true
			},
			// 期望的 JSON 字符串，包含布尔型字段为 true，内部结构体为空
			expected: NewNullString(`{"boolean":true,"string":"","integer":0,"inner_struct":{"string_list":null}}`, true),
		},
		{
			name: "Single struct with one field populated omit empty",
			// 定义一个结构体，包含布尔型和内部结构体字段
			input: struct {
				Boolean     bool   `json:"boolean,omitempty"` // 定义布尔型字段，并指定在 JSON 中如果为空则忽略
				String      string `json:"string,omitempty"`  // 定义字符串字段，并指定在 JSON 中如果为空则忽略
				Integer     int    `json:"integer,omitempty"` // 定义整型字段，并指定在 JSON 中如果为空则忽略
				InnerStruct struct { // 定义内部结构体字段
					StringList []string `json:"string_list,omitempty"` // 定义字符串数组字段，并指定在 JSON 中如果为空则忽略
				} `json:"inner_struct,omitempty"` // 指定内部结构体在 JSON 中的字段名，并指定如果为空则忽略
			}{
				Boolean: true, // 初始化布尔型字段为 true
			},
			// 期望的 JSON 字符串，包含布尔型字段为 true，内部结构体为空
			expected: NewNullString(`{"boolean":true,"inner_struct":{}}`, true),
		},
		{
			name: "Array of structs",
# 定义一个输入结构，包含布尔值、字符串和整数三种类型的字段
input: []struct {
    Boolean bool   `json:"boolean,omitempty"`  # 布尔值字段，json标签指定为boolean，omitempty表示在json中为空时忽略该字段
    String  string `json:"string,omitempty"`   # 字符串字段，json标签指定为string，omitempty表示在json中为空时忽略该字段
    Integer int    `json:"integer,omitempty"`  # 整数字段，json标签指定为integer，omitempty表示在json中为空时忽略该字段
}{
    # 第一个结构体实例
    {
        Boolean: true,    # 布尔值为true
        String:  "{}",     # 字符串为"{}"
        Integer: 1034,    # 整数为1034
    },
    # 第二个结构体实例
    {
        String: "[{}]",   # 字符串为"[{}]"
    },
    # 第三个结构体实例
    {
        Integer: -5000,   # 整数为-5000
        Boolean: false,    # 布尔值为false
    },
},
expected: NewNullString(`[{"boolean":true,"string":"{}","integer":1034},{"string":"[{}]"},{"integer":-5000}]`, true),  # 期望的输出结果，使用NewNullString函数创建一个包含指定json字符串的对象
# 遍历测试用例列表
for _, test := range tests:
    # 使用测试名称创建子测试
    t.Run(test.name, func(t *testing.T):
        # 调用ToNullString函数，获取结果
        result := ToNullString(test.input)
        # 断言结果与期望值相等
        assert.Equal(t, test.expected, result)
    )
# 结束遍历
```