# `v2ray-core\features\routing\dns\errors.generated.go`

```go
# 定义了一个名为 dns 的包
package dns

# 导入了名为 errors 的包
import "v2ray.com/core/common/errors"

# 定义了一个名为 errPathObjHolder 的结构体
type errPathObjHolder struct{}

# 定义了一个名为 newError 的函数，接收任意类型的参数，并返回一个 errors.Error 类型的指针
func newError(values ...interface{}) *errors.Error {
    # 调用 errors 包中的 New 函数，将传入的参数作为值，创建一个新的错误对象，并将 errPathObjHolder 结构体作为路径对象
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```