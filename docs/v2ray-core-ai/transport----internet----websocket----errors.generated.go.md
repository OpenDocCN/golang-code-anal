# `v2ray-core\transport\internet\websocket\errors.generated.go`

```
# 导入 websocket 包
import "v2ray.com/core/common/errors"

# 定义一个私有的错误路径对象持有者类型
type errPathObjHolder struct{}

# 定义一个新的错误对象，接受可变数量的参数，并将其封装成错误对象
func newError(values ...interface{}) *errors.Error:
    return errors.New(values...).WithPathObj(errPathObjHolder{})
```