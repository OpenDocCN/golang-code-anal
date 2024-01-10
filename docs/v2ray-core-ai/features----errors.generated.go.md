# `v2ray-core\features\errors.generated.go`

```
# 定义 features 包
package features

# 导入 errors 包
import "v2ray.com/core/common/errors"

# 定义 errPathObjHolder 结构体
type errPathObjHolder struct{}

# 定义 newError 函数，返回一个 errors.Error 类型的指针
# values... 表示可变参数，可以接受任意数量的参数
# 调用 errors 包中的 New 函数，将参数传递给它，并使用 errPathObjHolder 结构体作为路径对象
func newError(values ...interface{}) *errors.Error {
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```