# `v2ray-core\common\crypto\internal\chacha_core.generated.go`

```go
package internal

import "encoding/binary"

func ChaCha20Block(s *[16]uint32, out []byte, rounds int) {
    // 定义并初始化变量 x0 到 x15，分别为 s[0] 到 s[15]
    var x0, x1, x2, x3, x4, x5, x6, x7, x8, x9, x10, x11, x12, x13, x14, x15 = s[0], s[1], s[2], s[3], s[4], s[5], s[6], s[7], s[8], s[9], s[10], s[11], s[12], s[13], s[14], s[15]

    // 将 s[0]+x0 转换为小端字节序并写入 out 的前 4 个字节
    binary.LittleEndian.PutUint32(out[0:4], s[0]+x0)
    // 将 s[1]+x1 转换为小端字节序并写入 out 的接下来 4 个字节
    binary.LittleEndian.PutUint32(out[4:8], s[1]+x1)
    // 依此类推，将 s[2]+x2 到 s[15]+x15 转换为小端字节序并写入 out
}
```