# `kubo\core\coreiface\tests\block.go`

```
package tests

import (
    "bytes" // 导入 bytes 包，用于操作字节流
    "context" // 导入 context 包，用于控制程序的上下文
    "io" // 导入 io 包，用于进行 I/O 操作
    "strings" // 导入 strings 包，用于操作字符串
    "testing" // 导入 testing 包，用于编写测试函数

    "github.com/ipfs/boxo/path" // 导入路径相关的包
    ipld "github.com/ipfs/go-ipld-format" // 导入 ipld 包，用于处理 IPLD 数据格式
    coreiface "github.com/ipfs/kubo/core/coreiface" // 导入 coreiface 包，用于定义 IPFS 核心接口
    opt "github.com/ipfs/kubo/core/coreiface/options" // 导入 options 包，用于定义 IPFS 核心接口的选项
    mh "github.com/multiformats/go-multihash" // 导入 multihash 包，用于处理多哈希算法
)

var (
    pbCidV0  = "QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN" // 定义变量 pbCidV0，存储 CIDv0
    pbCid    = "bafybeiffndsajwhk3lwjewwdxqntmjm4b5wxaaanokonsggenkbw6slwk4" // 定义变量 pbCid，存储 CID
    rawCid   = "bafkreiffndsajwhk3lwjewwdxqntmjm4b5wxaaanokonsggenkbw6slwk4" // 定义变量 rawCid，存储原始字节的 CID
    cborCid  = "bafyreicnga62zhxnmnlt6ymq5hcbsg7gdhqdu6z4ehu3wpjhvqnflfy6nm" // 定义变量 cborCid，存储 CBOR 的 CID
    cborKCid = "bafyr2qgsohbwdlk7ajmmbb4lhoytmest4wdbe5xnexfvtxeatuyqqmwv3fgxp3pmhpc27gwey2cct56gloqefoqwcf3yqiqzsaqb7p4jefhcw" // 定义变量 cborKCid，存储 CBOR keccak-512 的 CID
)

// dag-pb
func pbBlock() io.Reader {
    return bytes.NewReader([]byte{10, 12, 8, 2, 18, 6, 104, 101, 108, 108, 111, 10, 24, 6}) // 返回一个包含特定字节的 Reader
}

// dag-cbor
func cborBlock() io.Reader {
    return bytes.NewReader([]byte{101, 72, 101, 108, 108, 111}) // 返回一个包含特定字节的 Reader
}

func (tp *TestSuite) TestBlock(t *testing.T) {
    tp.hasApi(t, func(api coreiface.CoreAPI) error { // 调用 TestSuite 结构体的 hasApi 方法
        if api.Block() == nil { // 如果 CoreAPI 的 Block 方法返回空
            return errAPINotImplemented // 返回错误信息
        }
        return nil // 返回空
    })

    t.Run("TestBlockPut (get raw CIDv1)", tp.TestBlockPut) // 运行测试函数 TestBlockPut
    t.Run("TestBlockPutCidCodec: dag-pb", tp.TestBlockPutCidCodecDagPb) // 运行测试函数 TestBlockPutCidCodecDagPb
    t.Run("TestBlockPutCidCodec: dag-cbor", tp.TestBlockPutCidCodecDagCbor) // 运行测试函数 TestBlockPutCidCodecDagCbor
    t.Run("TestBlockPutFormat (legacy): cbor → dag-cbor", tp.TestBlockPutFormatDagCbor) // 运行测试函数 TestBlockPutFormatDagCbor
    t.Run("TestBlockPutFormat (legacy): protobuf → dag-pb", tp.TestBlockPutFormatDagPb) // 运行测试函数 TestBlockPutFormatDagPb
    t.Run("TestBlockPutFormat (legacy): v0 → CIDv0", tp.TestBlockPutFormatV0) // 运行测试函数 TestBlockPutFormatV0
    t.Run("TestBlockPutHash", tp.TestBlockPutHash) // 运行测试函数 TestBlockPutHash
}
    # 运行测试函数 TestBlockGet，并将结果记录到测试对象 t 中
    t.Run("TestBlockGet", tp.TestBlockGet)
    # 运行测试函数 TestBlockRm，并将结果记录到测试对象 t 中
    t.Run("TestBlockRm", tp.TestBlockRm)
    # 运行测试函数 TestBlockStat，并将结果记录到测试对象 t 中
    t.Run("TestBlockStat", tp.TestBlockStat)
    # 运行测试函数 TestBlockPin，并将结果记录到测试对象 t 中
    t.Run("TestBlockPin", tp.TestBlockPin)
// 当没有传入选项时，生成的 CID 使用 'raw' 编解码器
func (tp *TestSuite) TestBlockPut(t *testing.T) {
    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 使用测试环境创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    // 如果出现错误，记录并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 调用 API 对象的 Block().Put 方法，传入上下文和示例块数据
    res, err := api.Block().Put(ctx, pbBlock())
    // 如果出现错误，记录并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 检查返回结果的根 CID 是否与 rawCid 相符，如果不符则记录错误
    if res.Path().RootCid().String() != rawCid {
        t.Errorf("got wrong cid: %s", res.Path().RootCid().String())
    }
}

// Format 已经被弃用，它使用了无效的编解码器名称。
// 确保 'cbor' 被修正为 'dag-cbor'
func (tp *TestSuite) TestBlockPutFormatDagCbor(t *testing.T) {
    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 使用测试环境创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    // 如果出现错误，记录并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 调用 API 对象的 Block().Put 方法，传入上下文、示例块数据和选项指定编解码器格式为 'cbor'
    res, err := api.Block().Put(ctx, cborBlock(), opt.Block.Format("cbor"))
    // 如果出现错误，记录并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 检查返回结果的根 CID 是否与 cborCid 相符，如果不符则记录错误
    if res.Path().RootCid().String() != cborCid {
        t.Errorf("got wrong cid: %s", res.Path().RootCid().String())
    }
}

// Format 已经被弃用，它使用了无效的编解码器名称。
// 确保 'protobuf' 被修正为 'dag-pb'
func (tp *TestSuite) TestBlockPutFormatDagPb(t *testing.T) {
    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 使用测试环境创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    // 如果出现错误，记录并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 调用 API 对象的 Block().Put 方法，传入上下文、示例块数据和选项指定编解码器格式为 'protobuf'
    res, err := api.Block().Put(ctx, pbBlock(), opt.Block.Format("protobuf"))
    // 如果出现错误，记录并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 检查返回结果的根 CID 是否与 pbCid 相符，如果不符则记录错误
    if res.Path().RootCid().String() != pbCid {
        t.Errorf("got wrong cid: %s", res.Path().RootCid().String())
    }
}

// Format 已经被弃用，它使用了无效的编解码器名称。
// 确保虚假编解码器 'v0' 被修正为 CIDv0（隐式使用 dag-pb 编解码器）
func (tp *TestSuite) TestBlockPutFormatV0(t *testing.T) {
    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 使用测试环境创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    // 如果出现错误，记录并终止测试
    if err != nil {
        t.Fatal(err)
    }
    // 调用 api.Block().Put 方法将 pbBlock() 的内容上传到服务器，并使用 opt.Block.Format("v0") 进行格式化，返回结果和错误
    res, err := api.Block().Put(ctx, pbBlock(), opt.Block.Format("v0"))
    // 如果出现错误，使用 t.Fatal 方法输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 检查返回结果的根 CID 是否与 pbCidV0 相同，如果不同则使用 t.Errorf 方法输出错误信息
    if res.Path().RootCid().String() != pbCidV0 {
        t.Errorf("got wrong cid: %s", res.Path().RootCid().String())
    }
func (tp *TestSuite) TestBlockPutCidCodecDagCbor(t *testing.T) {
    // 创建一个带有取消函数的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 使用测试套件的 makeAPI 方法创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    // 如果创建 API 对象时发生错误，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 调用 Block 方法的 Put 函数，将 CBOR 格式的数据放入区块，并指定 CID 编码为 "dag-cbor"
    res, err := api.Block().Put(ctx, cborBlock(), opt.Block.CidCodec("dag-cbor"))
    // 如果放入区块时发生错误，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 检查返回结果的根 CID 是否与预期的 CBOR CID 相符，如果不符则输出错误信息
    if res.Path().RootCid().String() != cborCid {
        t.Errorf("got wrong cid: %s", res.Path().RootCid().String())
    }
}

// 下面的测试函数与上面的函数类似，不再重复注释
func (tp *TestSuite) TestBlockPutCidCodecDagPb(t *testing.T) {
    // ...
}

func (tp *TestSuite) TestBlockPutHash(t *testing.T) {
    // ...
}

func (tp *TestSuite) TestBlockGet(t *testing.T) {
    // ...
    // ...
    // ...
    # 如果错误不为空，输出错误信息并终止测试
    if err != nil:
        t.Fatal(err)
    
    # 如果读取的数据不是"Hello"，输出错误信息
    if string(d) != "Hello":
        t.Error("didn't get correct data back")
    
    # 将路径转换为CID
    p := path.FromCid(res.Path().RootCid())
    
    # 解析路径并返回解析后的路径、解析后的CID和可能的错误
    rp, _, err := api.ResolvePath(ctx, p)
    # 如果有错误，输出错误信息并终止测试
    if err != nil:
        t.Fatal(err)
    # 如果解析后的CID不等于原始CID，输出错误信息
    if rp.RootCid().String() != res.Path().RootCid().String():
        t.Error("paths didn't match")
# 定义 TestBlockRm 函数，用于测试删除操作
func (tp *TestSuite) TestBlockRm(t *testing.T) {
    # 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    # 延迟调用取消函数
    defer cancel()
    # 调用 makeAPI 函数创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    # 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    # 调用 Block().Put 方法将数据写入块存储
    res, err := api.Block().Put(ctx, strings.NewReader(`Hello`), opt.Block.Format("raw"))
    # 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    # 调用 Block().Get 方法获取指定路径的数据
    r, err := api.Block().Get(ctx, res.Path())
    # 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    # 读取数据
    d, err := io.ReadAll(r)
    # 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    # 检查读取的数据是否与预期一致
    if string(d) != "Hello" {
        t.Error("didn't get correct data back")
    }

    # 调用 Block().Rm 方法删除指定路径的数据
    err = api.Block().Rm(ctx, res.Path())
    # 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    # 再次尝试获取已删除的数据，预期会出现错误
    _, err = api.Block().Get(ctx, res.Path())
    # 如果没有出现错误，输出错误信息并终止测试
    if err == nil {
        t.Fatal("expected err to exist")
    }
    # 如果出现的错误不是 NotFound 错误，输出错误信息
    if !ipld.IsNotFound(err) {
        t.Errorf("unexpected error; %s", err.Error())
    }

    # 再次尝试删除已删除的数据，预期会出现错误
    err = api.Block().Rm(ctx, res.Path())
    # 如果没有出现错误，输出错误信息并终止测试
    if err == nil {
        t.Fatal("expected err to exist")
    }
    # 如果出现的错误不是 NotFound 错误，输出错误信息
    if !ipld.IsNotFound(err) {
        t.Errorf("unexpected error; %s", err.Error())
    }

    # 强制删除已删除的数据
    err = api.Block().Rm(ctx, res.Path(), opt.Block.Force(true))
    # 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }
}

# 定义 TestBlockStat 函数，用于测试获取数据信息操作
func (tp *TestSuite) TestBlockStat(t *testing.T) {
    # 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    # 延迟调用取消函数
    defer cancel()
    # 调用 makeAPI 函数创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    # 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    # 调用 Block().Put 方法将数据写入块存储
    res, err := api.Block().Put(ctx, strings.NewReader(`Hello`), opt.Block.Format("raw"))
    # 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    # 调用 Block().Stat 方法获取指定路径的数据信息
    stat, err := api.Block().Stat(ctx, res.Path())
    # 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    # 检查获取的数据路径是否与预期一致
    if stat.Path().String() != res.Path().String() {
        t.Error("paths don't match")
    }

    # 检查获取的数据长度是否与预期一致
    if stat.Size() != len("Hello") {
        t.Error("length doesn't match")
    }
}

# 定义 TestBlockPin 函数，用于测试数据固化操作
func (tp *TestSuite) TestBlockPin(t *testing.T) {
    # 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    # 延迟调用取消函数
    defer cancel()
    // 调用 makeAPI 方法创建 API 对象，并返回结果和错误
    api, err := tp.makeAPI(t, ctx)
    // 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 调用 API 对象的 Block 方法，将字符串转换为 Reader，并使用指定的格式进行存储
    _, err = api.Block().Put(ctx, strings.NewReader(`Hello`), opt.Block.Format("raw"))
    // 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 调用 API 对象的 Pin 方法，列出所有的 pin，并检查数量是否为0
    if pins, err := api.Pin().Ls(ctx); err != nil || len(pins) != 0 {
        t.Fatal("expected 0 pins")
    }

    // 调用 API 对象的 Block 方法，将字符串转换为 Reader，并使用指定的格式进行存储，并将其 pin
    res, err := api.Block().Put(
        ctx,
        strings.NewReader(`Hello`),
        opt.Block.Pin(true),
        opt.Block.Format("raw"),
    )
    // 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 调用 accPins 方法，将 API 对象的 Pin 方法返回的结果转换为 pins 数组
    pins, err := accPins(api.Pin().Ls(ctx))
    // 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }
    // 检查 pins 数组的长度是否为1
    if len(pins) != 1 {
        t.Fatal("expected 1 pin")
    }
    // 检查 pins 数组中第一个元素的类型是否为 "recursive"
    if pins[0].Type() != "recursive" {
        t.Error("expected a recursive pin")
    }
    // 检查 pins 数组中第一个元素的路径是否与之前存储的路径相匹配
    if pins[0].Path().String() != res.Path().String() {
        t.Error("pin path didn't match")
    }
# 闭合前面的函数定义
```