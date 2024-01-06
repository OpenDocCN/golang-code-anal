# `grype\grype\db\v5\pkg\resolver\from_language.go`

```
package resolver

import (
	"github.com/anchore/grype/grype/db/v5/pkg/resolver/java"  // 导入 Java 解析器包
	"github.com/anchore/grype/grype/db/v5/pkg/resolver/python"  // 导入 Python 解析器包
	"github.com/anchore/grype/grype/db/v5/pkg/resolver/stock"  // 导入通用解析器包
	syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包
)

func FromLanguage(language syftPkg.Language) (Resolver, error) {
	var r Resolver  // 声明 Resolver 接口变量

	switch language {  // 根据语言类型进行判断
	case syftPkg.Python:  // 如果是 Python
		r = &python.Resolver{}  // 使用 Python 解析器
	case syftPkg.Java:  // 如果是 Java
		r = &java.Resolver{}  // 使用 Java 解析器
	default:  // 其他情况
		r = &stock.Resolver{}  // 使用通用解析器
	}
这是一个函数的结尾，返回r和nil。
```