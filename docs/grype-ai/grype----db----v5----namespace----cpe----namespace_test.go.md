# `grype\grype\db\v5\namespace\cpe\namespace_test.go`

```go
package cpe

import (
    "testing"  // 导入测试包
    "github.com/stretchr/testify/assert"  // 导入断言包
)

func TestFromString(t *testing.T) {
    successTests := []struct {  // 定义成功测试用例结构体切片
        namespaceString string  // 命名空间字符串
        result          *Namespace  // 结果命名空间指针
    }{
        {
            namespaceString: "abc.xyz:cpe",  // 命名空间字符串
            result:          NewNamespace("abc.xyz"),  // 创建新的命名空间对象
        },
    }

    for _, test := range successTests {  // 遍历成功测试用例
        result, _ := FromString(test.namespaceString)  // 调用 FromString 方法解析命名空间字符串
        assert.Equal(t, result, test.result)  // 使用断言判断结果是否符合预期
    }

    errorTests := []struct {  // 定义失败测试用例结构体切片
        namespaceString string  // 命名空间字符串
        errorMessage    string  // 错误消息
    }{
        {
            namespaceString: "",  // 空字符串
            errorMessage:    "unable to create CPE namespace from empty string",  // 错误消息
        },
        {
            namespaceString: "single-component",  // 单组件字符串
            errorMessage:    "unable to create CPE namespace from single-component: incorrect number of components",  // 错误消息
        },
        {
            namespaceString: "too:many:components",  // 多组件字符串
            errorMessage:    "unable to create CPE namespace from too:many:components: incorrect number of components",  // 错误消息
        },
        {
            namespaceString: "wrong:namespace_type",  // 错误的命名空间类型
            errorMessage:    "unable to create CPE namespace from wrong:namespace_type: type namespace_type is incorrect",  // 错误消息
        },
    }

    for _, test := range errorTests {  // 遍历失败测试用例
        _, err := FromString(test.namespaceString)  // 调用 FromString 方法解析命名空间字符串
        assert.EqualError(t, err, test.errorMessage)  // 使用断言判断错误消息是否符合预期
    }
}
```