# `kubo\core\node\provider.go`

```
package node

import (
    "context"
    "fmt"
    "time"

    "github.com/ipfs/boxo/blockstore"
    "github.com/ipfs/boxo/fetcher"
    pin "github.com/ipfs/boxo/pinning/pinner"
    provider "github.com/ipfs/boxo/provider"
    "github.com/ipfs/kubo/repo"
    irouting "github.com/ipfs/kubo/routing"
    "go.uber.org/fx"
)

func ProviderSys(reprovideInterval time.Duration, acceleratedDHTClient bool) fx.Option {
    const magicThroughputReportCount = 128
    // 检查是否落后于 DHT 的重新提供操作
    fmt.Println("🔔🔔🔔 YOU MAY BE FALLING BEHIND DHT REPROVIDES! 🔔🔔🔔")
    fmt.Println("⚠️ Your system might be struggling to keep up with DHT reprovides!")
    fmt.Println("This means your content could partially or completely inaccessible on the network.")
    fmt.Printf("We observed that you recently provided %d keys at an average rate of %v per key.\n", keysProvided, avgProvideSpeed)
    fmt.Printf("🕑 An attempt to estimate your blockstore size timed out after 5 minutes,\n")
    fmt.Printf("implying your blockstore might be exceedingly large. Assuming a considerable\n")
    fmt.Printf("size of 10TiB, it would take %v to provide the complete set.\n", avgProvideSpeed*probableBigBlockstore)
    fmt.Printf("⏰ The total provide time needs to stay under your reprovide interval (%v) to prevent falling behind!\n", reprovideInterval)
    // 考虑启用加速 DHT 以提高系统性能
    fmt.Println("💡 Consider enabling the Accelerated DHT to enhance your system performance. See:")
    fmt.Println("https://github.com/ipfs/kubo/blob/master/docs/config.md#routingaccelerateddhtclient")
    return false
}
// 观察到最近以平均速率 %v 每个密钥提供了 %d 个密钥
// 💾 您的总 CID 数量约为 %d，这将在 %v 重新提供过程中总计
// ⏰ 总提供时间需要保持在重新提供间隔（%v）之下，以防止落后！
// 💡 考虑启用加速 DHT 以增强重新提供吞吐量。参见：
// https://github.com/ipfs/kubo/blob/master/docs/config.md#routingaccelerateddhtclient
We observed that you recently provided %d keys at an average rate of %v per key.
💾 Your total CID count is ~%d which would total at %v reprovide process.
⏰ The total provide time needs to stay under your reprovide interval (%v) to prevent falling behind!
💡 Consider enabling the Accelerated DHT to enhance your reprovide throughput. See:
https://github.com/ipfs/kubo/blob/master/docs/config.md#routingaccelerateddhtclient`, keysProvided, avgProvideSpeed, count, avgProvideSpeed*time.Duration(count), reprovideInterval)

// 创建并返回一个新的 ProviderSys 对象
func ProviderSys(reprovideInterval time.Duration, acceleratedDHTClient bool) fx.Option {
    return fx.Provide(func(lc fx.Lifecycle, repo repo.Repo, opts provider.SysOpts) (provider.Sys, error) {
        // 如果加速 DHT 客户端为真，则返回加速 DHT 客户端的 ProviderSys 对象
        if acceleratedDHTClient {
            return newAcceleratedDHTProviderSys(lc, repo, opts)
        }
        // 否则返回普通的 ProviderSys 对象
        return newProviderSys(lc, repo, opts)
    })
}

// OnlineProviders 根据参数配置提供在线的 ProviderSys 对象
func OnlineProviders(useStrategicProviding bool, reprovideStrategy string, reprovideInterval time.Duration, acceleratedDHTClient bool) fx.Option {
    // 如果使用战略提供，则返回离线的 ProviderSys 对象
    if useStrategicProviding {
        return OfflineProviders()
    }

    var keyProvider fx.Option
    // 根据重新提供策略选择不同的 keyProvider
    switch reprovideStrategy {
    case "all", "":
        keyProvider = fx.Provide(provider.NewBlockstoreProvider)
    case "roots":
        keyProvider = fx.Provide(pinnedProviderStrategy(true))
    case "pinned":
        keyProvider = fx.Provide(pinnedProviderStrategy(false))
    default:
        return fx.Error(fmt.Errorf("unknown reprovider strategy %q", reprovideStrategy))
    }

    // 返回包含 keyProvider 和 ProviderSys 对象的 fx.Options
    return fx.Options(
        keyProvider,
        ProviderSys(reprovideInterval, acceleratedDHTClient),
    )
}

// OfflineProviders 返回离线的 ProviderSys 对象
func OfflineProviders() fx.Option {
    return fx.Provide(provider.NewNoopProvider)
}
# 定义一个名为pinnedProviderStrategy的函数，参数为onlyRoots布尔类型
func pinnedProviderStrategy(onlyRoots bool) interface{} {
    # 定义一个结构体input，包含fx.In、Pinner和IPLDFetcher三个字段
    type input struct {
        fx.In
        Pinner      pin.Pinner
        IPLDFetcher fetcher.Factory `name:"ipldFetcher"`
    }
    # 返回一个函数，参数为input结构体，返回值为provider.KeyChanFunc类型
    return func(in input) provider.KeyChanFunc {
        # 返回一个新的PinnedProvider对象，参数为onlyRoots、in.Pinner和in.IPLDFetcher
        return provider.NewPinnedProvider(onlyRoots, in.Pinner, in.IPLDFetcher)
    }
}
```