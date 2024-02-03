# `v2ray-core\app\dispatcher\errors.generated.go`

```go
# 定义一个名为 dispatcher 的包
package dispatcher

# 导入 errors 包
import "v2ray.com/core/common/errors"

# 定义一个名为 errPathObjHolder 的类型
type errPathObjHolder struct{}

# 定义一个名为 newError 的函数，接收任意数量的参数并返回一个 errors.Error 类型的指针
func newError(values ...interface{}) *errors.Error {
    # 调用 errors 包中的 New 函数，将传入的参数作为值创建一个新的错误对象，并将 errPathObjHolder{} 作为错误路径对象
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```