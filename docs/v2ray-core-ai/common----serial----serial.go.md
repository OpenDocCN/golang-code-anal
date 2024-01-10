# `v2ray-core\common\serial\serial.go`

```
package serial

import (
    "encoding/binary"  // 导入处理二进制数据的包
    "io"  // 导入输入输出操作的包
)

// ReadUint16 从读取器中读取前两个字节，然后将它们转换为 uint16 值。
func ReadUint16(reader io.Reader) (uint16, error) {
    var b [2]byte  // 创建一个长度为 2 的字节数组
    if _, err := io.ReadFull(reader, b[:]); err != nil {  // 从读取器中读取两个字节的数据到字节数组中
        return 0, err  // 如果出现错误，返回 0 和错误信息
    }
    return binary.BigEndian.Uint16(b[:]), nil  // 将字节数组中的数据按大端序解析为 uint16 值，并返回
}

// WriteUint16 将一个 uint16 值写入写入器中。
func WriteUint16(writer io.Writer, value uint16) (int, error) {
    var b [2]byte  // 创建一个长度为 2 的字节数组
    binary.BigEndian.PutUint16(b[:], value)  // 将 uint16 值按大端序写入字节数组
    return writer.Write(b[:])  // 将字节数组中的数据写入写入器中
}

// WriteUint64 将一个 uint64 值写入写入器中。
func WriteUint64(writer io.Writer, value uint64) (int, error) {
    var b [8]byte  // 创建一个长度为 8 的字节数组
    binary.BigEndian.PutUint64(b[:], value)  // 将 uint64 值按大端序写入字节数组
    return writer.Write(b[:])  // 将字节数组中的数据写入写入器中
}
```