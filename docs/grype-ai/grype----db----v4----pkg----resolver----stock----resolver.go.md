# `grype\grype\db\v4\pkg\resolver\stock\resolver.go`

```
// 声明一个名为 stock 的包
package stock

// 导入必要的包
import (
	"strings" // 导入字符串处理包
	grypePkg "github.com/anchore/grype/grype/pkg" // 导入 grypePkg 包
)

// 声明 Resolver 结构体
type Resolver struct {
}

// 定义 Resolver 结构体的 Normalize 方法，用于将字符串转换为小写
func (r *Resolver) Normalize(name string) string {
	return strings.ToLower(name) // 返回小写的字符串
}

// 定义 Resolver 结构体的 Resolve 方法，用于解析包
func (r *Resolver) Resolve(p grypePkg.Package) []string {
	return []string{r.Normalize(p.Name)} // 返回经过 Normalize 方法处理后的包名
}
```