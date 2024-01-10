# `v2ray-core\app\stats\counter.go`

```
// +build !confonly
// 声明该文件不仅仅是用于配置

package stats
// 导入 sync/atomic 包

import "sync/atomic"

// Counter 是 stats.Counter 的实现
type Counter struct {
    value int64
}

// Value 实现了 stats.Counter
func (c *Counter) Value() int64 {
    return atomic.LoadInt64(&c.value)
}

// Set 实现了 stats.Counter
func (c *Counter) Set(newValue int64) int64 {
    return atomic.SwapInt64(&c.value, newValue)
}

// Add 实现了 stats.Counter
func (c *Counter) Add(delta int64) int64 {
    return atomic.AddInt64(&c.value, delta)
}
```