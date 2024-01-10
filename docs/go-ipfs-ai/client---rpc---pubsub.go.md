# `kubo\client\rpc\pubsub.go`

```
package rpc

import (
    "bytes" // 导入 bytes 包，用于操作字节
    "context" // 导入 context 包，用于处理上下文
    "encoding/json" // 导入 json 包，用于处理 JSON 数据
    "io" // 导入 io 包，用于进行 I/O 操作

    iface "github.com/ipfs/kubo/core/coreiface" // 导入 coreiface 包，用于定义接口
    caopts "github.com/ipfs/kubo/core/coreiface/options" // 导入 options 包，用于定义选项
    "github.com/libp2p/go-libp2p/core/peer" // 导入 peer 包，用于处理 libp2p 的对等节点
    mbase "github.com/multiformats/go-multibase" // 导入 multibase 包，用于处理多种基数编码
)

type PubsubAPI HttpApi // 定义 PubsubAPI 类型为 HttpApi

func (api *PubsubAPI) Ls(ctx context.Context) ([]string, error) { // 定义 PubsubAPI 结构体的 Ls 方法
    var out struct { // 定义结构体 out
        Strings []string // 字符串数组
    }

    if err := api.core().Request("pubsub/ls").Exec(ctx, &out); err != nil { // 发送 pubsub/ls 请求并执行，将结果存储到 out 中
        return nil, err // 如果出错，返回 nil 和错误
    }
    topics := make([]string, len(out.Strings)) // 创建与 out.Strings 长度相同的字符串数组 topics
    for n, mb := range out.Strings { // 遍历 out.Strings
        _, topic, err := mbase.Decode(mb) // 解码 multibase 编码的字符串
        if err != nil { // 如果出错
            return nil, err // 返回 nil 和错误
        }
        topics[n] = string(topic) // 将解码后的字符串存储到 topics 中
    }
    return topics, nil // 返回 topics 和 nil
}

func (api *PubsubAPI) Peers(ctx context.Context, opts ...caopts.PubSubPeersOption) ([]peer.ID, error) { // 定义 PubsubAPI 结构体的 Peers 方法
    options, err := caopts.PubSubPeersOptions(opts...) // 获取 PubSubPeersOptions
    if err != nil { // 如果出错
        return nil, err // 返回 nil 和错误
    }

    var out struct { // 定义结构体 out
        Strings []string // 字符串数组
    }

    var optionalTopic string // 定义可选的主题字符串
    if len(options.Topic) > 0 { // 如果主题长度大于 0
        optionalTopic = toMultibase([]byte(options.Topic)) // 将主题转换为 multibase 编码的字符串
    }
    if err := api.core().Request("pubsub/peers", optionalTopic).Exec(ctx, &out); err != nil { // 发送 pubsub/peers 请求并执行，将结果存储到 out 中
        return nil, err // 如果出错，返回 nil 和错误
    }

    res := make([]peer.ID, len(out.Strings)) // 创建与 out.Strings 长度相同的 peer.ID 数组 res
    for i, sid := range out.Strings { // 遍历 out.Strings
        id, err := peer.Decode(sid) // 解码 peer ID
        if err != nil { // 如果出错
            return nil, err // 返回 nil 和错误
        }
        res[i] = id // 将解码后的 peer ID 存储到 res 中
    }
    return res, nil // 返回 res 和 nil
}

func (api *PubsubAPI) Publish(ctx context.Context, topic string, message []byte) error { // 定义 PubsubAPI 结构体的 Publish 方法
    return api.core().Request("pubsub/pub", toMultibase([]byte(topic)). // 发送 pubsub/pub 请求并执行，将主题转换为 multibase 编码的字符串
        FileBody(bytes.NewReader(message)). // 将消息内容作为文件体
        Exec(ctx, nil) // 执行请求
}

type pubsubSub struct { // 定义 pubsubSub 结构体
    messages chan pubsubMessage // 定义 pubsubMessage 类型的通道 messages

    done    chan struct{} // 定义结构体类型的通道 done
    rcloser func() error // 定义返回错误的函数 rcloser
}

type pubsubMessage struct { // 定义 pubsubMessage 结构体
    JFrom     string   `json:"from,omitempty"` // 定义 JSON 字段 JFrom
    JData     string   `json:"data,omitempty"` // 定义 JSON 字段 JData
    # 定义一个字符串类型的字段 JSeqno，并指定在 JSON 编码时使用 seqno 作为键，如果字段值为空则忽略
    JSeqno    string   `json:"seqno,omitempty"`
    # 定义一个字符串数组类型的字段 JTopicIDs，并指定在 JSON 编码时使用 topicIDs 作为键，如果字段值为空则忽略
    JTopicIDs []string `json:"topicIDs,omitempty"`

    # 从文本/multibase包装中解包后的真实值
    # 定义一个 peer.ID 类型的字段 from
    from   peer.ID
    # 定义一个字节数组类型的字段 data
    data   []byte
    # 定义一个字节数组类型的字段 seqno
    seqno  []byte
    # 定义一个字符串数组类型的字段 topics
    topics []string

    # 定义一个错误类型的字段 err
    err error
// 返回消息的发送者 peer.ID
func (msg *pubsubMessage) From() peer.ID {
    return msg.from
}

// 返回消息的数据 []byte
func (msg *pubsubMessage) Data() []byte {
    return msg.data
}

// 返回消息的序列号 []byte
func (msg *pubsubMessage) Seq() []byte {
    return msg.seqno
}

// 返回消息的主题 []string
// TODO: 我们是否要保持这个接口为 []string，还是改为更正确的 [][]byte？
func (msg *pubsubMessage) Topics() []string {
    return msg.topics
}

// 获取下一条消息
func (s *pubsubSub) Next(ctx context.Context) (iface.PubSubMessage, error) {
    select {
    case msg, ok := <-s.messages:
        if !ok {
            return nil, io.EOF
        }
        if msg.err != nil {
            return nil, msg.err
        }
        // 从 text/multibase 包装中解析值
        var err error
        msg.from, err = peer.Decode(msg.JFrom)
        if err != nil {
            return nil, err
        }
        _, msg.data, err = mbase.Decode(msg.JData)
        if err != nil {
            return nil, err
        }
        _, msg.seqno, err = mbase.Decode(msg.JSeqno)
        if err != nil {
            return nil, err
        }
        for _, mbt := range msg.JTopicIDs {
            _, topic, err := mbase.Decode(mbt)
            if err != nil {
                return nil, err
            }
            msg.topics = append(msg.topics, string(topic))
        }
        return &msg, nil
    case <-ctx.Done():
        return nil, ctx.Err()
    }
}

// 订阅主题
func (api *PubsubAPI) Subscribe(ctx context.Context, topic string, opts ...caopts.PubSubSubscribeOption) (iface.PubSubSubscription, error) {
    /* 现在我们没有选项（发现已被弃用）
    options, err := caopts.PubSubSubscribeOptions(opts...)
    if err != nil {
        return nil, err
    }
    */
    resp, err := api.core().Request("pubsub/sub", toMultibase([]byte(topic))).Send(ctx)
    if err != nil {
        return nil, err
    }
    if resp.Error != nil {
        return nil, resp.Error
    }
}
    # 创建一个 pubsubSub 结构体的指针，并初始化其中的字段
    sub := &pubsubSub{
        messages: make(chan pubsubMessage),  # 创建一个 pubsubMessage 类型的通道
        done:     make(chan struct{}),  # 创建一个空结构体类型的通道
        rcloser: func() error {  # 定义一个函数，用于关闭 resp
            return resp.Cancel()
        },
    }

    # 创建一个 JSON 解码器，用于从 resp.Output 中解码 JSON 数据
    dec := json.NewDecoder(resp.Output)

    # 启动一个 goroutine，用于从 JSON 解码器中读取消息并发送到 sub.messages 中
    go func() {
        defer close(sub.messages)  # 在 goroutine 结束时关闭 sub.messages 通道

        for {
            var msg pubsubMessage  # 声明一个 pubsubMessage 类型的变量
            if err := dec.Decode(&msg); err != nil {  # 从 JSON 解码器中解码消息到 msg 变量
                if err == io.EOF {  # 如果遇到文件结束错误
                    return  # 退出循环
                }
                msg.err = err  # 将错误信息保存到消息结构体中
            }

            select {
            case sub.messages <- msg:  # 将消息发送到 sub.messages 通道
            case <-sub.done:  # 如果 sub.done 通道被关闭
                return  # 退出循环
            case <-ctx.Done():  # 如果上下文被取消
                return  # 退出循环
            }
        }
    }()

    return sub, nil  # 返回 sub 结构体指针和空错误
// 关闭 pubsubSub 结构体的方法，用于关闭订阅
func (s *pubsubSub) Close() error {
    // 如果 done 通道不为空，则关闭通道
    if s.done != nil {
        close(s.done)
        s.done = nil
    }
    // 调用 rcloser 方法关闭资源
    return s.rcloser()
}

// 返回 PubsubAPI 结构体的 core 方法，返回类型为 *HttpApi
func (api *PubsubAPI) core() *HttpApi {
    return (*HttpApi)(api)
}

// 将字节编码为 URL 安全的 multibase，可用于通过 HTTP RPC（URL 或正文）发送
func toMultibase(data []byte) string {
    // 使用 mbase 包将数据编码为 Base64url 格式的 multibase
    mb, _ := mbase.Encode(mbase.Base64url, data)
    // 返回编码后的 multibase
    return mb
}
```