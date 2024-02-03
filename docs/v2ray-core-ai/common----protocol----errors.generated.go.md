# `v2ray-core\common\protocol\errors.generated.go`

```go
# 定义一个名为protocol的包
package protocol

# 导入errors包
import "v2ray.com/core/common/errors"

# 定义一个名为errPathObjHolder的结构体
type errPathObjHolder struct{}

# 定义一个名为newError的函数，接收任意类型的参数，并返回一个*errors.Error类型的错误对象
func newError(values ...interface{}) *errors.Error {
    # 调用errors包中的New函数，将传入的参数作为错误信息，创建一个新的错误对象，并将errPathObjHolder{}作为错误路径对象
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```