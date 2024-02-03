# `v2ray-core\proxy\mtproto\auth_test.go`

```go
package mtproto_test

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "crypto/rand"  // 导入 crypto/rand 包，用于生成随机数
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/google/go-cmp/cmp"  // 导入 github.com/google/go-cmp/cmp 包，用于比较数据

    "v2ray.com/core/common"  // 导入 v2ray.com/core/common 包
    . "v2ray.com/core/proxy/mtproto"  // 导入 v2ray.com/core/proxy/mtproto 包，并将其所有公开的符号都导入当前包的命名空间
)

func TestInverse(t *testing.T) {
    const size = 64  // 定义常量 size 为 64
    b := make([]byte, 64)  // 创建一个长度为 64 的字节切片
    for b[0] == b[size-1] {  // 循环直到字节切片的第一个元素不等于最后一个元素
        common.Must2(rand.Read(b))  // 生成随机数并将结果存储到字节切片 b 中
    }

    bi := Inverse(b)  // 调用 Inverse 函数，将结果存储到 bi 中
    if b[0] == bi[0] {  // 检查字节切片 b 和 bi 的第一个元素是否相等
        t.Fatal("seems bytes are not inversed: ", b[0], "vs", bi[0])  // 如果相等，则输出错误信息并终止测试
    }

    bii := Inverse(bi)  // 调用 Inverse 函数，将结果存储到 bii 中
    if r := cmp.Diff(bii, b); r != "" {  // 比较 bii 和 b 的差异
        t.Fatal(r)  // 如果有差异，则输出错误信息并终止测试
    }
}

func TestAuthenticationReadWrite(t *testing.T) {
    a := NewAuthentication(DefaultSessionContext())  // 调用 NewAuthentication 函数，将结果存储到 a 中
    b := bytes.NewReader(a.Header[:])  // 创建一个从字节切片 a.Header 中读取数据的 Reader
    a2, err := ReadAuthentication(b)  // 调用 ReadAuthentication 函数，将结果存储到 a2 中，并检查错误
    common.Must(err)  // 如果有错误，则输出错误信息并终止程序

    if r := cmp.Diff(a.EncodingKey[:], a2.DecodingKey[:]); r != "" {  // 比较 a.EncodingKey 和 a2.DecodingKey 的差异
        t.Error("decoding key: ", r)  // 如果有差异，则输出警告信息
    }

    if r := cmp.Diff(a.EncodingNonce[:], a2.DecodingNonce[:]); r != "" {  // 比较 a.EncodingNonce 和 a2.DecodingNonce 的差异
        t.Error("decoding nonce: ", r)  // 如果有差异，则输出警告信息
    }

    if r := cmp.Diff(a.DecodingKey[:], a2.EncodingKey[:]); r != "" {  // 比较 a.DecodingKey 和 a2.EncodingKey 的差异
        t.Error("encoding key: ", r)  // 如果有差异，则输出警告信息
    }

    if r := cmp.Diff(a.DecodingNonce[:], a2.EncodingNonce[:]); r != "" {  // 比较 a.DecodingNonce 和 a2.EncodingNonce 的差异
        t.Error("encoding nonce: ", r)  // 如果有差异，则输出警告信息
    }
}
```