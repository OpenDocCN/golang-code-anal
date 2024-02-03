# `v2ray-core\common\peer\latency.go`

```go
// 定义一个名为 peer 的包
package peer

// 导入 sync 包
import (
    "sync"
)

// 定义 Latency 接口
type Latency interface {
    Value() uint64
}

// 定义 HasLatency 接口
type HasLatency interface {
    ConnectionLatency() Latency
    HandshakeLatency() Latency
}

// 定义 AverageLatency 结构体
type AverageLatency struct {
    access sync.Mutex // 互斥锁，用于保护 value 字段
    value  uint64     // 平均延迟值
}

// 更新平均延迟值的方法
func (al *AverageLatency) Update(newValue uint64) {
    al.access.Lock() // 加锁
    defer al.access.Unlock() // 延迟解锁

    // 更新平均延迟值
    al.value = (al.value + newValue*2) / 3
}

// 获取平均延迟值的方法
func (al *AverageLatency) Value() uint64 {
    return al.value
}
```