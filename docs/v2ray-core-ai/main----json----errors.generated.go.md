# `v2ray-core\main\json\errors.generated.go`

```
# 导入 json 包和 errors 包
import "v2ray.com/core/common/errors"

# 定义一个私有结构体 errPathObjHolder
type errPathObjHolder struct{}

# 定义一个函数 newError，接收任意类型的参数，并返回一个 errors.Error 类型的指针
func newError(values ...interface{}) *errors.Error {
    # 调用 errors 包中的 New 函数，将参数传入并返回一个 errors.Error 类型的指针
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```