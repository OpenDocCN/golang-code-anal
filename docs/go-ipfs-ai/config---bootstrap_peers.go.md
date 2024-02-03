# `kubo\config\bootstrap_peers.go`

```go
// 引入必要的包
package config

import (
    "errors"
    "fmt"

    peer "github.com/libp2p/go-libp2p/core/peer"
    ma "github.com/multiformats/go-multiaddr"
)

// DefaultBootstrapAddresses 是 IPFS 的硬编码引导地址
// 它们是由 IPFS 团队运行的节点。稍后会有关于这些的文档。
// 与所有的 p2p 网络一样，引导是一个重要的安全问题。
//
// 注意：这里不在 cmd/ipfs/init.go 中，而是因为存在一个导入依赖问题。TODO: 将其移动到 config/default/ 包中。
var DefaultBootstrapAddresses = []string{
    "/dnsaddr/bootstrap.libp2p.io/p2p/QmNnooDu7bfjPFoTZYxMNLWUQJyrVwtbZg5gBMjTezGAJN",
    "/dnsaddr/bootstrap.libp2p.io/p2p/QmQCU2EcMqAqQPR2i9bChDtGNJchTbq5TbXJJ16u19uLTa",
    "/dnsaddr/bootstrap.libp2p.io/p2p/QmbLHAnMoJPWSCR5Zhtx6BHJX9KiKNN6tpvbUcqanj75Nb",
    "/dnsaddr/bootstrap.libp2p.io/p2p/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt",
    "/ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ",         // mars.i.ipfs.io
    "/ip4/104.131.131.82/udp/4001/quic-v1/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ", // mars.i.ipfs.io
}

// ErrInvalidPeerAddr 表示地址不是有效的对等地址。
var ErrInvalidPeerAddr = errors.New("invalid peer address")

// BootstrapPeers 返回配置中引导对等节点的地址信息
func (c *Config) BootstrapPeers() ([]peer.AddrInfo, error) {
    return ParseBootstrapPeers(c.Bootstrap)
}

// DefaultBootstrapPeers 返回默认引导对等节点的（解析后的）集合。
// 如果失败，它会为用户返回一个有意义的错误。
// 这里不在 cmd/ipfs/init 中，而是因为存在模块依赖问题。
func DefaultBootstrapPeers() ([]peer.AddrInfo, error) {
    ps, err := ParseBootstrapPeers(DefaultBootstrapAddresses)
    if err != nil {
        return nil, fmt.Errorf(`failed to parse hardcoded bootstrap peers: %w
This is a problem with the ipfs codebase. Please report it to the dev team`, err)
    }
    return ps, nil
}

// SetBootstrapPeers 设置引导对等节点
func (c *Config) SetBootstrapPeers(bps []peer.AddrInfo) {
    # 使用给定的参数创建 BootstrapPeerStrings 对象，并将其赋值给 c.Bootstrap
    c.Bootstrap = BootstrapPeerStrings(bps)
// 解析引导节点列表为 AddrInfos 列表
func ParseBootstrapPeers(addrs []string) ([]peer.AddrInfo, error) {
    // 创建一个 Multiaddr 切片，长度与输入地址切片相同
    maddrs := make([]ma.Multiaddr, len(addrs))
    // 遍历输入地址切片，将每个地址转换为 Multiaddr 对象
    for i, addr := range addrs {
        var err error
        maddrs[i], err = ma.NewMultiaddr(addr)
        // 如果转换出错，返回错误
        if err != nil {
            return nil, err
        }
    }
    // 将 Multiaddr 切片转换为 AddrInfo 切片
    return peer.AddrInfosFromP2pAddrs(maddrs...)
}

// 将 AddrInfos 列表格式化为适合序列化的引导节点列表
func BootstrapPeerStrings(bps []peer.AddrInfo) []string {
    // 创建一个字符串切片，初始容量为 AddrInfos 切片的长度
    bpss := make([]string, 0, len(bps))
    // 遍历 AddrInfos 切片，将每个 AddrInfo 转换为 P2P 地址
    for _, pi := range bps {
        addrs, err := peer.AddrInfoToP2pAddrs(&pi)
        // 如果转换出错，抛出异常
        if err != nil {
            // 程序员错误
            panic(err)
        }
        // 将 P2P 地址添加到字符串切片中
        for _, addr := range addrs {
            bpss = append(bpss, addr.String())
        }
    }
    // 返回字符串切片
    return bpss
}
```