# `v2ray-core\proxy\vless\inbound\errors.generated.go`

```go
# 定义一个名为inbound的包
package inbound

# 导入名为"errors"的包
import "v2ray.com/core/common/errors"

# 定义一个名为errPathObjHolder的结构体
type errPathObjHolder struct{}

# 定义一个名为newError的函数，接受任意数量的参数，返回一个错误对象
func newError(values ...interface{}) *errors.Error:
    # 调用errors包中的New函数，传入values参数，返回一个带有错误信息的错误对象
    return errors.New(values...).WithPathObj(errPathObjHolder{})
```