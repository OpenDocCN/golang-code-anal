# `kubo\routing\error.go`

```go
// 定义 routing 包
package routing

// 导入 fmt 包
import (
    "fmt"

    // 导入 config 包
    "github.com/ipfs/kubo/config"
)

// 定义 ParamNeededError 结构体
type ParamNeededError struct {
    ParamName  string          // 参数名
    RouterType config.RouterType  // 路由类型
}

// 定义 NewParamNeededErr 函数，返回 ParamNeededError 类型的错误
func NewParamNeededErr(param string, routing config.RouterType) error {
    return &ParamNeededError{
        ParamName:  param,   // 设置参数名
        RouterType: routing,  // 设置路由类型
    }
}

// 定义 ParamNeededError 结构体的 Error 方法，返回错误信息
func (e *ParamNeededError) Error() string {
    return fmt.Sprintf("configuration param '%v' is needed for %v delegated routing types", e.ParamName, e.RouterType)  // 返回错误信息
}
```