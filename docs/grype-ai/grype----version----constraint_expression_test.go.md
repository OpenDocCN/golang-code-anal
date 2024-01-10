# `grype\grype\version\constraint_expression_test.go`

```
package version

import (
    "testing"

    "github.com/go-test/deep"
)

func TestScanExpression(t *testing.T) {
    // 定义测试用例
    tests := []struct {
        phrase   string    // 输入的表达式
        expected [][]string  // 期望的结果
        err      bool        // 是否期望出错
    }{
        {
            phrase: "x,y||z",  // 输入表达式
            expected: [][]string{  // 期望的结果
                {
                    "x",
                    "y",
                },
                {
                    "z",
                },
            },
        },
        {
            phrase: "<1.0, >=2.0|| 3.0 || =4.0",  // 输入表达式
            expected: [][]string{  // 期望的结果
                {
                    "<1.0",
                    ">=2.0",
                },
                {
                    "3.0",
                },
                {
                    "=4.0",
                },
            },
        },
        {
            // parenthetical expression are not supported yet
            phrase: "(<1.0, >=2.0|| 3.0) || =4.0",  // 输入表达式
            err:    true,  // 期望出错
        },
        {
            phrase: ` > 1.0,  <=   2.0,,,    || = 3.0 `,  // 输入表达式
            expected: [][]string{  // 期望的结果
                {
                    ">1.0",
                    "<=2.0",
                },
                {
                    "=3.0",
                },
            },
        },
        {
            phrase: ` > 1.0,  <= "  (2.0||),,, ",   || = 3.0 `,  // 输入表达式
            expected: [][]string{  // 期望的结果
                {
                    ">1.0",
                    `<="  (2.0||),,, "`,
                },
                {
                    "=3.0",
                },
            },
        },
    }
    # 遍历测试用例数组，每个测试用例包含一个短语和期望的结果
    for _, test := range tests {
        # 使用测试短语运行子测试，每个子测试包含一个独立的测试函数
        t.Run(test.phrase, func(t *testing.T) {
            # 调用扫描表达式函数，获取实际结果和可能的错误
            actual, err := scanExpression(test.phrase)
            # 如果有错误且测试用例中不期望错误，则输出错误信息
            if err != nil && test.err == false {
                t.Fatalf("expected no error, got %+v", err)
            # 如果没有错误且测试用例中期望错误，则输出错误信息
            } else if err == nil && test.err {
                t.Fatalf("expected an error but did not get one")
            }

            # 比较期望结果和实际结果，输出不同之处
            for _, d := range deep.Equal(test.expected, actual) {
                t.Errorf("difference: %+v", d)
            }

        })
    }
# 闭合前面的函数定义
```