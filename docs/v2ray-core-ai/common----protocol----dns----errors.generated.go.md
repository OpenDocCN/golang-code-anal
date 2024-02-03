# `v2ray-core\common\protocol\dns\errors.generated.go`

```go
# 定义一个名为 dns 的包
package dns

# 导入名为 errors 的包
import "v2ray.com/core/common/errors"

# 定义一个名为 errPathObjHolder 的结构体
type errPathObjHolder struct{}

# 定义一个名为 newError 的函数，接受可变数量的参数，并返回一个错误对象
func newError(values ...interface{}) *errors.Error {
    # 调用 errors 包中的 New 函数，传入可变数量的参数，创建一个错误对象，并将错误对象与 errPathObjHolder 结构体关联
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```