# `kubo\test\cli\content_routing_http_test.go`

```
package cli

import (
    "context"  // 导入上下文包，用于控制函数调用的上下文
    "net/http"  // 导入网络包，用于创建 HTTP 客户端和服务器
    "net/http/httptest"  // 导入网络包，用于创建 HTTP 服务器的测试
    "os/exec"  // 导入操作系统执行包，用于执行操作系统命令
    "sync"  // 导入同步包，用于实现并发安全的数据结构
    "testing"  // 导入测试包，用于编写测试函数
    "time"  // 导入时间包，用于处理时间相关的操作

    "github.com/ipfs/boxo/ipns"  // 导入 IPNS 包
    "github.com/ipfs/boxo/routing/http/server"  // 导入 HTTP 服务器包
    "github.com/ipfs/boxo/routing/http/types"  // 导入 HTTP 类型包
    "github.com/ipfs/boxo/routing/http/types/iter"  // 导入迭代器包
    "github.com/ipfs/go-cid"  // 导入 CID 包
    "github.com/ipfs/kubo/test/cli/harness"  // 导入测试工具包
    "github.com/ipfs/kubo/test/cli/testutils"  // 导入测试工具包
    "github.com/libp2p/go-libp2p/core/peer"  // 导入 libp2p 核心包
    "github.com/libp2p/go-libp2p/core/routing"  // 导入 libp2p 核心包
    "github.com/stretchr/testify/assert"  // 导入断言包
)

type fakeHTTPContentRouter struct {
    m                   sync.Mutex  // 创建互斥锁
    provideBitswapCalls int  // 提供 Bitswap 调用次数
    findProvidersCalls  int  // 查找提供者调用次数
    findPeersCalls      int  // 查找对等体调用次数
}

func (r *fakeHTTPContentRouter) FindProviders(ctx context.Context, key cid.Cid, limit int) (iter.ResultIter[types.Record], error) {
    r.m.Lock()  // 加锁
    defer r.m.Unlock()  // 延迟解锁
    r.findProvidersCalls++  // 增加查找提供者调用次数
    return iter.FromSlice([]iter.Result[types.Record]{}), nil  // 返回空的结果迭代器
}

// nolint deprecated
func (r *fakeHTTPContentRouter) ProvideBitswap(ctx context.Context, req *server.BitswapWriteProvideRequest) (time.Duration, error) {
    r.m.Lock()  // 加锁
    defer r.m.Unlock()  // 延迟解锁
    r.provideBitswapCalls++  // 增加提供 Bitswap 调用次数
    return 0, nil  // 返回时间间隔和空错误
}

func (r *fakeHTTPContentRouter) FindPeers(ctx context.Context, pid peer.ID, limit int) (iter.ResultIter[*types.PeerRecord], error) {
    r.m.Lock()  // 加锁
    defer r.m.Unlock()  // 延迟解锁
    r.findPeersCalls++  // 增加查找对等体调用次数
    return iter.FromSlice([]iter.Result[*types.PeerRecord]{}), nil  // 返回空的结果迭代器
}

func (r *fakeHTTPContentRouter) GetIPNS(ctx context.Context, name ipns.Name) (*ipns.Record, error) {
    return nil, routing.ErrNotSupported  // 返回空和不支持的路由错误
}

func (r *fakeHTTPContentRouter) PutIPNS(ctx context.Context, name ipns.Name, rec *ipns.Record) error {
    return routing.ErrNotSupported  // 返回不支持的路由错误
}

func (r *fakeHTTPContentRouter) numFindProvidersCalls() int {
    r.m.Lock()  // 加锁
    defer r.m.Unlock()  // 延迟解锁
    return r.findProvidersCalls  // 返回查找提供者调用次数
}

// userAgentRecorder records the user agent of every HTTP request
type userAgentRecorder struct {
    # 定义一个名为 delegate 的变量，类型为 http.Handler
    delegate   http.Handler
    # 定义一个名为 userAgents 的变量，类型为字符串数组
    userAgents []string
}

// 实现 ServeHTTP 方法，记录用户代理信息并调用委托的 ServeHTTP 方法
func (r *userAgentRecorder) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    // 将请求的用户代理信息追加到用户代理记录列表中
    r.userAgents = append(r.userAgents, req.UserAgent())
    // 调用委托的 ServeHTTP 方法处理请求
    r.delegate.ServeHTTP(w, req)
}

// 测试 ContentRoutingHTTP
func TestContentRoutingHTTP(t *testing.T) {
    // 创建一个 fakeHTTPContentRouter 对象
    cr := &fakeHTTPContentRouter{}

    // 运行 content routing HTTP 服务器
    userAgentRecorder := &userAgentRecorder{delegate: server.Handler(cr)}
    server := httptest.NewServer(userAgentRecorder)
    // 在测试结束时关闭服务器
    t.Cleanup(func() { server.Close() })

    // 设置节点
    node := harness.NewT(t).NewNode().Init()
    node.Runner.Env["IPFS_HTTP_ROUTERS"] = server.URL
    node.StartDaemon()

    // 生成一个随机的 CID
    randStr := string(testutils.RandomBytes(100))
    res := node.PipeStrToIPFS(randStr, "add", "-qn")
    wantCIDStr := res.Stdout.Trimmed()

    // 运行测试用例：获取未缓存的块会导致 HTTP 查找
    t.Run("fetching an uncached block results in an HTTP lookup", func(t *testing.T) {
        statRes := node.Runner.Run(harness.RunRequest{
            Path:    node.IPFSBin,
            Args:    []string{"block", "stat", wantCIDStr},
            RunFunc: (*exec.Cmd).Start,
        })
        defer func() {
            if err := statRes.Cmd.Process.Kill(); err != nil {
                t.Logf("error killing 'block stat' cmd: %s", err)
            }
        }()

        // 验证内容路由器是否被调用
        assert.Eventually(t, func() bool {
            return cr.numFindProvidersCalls() > 0
        }, time.Minute, 10*time.Millisecond)

        // 验证用户代理记录不为空
        assert.NotEmpty(t, userAgentRecorder.userAgents)
        // 获取节点的版本信息
        version := node.IPFS("id", "-f", "<aver>").Stdout.Trimmed()
        // 遍历用户代理记录，验证用户代理信息与节点版本一致
        for _, userAgent := range userAgentRecorder.userAgents {
            assert.Equal(t, version, userAgent)
        }
    })
}
```