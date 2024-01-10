# `kubo\core\coreapi\object.go`

```
package coreapi

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "context"  // 导入 context 包，用于处理上下文
    "encoding/base64"  // 导入 base64 包，用于进行 base64 编码
    "encoding/json"  // 导入 json 包，用于处理 JSON 数据
    "encoding/xml"  // 导入 xml 包，用于处理 XML 数据
    "errors"  // 导入 errors 包，用于处理错误
    "fmt"  // 导入 fmt 包，用于格式化输入输出
    "io"  // 导入 io 包，用于进行 I/O 操作

    dag "github.com/ipfs/boxo/ipld/merkledag"  // 导入 dag 包，用于处理 IPLD MerkleDAG 数据结构
    "github.com/ipfs/boxo/ipld/merkledag/dagutils"  // 导入 dagutils 包，用于处理 IPLD MerkleDAG 数据结构的工具
    ft "github.com/ipfs/boxo/ipld/unixfs"  // 导入 ft 包，用于处理 UnixFS 数据结构
    "github.com/ipfs/boxo/path"  // 导入 path 包，用于处理路径
    pin "github.com/ipfs/boxo/pinning/pinner"  // 导入 pin 包，用于进行固定
    cid "github.com/ipfs/go-cid"  // 导入 cid 包，用于处理 CID（Content Identifier）
    ipld "github.com/ipfs/go-ipld-format"  // 导入 ipld 包，用于处理 IPLD 数据格式
    coreiface "github.com/ipfs/kubo/core/coreiface"  // 导入 coreiface 包，用于处理核心接口
    caopts "github.com/ipfs/kubo/core/coreiface/options"  // 导入 caopts 包，用于处理核心接口的选项
    "go.opentelemetry.io/otel/attribute"  // 导入 attribute 包，用于处理属性
    "go.opentelemetry.io/otel/trace"  // 导入 trace 包，用于处理跟踪

    "github.com/ipfs/kubo/tracing"  // 导入 tracing 包，用于处理跟踪
)

const inputLimit = 2 << 20  // 定义常量 inputLimit，表示输入限制为 2 的 20 次方

type ObjectAPI CoreAPI  // 定义类型 ObjectAPI，表示核心 API

type Link struct {  // 定义结构体 Link
    Name, Hash string  // 包含 Name 和 Hash 两个字符串字段
    Size       uint64  // 包含 Size 一个无符号整数字段
}

type Node struct {  // 定义结构体 Node
    Links []Link  // 包含 Links 一个 Link 类型的切片字段
    Data  string  // 包含 Data 一个字符串字段
}

func (api *ObjectAPI) New(ctx context.Context, opts ...caopts.ObjectNewOption) (ipld.Node, error) {  // 定义方法 New，接收上下文和选项，返回 IPLD 节点和错误
    ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "New")  // 调用跟踪包的 Span 方法，创建跟踪上下文
    defer span.End()  // 延迟执行结束跟踪

    options, err := caopts.ObjectNewOptions(opts...)  // 调用核心接口选项的 ObjectNewOptions 方法，获取选项和错误
    if err != nil {  // 如果有错误
        return nil, err  // 返回空和错误
    }

    var n ipld.Node  // 声明变量 n 为 IPLD 节点
    switch options.Type {  // 根据选项的类型进行判断
    case "empty":  // 如果是 empty 类型
        n = new(dag.ProtoNode)  // 创建一个 ProtoNode 类型的节点
    case "unixfs-dir":  // 如果是 unixfs-dir 类型
        n = ft.EmptyDirNode()  // 创建一个空的 UnixFS 目录节点
    default:  // 其他情况
        return nil, fmt.Errorf("unknown node type: %s", options.Type)  // 返回空和错误
    }

    err = api.dag.Add(ctx, n)  // 调用核心 API 的 dag.Add 方法，添加节点
    if err != nil {  // 如果有错误
        return nil, err  // 返回空和错误
    }
    return n, nil  // 返回节点和空
}

func (api *ObjectAPI) Put(ctx context.Context, src io.Reader, opts ...caopts.ObjectPutOption) (path.ImmutablePath, error) {  // 定义方法 Put，接收上下文、读取器和选项，返回不可变路径和错误
    ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "Put")  // 调用跟踪包的 Span 方法，创建跟踪上下文
    defer span.End()  // 延迟执行结束跟踪

    options, err := caopts.ObjectPutOptions(opts...)  // 调用核心接口选项的 ObjectPutOptions 方法，获取选项和错误
    if err != nil {  // 如果有错误
        return path.ImmutablePath{}, err  // 返回空路径和错误
    }
    span.SetAttributes(  // 设置跟踪上下文的属性
        attribute.Bool("pin", options.Pin),  // 设置 pin 属性为选项的 Pin 值
        attribute.String("datatype", options.DataType),  // 设置 datatype 属性为选项的 DataType 值
        attribute.String("inputenc", options.InputEnc),  // 设置 inputenc 属性为选项的 InputEnc 值
    )
    // 读取输入流的数据，限制在inputLimit+10的范围内
    data, err := io.ReadAll(io.LimitReader(src, inputLimit+10))
    if err != nil {
        return path.ImmutablePath{}, err
    }

    // 定义一个指向dag.ProtoNode的指针变量
    var dagnode *dag.ProtoNode
    // 根据输入编码格式进行不同的处理
    switch options.InputEnc {
    case "json":
        // 如果输入编码格式为json，则解析json数据
        node := new(Node)
        decoder := json.NewDecoder(bytes.NewReader(data))
        decoder.DisallowUnknownFields()
        err = decoder.Decode(node)
        if err != nil {
            return path.ImmutablePath{}, err
        }

        // 将解析后的数据反序列化为dagnode
        dagnode, err = deserializeNode(node, options.DataType)
        if err != nil {
            return path.ImmutablePath{}, err
        }

    case "protobuf":
        // 如果输入编码格式为protobuf，则解析protobuf数据
        dagnode, err = dag.DecodeProtobuf(data)

    case "xml":
        // 如果输入编码格式为xml，则解析xml数据
        node := new(Node)
        err = xml.Unmarshal(data, node)
        if err != nil {
            return path.ImmutablePath{}, err
        }

        // 将解析后的数据反序列化为dagnode
        dagnode, err = deserializeNode(node, options.DataType)
        if err != nil {
            return path.ImmutablePath{}, err
        }

    default:
        // 如果输入编码格式未知，则返回错误
        return path.ImmutablePath{}, errors.New("unknown object encoding")
    }

    // 检查处理过程中是否有错误发生
    if err != nil {
        return path.ImmutablePath{}, err
    }

    // 如果需要Pin，则在函数返回时解锁PinLock
    if options.Pin {
        defer api.blockstore.PinLock(ctx).Unlock(ctx)
    }

    // 将dagnode添加到DAG中
    err = api.dag.Add(ctx, dagnode)
    if err != nil {
        return path.ImmutablePath{}, err
    }

    // 如果需要Pin，则将dagnode以递归方式Pin到pinning系统中
    if options.Pin {
        if err := api.pinning.PinWithMode(ctx, dagnode.Cid(), pin.Recursive, ""); err != nil {
            return path.ImmutablePath{}, err
        }

        // 将Pin缓冲区中的内容刷新到持久存储中
        err = api.pinning.Flush(ctx)
        if err != nil {
            return path.ImmutablePath{}, err
        }
    }

    // 返回dagnode的Cid作为ImmutablePath
    return path.FromCid(dagnode.Cid()), nil
# 获取对象的方法，根据给定的路径获取节点
func (api *ObjectAPI) Get(ctx context.Context, path path.Path) (ipld.Node, error) {
    # 创建一个跟踪 span，并在函数结束时结束该 span
    ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "Get", trace.WithAttributes(attribute.String("path", path.String())))
    defer span.End()
    # 调用核心 API 的 ResolveNode 方法来解析节点
    return api.core().ResolveNode(ctx, path)
}

# 获取对象数据的方法，根据给定的路径获取节点的数据
func (api *ObjectAPI) Data(ctx context.Context, path path.Path) (io.Reader, error) {
    # 创建一个跟踪 span，并在函数结束时结束该 span
    ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "Data", trace.WithAttributes(attribute.String("path", path.String())))
    defer span.End()
    # 调用核心 API 的 ResolveNode 方法来解析节点
    nd, err := api.core().ResolveNode(ctx, path)
    if err != nil {
        return nil, err
    }
    # 将节点转换为 ProtoNode 类型
    pbnd, ok := nd.(*dag.ProtoNode)
    if !ok {
        return nil, dag.ErrNotProtobuf
    }
    # 返回节点数据的读取器和 nil 错误
    return bytes.NewReader(pbnd.Data()), nil
}

# 获取对象链接的方法，根据给定的路径获取节点的链接
func (api *ObjectAPI) Links(ctx context.Context, path path.Path) ([]*ipld.Link, error) {
    # 创建一个跟踪 span，并在函数结束时结束该 span
    ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "Links", trace.WithAttributes(attribute.String("path", path.String())))
    defer span.End()
    # 调用核心 API 的 ResolveNode 方法来解析节点
    nd, err := api.core().ResolveNode(ctx, path)
    if err != nil {
        return nil, err
    }
    # 获取节点的链接
    links := nd.Links()
    # 创建一个与链接相同长度的切片
    out := make([]*ipld.Link, len(links))
    # 将节点的链接转换为 ipld.Link 类型，并存储到切片中
    for n, l := range links {
        out[n] = (*ipld.Link)(l)
    }
    # 返回链接切片和 nil 错误
    return out, nil
}

# 获取对象状态的方法，根据给定的路径获取节点的状态
func (api *ObjectAPI) Stat(ctx context.Context, path path.Path) (*coreiface.ObjectStat, error) {
    # 创建一个跟踪 span，并在函数结束时结束该 span
    ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "Stat", trace.WithAttributes(attribute.String("path", path.String())))
    defer span.End()
    # 调用核心 API 的 ResolveNode 方法来解析节点
    nd, err := api.core().ResolveNode(ctx, path)
    if err != nil {
        return nil, err
    }
    # 获取节点的状态
    stat, err := nd.Stat()
    if err != nil {
        return nil, err
    }
    # 创建一个包含节点状态信息的对象
    out := &coreiface.ObjectStat{
        Cid:            nd.Cid(),
        NumLinks:       stat.NumLinks,
        BlockSize:      stat.BlockSize,
        LinksSize:      stat.LinksSize,
        DataSize:       stat.DataSize,
        CumulativeSize: stat.CumulativeSize,
    }
    # 返回状态对象和 nil 错误
    return out, nil
}
func (api *ObjectAPI) AddLink(ctx context.Context, base path.Path, name string, child path.Path, opts ...caopts.ObjectAddLinkOption) (path.ImmutablePath, error) {
    // 创建一个跟踪 span，记录 AddLink 操作的相关属性
    ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "AddLink", trace.WithAttributes(
        attribute.String("base", base.String()),
        attribute.String("name", name),
        attribute.String("child", child.String()),
    ))
    // 在函数返回前结束 span
    defer span.End()

    // 解析 AddLink 选项
    options, err := caopts.ObjectAddLinkOptions(opts...)
    if err != nil {
        return path.ImmutablePath{}, err
    }
    // 设置 span 的属性，记录是否创建链接
    span.SetAttributes(attribute.Bool("create", options.Create))

    // 解析 base 节点
    baseNd, err := api.core().ResolveNode(ctx, base)
    if err != nil {
        return path.ImmutablePath{}, err
    }

    // 解析 child 节点
    childNd, err := api.core().ResolveNode(ctx, child)
    if err != nil {
        return path.ImmutablePath{}, err
    }

    // 检查 baseNd 是否为 ProtoNode 类型
    basePb, ok := baseNd.(*dag.ProtoNode)
    if !ok {
        return path.ImmutablePath{}, dag.ErrNotProtobuf
    }

    // 根据选项判断是否创建节点
    var createfunc func() *dag.ProtoNode
    if options.Create {
        createfunc = ft.EmptyDirNode
    }

    // 创建 DagEditor 对象
    e := dagutils.NewDagEditor(basePb, api.dag)

    // 在指定路径插入节点
    err = e.InsertNodeAtPath(ctx, name, childNd, createfunc)
    if err != nil {
        return path.ImmutablePath{}, err
    }

    // 完成编辑，获取最终节点
    nnode, err := e.Finalize(ctx, api.dag)
    if err != nil {
        return path.ImmutablePath{}, err
    }

    // 返回最终节点的路径
    return path.FromCid(nnode.Cid()), nil
}

func (api *ObjectAPI) RmLink(ctx context.Context, base path.Path, link string) (path.ImmutablePath, error) {
    // 创建一个跟踪 span，记录 RmLink 操作的相关属性
    ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "RmLink", trace.WithAttributes(
        attribute.String("base", base.String()),
        attribute.String("link", link)),
    )
    // 在函数返回前结束 span
    defer span.End()

    // 解析 base 节点
    baseNd, err := api.core().ResolveNode(ctx, base)
    if err != nil {
        return path.ImmutablePath{}, err
    }

    // 检查 baseNd 是否为 ProtoNode 类型
    basePb, ok := baseNd.(*dag.ProtoNode)
    if !ok {
        return path.ImmutablePath{}, dag.ErrNotProtobuf
    }
}
    # 创建一个新的DagEditor对象，使用给定的basePb和api.dag
    e := dagutils.NewDagEditor(basePb, api.dag)

    # 删除指定的链接，如果出现错误则返回空的ImmutablePath和错误信息
    err = e.RmLink(ctx, link)
    if err != nil {
        return path.ImmutablePath{}, err
    }

    # 对DagEditor对象进行最终操作，返回最终节点和可能的错误信息
    nnode, err := e.Finalize(ctx, api.dag)
    if err != nil {
        return path.ImmutablePath{}, err
    }

    # 从最终节点的Cid创建一个路径，返回该路径和空的错误信息
    return path.FromCid(nnode.Cid()), nil
# 定义一个方法，用于向对象追加数据
func (api *ObjectAPI) AppendData(ctx context.Context, path path.Path, r io.Reader) (path.ImmutablePath, error) {
    # 创建一个跟踪 span，并设置相关属性
    ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "AppendData", trace.WithAttributes(attribute.String("path", path.String())))
    # 延迟执行 span 的结束操作
    defer span.End()
    # 调用 patchData 方法，传入参数并指定为追加数据
    return api.patchData(ctx, path, r, true)
}

# 定义一个方法，用于设置对象的数据
func (api *ObjectAPI) SetData(ctx context.Context, path path.Path, r io.Reader) (path.ImmutablePath, error) {
    # 创建一个跟踪 span，并设置相关属性
    ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "SetData", trace.WithAttributes(attribute.String("path", path.String())))
    # 延迟执行 span 的结束操作
    defer span.End()
    # 调用 patchData 方法，传入参数并指定为不追加数据
    return api.patchData(ctx, path, r, false)
}

# 定义一个方法，用于处理数据的更新操作
func (api *ObjectAPI) patchData(ctx context.Context, p path.Path, r io.Reader, appendData bool) (path.ImmutablePath, error) {
    # 解析节点，获取节点信息
    nd, err := api.core().ResolveNode(ctx, p)
    if err != nil {
        return path.ImmutablePath{}, err
    }
    # 判断节点是否为 ProtoNode 类型
    pbnd, ok := nd.(*dag.ProtoNode)
    if !ok {
        return path.ImmutablePath{}, dag.ErrNotProtobuf
    }
    # 读取输入流中的所有数据
    data, err := io.ReadAll(r)
    if err != nil {
        return path.ImmutablePath{}, err
    }
    # 如果需要追加数据，则将数据追加到节点的数据中
    if appendData {
        data = append(pbnd.Data(), data...)
    }
    # 设置节点的数据
    pbnd.SetData(data)
    # 将节点添加到 DAG 中
    err = api.dag.Add(ctx, pbnd)
    if err != nil {
        return path.ImmutablePath{}, err
    }
    # 返回节点的 CID 信息
    return path.FromCid(pbnd.Cid()), nil
}

# 定义一个方法，用于比较两个节点之间的差异
func (api *ObjectAPI) Diff(ctx context.Context, before path.Path, after path.Path) ([]coreiface.ObjectChange, error) {
    # 创建一个跟踪 span，并设置相关属性
    ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "Diff", trace.WithAttributes(
        attribute.String("before", before.String()),
        attribute.String("after", after.String()),
    ))
    # 延迟执行 span 的结束操作
    defer span.End()
    # 解析 before 节点
    beforeNd, err := api.core().ResolveNode(ctx, before)
    if err != nil {
        return nil, err
    }
    # 解析 after 节点
    afterNd, err := api.core().ResolveNode(ctx, after)
    if err != nil {
        return nil, err
    }
    # 比较两个节点之间的差异
    changes, err := dagutils.Diff(ctx, api.dag, beforeNd, afterNd)
    if err != nil {
        return nil, err
    }
    # 创建一个长度与changes相同的空对象变化数组
    out := make([]coreiface.ObjectChange, len(changes))
    # 遍历changes数组，将每个变化对象的信息填入out数组中
    for i, change := range changes {
        out[i] = coreiface.ObjectChange{
            # 将change的类型转换为coreiface.ChangeType类型，并赋值给out[i]的Type字段
            Type: coreiface.ChangeType(change.Type),
            # 将change的路径赋值给out[i]的Path字段
            Path: change.Path,
        }

        # 如果change的Before字段有定义
        if change.Before.Defined() {
            # 将change的Before字段转换为路径，并赋值给out[i]的Before字段
            out[i].Before = path.FromCid(change.Before)
        }

        # 如果change的After字段有定义
        if change.After.Defined() {
            # 将change的After字段转换为路径，并赋值给out[i]的After字段
            out[i].After = path.FromCid(change.After)
        }
    }

    # 返回填充好数据的out数组和空值
    return out, nil
# 返回 ObjectAPI 的核心核心API
func (api *ObjectAPI) core() coreiface.CoreAPI {
    return (*CoreAPI)(api)
}

# 反序列化节点，根据数据字段编码返回 ProtoNode 和错误
func deserializeNode(nd *Node, dataFieldEncoding string) (*dag.ProtoNode, error) {
    # 创建一个新的 ProtoNode
    dagnode := new(dag.ProtoNode)
    # 根据数据字段编码类型进行不同的处理
    switch dataFieldEncoding {
    case "text":
        # 如果是文本编码，则设置节点数据为 nd.Data 的字节数组
        dagnode.SetData([]byte(nd.Data))
    case "base64":
        # 如果是 base64 编码，则解码 nd.Data，并设置为节点数据
        data, err := base64.StdEncoding.DecodeString(nd.Data)
        if err != nil {
            return nil, err
        }
        dagnode.SetData(data)
    default:
        # 如果是未知的数据字段编码类型，则返回错误
        return nil, fmt.Errorf("unknown data field encoding")
    }

    # 创建一个链接数组，长度为 nd.Links 的长度
    links := make([]*ipld.Link, len(nd.Links))
    # 遍历 nd.Links，将每个链接的哈希解码为 CID，并设置到链接数组中
    for i, link := range nd.Links {
        c, err := cid.Decode(link.Hash)
        if err != nil {
            return nil, err
        }
        links[i] = &ipld.Link{
            Name: link.Name,
            Size: link.Size,
            Cid:  c,
        }
    }
    # 设置节点的链接数组
    if err := dagnode.SetLinks(links); err != nil {
        return nil, err
    }

    # 返回 ProtoNode 和 nil 错误
    return dagnode, nil
}
```