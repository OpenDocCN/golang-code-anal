# `kubo\test\cli\delegated_routing_v1_http_client_test.go`

```go
package cli

import (
    "net/http"  // 导入处理 HTTP 请求的包
    "net/http/httptest"  // 导入用于创建 HTTP 测试服务器的包
    "testing"  // 导入测试包

    "github.com/ipfs/kubo/config"  // 导入配置包
    "github.com/ipfs/kubo/test/cli/harness"  // 导入测试工具包
    . "github.com/ipfs/kubo/test/cli/testutils"  // 导入测试工具包
    "github.com/stretchr/testify/assert"  // 导入断言包
)

func TestHTTPDelegatedRouting(t *testing.T) {
    t.Parallel()  // 并行执行测试用例
    node := harness.NewT(t).NewNode().Init().StartDaemon()  // 创建并初始化节点

    fakeServer := func(contentType string, resp ...string) *httptest.Server {  // 定义一个模拟服务器函数
        return httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {  // 创建一个 HTTP 测试服务器
            w.Header().Set("Content-Type", contentType)  // 设置响应头的 Content-Type
            for _, r := range resp {  // 遍历响应内容
                _, err := w.Write([]byte(r))  // 写入响应内容
                if err != nil {  // 如果写入出错
                    panic(err)  // 抛出异常
                }
            }
        }))
    }

    findProvsCID := "baeabep4vu3ceru7nerjjbk37sxb7wmftteve4hcosmyolsbsiubw2vr6pqzj6mw7kv6tbn6nqkkldnklbjgm5tzbi4hkpkled4xlcr7xz4bq"  // 设置查找提供者的 CID
    provs := []string{"12D3KooWAobjw92XDcnQ1rRmRJDA3zAQpdPYUpZKrJxH6yccSpje", "12D3KooWARYacCc6eoCqvsS9RW9MA2vo51CV75deoiqssx3YgyYJ"}  // 设置提供者列表

    t.Run("default routing config has no routers defined", func(t *testing.T) {
        assert.Nil(t, node.ReadConfig().Routing.Routers)  // 断言默认路由配置中没有定义路由器
    })

    t.Run("no routers means findprovs returns no results", func(t *testing.T) {
        res := node.IPFS("routing", "findprovs", findProvsCID).Stdout.String()  // 调用 IPFS 命令执行查找提供者操作
        assert.Empty(t, res)  // 断言结果为空
    })

    t.Run("no routers means findprovs returns no results", func(t *testing.T) {
        res := node.IPFS("routing", "findprovs", findProvsCID).Stdout.String()  // 调用 IPFS 命令执行查找提供者操作
        assert.Empty(t, res)  // 断言结果为空
    })

    node.StopDaemon()  // 停止节点的守护进程
}
    # 测试缺少方法参数是否会导致守护进程失败
    t.Run("missing method params make the daemon fail", func(t *testing.T) {
        # 更新配置，设置路由类型为自定义
        node.UpdateConfig(func(cfg *config.Config) {
            cfg.Routing.Type = config.NewOptionalString("custom")
            # 设置路由方法
            cfg.Routing.Methods = config.Methods{
                "find-peers":     {RouterName: "TestDelegatedRouter"},
                "find-providers": {RouterName: "TestDelegatedRouter"},
                "get-ipns":       {RouterName: "TestDelegatedRouter"},
                "provide":        {RouterName: "TestDelegatedRouter"},
            }
        })
        # 运行 IPFS 守护进程
        res := node.RunIPFS("daemon")
        # 断言守护进程退出码为1
        assert.Equal(t, 1, res.ExitErr.ProcessState.ExitCode())
        # 断言标准错误输出包含特定信息
        assert.Contains(
            t,
            res.Stderr.String(),
            `method name "put-ipns" is missing from Routing.Methods config param`,
        )
    })

    # 测试错误的方法是否会导致守护进程失败
    t.Run("having wrong methods makes daemon fail", func(t *testing.T) {
        # 更新配置，设置路由类型为自定义
        node.UpdateConfig(func(cfg *config.Config) {
            cfg.Routing.Type = config.NewOptionalString("custom")
            # 设置路由方法
            cfg.Routing.Methods = config.Methods{
                "find-peers":     {RouterName: "TestDelegatedRouter"},
                "find-providers": {RouterName: "TestDelegatedRouter"},
                "get-ipns":       {RouterName: "TestDelegatedRouter"},
                "provide":        {RouterName: "TestDelegatedRouter"},
                "put-ipns":       {RouterName: "TestDelegatedRouter"},
                "NOT_SUPPORTED":  {RouterName: "TestDelegatedRouter"},
            }
        })
        # 运行 IPFS 守护进程
        res := node.RunIPFS("daemon")
        # 断言守护进程退出码为1
        assert.Equal(t, 1, res.ExitErr.ProcessState.ExitCode())
        # 断言标准错误输出包含特定信息
        assert.Contains(
            t,
            res.Stderr.String(),
            `method name "NOT_SUPPORTED" is not a supported method on Routing.Methods config param`,
        )
    })
    t.Run("adding HTTP delegated routing endpoint to Routing.Routers config works", func(t *testing.T) {
        // 创建一个假的服务器，返回指定的 JSON 数据
        server := fakeServer("application/json", ToJSONStr(JSONObj{
            "Providers": []JSONObj{
                {
                    "Schema":   "bitswap", // Legacy bitswap schema.
                    "Protocol": "transport-bitswap",
                    "ID":       provs[1],
                    "Addrs":    []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/tcp/4002"},
                },
                {
                    "Schema":    "peer",
                    "Protocols": []string{"transport-bitswap"},
                    "ID":        provs[0],
                    "Addrs":     []string{"/ip4/0.0.0.0/tcp/4001", "/ip4.0.0.0/tcp/4002"},
                },
            },
        }))
        // 在测试结束时关闭服务器
        t.Cleanup(server.Close)

        // 设置 IPFS 路由的类型为 custom
        node.IPFS("config", "Routing.Type", "custom")
        // 向 Routing.Routers 配置中添加 HTTP 委托路由端点
        node.IPFS("config", "Routing.Routers.TestDelegatedRouter", "--json", ToJSONStr(JSONObj{
            "Type": "http",
            "Parameters": JSONObj{
                "Endpoint": server.URL,
            },
        }))
        // 设置 IPFS 路由的方法
        node.IPFS("config", "Routing.Methods", "--json", ToJSONStr(JSONObj{
            "find-peers":     JSONObj{"RouterName": "TestDelegatedRouter"},
            "find-providers": JSONObj{"RouterName": "TestDelegatedRouter"},
            "get-ipns":       JSONObj{"RouterName": "TestDelegatedRouter"},
            "provide":        JSONObj{"RouterName": "TestDelegatedRouter"},
            "put-ipns":       JSONObj{"RouterName": "TestDelegatedRouter"},
        }))

        // 获取 TestDelegatedRouter 参数中的端点
        res := node.IPFS("config", "Routing.Routers.TestDelegatedRouter.Parameters.Endpoint")
        // 断言获取的端点与服务器的 URL 相等
        assert.Equal(t, res.Stdout.Trimmed(), server.URL)

        // 启动 IPFS 守护进程
        node.StartDaemon()
        // 查询提供者
        res = node.IPFS("routing", "findprovs", findProvsCID)
        // 断言查询结果与预期的提供者相等
        assert.Equal(t, provs[1]+"\n"+provs[0], res.Stdout.Trimmed())
    })

    // 停止 IPFS 守护进程
    node.StopDaemon()
    # 在 Routing.Routers 配置中添加 HTTP 委托路由端点，以流式传输方式工作
    t.Run("adding HTTP delegated routing endpoint to Routing.Routers config works (streaming)", func(t *testing.T) {
        # 创建一个虚假的服务器，返回类型为 application/x-ndjson，内容为 JSON 对象
        server := fakeServer("application/x-ndjson", ToJSONStr(JSONObj{
            "Schema":    "peer",
            "Protocols": []string{"transport-bitswap"},
            "ID":        provs[0],
            "Addrs":     []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/tcp/4002"},
        }), ToJSONStr(JSONObj{
            "Schema":   "bitswap", // Legacy bitswap schema.
            "Protocol": "transport-bitswap",
            "ID":       provs[1],
            "Addrs":    []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/tcp/4002"},
        }))
        # 在测试结束时关闭虚假服务器
        t.Cleanup(server.Close)

        # 使用 IPFS 命令行工具配置 Routing.Routers.TestDelegatedRouter 的参数
        node.IPFS("config", "Routing.Routers.TestDelegatedRouter", "--json", ToJSONStr(JSONObj{
            "Type": "http",
            "Parameters": JSONObj{
                "Endpoint": server.URL,
            },
        }))

        # 获取配置中 Routing.Routers.TestDelegatedRouter.Parameters.Endpoint 的值
        res := node.IPFS("config", "Routing.Routers.TestDelegatedRouter.Parameters.Endpoint")
        # 断言获取的值与服务器的 URL 相等
        assert.Equal(t, res.Stdout.Trimmed(), server.URL)

        # 启动 IPFS 守护进程
        node.StartDaemon()
        # 使用 IPFS 命令行工具查询路由信息，查找给定 CID 的提供者
        res = node.IPFS("routing", "findprovs", findProvsCID)
        # 断言获取的提供者信息与预期值相等
        assert.Equal(t, provs[0]+"\n"+provs[1], res.Stdout.Trimmed())
    })

    # HTTP 客户端应该发出 OpenCensus 指标
    t.Run("HTTP client should emit OpenCensus metrics", func(t *testing.T) {
        # 发送 HTTP GET 请求获取指标数据
        resp := node.APIClient().Get("/debug/metrics/prometheus")
        # 断言响应体中包含指定的指标名称
        assert.Contains(t, resp.Body, "routing_http_client_length_count")
    })
# 闭合前面的函数定义
```