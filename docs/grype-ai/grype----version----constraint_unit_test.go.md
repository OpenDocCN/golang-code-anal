# `grype\grype\version\constraint_unit_test.go`

```
package version

import (
    "reflect"  // 导入 reflect 包，用于深度比较
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/stretchr/testify/assert"  // 导入 testify 包，用于断言

)

func TestSplitFuzzyPhrase(t *testing.T) {
    tests := []struct {  // 定义测试用例结构体切片
        phrase   string         // 测试用例中的短语
        expected *constraintUnit  // 期望的约束单元
        err      bool            // 是否期望出现错误
    }

    for _, test := range tests {  // 遍历测试用例
        t.Run(test.phrase, func(t *testing.T) {  // 运行测试用例
            actual, err := parseUnit(test.phrase)  // 解析测试用例中的短语
            if err != nil && test.err == false {  // 如果出现错误但不期望出现错误
                t.Fatalf("expected no error, got %+v", err)  // 输出错误信息
            } else if err == nil && test.err {  // 如果没有出现错误但期望出现错误
                t.Fatalf("expected an error but did not get one")  // 输出错误信息
            }

            if !reflect.DeepEqual(test.expected, actual) {  // 深度比较期望的约束单元和实际结果
                t.Errorf("expected: '%+v' got: '%+v'", test.expected, actual)  // 输出比较结果
            }

        })
    }
}

func TestTrimQuotes(t *testing.T) {
    tests := []struct {  // 定义测试用例结构体切片
        name     string  // 测试用例的名称
        input    string  // 输入字符串
        expected string  // 期望的输出字符串
        err      bool    // 是否期望出现错误
    # 定义测试用例，包括输入、期望输出、是否预期出错
    {
        name:     "no quotes",
        input:    "test",
        expected: "test",
        err:      true,
    },
    {
        name:     "double quotes",
        input:    "\"test\"",
        expected: "test",
        err:      false,
    },
    {
        name:     "single quotes",
        input:    "'test'",
        expected: "test",
        err:      false,
    },
    {
        name:     "leading_single_quote",
        input:    "'test",
        expected: "'test",
        err:      true,
    },
    {
        name:     "trailing_single_quote",
        input:    "test'",
        expected: "test'",
        err:      true,
    },
    {
        name:     "leading_double_quote",
        input:    "'test",
        expected: "'test",
        err:      true,
    },
    {
        name:     "trailing_double_quote",
        input:    "test'",
        expected: "test'",
        err:      true,
    },
    {
        # 这会引发错误，但我认为这不是我们需要考虑的情况，所以应该没问题。
        name:     "nested double/double quotes",
        input:    "\"t\"es\"t\"",
        expected: "\"t\"es\"t\"",
        err:      true,
    },
    {
        name:     "nested single/single quotes",
        input:    "'t'es't'",
        expected: "t'es't",
        err:      false,
    },
    {
        name:     "nested single/double quotes",
        input:    "'t\"es\"t'",
        expected: "t\"es\"t",
        err:      false,
    },
    {
        name:     "nested double/single quotes",
        input:    "\"t'es't\"",
        expected: "t'es't",
        err:      false,
    },
}
    # 遍历测试用例数组
    for _, test := range tests {
        # 使用测试名称创建子测试，传入测试函数
        t.Run(test.name, func(t *testing.T) {
            # 调用 trimQuotes 函数，获取实际结果和可能的错误
            actual, err := trimQuotes(test.input)
            # 如果预期有错误
            if test.err {
                # 断言错误不为空，如果为空则输出错误信息
                assert.NotNil(t, err, "expected an error but did not get one")
            } else {
                # 断言错误为空，如果不为空则输出错误信息
                assert.Nil(t, err, "expected no error, got \"%+v\"", err)
            }
            # 断言实际结果和预期结果相等，如果不相等则输出错误信息
            assert.Equal(t, actual, test.expected, "output does not match expected: exp:%v got:%v", test.expected, actual)
        })
    }
# 闭合前面的函数定义
```