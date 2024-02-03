# `kubo\core\commands\dag\export.go`

```go
package dagcmd

import (
    "context" // 上下文包，用于控制程序的执行流程
    "errors" // 错误处理包，用于处理错误信息
    "fmt" // 格式化包，用于格式化输出
    "io" // 输入输出包，用于处理输入输出流
    "os" // 操作系统包，用于操作系统相关功能
    "time" // 时间包，用于处理时间相关功能

    "github.com/cheggaaa/pb" // 进度条包，用于显示进度条
    blocks "github.com/ipfs/go-block-format" // IPFS区块格式包，用于处理IPFS区块格式
    cid "github.com/ipfs/go-cid" // IPFS CID包，用于处理IPFS CID
    ipld "github.com/ipfs/go-ipld-format" // IPFS IPLD包，用于处理IPFS IPLD格式
    "github.com/ipfs/kubo/core/commands/cmdenv" // IPFS Kubo命令包，用于处理IPFS Kubo命令
    "github.com/ipfs/kubo/core/commands/cmdutils" // IPFS Kubo命令工具包，用于处理IPFS Kubo命令工具
    iface "github.com/ipfs/kubo/core/coreiface" // IPFS Kubo核心接口包，用于处理IPFS Kubo核心接口

    cmds "github.com/ipfs/go-ipfs-cmds" // IPFS命令包，用于处理IPFS命令
    gocar "github.com/ipld/go-car" // IPLD CAR包，用于处理IPLD CAR格式
    selectorparse "github.com/ipld/go-ipld-prime/traversal/selector/parse" // IPLD选择器解析包，用于解析IPLD选择器
)

func dagExport(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
    // Accept CID or a content path
    p, err := cmdutils.PathOrCidPath(req.Arguments[0]) // 获取路径或CID
    if err != nil {
        return err
    }

    api, err := cmdenv.GetApi(env, req) // 获取API
    if err != nil {
        return err
    }

    // Resolve path and confirm the root block is available, fail fast if not
    b, err := api.Block().Stat(req.Context, p) // 获取路径的状态信息
    if err != nil {
        return err
    }
    c := b.Path().RootCid() // 获取根CID

    pipeR, pipeW := io.Pipe() // 创建管道读写

    errCh := make(chan error, 2) // 创建错误通道
    go func() {
        defer func() {
            if err := pipeW.Close(); err != nil {
                errCh <- fmt.Errorf("stream flush failed: %s", err) // 关闭管道写入
            }
            close(errCh) // 关闭错误通道
        }()

        store := dagStore{dag: api.Dag(), ctx: req.Context} // 创建DAG存储
        dag := gocar.Dag{Root: c, Selector: selectorparse.CommonSelector_ExploreAllRecursively} // 创建CAR DAG
        // TraverseLinksOnlyOnce is safe for an exhaustive selector but won't be when we allow
        // arbitrary selectors here
        car := gocar.NewSelectiveCar(req.Context, store, []gocar.Dag{dag}, gocar.TraverseLinksOnlyOnce()) // 创建选择性CAR
        if err := car.Write(pipeW); err != nil {
            errCh <- err // 写入CAR数据
        }
    }()

    if err := res.Emit(pipeR); err != nil {
        pipeR.Close() // 忽略错误
        return err
    }

    err = <-errCh // 获取错误信息
}
    // 检查错误是否为未找到错误
    if ipld.IsNotFound(err) {
        // 获取请求中的 explicitOffline 标志，如果不存在则默认为 false
        explicitOffline, _ := req.Options["offline"].(bool)
        // 如果 explicitOffline 为 true，则添加提示信息到错误信息中
        if explicitOffline {
            err = fmt.Errorf("%s (currently offline, perhaps retry without the offline flag)", err)
        } else {
            // 获取环境中的节点信息
            node, envErr := cmdenv.GetNode(env)
            // 如果获取节点信息成功且节点不在线，则添加提示信息到错误信息中
            if envErr == nil && !node.IsOnline {
                err = fmt.Errorf("%s (currently offline, perhaps retry after attaching to the network)", err)
            }
        }
    }

    // 返回错误信息
    return err
}
// 完成 CLI 导出操作
func finishCLIExport(res cmds.Response, re cmds.ResponseEmitter) error {
    var showProgress bool
    val, specified := res.Request().Options[progressOptionName]
    if !specified {
        // 根据 TTY 的可用性设置默认值
        errStat, _ := os.Stderr.Stat()
        if (errStat.Mode() & os.ModeCharDevice) != 0 {
            showProgress = true
        }
    } else if val.(bool) {
        showProgress = true
    }

    // 简单的传递，不显示进度
    if !showProgress {
        return cmds.Copy(re, res)
    }

    // 创建进度条
    bar := pb.New64(0).SetUnits(pb.U_BYTES)
    bar.Output = os.Stderr
    bar.ShowSpeed = true
    bar.ShowElapsedTime = true
    bar.RefreshRate = 500 * time.Millisecond
    bar.Start()

    var processedOneResponse bool
    for {
        v, err := res.Next()
        if err == io.EOF {

            // 只在成功时写入最终进度条更新
            // 出现错误时看起来太奇怪
            bar.Finish()

            return re.Close()
        } else if err != nil {
            return re.CloseWithError(err)
        } else if processedOneResponse {
            return re.CloseWithError(errors.New("unexpected multipart response during emit, please file a bugreport"))
        }

        r, ok := v.(io.Reader)
        if !ok {
            // 一些编码响应，这不应该发生
            return errors.New("unexpected non-stream passed to PostRun: please file a bugreport")
        }

        processedOneResponse = true

        if err := re.Emit(bar.NewProxyReader(r)); err != nil {
            return err
        }
    }
}

// FIXME(@Jorropo): https://github.com/ipld/go-car/issues/315
// 定义 dagStore 结构体
type dagStore struct {
    dag iface.APIDagService
    ctx context.Context
}

// 实现 Get 方法
func (ds dagStore) Get(_ context.Context, c cid.Cid) (blocks.Block, error) {
    return ds.dag.Get(ds.ctx, c)
}
```