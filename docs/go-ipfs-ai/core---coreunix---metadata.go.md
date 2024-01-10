# `kubo\core\coreunix\metadata.go`

```
// 导入所需的包
package coreunix

import (
    dag "github.com/ipfs/boxo/ipld/merkledag"  // 导入 merkledag 包并重命名为 dag
    ft "github.com/ipfs/boxo/ipld/unixfs"  // 导入 unixfs 包并重命名为 ft
    cid "github.com/ipfs/go-cid"  // 导入 go-cid 包并重命名为 cid
    core "github.com/ipfs/kubo/core"  // 导入 core 包并重命名为 core
)

// 为给定的 IpfsNode 添加元数据，返回新节点的字符串表示和可能的错误
func AddMetadataTo(n *core.IpfsNode, skey string, m *ft.Metadata) (string, error) {
    // 解码字符串表示的 CID
    c, err := cid.Decode(skey)
    if err != nil {
        return "", err
    }

    // 根据 CID 获取节点
    nd, err := n.DAG.Get(n.Context(), c)
    if err != nil {
        return "", err
    }

    // 创建新的 ProtoNode
    mdnode := new(dag.ProtoNode)
    // 获取元数据的字节表示
    mdata, err := ft.BytesForMetadata(m)
    if err != nil {
        return "", err
    }

    // 设置 ProtoNode 的数据
    mdnode.SetData(mdata)
    // 将原节点作为文件链接到新节点
    if err := mdnode.AddNodeLink("file", nd); err != nil {
        return "", err
    }

    // 将新节点添加到 DAG
    err = n.DAG.Add(n.Context(), mdnode)
    if err != nil {
        return "", err
    }

    // 返回新节点的字符串表示和 nil 错误
    return mdnode.Cid().String(), nil
}

// 获取给定节点的元数据，返回元数据和可能的错误
func Metadata(n *core.IpfsNode, skey string) (*ft.Metadata, error) {
    // 解码字符串表示的 CID
    c, err := cid.Decode(skey)
    if err != nil {
        return nil, err
    }

    // 根据 CID 获取节点
    nd, err := n.DAG.Get(n.Context(), c)
    if err != nil {
        return nil, err
    }

    // 将节点转换为 ProtoNode
    pbnd, ok := nd.(*dag.ProtoNode)
    if !ok {
        return nil, dag.ErrNotProtobuf
    }

    // 从 ProtoNode 的数据中获取元数据
    return ft.MetadataFromBytes(pbnd.Data())
}
```