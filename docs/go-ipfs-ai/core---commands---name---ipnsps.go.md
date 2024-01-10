# `kubo\core\commands\name\ipnsps.go`

```
package name

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"  // 导入 io 包，用于输入输出操作
    "strings"  // 导入 strings 包，用于处理字符串

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入 cmds 包，用于处理 IPFS 命令
    "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入 cmdenv 包，用于处理 IPFS 命令环境
    ke "github.com/ipfs/kubo/core/commands/keyencode"  // 导入 keyencode 包，用于处理 IPFS 密钥编码
    record "github.com/libp2p/go-libp2p-record"  // 导入 record 包，用于处理 libp2p 记录
    "github.com/libp2p/go-libp2p/core/peer"  // 导入 peer 包，用于处理 libp2p 对等节点
)

type ipnsPubsubState struct {
    Enabled bool  // 定义结构体 ipnsPubsubState，包含 Enabled 字段
}

type ipnsPubsubCancel struct {
    Canceled bool  // 定义结构体 ipnsPubsubCancel，包含 Canceled 字段
}

type stringList struct {
    Strings []string  // 定义结构体 stringList，包含 Strings 字段
}

// IpnsPubsubCmd is the subcommand that allows us to manage the IPNS pubsub system
var IpnsPubsubCmd = &cmds.Command{  // 定义 IpnsPubsubCmd 变量，类型为 cmds.Command
    Status: cmds.Experimental,  // 设置命令状态为实验性
    Helptext: cmds.HelpText{  // 设置命令的帮助文本
        Tagline: "IPNS pubsub management",  // 设置命令的简短描述
        ShortDescription: `  // 设置命令的详细描述
Manage and inspect the state of the IPNS pubsub resolver.

Note: this command is experimental and subject to change as the system is refined
`,
    },
    Subcommands: map[string]*cmds.Command{  // 设置命令的子命令
        "state":  ipnspsStateCmd,  // 子命令 "state" 对应 ipnspsStateCmd
        "subs":   ipnspsSubsCmd,  // 子命令 "subs" 对应 ipnspsSubsCmd
        "cancel": ipnspsCancelCmd,  // 子命令 "cancel" 对应 ipnspsCancelCmd
    },
}

var ipnspsStateCmd = &cmds.Command{  // 定义 ipnspsStateCmd 变量，类型为 cmds.Command
    Status: cmds.Experimental,  // 设置命令状态为实验性
    Helptext: cmds.HelpText{  // 设置命令的帮助文本
        Tagline: "Query the state of IPNS pubsub.",  // 设置命令的简短描述
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {  // 设置命令的运行函数
        n, err := cmdenv.GetNode(env)  // 获取节点信息
        if err != nil {  // 如果出现错误
            return err  // 返回错误信息
        }

        return cmds.EmitOnce(res, &ipnsPubsubState{n.PSRouter != nil})  // 发送一次命令响应
    },
    Type: ipnsPubsubState{},  // 设置命令的类型为 ipnsPubsubState
    Encoders: cmds.EncoderMap{  // 设置命令的编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, ips *ipnsPubsubState) error {  // 设置文本编码器
            var state string  // 定义状态字符串
            if ips.Enabled {  // 如果 IPNS pubsub 已启用
                state = "enabled"  // 设置状态为已启用
            } else {  // 否则
                state = "disabled"  // 设置状态为已禁用
            }

            _, err := fmt.Fprintln(w, state)  // 格式化输出状态字符串
            return err  // 返回错误信息
        }),
    },
}

var ipnspsSubsCmd = &cmds.Command{  // 定义 ipnspsSubsCmd 变量，类型为 cmds.Command
    Status: cmds.Experimental,  // 设置命令状态为实验性
    Helptext: cmds.HelpText{  // 设置命令的帮助文本
        Tagline: "Show current name subscriptions.",  // 设置命令的简短描述
    },
    Options: []cmds.Option{  # 定义一个空的选项数组
        ke.OptionIPNSBase,  # 将ke.OptionIPNSBase添加到选项数组中
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {  # 定义一个运行函数，接受请求、响应和环境参数，并返回错误
        keyEnc, err := ke.KeyEncoderFromString(req.Options[ke.OptionIPNSBase.Name()].(string))  # 从请求的选项中获取IPNS基础选项的值，并将其转换为KeyEncoder
        if err != nil {  # 如果出现错误
            return err  # 返回错误
        }

        n, err := cmdenv.GetNode(env)  # 从环境中获取节点
        if err != nil {  # 如果出现错误
            return err  # 返回错误
        }

        if n.PSRouter == nil {  # 如果节点的PSRouter为空
            return cmds.Errorf(cmds.ErrClient, "IPNS pubsub subsystem is not enabled")  # 返回IPNS pubsub子系统未启用的错误
        }
        var paths []string  # 定义一个空的字符串数组
        for _, key := range n.PSRouter.GetSubscriptions() {  # 遍历节点的订阅列表
            ns, k, err := record.SplitKey(key)  # 将订阅键拆分为命名空间和键
            if err != nil || ns != "ipns" {  # 如果出现错误或者命名空间不是"ipns"
                // Not necessarily an error.  # 不一定是错误
                continue  # 继续下一次循环
            }
            pid, err := peer.IDFromBytes([]byte(k))  # 将键转换为对等ID
            if err != nil {  # 如果出现错误
                log.Errorf("ipns key not a valid peer ID: %s", err)  # 记录错误日志
                continue  # 继续下一次循环
            }
            paths = append(paths, "/ipns/"+keyEnc.FormatID(pid))  # 将格式化后的对等ID添加到路径数组中
        }

        return cmds.EmitOnce(res, &stringList{paths})  # 通过响应发射一次路径数组
    },
    Type: stringList{},  # 定义类型为stringList的类型
    Encoders: cmds.EncoderMap{  # 定义编码器映射
        cmds.Text: stringListEncoder(),  # 将文本编码器与stringList编码器关联
    },
# 定义一个名为ipnspsCancelCmd的命令对象
var ipnspsCancelCmd = &cmds.Command{
    # 设置命令对象的状态为实验性
    Status: cmds.Experimental,
    # 设置命令对象的帮助文本
    Helptext: cmds.HelpText{
        Tagline: "Cancel a name subscription.",
    },
    # 定义命令对象的运行函数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取节点信息
        n, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }
        # 如果节点的PSRouter为空，则返回错误
        if n.PSRouter == nil {
            return cmds.Errorf(cmds.ErrClient, "IPNS pubsub subsystem is not enabled")
        }
        # 获取请求中的参数作为名称
        name := req.Arguments[0]
        # 去除名称中的/ipns/前缀
        name = strings.TrimPrefix(name, "/ipns/")
        # 解码名称为对等点ID
        pid, err := peer.Decode(name)
        if err != nil {
            return cmds.Errorf(cmds.ErrClient, err.Error())
        }
        # 调用节点的PSRouter对象的Cancel方法取消订阅
        ok, err := n.PSRouter.Cancel("/ipns/" + string(pid))
        if err != nil {
            return err
        }
        # 发送一次性的响应
        return cmds.EmitOnce(res, &ipnsPubsubCancel{ok})
    },
    # 定义命令对象的参数
    Arguments: []cmds.Argument{
        cmds.StringArg("name", true, false, "Name to cancel the subscription for."),
    },
    # 定义命令对象的类型
    Type: ipnsPubsubCancel{},
    # 定义命令对象的编码器映射
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, ipc *ipnsPubsubCancel) error {
            # 定义状态字符串
            var state string
            # 根据取消状态设置状态字符串
            if ipc.Canceled {
                state = "canceled"
            } else {
                state = "no subscription"
            }
            # 将状态字符串写入到输出流中
            _, err := fmt.Fprintln(w, state)
            return err
        }),
    },
}

# 定义一个返回字符串列表的编码器函数
func stringListEncoder() cmds.EncoderFunc {
    return cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, list *stringList) error {
        # 遍历字符串列表并将每个字符串写入到输出流中
        for _, s := range list.Strings {
            _, err := fmt.Fprintln(w, s)
            if err != nil {
                return err
            }
        }
        return nil
    })
}
```