# `grype\internal\file\tar_test.go`

```
package file

import (
	"bytes"  // 导入 bytes 包，用于操作字节流
	"errors"  // 导入 errors 包，用于处理错误
	"fmt"  // 导入 fmt 包，用于格式化输出
	"strings"  // 导入 strings 包，用于处理字符串
	"testing"  // 导入 testing 包，用于编写测试用例

	"github.com/stretchr/testify/assert"  // 导入 testify 包中的 assert 模块，用于断言
)

func assertErrorAs(expectedErr interface{}) assert.ErrorAssertionFunc {
	return func(t assert.TestingT, actualErr error, i ...interface{}) bool {
		return errors.As(actualErr, &expectedErr)  // 断言 actualErr 是否为 expectedErr 类型的错误
	}
}

func TestSafeJoin(t *testing.T) {
	tests := []struct {  // 定义测试用例的数据结构
		prefix       string  // 定义一个字符串变量 prefix
		args         []string  // 定义一个字符串数组变量 args
		expected     string  // 定义一个字符串变量 expected
		errAssertion assert.ErrorAssertionFunc  // 定义一个错误断言函数变量 errAssertion
	}{
		// go cases...  // 注释：以下是测试用例
		{
			prefix: "/a/place",  // 设置 prefix 变量的值为 "/a/place"
			args: []string{  // 设置 args 变量的值为包含一个字符串 "/somewhere/else" 的字符串数组
				"somewhere/else",
			},
			expected:     "/a/place/somewhere/else",  // 设置 expected 变量的值为 "/a/place/somewhere/else"
			errAssertion: assert.NoError,  // 设置 errAssertion 变量的值为 assert.NoError，表示预期没有错误
		},
		{
			prefix: "/a/place",  // 设置 prefix 变量的值为 "/a/place"
			args: []string{  // 设置 args 变量的值为包含一个字符串 "/somewhere/../else" 的字符串数组
				"somewhere/../else",
			},
			expected:     "/a/place/else",  // 设置 expected 变量的值为 "/a/place/else"
// 定义一个测试用例，包括前缀、参数、期望结果和错误断言
{
    prefix: "/a/../place",
    args: []string{
        "somewhere/else",
    },
    expected:     "/place/somewhere/else",
    errAssertion: assert.NoError,
},
// zip slip examples....
// 定义一个测试用例，包括前缀、参数、期望结果和错误断言，用于测试 ZIP 文件解压时的安全性
{
    prefix: "/a/place",
    args: []string{
        "../../../etc/passwd",
    },
    expected:     "",
    errAssertion: assertErrorAs(&errZipSlipDetected{}),
},
		// 定义测试用例数组，每个测试用例包括前缀、参数、期望结果和错误断言
		tests := []struct {
			prefix:       "/a/place",
			// 参数为字符串切片，表示相对路径
			args:         []string{
				"../",
				"../",
			},
			// 期望结果为空字符串
			expected:     "",
			// 错误断言为检查是否为errZipSlipDetected类型的错误
			errAssertion: assertErrorAs(&errZipSlipDetected{}),
		},
		{
			prefix:       "/a/place",
			args:         []string{
				"../",
			},
			expected:     "",
			errAssertion: assertErrorAs(&errZipSlipDetected{}),
		}
	}

	// 遍历测试用例数组
	for _, test := range tests {
		// 使用测试前缀和参数创建子测试
		t.Run(fmt.Sprintf("%+v:%+v", test.prefix, test.args), func(t *testing.T) {
// 定义测试函数，用于测试 safeJoin 函数
func Test_safeJoin(t *testing.T) {
	// 定义测试用例
	tests := []struct {
		prefix       string
		args         []string
		expected     string
		errAssertion func(*testing.T, error)
	}{
		{
			prefix: "prefix",
			args:   []string{"arg1", "arg2"},
			expected: "prefix/arg1/arg2",
			errAssertion: func(t *testing.T, err error) {
				assert.NoError(t, err)
			},
		},
		// 添加更多测试用例...
	}

	// 遍历测试用例
	for _, test := range tests {
		// 执行 safeJoin 函数，获取返回值和错误
		actual, err := safeJoin(test.prefix, test.args...)
		// 断言错误
		test.errAssertion(t, err)
		// 断言返回值与期望值相等
		assert.Equal(t, test.expected, actual)
	}
}

// 定义测试函数，用于测试 copyWithLimits 函数
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
		{
			name:          "write bytes",
			input:         "something here",
			byteReadLimit: 1000,
			// 添加更多测试用例...
		},
		// 添加更多测试用例...
	}
	// 添加测试用例执行代码...
}
# 定义测试用例，包括名称、输入、字节读取限制、压缩文件中的路径、期望写入的内容、期望是否出现错误
{
    name:          "surpass upper limit",  # 测试用例名称
    input:         "something here",       # 输入内容
    byteReadLimit: 11,                     # 字节读取限制
    pathInArchive: "dont care",            # 压缩文件中的路径
    expectWritten: "something h",          # 期望写入的内容
    expectErr:     true,                   # 期望是否出现错误
},
{
    name:          "reach limit exactly",  # 测试用例名称
    input:         "something here",       # 输入内容
    byteReadLimit: 14,                     # 字节读取限制
    pathInArchive: "dont care",            # 压缩文件中的路径
    expectWritten: "something here",       # 期望写入的内容
    expectErr:     false,                  # 期望是否出现错误
}
# 定义测试用例结构体
type testCase struct {
    name           string
    input          string
    byteReadLimit  int64
    pathInArchive  string
    expectWritten  string
    expectErr      bool
}

# 定义测试用例数组
tests := []testCase{
    {
        name:           "test1",
        input:          "input data",
        byteReadLimit:  10,
        pathInArchive:  "test.txt",
        expectWritten:  "input data",
        expectErr:      false,
    },
    # 循环执行每个测试用例
    for _, test := range tests {
        # 在测试中运行 copyWithLimits 函数
        t.Run(test.name, func(t *testing.T) {
            # 创建一个字节缓冲区
            writer := &bytes.Buffer{}
            # 调用 copyWithLimits 函数，将输入字符串转换为字节流写入到缓冲区中
            err := copyWithLimits(writer, strings.NewReader(test.input), test.byteReadLimit, test.pathInArchive)
            # 检查返回的错误是否符合预期
            if (err != nil) != test.expectErr {
                t.Errorf("copyWithLimits() error = %v, want %v", err, test.expectErr)
                return
            } else if err != nil {
                # 断言错误信息中包含指定的路径
                assert.Contains(t, err.Error(), test.pathInArchive)
            }
            # 断言缓冲区中的内容与预期值相等
            assert.Equal(t, test.expectWritten, writer.String())
        })
    }
}
```