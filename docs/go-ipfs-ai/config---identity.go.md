# `kubo\config\identity.go`

```go
package config

import (
    "encoding/base64"  // 导入 base64 编码包
    ic "github.com/libp2p/go-libp2p/core/crypto"  // 导入 libp2p 加密包
)

const (
    IdentityTag     = "Identity"  // 定义常量 IdentityTag 为 "Identity"
    PrivKeyTag      = "PrivKey"  // 定义常量 PrivKeyTag 为 "PrivKey"
    PrivKeySelector = IdentityTag + "." + PrivKeyTag  // 定义常量 PrivKeySelector 为 IdentityTag 和 PrivKeyTag 的组合
)

// Identity tracks the configuration of the local node's identity.
type Identity struct {
    PeerID  string  // 定义结构体 Identity 的属性 PeerID 为字符串类型
    PrivKey string `json:",omitempty"`  // 定义结构体 Identity 的属性 PrivKey 为字符串类型，并指定 JSON 编码规则
}

// DecodePrivateKey is a helper to decode the users PrivateKey.
func (i *Identity) DecodePrivateKey(passphrase string) (ic.PrivKey, error) {
    pkb, err := base64.StdEncoding.DecodeString(i.PrivKey)  // 使用 base64 解码 i.PrivKey 字符串
    if err != nil {  // 如果解码出错
        return nil, err  // 返回空值和错误
    }

    // currently storing key unencrypted. in the future we need to encrypt it.
    // TODO(security)
    return ic.UnmarshalPrivateKey(pkb)  // 返回解码后的私钥
}
```