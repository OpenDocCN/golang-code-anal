# `kubo\client\rpc\unixfs.go`

```go
package rpc

import (
    "context" // 引入上下文包，用于处理请求的上下文信息
    "encoding/json" // 引入 JSON 包，用于处理 JSON 数据
    "errors" // 引入错误包，用于处理错误信息
    "fmt" // 引入格式化包，用于格式化输出
    "io" // 引入输入输出包，用于处理输入输出操作

    "github.com/ipfs/boxo/files" // 引入文件包
    unixfs "github.com/ipfs/boxo/ipld/unixfs" // 引入 Unixfs 包
    unixfs_pb "github.com/ipfs/boxo/ipld/unixfs/pb" // 引入 Unixfs Protocol Buffers 包
    "github.com/ipfs/boxo/path" // 引入路径包
    "github.com/ipfs/go-cid" // 引入 CID 包
    iface "github.com/ipfs/kubo/core/coreiface" // 引入接口包
    caopts "github.com/ipfs/kubo/core/coreiface/options" // 引入选项包
    mh "github.com/multiformats/go-multihash" // 引入多哈希包
)

type addEvent struct {
    Name  string // 事件名称
    Hash  string `json:",omitempty"` // 哈希值，可选的 JSON 字段
    Bytes int64  `json:",omitempty"` // 字节数，可选的 JSON 字段
    Size  string `json:",omitempty"` // 大小，可选的 JSON 字段
}

type UnixfsAPI HttpApi // UnixfsAPI 结构体继承自 HttpApi 结构体

func (api *UnixfsAPI) Add(ctx context.Context, f files.Node, opts ...caopts.UnixfsAddOption) (path.ImmutablePath, error) {
    // Add 方法，接收上下文、文件节点和选项，返回不可变路径和错误信息
    options, _, err := caopts.UnixfsAddOptions(opts...) // 获取 UnixfsAddOptions
    if err != nil {
        return path.ImmutablePath{}, err // 如果出错，返回空路径和错误信息
    }

    mht, ok := mh.Codes[options.MhType] // 获取哈希类型
    if !ok {
        return path.ImmutablePath{}, fmt.Errorf("unknowm mhType %d", options.MhType) // 如果哈希类型未知，返回空路径和错误信息
    }

    req := api.core().Request("add"). // 创建请求对象
        Option("hash", mht). // 设置哈希选项
        Option("chunker", options.Chunker). // 设置分块器选项
        Option("cid-version", options.CidVersion). // 设置 CID 版本选项
        Option("fscache", options.FsCache). // 设置文件缓存选项
        Option("inline", options.Inline). // 设置内联选项
        Option("inline-limit", options.InlineLimit). // 设置内联限制选项
        Option("nocopy", options.NoCopy). // 设置不复制选项
        Option("only-hash", options.OnlyHash). // 设置仅哈希选项
        Option("pin", options.Pin). // 设置固定选项
        Option("silent", options.Silent). // 设置静默选项
        Option("progress", options.Progress) // 设置进度选项

    if options.RawLeavesSet {
        req.Option("raw-leaves", options.RawLeaves) // 如果设置了原始叶子节点选项，设置原始叶子节点选项
    }

    switch options.Layout {
    case caopts.BalancedLayout:
        // noop, default
    case caopts.TrickleLayout:
        req.Option("trickle", true) // 如果布局为 TrickleLayout，设置滴答选项为 true
    }

    d := files.NewMapDirectory(map[string]files.Node{"": f}) // 创建新的映射目录

    version, err := api.core().loadRemoteVersion() // 加载远程版本
    if err != nil {
        return path.ImmutablePath{}, err // 如果出错，返回空路径和错误信息
    }
    // 检查版本是否低于 encodedAbsolutePathVersion，返回布尔值
    useEncodedAbsPaths := version.LT(encodedAbsolutePathVersion)
    // 从请求中获取文件数据，使用 MultiFileReader 进行读取
    req.Body(files.NewMultiFileReader(d, false, useEncodedAbsPaths))
    // 声明一个 addEvent 类型的变量 out
    var out addEvent
    // 发送请求并获取响应
    resp, err := req.Send(ctx)
    // 如果发送请求时出现错误，返回空的 ImmutablePath 对象和错误
    if err != nil {
        return path.ImmutablePath{}, err
    }
    // 如果响应中包含错误，返回空的 ImmutablePath 对象和响应中的错误
    if resp.Error != nil {
        return path.ImmutablePath{}, resp.Error
    }
    // 延迟关闭响应输出流
    defer resp.Output.Close()
    // 创建一个 JSON 解码器，使用响应输出流作为输入
    dec := json.NewDecoder(resp.Output)
loop:
    for {
        // 声明一个 addEvent 类型的变量 evt
        var evt addEvent
        // 解码 JSON 数据到 evt 变量，如果没有错误则继续，如果遇到文件末尾则跳出循环，如果有其他错误则返回空路径和错误
        switch err := dec.Decode(&evt); err {
        case nil:
        case io.EOF:
            break loop
        default:
            return path.ImmutablePath{}, err
        }
        // 将 evt 赋值给 out
        out = evt

        // 如果 options.Events 不为空
        if options.Events != nil {
            // 创建一个 iface.AddEvent 类型的变量 ifevt，并赋值
            ifevt := &iface.AddEvent{
                Name:  out.Name,
                Size:  out.Size,
                Bytes: out.Bytes,
            }

            // 如果 out.Hash 不为空
            if out.Hash != "" {
                // 解析 out.Hash 成 cid 对象，如果有错误则返回空路径和错误
                c, err := cid.Parse(out.Hash)
                if err != nil {
                    return path.ImmutablePath{}, err
                }
                // 将 cid 对象转换成路径并赋值给 ifevt.Path
                ifevt.Path = path.FromCid(c)
            }

            // 将 ifevt 发送到 options.Events 通道，如果发送被阻塞则等待，如果 ctx 被取消则返回空路径和 ctx 错误
            select {
            case options.Events <- ifevt:
            case <-ctx.Done():
                return path.ImmutablePath{}, ctx.Err()
            }
        }
    }

    // 解析 out.Hash 成 cid 对象，如果有错误则返回空路径和错误
    c, err := cid.Parse(out.Hash)
    if err != nil {
        return path.ImmutablePath{}, err
    }

    // 将 cid 对象转换成路径并返回
    return path.FromCid(c), nil
}

// 定义 lsLink 结构体
type lsLink struct {
    Name, Hash string
    Size       uint64
    Type       unixfs_pb.Data_DataType
    Target     string
}

// 定义 lsObject 结构体
type lsObject struct {
    Hash  string
    Links []lsLink
}

// 定义 lsOutput 结构体
type lsOutput struct {
    Objects []lsObject
}

// 定义 UnixfsAPI 结构体的 Ls 方法
func (api *UnixfsAPI) Ls(ctx context.Context, p path.Path, opts ...caopts.UnixfsLsOption) (<-chan iface.DirEntry, error) {
    // 解析 UnixfsLsOptions
    options, err := caopts.UnixfsLsOptions(opts...)
    if err != nil {
        return nil, err
    }

    // 发送 ls 请求并获取响应
    resp, err := api.core().Request("ls", p.String()).
        Option("resolve-type", options.ResolveChildren).
        Option("size", options.ResolveChildren).
        Option("stream", true).
        Send(ctx)
    if err != nil {
        return nil, err
    }
    if resp.Error != nil {
        return nil, resp.Error
    }

    // 创建一个输出通道
    dec := json.NewDecoder(resp.Output)
    out := make(chan iface.DirEntry)
    # 创建一个匿名函数并立即执行
    go func() {
        # 延迟关闭 resp 变量
        defer resp.Close()
        # 延迟关闭 out 通道
        defer close(out)

        # 无限循环，读取数据并处理
        for {
            # 定义 lsOutput 类型的变量 link
            var link lsOutput
            # 使用 dec 对象解码数据到 link 变量
            if err := dec.Decode(&link); err != nil {
                # 如果解码出错
                if err == io.EOF:
                    # 如果是文件结束错误，结束循环
                    return
                # 如果不是文件结束错误
                select {
                    # 将错误信息发送到 out 通道
                    case out <- iface.DirEntry{Err: err}:
                    # 如果上下文被取消，结束循环
                    case <-ctx.Done():
                }
                return
            }

            # 检查 link 对象中的 Objects 数组长度是否为1
            if len(link.Objects) != 1 {
                # 如果不是1，发送错误信息到 out 通道
                select {
                    case out <- iface.DirEntry{Err: errors.New("unexpected Objects len")}:
                    case <-ctx.Done():
                }
                return
            }

            # 检查 link 对象中的第一个对象的 Links 数组长度是否为1
            if len(link.Objects[0].Links) != 1 {
                # 如果不是1，发送错误信息到 out 通道
                select {
                    case out <- iface.DirEntry{Err: errors.New("unexpected Links len")}:
                    case <-ctx.Done():
                }
                return
            }

            # 获取 link 对象中的第一个对象的第一个链接
            l0 := link.Objects[0].Links[0]

            # 解码 l0.Hash 字段为 cid 对象
            c, err := cid.Decode(l0.Hash)
            if err != nil {
                # 如果解码出错，发送错误信息到 out 通道
                select {
                    case out <- iface.DirEntry{Err: err}:
                    case <-ctx.Done():
                }
                return
            }

            # 根据 l0.Type 类型确定文件类型
            var ftype iface.FileType
            switch l0.Type {
                case unixfs.TRaw, unixfs.TFile:
                    ftype = iface.TFile
                case unixfs.THAMTShard, unixfs.TDirectory, unixfs.TMetadata:
                    ftype = iface.TDirectory
                case unixfs.TSymlink:
                    ftype = iface.TSymlink
            }

            # 将文件信息发送到 out 通道
            select {
                case out <- iface.DirEntry{
                    Name:   l0.Name,
                    Cid:    c,
                    Size:   l0.Size,
                    Type:   ftype,
                    Target: l0.Target,
                }:
                case <-ctx.Done():
            }
        }
    }()

    # 返回 out 通道和空错误
    return out, nil
# 定义一个方法，返回UnixfsAPI类型的指针对应的HttpApi类型的指针
func (api *UnixfsAPI) core() *HttpApi {
    # 返回UnixfsAPI类型的指针对应的HttpApi类型的指针
    return (*HttpApi)(api)
}
```