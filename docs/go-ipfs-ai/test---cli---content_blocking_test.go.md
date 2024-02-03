# `kubo\test\cli\content_blocking_test.go`

```go
package cli

import (
    "context"  // 上下文包，用于控制取消操作
    "fmt"  // 格式化包，用于打印输出
    "io"  // 输入输出包，用于处理输入输出流
    "log"  // 日志包，用于记录日志
    "net/http"  // HTTP包，用于处理HTTP请求
    "net/url"  // URL包，用于解析URL
    "os"  // 操作系统包，用于操作系统功能
    "path/filepath"  // 文件路径包，用于处理文件路径
    "strings"  // 字符串包，用于处理字符串
    "testing"  // 测试包，用于编写测试函数

    "github.com/ipfs/kubo/test/cli/harness"  // 引入自定义测试包
    "github.com/libp2p/go-libp2p"  // 引入libp2p包
    "github.com/libp2p/go-libp2p/core/peer"  // 引入libp2p核心包
    libp2phttp "github.com/libp2p/go-libp2p/p2p/http"  // 引入libp2p HTTP包
    "github.com/stretchr/testify/assert"  // 引入测试断言包
    "github.com/stretchr/testify/require"  // 引入测试要求包
)

func TestContentBlocking(t *testing.T) {
    // 注意：我们不能使用 t.Parallel() 运行此测试，因为我们设置了 IPFS_NS_MAP
    // 并行运行可能会影响其他测试

    const blockedMsg = "blocked and cannot be provided"  // 定义常量，表示被阻止且无法提供
    const statusExpl = "specific HTTP error code is expected"  // 定义常量，表示预期特定的HTTP错误代码
    const bodyExpl = "Error message informing about content block is expected"  // 定义常量，表示预期的内容阻止错误消息

    h := harness.NewT(t)  // 创建测试工具实例

    // 初始化 IPFS_PATH
    node := h.NewNode().Init("--empty-repo", "--profile=test")  // 初始化IPFS节点

    // 创建测试中使用的CID
    h.WriteFile("blocked-dir/subdir/indirectly-blocked-file.txt", "indirectly blocked file content")  // 写入文件内容
    parentDirCID := node.IPFS("add", "--raw-leaves", "-Q", "-r", filepath.Join(h.Dir, "blocked-dir")).Stdout.Trimmed()  // 获取CID

    h.WriteFile("directly-blocked-file.txt", "directly blocked file content")  // 写入文件内容
    blockedCID := node.IPFS("add", "--raw-leaves", "-Q", filepath.Join(h.Dir, "directly-blocked-file.txt")).Stdout.Trimmed()  // 获取CID

    h.WriteFile("not-blocked-file.txt", "not blocked file content")  // 写入文件内容
    allowedCID := node.IPFS("add", "--raw-leaves", "-Q", filepath.Join(h.Dir, "not-blocked-file.txt")).Stdout.Trimmed()  // 获取CID

    // 在 $IPFS_PATH/denylists/test.deny 创建拒绝列表
    // 将拒绝列表写入临时文件
    denylistTmp := h.WriteToTemp("name: test list\n---\n" +
        "//QmX9dhRcQcKUw3Ws8485T5a9dtjrSCQaUAHnG4iK9i4ceM\n" + // 双哈希（sha256）CID块：base58btc（sha256-multihash（QmVTF1yEejXd9iMgoRTFDxBv7HAz9kuZcQNBzHrceuK9HR））
        "//gW813G35CnLsy7gRYYHuf63hrz71U1xoLFDVeV7actx6oX\n" + // 双哈希（blake3）blake3根CID下的路径块：base58btc（blake3-multihash（gW7Nhu4HrfDtphEivm3Z9NNE7gpdh5Tga8g6JNZc1S8E47/path））
        "//8526ba05eec55e28f8db5974cc891d0d92c8af69d386fc6464f1e9f372caf549\n" + // 旧版CID双哈希块：sha256（bafkqahtcnrxwg23fmqqgi33vmjwgk2dbonuca3dfm5qwg6jamnuwicq/）
        "//e5b7d2ce2594e2e09901596d8e1f29fa249b74c8c9e32ea01eda5111e4d33f07\n" + // 旧版路径双哈希块：sha256（bafyaagyscufaqalqaacauaqiaejao43vmjygc5didacauaqiae/subpath）
        "/ipfs/" + blockedCID + "\n" + // 阻止特定CID
        "/ipfs/" + parentDirCID + "/subdir*\n" + // 仅阻止特定子路径
        "/ipns/blocked-cid.example.com\n" +
        "/ipns/blocked-dnslink.example.com\n")

    // 如果失败，则创建denylists目录
    if err := os.MkdirAll(filepath.Join(node.Dir, "denylists"), 0o777); err != nil {
        log.Panicf("failed to create denylists dir: %s", err.Error())
    }
    // 如果失败，则重命名临时文件为test.deny
    if err := os.Rename(denylistTmp, filepath.Join(node.Dir, "denylists", "test.deny")); err != nil {
        log.Panicf("failed to create test denylist: %s", err.Error())
    }

    // 向namesys解析缓存添加两个条目
    // /ipns/blocked-cid.example.com指向一个被阻止的CID（以确认阻止影响/ipns解析）
    // /ipns/blocked-dnslink.example.com具有安全CID（用于测试/ipns/路径的阻止）
    os.Setenv("IPFS_NS_MAP", "blocked-cid.example.com:/ipfs/"+blockedCID+",blocked-dnslink.example.com/ipns/QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn")
    defer os.Unsetenv("IPFS_NS_MAP")

    // 启用GatewayOverLibp2p，因为我们也想在那里测试拒绝列表
    node.IPFS("config", "--json", "Experimental.GatewayOverLibp2p", "true")
    // 启动守护进程，它应该从 $IPFS_PATH/denylists/test.deny 中获取拒绝列表
    node.StartDaemon() // 我们需要在线模式进行 GatewayOverLibp2p 测试
    client := node.GatewayClient()

    // 首先，确认网关是否正常工作
    t.Run("Gateway Allows CID that is not blocked", func(t *testing.T) {
        t.Parallel()
        resp := client.Get("/ipfs/" + allowedCID)
        assert.Equal(t, http.StatusOK, resp.StatusCode)
        assert.Equal(t, "not blocked file content", resp.Body)
    })

    // 然后，最基本的阻止情况是否有效？
    t.Run("Gateway Denies directly blocked CID", func(t *testing.T) {
        t.Parallel()
        resp := client.Get("/ipfs/" + blockedCID)
        assert.Equal(t, http.StatusGone, resp.StatusCode, statusExpl)
        assert.NotEqual(t, "directly blocked file content", resp.Body)
        assert.Contains(t, resp.Body, blockedMsg, bodyExpl)
    })

    // 确认被阻止子路径的父路径未被阻止
    t.Run("Gateway Allows parent Path that is not blocked", func(t *testing.T) {
        t.Parallel()
        resp := client.Get("/ipfs/" + parentDirCID)
        assert.Equal(t, http.StatusOK, resp.StatusCode)
    })

    // 现在，我们想要在 CLI 和 Gateway 中覆盖的所有测试用例
    testCases := []struct {
        name string
        path string
    }{
        {
            name: "directly blocked CID",  # 直接被阻止的 CID
            path: "/ipfs/" + blockedCID,  # 被阻止的 CID 的路径
        },
        {
            name: "indirectly blocked file (on a blocked subpath)",  # 间接被阻止的文件（在被阻止的子路径上）
            path: "/ipfs/" + parentDirCID + "/subdir/indirectly-blocked-file.txt",  # 间接被阻止的文件的路径
        },
        {
            name: "/ipns path that resolves to a blocked CID",  # 解析为被阻止的 CID 的 /ipns 路径
            path: "/ipns/blocked-cid.example.com",  # 被阻止的 CID 的 /ipns 路径
        },
        {
            name: "/ipns Path that is blocked by DNSLink name",  # 被 DNSLink 名称阻止的 /ipns 路径
            path: "/ipns/blocked-dnslink.example.com",  # 被 DNSLink 名称阻止的 /ipns 路径
        },
        {
            name: "double-hash CID block (sha256-multihash)",  # 双哈希 CID 阻止（sha256-multihash）
            path: "/ipfs/QmVTF1yEejXd9iMgoRTFDxBv7HAz9kuZcQNBzHrceuK9HR",  # 双哈希 CID 阻止的路径
        },
        {
            name: "double-hash Path block (blake3-multihash)",  # 双哈希路径阻止（blake3-multihash）
            path: "/ipfs/bafyb4ieqht3b2rssdmc7sjv2cy2gfdilxkfh7623nvndziyqnawkmo266a/path",  # 双哈希路径阻止的路径
        },
        {
            name: "legacy CID double-hash block (sha256)",  # 传统 CID 双哈希阻止（sha256）
            path: "/ipfs/bafkqahtcnrxwg23fmqqgi33vmjwgk2dbonuca3dfm5qwg6jamnuwicq",  # 传统 CID 双哈希阻止的路径
        },

        {
            name: "legacy Path double-hash block (sha256)",  # 传统路径双哈希阻止（sha256）
            path: "/ipfs/bafyaagyscufaqalqaacauaqiaejao43vmjygc5didacauaqiae/subpath",  # 传统路径双哈希阻止的路径
        },
    }

    // Which specific cliCmds we test against testCases
    cliCmds := [][]string{  # 定义一个二维字符串数组 cliCmds
        {"block", "get"},  # 第一个测试命令
        {"block", "stat"},  # 第二个测试命令
        {"dag", "get"},  # 第三个测试命令
        {"dag", "export"},  # 第四个测试命令
        {"dag", "stat"},  # 第五个测试命令
        {"cat"},  # 第六个测试命令
        {"ls"},  # 第七个测试命令
        {"get"},  # 第八个测试命令
        {"refs"},  # 第九个测试命令
    }

    expectedMsg := blockedMsg  # 定义一个变量 expectedMsg，赋值为 blockedMsg
    for _, testCase := range testCases {

        // 遍历测试用例列表
        for _, cmd := range cliCmds {
            // 遍历命令列表
            cmd := cmd
            // 根据命令和测试用例名称生成测试名称
            cliTestName := fmt.Sprintf("CLI '%s' denies %s", strings.Join(cmd, " "), testCase.name)
            // 运行测试
            t.Run(cliTestName, func(t *testing.T) {
                t.Parallel()
                // 拼接命令和测试用例路径，执行IPFS命令，获取标准错误输出
                args := append(cmd, testCase.path)
                errMsg := node.RunIPFS(args...).Stderr.Trimmed()
                // 判断标准错误输出中是否包含预期的消息
                if !strings.Contains(errMsg, expectedMsg) {
                    t.Errorf("Expected STDERR error message %q, but got: %q", expectedMsg, errMsg)
                }
            })
        }

        // 根据测试用例路径确认denylist是否生效
        gwTestName := fmt.Sprintf("Gateway denies %s", testCase.name)
        t.Run(gwTestName, func(t *testing.T) {
            // 发送HTTP GET请求，确认状态码和响应内容
            resp := client.Get(testCase.path)
            assert.Equal(t, http.StatusGone, resp.StatusCode, statusExpl)
            assert.Contains(t, resp.Body, blockedMsg, bodyExpl)
        })

    }

    // 子域名网关的额外边缘情况

    t.Run("Gateway Denies /ipns Path that is blocked by DNSLink name (subdomain redirect)", func(t *testing.T) {
        t.Parallel()

        // 获取网关URL并发送HTTP GET请求，确认状态码和响应内容
        gwURL, _ := url.Parse(node.GatewayURL())
        resp := client.Get("/ipns/blocked-dnslink.example.com", func(r *http.Request) {
            r.Host = "localhost:" + gwURL.Port()
        })

        assert.Equal(t, http.StatusGone, resp.StatusCode, statusExpl)
        assert.Contains(t, resp.Body, blockedMsg, bodyExpl)
    })
    # 测试网关拒绝被 DNSLink 名称阻止的 /ipns 路径（子域名，无 TLS）
    t.Run("Gateway Denies /ipns Path that is blocked by DNSLink name (subdomain, no TLS)", func(t *testing.T) {
        t.Parallel()

        # 解析网关 URL
        gwURL, _ := url.Parse(node.GatewayURL())
        # 发送 GET 请求
        resp := client.Get("/", func(r *http.Request) {
            # 设置请求头中的 Host 字段
            r.Host = "blocked-dnslink.example.com.ipns.localhost:" + gwURL.Port()
        })

        # 断言响应状态码为 http.StatusGone
        assert.Equal(t, http.StatusGone, resp.StatusCode, statusExpl)
        # 断言响应体包含特定消息
        assert.Contains(t, resp.Body, blockedMsg, bodyExpl)
    })

    # 测试网关拒绝被 DNSLink 名称阻止的 /ipns 路径（子域名，TLS 内联）
    t.Run("Gateway Denies /ipns Path that is blocked by DNSLink name (subdomain, inlined for TLS)", func(t *testing.T) {
        t.Parallel()

        # 解析网关 URL
        gwURL, _ := url.Parse(node.GatewayURL())
        # 发送 GET 请求
        resp := client.Get("/", func(r *http.Request) {
            # 内联 DNSLink 以适应单个 DNS 标签以进行 TLS 互操作
            r.Host = "blocked--dnslink-example-com.ipns.localhost:" + gwURL.Port()
        })

        # 断言响应状态码为 http.StatusGone
        assert.Equal(t, http.StatusGone, resp.StatusCode, statusExpl)
        # 断言响应体包含特定消息
        assert.Contains(t, resp.Body, blockedMsg, bodyExpl)
    })

    # 当网关以 NoFetch 模式运行时，我们需要确认拒绝列表是否生效
    # （通常会将 blockservice 切换为只读模式，并且该切换可能导致拒绝列表不被应用，因为它是一个单独的代码路径）
    t.Run("GatewayNoFetch", func(t *testing.T) {
        // 注意：我们不并行运行这个测试，因为它需要使用不同的配置重启

        // 切换网关到 NoFetch 模式
        node.StopDaemon()
        node.IPFS("config", "--json", "Gateway.NoFetch", "true")
        node.StartDaemon()

        // 更新客户端，因为测试节点的端口可能在重启后发生变化
        client = node.GatewayClient()

        // 首先，确认网关正常工作
        t.Run("Allows CID that is not blocked", func(t *testing.T) {
            resp := client.Get("/ipfs/" + allowedCID)
            assert.Equal(t, http.StatusOK, resp.StatusCode)
            assert.Equal(t, "not blocked file content", resp.Body)
        })

        // 然后，最基本的阻止情况是否有效？
        t.Run("Denies directly blocked CID", func(t *testing.T) {
            resp := client.Get("/ipfs/" + blockedCID)
            assert.Equal(t, http.StatusGone, resp.StatusCode, statusExpl)
            assert.NotEqual(t, "directly blocked file content", resp.Body)
            assert.Contains(t, resp.Body, blockedMsg, bodyExpl)
        })

        // 恢复默认设置
        node.StopDaemon()
        node.IPFS("config", "--json", "Gateway.NoFetch", "false")
        node.StartDaemon()
        client = node.GatewayClient()
    })

    // 我们需要确认拒绝列表在通过 libp2p 暴露的不可信任网关上是活动的
    // 当 Experimental.GatewayOverLibp2p=true 时
    // （https://github.com/ipfs/kubo/blob/master/docs/experimental-features.md#http-gateway-over-libp2p）
    // 注意：这种类型的网关被硬编码为 NoFetch：它不会获取不在本地存储中的数据，因此我们只需要运行一次：对允许的 CID 和 blockedCID 进行简单的烟雾测试。
    t.Run("GatewayOverLibp2p", func(t *testing.T) {
        t.Parallel()

        // 创建一个连接到我们节点的 libp2p 客户端，通过 /http1.1 连接，然后在 /ipfs/gateway 子协议上进行网关语义交互
        clientHost, err := libp2p.New(libp2p.NoListenAddrs)
        require.NoError(t, err)
        err = clientHost.Connect(context.Background(), peer.AddrInfo{
            ID:    node.PeerID(),
            Addrs: node.SwarmAddrs(),
        })
        require.NoError(t, err)

        // 使用 libp2phttp.Host 创建 libp2p 客户端，并指定使用 /ipfs/gateway 子协议，连接到指定的节点
        libp2pClient, err := (&libp2phttp.Host{StreamHost: clientHost}).NamespacedClient("/ipfs/gateway", peer.AddrInfo{ID: node.PeerID()})
        require.NoError(t, err)

        t.Run("Serves Allowed CID", func(t *testing.T) {
            t.Parallel()
            // 获取允许的 CID 的内容
            resp, err := libp2pClient.Get(fmt.Sprintf("/ipfs/%s?format=raw", allowedCID))
            require.NoError(t, err)
            defer resp.Body.Close()
            // 断言响应状态码为 200
            assert.Equal(t, http.StatusOK, resp.StatusCode)
            // 读取响应体内容
            body, err := io.ReadAll(resp.Body)
            require.NoError(t, err)
            // 断言响应体内容与预期相等
            require.Equal(t, string(body), "not blocked file content", bodyExpl)
        })

        t.Run("Denies Blocked CID", func(t *testing.T) {
            t.Parallel()
            // 获取被阻止的 CID 的内容
            resp, err := libp2pClient.Get(fmt.Sprintf("/ipfs/%s?format=raw", blockedCID))
            require.NoError(t, err)
            defer resp.Body.Close()
            // 断言响应状态码为 410
            assert.Equal(t, http.StatusGone, resp.StatusCode, statusExpl)
            // 读取响应体内容
            body, err := io.ReadAll(resp.Body)
            require.NoError(t, err)
            // 断言响应体内容不等于 "directly blocked file content"
            assert.NotEqual(t, string(body), "directly blocked file content")
            // 断言响应体内容包含特定的消息
            assert.Contains(t, string(body), blockedMsg, bodyExpl)
        })
    })
# 闭合前面的函数定义
```