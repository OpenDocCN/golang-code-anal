# `grype\grype\db\v5\namespace\language\namespace_test.go`

```
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
			namespaceString: "github:language:python",  // 设置命名空间字符串
			result:          NewNamespace("github", syftPkg.Python, ""),  // 设置结果命名空间
		},
		{
# 创建一个包含 namespaceString 和 result 的对象数组
[
    {
        # 设置 namespaceString 为 "github:language:ruby"，result 为包含 "github" 命名空间和 Ruby 语言的新命名空间对象
        namespaceString: "github:language:ruby",
        result:          NewNamespace("github", syftPkg.Ruby, ""),
    },
    {
        # 设置 namespaceString 为 "github:language:java"，result 为包含 "github" 命名空间和 Java 语言的新命名空间对象
        namespaceString: "github:language:java",
        result:          NewNamespace("github", syftPkg.Java, ""),
    },
    {
        # 设置 namespaceString 为 "github:language:rust"，result 为包含 "github" 命名空间和 Rust 语言的新命名空间对象
        namespaceString: "github:language:rust",
        result:          NewNamespace("github", syftPkg.Rust, ""),
    },
    {
        # 设置 namespaceString 为 "abc.xyz:language:something"，result 为包含 "abc.xyz" 命名空间和 "something" 语言的新命名空间对象
        namespaceString: "abc.xyz:language:something",
        result:          NewNamespace("abc.xyz", syftPkg.Language("something"), ""),
    },
    {
        # 设置 namespaceString 为 "abc.xyz:language:something:another-package-manager"，result 为包含 "abc.xyz" 命名空间、"something" 语言和 "another-package-manager" 类型的新命名空间对象
        namespaceString: "abc.xyz:language:something:another-package-manager",
        result:          NewNamespace("abc.xyz", syftPkg.Language("something"), syftPkg.Type("another-package-manager")),
    },
]
# 遍历成功测试用例列表
for _, test := range successTests:
    # 从字符串创建命名空间对象，并获取结果
    result, _ := FromString(test.namespaceString)
    # 断言结果与预期结果相等
    assert.Equal(t, result, test.result)

# 错误测试用例列表
errorTests := []struct {
    namespaceString string
    errorMessage    string
}{
    # 空字符串作为命名空间字符串，预期出错信息
    {
        namespaceString: "",
        errorMessage:    "unable to create language namespace from empty string",
    },
    # 单组件字符串作为命名空间字符串，预期出错信息
    {
        namespaceString: "single-component",
        errorMessage:    "unable to create language namespace from single-component: incorrect number of components",
    },
    # 两组件字符串作为命名空间字符串
# 定义测试用例，包括命名空间字符串和预期错误信息
errorTests = [
    {
        namespaceString: "two:components",
        errorMessage: "unable to create language namespace from two:components: incorrect number of components",
    },
    {
        namespaceString: "too:many:components:a:b",
        errorMessage: "unable to create language namespace from too:many:components:a:b: incorrect number of components",
    },
    {
        namespaceString: "wrong:namespace_type:a:b",
        errorMessage: "unable to create language namespace from wrong:namespace_type:a:b: type namespace_type is incorrect",
    },
]

# 遍历测试用例列表
for _, test := range errorTests:
    # 调用FromString函数，将命名空间字符串转换为命名空间对象，并捕获返回的错误信息
    _, err := FromString(test.namespaceString)
    # 使用断言检查返回的错误信息是否符合预期
    assert.EqualError(t, err, test.errorMessage)
}
```