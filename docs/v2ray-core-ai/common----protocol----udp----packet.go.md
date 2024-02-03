# `v2ray-core\common\protocol\udp\packet.go`

```go
// 导入 UDP 包
package udp

// 导入 buf 包和 net 包
import (
    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/net"
)

// Packet 结构体表示一个 UDP 数据包，包括数据载荷、源地址和目标地址
type Packet struct {
    // Payload 表示数据载荷
    Payload *buf.Buffer
    // Source 表示源地址
    Source  net.Destination
    // Target 表示目标地址
    Target  net.Destination
}
```