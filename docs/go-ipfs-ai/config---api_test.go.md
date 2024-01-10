# `kubo\config\api_test.go`

```
package config

import (
    "testing"  // 导入测试包
    "github.com/stretchr/testify/assert"  // 导入断言包
)

func TestConvertAuthSecret(t *testing.T) {
    // 遍历测试用例
    for _, testCase := range []struct {
        input  string   // 输入字符串
        output string   // 期望输出字符串
    }{
        {"", ""},  // 空字符串测试用例
        {"someToken", "Bearer someToken"},  // 普通字符串测试用例
        {"bearer:someToken", "Bearer someToken"},  // 带前缀的字符串测试用例
        {"basic:user:pass", "Basic dXNlcjpwYXNz"},  // 基本认证字符串测试用例
        {"basic:dXNlcjpwYXNz", "Basic dXNlcjpwYXNz"},  // 带前缀的基本认证字符串测试用例
    } {
        // 使用断言判断转换后的输出是否符合预期
        assert.Equal(t, testCase.output, ConvertAuthSecret(testCase.input))
    }
}
```