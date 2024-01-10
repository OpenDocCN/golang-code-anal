# `kubo\core\commands\name\ipns.go`

```
package name

import (
    "errors"  // 导入 errors 包，用于处理错误
    "fmt"  // 导入 fmt 包，用于格式化输入输出
    "io"  // 导入 io 包，用于进行输入输出操作
    "strings"  // 导入 strings 包，用于处理字符串
    "time"  // 导入 time 包，用于处理时间

    "github.com/ipfs/boxo/namesys"  // 导入 namesys 包
    "github.com/ipfs/boxo/path"  // 导入 path 包
    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入 go-ipfs-cmds 包
    logging "github.com/ipfs/go-log"  // 导入 go-log 包
    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入 cmdenv 包
    options "github.com/ipfs/kubo/core/coreiface/options"  // 导入 options 包
)

var log = logging.Logger("core/commands/ipns")  // 定义名为 log 的日志记录器

type ResolvedPath struct {
    Path string  // 定义 ResolvedPath 结构体，包含 Path 字段
}

const (
    recursiveOptionName      = "recursive"  // 定义常量 recursiveOptionName
    nocacheOptionName        = "nocache"  // 定义常量 nocacheOptionName
    dhtRecordCountOptionName = "dht-record-count"  // 定义常量 dhtRecordCountOptionName
    dhtTimeoutOptionName     = "dht-timeout"  // 定义常量 dhtTimeoutOptionName
    streamOptionName         = "stream"  // 定义常量 streamOptionName
)

var IpnsCmd = &cmds.Command{  // 定义名为 IpnsCmd 的命令
    Helptext: cmds.HelpText{  // 设置命令的帮助文本
        Tagline: "Resolve IPNS names.",  // 设置命令的标语
        ShortDescription: `  // 设置命令的简短描述
IPNS is a PKI namespace, where names are the hashes of public keys, and
the private key enables publishing new (signed) values. In both publish
and resolve, the default name used is the node's own PeerID,
which is the hash of its public key.
`,
        LongDescription: `  // 设置命令的详细描述
IPNS is a PKI namespace, where names are the hashes of public keys, and
the private key enables publishing new (signed) values. In both publish
and resolve, the default name used is the node's own PeerID,
which is the hash of its public key.

You can use the 'ipfs key' commands to list and generate more names and their
respective keys.

Examples:

Resolve the value of your name:

  > ipfs name resolve
  /ipfs/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy

Resolve the value of another name:

  > ipfs name resolve QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ
  /ipfs/QmSiTko9JZyabH56y2fussEt1A5oDqsFXB3CkvAqraFryz

Resolve the value of a dnslink:

  > ipfs name resolve ipfs.io
  /ipfs/QmaBvfZooxWkrv7D3r8LS9moNjzD2o525XMZze69hhoxf5

`,
    },

    Arguments: []cmds.Argument{  // 设置命令的参数
        cmds.StringArg("name", false, false, "The IPNS name to resolve. Defaults to your node's peerID."),  // 设置字符串类型的参数 name
    },
    # 定义一个选项数组，包含不同类型的命令选项
    Options: []cmds.Option{
        # 定义一个布尔类型的选项，用于递归解析直到结果不再是 IPNS 名称
        cmds.BoolOption(recursiveOptionName, "r", "Resolve until the result is not an IPNS name.").WithDefault(true),
        # 定义一个布尔类型的选项，用于禁用缓存条目
        cmds.BoolOption(nocacheOptionName, "n", "Do not use cached entries."),
        # 定义一个无符号整数类型的选项，用于设置 DHT 解析的记录数
        cmds.UintOption(dhtRecordCountOptionName, "dhtrc", "Number of records to request for DHT resolution.").WithDefault(uint(namesys.DefaultResolverDhtRecordCount)),
        # 定义一个字符串类型的选项，用于设置 DHT 解析的超时时间
        cmds.StringOption(dhtTimeoutOptionName, "dhtt", "Max time to collect values during DHT resolution e.g. \"30s\". Pass 0 for no timeout.").WithDefault(namesys.DefaultResolverDhtTimeout.String()),
        # 定义一个布尔类型的选项，用于在找到条目时流式传输条目
        cmds.BoolOption(streamOptionName, "s", "Stream entries as they are found."),
    },
    # 定义一个编码器映射，将不同类型的编码器与请求、写入器和已解析路径相关联
    Encoders: cmds.EncoderMap{
        # 将文本编码器与请求、写入器和已解析路径相关联
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, rp *ResolvedPath) error {
            # 将已解析路径的路径写入到写入器中
            _, err := fmt.Fprintln(w, rp.Path)
            return err
        }),
    },
    # 定义一个已解析路径类型的对象
    Type: ResolvedPath{},
# 闭合前面的函数定义
```