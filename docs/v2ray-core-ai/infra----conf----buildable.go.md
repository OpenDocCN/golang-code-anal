# `v2ray-core\infra\conf\buildable.go`

```go
# 定义一个名为conf的包
package conf

# 导入名为proto的包，该包提供了与协议缓冲区相关的功能
import "github.com/golang/protobuf/proto"

# 定义一个接口类型Buildable，该接口包含一个名为Build的方法，返回一个proto.Message类型和一个error类型
type Buildable interface {
    Build() (proto.Message, error)
}
```