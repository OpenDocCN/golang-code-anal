# `trojan-go\common\io_test.go`

```go
package common

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "crypto/rand"  // 导入 crypto/rand 包，用于生成随机数
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/v2fly/v2ray-core/v4/common"  // 导入 v2ray-core 包中的 common 模块
)

func TestBufferedReader(t *testing.T) {
    payload := [1024]byte{}  // 创建一个长度为 1024 的字节数组
    rand.Reader.Read(payload[:])  // 从随机数生成器中读取随机字节填充到 payload 中
    rawReader := bytes.NewBuffer(payload[:])  // 使用 payload 创建一个字节流的缓冲区
    r := RewindReader{  // 创建一个 RewindReader 对象
        rawReader: rawReader,  // 设置 rawReader 字段为之前创建的字节流缓冲区
    }
    r.SetBufferSize(2048)  // 设置 RewindReader 对象的缓冲区大小为 2048
    buf1 := make([]byte, 512)  // 创建一个长度为 512 的字节数组
    buf2 := make([]byte, 512)  // 创建另一个长度为 512 的字节数组
    common.Must2(r.Read(buf1))  // 从 RewindReader 对象中读取数据到 buf1 中
    r.Rewind()  // 将 RewindReader 对象的读取位置重置到起始位置
    common.Must2(r.Read(buf2))  // 从 RewindReader 对象中再次读取数据到 buf2 中
    if !bytes.Equal(buf1, buf2) {  // 判断 buf1 和 buf2 中的数据是否相等
        t.Fail()  // 如果不相等，则测试失败
    }
    buf3 := make([]byte, 512)  // 创建另一个长度为 512 的字节数组
    common.Must2(r.Read(buf3))  // 从 RewindReader 对象中读取数据到 buf3 中
    if !bytes.Equal(buf3, payload[512:]) {  // 判断 buf3 中的数据是否与 payload 的后半部分相等
        t.Fail()  // 如果不相等，则测试失败
    }
    r.Rewind()  // 将 RewindReader 对象的读取位置重置到起始位置
    buf4 := make([]byte, 1024)  // 创建一个长度为 1024 的字节数组
    common.Must2(r.Read(buf4))  // 从 RewindReader 对象中读取数据到 buf4 中
    if !bytes.Equal(payload[:], buf4) {  // 判断 buf4 中的数据是否与 payload 中的数据相等
        t.Fail()  // 如果不相等，则测试失败
    }
}
```