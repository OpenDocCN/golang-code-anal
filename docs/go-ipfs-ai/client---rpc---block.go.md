# `kubo\client\rpc\block.go`

```go
package rpc

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "context"  // 导入 context 包，用于处理上下文
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"  // 导入 io 包，用于进行输入输出操作

    "github.com/ipfs/boxo/path"  // 导入路径相关的包
    "github.com/ipfs/go-cid"  // 导入 cid 相关的包
    iface "github.com/ipfs/kubo/core/coreiface"  // 导入 coreiface 包
    caopts "github.com/ipfs/kubo/core/coreiface/options"  // 导入 options 包
    mc "github.com/multiformats/go-multicodec"  // 导入 multicodec 包
    mh "github.com/multiformats/go-multihash"  // 导入 multihash 包
)

type BlockAPI HttpApi  // 定义 BlockAPI 结构体，继承自 HttpApi

type blockStat struct {  // 定义 blockStat 结构体
    Key   string  // 键
    BSize int `json:"Size"`  // 大小

    cid cid.Cid  // CID
}

func (s *blockStat) Size() int {  // 定义 Size 方法
    return s.BSize  // 返回大小
}

func (s *blockStat) Path() path.ImmutablePath {  // 定义 Path 方法
    return path.FromCid(s.cid)  // 返回路径
}

func (api *BlockAPI) Put(ctx context.Context, r io.Reader, opts ...caopts.BlockPutOption) (iface.BlockStat, error) {  // 定义 Put 方法
    options, err := caopts.BlockPutOptions(opts...)  // 获取选项
    px := options.CidPrefix  // 获取 CID 前缀
    if err != nil {  // 如果有错误
        return nil, err  // 返回空和错误
    }

    mht, ok := mh.Codes[px.MhType]  // 获取 mhType
    if !ok {  // 如果不存在
        return nil, fmt.Errorf("unknowm mhType %d", px.MhType)  // 返回错误
    }

    var cidOptKey, cidOptVal string  // 定义 CID 选项键和值
    switch {  // 开始 switch 语句
    case px.Version == 0 && px.Codec == cid.DagProtobuf:  // 如果版本为 0 并且编解码为 DagProtobuf
        // ensure legacy --format=v0 passes as BlockPutOption still works
        cidOptKey = "format"  // 设置键为 format
        cidOptVal = "v0"  // 设置值为 v0
    default:  // 默认情况
        // pass codec as string
        cidOptKey = "cid-codec"  // 设置键为 cid-codec
        cidOptVal = mc.Code(px.Codec).String()  // 设置值为编解码的字符串表示
    }

    req := api.core().Request("block/put").  // 创建请求对象
        Option("mhtype", mht).  // 设置选项 mhtype
        Option("mhlen", px.MhLength).  // 设置选项 mhlen
        Option(cidOptKey, cidOptVal).  // 设置 CID 选项键值
        Option("pin", options.Pin).  // 设置选项 pin
        FileBody(r)  // 设置文件体

    var out blockStat  // 定义 blockStat 对象
    if err := req.Exec(ctx, &out); err != nil {  // 如果执行请求出错
        return nil, err  // 返回空和错误
    }
    out.cid, err = cid.Parse(out.Key)  // 解析 CID
    if err != nil {  // 如果有错误
        return nil, err  // 返回空和错误
    }

    return &out, nil  // 返回结果和空
}

func (api *BlockAPI) Get(ctx context.Context, p path.Path) (io.Reader, error) {  // 定义 Get 方法
    resp, err := api.core().Request("block/get", p.String()).Send(ctx)  // 发送请求
    if err != nil {  // 如果有错误
        return nil, err  // 返回空和错误
    }
    // 如果响应中存在错误，则返回空值和解析后的错误，如果没有找到则返回错误
    if resp.Error != nil {
        return nil, parseErrNotFoundWithFallbackToError(resp.Error)
    }

    // TODO: make get return ReadCloser to avoid copying
    // 延迟关闭响应
    defer resp.Close()
    // 创建一个新的字节缓冲区
    b := new(bytes.Buffer)
    // 将响应输出复制到字节缓冲区中
    if _, err := io.Copy(b, resp.Output); err != nil {
        return nil, err
    }

    // 返回字节缓冲区和空值
    return b, nil
# 删除指定路径的块，可选参数为选项
func (api *BlockAPI) Rm(ctx context.Context, p path.Path, opts ...caopts.BlockRmOption) error {
    # 解析块删除选项
    options, err := caopts.BlockRmOptions(opts...)
    if err != nil {
        return err
    }

    # 定义一个结构体用于存储已删除的块的哈希和错误信息
    removedBlock := struct {
        Hash  string `json:",omitempty"`
        Error string `json:",omitempty"`
    }{}

    # 创建一个请求对象，指定路径和选项
    req := api.core().Request("block/rm").
        Option("force", options.Force).
        Arguments(p.String())

    # 执行请求，将结果存储到 removedBlock 中
    if err := req.Exec(ctx, &removedBlock); err != nil {
        return err
    }

    # 解析错误信息，如果是未找到错误则返回特定错误信息
    return parseErrNotFoundWithFallbackToMSG(removedBlock.Error)
}

# 获取指定路径块的状态信息
func (api *BlockAPI) Stat(ctx context.Context, p path.Path) (iface.BlockStat, error) {
    # 定义一个 blockStat 结构体变量用于存储块的状态信息
    var out blockStat
    # 执行请求，将结果存储到 out 中
    err := api.core().Request("block/stat", p.String()).Exec(ctx, &out)
    if err != nil {
        return nil, parseErrNotFoundWithFallbackToError(err)
    }
    # 解析块的 Key 字段为 CID 对象
    out.cid, err = cid.Parse(out.Key)
    if err != nil {
        return nil, err
    }

    return &out, nil
}

# 返回 BlockAPI 对应的 HttpApi 对象
func (api *BlockAPI) core() *HttpApi {
    return (*HttpApi)(api)
}
```