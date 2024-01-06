# `grype\grype\db\v4\pkg\resolver\from_language.go`

```
package resolver

import (
	"github.com/anchore/grype/grype/db/v4/pkg/resolver/java"  // 导入 Java 解析器包
	"github.com/anchore/grype/grype/db/v4/pkg/resolver/python"  // 导入 Python 解析器包
	"github.com/anchore/grype/grype/db/v4/pkg/resolver/stock"  // 导入通用解析器包
	syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包
)

func FromLanguage(language syftPkg.Language) (Resolver, error) {
	var r Resolver  // 声明 Resolver 接口变量

	switch language {  // 根据语言类型选择解析器
	case syftPkg.Python:
		r = &python.Resolver{}  // 使用 Python 解析器
	case syftPkg.Java:
		r = &java.Resolver{}  // 使用 Java 解析器
	default:
		r = &stock.Resolver{}  // 使用通用解析器
	}
# 返回变量 r 的值，并且没有错误发生。
```