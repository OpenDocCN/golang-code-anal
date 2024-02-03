# `v2ray-core\proxy\shadowsocks\errors.generated.go`

```go
# 导入 shadowsocks 包和 errors 包
package shadowsocks

# 导入 "v2ray.com/core/common/errors" 包

import "v2ray.com/core/common/errors"

# 定义一个结构体 errPathObjHolder
type errPathObjHolder struct{}

# 定义一个函数 newError，接收可变数量的参数，返回一个 errors.Error 类型的指针
# 使用 errors.New(values...) 创建一个错误对象，并将 errPathObjHolder 结构体作为路径对象
func newError(values ...interface{}) *errors.Error {
    return errors.New(values...).WithPathObj(errPathObjHolder{})
}
```