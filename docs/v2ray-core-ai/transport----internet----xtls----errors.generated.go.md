# `v2ray-core\transport\internet\xtls\errors.generated.go`

```
# 导入名为 "errors" 的包
package xtls

# 导入 "v2ray.com/core/common/errors" 包
import "v2ray.com/core/common/errors"

# 定义名为 "errPathObjHolder" 的结构体
type errPathObjHolder struct{}

# 定义名为 "newError" 的函数，接受可变数量的参数，并返回一个错误对象
func newError(values ...interface{}) *errors.Error:
    # 调用 "errors.New" 函数创建一个错误对象，并将其路径对象设置为 "errPathObjHolder"
    return errors.New(values...).WithPathObj(errPathObjHolder{})
```