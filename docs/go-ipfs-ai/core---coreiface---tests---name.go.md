# `kubo\core\coreiface\tests\name.go`

```
package tests

import (
    "context"  // 上下文包，用于控制函数调用的生命周期
    "io"  // 输入输出包，提供了基本的输入输出功能
    "math/rand"  // 随机数包，用于生成随机数
    "testing"  // 测试包，用于编写和执行测试程序
    "time"  // 时间包，提供了时间的显示和测量

    "github.com/ipfs/boxo/files"  // 文件包，用于处理文件相关操作
    "github.com/ipfs/boxo/ipns"  // IPNS包，用于处理IPNS相关操作
    "github.com/ipfs/boxo/path"  // 路径包，用于处理路径相关操作
    coreiface "github.com/ipfs/kubo/core/coreiface"  // 核心接口包，定义了IPFS核心接口
    opt "github.com/ipfs/kubo/core/coreiface/options"  // 选项包，用于定义IPFS核心接口的选项
    "github.com/stretchr/testify/require"  // 断言包，用于编写测试断言
)

func (tp *TestSuite) TestName(t *testing.T) {
    tp.hasApi(t, func(api coreiface.CoreAPI) error {
        if api.Name() == nil {
            return errAPINotImplemented  // 如果API的名称为空，则返回未实现错误
        }
        return nil
    })

    t.Run("TestPublishResolve", tp.TestPublishResolve)  // 运行测试发布和解析
    t.Run("TestBasicPublishResolveKey", tp.TestBasicPublishResolveKey)  // 运行基本发布和解析密钥测试
    t.Run("TestBasicPublishResolveTimeout", tp.TestBasicPublishResolveTimeout)  // 运行基本发布和解析超时测试
}

var rnd = rand.New(rand.NewSource(0x62796532303137))  // 生成随机数生成器

func addTestObject(ctx context.Context, api coreiface.CoreAPI) (path.Path, error) {
    return api.Unixfs().Add(ctx, files.NewReaderFile(&io.LimitedReader{R: rnd, N: 4092}))  // 向IPFS添加测试对象
}

func (tp *TestSuite) TestPublishResolve(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())  // 创建一个带有取消功能的上下文
    defer cancel()  // 延迟调用取消上下文的函数
    init := func() (coreiface.CoreAPI, path.Path) {
        apis, err := tp.MakeAPISwarm(t, ctx, 5)  // 创建API群集
        require.NoError(t, err)  // 断言没有错误发生
        api := apis[0]  // 获取第一个API

        p, err := addTestObject(ctx, api)  // 向IPFS添加测试对象
        require.NoError(t, err)  // 断言没有错误发生
        return api, p  // 返回API和路径
    }
    }

    t.Run("default", func(t *testing.T) {
        run(t, []opt.NameResolveOption{})  // 运行默认名称解析选项
    })

    t.Run("nocache", func(t *testing.T) {
        run(t, []opt.NameResolveOption{opt.Name.Cache(false)})  // 运行禁用缓存的名称解析选项
    })
}

func (tp *TestSuite) TestBasicPublishResolveKey(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())  // 创建一个带有取消功能的上下文
    defer cancel()  // 延迟调用取消上下文的函数
    apis, err := tp.MakeAPISwarm(t, ctx, 5)  // 创建API群集
    require.NoError(t, err)  // 断言没有错误发生
    api := apis[0]  // 获取第一个API

    k, err := api.Key().Generate(ctx, "foo")  // 生成一个新的密钥
    require.NoError(t, err)  // 断言没有错误发生

    p, err := addTestObject(ctx, api)  // 向IPFS添加测试对象
    require.NoError(t, err)  // 断言没有错误发生
}
    # 调用 api 的 Name 方法，使用上下文和选项发布内容，并返回发布的名称和错误
    name, err := api.Name().Publish(ctx, p, opt.Name.Key(k.Name()))
    # 断言错误为空
    require.NoError(t, err)
    # 断言发布的名称与根据对等点 ID 生成的 IPNS 名称相等
    require.Equal(t, name.String(), ipns.NameFromPeer(k.ID()).String())

    # 调用 api 的 Name 方法，使用上下文和发布的名称解析内容，并返回解析后的路径和错误
    resPath, err := api.Name().Resolve(ctx, name.String())
    # 断言错误为空
    require.NoError(t, err)
    # 断言解析后的路径与原始内容的对等点 ID 字符串相等
    require.Equal(t, p.String(), resPath.String())
// 定义一个名为TestBasicPublishResolveTimeout的测试函数，用于测试基本的发布和解析超时
func (tp *TestSuite) TestBasicPublishResolveTimeout(t *testing.T) {
    // 跳过测试，因为在这个时间分辨率下ValidTime似乎不起作用
    t.Skip("ValidTime doesn't appear to work at this time resolution")

    // 创建一个带有取消函数的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 使用tp对象创建一个包含5个节点的API群集
    apis, err := tp.MakeAPISwarm(t, ctx, 5)
    require.NoError(t, err)
    // 获取第一个API节点
    api := apis[0]
    // 向API节点添加一个测试对象
    p, err := addTestObject(ctx, api)
    require.NoError(t, err)

    // 获取API节点的自身信息
    self, err := api.Key().Self(ctx)
    require.NoError(t, err)

    // 发布对象的名称，并设置有效时间为100毫秒
    name, err := api.Name().Publish(ctx, p, opt.Name.ValidTime(time.Millisecond*100))
    require.NoError(t, err)
    // 断言发布的名称与API节点的ID相关的IPNS名称相等
    require.Equal(t, name.String(), ipns.NameFromPeer(self.ID()).String())

    // 等待1秒
    time.Sleep(time.Second)

    // 解析发布的名称
    _, err = api.Name().Resolve(ctx, name.String())
    require.NoError(t, err)
}

// 待办事项：创建swarm api后，添加多节点测试
// TODO: When swarm api is created, add multinode tests
```