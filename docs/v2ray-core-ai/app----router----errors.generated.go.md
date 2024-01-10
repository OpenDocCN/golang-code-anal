# `v2ray-core\app\router\errors.generated.go`

```
# 定义一个名为router的包
package router

# 导入名为errors的包，该包提供了错误处理的功能
import "v2ray.com/core/common/errors"

# 定义一个名为errPathObjHolder的结构体
type errPathObjHolder struct{}

# 定义一个名为newError的函数，接受任意数量的参数，并返回一个错误对象
func newError(values ...interface{}) *errors.Error:
    # 调用errors包中的New函数，传入values参数，并使用WithPathObj方法添加错误路径对象
    return errors.New(values...).WithPathObj(errPathObjHolder{})
```