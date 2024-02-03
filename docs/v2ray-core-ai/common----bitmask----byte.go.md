# `v2ray-core\common\bitmask\byte.go`

```go
// 定义 Byte 类型，表示一个字节的位掩码
type Byte byte

// 判断当前位掩码是否包含另一个位掩码
func (b Byte) Has(bb Byte) bool {
    return (b & bb) != 0
}

// 设置当前位掩码的指定位
func (b *Byte) Set(bb Byte) {
    *b |= bb
}

// 清除当前位掩码的指定位
func (b *Byte) Clear(bb Byte) {
    *b &= ^bb
}

// 切换当前位掩码的指定位
func (b *Byte) Toggle(bb Byte) {
    *b ^= bb
}
```