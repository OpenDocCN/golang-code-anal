# `kubo\test\integration\pubsub_msg_seen_cache_test.go`

```go
package integrationtest

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "context"  // 导入 context 包，用于处理上下文
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"  // 导入 io 包，用于进行输入输出操作
    "testing"  // 导入 testing 包，用于编写测试函数
    "time"  // 导入 time 包，用于处理时间

    "go.uber.org/fx"  // 导入 fx 包，用于依赖注入

    "github.com/ipfs/boxo/bootstrap"  // 导入 boxo/bootstrap 包
    "github.com/ipfs/kubo/config"  // 导入 kubo/config 包
    "github.com/ipfs/kubo/core"  // 导入 kubo/core 包
    "github.com/ipfs/kubo/core/coreapi"  // 导入 kubo/core/coreapi 包
    libp2p2 "github.com/ipfs/kubo/core/node/libp2p"  // 导入 kubo/core/node/libp2p 包，并重命名为 libp2p2
    "github.com/ipfs/kubo/repo"  // 导入 kubo/repo 包

    "github.com/ipfs/go-datastore"  // 导入 go-datastore 包
    syncds "github.com/ipfs/go-datastore/sync"  // 导入 sync 包，并重命名为 syncds

    pubsub "github.com/libp2p/go-libp2p-pubsub"  // 导入 go-libp2p-pubsub 包
    pubsub_pb "github.com/libp2p/go-libp2p-pubsub/pb"  // 导入 go-libp2p-pubsub/pb 包
    "github.com/libp2p/go-libp2p-pubsub/timecache"  // 导入 go-libp2p-pubsub/timecache 包
    "github.com/libp2p/go-libp2p/core/peer"  // 导入 go-libp2p/core/peer 包

    mock "github.com/ipfs/kubo/core/mock"  // 导入 kubo/core/mock 包
    mocknet "github.com/libp2p/go-libp2p/p2p/net/mock"  // 导入 go-libp2p/p2p/net/mock 包
)

func TestMessageSeenCacheTTL(t *testing.T) {
    t.Skip("skipping PubSub seen cache TTL test due to flakiness")  // 跳过 PubSub seen cache TTL 测试
    if err := RunMessageSeenCacheTTLTest(t, "10s"); err != nil {  // 如果运行 RunMessageSeenCacheTTLTest 函数出错
        t.Fatal(err)  // 输出错误信息
    }
}

func mockNode(ctx context.Context, mn mocknet.Mocknet, pubsubEnabled bool, seenMessagesCacheTTL string) (*core.IpfsNode, error) {
    ds := syncds.MutexWrap(datastore.NewMapDatastore())  // 使用 sync 包中的 MutexWrap 函数包装新建的 MapDatastore 对象，并赋值给 ds
    cfg, err := config.Init(io.Discard, 2048)  // 调用 config 包中的 Init 函数，初始化配置，并将结果赋值给 cfg 和 err
    if err != nil {  // 如果 err 不为空
        return nil, err  // 返回空和 err
    }
    count := len(mn.Peers())  // 获取 mn 中对等节点的数量，并赋值给 count
    cfg.Addresses.Swarm = []string{  // 设置配置中 Swarm 地址的字符串数组
        fmt.Sprintf("/ip4/18.0.%d.%d/tcp/4001", count>>16, count&0xFF),  // 格式化输出 Swarm 地址
    }
    cfg.Datastore = config.Datastore{}  // 设置配置中的 Datastore 为一个空的 Datastore 对象
    if pubsubEnabled {  // 如果 pubsubEnabled 为真
        cfg.Pubsub.Enabled = config.True  // 设置配置中的 Pubsub.Enabled 为 True
        var ttl *config.OptionalDuration  // 声明一个 OptionalDuration 类型的指针变量 ttl
        if len(seenMessagesCacheTTL) > 0 {  // 如果 seenMessagesCacheTTL 的长度大于 0
            ttl = &config.OptionalDuration{}  // 将一个新建的 OptionalDuration 对象的地址赋值给 ttl
            if err = ttl.UnmarshalJSON([]byte(seenMessagesCacheTTL)); err != nil {  // 将 seenMessagesCacheTTL 转换为 JSON 格式，并解析到 ttl 中，如果出错
                return nil, err  // 返回空和 err
            }
        }
        cfg.Pubsub.SeenMessagesTTL = ttl  // 设置配置中的 Pubsub.SeenMessagesTTL 为 ttl
    }
    # 使用给定的上下文和构建配置创建一个新的节点
    return core.NewNode(ctx, &core.BuildCfg{
        # 设置节点在线状态为真
        Online:  true,
        # 设置节点的路由选项为libp2p2.DHTServerOption
        Routing: libp2p2.DHTServerOption,
        # 使用模拟的存储库创建一个存储库的模拟对象
        Repo: &repo.Mock{
            C: *cfg,  # 设置配置
            D: ds,    # 设置数据存储
        },
        # 使用模拟的主机选项创建一个主机
        Host: mock.MockHostOption(mn),
        # 设置额外的选项，以键值对的形式存储
        ExtraOpts: map[string]bool{
            "pubsub": pubsubEnabled,  # 设置pubsub选项为pubsubEnabled变量的值
        },
    })
// 运行消息已查看缓存TTL测试
func RunMessageSeenCacheTTLTest(t *testing.T, seenMessagesCacheTTL string) error {
    // 创建一个带有取消函数的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()

    var bootstrapNode, consumerNode, producerNode *core.IpfsNode
    var bootstrapPeerID, consumerPeerID, producerPeerID peer.ID

    // 创建一个模拟网络
    mn := mocknet.New()
    // 创建一个模拟节点作为引导节点，不需要PubSub配置
    bootstrapNode, err := mockNode(ctx, mn, false, "")
    if err != nil {
        t.Fatal(err)
    }
    // 获取引导节点的PeerID
    bootstrapPeerID = bootstrapNode.PeerHost.ID()
    // 延迟关闭引导节点
    defer bootstrapNode.Close()

    // 创建一个模拟节点作为消费者节点，使用传入的已查看消息缓存TTL
    consumerNode, err = mockNode(ctx, mn, true, seenMessagesCacheTTL)
    if err != nil {
        t.Fatal(err)
    }
    // 获取消费者节点的PeerID
    consumerPeerID = consumerNode.PeerHost.ID()
    // 延迟关闭消费者节点
    defer consumerNode.Close()

    // 解析传入的已查看消息缓存TTL为时间间隔
    ttl, err := time.ParseDuration(seenMessagesCacheTTL)
    if err != nil {
        t.Fatal(err)
    }

    // 用于记录时间线的起始时间
    startTime := time.Time{}

    // 用于覆盖消息ID的发送消息ID
    sendMsgID := ""

    // 为生产者设置pubsub消息ID生成覆盖
    # 注册 FX 期权函数，返回期权选项和错误信息
    core.RegisterFXOptionFunc(func(info core.FXNodeInfo) ([]fx.Option, error) {
        # 创建一个公共订阅选项的空数组
        var pubsubOptions []pubsub.Option
        # 将公共订阅选项附加到数组中
        pubsubOptions = append(
            pubsubOptions,
            # 设置已见消息的存活时间
            pubsub.WithSeenMessagesTTL(ttl),
            # 设置消息 ID 函数
            pubsub.WithMessageIdFn(func(pmsg *pubsub_pb.Message) string {
                # 获取当前时间
                now := time.Now()
                # 如果起始时间的秒数为 0，则将起始时间设置为当前时间
                if startTime.Second() == 0 {
                    startTime = now
                }
                # 计算时间间隔
                timeElapsed := now.Sub(startTime).Seconds()
                # 获取消息内容
                msg := string(pmsg.Data)
                # 获取消息发送者的 peer ID
                from, _ := peer.IDFromBytes(pmsg.From)
                var msgID string
                # 如果发送者是生产者 peer，则使用发送消息的 ID
                if from == producerPeerID {
                    msgID = sendMsgID
                    t.Logf("sending [%s] with message ID [%s] at T%fs", msg, msgID, timeElapsed)
                } else {
                    # 否则使用默认的消息 ID 函数
                    msgID = pubsub.DefaultMsgIdFn(pmsg)
                }
                return msgID
            }),
            # 设置已见消息策略
            pubsub.WithSeenMessagesStrategy(timecache.Strategy_LastSeen),
        )
        # 返回 FX 选项数组和 nil 错误
        return append(
            info.FXOptions,
            # 提供主题发现功能
            fx.Provide(libp2p2.TopicDiscovery()),
            # 装饰 GossipSub 功能
            fx.Decorate(libp2p2.GossipSub(pubsubOptions...)),
        ), nil
    })

    # 创建生产者节点
    producerNode, err = mockNode(ctx, mn, false, "") // PubSub configuration comes from overrides above
    if err != nil {
        t.Fatal(err)
    }
    # 获取生产者节点的 peer ID
    producerPeerID = producerNode.PeerHost.ID()
    # 延迟关闭生产者节点
    defer producerNode.Close()

    # 输出引导节点的 peer ID、消费者节点的 peer ID 和生产者节点的 peer ID
    t.Logf("bootstrap peer=%s, consumer peer=%s, producer peer=%s", bootstrapPeerID, consumerPeerID, producerPeerID)

    # 创建生产者节点的核心 API
    producerAPI, err := coreapi.NewCoreAPI(producerNode)
    if err != nil {
        t.Fatal(err)
    }
    # 创建消费者节点的核心 API
    consumerAPI, err := coreapi.NewCoreAPI(consumerNode)
    if err != nil {
        t.Fatal(err)
    }

    # 将所有节点连接起来
    err = mn.LinkAll()
    if err != nil {
        t.Fatal(err)
    }

    # 获取引导节点的 peer 信息
    bis := bootstrapNode.Peerstore.PeerInfo(bootstrapNode.PeerHost.ID())
    # 创建包含引导节点 peer 信息的引导配置
    bcfg := bootstrap.BootstrapConfigWithPeers([]peer.AddrInfo{bis})
    // 如果生产者节点的引导过程出现错误，则记录错误并终止测试
    if err = producerNode.Bootstrap(bcfg); err != nil {
        t.Fatal(err)
    }
    // 如果消费者节点的引导过程出现错误，则记录错误并终止测试
    if err = consumerNode.Bootstrap(bcfg); err != nil {
        t.Fatal(err)
    }

    // 设置消费者订阅
    const TopicName = "topic"
    consumerSubscription, err := consumerAPI.PubSub().Subscribe(ctx, TopicName)
    if err != nil {
        t.Fatal(err)
    }
    // 内联定义的实用函数，包括上下文以便在闭包中使用
    now := func() float64 {
        return time.Since(startTime).Seconds()
    }
    ctr := 0
    msgGen := func() string {
        ctr++
        return fmt.Sprintf("msg_%d", ctr)
    }
    produceMessage := func() string {
        msgTxt := msgGen()
        // 发布消息到指定主题
        err = producerAPI.PubSub().Publish(ctx, TopicName, []byte(msgTxt))
        if err != nil {
            t.Fatal(err)
        }
        return msgTxt
    }
    consumeMessage := func(msgTxt string, shouldFind bool) {
        // 设置一个单独的定时上下文来接收消息
        rxCtx, rxCancel := context.WithTimeout(context.Background(), time.Second)
        defer rxCancel()
        // 获取下一条消息
        msg, err := consumerSubscription.Next(rxCtx)
        if shouldFind {
            if err != nil {
                t.Logf("expected but did not receive [%s] at T%fs", msgTxt, now())
                t.Fatal(err)
            }
            t.Logf("received [%s] at T%fs", string(msg.Data()), now())
            if !bytes.Equal(msg.Data(), []byte(msgTxt)) {
                t.Fatalf("consumed data [%s] does not match published data [%s]", string(msg.Data()), msgTxt)
            }
        } else {
            if err == nil {
                t.Logf("not expected but received [%s] at T%fs", string(msg.Data()), now())
                t.Fail()
            }
            t.Logf("did not receive [%s] at T%fs", msgTxt, now())
        }
    }

    const MsgID1 = "MsgID1"
    const MsgID2 = "MsgID2"
    const MsgID3 = "MsgID3"

    // 发送带有我们将要复制的消息ID的消息1
    // 记录发送消息1的时间
    sentMsg1 := time.Now()
    // 设置发送消息的ID为MsgID1
    sendMsgID = MsgID1
    // 生成消息文本
    msgTxt := produceMessage()
    // 应该找到消息，因为它是新的
    consumeMessage(msgTxt, true)

    // 发送带有重复消息ID的消息2
    sendMsgID = MsgID1
    msgTxt = produceMessage()
    // 不应该找到消息，因为它已经被去重（在SeenMessagesTTL窗口内发送了2次）
    consumeMessage(msgTxt, false)

    // 发送带有新消息ID的消息3
    sendMsgID = MsgID2
    msgTxt = produceMessage()
    // 应该找到消息，因为它是新的
    consumeMessage(msgTxt, true)

    // 等待直到消息1发送后的SeenMessagesTTL窗口快要过去
    time.Sleep(time.Until(sentMsg1.Add(ttl - 100*time.Millisecond)))

    // 发送带有重复消息ID的消息4
    sendMsgID = MsgID1
    msgTxt = produceMessage()
    // 不应该找到消息，因为它已经被去重（在SeenMessagesTTL窗口内发送了3次）。然而，这次消息的过期时间也应该被推迟一个整个SeenMessagesTTL窗口，因为默认时间缓存现在实现了一个滑动窗口算法。
    consumeMessage(msgTxt, false)

    // 发送带有重复消息ID的消息5。这将是上面最后一次尝试的一秒后，因为确定找不到消息需要一秒的时间。这将使得这次尝试在消息1开始的SeenMessagesTTL窗口过期后约1秒。
    sentMsg5 := time.Now()
    sendMsgID = MsgID1
    msgTxt = produceMessage()
    // 不应该找到消息，因为它已经被去重（自更新SeenMessagesTTL窗口开始以来发送了2次）。这次，过期时间应该再次被推迟一个SeenMessagesTTL窗口。
    consumeMessage(msgTxt, false)

    // 发送一个在SeenMessagesTTL窗口内未见过的消息6
    sendMsgID = MsgID2
    msgTxt = produceMessage()
    // 如果自上次读取以来的时间大于 SeenMessagesTTL，则应该找到消息，因此它看起来像是一条新消息。
    consumeMessage(msgTxt, true)

    // 休眠一个完整的 SeenMessagesTTL 窗口，让缓存条目超时
    time.Sleep(time.Until(sentMsg5.Add(ttl + 100*time.Millisecond)))

    // 使用重复的消息 ID 发送消息 7
    sendMsgID = MsgID1
    msgTxt = produceMessage()
    // 这次应该找到消息，因为自上次读取以来的时间大于 SeenMessagesTTL，所以它看起来像是一条新消息。
    consumeMessage(msgTxt, true)

    // 使用全新的消息 ID 发送消息 8
    //
    // 这一步并不是严格必要的，但为了谨慎起见已经添加了。
    sendMsgID = MsgID3
    msgTxt = produceMessage()
    // 应该找到消息，因为它是全新的
    consumeMessage(msgTxt, true)
    return nil
# 闭合前面的函数定义
```