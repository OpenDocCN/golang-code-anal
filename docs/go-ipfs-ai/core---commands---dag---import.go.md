# `kubo\core\commands\dag\import.go`

```
package dagcmd

import (
    "errors" // 导入错误处理包
    "fmt" // 导入格式化包
    "io" // 导入输入输出包

    "github.com/ipfs/boxo/files" // 导入文件操作包
    blocks "github.com/ipfs/go-block-format" // 导入区块格式包
    cid "github.com/ipfs/go-cid" // 导入CID包
    cmds "github.com/ipfs/go-ipfs-cmds" // 导入IPFS命令包
    ipld "github.com/ipfs/go-ipld-format" // 导入IPLD格式包
    ipldlegacy "github.com/ipfs/go-ipld-legacy" // 导入旧版IPLD包
    "github.com/ipfs/kubo/core/coreiface/options" // 导入选项包
    gocarv2 "github.com/ipld/go-car/v2" // 导入CAR包

    "github.com/ipfs/kubo/core/commands/cmdenv" // 导入命令环境包
    "github.com/ipfs/kubo/core/commands/cmdutils" // 导入命令工具包
)

func dagImport(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
    node, err := cmdenv.GetNode(env) // 获取节点
    if err != nil {
        return err
    }

    api, err := cmdenv.GetApi(env, req) // 获取API
    if err != nil {
        return err
    }

    blockDecoder := ipldlegacy.NewDecoder() // 创建旧版IPLD解码器

    // 在导入时确保不会出现任何网络请求
    // 如果基于导入内容和区块存储中的内容无法进行固定: 那就没办法了
    api, err = api.WithOptions(options.Api.Offline(true)) // 设置API选项为离线模式
    if err != nil {
        return err
    }

    doPinRoots, _ := req.Options[pinRootsOptionName].(bool) // 获取是否固定根节点选项的值

    // 获取固定锁（也兼作GC锁），以便无论流入的CAR的大小如何，都不会在我们有机会固定可能在最后出现的根节点之前消失
    // 这对于像dagger这样的用例尤为重要:
    //    ipfs dag import $( ... | ipfs-dagger --stdout=carfifos )
    //
    if doPinRoots {
        unlocker := node.Blockstore.PinLock(req.Context) // 获取固定锁
        defer unlocker.Unlock(req.Context) // 在函数返回前解锁
    }

    // 这*不是*一个事务
    // 它只是一种减轻区块存储压力的方法
    // 类似于pinner.Pin/pinner.Flush
    batch := ipld.NewBatch(req.Context, api.Dag()) // 创建新的批处理

    roots := cid.NewSet() // 创建CID集合
    var blockCount, blockBytesCount uint64 // 定义区块数量和区块字节数

    // 记住最后一个有效的区块，并提供有意义的错误消息
    // 当导入一个被截断/损坏的 CAR 文件时发生错误
    importError := func(previous blocks.Block, current blocks.Block, err error) error {
        // 如果当前块不为空，则返回导入失败的块和错误信息
        if current != nil {
            return fmt.Errorf("import failed at block %q: %w", current.Cid(), err)
        }
        // 如果前一个块不为空，则返回导入失败的块和错误信息
        if previous != nil {
            return fmt.Errorf("import failed after block %q: %w", previous.Cid(), err)
        }
        // 如果前一个块和当前块都为空，则返回导入失败的错误信息
        return fmt.Errorf("import failed: %w", err)
    }

    // 获取请求中文件的条目
    it := req.Files.Entries()
    // 遍历迭代器，获取下一个文件
    for it.Next() {
        // 从迭代器中获取文件句柄
        file := files.FileFromEntry(it)
        // 如果文件句柄为空，返回错误
        if file == nil {
            return errors.New("expected a file handle")
        }

        // 导入数据块
        err = func() error {
            // 包装一个延迟关闭作用域
            //
            // 在开始之前，it() 中的每个文件都已经打开
            // 这里尽早关闭文件，以保持整洁，并且在关闭的 FIFO 上表面潜在的写入错误
            // 这不能帮助解决句柄耗尽的问题
            defer file.Close()

            var previous blocks.Block

            // 创建一个新的块读取器
            car, err := gocarv2.NewBlockReader(file)
            if err != nil {
                return err
            }

            // 将 car 中的根添加到 roots 中
            for _, c := range car.Roots {
                roots.Add(c)
            }

            for {
                // 读取下一个块
                block, err := car.Next()
                // 如果出现错误并且不是 EOF，则返回导入错误
                if err != nil && err != io.EOF {
                    return importError(previous, block, err)
                } else if block == nil {
                    break
                }
                // 检查块大小是否符合要求
                if err := cmdutils.CheckBlockSize(req, uint64(len(block.RawData()))); err != nil {
                    return importError(previous, block, err)
                }

                // 双重解码并不是最佳选择，但我们需要它来进行批处理
                nd, err := blockDecoder.DecodeNode(req.Context, block)
                if err != nil {
                    return importError(previous, block, err)
                }

                // 将节点添加到批处理中
                if err := batch.Add(req.Context, nd); err != nil {
                    return importError(previous, block, err)
                }
                // 块计数加一
                blockCount++
                // 块字节数计数增加
                blockBytesCount += uint64(len(block.RawData()))
                previous = block
            }
            return nil
        }()
        // 如果有错误，返回错误
        if err != nil {
            return err
        }
    }

    // 提交批处理
    if err := batch.Commit(); err != nil {
        return err
    }
    // 如果需要对根节点进行固定（pinning），则执行以下操作
    if doPinRoots {
        // 遍历所有根节点
        err = roots.ForEach(func(c cid.Cid) error {
            ret := RootMeta{Cid: c}

            // 通过节点的 CID 从块存储中获取块
            if block, err := node.Blockstore.Get(req.Context, c); err != nil {
                ret.PinErrorMsg = err.Error()
            } else if nd, err := blockDecoder.DecodeNode(req.Context, block); err != nil {
                ret.PinErrorMsg = err.Error()
            } else if err := node.Pinning.Pin(req.Context, nd, true, ""); err != nil {
                ret.PinErrorMsg = err.Error()
            } else if err := node.Pinning.Flush(req.Context); err != nil {
                ret.PinErrorMsg = err.Error()
            }

            // 发送 CarImportOutput 结构体到结果通道
            return res.Emit(&CarImportOutput{Root: &ret})
        })
        if err != nil {
            return err
        }
    }

    // 获取是否需要统计信息的选项
    stats, _ := req.Options[statsOptionName].(bool)
    if stats {
        // 发送 CarImportOutput 结构体到结果通道，包含统计信息
        err = res.Emit(&CarImportOutput{
            Stats: &CarImportStats{
                BlockCount:      blockCount,
                BlockBytesCount: blockBytesCount,
            },
        })
        if err != nil {
            return err
        }
    }

    // 返回空值
    return nil
# 闭合前面的函数定义
```