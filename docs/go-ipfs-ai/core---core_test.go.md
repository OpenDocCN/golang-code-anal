# `kubo\core\core_test.go`

```go
package core

import (
    "testing"  // 导入测试包
    context "context"  // 导入上下文包

    "github.com/ipfs/kubo/repo"  // 导入repo包

    datastore "github.com/ipfs/go-datastore"  // 导入数据存储包
    syncds "github.com/ipfs/go-datastore/sync"  // 导入同步数据存储包
    config "github.com/ipfs/kubo/config"  // 导入配置包
)

func TestInitialization(t *testing.T) {
    ctx := context.Background()  // 创建一个空的上下文
    id := testIdentity  // 设置测试身份

    good := []*config.Config{  // 创建一个好的配置数组
        {
            Identity: id,  // 设置身份
            Addresses: config.Addresses{  // 设置地址
                Swarm: []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/udp/4001/quic-v1"},  // 设置Swarm地址
                API:   []string{"/ip4/127.0.0.1/tcp/8000"},  // 设置API地址
            },
        },

        {
            Identity: id,  // 设置身份
            Addresses: config.Addresses{  // 设置地址
                Swarm: []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/udp/4001/quic-v1"},  // 设置Swarm地址
                API:   []string{"/ip4/127.0.0.1/tcp/8000"},  // 设置API地址
            },
        },
    }

    bad := []*config.Config{  // 创建一个坏的配置数组
        {},
    }

    for i, c := range good {  // 遍历好的配置数组
        r := &repo.Mock{  // 创建repo模拟对象
            C: *c,  // 设置配置
            D: syncds.MutexWrap(datastore.NewMapDatastore()),  // 设置数据存储
        }
        n, err := NewNode(ctx, &BuildCfg{Repo: r})  // 创建新节点
        if n == nil || err != nil {  // 如果节点为空或者有错误
            t.Error("Should have constructed.", i, err)  // 输出错误信息
        }
    }

    for i, c := range bad {  // 遍历坏的配置数组
        r := &repo.Mock{  // 创建repo模拟对象
            C: *c,  // 设置配置
            D: syncds.MutexWrap(datastore.NewMapDatastore()),  // 设置数据存储
        }
        n, err := NewNode(ctx, &BuildCfg{Repo: r})  // 创建新节点
        if n != nil || err == nil {  // 如果节点不为空或者没有错误
            t.Error("Should have failed to construct.", i)  // 输出错误信息
        }
    }
}

var testIdentity = config.Identity{  // 设置测试身份
    PeerID:  "QmNgdzLieYi8tgfo2WfTUzNVH5hQK9oAYGVf6dxN12NrHt",  // 设置对等ID
}
```