# `kubo\core\node\builder.go`

```go
package node

import (
    "context"
    "crypto/rand"
    "encoding/base64"
    "errors"

    "go.uber.org/fx"

    "github.com/ipfs/kubo/core/node/helpers"
    "github.com/ipfs/kubo/core/node/libp2p"
    "github.com/ipfs/kubo/repo"

    ds "github.com/ipfs/go-datastore"
    dsync "github.com/ipfs/go-datastore/sync"
    cfg "github.com/ipfs/kubo/config"
    "github.com/libp2p/go-libp2p/core/crypto"
    peer "github.com/libp2p/go-libp2p/core/peer"
)

type BuildCfg struct {
    // 如果设置为 true，则节点将启用网络功能
    Online bool

    // ExtraOpts 是用于配置 ipfs 节点创建的额外选项的映射
    ExtraOpts map[string]bool

    // 如果设置为 true，则节点应运行更昂贵的进程，这将在长期内提高性能
    Permanent bool

    // DisableEncryptedConnections 完全禁用连接加密。除非进行测试，否则不要设置此选项。
    DisableEncryptedConnections bool

    // 如果设置为 true，则将构建由 nil datastore 支持的 Repo
    NilRepo bool

    Routing libp2p.RoutingOption
    Host    libp2p.HostOption
    Repo    repo.Repo
}

func (cfg *BuildCfg) getOpt(key string) bool {
    if cfg.ExtraOpts == nil {
        return false
    }

    return cfg.ExtraOpts[key]
}

func (cfg *BuildCfg) fillDefaults() error {
    if cfg.Repo != nil && cfg.NilRepo {
        return errors.New("不能同时设置 Repo 并指定 nilrepo")
    }

    if cfg.Repo == nil {
        var d ds.Datastore
        if cfg.NilRepo {
            d = ds.NewNullDatastore()
        } else {
            d = ds.NewMapDatastore()
        }
        r, err := defaultRepo(dsync.MutexWrap(d))
        if err != nil {
            return err
        }
        cfg.Repo = r
    }

    if cfg.Routing == nil {
        cfg.Routing = libp2p.DHTOption
    }

    if cfg.Host == nil {
        cfg.Host = libp2p.DefaultHostOption
    }

    return nil
}
// options creates fx option group from this build config
func (cfg *BuildCfg) options(ctx context.Context) (fx.Option, *cfg.Config) {
    // 填充默认配置，如果出错则返回错误
    err := cfg.fillDefaults()
    if err != nil {
        return fx.Error(err), nil
    }

    // 提供 repo.Repo 实例的 fx 选项
    repoOption := fx.Provide(func(lc fx.Lifecycle) repo.Repo {
        // 在生命周期结束时关闭 Repo
        lc.Append(fx.Hook{
            OnStop: func(ctx context.Context) error {
                return cfg.Repo.Close()
            },
        })

        return cfg.Repo
    })

    // 提供 helpers.MetricsCtx 实例的 fx 选项
    metricsCtx := fx.Provide(func() helpers.MetricsCtx {
        return helpers.MetricsCtx(ctx)
    })

    // 提供 libp2p.HostOption 实例的 fx 选项
    hostOption := fx.Provide(func() libp2p.HostOption {
        return cfg.Host
    })

    // 提供 libp2p.RoutingOption 实例的 fx 选项
    routingOption := fx.Provide(func() libp2p.RoutingOption {
        return cfg.Routing
    })

    // 获取 Repo 的配置信息，如果出错则返回错误
    conf, err := cfg.Repo.Config()
    if err != nil {
        return fx.Error(err), nil
    }

    // 返回 fx 选项组和配置信息
    return fx.Options(
        repoOption,
        hostOption,
        routingOption,
        metricsCtx,
    ), conf
}

// 创建默认的 Repo 实例
func defaultRepo(dstore repo.Datastore) (repo.Repo, error) {
    // 创建默认配置实例
    c := cfg.Config{}
    // 使用随机数生成 RSA 密钥对
    priv, pub, err := crypto.GenerateKeyPairWithReader(crypto.RSA, 2048, rand.Reader)
    if err != nil {
        return nil, err
    }

    // 从公钥生成 peer.ID
    pid, err := peer.IDFromPublicKey(pub)
    if err != nil {
        return nil, err
    }

    // 将私钥序列化
    privkeyb, err := crypto.MarshalPrivateKey(priv)
    if err != nil {
        return nil, err
    }

    // 设置默认的引导地址和 Swarm 地址
    c.Bootstrap = cfg.DefaultBootstrapAddresses
    c.Addresses.Swarm = []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/udp/4001/quic-v1"}
    c.Identity.PeerID = pid.String()
    c.Identity.PrivKey = base64.StdEncoding.EncodeToString(privkeyb)

    // 返回 Mock Repo 实例和 nil 错误
    return &repo.Mock{
        D: dstore,
        C: c,
    }, nil
}
```