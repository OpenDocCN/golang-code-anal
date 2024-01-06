# `grype\grype\db\v5\pkg\resolver\resolver.go`

```
# 声明一个名为 resolver 的包
package resolver

# 导入 grypePkg 包中的内容，并将其命名为 grypePkg
import (
	grypePkg "github.com/anchore/grype/grype/pkg"
)

# 声明一个接口类型 Resolver
type Resolver interface {
	# 声明一个方法 Normalize，接收一个字符串参数并返回一个字符串
	Normalize(string) string
	# 声明一个方法 Resolve，接收一个 grypePkg.Package 类型的参数并返回一个字符串切片
	Resolve(p grypePkg.Package) []string
}
```