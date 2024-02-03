# `kubo\core\node\libp2p\topicdiscovery.go`

```go
package libp2p

import (
    "math/rand"  // 导入数学随机数包
    "time"  // 导入时间包

    "github.com/libp2p/go-libp2p/core/discovery"  // 导入 libp2p 的发现功能包
    "github.com/libp2p/go-libp2p/core/host"  // 导入 libp2p 的主机功能包
    "github.com/libp2p/go-libp2p/p2p/discovery/backoff"  // 导入 libp2p 的发现功能的退避包
    disc "github.com/libp2p/go-libp2p/p2p/discovery/routing"  // 导入 libp2p 的发现功能的路由包

    "github.com/libp2p/go-libp2p/core/routing"  // 导入 libp2p 的路由功能包
)

func TopicDiscovery() interface{} {
    return func(host host.Host, cr routing.ContentRouting) (service discovery.Discovery, err error) {
        baseDisc := disc.NewRoutingDiscovery(cr)  // 创建基于内容路由的发现功能
        minBackoff, maxBackoff := time.Second*60, time.Hour  // 设置最小和最大退避时间
        rng := rand.New(rand.NewSource(rand.Int63()))  // 创建随机数生成器
        d, err := backoff.NewBackoffDiscovery(  // 创建带有退避功能的发现功能
            baseDisc,
            backoff.NewExponentialBackoff(minBackoff, maxBackoff, backoff.FullJitter, time.Second, 5.0, 0, rng),
        )
        if err != nil {  // 如果发生错误
            return nil, err  // 返回空和错误
        }

        return d, nil  // 返回发现功能和空错误
    }
}
```