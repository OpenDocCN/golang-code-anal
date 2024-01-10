# `grype\grype\db\v5\pkg\resolver\python\resolver.go`

```
package python

import (
    "regexp"  // 导入正则表达式包
    "strings"  // 导入字符串处理包

    grypePkg "github.com/anchore/grype/grype/pkg"  // 导入 grype 包
)

type Resolver struct {
}

func (r *Resolver) Normalize(name string) string {
    // 规范化 Python 包的命名，参考 PEP 503 规范
    // https://peps.python.org/pep-0503/#normalized-names
    // 该代码源自官方 Python 规范命名的实现
    // https://packaging.pypa.io/en/latest/_modules/packaging/utils.html#canonicalize_name
    return strings.ToLower(regexp.MustCompile(`[-_.]+`).ReplaceAllString(name, "-"))  // 返回规范化后的包名
}

func (r *Resolver) Resolve(p grypePkg.Package) []string {
    // 规范化 Python 包的命名，参考 PEP 503 规范
    // https://peps.python.org/pep-0503/#normalized-names
    // 该代码源自官方 Python 规范命名的实现
    // https://packaging.pypa.io/en/latest/_modules/packaging/utils.html#canonicalize_name
    return []string{r.Normalize(p.Name)}  // 返回规范化后的包名列表
}
```