# `v2ray-core\main\jsonem\errors.generated.go`

```
# 定义了一个名为jsonem的包
package jsonem

# 导入了v2ray.com/core/common/errors包
import "v2ray.com/core/common/errors"

# 定义了一个名为errPathObjHolder的类型
type errPathObjHolder struct{}

# 定义了一个名为newError的函数，接收任意数量的参数，返回一个错误对象
func newError(values ...interface{}) *errors.Error {
    # 调用errors包中的New函数，传入values参数，返回一个错误对象，并设置错误路径对象为errPathObjHolder
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```