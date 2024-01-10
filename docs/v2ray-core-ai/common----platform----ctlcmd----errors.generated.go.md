# `v2ray-core\common\platform\ctlcmd\errors.generated.go`

```
# 定义了一个名为 ctlcmd 的包
package ctlcmd

# 导入了名为 errors 的包，该包位于 "v2ray.com/core/common" 下
import "v2ray.com/core/common/errors"

# 定义了一个名为 errPathObjHolder 的结构体
type errPathObjHolder struct{}

# 定义了一个名为 newError 的函数，接收任意数量的参数，并返回一个 errors.Error 类型的指针
func newError(values ...interface{}) *errors.Error:
    # 调用 errors 包中的 New 函数，将传入的参数作为值，创建一个错误对象
    return errors.New(values...).WithPathObj(errPathObjHolder{})
```