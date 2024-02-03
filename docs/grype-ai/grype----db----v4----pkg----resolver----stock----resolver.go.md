# `grype\grype\db\v4\pkg\resolver\stock\resolver.go`

```go
# 导入 stock 包和必要的依赖包
package stock

import (
    "strings"  # 导入 strings 包，用于字符串操作

    grypePkg "github.com/anchore/grype/grype/pkg"  # 导入 grypePkg 包
)

# 定义 Resolver 结构体
type Resolver struct {
}

# 定义 Normalize 方法，用于规范化包名
func (r *Resolver) Normalize(name string) string {
    return strings.ToLower(name)  # 将包名转换为小写并返回
}

# 定义 Resolve 方法，用于解析包
func (r *Resolver) Resolve(p grypePkg.Package) []string {
    return []string{r.Normalize(p.Name)}  # 返回规范化后的包名
}
```