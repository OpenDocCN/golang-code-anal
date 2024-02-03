# `grype\grype\db\v4\namespace\language\namespace_test.go`

```go
package language

import (
    "testing"  // 导入测试包
    "github.com/stretchr/testify/assert"  // 导入断言包
    syftPkg "github.com/anchore/syft/syft/pkg"  // 导入syftPkg包
)

func TestFromString(t *testing.T) {  // 定义测试函数
    successTests := []struct {  // 定义成功测试用例结构体切片
        namespaceString string  // 命名空间字符串
        result          *Namespace  // 结果命名空间指针
    }{
        {
            namespaceString: "github:language:python",  // 命名空间字符串为github:language:python
            result:          NewNamespace("github", syftPkg.Python, ""),  // 创建新的命名空间对象
        },
        {
            namespaceString: "github:language:ruby",  // 命名空间字符串为github:language:ruby
            result:          NewNamespace("github", syftPkg.Ruby, ""),  // 创建新的命名空间对象
        },
        {
            namespaceString: "github:language:java",  // 命名空间字符串为github:language:java
            result:          NewNamespace("github", syftPkg.Java, ""),  // 创建新的命名空间对象
        },
        {
            namespaceString: "abc.xyz:language:something",  // 命名空间字符串为abc.xyz:language:something
            result:          NewNamespace("abc.xyz", syftPkg.Language("something"), ""),  // 创建新的命名空间对象
        },
        {
            namespaceString: "abc.xyz:language:something:another-package-manager",  // 命名空间字符串为abc.xyz:language:something:another-package-manager
            result:          NewNamespace("abc.xyz", syftPkg.Language("something"), syftPkg.Type("another-package-manager")),  // 创建新的命名空间对象
        },
    }

    for _, test := range successTests {  // 遍历成功测试用例
        result, _ := FromString(test.namespaceString)  // 调用FromString函数
        assert.Equal(t, result, test.result)  // 使用断言判断结果是否相等
    }

    errorTests := []struct {  // 定义错误测试用例结构体切片
        namespaceString string  // 命名空间字符串
        errorMessage    string  // 错误信息
    # 定义包含错误测试数据的数组
    errorTests := []struct {
        namespaceString string
        errorMessage    string
    }{
        # 第一个错误测试数据，空字符串
        {
            namespaceString: "",
            errorMessage:    "unable to create language namespace from empty string",
        },
        # 第二个错误测试数据，单个组件的字符串
        {
            namespaceString: "single-component",
            errorMessage:    "unable to create language namespace from single-component: incorrect number of components",
        },
        # 第三个错误测试数据，两个组件的字符串
        {
            namespaceString: "two:components",
            errorMessage:    "unable to create language namespace from two:components: incorrect number of components",
        },
        # 第四个错误测试数据，过多组件的字符串
        {
            namespaceString: "too:many:components:a:b",
            errorMessage:    "unable to create language namespace from too:many:components:a:b: incorrect number of components",
        },
        # 第五个错误测试数据，错误的命名空间类型
        {
            namespaceString: "wrong:namespace_type:a:b",
            errorMessage:    "unable to create language namespace from wrong:namespace_type:a:b: type namespace_type is incorrect",
        },
    }

    # 遍历错误测试数据数组
    for _, test := range errorTests {
        # 使用错误测试数据中的字符串创建语言命名空间
        _, err := FromString(test.namespaceString)
        # 断言返回的错误信息与预期的错误信息相等
        assert.EqualError(t, err, test.errorMessage)
    }
# 闭合前面的函数定义
```