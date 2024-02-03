# `v2ray-core\proxy\vless\errors.generated.go`

```go
# 定义 vless 包
package vless

# 导入 errors 包
import "v2ray.com/core/common/errors"

# 定义 errPathObjHolder 结构体
type errPathObjHolder struct{}

# 定义 newError 函数，接收可变参数并返回 errors.Error 类型
func newError(values ...interface{}) *errors.Error {
    # 调用 errors 包的 New 函数，传入可变参数，并使用 errPathObjHolder 结构体作为路径对象
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```