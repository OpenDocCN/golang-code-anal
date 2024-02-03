# `v2ray-core\common\mux\errors.generated.go`

```go
# 定义一个名为 mux 的包
package mux

# 导入 errors 包
import "v2ray.com/core/common/errors"

# 定义一个名为 errPathObjHolder 的结构体
type errPathObjHolder struct{}

# 定义一个名为 newError 的函数，接收任意类型的参数，并返回一个 errors.Error 类型的指针
func newError(values ...interface{}) *errors.Error {
    # 调用 errors 包中的 New 函数，将参数传递给它，并使用 errPathObjHolder 结构体作为路径对象
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```