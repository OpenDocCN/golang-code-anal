# `v2ray-core\proxy\socks\errors.generated.go`

```
# 定义一个名为socks的包
package socks

# 导入v2ray.com/core/common/errors包
import "v2ray.com/core/common/errors"

# 定义一个名为errPathObjHolder的结构体
type errPathObjHolder struct{}

# 定义一个名为newError的函数，接收任意类型的参数，并返回一个错误对象
func newError(values ...interface{}) *errors.Error {
    # 调用errors包中的New函数，传入values参数，并使用WithPathObj方法设置错误对象的路径对象为errPathObjHolder结构体
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```