# `kubo\core\commands\dht_test.go`

```go
package commands

import (
    "testing"  // 导入测试包

    "github.com/ipfs/boxo/namesys"  // 导入namesys包

    ipns "github.com/ipfs/boxo/ipns"  // 导入ipns包
    "github.com/libp2p/go-libp2p/core/test"  // 导入test包
)

func TestKeyTranslation(t *testing.T) {
    pid := test.RandPeerIDFatal(t)  // 生成随机的PeerID
    pkname := namesys.PkRoutingKey(pid)  // 使用PeerID生成PK路由键
    ipnsname := ipns.NameFromPeer(pid).RoutingKey()  // 使用PeerID生成IPNS路由键

    pkk, err := escapeDhtKey("/pk/" + pid.String())  // 转义DHT键
    if err != nil {
        t.Fatal(err)  // 如果有错误则测试失败
    }

    ipnsk, err := escapeDhtKey("/ipns/" + pid.String())  // 转义DHT键
    if err != nil {
        t.Fatal(err)  // 如果有错误则测试失败
    }

    if pkk != pkname {
        t.Fatal("keys didn't match!")  // 如果PK键不匹配则测试失败
    }

    if ipnsk != string(ipnsname) {
        t.Fatal("keys didn't match!")  // 如果IPNS键不匹配则测试失败
    }
}
```