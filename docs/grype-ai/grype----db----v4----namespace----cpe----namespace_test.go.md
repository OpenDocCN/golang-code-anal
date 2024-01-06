# `grype\grype\db\v4\namespace\cpe\namespace_test.go`

```
package cpe

import (
	"testing"  // 导入测试包
	"github.com/stretchr/testify/assert"  // 导入断言包
)

func TestFromString(t *testing.T) {  // 定义测试函数
	successTests := []struct {  // 定义成功测试用例的结构体切片
		namespaceString string  // 命名空间字符串
		result          *Namespace  // 结果命名空间指针
	}{
		{
			namespaceString: "abc.xyz:cpe",  // 设置命名空间字符串
			result:          NewNamespace("abc.xyz"),  // 设置预期结果
		},
	}

	for _, test := range successTests {  // 遍历成功测试用例
	// 从字符串中解析出结果，并与预期结果进行比较
	result, _ := FromString(test.namespaceString)
	assert.Equal(t, result, test.result)
	}

	// 错误测试用例，包括命名空间字符串和预期的错误信息
	errorTests := []struct {
		namespaceString string
		errorMessage    string
	}{
		{
			namespaceString: "",
			errorMessage:    "unable to create CPE namespace from empty string",
		},
		{
			namespaceString: "single-component",
			errorMessage:    "unable to create CPE namespace from single-component: incorrect number of components",
		},
		{
			namespaceString: "too:many:components",
			errorMessage:    "unable to create CPE namespace from too:many:components: incorrect number of components",
		},
```
# 定义一个包含命名空间字符串和错误消息的测试数据结构
errorTests = [
    {
        namespaceString: "wrong:namespace_type",
        errorMessage:    "unable to create CPE namespace from wrong:namespace_type: type namespace_type is incorrect",
    },
]

# 遍历错误测试数据
for _, test := range errorTests:
    # 将命名空间字符串转换为CPE对象，并捕获错误
    _, err := FromString(test.namespaceString)
    # 断言捕获的错误消息与预期错误消息相等
    assert.EqualError(t, err, test.errorMessage)
}
```