# `grype\grype\db\v4\pkg\resolver\resolver.go`

```go
# 声明一个名为 resolver 的包
package resolver

# 导入 grypePkg 包，并将其命名为 grypePkg
import (
    grypePkg "github.com/anchore/grype/grype/pkg"
)

# 声明一个接口类型 Resolver
type Resolver interface {
    # 声明接口方法 Normalize，接收一个字符串参数，返回一个字符串
    Normalize(string) string
    # 声明接口方法 Resolve，接收一个 grypePkg.Package 类型参数，返回一个字符串切片
    Resolve(p grypePkg.Package) []string
}
```