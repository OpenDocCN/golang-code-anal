# `kubo\core\coreunix\metadata_test.go`

```go
package coreunix

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "context"  // 导入 context 包，用于处理上下文
    "io"  // 导入 io 包，用于进行 I/O 操作
    "testing"  // 导入 testing 包，用于编写测试

    bserv "github.com/ipfs/boxo/blockservice"  // 导入 boxo/blockservice 包，并重命名为 bserv
    merkledag "github.com/ipfs/boxo/ipld/merkledag"  // 导入 boxo/ipld/merkledag 包，并重命名为 merkledag
    ft "github.com/ipfs/boxo/ipld/unixfs"  // 导入 boxo/ipld/unixfs 包，并重命名为 ft
    importer "github.com/ipfs/boxo/ipld/unixfs/importer"  // 导入 boxo/ipld/unixfs/importer 包，并重命名为 importer
    uio "github.com/ipfs/boxo/ipld/unixfs/io"  // 导入 boxo/ipld/unixfs/io 包，并重命名为 uio
    core "github.com/ipfs/kubo/core"  // 导入 kubo/core 包，并重命名为 core

    bstore "github.com/ipfs/boxo/blockstore"  // 导入 boxo/blockstore 包，并重命名为 bstore
    chunker "github.com/ipfs/boxo/chunker"  // 导入 boxo/chunker 包，并重命名为 chunker
    offline "github.com/ipfs/boxo/exchange/offline"  // 导入 boxo/exchange/offline 包，并重命名为 offline
    u "github.com/ipfs/boxo/util"  // 导入 boxo/util 包，并重命名为 u
    cid "github.com/ipfs/go-cid"  // 导入 go-cid 包，并重命名为 cid
    ds "github.com/ipfs/go-datastore"  // 导入 go-datastore 包，并重命名为 ds
    dssync "github.com/ipfs/go-datastore/sync"  // 导入 go-datastore/sync 包，并重命名为 dssync
    ipld "github.com/ipfs/go-ipld-format"  // 导入 go-ipld-format 包，并重命名为 ipld
)

func getDagserv(t *testing.T) ipld.DAGService {
    db := dssync.MutexWrap(ds.NewMapDatastore())  // 创建一个新的 MapDatastore 并使用 MutexWrap 进行包装
    bs := bstore.NewBlockstore(db)  // 使用 MapDatastore 创建一个新的 Blockstore
    blockserv := bserv.New(bs, offline.Exchange(bs))  // 使用 Blockstore 和 offline.Exchange 创建一个新的 Blockservice
    return merkledag.NewDAGService(blockserv)  // 返回一个基于 Blockservice 的新的 DAGService
}

func TestMetadata(t *testing.T) {
    ctx := context.Background()  // 创建一个新的空白上下文
    // Make some random node
    ds := getDagserv(t)  // 获取一个 DAGService
    data := make([]byte, 1000)  // 创建一个长度为 1000 的字节切片
    _, err := io.ReadFull(u.NewTimeSeededRand(), data)  // 从随机数生成器中读取随机数据到字节切片中
    if err != nil {  // 如果发生错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }
    r := bytes.NewReader(data)  // 创建一个从字节切片中读取数据的 Reader
    nd, err := importer.BuildDagFromReader(ds, chunker.DefaultSplitter(r))  // 使用 DAGService 和默认的分块器构建一个 DAG
    if err != nil {  // 如果发生错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }

    c := nd.Cid()  // 获取 DAG 的 CID

    m := new(ft.Metadata)  // 创建一个新的 Metadata 对象
    m.MimeType = "THIS IS A TEST"  // 设置 MIME 类型为 "THIS IS A TEST"

    // Such effort, many compromise
    ipfsnode := &core.IpfsNode{DAG: ds}  // 创建一个包含 DAGService 的 IpfsNode 对象

    mdk, err := AddMetadataTo(ipfsnode, c.String(), m)  // 向 IpfsNode 添加 Metadata，并获取返回的 CID
    if err != nil {  // 如果发生错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }

    rec, err := Metadata(ipfsnode, mdk)  // 从 IpfsNode 获取 Metadata
    if err != nil {  // 如果发生错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }
    if rec.MimeType != m.MimeType {  // 如果获取的 MIME 类型与设置的不一致
        t.Fatalf("something went wrong in conversion: '%s' != '%s'", rec.MimeType, m.MimeType)  // 输出错误信息并终止测试
    }

    cdk, err := cid.Decode(mdk)  // 解码返回的 CID
    if err != nil {  // 如果发生错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }

    retnode, err := ds.Get(ctx, cdk)  // 从 DAGService 获取指定 CID 的节点
    if err != nil {  // 如果发生错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }
}
    // 将 retnode 转换为 merkledag.ProtoNode 类型的变量 rtnpb，同时判断是否转换成功
    rtnpb, ok := retnode.(*merkledag.ProtoNode)
    if !ok {
        // 如果转换失败，则输出错误信息并终止测试
        t.Fatal("expected protobuf node")
    }

    // 使用 rtnpb 创建一个新的 DagReader 对象 ndr
    ndr, err := uio.NewDagReader(ctx, rtnpb, ds)
    if err != nil {
        // 如果创建过程中出现错误，则输出错误信息并终止测试
        t.Fatal(err)
    }

    // 从 ndr 中读取所有数据并存储到 out 中
    out, err := io.ReadAll(ndr)
    if err != nil {
        // 如果读取过程中出现错误，则输出错误信息并终止测试
        t.Fatal(err)
    }

    // 比较读取的数据 out 和原始数据 data 是否相等，如果不相等则输出错误信息并终止测试
    if !bytes.Equal(out, data) {
        t.Fatal("read incorrect data")
    }
# 闭合前面的函数定义
```