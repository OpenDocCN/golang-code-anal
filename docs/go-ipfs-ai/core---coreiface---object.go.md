# `kubo\core\coreiface\object.go`

```go
package iface

import (
    "context"  // 导入上下文包，用于处理请求的取消、截止和超时
    "io"  // 导入输入输出包，提供了基本的接口和函数
    "github.com/ipfs/boxo/path"  // 导入路径包，用于处理路径相关的操作
    "github.com/ipfs/kubo/core/coreiface/options"  // 导入选项包，提供了核心接口的选项

    "github.com/ipfs/go-cid"  // 导入 CID 包，用于处理内容标识符
    ipld "github.com/ipfs/go-ipld-format"  // 导入 IPLD 包，用于处理分布式图形数据结构
)

// ObjectStat provides information about dag nodes
type ObjectStat struct {
    // Cid is the CID of the node
    Cid cid.Cid  // CID 是节点的内容标识符
    // NumLinks is number of links the node contains
    NumLinks int  // NumLinks 是节点包含的链接数
    // BlockSize is size of the raw serialized node
    BlockSize int  // BlockSize 是原始序列化节点的大小
    // LinksSize is size of the links block section
    LinksSize int  // LinksSize 是链接块部分的大小
    // DataSize is the size of data block section
    DataSize int  // DataSize 是数据块部分的大小
    // CumulativeSize is size of the tree (BlockSize + link sizes)
    CumulativeSize int  // CumulativeSize 是树的大小（BlockSize + 链接大小）
}

// ChangeType denotes type of change in ObjectChange
type ChangeType int  // ChangeType 表示 ObjectChange 中的更改类型

const (
    // DiffAdd is set when a link was added to the graph
    DiffAdd ChangeType = iota  // DiffAdd 在将链接添加到图形时设置
    // DiffRemove is set when a link was removed from the graph
    DiffRemove  // DiffRemove 在从图形中删除链接时设置
    // DiffMod is set when a link was changed in the graph
    DiffMod  // DiffMod 在图形中更改链接时设置
)

// ObjectChange represents a change ia a graph
type ObjectChange struct {
    // Type of the change, either:
    // * DiffAdd - Added a link
    // * DiffRemove - Removed a link
    // * DiffMod - Modified a link
    Type ChangeType  // 更改的类型，可以是 DiffAdd、DiffRemove 或 DiffMod
    // Path to the changed link
    Path string  // 更改的链接路径
    // Before holds the link path before the change. Note that when a link is
    // added, this will be nil.
    Before path.ImmutablePath  // 更改之前的链接路径。注意，当添加链接时，这将为 nil。
    // After holds the link path after the change. Note that when a link is
    // removed, this will be nil.
    After path.ImmutablePath  // 更改后的链接路径。注意，当删除链接时，这将为 nil。
}

// ObjectAPI specifies the interface to MerkleDAG and contains useful utilities
// for manipulating MerkleDAG data structures.
type ObjectAPI interface {
    // New creates new, empty (by default) dag-node.
    New(context.Context, ...options.ObjectNewOption) (ipld.Node, error)  // New 创建新的、空的（默认）DAG 节点
    // Put imports the data into merkledag
    // 使用给定的上下文和读取器将数据放入存储，并可以传入额外的选项
    Put(context.Context, io.Reader, ...options.ObjectPutOption) (path.ImmutablePath, error)

    // 根据路径返回节点
    Get(context.Context, path.Path) (ipld.Node, error)

    // 根据路径返回节点的数据读取器
    Data(context.Context, path.Path) (io.Reader, error)

    // 根据路径返回节点包含的链接或引用
    Links(context.Context, path.Path) ([]*ipld.Link, error)

    // 根据路径返回节点的信息
    Stat(context.Context, path.Path) (*ObjectStat, error)

    // 在指定路径下添加一个链接。子路径可以指向父路径中的子目录，该子目录必须存在（可以使用 WithCreate 选项进行覆盖）
    AddLink(ctx context.Context, base path.Path, name string, child path.Path, opts ...options.ObjectAddLinkOption) (path.ImmutablePath, error)

    // 从节点中删除一个链接
    RmLink(ctx context.Context, base path.Path, link string) (path.ImmutablePath, error)

    // 向节点追加数据
    AppendData(context.Context, path.Path, io.Reader) (path.ImmutablePath, error)

    // 设置节点中包含的数据
    SetData(context.Context, path.Path, io.Reader) (path.ImmutablePath, error)

    // 返回将第一个对象转换为第二个对象所需的一组更改
    Diff(context.Context, path.Path, path.Path) ([]ObjectChange, error)
# 闭合前面的函数定义
```