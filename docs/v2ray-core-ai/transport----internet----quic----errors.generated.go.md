# `v2ray-core\transport\internet\quic\errors.generated.go`

```
# 定义一个名为 quic 的包
package quic

# 导入 v2ray.com/core/common/errors 包
import "v2ray.com/core/common/errors"

# 定义一个名为 errPathObjHolder 的类型
type errPathObjHolder struct{}

# 定义一个名为 newError 的函数，接受可变数量的参数，返回一个 errors.Error 类型的指针
func newError(values ...interface{}) *errors.Error {
    # 调用 errors 包中的 New 函数，传入可变数量的参数，返回一个错误对象
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```