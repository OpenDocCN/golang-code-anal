# `grype\grype\db\v5\pkg\resolver\from_language_test.go`

```
package resolver

import (
    "testing"  // 导入测试包

    "github.com/stretchr/testify/assert"  // 导入断言包

    "github.com/anchore/grype/grype/db/v5/pkg/resolver/java"  // 导入 Java 包解析器
    "github.com/anchore/grype/grype/db/v5/pkg/resolver/python"  // 导入 Python 包解析器
    "github.com/anchore/grype/grype/db/v5/pkg/resolver/stock"  // 导入通用包解析器
    syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 别名

)

func TestFromLanguage(t *testing.T) {
    tests := []struct {  // 定义测试用例结构体
        language syftPkg.Language  // 语言类型
        result   Resolver  // 解析器结果
    }{
        {
            language: syftPkg.Python,  // Python 语言
            result:   &python.Resolver{},  // 使用 Python 包解析器
        },
        {
            language: syftPkg.Java,  // Java 语言
            result:   &java.Resolver{},  // 使用 Java 包解析器
        },
        {
            language: syftPkg.Ruby,  // Ruby 语言
            result:   &stock.Resolver{},  // 使用通用包解析器
        },
        {
            language: syftPkg.Dart,  // Dart 语言
            result:   &stock.Resolver{},  // 使用通用包解析器
        },
        {
            language: syftPkg.Rust,  // Rust 语言
            result:   &stock.Resolver{},  // 使用通用包解析器
        },
        {
            language: syftPkg.Go,  // Go 语言
            result:   &stock.Resolver{},  // 使用通用包解析器
        },
        {
            language: syftPkg.JavaScript,  // JavaScript 语言
            result:   &stock.Resolver{},  // 使用通用包解析器
        },
        {
            language: syftPkg.Dotnet,  // Dotnet 语言
            result:   &stock.Resolver{},  // 使用通用包解析器
        },
        {
            language: syftPkg.PHP,  // PHP 语言
            result:   &stock.Resolver{},  // 使用通用包解析器
        },
        {
            language: syftPkg.Ruby,  // Ruby 语言
            result:   &stock.Resolver{},  // 使用通用包解析器
        },
        {
            language: syftPkg.Language("something-new"),  // 新语言
            result:   &stock.Resolver{},  // 使用通用包解析器
        },
    }

    for _, test := range tests {  // 遍历测试用例
        result, err := FromLanguage(test.language)  // 根据语言获取解析器
        assert.NoError(t, err)  // 断言无错误
        assert.Equal(t, result, test.result)  // 断言结果与预期一致
    }
}
```