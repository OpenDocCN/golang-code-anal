# `kubo\core\coreiface\tests\dht.go`

```go
// 导入所需的包
package tests

import (
    "context"
    "io"
    "testing"
    "time"

    iface "github.com/ipfs/kubo/core/coreiface"
    "github.com/ipfs/kubo/core/coreiface/options"
)

// 测试 Dht 方法
func (tp *TestSuite) TestDht(t *testing.T) {
    // 检查 API 是否存在
    tp.hasApi(t, func(api iface.CoreAPI) error {
        if api.Dht() == nil {
            return errAPINotImplemented
        }
        return nil
    })

    // 分别运行三个测试用例
    t.Run("TestDhtFindPeer", tp.TestDhtFindPeer)
    t.Run("TestDhtFindProviders", tp.TestDhtFindProviders)
    t.Run("TestDhtProvide", tp.TestDhtProvide)
}

// 测试 DhtFindPeer 方法
func (tp *TestSuite) TestDhtFindPeer(t *testing.T) {
    // 创建上下文和取消函数
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    // 创建 API Swarm
    apis, err := tp.MakeAPISwarm(t, ctx, 5)
    if err != nil {
        t.Fatal(err)
    }

    // 获取自身信息
    self0, err := apis[0].Key().Self(ctx)
    if err != nil {
        t.Fatal(err)
    }

    // 获取本地地址
    laddrs0, err := apis[0].Swarm().LocalAddrs(ctx)
    if err != nil {
        t.Fatal(err)
    }
    if len(laddrs0) != 1 {
        t.Fatal("unexpected number of local addrs")
    }

    // 等待3秒
    time.Sleep(3 * time.Second)

    // 查找对等节点
    pi, err := apis[2].Dht().FindPeer(ctx, self0.ID())
    if err != nil {
        t.Fatal(err)
    }

    // 检查查找到的地址是否与本地地址一致
    if pi.Addrs[0].String() != laddrs0[0].String() {
        t.Errorf("got unexpected address from FindPeer: %s", pi.Addrs[0].String())
    }

    // 获取自身信息
    self2, err := apis[2].Key().Self(ctx)
    if err != nil {
        t.Fatal(err)
    }

    // 再次查找对等节点
    pi, err = apis[1].Dht().FindPeer(ctx, self2.ID())
    if err != nil {
        t.Fatal(err)
    }

    // 获取本地地址
    laddrs2, err := apis[2].Swarm().LocalAddrs(ctx)
    if err != nil {
        t.Fatal(err)
    }
    if len(laddrs2) != 1 {
        t.Fatal("unexpected number of local addrs")
    }

    // 检查查找到的地址是否与本地地址一致
    if pi.Addrs[0].String() != laddrs2[0].String() {
        t.Errorf("got unexpected address from FindPeer: %s", pi.Addrs[0].String())
    }
}

// 测试 DhtFindProviders 方法
func (tp *TestSuite) TestDhtFindProviders(t *testing.T) {
    // 创建上下文和取消函数
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    # 使用 MakeAPISwarm 创建一个包含 5 个节点的 API 群集，并返回错误信息
    apis, err := tp.MakeAPISwarm(t, ctx, 5)
    # 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    # 在第一个节点上添加测试对象，并返回错误信息
    p, err := addTestObject(ctx, apis[0])
    # 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    # 等待 3 秒
    time.Sleep(3 * time.Second)

    # 在第三个节点上使用 Dht().FindProviders 查找提供者，并返回结果和错误信息
    out, err := apis[2].Dht().FindProviders(ctx, p, options.Dht.NumProviders(1))
    # 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    # 从结果中获取提供者
    provider := <-out

    # 在第一个节点上获取自身信息，并返回结果和错误信息
    self0, err := apis[0].Key().Self(ctx)
    # 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    # 检查提供者的 ID 是否与第一个节点的 ID 相同，如果不同则输出错误信息
    if provider.ID.String() != self0.ID().String() {
        t.Errorf("got wrong provider: %s != %s", provider.ID.String(), self0.ID().String())
    }
// 定义一个名为 TestDhtProvide 的方法，接收一个指向 TestSuite 结构体的指针和一个 *testing.T 类型的参数
func (tp *TestSuite) TestDhtProvide(t *testing.T) {
    // 创建一个带有取消函数的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 使用 MakeAPISwarm 方法创建一组 API，数量为 5
    apis, err := tp.MakeAPISwarm(t, ctx, 5)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 将 apis[0] 设置为离线模式
    off0, err := apis[0].WithOptions(options.Api.Offline(true))
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 将一个大小为 4092 的随机数据块放入块存储，并返回其路径
    s, err := off0.Block().Put(ctx, &io.LimitedReader{R: rnd, N: 4092})
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 获取数据块的路径
    p := s.Path()

    // 等待 3 秒
    time.Sleep(3 * time.Second)

    // 在 apis[2] 中查找提供者，限制提供者数量为 1
    out, err := apis[2].Dht().FindProviders(ctx, p, options.Dht.NumProviders(1))
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 从通道中读取值并判断是否存在提供者
    _, ok := <-out

    // 如果存在提供者，记录错误并终止测试
    if ok {
        t.Fatal("did not expect to find any providers")
    }

    // 获取 apis[0] 的节点 ID
    self0, err := apis[0].Key().Self(ctx)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 将数据块提供给网络
    err = apis[0].Dht().Provide(ctx, p)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 再次在 apis[2] 中查找提供者，限制提供者数量为 1
    out, err = apis[2].Dht().FindProviders(ctx, p, options.Dht.NumProviders(1))
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 从通道中读取提供者信息
    provider := <-out

    // 检查提供者的节点 ID 是否与 apis[0] 的节点 ID 相同，如果不同则记录错误
    if provider.ID.String() != self0.ID().String() {
        t.Errorf("got wrong provider: %s != %s", provider.ID.String(), self0.ID().String())
    }
}
```