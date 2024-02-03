# `kubo\core\node\libp2p\relay.go`

```go
package libp2p

import (
    "context"

    "github.com/ipfs/kubo/config"  // 导入配置包
    "github.com/libp2p/go-libp2p"   // 导入 libp2p 包
    "github.com/libp2p/go-libp2p/core/peer"  // 导入 libp2p 核心对等节点包
    "github.com/libp2p/go-libp2p/p2p/host/autorelay"  // 导入 libp2p 自动中继主机包
    "github.com/libp2p/go-libp2p/p2p/protocol/circuitv2/relay"  // 导入 libp2p 电路协议中继包
    "go.uber.org/fx"  // 导入 fx 包
)

func RelayTransport(enableRelay bool) func() (opts Libp2pOpts, err error) {
    return func() (opts Libp2pOpts, err error) {
        if enableRelay {  // 如果启用中继
            opts.Opts = append(opts.Opts, libp2p.EnableRelay())  // 将启用中继选项添加到选项列表中
        } else {
            opts.Opts = append(opts.Opts, libp2p.DisableRelay())  // 否则将禁用中继选项添加到选项列表中
        }
        return  // 返回选项列表
    }
}

func RelayService(enable bool, relayOpts config.RelayService) func() (opts Libp2pOpts, err error) {
    # 返回一个函数，该函数返回 Libp2pOpts 和 error
    return func() (opts Libp2pOpts, err error) {
        # 如果 enable 为真
        if enable {
            # 获取默认资源
            def := relay.DefaultResources()
            # 真正的默认值在 go-libp2p 中
            # 在这里我们应用用户配置中的任何覆盖
            # 将用户配置中的覆盖应用到选项中
            opts.Opts = append(opts.Opts, libp2p.EnableRelayService(relay.WithResources(relay.Resources{
                # 设置数据传输限制
                Limit: &relay.RelayLimit{
                    Data:     relayOpts.ConnectionDataLimit.WithDefault(def.Limit.Data),
                    Duration: relayOpts.ConnectionDurationLimit.WithDefault(def.Limit.Duration),
                },
                # 设置最大电路数
                MaxCircuits:            int(relayOpts.MaxCircuits.WithDefault(int64(def.MaxCircuits))),
                # 设置缓冲区大小
                BufferSize:             int(relayOpts.BufferSize.WithDefault(int64(def.BufferSize))),
                # 设置预留时间
                ReservationTTL:         relayOpts.ReservationTTL.WithDefault(def.ReservationTTL),
                # 设置最大预留数
                MaxReservations:        int(relayOpts.MaxReservations.WithDefault(int64(def.MaxReservations))),
                # 设置每个 IP 的最大预留数
                MaxReservationsPerIP:   int(relayOpts.MaxReservationsPerIP.WithDefault(int64(def.MaxReservationsPerIP))),
                # 设置每个对等体的最大预留数
                MaxReservationsPerPeer: int(relayOpts.MaxReservationsPerPeer.WithDefault(int64(def.MaxReservationsPerPeer))),
                # 设置每个 ASN 的最大预留数
                MaxReservationsPerASN:  int(relayOpts.MaxReservationsPerASN.WithDefault(int64(def.MaxReservationsPerASN))),
            })))
        }
        # 返回结果
        return
    }
// MaybeAutoRelay 函数根据参数决定是否启用自动中继功能，并返回相应的 fx.Option
func MaybeAutoRelay(staticRelays []string, cfgPeering config.Peering, enabled bool) fx.Option {
    // 如果未启用自动中继功能，则返回空的 fx.Options
    if !enabled {
        return fx.Options()
    }

    // 如果静态中继列表不为空，则提供一个函数，该函数返回 Libp2pOpts 和 error
    if len(staticRelays) > 0 {
        return fx.Provide(func() (opts Libp2pOpts, err error) {
            // 如果静态中继列表不为空，则创建一个空的 peer.AddrInfo 切片
            if len(staticRelays) > 0 {
                static := make([]peer.AddrInfo, 0, len(staticRelays))
                // 遍历静态中继列表，将每个地址解析为 peer.AddrInfo，并添加到 static 切片中
                for _, s := range staticRelays {
                    var addr *peer.AddrInfo
                    addr, err = peer.AddrInfoFromString(s)
                    if err != nil {
                        return
                    }
                    static = append(static, *addr)
                }
                // 将启用自动中继功能并使用静态中继的选项添加到 opts.Opts 中
                opts.Opts = append(opts.Opts, libp2p.EnableAutoRelayWithStaticRelays(static))
            }
            return
        })
    }

    // 如果静态中继列表为空，则创建一个 peer.AddrInfo 类型的通道
    peerChan := make(chan peer.AddrInfo)
}
    return fx.Options(
        // 提供 AutoRelay 选项
        fx.Provide(func() (opts Libp2pOpts, err error) {
            // 将 AutoRelay 选项附加到 opts.Opts 中
            opts.Opts = append(opts.Opts,
                // 启用自动中继，并指定对等节点来源
                libp2p.EnableAutoRelayWithPeerSource(
                    func(ctx context.Context, numPeers int) <-chan peer.AddrInfo {
                        // TODO(9257): 让这段代码更智能（拥有状态，并尝试向外扩展搜索），而不是一个长时间运行的任务，只是轮询我们的 K 群集。
                        // 创建一个通道用于存储 peer.AddrInfo
                        r := make(chan peer.AddrInfo)
                        // 启动一个 goroutine
                        go func() {
                            // 在函数返回前关闭通道
                            defer close(r)
                            // 循环直到 numPeers 为 0
                            for ; numPeers != 0; numPeers-- {
                                select {
                                // 从 peerChan 中读取值
                                case v, ok := <-peerChan:
                                    // 如果 peerChan 已关闭，则返回
                                    if !ok {
                                        return
                                    }
                                    select {
                                    // 将值发送到 r 通道
                                    case r <- v:
                                    // 如果 ctx 被取消，则返回
                                    case <-ctx.Done():
                                        return
                                    }
                                // 如果 ctx 被取消，则返回
                                case <-ctx.Done():
                                    return
                                }
                            }
                        }()
                        return r
                    },
                    // 设置自动中继的最小间隔为 0
                    autorelay.WithMinInterval(0),
                ))
            return
        }),
        // 调用 autoRelayFeeder 函数并传入参数 cfgPeering 和 peerChan
        autoRelayFeeder(cfgPeering, peerChan),
    )
func HolePunching(flag config.Flag, hasRelayClient bool) func() (opts Libp2pOpts, err error) {
    // 返回一个函数，该函数返回Libp2pOpts和错误
    return func() (opts Libp2pOpts, err error) {
        // 如果标志为默认值
        if flag.WithDefault(true) {
            // 如果没有中继客户端
            if !hasRelayClient {
                // 如果显式启用了打洞，但中继客户端被禁用，则发生panic，否则静默禁用打洞
                if flag != config.Default {
                    log.Fatal("Failed to enable `Swarm.EnableHolePunching`, it requires `Swarm.RelayClient.Enabled` to be true.")
                } else {
                    log.Info("HolePunching has been disabled due to the RelayClient being disabled.")
                }
                return
            }
            // 将打洞选项添加到opts中
            opts.Opts = append(opts.Opts, libp2p.EnableHolePunching())
        }
        return
    }
}
```