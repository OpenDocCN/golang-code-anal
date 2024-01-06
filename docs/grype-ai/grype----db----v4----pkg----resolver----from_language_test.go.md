# `grype\grype\db\v4\pkg\resolver\from_language_test.go`

```
package resolver

import (
	"testing"  // 导入测试包

	"github.com/stretchr/testify/assert"  // 导入断言包

	"github.com/anchore/grype/grype/db/v4/pkg/resolver/java"  // 导入 Java 包解析器
	"github.com/anchore/grype/grype/db/v4/pkg/resolver/python"  // 导入 Python 包解析器
	"github.com/anchore/grype/grype/db/v4/pkg/resolver/stock"  // 导入通用包解析器
	syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包
)

func TestFromLanguage(t *testing.T) {  // 定义测试函数
	tests := []struct {  // 定义测试用例
		language syftPkg.Language  // 语言类型
		result   Resolver  // 解析器结果
	}{
		{
			language: syftPkg.Python,  // Python 语言类型
```

		{
			// 创建一个 Python 解析器对象，并将其存储在 result 中
			language: syftPkg.Python,
			result:   &python.Resolver{},
		},
		{
			// 创建一个 Java 解析器对象，并将其存储在 result 中
			language: syftPkg.Java,
			result:   &java.Resolver{},
		},
		{
			// 创建一个 Ruby 解析器对象，并将其存储在 result 中
			language: syftPkg.Ruby,
			result:   &stock.Resolver{},
		},
		{
			// 创建一个 Dart 解析器对象，并将其存储在 result 中
			language: syftPkg.Dart,
			result:   &stock.Resolver{},
		},
		{
			// 创建一个 Rust 解析器对象，并将其存储在 result 中
			language: syftPkg.Rust,
			result:   &stock.Resolver{},
		},
		{
			// 创建一个 Go 解析器对象，并将其存储在 result 中
			language: syftPkg.Go,
		{
			// 创建一个包含语言和结果的对象，语言为syftPkg.Go，结果为stock.Resolver类型的指针
			language: syftPkg.Go,
			result:   &stock.Resolver{},
		},
		{
			// 创建一个包含语言和结果的对象，语言为syftPkg.JavaScript，结果为stock.Resolver类型的指针
			language: syftPkg.JavaScript,
			result:   &stock.Resolver{},
		},
		{
			// 创建一个包含语言和结果的对象，语言为syftPkg.Dotnet，结果为stock.Resolver类型的指针
			language: syftPkg.Dotnet,
			result:   &stock.Resolver{},
		},
		{
			// 创建一个包含语言和结果的对象，语言为syftPkg.PHP，结果为stock.Resolver类型的指针
			language: syftPkg.PHP,
			result:   &stock.Resolver{},
		},
		{
			// 创建一个包含语言和结果的对象，语言为syftPkg.Ruby，结果为stock.Resolver类型的指针
			language: syftPkg.Ruby,
			result:   &stock.Resolver{},
		},
		{
			// 创建一个包含语言和结果的对象，语言为自定义的"something-new"，结果为stock.Resolver类型的指针
			language: syftPkg.Language("something-new"),
# 创建一个测试用例切片，包含不同的语言和预期结果
tests := []struct {
    language string   // 语言
    result   *stock.Resolver   // 预期结果
}{
    { "english", &stock.Resolver{} },   // 英语对应的预期结果
    { "chinese", &stock.Resolver{} },   // 中文对应的预期结果
}

# 遍历测试用例切片，对每个测试用例进行测试
for _, test := range tests {
    # 调用 FromLanguage 函数，根据语言获取结果
    result, err := FromLanguage(test.language)
    # 断言测试结果是否没有错误
    assert.NoError(t, err)
    # 断言获取的结果与预期结果相等
    assert.Equal(t, result, test.result)
}
```