# `grype\grype\db\v5\pkg\resolver\resolver.go`

```go
# 声明一个名为 resolver 的包
package resolver

# 导入 grypePkg 包，并将其命名为 grypePkg
import (
    grypePkg "github.com/anchore/grype/grype/pkg"
)

# 声明一个名为 Resolver 的接口
type Resolver interface {
    # 声明一个名为 Normalize 的方法，接收一个字符串参数并返回一个字符串
    Normalize(string) string
    # 声明一个名为 Resolve 的方法，接收一个 grypePkg.Package 类型的参数，并返回一个字符串切片
    Resolve(p grypePkg.Package) []string
}
```