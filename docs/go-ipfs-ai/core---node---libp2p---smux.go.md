# `kubo\core\node\libp2p\smux.go`

```go
package libp2p

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "os"   // 导入 os 包，用于访问操作系统功能

    "github.com/ipfs/kubo/config"  // 导入外部包

    "github.com/libp2p/go-libp2p"  // 导入 libp2p 包
    "github.com/libp2p/go-libp2p/p2p/muxer/yamux"  // 导入 libp2p 中的 yamux 多路复用器
)

func makeSmuxTransportOption(tptConfig config.Transports) (libp2p.Option, error) {
    // 检查环境变量 LIBP2P_MUX_PREFS 是否为空，如果不为空则返回错误
    if prefs := os.Getenv("LIBP2P_MUX_PREFS"); prefs != "" {
        return nil, fmt.Errorf("configuring muxers with LIBP2P_MUX_PREFS is no longer supported, use Swarm.Transports.Multiplexers")
    }
    // 检查 tptConfig 中的 Mplex 是否为 0，如果不为 0 则返回错误
    if tptConfig.Multiplexers.Mplex != 0 {
        return nil, fmt.Errorf("Swarm.Transports.Multiplexers.Mplex is no longer supported, remove it from your config, see https://github.com/libp2p/specs/issues/553")
    }
    // 检查 tptConfig 中的 Yamux 是否小于 0，如果小于 0 则返回错误
    if tptConfig.Multiplexers.Yamux < 0 {
        return nil, fmt.Errorf("running libp2p with Swarm.Transports.Multiplexers.Yamux disabled is not supported")
    }

    // 返回 yamux 多路复用器的 libp2p 选项
    return libp2p.Muxer(yamux.ID, yamux.DefaultTransport), nil
}

func SmuxTransport(tptConfig config.Transports) func() (opts Libp2pOpts, err error) {
    // 返回一个函数，该函数返回 Libp2pOpts 和错误
    return func() (opts Libp2pOpts, err error) {
        // 调用 makeSmuxTransportOption 函数获取 libp2p 选项和错误
        res, err := makeSmuxTransportOption(tptConfig)
        // 如果有错误则返回 opts 和错误
        if err != nil {
            return opts, err
        }
        // 将结果追加到 opts.Opts 中并返回 opts 和 nil
        opts.Opts = append(opts.Opts, res)
        return opts, nil
    }
}
```