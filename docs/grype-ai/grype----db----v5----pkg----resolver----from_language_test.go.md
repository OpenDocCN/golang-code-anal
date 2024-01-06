# `grype\grype\db\v5\pkg\resolver\from_language_test.go`

```
package resolver

import (
	"testing"  // 导入测试包

	"github.com/stretchr/testify/assert"  // 导入断言包

	"github.com/anchore/grype/grype/db/v5/pkg/resolver/java"  // 导入 Java 包解析器
	"github.com/anchore/grype/grype/db/v5/pkg/resolver/python"  // 导入 Python 包解析器
	"github.com/anchore/grype/grype/db/v5/pkg/resolver/stock"  // 导入通用包解析器
	syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包
)

func TestFromLanguage(t *testing.T) {  // 定义测试函数
	tests := []struct {  // 定义测试用例
		language syftPkg.Language  // 测试用例中的语言类型
		result   Resolver  // 测试用例中的预期结果
	}{
		{
			language: syftPkg.Python,  // 设置测试用例的语言类型为 Python
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
			// 创建一个包含语言和解析器的对象，语言为 syftPkg.Go，解析器为 stock.Resolver
			language: syftPkg.Go,
			result:   &stock.Resolver{},
		},
		{
			// 创建一个包含语言和解析器的对象，语言为 syftPkg.JavaScript，解析器为 stock.Resolver
			language: syftPkg.JavaScript,
			result:   &stock.Resolver{},
		},
		{
			// 创建一个包含语言和解析器的对象，语言为 syftPkg.Dotnet，解析器为 stock.Resolver
			language: syftPkg.Dotnet,
			result:   &stock.Resolver{},
		},
		{
			// 创建一个包含语言和解析器的对象，语言为 syftPkg.PHP，解析器为 stock.Resolver
			language: syftPkg.PHP,
			result:   &stock.Resolver{},
		},
		{
			// 创建一个包含语言和解析器的对象，语言为 syftPkg.Ruby，解析器为 stock.Resolver
			language: syftPkg.Ruby,
			result:   &stock.Resolver{},
		},
		{
			// 创建一个包含语言和解析器的对象，语言为 "something-new"，解析器为 stock.Resolver
			language: syftPkg.Language("something-new"),
			result:   &stock.Resolver{},
		},
# 创建一个测试用例切片，包含不同的语言和预期结果
tests := []struct {
    language string  // 语言
    result   *stock.Resolver  // 预期结果
}{
    { "english", &stock.Resolver{} },  // 英语对应的预期结果
    { "chinese", &stock.Resolver{} },  // 中文对应的预期结果
}

# 遍历测试用例切片，对每个测试用例进行测试
for _, test := range tests {
    # 调用 FromLanguage 函数，根据语言获取结果
    result, err := FromLanguage(test.language)
    # 断言错误为空
    assert.NoError(t, err)
    # 断言实际结果与预期结果相等
    assert.Equal(t, result, test.result)
}
```