# `v2ray-core\transport\internet\tcp\errors.generated.go`

```go
# 定义一个名为tcp的包
package tcp

# 导入errors包
import "v2ray.com/core/common/errors"

# 定义一个名为errPathObjHolder的结构体
type errPathObjHolder struct{}

# 定义一个名为newError的函数，接收任意类型的参数，并返回一个errors.Error类型的指针
func newError(values ...interface{}) *errors.Error:
    # 调用errors包中的New函数，传入values参数，并使用WithPathObj方法设置错误路径对象为errPathObjHolder结构体
    return errors.New(values...).WithPathObj(errPathObjHolder{})
```