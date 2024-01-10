# `kubo\core\node\libp2p\routingopt_test.go`

```
package libp2p

import (
    "testing"

    config "github.com/ipfs/kubo/config"  // 导入配置包
    "github.com/stretchr/testify/require"  // 导入断言包
)

func TestHttpAddrsFromConfig(t *testing.T) {
    require.Equal(t, []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/udp/4001/quic-v1"},
        httpAddrsFromConfig(config.Addresses{  // 测试从配置中获取 HTTP 地址
            Swarm: []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/udp/4001/quic-v1"},
        }), "Swarm addrs should be taken by default")  // 断言默认情况下应该获取 Swarm 地址

    require.Equal(t, []string{"/ip4/192.168.0.1/tcp/4001"},
        httpAddrsFromConfig(config.Addresses{  // 测试从配置中获取 HTTP 地址
            Swarm:    []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/udp/4001/quic-v1"},
            Announce: []string{"/ip4/192.168.0.1/tcp/4001"},  // 用指定的 Announce 地址覆盖 Swarm 地址
        }), "Announce addrs should override Swarm if specified")  // 断言如果指定了 Announce 地址，应该覆盖 Swarm 地址

    require.Equal(t, []string{"/ip4/0.0.0.0/udp/4001/quic-v1"},
        httpAddrsFromConfig(config.Addresses{  // 测试从配置中获取 HTTP 地址
            Swarm:      []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/udp/4001/quic-v1"},
            NoAnnounce: []string{"/ip4/0.0.0.0/tcp/4001"},  // 断言 Swarm 地址不应包含 NoAnnounce 地址
        }), "Swarm addrs should not contain NoAnnounce addrs")

    require.Equal(t, []string{"/ip4/192.168.0.1/tcp/4001", "/ip4/192.168.0.2/tcp/4001"},
        httpAddrsFromConfig(config.Addresses{  // 测试从配置中获取 HTTP 地址
            Swarm:          []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/udp/4001/quic-v1"},
            Announce:       []string{"/ip4/192.168.0.1/tcp/4001"},  // 指定 Announce 地址
            AppendAnnounce: []string{"/ip4/192.168.0.2/tcp/4001"},  // 包含指定的 AppendAnnounce 地址
        }), "AppendAnnounce addrs should be included if specified")  // 断言如果指定了 AppendAnnounce 地址，应该包含在结果中
}
```