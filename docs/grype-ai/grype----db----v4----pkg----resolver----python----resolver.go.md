# `grype\grype\db\v4\pkg\resolver\python\resolver.go`

```
package python

import (
	"regexp"  // 导入正则表达式包
	"strings"  // 导入字符串处理包

	grypePkg "github.com/anchore/grype/grype/pkg"  // 导入外部包

)

type Resolver struct {
}

func (r *Resolver) Normalize(name string) string {
	// 规范化 Python 包的命名，根据 PEP 503 规范定义，代码来源于官方 Python 规范实现
	// 可以在 https://peps.python.org/pep-0503/#normalized-names 查看 PEP 503 规范
	// 也可以在 https://packaging.pypa.io/en/latest/_modules/packaging/utils.html#canonicalize_name 查看官方实现代码

	return strings.ToLower(regexp.MustCompile(`[-_.]+`).ReplaceAllString(name, "-"))  // 使用正则表达式将包名规范化并转换为小写
}
// Resolve函数用于解析给定的包，返回规范化后的包名
func (r *Resolver) Resolve(p grypePkg.Package) []string {
	// 在Python中，包的规范命名由PEP 503定义，可以在https://peps.python.org/pep-0503/#normalized-names找到，这段代码是从官方Python规范命名的实现中派生出来的
	// 官方Python规范命名的实现可以在https://packaging.pypa.io/en/latest/_modules/packaging/utils.html#canonicalize_name找到
	// 返回规范化后的包名
	return []string{r.Normalize(p.Name)}
}
```