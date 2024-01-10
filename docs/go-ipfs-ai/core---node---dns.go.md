# `kubo\core\node\dns.go`

```
package node

import (
    "math"  // 导入 math 包，用于数学计算
    "time"  // 导入 time 包，用于时间相关操作

    "github.com/ipfs/boxo/gateway"  // 导入 boxo/gateway 包，用于网关操作
    config "github.com/ipfs/kubo/config"  // 导入 kubo/config 包，用于配置操作
    doh "github.com/libp2p/go-doh-resolver"  // 导入 go-doh-resolver 包，用于 DNS-over-HTTPS 解析
    madns "github.com/multiformats/go-multiaddr-dns"  // 导入 go-multiaddr-dns 包，用于多地址 DNS 解析
)

func DNSResolver(cfg *config.Config) (*madns.Resolver, error) {
    var dohOpts []doh.Option  // 声明 dohOpts 变量，用于存储 doh.Option 类型的切片
    if !cfg.DNS.MaxCacheTTL.IsDefault() {  // 如果配置中的 DNS 最大缓存时间不是默认值
        dohOpts = append(dohOpts, doh.WithMaxCacheTTL(cfg.DNS.MaxCacheTTL.WithDefault(time.Duration(math.MaxUint32)*time.Second)))  // 将配置中的最大缓存时间添加到 dohOpts 中
    }

    return gateway.NewDNSResolver(cfg.DNS.Resolvers, dohOpts...)  // 返回一个新的 DNS 解析器
}
```