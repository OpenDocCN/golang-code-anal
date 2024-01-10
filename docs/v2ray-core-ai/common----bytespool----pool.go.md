# `v2ray-core\common\bytespool\pool.go`

```
package bytespool

import "sync"

// 创建一个分配函数，用于生成指定大小的字节切片
func createAllocFunc(size int32) func() interface{} {
    return func() interface{} {
        return make([]byte, size)
    }
}

// 以下参数控制缓冲池的大小。
// 有 numPools 个缓冲池。从 2k 大小开始，每个池的大小是前一个池的 sizeMulti 倍。
// 包 buf 保证不使用大于最大池的缓冲区。
// 其他数据包可能使用更大的缓冲区。
const (
    numPools  = 4
    sizeMulti = 4
)

var (
    pool     [numPools]sync.Pool
    poolSize [numPools]int32
)

func init() {
    size := int32(2048)
    for i := 0; i < numPools; i++ {
        pool[i] = sync.Pool{
            New: createAllocFunc(size),
        }
        poolSize[i] = size
        size *= sizeMulti
    }
}

// GetPool 返回一个生成至少给定大小的字节数组的 sync.Pool。
// 如果不存在这样的池，则可能返回 nil。
//
// v2ray:api:stable
func GetPool(size int32) *sync.Pool {
    for idx, ps := range poolSize {
        if size <= ps {
            return &pool[idx]
        }
    }
    return nil
}

// Alloc 返回至少给定大小的字节切片。返回切片的最小大小为 2048。
//
// v2ray:api:stable
func Alloc(size int32) []byte {
    pool := GetPool(size)
    if pool != nil {
        return pool.Get().([]byte)
    }
    return make([]byte, size)
}

// Free 将字节切片放入内部池中。
//
// v2ray:api:stable
func Free(b []byte) {
    size := int32(cap(b))
    b = b[0:cap(b)]
    for i := numPools - 1; i >= 0; i-- {
        if size >= poolSize[i] {
            pool[i].Put(b) // nolint: megacheck
            return
        }
    }
}
```