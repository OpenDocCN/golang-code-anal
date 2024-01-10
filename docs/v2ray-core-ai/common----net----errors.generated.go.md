# `v2ray-core\common\net\errors.generated.go`

```
# 定义一个名为net的包
package net

# 导入errors包
import "v2ray.com/core/common/errors"

# 定义一个名为errPathObjHolder的结构体
type errPathObjHolder struct{}

# 定义一个名为newError的函数，接收任意类型的参数，并返回一个errors.Error类型的指针
func newError(values ...interface{}) *errors.Error {
    # 调用errors包中的New函数，将传入的参数作为错误信息，返回一个带有错误信息的errors.Error类型的指针
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```