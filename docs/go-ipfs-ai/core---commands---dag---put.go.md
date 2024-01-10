# `kubo\core\commands\dag\put.go`

```
package dagcmd

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "fmt"  // 导入 fmt 包，用于格式化输出

    blocks "github.com/ipfs/go-block-format"  // 导入 go-block-format 包，用于处理 IPFS 块格式
    "github.com/ipfs/go-cid"  // 导入 go-cid 包，用于处理 IPFS CID
    ipldlegacy "github.com/ipfs/go-ipld-legacy"  // 导入 go-ipld-legacy 包，用于处理 IPFS 旧版 IPLD
    "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入 cmdenv 包，用于处理 Kubo 核心命令
    "github.com/ipfs/kubo/core/commands/cmdutils"  // 导入 cmdutils 包，用于处理 Kubo 核心命令
    "github.com/ipld/go-ipld-prime/multicodec"  // 导入 go-ipld-prime/multicodec 包，用于处理 IPLD 多编解码器
    basicnode "github.com/ipld/go-ipld-prime/node/basic"  // 导入 go-ipld-prime/node/basic 包，用于处理 IPLD 基本节点

    "github.com/ipfs/boxo/files"  // 导入 files 包，用于处理 IPFS 文件
    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入 go-ipfs-cmds 包，用于处理 IPFS 命令
    ipld "github.com/ipfs/go-ipld-format"  // 导入 go-ipld-format 包，用于处理 IPFS IPLD
    mc "github.com/multiformats/go-multicodec"  // 导入 go-multicodec 包，用于处理多格式编解码器

    // 预期可用的格式/ienc 编解码器的最小集合
    _ "github.com/ipld/go-codec-dagpb"  // 导入 go-codec-dagpb 包
    _ "github.com/ipld/go-ipld-prime/codec/cbor"  // 导入 go-ipld-prime/codec/cbor 包
    _ "github.com/ipld/go-ipld-prime/codec/dagcbor"  // 导入 go-ipld-prime/codec/dagcbor 包
    _ "github.com/ipld/go-ipld-prime/codec/dagjson"  // 导入 go-ipld-prime/codec/dagjson 包
    _ "github.com/ipld/go-ipld-prime/codec/json"  // 导入 go-ipld-prime/codec/json 包
    _ "github.com/ipld/go-ipld-prime/codec/raw"  // 导入 go-ipld-prime/codec/raw 包
)

func dagPut(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
    api, err := cmdenv.GetApi(env, req)  // 获取环境中的 API
    if err != nil {
        return err
    }

    inputCodec, _ := req.Options["input-codec"].(string)  // 获取请求中的输入编解码器选项
    storeCodec, _ := req.Options["store-codec"].(string)  // 获取请求中的存储编解码器选项
    hash, _ := req.Options["hash"].(string)  // 获取请求中的哈希选项
    dopin, _ := req.Options["pin"].(bool)  // 获取请求中的 pin 选项

    var icodec mc.Code  // 定义输入编解码器
    if err := icodec.Set(inputCodec); err != nil {  // 设置输入编解码器
        return err
    }
    var scodec mc.Code  // 定义存储编解码器
    if err := scodec.Set(storeCodec); err != nil {  // 设置存储编解码器
        return err
    }
    var mhType mc.Code  // 定义哈希类型
    if err := mhType.Set(hash); err != nil {  // 设置哈希类型
        return err
    }

    cidPrefix := cid.Prefix{  // 定义 CID 前缀
        Version:  1,
        Codec:    uint64(scodec),
        MhType:   uint64(mhType),
        MhLength: -1,
    }

    decoder, err := multicodec.LookupDecoder(uint64(icodec))  // 查找输入编解码器的解码器
    if err != nil {
        return err
    }
    encoder, err := multicodec.LookupEncoder(uint64(scodec))  // 查找存储编解码器的编码器
    if err != nil {
        return err
    }

    var adder ipld.NodeAdder = api.Dag()  // 获取 API 中的 DAG 添加器
    # 如果需要进行 pin 操作
    if dopin {
        # 创建一个用于 pinning 的添加器
        adder = api.Dag().Pinning()
    }
    # 创建一个新的 IPLD 批处理器
    b := ipld.NewBatch(req.Context, adder)

    # 获取请求中的文件条目迭代器
    it := req.Files.Entries()
    # 遍历文件条目迭代器
    for it.Next() {
        # 从文件条目创建文件对象
        file := files.FileFromEntry(it)
        # 如果文件对象为空，则返回错误
        if file == nil {
            return fmt.Errorf("expected a regular file")
        }

        # 创建一个新的基础节点构建器
        node := basicnode.Prototype.Any.NewBuilder()
        # 使用解码器将文件数据解码到节点中
        if err := decoder(node, file); err != nil {
            return err
        }
        # 构建节点
        n := node.Build()

        # 创建一个新的字节缓冲区
        bd := bytes.NewBuffer([]byte{})
        # 使用编码器将节点编码到字节缓冲区中
        if err := encoder(n, bd); err != nil {
            return err
        }

        # 计算字节缓冲区的内容的 CID 前缀
        blockCid, err := cidPrefix.Sum(bd.Bytes())
        if err != nil {
            return err
        }
        # 使用字节缓冲区的内容和 CID 创建一个新的块
        blk, err := blocks.NewBlockWithCid(bd.Bytes(), blockCid)
        if err != nil {
            return err
        }
        # 创建一个 IPLD 老版本节点
        ln := ipldlegacy.LegacyNode{
            Block: blk,
            Node:  n,
        }

        # 检查块大小是否符合要求
        if err := cmdutils.CheckBlockSize(req, uint64(bd.Len())); err != nil {
            return err
        }

        # 将节点添加到 IPLD 批处理器中
        if err := b.Add(req.Context, &ln); err != nil {
            return err
        }

        # 获取节点的 CID，并将其发送到输出流中
        cid := ln.Cid()
        if err := res.Emit(&OutputObject{Cid: cid}); err != nil {
            return err
        }
    }
    # 如果文件条目迭代器出现错误，则返回该错误
    if it.Err() != nil {
        return it.Err()
    }

    # 提交 IPLD 批处理器中的所有更改
    if err := b.Commit(); err != nil {
        return err
    }

    # 操作成功完成，返回空错误
    return nil
# 闭合前面的函数定义
```