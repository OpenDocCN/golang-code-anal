# `v2ray-core\transport\internet\quic\pool.go`

```
// +build !confonly
// 定义了一个名为quic的包，且不仅仅是用于配置

package quic

import (
    "sync"  // 导入sync包，用于实现同步功能
    "v2ray.com/core/common/bytespool"  // 导入bytespool包，用于管理字节池
)

var pool *sync.Pool  // 声明一个sync.Pool类型的变量pool

func init() {
    pool = bytespool.GetPool(2048)  // 初始化pool变量，使用bytespool包中的GetPool方法，设置字节池大小为2048
}

func getBuffer() []byte {  // 定义一个名为getBuffer的函数，返回一个字节数组
    return pool.Get().([]byte)  // 从字节池中获取一个字节数组并返回
}

func putBuffer(p []byte) {  // 定义一个名为putBuffer的函数，接收一个字节数组作为参数
    pool.Put(p)  // 将传入的字节数组放回字节池
}
```