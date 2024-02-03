# `kubo\core\coreiface\idfmt.go`

```go
// 导入所需的包
package iface

import (
    "github.com/libp2p/go-libp2p/core/peer"  // 导入 peer 包
    mbase "github.com/multiformats/go-multibase"  // 导入 mbase 包
)

// 格式化 peer.ID 为指定格式的字符串
func FormatKeyID(id peer.ID) string {
    // 将 peer.ID 转换为 CID，并以 Base36 格式输出字符串
    if s, err := peer.ToCid(id).StringOfBase(mbase.Base36); err != nil {
        panic(err)  // 如果出错则抛出异常
    } else {
        return s  // 返回格式化后的字符串
    }
}

// 格式化给定的 IPNS key
func FormatKey(key Key) string {
    return FormatKeyID(key.ID())  // 调用 FormatKeyID 函数格式化 key 的 ID
}
```