# `v2ray-core\main\confloader\external\errors.generated.go`

```go
# 定义 external 包
package external

# 导入 errors 包
import "v2ray.com/core/common/errors"

# 定义 errPathObjHolder 结构体
type errPathObjHolder struct{}

# 定义 newError 函数，接收可变参数并返回一个 errors.Error 类型的指针
func newError(values ...interface{}) *errors.Error:
    # 调用 errors 包的 New 函数，传入可变参数，并使用 WithPathObj 方法设置错误路径对象为 errPathObjHolder
    return errors.New(values...).WithPathObj(errPathObjHolder{})
```