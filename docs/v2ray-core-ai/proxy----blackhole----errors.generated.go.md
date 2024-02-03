# `v2ray-core\proxy\blackhole\errors.generated.go`

```go
package blackhole

import "v2ray.com/core/common/errors"

type errPathObjHolder struct{}  // 定义一个空结构体 errPathObjHolder

func newError(values ...interface{}) *errors.Error {
    return errors.New(values...).WithPathObj(errPathObjHolder{})  // 使用传入的参数创建一个新的错误对象，并设置错误路径对象为 errPathObjHolder
}
```