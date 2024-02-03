# `kubo\core\coreiface\tests\routing.go`

```go
package tests

import (
    "context"  // 导入上下文包，用于处理请求的取消、超时等
    "testing"  // 导入测试包，用于编写测试函数
    "time"     // 导入时间包，用于处理时间相关操作

    "github.com/ipfs/boxo/ipns"  // 导入IPNS包，用于处理IPNS相关操作
    "github.com/ipfs/boxo/path"  // 导入路径包，用于处理路径相关操作
    iface "github.com/ipfs/kubo/core/coreiface"  // 导入接口包，用于定义接口
    "github.com/ipfs/kubo/core/coreiface/options"  // 导入选项包，用于处理选项相关操作
    "github.com/stretchr/testify/require"  // 导入断言包，用于编写测试断言
)

func (tp *TestSuite) TestRouting(t *testing.T) {
    tp.hasApi(t, func(api iface.CoreAPI) error {  // 调用hasApi方法，传入测试对象和回调函数
        if api.Routing() == nil {  // 判断API对象的Routing方法是否为空
            return errAPINotImplemented  // 如果为空，返回未实现错误
        }
        return nil  // 如果不为空，返回空
    })

    t.Run("TestRoutingGet", tp.TestRoutingGet)  // 运行TestRoutingGet测试函数
    t.Run("TestRoutingPut", tp.TestRoutingPut)  // 运行TestRoutingPut测试函数
    t.Run("TestRoutingPutOffline", tp.TestRoutingPutOffline)  // 运行TestRoutingPutOffline测试函数
}

func (tp *TestSuite) testRoutingPublishKey(t *testing.T, ctx context.Context, api iface.CoreAPI, opts ...options.NamePublishOption) (path.Path, ipns.Name) {
    p, err := addTestObject(ctx, api)  // 调用addTestObject方法，传入上下文和API对象
    require.NoError(t, err)  // 断言错误为空

    name, err := api.Name().Publish(ctx, p, opts...)  // 调用API对象的Publish方法，传入上下文、路径和选项
    require.NoError(t, err)  // 断言错误为空

    time.Sleep(3 * time.Second)  // 休眠3秒
    return p, name  // 返回路径和名称
}

func (tp *TestSuite) TestRoutingGet(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())  // 创建上下文和取消函数
    defer cancel()  // 延迟调用取消函数

    apis, err := tp.MakeAPISwarm(t, ctx, 2)  // 调用MakeAPISwarm方法，传入测试对象、上下文和数量
    require.NoError(t, err)  // 断言错误为空

    // Node 1: publishes an IPNS name
    p, name := tp.testRoutingPublishKey(t, ctx, apis[0])  // 调用testRoutingPublishKey方法，传入测试对象、上下文和API对象

    // Node 2: retrieves the best value for the IPNS name.
    data, err := apis[1].Routing().Get(ctx, ipns.NamespacePrefix+name.String())  // 调用API对象的Routing方法的Get方法，传入上下文和IPNS名称
    require.NoError(t, err)  // 断言错误为空

    rec, err := ipns.UnmarshalRecord(data)  // 调用UnmarshalRecord方法，传入数据
    require.NoError(t, err)  // 断言错误为空

    val, err := rec.Value()  // 调用Value方法
    require.NoError(t, err)  // 断言错误为空
    require.Equal(t, p.String(), val.String())  // 断言相等
}

func (tp *TestSuite) TestRoutingPut(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())  // 创建上下文和取消函数
    defer cancel()  // 延迟调用取消函数
    apis, err := tp.MakeAPISwarm(t, ctx, 2)  // 调用MakeAPISwarm方法，传入测试对象、上下文和数量
    require.NoError(t, err)  // 断言错误为空

    // Create and publish IPNS entry.
    _, name := tp.testRoutingPublishKey(t, ctx, apis[0])  // 调用testRoutingPublishKey方法，传入测试对象、上下文和API对象

    // Get valid routing value.
    // 从第一个API获取数据，使用IPNS命名空间前缀和名称字符串构建路由路径，返回数据和错误
    data, err := apis[0].Routing().Get(ctx, ipns.NamespacePrefix+name.String())
    // 确保没有错误发生
    require.NoError(t, err)

    // 将获取到的数据放入路由中
    err = apis[1].Routing().Put(ctx, ipns.NamespacePrefix+name.String(), data)
    // 确保没有错误发生
    require.NoError(t, err)
// 定义一个测试函数，用于测试离线情况下的路由PUT操作
func (tp *TestSuite) TestRoutingPutOffline(t *testing.T) {
    // 创建一个上下文，并在函数返回时取消
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    // 初始化一个swarm，并发布一个IPNS条目以获取有效的有效负载
    apis, err := tp.MakeAPISwarm(t, ctx, 2)
    require.NoError(t, err)

    // 获取IPNS条目的名称
    _, name := tp.testRoutingPublishKey(t, ctx, apis[0], options.Name.AllowOffline(true))
    // 从路由中获取数据
    data, err := apis[0].Routing().Get(ctx, ipns.NamespacePrefix+name.String())
    require.NoError(t, err)

    // 初始化我们的离线节点，并尝试放置有效负载
    api, err := tp.makeAPIWithIdentityAndOffline(t, ctx)
    require.NoError(t, err)

    // 尝试在离线状态下执行路由PUT操作，预期会失败
    err = api.Routing().Put(ctx, ipns.NamespacePrefix+name.String(), data)
    require.Error(t, err, "this operation should fail because we are offline")

    // 尝试在离线状态下执行路由PUT操作，预期会成功
    err = api.Routing().Put(ctx, ipns.NamespacePrefix+name.String(), data, options.Put.AllowOffline(true))
    require.NoError(t, err)
}
```