# `v2ray-core\common\crypto\errors.generated.go`

```
# 导入 crypto 包和 errors 包
package crypto

# 导入 errors 包
import "v2ray.com/core/common/errors"

# 定义一个私有结构体 errPathObjHolder
type errPathObjHolder struct{}

# 定义一个新的错误，接受可变参数，并返回一个 errors.Error 对象
func newError(values ...interface{}) *errors.Error {
    # 调用 errors 包的 New 方法创建一个错误对象，并设置错误路径对象为 errPathObjHolder 结构体
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```