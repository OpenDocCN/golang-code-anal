# `v2ray-core\proxy\mtproto\errors.generated.go`

```go
# 定义名为 mtproto 的包
package mtproto

# 导入名为 errors 的包
import "v2ray.com/core/common/errors"

# 定义名为 errPathObjHolder 的结构体
type errPathObjHolder struct{}

# 定义名为 newError 的函数，接收任意类型的参数并返回一个错误对象
func newError(values ...interface{}) *errors.Error {
    # 调用 errors 包中的 New 函数，传入参数并返回一个错误对象
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```