# `kubo\core\commands\refs.go`

```
package commands

import (
    "context" // 上下文包，用于控制函数调用的行为
    "errors" // 错误处理包，用于处理错误信息
    "fmt" // 格式化包，用于格式化输出
    "io" // 输入输出包，用于处理输入输出流
    "strings" // 字符串包，用于处理字符串

    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv" // 导入自定义包
    "github.com/ipfs/kubo/core/commands/cmdutils" // 导入自定义包

    merkledag "github.com/ipfs/boxo/ipld/merkledag" // 导入第三方包
    cid "github.com/ipfs/go-cid" // 导入第三方包
    cidenc "github.com/ipfs/go-cidutil/cidenc" // 导入第三方包
    cmds "github.com/ipfs/go-ipfs-cmds" // 导入第三方包
    ipld "github.com/ipfs/go-ipld-format" // 导入第三方包
    iface "github.com/ipfs/kubo/core/coreiface" // 导入自定义包
)

var refsEncoderMap = cmds.EncoderMap{ // 定义命令行输出格式的编码器映射
    cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *RefWrapper) error { // 使用文本格式输出
        if out.Err != "" { // 如果有错误信息
            return fmt.Errorf(out.Err) // 返回错误信息
        }
        fmt.Fprintln(w, out.Ref) // 格式化输出引用

        return nil // 返回空值
    }),
}

// KeyList is a general type for outputting lists of keys
type KeyList struct { // 定义 KeyList 结构体
    Keys []cid.Cid // 包含 cid.Cid 类型的 Keys 列表
}

const ( // 定义常量
    refsFormatOptionName    = "format" // 引用格式选项名称
    refsEdgesOptionName     = "edges" // 边缘选项名称
    refsUniqueOptionName    = "unique" // 唯一选项名称
    refsRecursiveOptionName = "recursive" // 递归选项名称
    refsMaxDepthOptionName  = "max-depth" // 最大深度选项名称
)

// RefsCmd is the `ipfs refs` command
var RefsCmd = &cmds.Command{ // 定义 RefsCmd 命令
    Helptext: cmds.HelpText{ // 帮助文本
        Tagline: "List links (references) from an object.", // 简短描述
        ShortDescription: ` // 简短描述
Lists the hashes of all the links an IPFS or IPNS object(s) contains,
with the following format:

  <link base58 hash>

List all references recursively by using the flag '-r'.

NOTE: Like most other commands, Kubo will try to fetch the blocks of the passed path if they can't be found in the local store if it is running in online mode.
`,
    },
    Subcommands: map[string]*cmds.Command{ // 子命令映射
        "local": RefsLocalCmd, // 本地引用命令
    },
    Arguments: []cmds.Argument{ // 参数列表
        cmds.StringArg("ipfs-path", true, true, "Path to the object(s) to list refs from.").EnableStdin(), // IPFS 路径参数
    },
    # 定义一个 Options 列表，包含多个 cmds.Option 对象
    Options: []cmds.Option{
        # 添加一个字符串选项，用于指定边的格式，包含默认值和说明
        cmds.StringOption(refsFormatOptionName, "Emit edges with given format. Available tokens: <src> <dst> <linkname>.").WithDefault("<dst>"),
        # 添加一个布尔选项，用于指定是否以 `<from> -> <to>` 格式输出边
        cmds.BoolOption(refsEdgesOptionName, "e", "Emit edge format: `<from> -> <to>`."),
        # 添加一个布尔选项，用于指定是否在输出中省略重复的引用
        cmds.BoolOption(refsUniqueOptionName, "u", "Omit duplicate refs from output."),
        # 添加一个布尔选项，用于指定是否递归列出子节点的链接
        cmds.BoolOption(refsRecursiveOptionName, "r", "Recursively list links of child nodes."),
        # 添加一个整数选项，用于限制递归引用的深度，包含默认值和说明
        cmds.IntOption(refsMaxDepthOptionName, "Only for recursive refs, limits fetch and listing to the given depth").WithDefault(-1),
    },
    # 运行函数，处理请求并返回错误
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 解析请求的主体参数
        err := req.ParseBodyArgs()
        if err != nil {
            return err
        }

        # 获取请求的上下文
        ctx := req.Context
        # 获取 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        # 获取 CID 编码器
        enc, err := cmdenv.GetCidEncoder(req)
        if err != nil {
            return err
        }

        # 获取是否唯一、递归、最大深度、边缘、格式等选项
        unique, _ := req.Options[refsUniqueOptionName].(bool)
        recursive, _ := req.Options[refsRecursiveOptionName].(bool)
        maxDepth, _ := req.Options[refsMaxDepthOptionName].(int)
        edges, _ := req.Options[refsEdgesOptionName].(bool)
        format, _ := req.Options[refsFormatOptionName].(string)

        # 如果不是递归，则最大深度为1，只写入直接引用
        if !recursive {
            maxDepth = 1
        }

        # 如果包含边缘，则检查格式是否为"<dst>"，如果不是则返回错误
        if edges {
            if format != "<dst>" {
                return errors.New("using format argument with edges is not allowed")
            }
            # 将格式设置为"<src> -> <dst>"
            format = "<src> -> <dst>"
        }

        # TODO: 使用会话进行解析
        # 获取路径对应的对象
        objs, err := objectsForPaths(ctx, api, req.Arguments)
        if err != nil {
            return err
        }

        # 创建 RefWriter 对象
        rw := RefWriter{
            res:      res,
            DAG:      merkledag.NewSession(ctx, api.Dag()),
            Ctx:      ctx,
            Unique:   unique,
            PrintFmt: format,
            MaxDepth: maxDepth,
        }

        # 遍历对象列表，写入引用
        for _, o := range objs {
            if _, err := rw.WriteRefs(o, enc); err != nil {
                if err := res.Emit(&RefWrapper{Err: err.Error()}); err != nil {
                    return err
                }
            }
        }

        # 返回空错误
        return nil
    },
    # 引用编码器映射
    Encoders: refsEncoderMap,
    # 类型为 RefWrapper
    Type:     RefWrapper{},
// 定义名为 RefsLocalCmd 的变量，类型为 cmds.Command，用于列出所有本地引用
var RefsLocalCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "List all local references.", // 简短描述：列出所有本地引用
        ShortDescription: `
Displays the hashes of all local objects. NOTE: This treats all local objects as "raw blocks" and returns CIDv1-Raw CIDs.
`, // 显示所有本地对象的哈希值。注意：这将所有本地对象视为“原始块”并返回 CIDv1-Raw CIDs。
    },

    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        ctx := req.Context
        n, err := cmdenv.GetNode(env) // 获取节点
        if err != nil {
            return err
        }

        // todo: make async
        allKeys, err := n.Blockstore.AllKeysChan(ctx) // 获取所有密钥的通道
        if err != nil {
            return err
        }

        for k := range allKeys { // 遍历所有密钥
            err := res.Emit(&RefWrapper{Ref: k.String()}) // 发送引用到输出
            if err != nil {
                return err
            }
        }

        return nil
    },
    Encoders: refsEncoderMap, // 编码器映射
    Type:     RefWrapper{}, // 类型为 RefWrapper
}

// 根据给定的路径获取对象
func objectsForPaths(ctx context.Context, n iface.CoreAPI, paths []string) ([]cid.Cid, error) {
    roots := make([]cid.Cid, len(paths)) // 创建与路径长度相同的根CID切片
    for i, sp := range paths { // 遍历路径
        p, err := cmdutils.PathOrCidPath(sp) // 获取路径或CID路径
        if err != nil {
            return nil, err
        }
        o, _, err := n.ResolvePath(ctx, p) // 解析路径
        if err != nil {
            return nil, err
        }
        roots[i] = o.RootCid() // 将根CID存入切片
    }
    return roots, nil
}

// 定义名为 RefWrapper 的结构体，包含 Ref 和 Err 两个字段
type RefWrapper struct {
    Ref string
    Err string
}

// 定义名为 RefWriter 的结构体，包含 res、DAG、Ctx、Unique、MaxDepth 和 PrintFmt 字段
type RefWriter struct {
    res cmds.ResponseEmitter
    DAG ipld.NodeGetter
    Ctx context.Context

    Unique   bool
    MaxDepth int
    PrintFmt string

    seen map[string]int
}

// WriteRefs 方法用于将给定对象的引用写入底层写入器
func (rw *RefWriter) WriteRefs(c cid.Cid, enc cidenc.Encoder) (int, error) {
    n, err := rw.DAG.Get(rw.Ctx, c) // 获取节点
    if err != nil {
        return 0, err
    }
    return rw.writeRefsRecursive(n, 0, enc) // 递归写入引用
}

// writeRefsRecursive 方法用于递归写入引用
func (rw *RefWriter) writeRefsRecursive(n ipld.Node, depth int, enc cidenc.Encoder) (int, error) {
    nc := n.Cid() // 获取节点的CID

    var count int
    // 遍历 ipld.GetDAG 返回的节点列表，同时获取索引 i 和节点 ng
    for i, ng := range ipld.GetDAG(rw.Ctx, rw.DAG, n) {
        // 获取当前节点的链接 lc
        lc := n.Links()[i].Cid
        // 调用 rw.visit 方法，获取是否需要继续深入遍历子节点的标志 goDeeper 和是否需要写入的标志 shouldWrite
        goDeeper, shouldWrite := rw.visit(lc, depth+1) // The children are at depth+1

        // 如果不需要写入并且不需要继续深入遍历，则跳过当前循环，继续下一个链接
        if !shouldWrite && !goDeeper {
            continue
        }

        // 如果需要写入或者需要继续深入遍历，则获取节点 nd
        nd, err := ng.Get(rw.Ctx)
        if err != nil {
            return count, err
        }

        // 如果需要写入，则调用 rw.WriteEdge 方法写入节点，并增加计数
        if shouldWrite {
            if err := rw.WriteEdge(nc, lc, n.Links()[i].Name, enc); err != nil {
                return count, err
            }
            count++
        }

        // 如果需要继续深入遍历，则递归调用 rw.writeRefsRecursive 方法，并累加计数
        if goDeeper {
            c, err := rw.writeRefsRecursive(nd, depth+1, enc)
            count += c
            if err != nil {
                return count, err
            }
        }
    }

    // 遍历结束，返回计数和空错误
    return count, nil
// visit函数返回两个值：
// - 第一个布尔值表示我们是否应该继续遍历DAG
// - 第二个布尔值表示我们是否应该打印CID
//
// visit函数将根据rw.MaxDepth、先前访问的CID以及rw.Unique的设置进行分支修剪。即rw.Unique = false和rw.MaxDepth = -1将禁用任何修剪。但将rw.Unique设置为true将修剪已访问的分支，但以保留一组访问过的CID为代价。
func (rw *RefWriter) visit(c cid.Cid, depth int) (bool, bool) {
    atMaxDepth := rw.MaxDepth >= 0 && depth == rw.MaxDepth
    overMaxDepth := rw.MaxDepth >= 0 && depth > rw.MaxDepth

    // 当超过最大深度时，可以立即进行快捷方式。实际上，这仅适用于使用--maxDepth=0调用refs时，因为根的子节点已经超过了最大深度。否则，不应该触发此操作。
    if overMaxDepth {
        return false, false
    }

    // 如果我们不需要唯一的输出，可以立即进行快捷方式：
    //   - 当不在最大深度时，我们继续遍历
    //   - 总是打印
    if !rw.Unique {
        return !atMaxDepth, true
    }

    // 从这一点开始，Unique == true。
    // 因此，我们跟踪已见的Cid及其深度。
    if rw.seen == nil {
        rw.seen = make(map[string]int)
    }
    key := string(c.Bytes())
    oldDepth, ok := rw.seen[key]

    // 从这一点开始，Unique == true && depth < MaxDepth（或无限）。

    // 分支修剪情况：
    // - 我们之前看到了Cid，要么：
    //   - 深度是无限的（MaxDepth = -1）
    //   - 我们在DAG中看到了更高的（较小的深度）（这意味着我们必须先探索足够深）
    // 因为我们看到了CID，所以我们不会再次打印它。
    if ok && (rw.MaxDepth < 0 || oldDepth <= depth) {
        return false, false
    }

    // 最后一种情况，我们必须从这个CID继续探索DAG（除非我们达到深度限制）。
    // 我们记录下它的深度，因为它要么没有看到过
    // 如果当前深度小于上次记录的深度，则表示该键值对是新出现的，需要打印出来
    // 将当前深度记录到seen字典中
    rw.seen[key] = depth
    // 返回当前深度是否达到最大深度以及是否成功的标志位的取反
    return !atMaxDepth, !ok
// WriteEdge 函数用于写入一条边
func (rw *RefWriter) WriteEdge(from, to cid.Cid, linkname string, enc cidenc.Encoder) error {
    // 检查是否有上下文，如果有则检查上下文是否已完成
    if rw.Ctx != nil {
        select {
        case <-rw.Ctx.Done(): // 仅作为预防措施
            return rw.Ctx.Err()
        default:
        }
    }

    var s string
    // 根据不同情况选择不同的打印格式
    switch {
    case rw.PrintFmt != "":
        s = rw.PrintFmt
        s = strings.Replace(s, "<src>", enc.Encode(from), -1)
        s = strings.Replace(s, "<dst>", enc.Encode(to), -1)
        s = strings.Replace(s, "<linkname>", linkname, -1)
    default:
        s += enc.Encode(to)
    }

    // 将结果写入输出
    return rw.res.Emit(&RefWrapper{Ref: s})
}
```