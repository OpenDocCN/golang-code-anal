# `v2ray-core\common\protocol\context.go`

```
# 定义一个名为protocol的包
package protocol

# 导入context包
import (
    "context"
)

# 定义一个类型为key的常量
type key int

# 定义一个枚举类型的常量requestKey，并初始化为0
const (
    requestKey key = iota
)

# 定义一个函数，用于将请求头信息存储到上下文中
func ContextWithRequestHeader(ctx context.Context, request *RequestHeader) context.Context {
    return context.WithValue(ctx, requestKey, request)
}

# 定义一个函数，用于从上下文中获取请求头信息
func RequestHeaderFromContext(ctx context.Context) *RequestHeader {
    # 从上下文中获取requestKey对应的值
    request := ctx.Value(requestKey)
    # 如果值为空，则返回空
    if request == nil {
        return nil
    }
    # 将值转换为RequestHeader类型并返回
    return request.(*RequestHeader)
}
```