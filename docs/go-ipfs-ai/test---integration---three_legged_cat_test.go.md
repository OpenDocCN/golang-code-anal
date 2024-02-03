# `kubo\test\integration\three_legged_cat_test.go`

```go
// 定义一个名为 integrationtest 的包
package integrationtest

// 导入所需的包
import (
    "bytes"
    "context"
    "errors"
    "io"
    "math"
    "testing"
    "time"

    bootstrap2 "github.com/ipfs/boxo/bootstrap"
    "github.com/ipfs/kubo/core/coreapi"
    mock "github.com/ipfs/kubo/core/mock"
    "github.com/ipfs/kubo/thirdparty/unit"

    "github.com/ipfs/boxo/files"
    testutil "github.com/libp2p/go-libp2p-testing/net"
    "github.com/libp2p/go-libp2p/core/peer"
    mocknet "github.com/libp2p/go-libp2p/p2p/net/mock"
)

// 定义名为 TestThreeLeggedCatTransfer 的测试函数
func TestThreeLeggedCatTransfer(t *testing.T) {
    // 创建一个测试配置对象
    conf := testutil.LatencyConfig{
        NetworkLatency:    0,
        RoutingLatency:    0,
        BlockstoreLatency: 0,
    }
    // 如果运行 RunThreeLeggedCat 函数出现错误，则标记测试失败
    if err := RunThreeLeggedCat(RandomBytes(100*unit.MB), conf); err != nil {
        t.Fatal(err)
    }
}

// 定义名为 TestThreeLeggedCatDegenerateSlowBlockstore 的测试函数
func TestThreeLeggedCatDegenerateSlowBlockstore(t *testing.T) {
    // 跳过测试，除非是史诗级测试
    SkipUnlessEpic(t)
    // 创建一个测试配置对象
    conf := testutil.LatencyConfig{BlockstoreLatency: 50 * time.Millisecond}
    // 如果运行 RunThreeLeggedCat 函数出现错误，则标记测试失败
    if err := RunThreeLeggedCat(RandomBytes(1*unit.KB), conf); err != nil {
        t.Fatal(err)
    }
}

// 定义名为 TestThreeLeggedCatDegenerateSlowNetwork 的测试函数
func TestThreeLeggedCatDegenerateSlowNetwork(t *testing.T) {
    // 跳过测试，除非是史诗级测试
    SkipUnlessEpic(t)
    // 创建一个测试配置对象
    conf := testutil.LatencyConfig{NetworkLatency: 400 * time.Millisecond}
    // 如果运行 RunThreeLeggedCat 函数出现错误，则标记测试失败
    if err := RunThreeLeggedCat(RandomBytes(1*unit.KB), conf); err != nil {
        t.Fatal(err)
    }
}

// 定义名为 TestThreeLeggedCatDegenerateSlowRouting 的测试函数
func TestThreeLeggedCatDegenerateSlowRouting(t *testing.T) {
    // 跳过测试，除非是史诗级测试
    SkipUnlessEpic(t)
    // 创建一个测试配置对象
    conf := testutil.LatencyConfig{RoutingLatency: 400 * time.Millisecond}
    // 如果运行 RunThreeLeggedCat 函数出现错误，则标记测试失败
    if err := RunThreeLeggedCat(RandomBytes(1*unit.KB), conf); err != nil {
        t.Fatal(err)
    }
}

// 定义名为 TestThreeLeggedCat100MBMacbookCoastToCoast 的测试函数
func TestThreeLeggedCat100MBMacbookCoastToCoast(t *testing.T) {
    // 跳过测试，除非是史诗级测试
    SkipUnlessEpic(t)
    // 创建一个测试配置对象，设置网络延迟、块存储延迟和路由延迟
    conf := testutil.LatencyConfig{}.NetworkNYtoSF().BlockstoreSlowSSD2014().RoutingSlow()
    // 如果运行 RunThreeLeggedCat 函数出现错误，则标记测试失败
    if err := RunThreeLeggedCat(RandomBytes(100*unit.MB), conf); err != nil {
        t.Fatal(err)
    }
}

// 定义名为 RunThreeLeggedCat 的函数，接受数据和测试配置作为参数
func RunThreeLeggedCat(data []byte, conf testutil.LatencyConfig) error {
    // 创建一个上下文对象和取消函数
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()

    // 创建网络
    mn := mocknet.New()
    mn.SetLinkDefaults(mocknet.LinkOptions{
        Latency: conf.NetworkLatency,  // 设置网络延迟
        // TODO 添加到 conf 中。这很棘手，因为我们希望 0 值也能正常工作。
        Bandwidth: math.MaxInt32,  // 设置网络带宽
    })

    // 创建模拟的公共节点 bootstrap
    bootstrap, err := mock.MockPublicNode(ctx, mn)
    if err != nil {
        return err
    }
    // 延迟关闭 bootstrap
    defer bootstrap.Close()

    // 创建模拟的公共节点 adder
    adder, err := mock.MockPublicNode(ctx, mn)
    if err != nil {
        return err
    }
    // 延迟关闭 adder
    defer adder.Close()

    // 创建模拟的公共节点 catter
    catter, err := mock.MockPublicNode(ctx, mn)
    if err != nil {
        return err
    }
    // 延迟关闭 catter
    defer catter.Close()

    // 创建 adder 的 CoreAPI
    adderAPI, err := coreapi.NewCoreAPI(adder)
    if err != nil {
        return err
    }

    // 创建 catter 的 CoreAPI
    catterAPI, err := coreapi.NewCoreAPI(catter)
    if err != nil {
        return err
    }

    // 将所有节点连接起来
    err = mn.LinkAll()
    if err != nil {
        return err
    }

    // 获取 bootstrap 的 PeerInfo
    bis := bootstrap.Peerstore.PeerInfo(bootstrap.PeerHost.ID())
    // 使用 PeerInfo 创建 BootstrapConfig
    bcfg := bootstrap2.BootstrapConfigWithPeers([]peer.AddrInfo{bis})
    // 尝试让 adder 进行引导
    if err := adder.Bootstrap(bcfg); err != nil {
        return err
    }
    // 尝试让 catter 进行引导
    if err := catter.Bootstrap(bcfg); err != nil {
        return err
    }

    // 将数据添加到 adder 节点
    added, err := adderAPI.Unixfs().Add(ctx, files.NewBytesFile(data))
    if err != nil {
        return err
    }

    // 从 catter 节点获取添加的数据
    readerCatted, err := catterAPI.Unixfs().Get(ctx, added)
    if err != nil {
        return err
    }

    // 验证数据
    var bufout bytes.Buffer
    _, err = io.Copy(&bufout, readerCatted.(io.Reader))
    if err != nil {
        return err
    }
    // 检查 catter 获取的数据是否与添加的数据匹配
    if !bytes.Equal(bufout.Bytes(), data) {
        return errors.New("catted data does not match added data")
    }
    return nil
# 闭合前面的函数定义
```