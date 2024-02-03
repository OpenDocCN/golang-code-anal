# `kubo\client\rpc\object.go`

```go
package rpc

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "context"  // 导入 context 包，用于处理上下文
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"  // 导入 io 包，用于进行输入输出操作

    "github.com/ipfs/boxo/ipld/merkledag"  // 导入 merkledag 包，用于处理 IPLD MerkleDAG 数据结构
    ft "github.com/ipfs/boxo/ipld/unixfs"  // 导入 unixfs 包，并重命名为 ft，用于处理 UnixFS 数据结构
    "github.com/ipfs/boxo/path"  // 导入 path 包，用于处理路径
    "github.com/ipfs/go-cid"  // 导入 cid 包，用于处理 CID（Content Identifier）
    ipld "github.com/ipfs/go-ipld-format"  // 导入 ipld 包，并重命名为 ipld，用于处理 IPLD 数据结构
    iface "github.com/ipfs/kubo/core/coreiface"  // 导入 coreiface 包，并重命名为 iface，用于定义 IPFS 核心接口
    caopts "github.com/ipfs/kubo/core/coreiface/options"  // 导入 options 包，并重命名为 caopts，用于定义 IPFS 核心接口选项
)

type ObjectAPI HttpApi  // 定义 ObjectAPI 结构体，继承自 HttpApi 结构体

type objectOut struct {  // 定义 objectOut 结构体
    Hash string  // 声明 Hash 字段，类型为字符串
}

func (api *ObjectAPI) New(ctx context.Context, opts ...caopts.ObjectNewOption) (ipld.Node, error) {  // 定义 New 方法，接收上下文和选项参数，返回 IPLD 节点和错误
    options, err := caopts.ObjectNewOptions(opts...)  // 获取选项参数
    if err != nil {  // 如果出现错误
        return nil, err  // 返回空和错误
    }

    var n ipld.Node  // 声明 IPLD 节点
    switch options.Type {  // 根据选项类型进行判断
    case "empty":  // 如果是 empty 类型
        n = new(merkledag.ProtoNode)  // 创建一个 ProtoNode 类型的节点
    case "unixfs-dir":  // 如果是 unixfs-dir 类型
        n = ft.EmptyDirNode()  // 创建一个空的 UnixFS 目录节点
    default:  // 其他情况
        return nil, fmt.Errorf("unknown object type: %s", options.Type)  // 返回空和错误信息
    }

    return n, nil  // 返回节点和空
}

func (api *ObjectAPI) Put(ctx context.Context, r io.Reader, opts ...caopts.ObjectPutOption) (path.ImmutablePath, error) {  // 定义 Put 方法，接收上下文、读取器和选项参数，返回不可变路径和错误
    options, err := caopts.ObjectPutOptions(opts...)  // 获取选项参数
    if err != nil {  // 如果出现错误
        return path.ImmutablePath{}, err  // 返回空路径和错误
    }

    var out objectOut  // 声明 objectOut 结构体
    err = api.core().Request("object/put").  // 发起对象存储请求
        Option("inputenc", options.InputEnc).  // 设置输入编码选项
        Option("datafieldenc", options.DataType).  // 设置数据字段编码选项
        Option("pin", options.Pin).  // 设置固定选项
        FileBody(r).  // 设置文件体
        Exec(ctx, &out)  // 执行请求并将结果存储到 out 中
    if err != nil {  // 如果出现错误
        return path.ImmutablePath{}, err  // 返回空路径和错误
    }

    c, err := cid.Parse(out.Hash)  // 解析哈希值
    if err != nil {  // 如果出现错误
        return path.ImmutablePath{}, err  // 返回空路径和错误
    }

    return path.FromCid(c), nil  // 返回路径和空
}

func (api *ObjectAPI) Get(ctx context.Context, p path.Path) (ipld.Node, error) {  // 定义 Get 方法，接收上下文和路径，返回 IPLD 节点和错误
    r, err := api.core().Block().Get(ctx, p)  // 获取块数据
    if err != nil {  // 如果出现错误
        return nil, err  // 返回空和错误
    }
    b, err := io.ReadAll(r)  // 读取所有数据
    if err != nil {  // 如果出现错误
        return nil, err  // 返回空和错误
    }

    return merkledag.DecodeProtobuf(b)  // 返回解码后的 Protobuf 数据
}

func (api *ObjectAPI) Data(ctx context.Context, p path.Path) (io.Reader, error) {  // 定义 Data 方法，接收上下文和路径，返回读取器和错误
    // 发送请求并获取响应
    resp, err := api.core().Request("object/data", p.String()).Send(ctx)
    // 如果发生错误，返回空和错误信息
    if err != nil {
        return nil, err
    }
    // 如果响应中包含错误信息，返回空和响应中的错误信息
    if resp.Error != nil {
        return nil, resp.Error
    }

    // 延迟关闭响应
    defer resp.Close()
    // 创建一个新的字节缓冲区
    b := new(bytes.Buffer)
    // 将响应输出复制到字节缓冲区中
    if _, err := io.Copy(b, resp.Output); err != nil {
        return nil, err
    }

    // 返回字节缓冲区和空
    return b, nil
// Links 方法用于获取指定路径下的所有链接信息，并返回链接数组和错误信息
func (api *ObjectAPI) Links(ctx context.Context, p path.Path) ([]*ipld.Link, error) {
    // 定义一个结构体变量 out 用于存储链接信息
    var out struct {
        Links []struct {
            Name string
            Hash string
            Size uint64
        }
    }
    // 发起 object/links 请求，并将结果存储到 out 变量中
    if err := api.core().Request("object/links", p.String()).Exec(ctx, &out); err != nil {
        return nil, err
    }
    // 根据 out 中的链接信息创建链接数组 res
    res := make([]*ipld.Link, len(out.Links))
    // 遍历 out 中的链接信息，将其转换为 ipld.Link 类型，并存储到 res 数组中
    for i, l := range out.Links {
        c, err := cid.Parse(l.Hash)
        if err != nil {
            return nil, err
        }

        res[i] = &ipld.Link{
            Cid:  c,
            Name: l.Name,
            Size: l.Size,
        }
    }

    return res, nil
}

// Stat 方法用于获取指定路径的对象统计信息，并返回对象统计信息和错误信息
func (api *ObjectAPI) Stat(ctx context.Context, p path.Path) (*iface.ObjectStat, error) {
    // 定义一个结构体变量 out 用于存储对象统计信息
    var out struct {
        Hash           string
        NumLinks       int
        BlockSize      int
        LinksSize      int
        DataSize       int
        CumulativeSize int
    }
    // 发起 object/stat 请求，并将结果存储到 out 变量中
    if err := api.core().Request("object/stat", p.String()).Exec(ctx, &out); err != nil {
        return nil, err
    }
    // 解析对象的哈希值，并存储到变量 c 中
    c, err := cid.Parse(out.Hash)
    if err != nil {
        return nil, err
    }
    // 返回对象统计信息
    return &iface.ObjectStat{
        Cid:            c,
        NumLinks:       out.NumLinks,
        BlockSize:      out.BlockSize,
        LinksSize:      out.LinksSize,
        DataSize:       out.DataSize,
        CumulativeSize: out.CumulativeSize,
    }, nil
}

// AddLink 方法用于向指定路径的对象添加链接，并返回新的路径和错误信息
func (api *ObjectAPI) AddLink(ctx context.Context, base path.Path, name string, child path.Path, opts ...caopts.ObjectAddLinkOption) (path.ImmutablePath, error) {
    // 解析添加链接的选项
    options, err := caopts.ObjectAddLinkOptions(opts...)
    if err != nil {
        return path.ImmutablePath{}, err
    }
    // 定义一个 objectOut 结构体变量 out 用于存储添加链接的结果
    var out objectOut
    // 发起 object/patch/add-link 请求，并将结果存储到 out 变量中
    err = api.core().Request("object/patch/add-link", base.String(), name, child.String()).
        Option("create", options.Create).
        Exec(ctx, &out)
    if err != nil {
        return path.ImmutablePath{}, err
    }
    // 解析添加链接后的对象哈希值，并存储到变量 c 中
    c, err := cid.Parse(out.Hash)
    # 如果错误不为空，则返回一个空的不可变路径和错误
    if err != nil:
        return path.ImmutablePath{}, err
    # 否则，根据 CID 创建路径并返回路径和空错误
    return path.FromCid(c), nil
# 删除链接，返回被删除链接的路径和错误信息
func (api *ObjectAPI) RmLink(ctx context.Context, base path.Path, link string) (path.ImmutablePath, error) {
    # 定义对象输出
    var out objectOut
    # 发起请求，执行删除链接操作
    err := api.core().Request("object/patch/rm-link", base.String(), link).
        Exec(ctx, &out)
    # 如果发生错误，返回空路径和错误信息
    if err != nil {
        return path.ImmutablePath{}, err
    }
    # 解析哈希值
    c, err := cid.Parse(out.Hash)
    # 如果解析失败，返回空路径和错误信息
    if err != nil {
        return path.ImmutablePath{}, err
    }
    # 返回路径
    return path.FromCid(c), nil
}

# 追加数据，返回追加数据后的路径和错误信息
func (api *ObjectAPI) AppendData(ctx context.Context, p path.Path, r io.Reader) (path.ImmutablePath, error) {
    # 定义对象输出
    var out objectOut
    # 发起请求，执行追加数据操作
    err := api.core().Request("object/patch/append-data", p.String()).
        FileBody(r).
        Exec(ctx, &out)
    # 如果发生错误，返回空路径和错误信息
    if err != nil {
        return path.ImmutablePath{}, err
    }
    # 解析哈希值
    c, err := cid.Parse(out.Hash)
    # 如果解析失败，返回空路径和错误信息
    if err != nil {
        return path.ImmutablePath{}, err
    }
    # 返回路径
    return path.FromCid(c), nil
}

# 设置数据，返回设置数据后的路径和错误信息
func (api *ObjectAPI) SetData(ctx context.Context, p path.Path, r io.Reader) (path.ImmutablePath, error) {
    # 定义对象输出
    var out objectOut
    # 发起请求，执行设置数据操作
    err := api.core().Request("object/patch/set-data", p.String()).
        FileBody(r).
        Exec(ctx, &out)
    # 如果发生错误，返回空路径和错误信息
    if err != nil {
        return path.ImmutablePath{}, err
    }
    # 解析哈希值
    c, err := cid.Parse(out.Hash)
    # 如果解析失败，返回空路径和错误信息
    if err != nil {
        return path.ImmutablePath{}, err
    }
    # 返回路径
    return path.FromCid(c), nil
}

# 定义变更结构体
type change struct {
    Type   iface.ChangeType
    Path   string
    Before cid.Cid
    After  cid.Cid
}

# 比较两个路径的差异，返回对象变更列表和错误信息
func (api *ObjectAPI) Diff(ctx context.Context, a path.Path, b path.Path) ([]iface.ObjectChange, error) {
    # 定义输出结构体
    var out struct {
        Changes []change
    }
    # 发起请求，执行比较差异操作
    if err := api.core().Request("object/diff", a.String(), b.String()).Exec(ctx, &out); err != nil {
        return nil, err
    }
    # 创建对象变更列表
    res := make([]iface.ObjectChange, len(out.Changes))
    # 遍历 out.Changes 列表，同时获取索引和值
    for i, ch := range out.Changes {
        # 根据索引 i 创建 iface.ObjectChange 对象，并设置 Type 和 Path 属性
        res[i] = iface.ObjectChange{
            Type: ch.Type,
            Path: ch.Path,
        }
        # 如果 ch.Before 不是空值，则设置 res[i].Before 为对应的 path.FromCid(ch.Before) 值
        if ch.Before != cid.Undef {
            res[i].Before = path.FromCid(ch.Before)
        }
        # 如果 ch.After 不是空值，则设置 res[i].After 为对应的 path.FromCid(ch.After) 值
        if ch.After != cid.Undef {
            res[i].After = path.FromCid(ch.After)
        }
    }
    # 返回结果列表 res 和空值
    return res, nil
# 结构体方法，返回ObjectAPI类型的指针转换为HttpApi类型的指针
func (api *ObjectAPI) core() *HttpApi {
    return (*HttpApi)(api)
}
```