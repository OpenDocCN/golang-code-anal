# `kubo\core\commands\dag\stat.go`

```go
package dagcmd

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "io" // 导入 io 包，用于实现 I/O 操作
    "os" // 导入 os 包，提供操作系统功能

    mdag "github.com/ipfs/boxo/ipld/merkledag" // 导入 merkledag 包，用于处理 IPLD Merkle DAG 数据结构
    "github.com/ipfs/boxo/ipld/merkledag/traverse" // 导入 traverse 包，用于遍历 Merkle DAG
    cid "github.com/ipfs/go-cid" // 导入 cid 包，用于处理 Content Identifier (CID)
    cmds "github.com/ipfs/go-ipfs-cmds" // 导入 cmds 包，用于创建命令行接口
    "github.com/ipfs/kubo/core/commands/cmdenv" // 导入 cmdenv 包，用于管理命令行环境
    "github.com/ipfs/kubo/core/commands/cmdutils" // 导入 cmdutils 包，用于创建命令行工具
    "github.com/ipfs/kubo/core/commands/e" // 导入 e 包，用于处理错误
)

// TODO cache every cid traversal in a dp cache
// if the cid exists in the cache, don't traverse it, and use the cached result
// to compute the new state

// 定义 dagStat 函数，接收请求、响应和环境参数，返回错误
func dagStat(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
    progressive := req.Options[progressOptionName].(bool) // 从请求参数中获取 progressOptionName 对应的布尔值
    api, err := cmdenv.GetApi(env, req) // 从环境中获取 API 对象
    if err != nil {
        return err // 如果获取 API 对象出错，返回错误
    }
    nodeGetter := mdag.NewSession(req.Context, api.Dag()) // 创建一个新的 Merkle DAG 会话

    cidSet := cid.NewSet() // 创建一个新的 CID 集合
    dagStatSummary := &DagStatSummary{DagStatsArray: []*DagStat{}} // 创建一个新的 DagStatSummary 结构体实例，初始化其中的 DagStatsArray 字段为空数组
    # 遍历请求参数中的每个参数
    for _, a := range req.Arguments:
        # 将参数转换为路径或 CID 路径
        p, err := cmdutils.PathOrCidPath(a)
        if err != nil:
            return err
        # 解析路径
        rp, remainder, err := api.ResolvePath(req.Context, p)
        if err != nil:
            return err
        # 如果剩余部分长度大于 0，则返回错误
        if len(remainder) > 0:
            return fmt.Errorf("cannot return size for anything other than a DAG with a root CID")

        # 获取节点对象
        obj, err := nodeGetter.Get(req.Context, rp.RootCid())
        if err != nil:
            return err
        # 创建 DAG 统计对象
        dagstats := &DagStat{Cid: rp.RootCid()}
        # 将 DAG 统计对象添加到汇总中
        dagStatSummary.appendStats(dagstats)
        # 遍历节点对象
        err = traverse.Traverse(obj, traverse.Options{
            DAG:   nodeGetter,
            Order: traverse.DFSPre,
            Func: func(current traverse.State) error {
                # 计算当前节点的大小
                currentNodeSize := uint64(len(current.Node.RawData()))
                dagstats.Size += currentNodeSize
                dagstats.NumBlocks++
                # 如果当前节点的 CID 不在集合中，则增加总大小
                if !cidSet.Has(current.Node.Cid()):
                    dagStatSummary.incrementTotalSize(currentNodeSize)
                # 增加冗余大小
                dagStatSummary.incrementRedundantSize(currentNodeSize)
                # 将当前节点的 CID 添加到集合中
                cidSet.Add(current.Node.Cid())
                # 如果是渐进式，则发送 DAG 统计汇总
                if progressive:
                    if err := res.Emit(dagStatSummary); err != nil:
                        return err
                return nil
            },
            ErrFunc:        nil,
            SkipDuplicates: true,
        })
        # 如果遍历出错，则返回错误
        if err != nil:
            return fmt.Errorf("error traversing DAG: %w", err)
    }

    # 设置 DAG 统计汇总的唯一块数
    dagStatSummary.UniqueBlocks = cidSet.Len()
    # 计算汇总
    dagStatSummary.calculateSummary()

    # 发送 DAG 统计汇总
    if err := res.Emit(dagStatSummary); err != nil:
        return err
    return nil
func finishCLIStat(res cmds.Response, re cmds.ResponseEmitter) error {
    // 定义一个指向DagStatSummary结构体的指针变量
    var dagStats *DagStatSummary
    // 循环遍历res中的数据
    for {
        // 从res中获取下一个值和可能的错误
        v, err := res.Next()
        // 如果有错误
        if err != nil {
            // 如果错误是io.EOF，表示已经读取完毕，退出循环
            if err == io.EOF {
                break
            }
            // 如果不是io.EOF，返回错误
            return err
        }
        // 根据v的类型进行不同的处理
        switch out := v.(type) {
        // 如果v的类型是*DagStatSummary
        case *DagStatSummary:
            // 将v转换为*DagStatSummary类型，并赋值给dagStats
            dagStats = out
            // 如果dagStats的Ratio为0
            if dagStats.Ratio == 0 {
                // 获取dagStats.DagStatsArray的长度
                length := len(dagStats.DagStatsArray)
                // 如果长度大于0
                if length > 0 {
                    // 获取当前统计信息
                    currentStat := dagStats.DagStatsArray[length-1]
                    // 输出CID、Size和NumBlocks到标准错误输出
                    fmt.Fprintf(os.Stderr, "CID: %s, Size: %d, NumBlocks: %d\n", currentStat.Cid, currentStat.Size, currentStat.NumBlocks)
                }
            }
        // 如果v的类型不是*DagStatSummary
        default:
            // 返回类型错误
            return e.TypeErr(out, v)
        }
    }
    // 将dagStats发送到re中
    return re.Emit(dagStats)
}
```