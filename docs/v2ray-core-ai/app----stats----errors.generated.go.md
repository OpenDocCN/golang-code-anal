# `v2ray-core\app\stats\errors.generated.go`

```
# 定义一个名为 stats 的包
package stats

# 导入 errors 包
import "v2ray.com/core/common/errors"

# 定义一个名为 errPathObjHolder 的结构体
type errPathObjHolder struct{}

# 定义一个名为 newError 的函数，接收任意类型的参数，并返回一个 errors.Error 类型的指针
func newError(values ...interface{}) *errors.Error {
    # 调用 errors 包中的 New 函数，传入参数 values，并将错误路径对象设置为 errPathObjHolder 结构体
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```