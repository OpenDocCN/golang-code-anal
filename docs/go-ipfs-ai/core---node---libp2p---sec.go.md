# `kubo\core\node\libp2p\sec.go`

```go
package libp2p

import (
    "github.com/ipfs/kubo/config"  // 导入配置包

    "github.com/libp2p/go-libp2p"  // 导入 libp2p 包
    "github.com/libp2p/go-libp2p/p2p/security/noise"  // 导入 noise 安全传输包
    tls "github.com/libp2p/go-libp2p/p2p/security/tls"  // 导入 tls 安全传输包
)

const secioEnabledWarning = `The SECIO security transport was enabled in the config but is no longer supported.

SECIO disabled by default in go-ipfs 0.7 removed in go-ipfs 0.9. Please remove
Swarm.Transports.Security.SECIO from your IPFS config.`  // 定义 SECIO 被禁用的警告信息

func Security(enabled bool, tptConfig config.Transports) interface{} {
    if !enabled {  // 如果未启用安全传输
        return func() (opts Libp2pOpts) {  // 返回一个函数，该函数返回 Libp2pOpts 结构
            log.Errorf(`Your IPFS node has been configured to run WITHOUT ENCRYPTED CONNECTIONS.
        You will not be able to connect to any nodes configured to use encrypted connections`)  // 输出错误信息
            opts.Opts = append(opts.Opts, libp2p.NoSecurity)  // 将 NoSecurity 选项添加到 opts 结构中
            return opts  // 返回 opts 结构
        }
    }

    if _, enabled := tptConfig.Security.SECIO.WithDefault(config.Disabled); enabled {  // 如果 SECIO 被启用
        log.Error(secioEnabledWarning)  // 输出 SECIO 被禁用的警告信息
    }

    // 使用新的配置选项。
    return func() (opts Libp2pOpts) {  // 返回一个函数，该函数返回 Libp2pOpts 结构
        opts.Opts = append(opts.Opts, prioritizeOptions([]priorityOption{{
            priority:        tptConfig.Security.TLS,  // 设置 TLS 优先级
            defaultPriority: 200,  // 设置默认优先级为 200
            opt:             libp2p.Security(tls.ID, tls.New),  // 使用 TLS 创建安全传输
        }, {
            priority:        tptConfig.Security.Noise,  // 设置 Noise 优先级
            defaultPriority: 100,  // 设置默认优先级为 100
            opt:             libp2p.Security(noise.ID, noise.New),  // 使用 Noise 创建安全传输
        }}))
        return opts  // 返回 opts 结构
    }
}
```