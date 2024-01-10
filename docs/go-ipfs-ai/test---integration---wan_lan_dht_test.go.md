# `kubo\test\integration\wan_lan_dht_test.go`

```
package integrationtest

import (
    "context"  // 导入上下文包，用于控制程序的执行流程
    "encoding/binary"  // 导入二进制编码包，用于处理二进制数据
    "fmt"  // 导入格式化包，用于格式化输出
    "math"  // 导入数学包，提供数学函数和常量
    "math/rand"  // 导入随机数包，用于生成随机数
    "net"  // 导入网络包，提供网络相关的函数和类型
    "testing"  // 导入测试包，用于编写测试函数
    "time"  // 导入时间包，提供时间相关的函数和类型

    "github.com/ipfs/go-cid"  // 导入CID包，用于处理CID（Content Identifier）
    "github.com/ipfs/kubo/core"  // 导入core包，提供核心功能
    mock "github.com/ipfs/kubo/core/mock"  // 导入模拟包，用于模拟核心功能
    libp2p2 "github.com/ipfs/kubo/core/node/libp2p"  // 导入libp2p包，提供libp2p节点功能

    testutil "github.com/libp2p/go-libp2p-testing/net"  // 导入测试工具包，提供网络测试工具
    corenet "github.com/libp2p/go-libp2p/core/network"  // 导入核心网络包，提供核心网络功能
    "github.com/libp2p/go-libp2p/core/peerstore"  // 导入peerstore包，提供对等节点存储功能
    mocknet "github.com/libp2p/go-libp2p/p2p/net/mock"  // 导入模拟网络包，提供模拟网络功能

    ma "github.com/multiformats/go-multiaddr"  // 导入多地址包，提供多地址功能
)

func TestDHTConnectivityFast(t *testing.T) {
    conf := testutil.LatencyConfig{  // 定义测试配置
        NetworkLatency:    0,  // 设置网络延迟为0
        RoutingLatency:    0,  // 设置路由延迟为0
        BlockstoreLatency: 0,  // 设置块存储延迟为0
    }
    if err := RunDHTConnectivity(conf, 5); err != nil {  // 调用RunDHTConnectivity函数进行DHT连接测试
        t.Fatal(err)  // 如果测试失败，输出错误信息
    }
}

func TestDHTConnectivitySlowNetwork(t *testing.T) {
    SkipUnlessEpic(t)  // 跳过非史诗级测试
    conf := testutil.LatencyConfig{NetworkLatency: 400 * time.Millisecond}  // 定义测试配置，设置网络延迟为400毫秒
    if err := RunDHTConnectivity(conf, 5); err != nil {  // 调用RunDHTConnectivity函数进行DHT连接测试
        t.Fatal(err)  // 如果测试失败，输出错误信息
    }
}

func TestDHTConnectivitySlowRouting(t *testing.T) {
    SkipUnlessEpic(t)  // 跳过非史诗级测试
    conf := testutil.LatencyConfig{RoutingLatency: 400 * time.Millisecond}  // 定义测试配置，设置路由延迟为400毫秒
    if err := RunDHTConnectivity(conf, 5); err != nil {  // 调用RunDHTConnectivity函数进行DHT连接测试
        t.Fatal(err)  // 如果测试失败，输出错误信息
    }
}

// wan prefix must have a real corresponding ASN for the peer diversity filter to work.
var (
    wanPrefix = net.ParseIP("2001:218:3004::")  // 定义WAN前缀IP地址
    lanPrefix = net.ParseIP("fe80::")  // 定义LAN前缀IP地址
)

func makeAddr(n uint32, wan bool) ma.Multiaddr {
    var ip net.IP  // 定义IP地址变量
    if wan {  // 如果是WAN
        ip = append(net.IP{}, wanPrefix...)  // 将WAN前缀IP地址复制到IP变量
    } else {  // 如果是LAN
        ip = append(net.IP{}, lanPrefix...)  // 将LAN前缀IP地址复制到IP变量
    }

    binary.LittleEndian.PutUint32(ip[12:], n)  // 将n转换为小端字节序并写入IP地址的第12个字节开始的位置
    addr, _ := ma.NewMultiaddr(fmt.Sprintf("/ip6/%s/tcp/4242", ip))  // 创建多地址，包含IP地址和端口
    return addr  // 返回创建的多地址
}

func RunDHTConnectivity(conf testutil.LatencyConfig, numPeers int) error {
    ctx, cancel := context.WithCancel(context.Background())  // 创建带有取消功能的上下文
    defer cancel()  // 延迟调用取消函数

    // create network
    mn := mocknet.New()  // 创建模拟网络
    # 设置模拟网络连接的默认参数，包括延迟和带宽
    mn.SetLinkDefaults(mocknet.LinkOptions{
        Latency:   conf.NetworkLatency,
        Bandwidth: math.MaxInt32,
    })

    # 创建一个新的节点作为测试节点，并设置为在线状态，使用模拟主机选项
    testPeer, err := core.NewNode(ctx, &core.BuildCfg{
        Online: true,
        Host:   mock.MockHostOption(mn),
    })
    if err != nil {
        return err
    }
    defer testPeer.Close()  # 延迟关闭测试节点

    # 初始化 WAN 和 LAN 节点的列表
    wanPeers := []*core.IpfsNode{}
    lanPeers := []*core.IpfsNode{}

    # 创建一个连接上下文，并设置超时时间为 15 秒
    connectionContext, connCtxCancel := context.WithTimeout(ctx, 15*time.Second)
    defer connCtxCancel()  # 延迟取消连接上下文

    # 循环创建指定数量的 WAN 和 LAN 节点
    for i := 0; i < numPeers; i++ {
        # 创建一个新的 WAN 节点，并设置为在线状态，使用 DHT 服务器选项和模拟主机选项
        wanPeer, err := core.NewNode(ctx, &core.BuildCfg{
            Online:  true,
            Routing: libp2p2.DHTServerOption,
            Host:    mock.MockHostOption(mn),
        })
        if err != nil {
            return err
        }
        defer wanPeer.Close()  # 延迟关闭 WAN 节点

        # 为 WAN 节点添加地址，并与已有的 WAN 节点建立连接
        wanAddr := makeAddr(uint32(i), true)
        wanPeer.Peerstore.AddAddr(wanPeer.Identity, wanAddr, peerstore.PermanentAddrTTL)
        for _, p := range wanPeers:
            _, _ = mn.LinkPeers(p.Identity, wanPeer.Identity)
            _ = wanPeer.PeerHost.Connect(connectionContext, p.Peerstore.PeerInfo(p.Identity))
        wanPeers = append(wanPeers, wanPeer)  # 将 WAN 节点添加到 WAN 节点列表中

        # 创建一个新的 LAN 节点，并设置为在线状态，使用模拟主机选项
        lanPeer, err := core.NewNode(ctx, &core.BuildCfg{
            Online: true,
            Host:   mock.MockHostOption(mn),
        })
        if err != nil {
            return err
        }
        defer lanPeer.Close()  # 延迟关闭 LAN 节点

        # 为 LAN 节点添加地址，并与已有的 LAN 节点建立连接
        lanAddr := makeAddr(uint32(i), false)
        lanPeer.Peerstore.AddAddr(lanPeer.Identity, lanAddr, peerstore.PermanentAddrTTL)
        for _, p := range lanPeers:
            _, _ = mn.LinkPeers(p.Identity, lanPeer.Identity)
            _ = lanPeer.PeerHost.Connect(connectionContext, p.Peerstore.PeerInfo(p.Identity))
        lanPeers = append(lanPeers, lanPeer)  # 将 LAN 节点添加到 LAN 节点列表中
    }
    connCtxCancel()  # 取消连接上下文

    # 为测试节点添加 WAN 地址
    wanAddr := makeAddr(0, true)
    # 将 WAN 地址添加到测试对等体的对等存储中，设置永久地址的生存时间
    testPeer.Peerstore.AddAddr(testPeer.Identity, wanAddr, peerstore.PermanentAddrTTL)
    # 创建一个局域网地址
    lanAddr := makeAddr(0, false)
    # 将局域网地址添加到测试对等体的对等存储中，设置永久地址的生存时间
    testPeer.Peerstore.AddAddr(testPeer.Identity, lanAddr, peerstore.PermanentAddrTTL)

    # 测试对等体连接到一个局域网对等体
    for _, p := range lanPeers:
        # 如果连接对等体出错，则返回错误
        if _, err := mn.LinkPeers(testPeer.Identity, p.Identity); err != nil:
            return err
    # 使用上下文和局域网对等体的对等信息连接测试对等体的对等主机
    err = testPeer.PeerHost.Connect(ctx, lanPeers[0].Peerstore.PeerInfo(lanPeers[0].Identity))
    if err != nil:
        return err

    # 创建一个具有60秒超时的启动上下文
    startupCtx, startupCancel := context.WithTimeout(ctx, time.Second*60)
# 循环等待直到满足条件
StartupWait:
    # 无限循环
    for {
        # 选择不同的 case 进行处理
        select {
        # 从 testPeer.DHT.LAN.RefreshRoutingTable() 通道接收错误信息
        case err := <-testPeer.DHT.LAN.RefreshRoutingTable():
            # 如果接收到错误信息，则打印错误信息
            if err != nil:
                fmt.Printf("Error refreshing routing table: %v\n", err)
            # 如果 LAN 路由表为空或者出现错误，则等待 100 毫秒后继续循环
            if testPeer.DHT.LAN.RoutingTable() == nil ||
                testPeer.DHT.LAN.RoutingTable().Size() == 0 ||
                err != nil:
                time.Sleep(100 * time.Millisecond)
                continue
            # 跳出循环
            break StartupWait
        # 如果 startupCtx 被取消，则执行相应操作
        case <-startupCtx.Done():
            startupCancel()
            return fmt.Errorf("expected faster dht bootstrap")
        }
    }
    # 取消启动操作
    startupCancel()

    # 选择一个 LAN 节点并验证 LAN DHT 是否正常运行
    i := rand.Intn(len(lanPeers))
    # 如果 testPeer 与 lanPeers[i] 相连，则选择下一个节点
    if testPeer.PeerHost.Network().Connectedness(lanPeers[i].Identity) == corenet.Connected:
        i = (i + 1) % len(lanPeers)
        # 如果 testPeer 与 lanPeers[i] 相连，则关闭连接并清除地址
        if testPeer.PeerHost.Network().Connectedness(lanPeers[i].Identity) == corenet.Connected:
            _ = testPeer.PeerHost.Network().ClosePeer(lanPeers[i].Identity)
            testPeer.PeerHost.Peerstore().ClearAddrs(lanPeers[i].Identity)
    # 创建一个新的 CID，并验证测试节点是否能找到它
    provideCid := cid.NewCidV1(cid.Raw, []byte("Lan Provide Record"))
    provideCtx, cancel := context.WithTimeout(ctx, time.Second)
    defer cancel()
    # 如果 lanPeers[i] 提供 CID 失败，则返回错误
    if err := lanPeers[i].DHT.Provide(provideCtx, provideCid, true); err != nil:
        return err
    # 异步查找提供者
    provChan := testPeer.DHT.FindProvidersAsync(provideCtx, provideCid, 0)
    prov, ok := <-provChan
    # 如果通道关闭或者未找到提供者，则返回错误
    if !ok || prov.ID == "":
        return fmt.Errorf("Expected provider. stream closed early")
    # 如果找到的提供者不是 lanPeers[i]，则返回错误
    if prov.ID != lanPeers[i].Identity:
        return fmt.Errorf("Unexpected lan peer provided record")

    # 现在，与 WAN 节点建立连接
    for _, p := range wanPeers:
        # 如果连接失败，则返回错误
        if _, err := mn.LinkPeers(testPeer.Identity, p.Identity); err != nil:
            return err
    }
    # 使用 testPeer.PeerHost.Connect 方法连接到指定的对等节点
    err = testPeer.PeerHost.Connect(ctx, wanPeers[0].Peerstore.PeerInfo(wanPeers[0].Identity))
    # 如果连接过程中出现错误
    if err != nil:
        # 返回错误
        return err

    # 使用 context.WithTimeout 方法创建一个带有超时的上下文
    startupCtx, startupCancel = context.WithTimeout(ctx, time.Second*60)
# 循环等待WAN启动
WanStartupWait:
    # 进入无限循环
    for {
        # 选择一个case进行处理
        select {
        # 从testPeer.DHT.WAN.RefreshRoutingTable()通道接收错误信息
        case err := <-testPeer.DHT.WAN.RefreshRoutingTable():
            # 如果出现错误
            // if err != nil {
            //    fmt.Printf("Error refreshing routing table: %v\n", err)
            // }
            # 如果WAN的路由表为空或者出现错误
            if testPeer.DHT.WAN.RoutingTable() == nil ||
                testPeer.DHT.WAN.RoutingTable().Size() == 0 ||
                err != nil {
                # 等待100毫秒
                time.Sleep(100 * time.Millisecond)
                # 继续下一次循环
                continue
            }
            # 跳出循环
            break WanStartupWait
        # 如果startupCtx.Done()通道被关闭
        case <-startupCtx.Done():
            # 取消启动
            startupCancel()
            # 返回错误信息
            return fmt.Errorf("expected faster wan dht bootstrap")
        }
    }
    # 取消启动
    startupCancel()

    # 选择一个WAN节点并验证WAN DHT是否正常运行
    i = rand.Intn(len(wanPeers))
    # 如果testPeer与wanPeers[i]相连
    if testPeer.PeerHost.Network().Connectedness(wanPeers[i].Identity) == corenet.Connected:
        # 选择下一个wanPeers
        i = (i + 1) % len(wanPeers)
        # 如果testPeer与wanPeers[i]相连
        if testPeer.PeerHost.Network().Connectedness(wanPeers[i].Identity) == corenet.Connected:
            # 关闭wanPeers[i]的连接
            _ = testPeer.PeerHost.Network().ClosePeer(wanPeers[i].Identity)
            # 清除wanPeers[i]的地址
            testPeer.PeerHost.Peerstore().ClearAddrs(wanPeers[i].Identity)
    }

    # WAN节点将提供一个新的CID，并验证测试节点是否可以找到它
    wanCid := cid.NewCidV1(cid.Raw, []byte("Wan Provide Record"))
    # 创建一个带有超时的上下文
    wanProvideCtx, cancel := context.WithTimeout(ctx, time.Second)
    # 延迟取消上下文
    defer cancel()
    # 如果wanPeers[i]提供CID失败
    if err := wanPeers[i].DHT.Provide(wanProvideCtx, wanCid, true); err != nil {
        # 返回错误信息
        return err
    }
    # 异步查找提供者
    provChan = testPeer.DHT.FindProvidersAsync(wanProvideCtx, wanCid, 0)
    # 从通道中接收提供者信息
    prov, ok = <-provChan
    # 如果通道关闭或者prov.ID为空
    if !ok || prov.ID == "" {
        # 返回错误信息
        return fmt.Errorf("Expected one provider, closed early")
    }
    # 如果提供者ID不是wanPeers[i]的身份
    if prov.ID != wanPeers[i].Identity {
        # 返回错误信息
        return fmt.Errorf("Unexpected lan peer provided record")
    }

    # 最后，从wanPeers中重新分享lan提供的CID，并期望得到合并的结果
    i = rand.Intn(len(wanPeers))
    # 检查测试节点与 WAN 节点的连接状态，如果已连接，则关闭连接并清除地址
    if testPeer.PeerHost.Network().Connectedness(wanPeers[i].Identity) == corenet.Connected:
        _ = testPeer.PeerHost.Network().ClosePeer(wanPeers[i].Identity)
        testPeer.PeerHost.Peerstore().ClearAddrs(wanPeers[i].Identity)

    # 创建一个带有超时的上下文
    provideCtx, cancel = context.WithTimeout(ctx, time.Second)
    # 延迟取消上下文
    defer cancel()
    # 使用 WAN 节点的 DHT 服务提供数据
    if err := wanPeers[i].DHT.Provide(provideCtx, provideCid, true); err != nil:
        return err
    # 异步查找提供者
    provChan = testPeer.DHT.FindProvidersAsync(provideCtx, provideCid, 0)
    # 从通道中获取提供者信息
    prov, ok = <-provChan
    # 如果通道关闭，则返回错误
    if !ok:
        return fmt.Errorf("Expected two providers, got 0")
    # 再次从通道中获取提供者信息
    prov, ok = <-provChan
    # 如果通道关闭，则返回错误
    if !ok:
        return fmt.Errorf("Expected two providers, got 1")

    # 没有错误发生，返回空
    return nil
# 闭合前面的函数定义
```