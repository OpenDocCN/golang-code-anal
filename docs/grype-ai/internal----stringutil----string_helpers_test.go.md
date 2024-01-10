# `grype\internal\stringutil\string_helpers_test.go`

```
package stringutil

import (
    "testing"  // 导入测试包
    "github.com/stretchr/testify/assert"  // 导入断言包
)

func TestHasAnyOfSuffixes(t *testing.T) {  // 定义测试函数
    tests := []struct {  // 定义测试用例结构体切片
        name     string  // 测试用例名称
        input    string  // 输入字符串
        suffixes []string  // 后缀切片
        expected bool  // 期望结果
    }{  // 测试用例列表
        {
            name:  "go case",  // 测试用例名称
            input: "this has something",  // 输入字符串
            suffixes: []string{  // 后缀切片
                "has something",  // 后缀1
                "has NOT something",  // 后缀2
            },
            expected: true,  // 期望结果
        },
        {
            name:  "no match",  // 测试用例名称
            input: "this has something",  // 输入字符串
            suffixes: []string{  // 后缀切片
                "has NOT something",  // 后缀
            },
            expected: false,  // 期望结果
        },
        {
            name:     "empty",  // 测试用例名称
            input:    "this has something",  // 输入字符串
            suffixes: []string{},  // 空后缀切片
            expected: false,  // 期望结果
        },
        {
            name:  "positive match last",  // 测试用例名称
            input: "this has something",  // 输入字符串
            suffixes: []string{  // 后缀切片
                "that does not have",  // 后缀1
                "something",  // 后缀2
            },
            expected: true,  // 期望结果
        },
        {
            name:  "empty input",  // 测试用例名称
            input: "",  // 空输入字符串
            suffixes: []string{  // 后缀切片
                "that does not have",  // 后缀1
                "this has",  // 后缀2
            },
            expected: false,  // 期望结果
        },
    }

    for _, test := range tests {  // 遍历测试用例
        t.Run(test.name, func(t *testing.T) {  // 运行测试用例
            assert.Equal(t, test.expected, HasAnyOfSuffixes(test.input, test.suffixes...))  // 使用断言判断测试结果
        })
    }
}

func TestHasAnyOfPrefixes(t *testing.T) {  // 定义测试函数
    tests := []struct {  // 定义测试用例结构体切片
        name     string  // 测试用例名称
        input    string  // 输入字符串
        prefixes []string  // 前缀切片
        expected bool  // 期望结果
    # 测试用例列表，包含多个测试用例
    {
        # 测试用例1：名称为"go case"，输入为"this has something"，前缀列表包含两个字符串
        name:  "go case",
        input: "this has something",
        prefixes: []string{
            "this has",
            "that does not have",
        },
        expected: true,
    },
    {
        # 测试用例2：名称为"no match"，输入为"this has something"，前缀列表包含两个字符串
        name:  "no match",
        input: "this has something",
        prefixes: []string{
            "this DOES NOT has",
            "that does not have",
        },
        expected: false,
    },
    {
        # 测试用例3：名称为"empty"，输入为"this has something"，前缀列表为空
        name:     "empty",
        input:    "this has something",
        prefixes: []string{},
        expected: false,
    },
    {
        # 测试用例4：名称为"positive match last"，输入为"this has something"，前缀列表包含两个字符串
        name:  "positive match last",
        input: "this has something",
        prefixes: []string{
            "that does not have",
            "this has",
        },
        expected: true,
    },
    {
        # 测试用例5：名称为"empty input"，输入为空字符串，前缀列表包含两个字符串
        name:  "empty input",
        input: "",
        prefixes: []string{
            "that does not have",
            "this has",
        },
        expected: false,
    }
    }

    # 遍历测试用例列表，对每个测试用例进行测试
    for _, test := range tests {
        # 使用测试框架运行测试用例，并断言结果是否符合预期
        t.Run(test.name, func(t *testing.T) {
            assert.Equal(t, test.expected, HasAnyOfPrefixes(test.input, test.prefixes...))
        })
    }
func TestSplitCommaSeparatedString(t *testing.T) {
    // 定义测试用例，包括输入字符串和期望的结果数组
    tests := []struct {
        input    string
        expected []string
    }{
        {
            input:    "testing",
            expected: []string{"testing"},
        },
        {
            input:    "",
            expected: []string{},
        },
        {
            input:    "testing1,testing2",
            expected: []string{"testing1", "testing2"},
        },
        {
            input:    "testing1,,testing2,testing3",
            expected: []string{"testing1", "testing2", "testing3"},
        },
        {
            input:    "testing1,testing2,,",
            expected: []string{"testing1", "testing2"},
        },
    }

    // 遍历测试用例
    for _, test := range tests {
        // 对每个测试用例运行子测试
        t.Run(test.input, func(t *testing.T) {
            // 断言测试结果是否符合预期
            assert.Equal(t, test.expected, SplitCommaSeparatedString(test.input))
        })
    }
}
```