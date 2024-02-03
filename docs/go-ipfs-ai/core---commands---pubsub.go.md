# `kubo\core\commands\pubsub.go`

```go
package commands

import (
    "context"  // 导入上下文包
    "fmt"  // 导入格式化包
    "io"  // 导入输入输出包
    "net/http"  // 导入HTTP包
    "sort"  // 导入排序包

    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入命令环境包
    mbase "github.com/multiformats/go-multibase"  // 导入多重基础包
    "github.com/pkg/errors"  // 导入错误包

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入IPFS命令包
    options "github.com/ipfs/kubo/core/coreiface/options"  // 导入选项包
)

var PubsubCmd = &cmds.Command{  // 定义命令PubsubCmd
    Status: cmds.Deprecated,  // 设置状态为已弃用
    Helptext: cmds.HelpText{  // 设置帮助文本
        Tagline: "An experimental publish-subscribe system on ipfs.",  // 设置标语
        ShortDescription: `  // 设置简短描述
ipfs pubsub allows you to publish messages to a given topic, and also to
subscribe to new messages on a given topic.

DEPRECATED FEATURE (see https://github.com/ipfs/kubo/issues/9717)

  It is not intended in its current state to be used in a production
  environment.  To use, the daemon must be run with
  '--enable-pubsub-experiment'.
`,
    },
    Subcommands: map[string]*cmds.Command{  // 设置子命令
        "pub":   PubsubPubCmd,  // 发布命令
        "sub":   PubsubSubCmd,  // 订阅命令
        "ls":    PubsubLsCmd,  // 列出命令
        "peers": PubsubPeersCmd,  // 对等命令
    },
}

type pubsubMessage struct {  // 定义pubsubMessage结构
    From     string   `json:"from,omitempty"`  // 发送者
    Data     string   `json:"data,omitempty"`  // 数据
    Seqno    string   `json:"seqno,omitempty"`  // 序列号
    TopicIDs []string `json:"topicIDs,omitempty"`  // 主题ID
}

var PubsubSubCmd = &cmds.Command{  // 定义命令PubsubSubCmd
    Status: cmds.Deprecated,  // 设置状态为已弃用
    Helptext: cmds.HelpText{  // 设置帮助文本
        Tagline: "Subscribe to messages on a given topic.",  // 设置标语
        ShortDescription: `  // 设置简短描述
ipfs pubsub sub subscribes to messages on a given topic.

DEPRECATED FEATURE (see https://github.com/ipfs/kubo/issues/9717)

  It is not intended in its current state to be used in a production
  environment.  To use, the daemon must be run with
  '--enable-pubsub-experiment'.

PEER ENCODING

  Peer IDs in From fields are encoded using the default text representation
  from go-libp2p. This ensures the same string values as in 'ipfs pubsub peers'.
// 主题和数据编码
// 主题、数据和序列号都是二进制数据。为了确保所有字节都能正确传输，RPC 客户端和服务器将在后台使用多重编码。

// 通过传递 --enc=json 可以检查格式。ipfs multibase 命令可用于在用户空间对多重基字符串进行编码/解码。

// 参数包括一个必填的主题名称，当通过 HTTP RPC 发送时，主题名称会进行多重基编码。
Arguments: []cmds.Argument{
    cmds.StringArg("topic", true, false, "Name of topic to subscribe to (multibase encoded when sent over HTTP RPC)."),
},

// 在运行之前对 URL 参数进行编码
PreRun: urlArgsEncoder,

// 运行函数
Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
    // 从环境中获取 API
    api, err := cmdenv.GetApi(env, req)
    if err != nil {
        return err
    }
    // 如果存在错误，则返回错误
    if err := urlArgsDecoder(req, env); err != nil {
        return err
    }

    // 获取请求中的主题
    topic := req.Arguments[0]

    // 订阅主题
    sub, err := api.PubSub().Subscribe(req.Context, topic)
    if err != nil {
        return err
    }
    defer sub.Close()

    // 如果 res 实现了 http.Flusher 接口，则刷新
    if f, ok := res.(http.Flusher); ok {
        f.Flush()
    }

    // 循环处理消息
    for {
        msg, err := sub.Next(req.Context)
        if err == io.EOF || err == context.Canceled {
            return nil
        } else if err != nil {
            return err
        }

        // 将字节转换为字符串
        encoder, _ := mbase.EncoderByName("base64url")
        psm := pubsubMessage{
            Data:  encoder.Encode(msg.Data()),
            From:  msg.From().String(),
            Seqno: encoder.Encode(msg.Seq()),
        }
        for _, topic := range msg.Topics() {
            psm.TopicIDs = append(psm.TopicIDs, encoder.Encode([]byte(topic)))
        }
        // 发送消息
        if err := res.Emit(&psm); err != nil {
            return err
        }
    }
},
    # 创建一个编码器映射，将命令类型与对应的编码器函数关联起来
    Encoders: cmds.EncoderMap{
        # 将文本类型的消息编码为指定类型的编码器函数
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, psm *pubsubMessage) error {
            # 解码 pubsubMessage 中的数据，并将解码后的数据写入到指定的 io.Writer 中
            _, dec, err := mbase.Decode(psm.Data)
            if err != nil:
                return err
            _, err = w.Write(dec)
            return err
        }),
        # 已废弃的、未记录的格式，用于测试，不再使用
        # <message.payload>\n<message.payload>\n
        "ndpayload": cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, psm *pubsubMessage) error {
            return errors.New("--enc=ndpayload was removed, use --enc=json instead")
        }),
        # 已废弃的、未记录的格式，用于测试，不再使用
        # <varint-len><message.payload><varint-len><message.payload>
        "lenpayload": cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, psm *pubsubMessage) error {
            return errors.New("--enc=lenpayload was removed, use --enc=json instead")
        }),
    },
    # 指定 pubsubMessage 的类型为默认类型
    Type: pubsubMessage{},
# 定义 PubsubPubCmd 命令
var PubsubPubCmd = &cmds.Command{
    # 命令状态为 Deprecated
    Status: cmds.Deprecated,
    # 命令的帮助文本
    Helptext: cmds.HelpText{
        # 命令的简短描述
        Tagline: "Publish data to a given pubsub topic.",
        # 命令的详细描述
        ShortDescription: `
ipfs pubsub pub publishes a message to a specified topic.
It reads binary data from stdin or a file.

DEPRECATED FEATURE (see https://github.com/ipfs/kubo/issues/9717)

  It is not intended in its current state to be used in a production
  environment.  To use, the daemon must be run with
  '--enable-pubsub-experiment'.

HTTP RPC ENCODING

  The data to be published is sent in HTTP request body as multipart/form-data.

  Topic names are binary data too. To ensure all bytes are transferred
  correctly via URL params, the RPC client and server will use multibase
  encoding behind the scenes.

`,
    },
    # 命令的参数
    Arguments: []cmds.Argument{
        # topic 参数
        cmds.StringArg("topic", true, false, "Topic to publish to (multibase encoded when sent over HTTP RPC)."),
        # data 参数
        cmds.FileArg("data", true, false, "The data to be published.").EnableStdin(),
    },
    # 在运行之前执行的函数
    PreRun: urlArgsEncoder,
    # 命令的运行函数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 获取 API
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }
        # 解码 URL 参数
        if err := urlArgsDecoder(req, env); err != nil {
            return err
        }

        # 获取 topic 参数
        topic := req.Arguments[0]

        # 读取作为文件传递的数据
        file, err := cmdenv.GetFileArg(req.Files.Entries())
        if err != nil {
            return err
        }
        defer file.Close()
        # 读取文件数据
        data, err := io.ReadAll(file)
        if err != nil {
            return err
        }

        # 发布数据
        return api.PubSub().Publish(req.Context, topic, data)
    },
}

# 定义 PubsubLsCmd 命令
var PubsubLsCmd = &cmds.Command{
    # 命令状态为 Deprecated
    Status: cmds.Deprecated,
    # 命令的帮助文本
    Helptext: cmds.HelpText{
        # 命令的简短描述
        Tagline: "List subscribed topics by name.",
        # 命令的详细描述
        ShortDescription: `
ipfs pubsub ls lists out the names of topics you are currently subscribed to.
DEPRECATED FEATURE (see https://github.com/ipfs/kubo/issues/9717)

  It is not intended in its current state to be used in a production
  environment.  To use, the daemon must be run with
  '--enable-pubsub-experiment'.

TOPIC ENCODING

  Topic names are a binary data. To ensure all bytes are transferred
  correctly RPC client and server will use multibase encoding behind
  the scenes.

  You can inspect the format by passing --enc=json. ipfs multibase commands
  can be used for encoding/decoding multibase strings in the userland.



    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 获取API对象和错误信息
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        // 获取当前主题列表
        l, err := api.PubSub().Ls(req.Context)
        if err != nil {
            return err
        }

        // 使用multibase编码器对主题进行编码
        encoder, _ := mbase.EncoderByName("base64url")
        for n, topic := range l {
            l[n] = encoder.Encode([]byte(topic))
        }

        // 发送一次编码后的主题列表
        return cmds.EmitOnce(res, stringList{l})
    },
    Type: stringList{},
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(multibaseDecodedStringListEncoder),
    },
}

// 将多重基编码的字符串列表编码为文本表示，其中每个字符串都放在单独的一行中，非可打印/不安全的字符被转义
// （这可以保护终端输出不被非ASCII主题名称破坏）
func safeTextListEncoder(req *cmds.Request, w io.Writer, list *stringList) error {
    # 遍历字符串列表中的每个字符串
    for _, str := range list.Strings {
        # 将经过 cmdenv.EscNonPrint 处理的字符串格式化后写入 w 中
        _, err := fmt.Fprintf(w, "%s\n", cmdenv.EscNonPrint(str))
        # 如果出现错误，返回错误信息
        if err != nil {
            return err
        }
    }
    # 如果没有出现错误，返回空值
    return nil
// 定义 PubsubPeersCmd 命令，用于列出当前正在进行 pubsub 通信的对等节点
var PubsubPeersCmd = &cmds.Command{
    // 命令状态为已废弃
    Status: cmds.Deprecated,
    // 命令的帮助文本
    Helptext: cmds.HelpText{
        // 命令的简短描述
        Tagline: "List peers we are currently pubsubbing with.",
        // 命令的详细描述
        ShortDescription: `
ipfs pubsub peers with no arguments lists out the pubsub peers you are
currently connected to. If given a topic, it will list connected peers who are
subscribed to the named topic.

DEPRECATED FEATURE (see https://github.com/ipfs/kubo/issues/9717)

  It is not intended in its current state to be used in a production
  environment.  To use, the daemon must be run with
  '--enable-pubsub-experiment'.

TOPIC AND DATA ENCODING

  Topic names are a binary data. To ensure all bytes are transferred
  correctly RPC client and server will use multibase encoding behind
  the scenes.

  You can inspect the format by passing --enc=json. ipfs multibase commands
  can be used for encoding/decoding multibase strings in the userland.
`,
    },
    // 命令的参数
    Arguments: []cmds.Argument{
        cmds.StringArg("topic", false, false, "Topic to list connected peers of."),
    },
    // 在运行之前对 URL 参数进行编码
    PreRun: urlArgsEncoder,
    // 命令的运行函数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 获取 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }
        // 解码 URL 参数
        if err := urlArgsDecoder(req, env); err != nil {
            return err
        }

        var topic string
        if len(req.Arguments) == 1 {
            topic = req.Arguments[0]
        }

        // 获取 pubsub 通信的对等节点
        peers, err := api.PubSub().Peers(req.Context, options.PubSub.Topic(topic))
        if err != nil {
            return err
        }

        // 创建字符串列表
        list := &stringList{make([]string, 0, len(peers))}

        // 遍历对等节点，将其加入字符串列表
        for _, peer := range peers {
            list.Strings = append(list.Strings, peer.String())
        }
        // 对字符串列表进行排序
        sort.Strings(list.Strings)
        return cmds.EmitOnce(res, list)
    },
    // 返回值类型为字符串列表
    Type: stringList{},
    // 编码器映射
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(safeTextListEncoder),
    },
}
// 将二进制数据编码为多基字符串，以便在 URL 参数中传递
// （避免 https://github.com/ipfs/kubo/issues/7939 中描述的问题）
func urlArgsEncoder(req *cmds.Request, env cmds.Environment) error {
    // 根据名称获取编码器，使用 base64url 编码器
    encoder, _ := mbase.EncoderByName("base64url")
    // 遍历请求参数，对每个参数进行编码
    for n, arg := range req.Arguments {
        req.Arguments[n] = encoder.Encode([]byte(arg))
    }
    return nil
}

// 解码作为多基字符串传递的二进制数据
// （避免 https://github.com/ipfs/kubo/issues/7939 中描述的问题）
func urlArgsDecoder(req *cmds.Request, env cmds.Environment) error {
    // 遍历请求参数，对每个参数进行解码
    for n, arg := range req.Arguments {
        // 解码参数，获取编码类型、数据和可能的错误
        encoding, data, err := mbase.Decode(arg)
        if err != nil {
            return errors.Wrap(err, "URL arg must be multibase encoded")
        }

        // 强制要求在通过 URL 参数传递数据时使用 URL 安全编码
        // - 如果不这样做，我们会遇到类似 https://github.com/ipfs/kubo/issues/7939 的数据损坏问题
        // - 我们不能只拒绝 base64，因为可能还有其他不是 URL 安全的基数 - 最好强制使用已知在 URL 上下文中安全的 base64url
        if encoding != mbase.Base64url {
            return errors.New("URL arg must be base64url encoded")
        }

        // 将解码后的数据转换为字符串
        req.Arguments[n] = string(data)
    }
    return nil
}
```