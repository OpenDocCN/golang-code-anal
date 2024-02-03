# `kubo\core\node\libp2p\nat.go`

```go
// 导入必要的包
package libp2p

import (
    "time" // 导入时间包

    config "github.com/ipfs/kubo/config" // 导入配置包
    "github.com/libp2p/go-libp2p" // 导入 libp2p 包
)

// 定义 NATPortMap 变量，使用 simpleOpt 函数对 libp2p.NATPortMap() 进行封装
var NatPortMap = simpleOpt(libp2p.NATPortMap())

// 定义 AutoNATService 函数，接受 throttle 参数，并返回一个函数类型 Libp2pOpts
func AutoNATService(throttle *config.AutoNATThrottleConfig) func() Libp2pOpts {
    return func() (opts Libp2pOpts) {
        // 向 opts.Opts 切片中添加 libp2p.EnableNATService() 选项
        opts.Opts = append(opts.Opts, libp2p.EnableNATService())
        // 如果 throttle 不为 nil，则向 opts.Opts 切片中添加 libp2p.AutoNATServiceRateLimit() 选项
        if throttle != nil {
            opts.Opts = append(opts.Opts,
                libp2p.AutoNATServiceRateLimit(
                    throttle.GlobalLimit,
                    throttle.PeerLimit,
                    throttle.Interval.WithDefault(time.Minute),
                ),
            )
        }
        // 返回 opts
        return opts
    }
}
```