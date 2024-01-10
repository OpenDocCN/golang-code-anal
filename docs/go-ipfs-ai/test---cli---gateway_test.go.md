# `kubo\test\cli\gateway_test.go`

```
package cli

import (
    "context"  // 上下文包，用于控制请求的取消、超时等
    "encoding/json"  // JSON 编解码包
    "fmt"  // 格式化包，用于打印输出
    "net/http"  // HTTP 客户端包
    "os"  // 操作系统功能包
    "path/filepath"  // 文件路径包
    "regexp"  // 正则表达式包
    "strconv"  // 字符串转换包
    "strings"  // 字符串处理包
    "testing"  // 测试包

    "github.com/ipfs/kubo/config"  // 导入自定义包
    "github.com/ipfs/kubo/test/cli/harness"  // 导入自定义包
    . "github.com/ipfs/kubo/test/cli/testutils"  // 导入自定义包
    "github.com/libp2p/go-libp2p/core/peer"  // 导入 libp2p 包
    "github.com/multiformats/go-multiaddr"  // 导入多地址格式包
    manet "github.com/multiformats/go-multiaddr/net"  // 导入多地址格式包
    "github.com/multiformats/go-multibase"  // 导入多基编码包
    "github.com/stretchr/testify/assert"  // 导入断言包
    "github.com/stretchr/testify/require"  // 导入断言包
)

func TestGateway(t *testing.T) {
    t.Parallel()  // 并行执行测试
    h := harness.NewT(t)  // 创建测试工具实例
    node := h.NewNode().Init().StartDaemon("--offline")  // 初始化并启动节点
    cid := node.IPFSAddStr("Hello Worlds!")  // 向 IPFS 添加字符串并返回 CID

    peerID, err := peer.ToCid(node.PeerID()).StringOfBase(multibase.Base36)  // 将节点的 PeerID 转换为指定进制的字符串
    assert.NoError(t, err)  // 断言没有错误发生

    client := node.GatewayClient()  // 创建网关客户端
    client.TemplateData = map[string]string{  // 设置模板数据
        "CID":    cid,  // 设置 CID
        "PeerID": peerID,  // 设置 PeerID
    }

    t.Run("GET IPFS path succeeds", func(t *testing.T) {  // 运行子测试
        t.Parallel()  // 并行执行子测试
        resp := client.Get("/ipfs/{{.CID}}")  // 发送 GET 请求获取指定 CID 的内容
        assert.Equal(t, 200, resp.StatusCode)  // 断言响应状态码为 200
    })

    t.Run("GET IPFS path with explicit ?filename succeeds with proper header", func(t *testing.T) {  // 运行子测试
        t.Parallel()  // 并行执行子测试
        resp := client.Get("/ipfs/{{.CID}}?filename=testтест.pdf")  // 发送 GET 请求获取指定 CID 的内容，并指定文件名
        assert.Equal(t, 200, resp.StatusCode)  // 断言响应状态码为 200
        assert.Equal(t,
            `inline; filename="test____.pdf"; filename*=UTF-8''test%D1%82%D0%B5%D1%81%D1%82.pdf`,
            resp.Headers.Get("Content-Disposition"),  // 断言响应头中的 Content-Disposition 字段值
        )
    })
    // 使用测试框架运行 GET 请求，获取具有显式 ?filename 和 &download=true 参数的 IPFS 路径，并验证是否成功并返回正确的头部信息
    t.Run("GET IPFS path with explicit ?filename and &download=true succeeds with proper header", func(t *testing.T) {
        t.Parallel()
        resp := client.Get("/ipfs/{{.CID}}?filename=testтест.mp4&download=true")
        assert.Equal(t, 200, resp.StatusCode) // 验证状态码是否为 200
        assert.Equal(t,
            `attachment; filename="test____.mp4"; filename*=UTF-8''test%D1%82%D0%B5%D1%81%D1%82.mp4`,
            resp.Headers.Get("Content-Disposition"), // 验证 Content-Disposition 头部信息是否正确
        )
    })

    // https://github.com/ipfs/go-ipfs/issues/4025#issuecomment-342250616
    // 使用测试框架运行 GET 请求，尝试在 IPFS 内容根之外注册服务器工作器，并验证是否返回错误
    t.Run("GET for Server Worker registration outside of an IPFS content root errors", func(t *testing.T) {
        t.Parallel()
        resp := client.Get("/ipfs/{{.CID}}?filename=sw.js", client.WithHeader("Service-Worker", "script"))
        assert.Equal(t, 400, resp.StatusCode) // 验证状态码是否为 400
        assert.Contains(t, resp.Body, "navigator.serviceWorker: registration is not allowed for this scope") // 验证响应体中是否包含特定错误信息
    })

    })
    t.Run("IPNS", func(t *testing.T) {
        t.Parallel()
        // 在 IPFS 上发布 IPNS 记录，允许离线访问，设置 TTL 为 42 小时，使用给定的 CID
        node.IPFS("name", "publish", "--allow-offline", "--ttl", "42h", cid)

        t.Run("GET invalid IPNS root returns 500 (Internal Server Error)", func(t *testing.T) {
            t.Parallel()
            // 发起 GET 请求，获取无效 IPNS 根路径，预期返回 500 (Internal Server Error)
            resp := client.Get("/ipns/QmInvalid/pleaseDontAddMe")
            assert.Equal(t, 500, resp.StatusCode)
        })

        t.Run("GET IPNS path succeeds", func(t *testing.T) {
            t.Parallel()
            // 发起 GET 请求，获取 IPNS 路径，预期返回状态码 200
            resp := client.Get("/ipns/{{.PeerID}}")
            assert.Equal(t, 200, resp.StatusCode)
            assert.Equal(t, "Hello Worlds!", resp.Body)
        })

        t.Run("GET IPNS path has correct Cache-Control", func(t *testing.T) {
            t.Parallel()
            // 发起 GET 请求，获取 IPNS 路径，预期返回状态码 200
            resp := client.Get("/ipns/{{.PeerID}}")
            assert.Equal(t, 200, resp.StatusCode)
            // 获取响应头中的 Cache-Control
            cacheControl := resp.Headers.Get("Cache-Control")
            assert.True(t, strings.HasPrefix(cacheControl, "public, max-age="))
            // 将 Cache-Control 中的最大年龄转换为整数
            maxAge, err := strconv.Atoi(strings.TrimPrefix(cacheControl, "public, max-age="))
            assert.NoError(t, err)
            // 断言最大年龄在 42 小时和 42 小时减 1 分钟之间
            assert.True(t, maxAge-151200 < 60) // MaxAge within 42h and 42h-1m
        })

        t.Run("GET /ipfs/ipns/{peerid} returns redirect to the valid path", func(t *testing.T) {
            t.Parallel()
            // 发起 GET 请求，获取 /ipfs/ipns/{peerid} 路径，预期重定向到有效路径
            resp := client.Get("/ipfs/ipns/{{.PeerID}}?query=to-remember")

            assert.Contains(t,
                resp.Body,
                fmt.Sprintf(`<meta http-equiv="refresh" content="10;url=/ipns/%s?query=to-remember" />`, peerID),
            )
            assert.Contains(t,
                resp.Body,
                fmt.Sprintf(`<link rel="canonical" href="/ipns/%s?query=to-remember" />`, peerID),
            )
        })
    })

    t.Run("GET invalid IPFS path errors", func(t *testing.T) {
        t.Parallel()
        // 发起 GET 请求，获取无效的 IPFS 路径，预期返回状态码 400
        assert.Equal(t, 400, client.Get("/ipfs/12345").StatusCode)
    })
    # 运行测试：当使用无效路径时，应返回404错误
    t.Run("GET invalid path errors", func(t *testing.T) {
        t.Parallel()
        assert.Equal(t, 404, client.Get("/12345").StatusCode)
    })

    # TODO: 这些使用 API URL 的测试不应该是网关测试的一部分...
    # 运行测试：当使用"/webui"路径时，应返回301或302状态码
    t.Run("GET /webui returns 301 or 302", func(t *testing.T) {
        t.Parallel()
        resp := node.APIClient().DisableRedirects().Get("/webui")
        assert.Contains(t, []int{302, 301}, resp.StatusCode)
    })

    # 运行测试：当使用"/webui/"路径时，应返回301或302状态码
    t.Run("GET /webui/ returns 301 or 302", func(t *testing.T) {
        t.Parallel()
        resp := node.APIClient().DisableRedirects().Get("/webui/")
        assert.Contains(t, []int{302, 301}, resp.StatusCode)
    })

    # 运行测试：当使用"/webui/"路径时，应返回用户指定的头信息
    t.Run("GET /webui/ returns user-specified headers", func(t *testing.T) {
        t.Parallel()

        # 定义头信息和值
        header := "Access-Control-Allow-Origin"
        values := []string{"http://localhost:3000", "https://webui.ipfs.io"}

        # 创建新的节点并初始化
        node := harness.NewT(t).NewNode().Init()
        # 更新配置，设置API的HTTP头信息
        node.UpdateConfig(func(cfg *config.Config) {
            cfg.API.HTTPHeaders = map[string][]string{header: values}
        })
        # 启动守护进程
        node.StartDaemon()

        # 发送GET请求，获取响应
        resp := node.APIClient().DisableRedirects().Get("/webui/")
        # 断言响应头信息中包含指定的值
        assert.Equal(t, resp.Headers.Values(header), values)
        # 断言响应状态码为301或302
        assert.Contains(t, []int{302, 301}, resp.StatusCode)
    })
    # 使用测试框架运行 GET /logs 接口返回日志的测试
    t.Run("GET /logs returns logs", func(t *testing.T) {
        t.Parallel()
        # 创建 API 客户端
        apiClient := node.APIClient()
        # 构建请求 URL
        reqURL := apiClient.BuildURL("/logs")

        # 创建带有取消功能的上下文
        ctx, cancel := context.WithCancel(context.Background())
        # 延迟取消上下文
        defer cancel()

        # 创建 HTTP 请求
        req, err := http.NewRequestWithContext(ctx, http.MethodGet, reqURL, nil)
        require.NoError(t, err)

        # 发起 HTTP 请求
        resp, err := apiClient.Client.Do(req)
        require.NoError(t, err)
        # 延迟关闭响应体
        defer resp.Body.Close()

        # 读取输出的第一行并解析为 JSON
        dec := json.NewDecoder(resp.Body)
        event := struct{ Event string }{}
        err = dec.Decode(&event)
        require.NoError(t, err)

        # 断言事件内容
        assert.Equal(t, "log API client connected", event.Event)
    })

    # 使用测试框架运行 POST /api/v0/version 接口成功的测试
    t.Run("POST /api/v0/version succeeds", func(t *testing.T) {
        t.Parallel()
        # 发起 POST 请求
        resp := node.APIClient().Post("/api/v0/version", nil)
        # 断言状态码为 200
        assert.Equal(t, 200, resp.StatusCode)

        # 断言响应头中的传输编码
        assert.Len(t, resp.Resp.TransferEncoding, 1)
        assert.Equal(t, "chunked", resp.Resp.TransferEncoding[0])

        # 解析响应体中的 JSON
        vers := struct{ Version string }{}
        err := json.Unmarshal([]byte(resp.Body), &vers)
        require.NoError(t, err)
        # 断言版本号不为空
        assert.NotEmpty(t, vers.Version)
    })
    t.Run("pprof", func(t *testing.T) {
        t.Parallel()
        node := harness.NewT(t).NewNode().Init().StartDaemon()
        apiClient := node.APIClient()
        t.Run("mutex", func(t *testing.T) {
            t.Parallel()
            t.Run("setting the mutex fraction works (negative so it doesn't enable)", func(t *testing.T) {
                t.Parallel()
                // 发送 POST 请求设置互斥量分数为负数，以确保不启用
                resp := apiClient.Post("/debug/pprof-mutex/?fraction=-1", nil)
                // 断言响应状态码为 200
                assert.Equal(t, 200, resp.StatusCode)
            })
            t.Run("mutex endpoint doesn't accept a string as an argument", func(t *testing.T) {
                t.Parallel()
                // 发送 POST 请求设置互斥量分数为字符串，预期会返回 400
                resp := apiClient.Post("/debug/pprof-mutex/?fraction=that_is_a_string", nil)
                // 断言响应状态码为 400
                assert.Equal(t, 400, resp.StatusCode)
            })
            t.Run("mutex endpoint returns 405 on GET", func(t *testing.T) {
                t.Parallel()
                // 发送 GET 请求到互斥量端点，预期会返回 405
                resp := apiClient.Get("/debug/pprof-mutex/?fraction=-1")
                // 断言响应状态码为 405
                assert.Equal(t, 405, resp.StatusCode)
            })
        })
        t.Run("block", func(t *testing.T) {
            t.Parallel()
            t.Run("setting the block profiler rate works (0 so it doesn't enable)", func(t *testing.T) {
                t.Parallel()
                // 发送 POST 请求设置阻塞分析器速率为 0，以确保不启用
                resp := apiClient.Post("/debug/pprof-block/?rate=0", nil)
                // 断言响应状态码为 200
                assert.Equal(t, 200, resp.StatusCode)
            })
            t.Run("block profiler endpoint doesn't accept a string as an argument", func(t *testing.T) {
                t.Parallel()
                // 发送 POST 请求设置阻塞分析器速率为字符串，预期会返回 400
                resp := apiClient.Post("/debug/pprof-block/?rate=that_is_a_string", nil)
                // 断言响应状态码为 400
                assert.Equal(t, 400, resp.StatusCode)
            })
            t.Run("block profiler endpoint returns 405 on GET", func(t *testing.T) {
                t.Parallel()
                // 发送 GET 请求到阻塞分析器端点，预期会返回 405
                resp := apiClient.Get("/debug/pprof-block/?rate=0")
                // 断言响应状态码为 405
                assert.Equal(t, 405, resp.StatusCode)
            })
        })
    })
    # 运行测试，检查索引内容类型
    t.Run("index content types", func(t *testing.T) {
        # 并行执行测试
        t.Parallel()
        # 创建测试工具实例
        h := harness.NewT(t)
        # 初始化并启动守护节点
        node := h.NewNode().Init().StartDaemon()

        # 写入 index.html 文件
        h.WriteFile("index/index.html", "<p></p>")
        # 将文件添加到 IPFS，并获取返回的 CID
        cid := node.IPFS("add", "-Q", "-r", filepath.Join(h.Dir, "index")).Stderr.Trimmed()

        # 创建 API 客户端
        apiClient := node.APIClient()
        # 设置模板数据，包括 CID
        apiClient.TemplateData = map[string]string{"CID": cid}

        # 运行测试，检查 GET 请求 index.html 的内容类型是否正确
        t.Run("GET index.html has correct content type", func(t *testing.T) {
            # 并行执行测试
            t.Parallel()
            # 发送 GET 请求获取 index.html 内容
            res := apiClient.Get("/ipfs/{{.CID}}/")
            # 断言返回的内容类型是否为 text/html; charset=utf-8
            assert.Equal(t, "text/html; charset=utf-8", res.Resp.Header.Get("Content-Type"))
        })

        # 运行测试，检查 HEAD 请求 index.html 是否没有内容
        t.Run("HEAD index.html has no content", func(t *testing.T) {
            # 并行执行测试
            t.Parallel()
            # 发送 HEAD 请求获取 index.html 内容
            res := apiClient.Head("/ipfs/{{.CID}}/")
            # 断言返回的内容是否为空
            assert.Equal(t, "", res.Body)
            # 断言返回的内容长度是否为空
            assert.Equal(t, "", res.Resp.Header.Get("Content-Length"))
        })
    })
    # 运行只读 API 测试
    t.Run("readonly API", func(t *testing.T) {
        t.Parallel()

        # 获取网关客户端
        client := node.GatewayClient()

        # 设置文件内容
        fileContents := "12345"
        # 写入文件到指定路径
        h.WriteFile("readonly/dir/test", fileContents)
        # 将指定目录添加到 IPFS，并获取返回的 CID 列表
        cids := node.IPFS("add", "-r", "-q", filepath.Join(h.Dir, "readonly/dir")).Stdout.Lines()

        # 获取根 CID
        rootCID := cids[len(cids)-1]
        # 设置模板数据，包括根 CID
        client.TemplateData = map[string]string{"RootCID": rootCID}

        # 通过只读 API 获取 IPFS 目录文件成功
        t.Run("Get IPFS directory file through readonly API succeeds", func(t *testing.T) {
            t.Parallel()
            # 发送 GET 请求获取指定文件内容
            resp := client.Get("/api/v0/cat?arg={{.RootCID}}/test")
            # 断言状态码为 200
            assert.Equal(t, 200, resp.StatusCode)
            # 断言返回的文件内容与预期一致
            assert.Equal(t, fileContents, resp.Body)
        })

        # 通过只读 API 引用 IPFS 目录文件成功
        t.Run("refs IPFS directory file through readonly API succeeds", func(t *testing.T) {
            t.Parallel()
            # 发送 GET 请求引用指定文件
            resp := client.Get("/api/v0/refs?arg={{.RootCID}}/test")
            # 断言状态码为 200
            assert.Equal(t, 200, resp.StatusCode)
        })

        # 测试网关 API 是否经过了安全处理
        t.Run("test gateway API is sanitized", func(t *testing.T) {
            t.Parallel()
            # 遍历需要测试的命令列表
            for _, cmd := range []string{
                "add",
                "block/put",
                "bootstrap",
                "config",
                "dag/put",
                "dag/import",
                "dht",
                "diag",
                "id",
                "mount",
                "name/publish",
                "object/put",
                "object/new",
                "object/patch",
                "pin",
                "ping",
                "repo",
                "stats",
                "swarm",
                "file",
                "update",
                "bitswap",
            } {
                # 对每个命令进行测试
                t.Run(cmd, func(t *testing.T) {
                    cmd := cmd
                    t.Parallel()
                    # 发送 GET 请求，断言状态码为 404
                    assert.Equal(t, 404, client.Get("/api/v0/"+cmd).StatusCode)
                })
            }
        })
    })
    # 运行测试 "refs/local"
    t.Run("refs/local", func(t *testing.T) {
        # 标记测试为并行执行
        t.Parallel()
        # 获取节点网关地址并转换为多地址格式
        gatewayAddr := URLStrToMultiaddr(node.GatewayURL())
        # 运行 IPFS 命令，获取结果
        res := node.RunIPFS("--api", gatewayAddr.String(), "refs", "local")
        # 断言结果的标准错误输出包含特定错误信息
        assert.Contains(t,
            res.Stderr.Trimmed(),
            `Error: invalid path "local":`,
        )
    })

    # 运行测试 "raw leaves node"
    t.Run("raw leaves node", func(t *testing.T) {
        # 标记测试为并行执行
        t.Parallel()
        # 定义内容字符串
        contents := "This is RAW!"
        # 将内容添加到 IPFS，并使用原始叶子节点
        cid := node.IPFSAddStr(contents, "--raw-leaves")
        # 断言 IPFS 获取的内容与原始内容相等
        assert.Equal(t, contents, client.Get("/ipfs/"+cid).Body)
    })

    # 运行测试 "compact blocks"
    t.Run("compact blocks", func(t *testing.T) {
        # 标记测试为并行执行
        t.Parallel()
        # 定义两个压缩块的十六进制字符串
        block1 := "\x0a\x09\x08\x02\x12\x03\x66\x6f\x6f\x18\x03"
        block2 := "\x0a\x04\x08\x02\x18\x06\x12\x24\x0a\x22\x12\x20\xcf\x92\xfd\xef\xcd\xc3\x4c\xac\x00\x9c" +
            "\x8b\x05\xeb\x66\x2b\xe0\x61\x8d\xb9\xde\x55\xec\xd4\x27\x85\xe9\xec\x67\x12\xf8\xdf\x65" +
            "\x12\x24\x0a\x22\x12\x20\xcf\x92\xfd\xef\xcd\xc3\x4c\xac\x00\x9c\x8b\x05\xeb\x66\x2b\xe0" +
            "\x61\x8d\xb9\xde\x55\xec\xd4\x27\x85\xe9\xec\x67\x12\xf8\xdf\x65"
        # 将第一个块添加到 IPFS
        node.PipeStrToIPFS(block1, "block", "put")
        # 将第二个块添加到 IPFS，并指定 CID 编码格式为 dag-pb
        block2CID := node.PipeStrToIPFS(block2, "block", "put", "--cid-codec=dag-pb").Stdout.Trimmed()
        # 获取 IPFS 中指定 CID 的内容
        resp := client.Get("/ipfs/" + block2CID)
        # 断言响应状态码为 200
        assert.Equal(t, 200, resp.StatusCode)
        # 断言响应内容为 "foofoo"
        assert.Equal(t, "foofoo", resp.Body)
    })

    # 运行测试 "verify gateway file"
    t.Run("verify gateway file", func(t *testing.T) {
        # 标记测试为并行执行
        t.Parallel()
        # 使用正则表达式匹配节点守护进程的标准输出，提取网关服务器地址
        r := regexp.MustCompile(`Gateway server listening on (?P<addr>.+)\s`)
        matches := r.FindStringSubmatch(node.Daemon.Stdout.String())
        ma, err := multiaddr.NewMultiaddr(matches[1])
        require.NoError(t, err)
        netAddr, err := manet.ToNetAddr(ma)
        require.NoError(t, err)
        expURL := "http://" + netAddr.String()
        # 读取节点目录下的 "gateway" 文件内容
        b, err := os.ReadFile(filepath.Join(node.Dir, "gateway"))
        require.NoError(t, err)
        # 断言读取的网关地址与期望的地址相等
        assert.Equal(t, expURL, string(b))
    })
    # 运行测试函数，验证在未指定情况下网关文件是否可拨号
    t.Run("verify gateway file diallable while on unspecified", func(t *testing.T) {
        # 标记测试函数可以并行执行
        t.Parallel()
        # 创建新的节点并初始化
        node := harness.NewT(t).NewNode().Init()
        # 更新配置，设置网关地址为指定的IP和端口
        node.UpdateConfig(func(cfg *config.Config) {
            cfg.Addresses.Gateway = config.Strings{"/ip4/127.0.0.1/tcp/32563"}
        })
        # 启动守护进程
        node.StartDaemon()

        # 读取网关文件内容
        b, err := os.ReadFile(filepath.Join(node.Dir, "gateway"))
        require.NoError(t, err)

        # 断言网关地址是否符合预期
        assert.Equal(t, "http://127.0.0.1:32563", string(b))
    })
    t.Run("NoFetch", func(t *testing.T) {
        // 运行测试用例 "NoFetch"
        t.Parallel()
        // 并行执行测试用例
        nodes := harness.NewT(t).NewNodes(2).Init()
        // 创建包含两个节点的测试环境
        node1 := nodes[0]
        // 获取第一个节点
        node2 := nodes[1]
        // 获取第二个节点

        node1.UpdateConfig(func(cfg *config.Config) {
            // 更新第一个节点的配置，设置网关不进行数据获取
            cfg.Gateway.NoFetch = true
        })

        node2PeerID, err := peer.ToCid(node2.PeerID()).StringOfBase(multibase.Base36)
        // 获取第二个节点的 PeerID，并转换成指定进制的字符串
        assert.NoError(t, err)
        // 断言错误为空

        nodes.StartDaemons().Connect()
        // 启动节点并连接

        t.Run("not present", func(t *testing.T) {
            // 运行子测试用例 "not present"
            cidFoo := node2.IPFSAddStr("foo")
            // 在第二个节点上添加名为 "foo" 的内容，并获取其 CID

            t.Run("not present key from node 1", func(t *testing.T) {
                // 运行子测试用例 "not present key from node 1"
                t.Parallel()
                // 并行执行测试用例
                assert.Equal(t, 500, node1.GatewayClient().Get("/ipfs/"+cidFoo).StatusCode)
                // 断言从第一个节点的网关获取 "/ipfs/"+cidFoo 的状态码为 500
            })

            t.Run("not present IPNS key from node 1", func(t *testing.T) {
                // 运行子测试用例 "not present IPNS key from node 1"
                t.Parallel()
                // 并行执行测试用例
                assert.Equal(t, 500, node1.GatewayClient().Get("/ipns/"+node2PeerID).StatusCode)
                // 断言从第一个节点的网关获取 "/ipns/"+node2PeerID 的状态码为 500
            })
        })

        t.Run("present", func(t *testing.T) {
            // 运行子测试用例 "present"
            cidBar := node1.IPFSAddStr("bar")
            // 在第一个节点上添加名为 "bar" 的内容，并获取其 CID

            t.Run("present key from node 1", func(t *testing.T) {
                // 运行子测试用例 "present key from node 1"
                t.Parallel()
                // 并行执行测试用例
                assert.Equal(t, 200, node1.GatewayClient().Get("/ipfs/"+cidBar).StatusCode)
                // 断言从第一个节点的网关获取 "/ipfs/"+cidBar 的状态码为 200
            })

            t.Run("present IPNS key from node 1", func(t *testing.T) {
                // 运行子测试用例 "present IPNS key from node 1"
                t.Parallel()
                // 并行执行测试用例
                node2.IPFS("name", "publish", "/ipfs/"+cidBar)
                // 在第二个节点上发布 "/ipfs/"+cidBar 的内容
                assert.Equal(t, 200, node1.GatewayClient().Get("/ipns/"+node2PeerID).StatusCode)
                // 断言从第一个节点的网关获取 "/ipns/"+node2PeerID 的状态码为 200
            })
        })
    })

    })
    # 运行测试用例"DisableHTMLErrors"
    t.Run("DisableHTMLErrors", func(t *testing.T) {
        # 并行执行测试
        t.Parallel()

        # 运行子测试"Returns HTML error without DisableHTMLErrors, Accept contains text/html"
        t.Run("Returns HTML error without DisableHTMLErrors, Accept contains text/html", func(t *testing.T) {
            # 并行执行子测试
            t.Parallel()

            # 创建新的测试节点
            node := harness.NewT(t).NewNode().Init()
            # 启动守护进程
            node.StartDaemon()
            # 获取网关客户端
            client := node.GatewayClient()

            # 发送带有"Accept"头部为"text/html"的GET请求
            res := client.Get("/ipfs/invalid-thing", func(r *http.Request) {
                r.Header.Set("Accept", "text/html")
            })
            # 断言状态码不等于200
            assert.NotEqual(t, http.StatusOK, res.StatusCode)
            # 断言响应头部包含"Content-Type"为"text/html"
            assert.Contains(t, res.Resp.Header.Get("Content-Type"), "text/html")
        })

        # 运行子测试"Does not return HTML error with DisableHTMLErrors enabled, and Accept contains text/html"
        t.Run("Does not return HTML error with DisableHTMLErrors enabled, and Accept contains text/html", func(t *testing.T) {
            # 并行执行子测试
            t.Parallel()

            # 创建新的测试节点
            node := harness.NewT(t).NewNode().Init()
            # 更新配置，启用DisableHTMLErrors
            node.UpdateConfig(func(cfg *config.Config) {
                cfg.Gateway.DisableHTMLErrors = config.True
            })
            # 启动守护进程
            node.StartDaemon()
            # 获取网关客户端
            client := node.GatewayClient()

            # 发送带有"Accept"头部为"text/html"的GET请求
            res := client.Get("/ipfs/invalid-thing", func(r *http.Request) {
                r.Header.Set("Accept", "text/html")
            })
            # 断言状态码不等于200
            assert.NotEqual(t, http.StatusOK, res.StatusCode)
            # 断言响应头部不包含"Content-Type"为"text/html"
            assert.NotContains(t, res.Resp.Header.Get("Content-Type"), "text/html")
        })
    })
# 闭合前面的函数定义
```