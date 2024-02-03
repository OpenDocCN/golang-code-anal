# `kubo\core\commands\name\name.go`

```go
package name

import (
    "bytes"  // 导入 bytes 包，用于操作二进制数据
    "encoding/hex"  // 导入 hex 包，用于十六进制编码和解码
    "fmt"  // 导入 fmt 包，用于格式化 I/O
    "io"  // 导入 io 包，提供了基本的 I/O 接口
    "text/tabwriter"  // 导入 tabwriter 包，用于格式化表格输出
    "time"  // 导入 time 包，提供了时间的显示和测量用的函数

    "github.com/ipfs/boxo/ipns"  // 导入 ipns 包
    ipns_pb "github.com/ipfs/boxo/ipns/pb"  // 导入 ipns_pb 包
    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入 cmds 包
    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入 cmdenv 包
    "google.golang.org/protobuf/proto"  // 导入 proto 包
)

type IpnsEntry struct {
    Name  string  // 定义 IpnsEntry 结构体，包含 Name 字段
    Value string  // 定义 IpnsEntry 结构体，包含 Value 字段
}

var NameCmd = &cmds.Command{  // 定义 NameCmd 变量，类型为 *cmds.Command
    Helptext: cmds.HelpText{  // 设置 NameCmd 的 Helptext 字段
        Tagline: "Publish and resolve IPNS names.",  // 设置 Tagline 字段
        ShortDescription: `  // 设置 ShortDescription 字段
IPNS is a PKI namespace, where names are the hashes of public keys, and
the private key enables publishing new (signed) values. In both publish
and resolve, the default name used is the node's own PeerID,
which is the hash of its public key.
`,  // 设置 ShortDescription 字段
        LongDescription: `  // 设置 LongDescription 字段
IPNS is a PKI namespace, where names are the hashes of public keys, and
the private key enables publishing new (signed) values. In both publish
and resolve, the default name used is the node's own PeerID,
which is the hash of its public key.

You can use the 'ipfs key' commands to list and generate more names and their
respective keys.

Examples:

Publish an <ipfs-path> with your default name:

  > ipfs name publish /ipfs/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy
  Published to QmbCMUZw6JFeZ7Wp9jkzbye3Fzp2GGcPgC3nmeUjfVF87n: /ipfs/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy

Publish an <ipfs-path> with another name, added by an 'ipfs key' command:

  > ipfs key gen --type=rsa --size=2048 mykey
  > ipfs name publish --key=mykey /ipfs/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy
  Published to QmSrPmbaUKA3ZodhzPWZnpFgcPMFWF4QsxXbkWfEptTBJd: /ipfs/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy

Resolve the value of your name:

  > ipfs name resolve
  /ipfs/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy

Resolve the value of another name:

  > ipfs name resolve QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ
  /ipfs/QmSiTko9JZyabH56y2fussEt1A5oDqsFXB3CkvAqraFryz
`,  // 设置 LongDescription 字段
    },
}
// 解析dnslink的值
Resolve the value of a dnslink:

  > ipfs name resolve ipfs.io
  /ipfs/QmaBvfZooxWkrv7D3r8LS9moNjzD2o525XMZze69hhoxf5

`,
    },

    // 子命令映射
    Subcommands: map[string]*cmds.Command{
        "publish": PublishCmd,  // 发布命令
        "resolve": IpnsCmd,     // 解析命令
        "pubsub":  IpnsPubsubCmd,  // 发布订阅命令
        "inspect": IpnsInspectCmd,  // 检查命令
    },
}

// IPNS检查验证
type IpnsInspectValidation struct {
    Valid  bool    // 是否有效
    Reason string  // 原因
    Name   string  // 名称
}

// IpnsInspectEntry包含从IPNS条目反序列化的值：
// https://github.com/ipfs/specs/blob/main/ipns/IPNS.md#record-serialization-format
type IpnsInspectEntry struct {
    Value        string             // 值
    ValidityType *ipns.ValidityType // 有效性类型
    Validity     *time.Time         // 有效性
    Sequence     *uint64            // 序列
    TTL          *time.Duration     // TTL
}

type IpnsInspectResult struct {
    Entry         IpnsInspectEntry  // 条目
    PbSize        int               // PbSize
    SignatureType string            // 签名类型
    HexDump       string            // 十六进制转储
    Validation    *IpnsInspectValidation  // 验证
}

var IpnsInspectCmd = &cmds.Command{
    Status: cmds.Experimental,  // 实验状态
    Helptext: cmds.HelpText{
        Tagline: "Inspects an IPNS Record",  // 检查IPNS记录
        ShortDescription: `
Prints values inside of IPNS Record protobuf and its DAG-CBOR Data field.
Passing --verify will verify signature against provided public key.
`,  // 打印IPNS记录protobuf和其DAG-CBOR数据字段中的值。通过--verify将根据提供的公钥验证签名。

        LongDescription: `
Prints values inside of IPNS Record protobuf and its DAG-CBOR Data field.

The input can be a file or STDIN, the output can be JSON:

  $ ipfs routing get "/ipns/$PEERID" > ipns_record
  $ ipfs name inspect --enc=json < ipns_record

Values in PublicKey, SignatureV1 and SignatureV2 fields are raw bytes encoded
in Multibase. The Data field is DAG-CBOR represented as DAG-JSON.

Passing --verify will verify signature against provided public key.
`,  // 打印IPNS记录protobuf和其DAG-CBOR数据字段中的值。输入可以是文件或STDIN，输出可以是JSON。在PublicKey、SignatureV1和SignatureV2字段中的值是以Multibase编码的原始字节。数据字段是以DAG-JSON表示的DAG-CBOR。通过--verify将根据提供的公钥验证签名。
    },
    Arguments: []cmds.Argument{
        cmds.FileArg("record", true, false, "The IPNS record payload to be verified.").EnableStdin(),  // 要验证的IPNS记录有效载荷
    },
    # 创建一个空的选项列表
    Options: []cmds.Option{
        # 添加一个字符串选项，用于验证公共IPNS密钥的CID
        cmds.StringOption("verify", "CID of the public IPNS key to validate against."),
        # 添加一个布尔选项，用于指定是否包含原始Protobuf记录的完整十六进制转储，默认值为True
        cmds.BoolOption("dump", "Include a full hex dump of the raw Protobuf record.").WithDefault(true),
    },
    # 创建一个IpnsInspectResult类型的对象
    Type: IpnsInspectResult{},
    # 创建一个命令编码器映射
    Encoders: cmds.EncoderMap{
        # 将文本命令映射到一个特定的编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *IpnsInspectResult) error {
            # 创建一个制表符写入器
            tw := tabwriter.NewWriter(w, 0, 0, 1, ' ', 0)
            # 在函数返回时刷新并关闭制表符写入器
            defer tw.Flush()

            # 如果条目值不为空，则将值写入制表符写入器
            if out.Entry.Value != "" {
                fmt.Fprintf(tw, "Value:\t%q\n", out.Entry.Value)
            }

            # 如果条目有效性类型不为空，则将有效性类型写入制表符写入器
            if out.Entry.ValidityType != nil {
                fmt.Fprintf(tw, "Validity Type:\t%q\n", *out.Entry.ValidityType)
            }

            # 如果条目有效性不为空，则将有效性写入制表符写入器
            if out.Entry.Validity != nil {
                fmt.Fprintf(tw, "Validity:\t%q\n", out.Entry.Validity.Format(time.RFC3339Nano))
            }

            # 如果条目序列不为空，则将序列写入制表符写入器
            if out.Entry.Sequence != nil {
                fmt.Fprintf(tw, "Sequence:\t%d\n", *out.Entry.Sequence)
            }

            # 如果条目TTL不为空，则将TTL写入制表符写入器
            if out.Entry.TTL != nil {
                fmt.Fprintf(tw, "TTL:\t%s\n", out.Entry.TTL.String())
            }

            # 将Protobuf大小写入制表符写入器
            fmt.Fprintf(tw, "Protobuf Size:\t%d\n", out.PbSize)
            # 将签名类型写入制表符写入器
            fmt.Fprintf(tw, "Signature Type:\t%s\n", out.SignatureType)

            # 如果验证为空，则刷新制表符写入器并将消息写入输出
            if out.Validation == nil {
                tw.Flush()
                fmt.Fprintf(w, "\nThis record was not validated.\n")
            } else {
                # 否则，刷新制表符写入器并将验证结果消息写入输出
                tw.Flush()
                fmt.Fprintf(w, "\nValidation results:\n")

                # 将验证结果写入制表符写入器
                fmt.Fprintf(tw, "\tValid:\t%v\n", out.Validation.Valid)
                # 如果验证原因不为空，则将原因写入制表符写入器
                if out.Validation.Reason != "" {
                    fmt.Fprintf(tw, "\tReason:\t%s\n", out.Validation.Reason)
                }
                # 将验证名称写入制表符写入器
                fmt.Fprintf(tw, "\tName:\t%s\n", out.Validation.Name)
            }

            # 如果十六进制转储不为空，则刷新制表符写入器并将十六进制转储写入输出
            if out.HexDump != "" {
                tw.Flush()
                fmt.Fprintf(w, "\nHex Dump:\n")
                fmt.Fprintf(w, out.HexDump)
            }

            # 返回空值
            return nil
        }),
    },
# 闭合前面的函数定义
```