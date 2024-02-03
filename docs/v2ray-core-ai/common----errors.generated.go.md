# `v2ray-core\common\errors.generated.go`

```go
// 定义 common 包
package common

// 导入 errors 包
import "v2ray.com/core/common/errors"

// 定义 errPathObjHolder 结构体
type errPathObjHolder struct{}

// 定义 newError 函数，返回一个 errors.Error 类型的指针
func newError(values ...interface{}) *errors.Error {
    // 调用 errors 包的 New 函数，传入 values 参数，并设置错误路径对象为 errPathObjHolder 结构体
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```