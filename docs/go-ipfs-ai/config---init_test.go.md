# `kubo\config\init_test.go`

```go
package config

import (
    "bytes"  // 导入 bytes 包
    "testing"  // 导入 testing 包

    "github.com/ipfs/kubo/core/coreiface/options"  // 导入 options 包
    crypto_pb "github.com/libp2p/go-libp2p/core/crypto/pb"  // 导入 crypto_pb 包
)

func TestCreateIdentity(t *testing.T) {
    writer := bytes.NewBuffer(nil)  // 创建一个新的字节缓冲区
    id, err := CreateIdentity(writer, []options.KeyGenerateOption{options.Key.Type(options.Ed25519Key)})  // 调用 CreateIdentity 函数创建身份，并传入参数
    if err != nil {  // 如果出现错误
        t.Fatal(err)  // 输出错误信息
    }
    pk, err := id.DecodePrivateKey("")  // 解码私钥
    if err != nil {  // 如果出现错误
        t.Fatal(err)  // 输出错误信息
    }
    if pk.Type() != crypto_pb.KeyType_Ed25519 {  // 如果私钥类型不是 Ed25519
        t.Fatal("unexpected type:", pk.Type())  // 输出错误信息
    }

    id, err = CreateIdentity(writer, []options.KeyGenerateOption{options.Key.Type(options.RSAKey)})  // 调用 CreateIdentity 函数创建身份，并传入参数
    if err != nil {  // 如果出现错误
        t.Fatal(err)  // 输出错误信息
    }
    pk, err = id.DecodePrivateKey("")  // 解码私钥
    if err != nil {  // 如果出现错误
        t.Fatal(err)  // 输出错误信息
    }
    if pk.Type() != crypto_pb.KeyType_RSA {  // 如果私钥类型不是 RSA
        t.Fatal("unexpected type:", pk.Type())  // 输出错误信息
    }
}

func TestCreateIdentityOptions(t *testing.T) {
    var w bytes.Buffer  // 创建一个新的字节缓冲区

    // ed25519 keys with bit size must fail.
    _, err := CreateIdentity(&w, []options.KeyGenerateOption{  // 调用 CreateIdentity 函数创建身份，并传入参数
        options.Key.Type(options.Ed25519Key),  // 设置密钥类型为 Ed25519
        options.Key.Size(2048),  // 设置密钥大小为 2048
    })
    if err == nil {  // 如果没有出现错误
        t.Errorf("ed25519 keys cannot have a custom bit size")  // 输出错误信息
    }
}
```