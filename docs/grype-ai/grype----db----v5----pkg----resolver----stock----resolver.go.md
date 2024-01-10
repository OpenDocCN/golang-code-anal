# `grype\grype\db\v5\pkg\resolver\stock\resolver.go`

```
# 声明一个名为 stock 的包
package stock

# 导入需要使用的包
import (
    "strings"  # 导入 strings 包，用于字符串操作

    grypePkg "github.com/anchore/grype/grype/pkg"  # 导入 grypePkg 包，用于处理 grype 相关的功能
)

# 声明一个名为 Resolver 的结构体
type Resolver struct {
}

# 声明 Resolver 结构体的方法 Normalize，用于规范化包名
func (r *Resolver) Normalize(name string) string {
    return strings.ToLower(name)  # 将包名转换为小写并返回
}

# 声明 Resolver 结构体的方法 Resolve，用于解析包
func (r *Resolver) Resolve(p grypePkg.Package) []string {
    return []string{r.Normalize(p.Name)}  # 返回规范化后的包名
}
```