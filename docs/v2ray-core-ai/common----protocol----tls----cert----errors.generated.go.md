# `v2ray-core\common\protocol\tls\cert\errors.generated.go`

```go
# 定义 cert 包
package cert

# 导入 errors 包
import "v2ray.com/core/common/errors"

# 定义 errPathObjHolder 结构体
type errPathObjHolder struct{}

# 定义 newError 函数，接收任意类型的参数并返回一个 errors.Error 类型的指针
func newError(values ...interface{}) *errors.Error:
    # 调用 errors 包的 New 函数，传入参数并返回一个错误对象，同时设置错误对象的路径为 errPathObjHolder 结构体
    return errors.New(values...).WithPathObj(errPathObjHolder{})
```