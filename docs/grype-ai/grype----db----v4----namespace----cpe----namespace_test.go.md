# `grype\grype\db\v4\namespace\cpe\namespace_test.go`

```
package cpe

import (
    "testing"  // 导入测试包
    "github.com/stretchr/testify/assert"  // 导入断言包
)

func TestFromString(t *testing.T) {
    successTests := []struct {  // 定义成功测试的结构体切片
        namespaceString string  // 命名空间字符串
        result          *Namespace  // 结果命名空间指针
    }{
        {
            namespaceString: "abc.xyz:cpe",  // 命名空间字符串
            result:          NewNamespace("abc.xyz"),  // 创建新的命名空间对象
        },
    }

    for _, test := range successTests {  // 遍历成功测试
        result, _ := FromString(test.namespaceString)  // 从字符串创建命名空间对象
        assert.Equal(t, result, test.result)  // 使用断言比较结果和预期值
    }

    errorTests := []struct {  // 定义错误测试的结构体切片
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

    for _, test := range errorTests {  // 遍历错误测试
        _, err := FromString(test.namespaceString)  // 从字符串创建命名空间对象
        assert.EqualError(t, err, test.errorMessage)  // 使用断言比较错误消息
    }
}
```