# `kubo\core\node\libp2p\addrs.go`

```
package libp2p

import (
    "fmt"

    "github.com/libp2p/go-libp2p"  // 导入 libp2p 包
    p2pbhost "github.com/libp2p/go-libp2p/p2p/host/basic"  // 导入 p2pbhost 包
    ma "github.com/multiformats/go-multiaddr"  // 导入 ma 包
    mamask "github.com/whyrusleeping/multiaddr-filter"  // 导入 mamask 包
)

// 定义一个函数 AddrFilters，接受一个字符串数组作为参数，返回一个函数，该函数返回 ma.Filters、Libp2pOpts 和 error
func AddrFilters(filters []string) func() (*ma.Filters, Libp2pOpts, error) {
    return func() (filter *ma.Filters, opts Libp2pOpts, err error) {
        filter = ma.NewFilters()  // 创建一个新的 ma.Filters 对象
        opts.Opts = append(opts.Opts, libp2p.ConnectionGater((*filtersConnectionGater)(filter)))  // 将 ConnectionGater 添加到 opts.Opts 中
        for _, s := range filters {  // 遍历 filters 数组
            f, err := mamask.NewMask(s)  // 使用 mamask 包中的 NewMask 函数创建一个新的 Mask 对象
            if err != nil {  // 如果有错误
                return filter, opts, fmt.Errorf("incorrectly formatted address filter in config: %s", s)  // 返回错误信息
            }
            filter.AddFilter(*f, ma.ActionDeny)  // 将新创建的 Filter 添加到 filter 中
        }
        return filter, opts, nil  // 返回 filter、opts 和 nil
    }
}

// 定义一个函数 makeAddrsFactory，接受三个字符串数组作为参数，返回 p2pbhost.AddrsFactory 和 error
func makeAddrsFactory(announce []string, appendAnnouce []string, noAnnounce []string) (p2pbhost.AddrsFactory, error) {
    var err error  // 声明一个错误变量，用于在 for 循环中赋值
    existing := make(map[string]bool)  // 创建一个 map，用于避免重复

    annAddrs := make([]ma.Multiaddr, len(announce))  // 创建一个长度为 announce 数组长度的 ma.Multiaddr 数组
    for i, addr := range announce {  // 遍历 announce 数组
        annAddrs[i], err = ma.NewMultiaddr(addr)  // 使用 ma 包中的 NewMultiaddr 函数创建一个新的 Multiaddr 对象
        if err != nil {  // 如果有错误
            return nil, err  // 返回 nil 和错误信息
        }
        existing[addr] = true  // 将地址添加到 existing map 中
    }

    var appendAnnAddrs []ma.Multiaddr  // 声明一个 ma.Multiaddr 数组
    for _, addr := range appendAnnouce {  // 遍历 appendAnnouce 数组
        if existing[addr] {  // 如果地址已经存在
            // skip AppendAnnounce that is on the Announce list already
            continue  // 跳过当前循环
        }
        appendAddr, err := ma.NewMultiaddr(addr)  // 使用 ma 包中的 NewMultiaddr 函数创建一个新的 Multiaddr 对象
        if err != nil {  // 如果有错误
            return nil, err  // 返回 nil 和错误信息
        }
        appendAnnAddrs = append(appendAnnAddrs, appendAddr)  // 将新创建的 Multiaddr 添加到 appendAnnAddrs 数组中
    }

    filters := ma.NewFilters()  // 创建一个新的 ma.Filters 对象
    noAnnAddrs := map[string]bool{}  // 创建一个空的 map
}
    for _, addr := range noAnnounce {
        // 遍历 noAnnounce 切片中的地址
        f, err := mamask.NewMask(addr)
        // 使用地址创建新的过滤器
        if err == nil {
            // 如果没有错误，将过滤器添加到 filters 中并继续下一个地址
            filters.AddFilter(*f, ma.ActionDeny)
            continue
        }
        // 如果有错误，将地址转换为 Multiaddr 对象
        maddr, err := ma.NewMultiaddr(addr)
        if err != nil {
            // 如果转换出错，返回空值和错误
            return nil, err
        }
        // 将地址的字节表示作为键，值为 true 存入 noAnnAddrs 字典中
        noAnnAddrs[string(maddr.Bytes())] = true
    }

    // 返回一个匿名函数和空值
    return func(allAddrs []ma.Multiaddr) []ma.Multiaddr {
        var addrs []ma.Multiaddr
        if len(annAddrs) > 0 {
            // 如果 annAddrs 切片不为空，将其赋值给 addrs
            addrs = annAddrs
        } else {
            // 如果 annAddrs 切片为空，将 allAddrs 赋值给 addrs
            addrs = allAddrs
        }
        // 将 appendAnnAddrs 切片中的地址追加到 addrs 切片中
        addrs = append(addrs, appendAnnAddrs...)

        var out []ma.Multiaddr
        for _, maddr := range addrs {
            // 遍历 addrs 切片中的地址
            // 检查是否存在精确匹配
            ok := noAnnAddrs[string(maddr.Bytes())]
            // 检查是否存在 /ipcidr 匹配，并且地址未被 filters 阻止
            if !ok && !filters.AddrBlocked(maddr) {
                // 如果不存在精确匹配并且地址未被阻止，将地址追加到 out 切片中
                out = append(out, maddr)
            }
        }
        // 返回 out 切片
        return out
    }, nil
# 创建一个函数，用于生成Libp2pOpts和错误的函数
func AddrsFactory(announce []string, appendAnnouce []string, noAnnounce []string) func() (opts Libp2pOpts, err error) {
    # 返回一个函数，该函数返回Libp2pOpts和错误
    return func() (opts Libp2pOpts, err error) {
        # 调用makeAddrsFactory函数，传入参数announce, appendAnnouce, noAnnounce，并接收返回的addrsFactory和err
        addrsFactory, err := makeAddrsFactory(announce, appendAnnouce, noAnnounce)
        # 如果出现错误，则返回opts和err
        if err != nil {
            return opts, err
        }
        # 将addrsFactory作为libp2p.AddrsFactory的参数，添加到opts.Opts中
        opts.Opts = append(opts.Opts, libp2p.AddrsFactory(addrsFactory))
        # 返回opts和nil的错误
        return
    }
}

# 创建一个函数，用于生成监听地址的Libp2pOpts
func ListenOn(addresses []string) interface{} {
    # 返回一个函数，该函数返回Libp2pOpts
    return func() (opts Libp2pOpts) {
        # 返回包含libp2p.ListenAddrStrings(addresses...)的Libp2pOpts
        return Libp2pOpts{
            Opts: []libp2p.Option{
                libp2p.ListenAddrStrings(addresses...),
            },
        }
    }
}
```