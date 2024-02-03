# `v2ray-core\proxy\vmess\outbound\errors.generated.go`

```go
# 定义一个名为outbound的包
package outbound

# 导入名为"errors"的包
import "v2ray.com/core/common/errors"

# 定义一个名为errPathObjHolder的结构体
type errPathObjHolder struct{}

# 定义一个名为newError的函数，接受任意类型的参数，并返回一个*errors.Error类型的值
func newError(values ...interface{}) *errors.Error {
    # 调用errors包中的New函数，传入values参数，并调用WithPathObj方法，传入errPathObjHolder{}作为参数
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```