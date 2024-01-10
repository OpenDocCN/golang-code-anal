# `kubo\core\commands\stat_provide.go`

```
package commands

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"  // 导入 io 包，用于输入输出操作
    "text/tabwriter"  // 导入 text/tabwriter 包，用于格式化输出表格
    "time"  // 导入 time 包，用于时间相关操作

    humanize "github.com/dustin/go-humanize"  // 导入第三方库，用于人性化显示数据大小
    "github.com/ipfs/boxo/provider"  // 导入 provider 包
    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入 cmds 包
    "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入 cmdenv 包
    "golang.org/x/exp/constraints"  // 导入 constraints 包
)

var statProvideCmd = &cmds.Command{  // 定义命令
    Helptext: cmds.HelpText{  // 命令的帮助文本
        Tagline: "Returns statistics about the node's (re)provider system.",  // 简短的描述
        ShortDescription: `  // 详细描述
Returns statistics about the content the node is advertising.

This interface is not stable and may change from release to release.
`,
    },
    Arguments: []cmds.Argument{},  // 命令的参数
    Options:   []cmds.Option{},  // 命令的选项
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {  // 命令的执行函数
        nd, err := cmdenv.GetNode(env)  // 获取节点
        if err != nil {
            return err
        }

        if !nd.IsOnline {  // 如果节点不在线
            return ErrNotOnline  // 返回错误信息
        }

        stats, err := nd.Provider.Stat()  // 获取节点的统计信息
        if err != nil {
            return err
        }

        if err := res.Emit(stats); err != nil {  // 输出统计信息
            return err
        }

        return nil
    },
    Encoders: cmds.EncoderMap{  // 命令的编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, s *provider.ReproviderStats) error {  // 文本编码器
            wtr := tabwriter.NewWriter(w, 1, 2, 1, ' ', 0)  // 创建 tabwriter
            defer wtr.Flush()  // 延迟关闭 tabwriter

            fmt.Fprintf(wtr, "TotalProvides:\t%s\n", humanNumber(s.TotalProvides))  // 格式化输出总提供数
            fmt.Fprintf(wtr, "AvgProvideDuration:\t%s\n", humanDuration(s.AvgProvideDuration))  // 格式化输出平均提供持续时间
            fmt.Fprintf(wtr, "LastReprovideDuration:\t%s\n", humanDuration(s.LastReprovideDuration))  // 格式化输出最后一次重新提供持续时间
            fmt.Fprintf(wtr, "LastReprovideBatchSize:\t%s\n", humanNumber(s.LastReprovideBatchSize))  // 格式化输出最后一次重新提供批量大小
            return nil
        }),
    },
    Type: provider.ReproviderStats{},  // 命令的类型
}

func humanDuration(val time.Duration) string {  // 定义函数，将时间持续时间转换为人类可读的格式
    return val.Truncate(time.Microsecond).String()  // 返回格式化后的时间字符串
}

func humanNumber[T constraints.Float | constraints.Integer](n T) string {  // 定义函数，将数字转换为人类可读的格式
    # 将整数 n 转换为浮点数
    nf := float64(n)
    # 使用 humanSI 函数将浮点数 nf 转换为带有单位的字符串，保留 0 位小数
    str := humanSI(nf, 0)
    # 使用 humanFull 函数将浮点数 nf 转换为完整的带有单位的字符串，保留 0 位小数
    fullStr := humanFull(nf, 0)
    # 如果简化后的字符串和完整字符串不相等
    if str != fullStr {
        # 返回简化字符串和完整字符串的格式化字符串
        return fmt.Sprintf("%s\t(%s)", str, fullStr)
    }
    # 否则返回简化字符串
    return str
# 以人类可读的方式格式化浮点数值和单位，返回格式化后的字符串
func humanSI(val float64, decimals int) string:
    # 调用 humanize 包的 ComputeSI 函数，获取格式化后的值和单位
    v, unit := humanize.ComputeSI(val)
    # 调用 humanFull 函数，将格式化后的值和单位拼接成字符串并返回
    return fmt.Sprintf("%s%s", humanFull(v, decimals), unit)

# 以人类可读的方式格式化浮点数值，返回格式化后的字符串
func humanFull(val float64, decimals int) string:
    # 调用 humanize 包的 CommafWithDigits 函数，将浮点数值格式化为字符串
    return humanize.CommafWithDigits(val, decimals)
```