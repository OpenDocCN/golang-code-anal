# `kubo\core\node\libp2p\transport.go`

```go
package libp2p

import (
    "fmt" // 导入 fmt 包，用于格式化输出

    "github.com/ipfs/kubo/config" // 导入配置包
    "github.com/libp2p/go-libp2p" // 导入 libp2p 包
    "github.com/libp2p/go-libp2p/core/metrics" // 导入 libp2p 核心指标包
    quic "github.com/libp2p/go-libp2p/p2p/transport/quic" // 导入 QUIC 传输协议包
    "github.com/libp2p/go-libp2p/p2p/transport/tcp" // 导入 TCP 传输协议包
    webrtc "github.com/libp2p/go-libp2p/p2p/transport/webrtc" // 导入 WebRTC 传输协议包
    "github.com/libp2p/go-libp2p/p2p/transport/websocket" // 导入 WebSocket 传输协议包
    webtransport "github.com/libp2p/go-libp2p/p2p/transport/webtransport" // 导入 Web 传输协议包

    "go.uber.org/fx" // 导入 Uber 的 fx 包
)

func Transports(tptConfig config.Transports) interface{} {
    return func(pnet struct {
        fx.In // 使用 fx 框架的依赖注入
        Fprint PNetFingerprint `optional:"true"` // 可选的 PNetFingerprint 依赖注入
    },
    // 定义一个函数，接收Libp2pOpts和error类型的参数
    ) (opts Libp2pOpts, err error) {
        // 检查是否启用了私有网络
        privateNetworkEnabled := pnet.Fprint != nil

        // 如果配置中启用了TCP网络，则将TCP传输协议添加到选项中
        if tptConfig.Network.TCP.WithDefault(true) {
            // TODO(9290): Make WithMetrics configurable
            opts.Opts = append(opts.Opts, libp2p.Transport(tcp.NewTCPTransport, tcp.WithMetrics()))
        }

        // 如果配置中启用了Websocket网络，则将Websocket传输协议添加到选项中
        if tptConfig.Network.Websocket.WithDefault(true) {
            opts.Opts = append(opts.Opts, libp2p.Transport(websocket.New))
        }

        // 如果配置中启用了QUIC网络，则将QUIC传输协议添加到选项中
        if tptConfig.Network.QUIC.WithDefault(!privateNetworkEnabled) {
            // 如果私有网络已启用，则返回错误
            if privateNetworkEnabled {
                return opts, fmt.Errorf(
                    "QUIC transport does not support private networks, please disable Swarm.Transports.Network.QUIC",
                )
            }
            opts.Opts = append(opts.Opts, libp2p.Transport(quic.NewTransport))
        }

        // 如果配置中启用了WebTransport网络，则将WebTransport传输协议添加到选项中
        if tptConfig.Network.WebTransport.WithDefault(!privateNetworkEnabled) {
            // 如果私有网络已启用，则返回错误
            if privateNetworkEnabled {
                return opts, fmt.Errorf(
                    "WebTransport transport does not support private networks, please disable Swarm.Transports.Network.WebTransport",
                )
            }
            opts.Opts = append(opts.Opts, libp2p.Transport(webtransport.New))
        }

        // 如果配置中启用了WebRTCDirect网络，则将WebRTCDirect传输协议添加到选项中
        if tptConfig.Network.WebRTCDirect.WithDefault(false) {
            // 如果私有网络已启用，则返回错误
            if privateNetworkEnabled {
                return opts, fmt.Errorf(
                    "WebRTC Direct transport does not support private networks, please disable Swarm.Transports.Network.WebRTCDirect",
                )
            }
            opts.Opts = append(opts.Opts, libp2p.Transport(webrtc.New))
        }

        // 返回选项和空错误
        return opts, nil
    }
# 定义一个名为 BandwidthCounter 的函数，返回两个值：opts 和 reporter
func BandwidthCounter() (opts Libp2pOpts, reporter *metrics.BandwidthCounter) {
    # 创建一个新的带宽计数器
    reporter = metrics.NewBandwidthCounter()
    # 将带宽计数器添加到选项中
    opts.Opts = append(opts.Opts, libp2p.BandwidthReporter(reporter))
    # 返回选项和带宽计数器
    return opts, reporter
}
```