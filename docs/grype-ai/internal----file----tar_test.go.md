# `grype\internal\file\tar_test.go`

```go
package file

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "errors"  // 导入 errors 包，用于处理错误
    "fmt"  // 导入 fmt 包，用于格式化输出
    "strings"  // 导入 strings 包，用于字符串操作
    "testing"  // 导入 testing 包，用于编写测试用例

    "github.com/stretchr/testify/assert"  // 导入 testify/assert 包，用于断言
)

// 定义一个函数，返回一个错误断言函数
func assertErrorAs(expectedErr interface{}) assert.ErrorAssertionFunc {
    return func(t assert.TestingT, actualErr error, i ...interface{}) bool {
        return errors.As(actualErr, &expectedErr)
    }
}

// 定义测试函数 TestSafeJoin
func TestSafeJoin(t *testing.T) {
    // 定义测试用例
    tests := []struct {
        prefix       string  // 前缀字符串
        args         []string  // 参数字符串数组
        expected     string  // 期望的结果字符串
        errAssertion assert.ErrorAssertionFunc  // 错误断言函数
    }{
        // go cases...
        {
            prefix: "/a/place",  // 设置前缀
            args: []string{
                "somewhere/else",  // 设置参数
            },
            expected:     "/a/place/somewhere/else",  // 设置期望的结果
            errAssertion: assert.NoError,  // 设置错误断言函数
        },
        {
            prefix: "/a/place",  // 设置前缀
            args: []string{
                "somewhere/../else",  // 设置参数
            },
            expected:     "/a/place/else",  // 设置期望的结果
            errAssertion: assert.NoError,  // 设置错误断言函数
        },
        {
            prefix: "/a/../place",  // 设置前缀
            args: []string{
                "somewhere/else",  // 设置参数
            },
            expected:     "/place/somewhere/else",  // 设置期望的结果
            errAssertion: assert.NoError,  // 设置错误断言函数
        },
        // zip slip examples....
        {
            prefix: "/a/place",  // 设置前缀
            args: []string{
                "../../../etc/passwd",  // 设置参数
            },
            expected:     "",  // 设置期望的结果
            errAssertion: assertErrorAs(&errZipSlipDetected{}),  // 设置错误断言函数
        },
        {
            prefix: "/a/place",  // 设置前缀
            args: []string{
                "../",  // 设置参数
                "../",  // 设置参数
            },
            expected:     "",  // 设置期望的结果
            errAssertion: assertErrorAs(&errZipSlipDetected{}),  // 设置错误断言函数
        },
        {
            prefix: "/a/place",  // 设置前缀
            args: []string{
                "../",  // 设置参数
            },
            expected:     "",  // 设置期望的结果
            errAssertion: assertErrorAs(&errZipSlipDetected{}),  // 设置错误断言函数
        },
    }
}
    # 遍历测试用例列表
    for _, test := range tests {
        # 使用测试用例的前缀和参数运行子测试
        t.Run(fmt.Sprintf("%+v:%+v", test.prefix, test.args), func(t *testing.T) {
            # 调用 safeJoin 函数，获取返回值和错误信息
            actual, err := safeJoin(test.prefix, test.args...)
            # 对错误信息进行断言
            test.errAssertion(t, err)
            # 对返回值和期望值进行断言
            assert.Equal(t, test.expected, actual)
        })
    }
func Test_copyWithLimits(t *testing.T) {
    // 定义测试用例
    tests := []struct {
        name          string
        input         string
        byteReadLimit int64
        pathInArchive string
        expectWritten string
        expectErr     bool
    }{
        // 第一个测试用例
        {
            name:          "write bytes",
            input:         "something here",
            byteReadLimit: 1000,
            pathInArchive: "dont care",
            expectWritten: "something here",
            expectErr:     false,
        },
        // 第二个测试用例
        {
            name:          "surpass upper limit",
            input:         "something here",
            byteReadLimit: 11,
            pathInArchive: "dont care",
            expectWritten: "something h",
            expectErr:     true,
        },
        // 第三个测试用例
        {
            name:          "reach limit exactly",
            input:         "something here",
            byteReadLimit: 14,
            pathInArchive: "dont care",
            expectWritten: "something here",
            expectErr:     true,
        },
    }
    // 遍历测试用例
    for _, test := range tests {
        // 运行测试用例
        t.Run(test.name, func(t *testing.T) {
            // 创建一个字节缓冲区
            writer := &bytes.Buffer{}
            // 调用 copyWithLimits 函数，将输入字符串的内容复制到缓冲区中，并限制读取的字节数
            err := copyWithLimits(writer, strings.NewReader(test.input), test.byteReadLimit, test.pathInArchive)
            // 检查是否返回了预期的错误
            if (err != nil) != test.expectErr {
                t.Errorf("copyWithLimits() error = %v, want %v", err, test.expectErr)
                return
            } else if err != nil {
                // 断言错误信息中包含指定的路径
                assert.Contains(t, err.Error(), test.pathInArchive)
            }
            // 断言缓冲区中的内容与预期的相符
            assert.Equal(t, test.expectWritten, writer.String())

        })
    }
}
```