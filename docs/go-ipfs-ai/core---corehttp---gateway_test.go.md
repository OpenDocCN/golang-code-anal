# `kubo\core\corehttp\gateway_test.go`

```go
package corehttp

import (
    "context"  // 上下文包，用于控制请求的取消、超时等
    "errors"   // 错误处理包，用于创建和处理错误
    "io"       // 输入输出包，提供了基本的输入输出功能
    "net/http" // HTTP 客户端和服务端包，用于创建和处理 HTTP 请求和响应
    "net/http/httptest" // HTTP 测试包，用于创建 HTTP 服务器的测试实例
    "strings"  // 字符串处理包，提供了对字符串的基本操作
    "testing"  // 测试包，用于编写和运行测试用例

    "github.com/ipfs/boxo/namesys"  // 导入外部包
    version "github.com/ipfs/kubo"  // 导入外部包
    "github.com/ipfs/kubo/core"     // 导入外部包
    "github.com/ipfs/kubo/core/coreapi"  // 导入外部包
    "github.com/ipfs/kubo/repo"     // 导入外部包
    "github.com/stretchr/testify/assert"  // 导入外部包

    "github.com/ipfs/boxo/path"  // 导入外部包
    "github.com/ipfs/go-datastore"  // 导入外部包
    syncds "github.com/ipfs/go-datastore/sync"  // 导入外部包
    "github.com/ipfs/kubo/config"  // 导入外部包
    iface "github.com/ipfs/kubo/core/coreiface"  // 导入外部包
    ci "github.com/libp2p/go-libp2p/core/crypto"  // 导入外部包
)

type mockNamesys map[string]path.Path  // 定义名字系统的模拟类型

func (m mockNamesys) Resolve(ctx context.Context, p path.Path, opts ...namesys.ResolveOption) (namesys.Result, error) {
    cfg := namesys.DefaultResolveOptions()  // 获取默认的解析选项
    for _, o := range opts {  // 遍历解析选项
        o(&cfg)  // 应用解析选项
    }
    depth := cfg.Depth  // 获取解析深度
    if depth == namesys.UnlimitedDepth {  // 如果解析深度为无限
        // max uint
        depth = ^uint(0)  // 设置深度为最大值
    }
    var (
        value path.Path  // 定义路径值
    )
    name := path.SegmentsToString(p.Segments()[:2]...)  // 获取路径的字符串表示
    for strings.HasPrefix(name, "/ipns/") {  // 当路径以 "/ipns/" 开头时
        if depth == 0 {  // 如果深度为 0
            return namesys.Result{Path: value}, namesys.ErrResolveRecursion  // 返回递归解析错误
        }
        depth--  // 深度减一

        v, ok := m[name]  // 获取路径对应的值
        if !ok {  // 如果不存在
            return namesys.Result{}, namesys.ErrResolveFailed  // 返回解析失败错误
        }
        value = v  // 设置路径值
        name = value.String()  // 获取路径的字符串表示
    }

    value, err := path.Join(value, p.Segments()[2:]...)  // 连接路径值和路径的后续部分
    return namesys.Result{Path: value}, err  // 返回路径值和错误
}

func (m mockNamesys) ResolveAsync(ctx context.Context, p path.Path, opts ...namesys.ResolveOption) <-chan namesys.AsyncResult {
    out := make(chan namesys.AsyncResult, 1)  // 创建异步结果通道
    res, err := m.Resolve(ctx, p, opts...)  // 解析路径
    out <- namesys.AsyncResult{Path: res.Path, TTL: res.TTL, LastMod: res.LastMod, Err: err}  // 发送解析结果到通道
    close(out)  // 关闭通道
    return out  // 返回通道
}

func (m mockNamesys) Publish(ctx context.Context, name ci.PrivKey, value path.Path, opts ...namesys.PublishOption) error {
    # 返回一个错误对象，表示该功能在模拟名称系统中尚未实现
    return errors.New("not implemented for mockNamesys")
}

// 实现 mockNamesys 接口的 GetResolver 方法
func (m mockNamesys) GetResolver(subs string) (namesys.Resolver, bool) {
    // 返回空解析器和 false
    return nil, false
}

// 使用 mockNamesys 创建新的 IpfsNode
func newNodeWithMockNamesys(ns mockNamesys) (*core.IpfsNode, error) {
    // 创建配置对象
    c := config.Config{
        Identity: config.Identity{
            PeerID: "QmTFauExutTsy4XP6JbMFcw2Wa9645HJt2bTqL6qYDCKfe", // 离线节点所需的对等节点 ID
        },
    }
    // 创建模拟的存储库
    r := &repo.Mock{
        C: c,
        D: syncds.MutexWrap(datastore.NewMapDatastore()),
    }
    // 创建新的节点
    n, err := core.NewNode(context.Background(), &core.BuildCfg{Repo: r})
    if err != nil {
        return nil, err
    }
    // 设置节点的 Namesys 属性
    n.Namesys = ns
    return n, nil
}

// 实现 delegatedHandler 结构体的 ServeHTTP 方法
type delegatedHandler struct {
    http.Handler
}

func (dh *delegatedHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    // 调用内部的 Handler 的 ServeHTTP 方法
    dh.Handler.ServeHTTP(w, r)
}

// 发送不带重定向的 HTTP 请求
func doWithoutRedirect(req *http.Request) (*http.Response, error) {
    tag := "without-redirect"
    // 创建自定义的 HTTP 客户端
    c := &http.Client{
        CheckRedirect: func(req *http.Request, via []*http.Request) error {
            return errors.New(tag)
        },
    }
    // 发送 HTTP 请求
    res, err := c.Do(req)
    if err != nil && !strings.Contains(err.Error(), tag) {
        return nil, err
    }
    return res, nil
}

// 创建测试服务器和节点
func newTestServerAndNode(t *testing.T, ns mockNamesys) (*httptest.Server, iface.CoreAPI, context.Context) {
    // 使用 mockNamesys 创建新的节点
    n, err := newNodeWithMockNamesys(ns)
    if err != nil {
        t.Fatal(err)
    }

    // 在这里需要这个变量，因为我们需要使用 listener 构建 handler，然后使用 handler 构建 server
    dh := &delegatedHandler{}
    ts := httptest.NewServer(dh)
    t.Cleanup(func() { ts.Close() })

    // 设置 delegatedHandler 的 Handler 属性
    dh.Handler, err = MakeHandler(n,
        ts.Listener,
        HostnameOption(),
        GatewayOption("/ipfs", "/ipns"),
        VersionOption(),
    )
    if err != nil {
        t.Fatal(err)
    }

    // 创建 CoreAPI
    api, err := coreapi.NewCoreAPI(n)
    if err != nil {
        t.Fatal(err)
    }

    return ts, api, n.Context()
}

// 测试版本信息
func TestVersion(t *testing.T) {
    // 设置当前提交的短哈希
    version.CurrentCommit = "theshortcommithash"
}
    # 创建一个模拟的名字系统对象
    ns := mockNamesys{}
    # 创建一个测试服务器和节点，并将名字系统对象传入
    ts, _, _ := newTestServerAndNode(t, ns)
    # 打印测试服务器的 URL
    t.Logf("test server url: %s", ts.URL)

    # 创建一个 HTTP GET 请求，指向测试服务器的 /version 路径
    req, err := http.NewRequest(http.MethodGet, ts.URL+"/version", nil)
    # 如果创建请求时出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    # 发送请求并获取响应
    res, err := doWithoutRedirect(req)
    # 如果发送请求时出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }
    # 读取响应体的内容
    body, err := io.ReadAll(res.Body)
    # 如果读取响应体时出现错误，记录错误并终止测试
    if err != nil {
        t.Fatalf("error reading response: %s", err)
    }
    # 将响应体内容转换为字符串
    s := string(body)

    # 检查响应中是否包含指定的提交信息，如果不包含，记录错误并终止测试
    if !strings.Contains(s, "Commit: theshortcommithash") {
        t.Fatalf("response doesn't contain commit:\n%s", s)
    }

    # 检查响应中是否包含客户端版本信息，如果不包含，记录错误并终止测试
    if !strings.Contains(s, "Client Version: "+version.GetUserAgentVersion()) {
        t.Fatalf("response doesn't contain client version:\n%s", s)
    }
func TestDeserializedResponsesInheritance(t *testing.T) {
    // 遍历测试用例
    for _, testCase := range []struct {
        globalSetting          config.Flag
        gatewaySetting         config.Flag
        expectedGatewaySetting bool
    }{
        // 定义测试用例
        {config.True, config.Default, true},
        {config.False, config.Default, false},
        {config.False, config.True, true},
        {config.True, config.False, false},
    } {
        // 创建配置对象
        c := config.Config{
            Identity: config.Identity{
                PeerID: "QmTFauExutTsy4XP6JbMFcw2Wa9645HJt2bTqL6qYDCKfe", // offline 节点所需的 PeerID
            },
            Gateway: config.Gateway{
                DeserializedResponses: testCase.globalSetting,
                PublicGateways: map[string]*config.GatewaySpec{
                    "example.com": {
                        DeserializedResponses: testCase.gatewaySetting,
                    },
                },
            },
        }
        // 创建模拟仓库对象
        r := &repo.Mock{
            C: c,
            D: syncds.MutexWrap(datastore.NewMapDatastore()),
        }
        // 创建新的节点
        n, err := core.NewNode(context.Background(), &core.BuildCfg{Repo: r})
        assert.NoError(t, err)

        // 获取网关配置
        gwCfg, err := getGatewayConfig(n)
        assert.NoError(t, err)

        // 断言公共网关中包含指定网关
        assert.Contains(t, gwCfg.PublicGateways, "example.com")
        // 断言网关设置与预期值相等
        assert.Equal(t, testCase.expectedGatewaySetting, gwCfg.PublicGateways["example.com"].DeserializedResponses)
    }
}
```