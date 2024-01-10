# `kubo\gc\gc_test.go`

```
package gc

import (
    "context"
    "testing"

    "github.com/ipfs/boxo/blockservice"
    "github.com/ipfs/boxo/blockstore"
    "github.com/ipfs/boxo/exchange/offline"
    "github.com/ipfs/boxo/ipld/merkledag"
    mdutils "github.com/ipfs/boxo/ipld/merkledag/test"
    pin "github.com/ipfs/boxo/pinning/pinner"
    "github.com/ipfs/boxo/pinning/pinner/dspinner"
    "github.com/ipfs/go-cid"
    "github.com/ipfs/go-datastore"
    dssync "github.com/ipfs/go-datastore/sync"
    "github.com/multiformats/go-multihash"
    "github.com/stretchr/testify/require"
)

func TestGC(t *testing.T) {
    ctx := context.Background() // 创建一个上下文对象

    ds := dssync.MutexWrap(datastore.NewMapDatastore()) // 使用同步锁包装数据存储对象
    bs := blockstore.NewGCBlockstore(blockstore.NewBlockstore(ds), blockstore.NewGCLocker()) // 创建一个带垃圾回收功能的区块存储对象
    bserv := blockservice.New(bs, offline.Exchange(bs)) // 创建一个区块服务对象
    dserv := merkledag.NewDAGService(bserv) // 创建一个基于区块服务的有向无环图服务对象
    pinner, err := dspinner.New(ctx, ds, dserv) // 创建一个新的固定器对象
    require.NoError(t, err) // 断言没有错误发生

    daggen := mdutils.NewDAGGenerator() // 创建一个有向无环图生成器对象

    var expectedKept []multihash.Multihash // 声明一个多哈希数组用于存储预期保留的哈希值
    var expectedDiscarded []multihash.Multihash // 声明一个多哈希数组用于存储预期丢弃的哈希值

    // add some pins
    for i := 0; i < 5; i++ { // 循环5次
        // direct
        root, _, err := daggen.MakeDagNode(dserv.Add, 0, 1) // 创建一个有向无环图节点，并将其添加到服务中
        require.NoError(t, err) // 断言没有错误发生
        err = pinner.PinWithMode(ctx, root, pin.Direct, "") // 使用指定模式将节点固定到存储中
        require.NoError(t, err) // 断言没有错误发生
        expectedKept = append(expectedKept, root.Hash()) // 将节点的哈希值添加到预期保留的哈希值数组中

        // recursive
        root, allCids, err := daggen.MakeDagNode(dserv.Add, 5, 2) // 创建一个有向无环图节点，并将其添加到服务中
        require.NoError(t, err) // 断言没有错误发生
        err = pinner.PinWithMode(ctx, root, pin.Recursive, "") // 使用递归模式将节点固定到存储中
        require.NoError(t, err) // 断言没有错误发生
        expectedKept = append(expectedKept, toMHs(allCids)...) // 将所有节点的哈希值添加到预期保留的哈希值数组中
    }

    err = pinner.Flush(ctx) // 刷新固定器
    require.NoError(t, err) // 断言没有错误发生

    // add more dags to be GCed
    for i := 0; i < 5; i++ { // 循环5次
        _, allCids, err := daggen.MakeDagNode(dserv.Add, 5, 2) // 创建一个有向无环图节点，并将其添加到服务中
        require.NoError(t, err) // 断言没有错误发生
        expectedDiscarded = append(expectedDiscarded, toMHs(allCids)...) // 将所有节点的哈希值添加到预期丢弃的哈希值数组中
    }
}
    // 定义一个空的CID切片，用于存储“尽力而为”的根CID
    var bestEffortRoots []cid.Cid
    // 循环5次，生成DAG节点并获取根CID和所有CID
    for i := 0; i < 5; i++ {
        root, allCids, err := daggen.MakeDagNode(dserv.Add, 5, 2)
        require.NoError(t, err)
        // 将根CID添加到bestEffortRoots切片中
        bestEffortRoots = append(bestEffortRoots, root)
        // 将所有CID转换为Multihash并添加到expectedKept切片中
        expectedKept = append(expectedKept, toMHs(allCids)...)
    }

    // 调用GC函数进行垃圾回收，传入上面生成的根CID切片
    ch := GC(ctx, bs, ds, pinner, bestEffortRoots)
    // 定义一个空的Multihash切片，用于存储被丢弃的数据的Multihash
    var discarded []multihash.Multihash
    // 遍历GC函数返回的通道，获取每个结果并将其对应的Multihash添加到discarded切片中
    for res := range ch {
        require.NoError(t, res.Error)
        discarded = append(discarded, res.KeyRemoved.Hash())
    }

    // 获取存储在数据存储中的所有数据的CID
    allKeys, err := bs.AllKeysChan(ctx)
    require.NoError(t, err)
    // 定义一个空的Multihash切片，用于存储保留的数据的Multihash
    var kept []multihash.Multihash
    // 遍历所有数据的CID，将其转换为Multihash并添加到kept切片中
    for key := range allKeys {
        kept = append(kept, key.Hash())
    }

    // 检查被丢弃的数据的Multihash是否符合预期
    require.ElementsMatch(t, expectedDiscarded, discarded)
    // 检查保留的数据的Multihash是否符合预期
    require.ElementsMatch(t, expectedKept, kept)
# 将CID数组转换为Multihash数组
func toMHs(cids []cid.Cid) []multihash.Multihash:
    # 创建一个与CID数组长度相同的Multihash数组
    res := make([]multihash.Multihash, len(cids))
    # 遍历CID数组，获取每个CID的哈希值，并存入Multihash数组中
    for i, c := range cids:
        res[i] = c.Hash()
    # 返回Multihash数组
    return res
```