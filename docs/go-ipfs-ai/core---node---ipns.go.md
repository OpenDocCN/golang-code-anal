# `kubo\core\node\ipns.go`

```
package node

import (
    "fmt"
    "time"

    "github.com/ipfs/boxo/ipns"
    util "github.com/ipfs/boxo/util"
    record "github.com/libp2p/go-libp2p-record"
    "github.com/libp2p/go-libp2p/core/crypto"
    "github.com/libp2p/go-libp2p/core/peerstore"
    madns "github.com/multiformats/go-multiaddr-dns"

    "github.com/ipfs/boxo/namesys"
    "github.com/ipfs/boxo/namesys/republisher"
    "github.com/ipfs/kubo/repo"
    irouting "github.com/ipfs/kubo/routing"
)

const DefaultIpnsCacheSize = 128

// RecordValidator provides namesys compatible routing record validator
func RecordValidator(ps peerstore.Peerstore) record.Validator {
    return record.NamespacedValidator{
        "pk":   record.PublicKeyValidator{},  // 使用公钥验证器作为 "pk" 命名空间的记录验证器
        "ipns": ipns.Validator{KeyBook: ps},  // 使用 IPNS 验证器作为 "ipns" 命名空间的记录验证器
    }
}

// Namesys creates new name system
func Namesys(cacheSize int) func(rt irouting.ProvideManyRouter, rslv *madns.Resolver, repo repo.Repo) (namesys.NameSystem, error) {
    return func(rt irouting.ProvideManyRouter, rslv *madns.Resolver, repo repo.Repo) (namesys.NameSystem, error) {
        opts := []namesys.Option{
            namesys.WithDatastore(repo.Datastore()),  // 使用提供的数据存储创建选项
            namesys.WithDNSResolver(rslv),  // 使用提供的 DNS 解析器创建选项
        }

        if cacheSize > 0 {
            opts = append(opts, namesys.WithCache(cacheSize))  // 如果缓存大小大于0，则使用提供的缓存大小创建选项
        }

        return namesys.NewNameSystem(rt, opts...)  // 使用提供的路由和选项创建新的名称系统
    }
}

// IpnsRepublisher runs new IPNS republisher service
func IpnsRepublisher(repubPeriod time.Duration, recordLifetime time.Duration) func(lcProcess, namesys.NameSystem, repo.Repo, crypto.PrivKey) error {
    // 定义一个函数，接受 lcProcess、NameSystem、Repo 和 PrivKey 四个参数，返回一个错误
    return func(lc lcProcess, namesys namesys.NameSystem, repo repo.Repo, privKey crypto.PrivKey) error {
        // 创建一个新的 Republisher 对象，传入 NameSystem、Repo 的数据存储、PrivKey 和 Repo 的密钥存储
        repub := republisher.NewRepublisher(namesys, repo.Datastore(), privKey, repo.Keystore())

        // 如果 repubPeriod 不为 0
        if repubPeriod != 0 {
            // 如果不是调试模式，并且 repubPeriod 小于 1 分钟或大于 1 天
            if !util.Debug && (repubPeriod < time.Minute || repubPeriod > (time.Hour*24)) {
                // 返回错误信息
                return fmt.Errorf("config setting IPNS.RepublishPeriod is not between 1min and 1day: %s", repubPeriod)
            }

            // 设置 repub 的间隔时间为 repubPeriod
            repub.Interval = repubPeriod
        }

        // 如果 recordLifetime 不为 0
        if recordLifetime != 0 {
            // 设置 repub 的记录生命周期为 recordLifetime
            repub.RecordLifetime = recordLifetime
        }

        // 将 repub.Run 添加到 lc 中
        lc.Append(repub.Run)
        // 返回空错误
        return nil
    }
# 闭合前面的函数定义
```