# `v2ray-core\proxy\http\errors.generated.go`

```go
# 定义一个名为http的包
package http

# 导入v2ray.com/core/common/errors包
import "v2ray.com/core/common/errors"

# 定义一个名为errPathObjHolder的类型
type errPathObjHolder struct{}

# 定义一个名为newError的函数，接受可变数量的参数，返回一个errors.Error类型的指针
func newError(values ...interface{}) *errors.Error {
    # 调用errors包中的New函数，传入values参数，返回一个带有错误信息的errors.Error类型
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```