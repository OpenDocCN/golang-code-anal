# `grype\grype\db\v5\pkg\resolver\python\resolver.go`

```
package python

import (
	"regexp"  // 导入正则表达式包
	"strings"  // 导入字符串处理包

	grypePkg "github.com/anchore/grype/grype/pkg"  // 导入第三方包

)

type Resolver struct {
}

func (r *Resolver) Normalize(name string) string {
	// 规范化 Python 包的命名，参考 PEP 503 规范
	// https://peps.python.org/pep-0503/#normalized-names
	// 该代码源自官方 Python 规范命名的实现
	// https://packaging.pypa.io/en/latest/_modules/packaging/utils.html#canonicalize_name

	return strings.ToLower(regexp.MustCompile(`[-_.]+`).ReplaceAllString(name, "-"))  // 使用正则表达式将包名规范化为小写并替换特殊字符
}
# 定义 Resolver 结构体的 Resolve 方法，接收一个 grypePkg.Package 类型的参数 p，并返回一个字符串数组
func (r *Resolver) Resolve(p grypePkg.Package) []string {
    // 在 Python 中，包的规范命名由 PEP 503 定义，可以在 https://peps.python.org/pep-0503/#normalized-names 找到相关信息。
    // 该代码是从官方 Python 规范命名的实现中派生出来的，可以在 https://packaging.pypa.io/en/latest/_modules/packaging/utils.html#canonicalize_name 找到官方实现的代码
    // 返回一个包含规范化后的包名的字符串数组
    return []string{r.Normalize(p.Name)}
}
```