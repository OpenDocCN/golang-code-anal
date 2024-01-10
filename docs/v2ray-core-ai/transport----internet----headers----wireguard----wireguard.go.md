# `v2ray-core\transport\internet\headers\wireguard\wireguard.go`

```
// 导入 wireguard 包
package wireguard

// 导入必要的包
import (
    "context"
    "v2ray.com/core/common"
)

// 定义 Wireguard 结构体
type Wireguard struct{}

// 实现 Size 方法，返回值为 int32 类型
func (Wireguard) Size() int32 {
    return 4
}

// 实现 Serialize 方法，将数据序列化到字节流中
func (Wireguard) Serialize(b []byte) {
    b[0] = 0x04
    b[1] = 0x00
    b[2] = 0x00
    b[3] = 0x00
}

// NewWireguard 根据给定的配置返回一个新的 Wireguard 实例
func NewWireguard(ctx context.Context, config interface{}) (interface{}, error) {
    return Wireguard{}, nil
}

// 在初始化时注册 WireguardConfig 类型的配置和 NewWireguard 方法
func init() {
    common.Must(common.RegisterConfig((*WireguardConfig)(nil), NewWireguard))
}
```