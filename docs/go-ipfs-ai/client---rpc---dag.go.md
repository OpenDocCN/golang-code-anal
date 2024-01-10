# `kubo\client\rpc\dag.go`

```
package rpc

import (
    "bytes" // 导入 bytes 包，用于操作字节流
    "context" // 导入 context 包，用于处理上下文
    "fmt" // 导入 fmt 包，用于格式化输出
    "io" // 导入 io 包，用于输入输出操作

    "github.com/ipfs/boxo/path" // 导入路径相关的包
    blocks "github.com/ipfs/go-block-format" // 导入区块格式相关的包
    "github.com/ipfs/go-cid" // 导入 CID 相关的包
    format "github.com/ipfs/go-ipld-format" // 导入 IPLD 格式相关的包
    "github.com/ipfs/kubo/core/coreiface/options" // 导入选项相关的包
    multicodec "github.com/multiformats/go-multicodec" // 导入多编解码格式相关的包
)

type (
    httpNodeAdder        HttpApi // 定义 httpNodeAdder 类型，继承自 HttpApi
    HttpDagServ          httpNodeAdder // 定义 HttpDagServ 类型，继承自 httpNodeAdder
    pinningHttpNodeAdder httpNodeAdder // 定义 pinningHttpNodeAdder 类型，继承自 httpNodeAdder
)

func (api *HttpDagServ) Get(ctx context.Context, c cid.Cid) (format.Node, error) {
    r, err := api.core().Block().Get(ctx, path.FromCid(c)) // 获取指定 CID 对应的区块数据
    if err != nil {
        return nil, err
    }

    data, err := io.ReadAll(r) // 读取区块数据
    if err != nil {
        return nil, err
    }

    blk, err := blocks.NewBlockWithCid(data, c) // 创建带有指定 CID 的新区块
    if err != nil {
        return nil, err
    }

    return api.ipldDecoder.DecodeNode(ctx, blk) // 使用 IPLD 解码器解码区块数据并返回节点
}

func (api *HttpDagServ) GetMany(ctx context.Context, cids []cid.Cid) <-chan *format.NodeOption {
    out := make(chan *format.NodeOption) // 创建用于存放节点选项的通道

    for _, c := range cids {
        // TODO: Consider limiting concurrency of this somehow
        go func(c cid.Cid) {
            n, err := api.Get(ctx, c) // 获取指定 CID 对应的节点

            select {
            case out <- &format.NodeOption{Node: n, Err: err}: // 将节点和错误信息发送到通道
            case <-ctx.Done():
            }
        }(c)
    }
    return out // 返回通道
}

func (api *httpNodeAdder) add(ctx context.Context, nd format.Node, pin bool) error {
    c := nd.Cid() // 获取节点的 CID
    prefix := c.Prefix() // 获取 CID 的前缀信息

    // preserve 'cid-codec' when sent over HTTP
    cidCodec := multicodec.Code(prefix.Codec).String() // 获取 CID 编解码格式

    // 'format' got replaced by 'cid-codec' in https://github.com/ipfs/interface-go-ipfs-core/pull/80
    // but we still support it here for backward-compatibility with use of CIDv0
    format := "" // 初始化 format 变量为空字符串
    if prefix.Version == 0 {
        cidCodec = "" // 如果 CID 版本为 0，则清空 cidCodec 变量
        format = "v0" // 设置 format 变量为 "v0"
    }
}
    // 调用api.core()获取核心API对象，然后调用Block()获取块API对象，并调用Put()方法上传数据块，
    // 传入上下文ctx、数据的字节流、哈希选项、CID编解码器、数据格式和是否固定的选项
    stat, err := api.core().Block().Put(ctx, bytes.NewReader(nd.RawData()),
        options.Block.Hash(prefix.MhType, prefix.MhLength),
        options.Block.CidCodec(cidCodec),
        options.Block.Format(format),
        options.Block.Pin(pin))
    // 如果上传过程中出现错误，则返回错误
    if err != nil {
        return err
    }
    // 如果上传成功，检查返回的路径是否与预期的CID匹配，如果不匹配则返回错误
    if !stat.Path().RootCid().Equals(c) {
        return fmt.Errorf("cids didn't match - local %s, remote %s", c.String(), stat.Path().RootCid().String())
    }
    // 如果CID匹配，则返回nil
    return nil
# 在 httpNodeAdder 结构体中定义了 addMany 方法，用于批量添加节点到系统中
func (api *httpNodeAdder) addMany(ctx context.Context, nds []format.Node, pin bool) error {
    # 遍历传入的节点数组
    for _, nd := range nds {
        # TODO: optimize，优化待完成
        # 调用 add 方法将节点添加到系统中
        if err := api.add(ctx, nd, pin); err != nil {
            return err
        }
    }
    return nil
}

# 在 HttpDagServ 结构体中定义了 AddMany 方法，用于向系统中添加多个节点
func (api *HttpDagServ) AddMany(ctx context.Context, nds []format.Node) error {
    # 调用 httpNodeAdder 结构体的 addMany 方法，pin 参数为 false
    return (*httpNodeAdder)(api).addMany(ctx, nds, false)
}

# 在 HttpDagServ 结构体中定义了 Add 方法，用于向系统中添加单个节点
func (api *HttpDagServ) Add(ctx context.Context, nd format.Node) error {
    # 调用 httpNodeAdder 结构体的 add 方法，pin 参数为 false
    return (*httpNodeAdder)(api).add(ctx, nd, false)
}

# 在 pinningHttpNodeAdder 结构体中定义了 Add 方法，用于向系统中添加单个节点并进行固定
func (api *pinningHttpNodeAdder) Add(ctx context.Context, nd format.Node) error {
    # 调用 httpNodeAdder 结构体的 add 方法，pin 参数为 true
    return (*httpNodeAdder)(api).add(ctx, nd, true)
}

# 在 pinningHttpNodeAdder 结构体中定义了 AddMany 方法，用于向系统中添加多个节点并进行固定
func (api *pinningHttpNodeAdder) AddMany(ctx context.Context, nds []format.Node) error {
    # 调用 httpNodeAdder 结构体的 addMany 方法，pin 参数为 true
    return (*httpNodeAdder)(api).addMany(ctx, nds, true)
}

# 在 httpNodeAdder 结构体中定义了 core 方法，用于获取 HttpApi 结构体指针
func (api *httpNodeAdder) core() *HttpApi {
    return (*HttpApi)(api)
}

# 在 HttpDagServ 结构体中定义了 core 方法，用于获取 HttpApi 结构体指针
func (api *HttpDagServ) core() *HttpApi {
    return (*HttpApi)(api)
}
```