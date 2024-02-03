# `kubo\core\node\libp2p\rcmgr_defaults.go`

```go
package libp2p

import (
    "fmt"

    "github.com/dustin/go-humanize"
    "github.com/ipfs/kubo/config"
    "github.com/ipfs/kubo/core/node/libp2p/fd"
    "github.com/libp2p/go-libp2p"
    rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager"
    "github.com/pbnjay/memory"
)

// 定义一个全局变量，表示资源限制为无限
var infiniteResourceLimits = rcmgr.InfiniteLimits.ToPartialLimitConfig().System

// This file defines implicit limit defaults used when Swarm.ResourceMgr.Enabled

// createDefaultLimitConfig creates LimitConfig to pass to libp2p's resource manager.
// The defaults follow the documentation in docs/libp2p-resource-management.md.
// Any changes in the logic here should be reflected there.
// 创建默认的限制配置，用于传递给 libp2p 的资源管理器
// 默认值遵循 docs/libp2p-resource-management.md 中的文档
// 任何此处逻辑的更改都应在那里反映出来。

func createDefaultLimitConfig(cfg config.SwarmConfig) (limitConfig rcmgr.ConcreteLimitConfig, logMessageForStartup string, err error) {
    // 计算默认的最大内存限制
    maxMemoryDefaultString := humanize.Bytes(uint64(memory.TotalMemory()) / 2)
    // 获取配置中的最大内存限制，如果没有则使用默认值
    maxMemoryString := cfg.ResourceMgr.MaxMemory.WithDefault(maxMemoryDefaultString)
    // 解析最大内存限制的字符串表示
    maxMemory, err := humanize.ParseBytes(maxMemoryString)
    if err != nil {
        return rcmgr.ConcreteLimitConfig{}, "", err
    }

    // 将最大内存限制转换为 MB
    maxMemoryMB := maxMemory / (1024 * 1024)
    // 获取配置中的最大文件描述符限制，如果没有则使用默认值
    maxFD := int(cfg.ResourceMgr.MaxFileDescriptors.WithDefault(int64(fd.GetNumFDs()) / 2))

    // 截至 2023-01-25，可能会打开一个不要求任何内存使用的连接
    // 与 libp2p 资源管理器/会计师（参见 https://github.com/libp2p/go-libp2p/issues/2010#issuecomment-1404280736）。
    // 因此，我们目前无法依赖内存限制来完全保护我们。
    // 在 https://github.com/libp2p/go-libp2p/issues/2010 得到解决之前，
    // 我们现在采取的代理是限制每 MB 至多 1 个入站连接。
    // 注意：这比 go-libp2p 的默认自动缩放限制更宽松，后者为每 1GB 64 个连接
    // （参见 https://github.com/libp2p/go-libp2p/blob/master/p2p/host/resource-manager/limit_defaults.go#L357）。
    // 目前采取的代理是限制每 MB 至多 1 个入站连接。
    // 注意：这比 go-libp2p 的默认自动缩放限制更宽松，后者为每 1GB 64 个连接
    // （参见 https://github.com/libp2p/go-libp2p/blob/master/p2p/host/resource-manager/limit_defaults.go#L357）。
    // 设置系统入站连接数为最大内存的1倍
    systemConnsInbound := int(1 * maxMemoryMB)

    }

    // 设置默认的服务限制为rcmgr.DefaultLimits
    scalingLimitConfig := rcmgr.DefaultLimits
    libp2p.SetDefaultServiceLimits(&scalingLimitConfig)

    // 以上部分限制中设置为rcmgr.DefaultLimit的任何内容都将被覆盖。
    // 在上面的partialLimits中未定义的scalingLimitConfig中的任何内容都将被添加（例如，libp2p的默认服务限制）。
    partialLimits = partialLimits.Build(scalingLimitConfig.Scale(int64(maxMemory), maxFD)).ToPartialLimitConfig()

    // 简单检查以覆盖自动缩放，确保限制与connmgr值相符。
    // 有办法打破这一点，但这应该已经捕捉到大部分问题。
    // 我们可能会在将来改进这一点。
    // 参见：https://github.com/ipfs/kubo/issues/9545
    if partialLimits.System.ConnsInbound > rcmgr.DefaultLimit && cfg.ConnMgr.Type.WithDefault(config.DefaultConnMgrType) != "none" {
        maxInboundConns := int64(partialLimits.System.ConnsInbound)
        if connmgrHighWaterTimesTwo := cfg.ConnMgr.HighWater.WithDefault(config.DefaultConnMgrHighWater) * 2; maxInboundConns < connmgrHighWaterTimesTwo {
            maxInboundConns = connmgrHighWaterTimesTwo
        }

        if maxInboundConns < config.DefaultResourceMgrMinInboundConns {
            maxInboundConns = config.DefaultResourceMgrMinInboundConns
        }

        // 也缩放System.StreamsInbound，但使用StreamsInbound与ConnsInbound的现有比率
        if partialLimits.System.StreamsInbound > rcmgr.DefaultLimit {
            partialLimits.System.StreamsInbound = rcmgr.LimitVal(maxInboundConns * int64(partialLimits.System.StreamsInbound) / int64(partialLimits.System.ConnsInbound))
        }
        partialLimits.System.ConnsInbound = rcmgr.LimitVal(maxInboundConns)
    }

    msg := fmt.Sprintf(`
# 根据默认的 go-libp2p资源管理器限制计算得出的值：
# - 'Swarm.ResourceMgr.MaxMemory': %q
# - 'Swarm.ResourceMgr.MaxFileDescriptors': %d
# 这些值可以通过 'ipfs swarm resources' 进行检查。

# 使用已经完整的数值，因此传入一个空的 ConcreteLimitConfig。
return partialLimits.Build(rcmgr.ConcreteLimitConfig{}), msg, nil
# 返回部分限制的构建结果和消息，没有错误
```