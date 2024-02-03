# `kubo\test\cli\pinning_remote_test.go`

```go
package cli

import (
    "errors"  // 引入 errors 包，用于处理错误
    "fmt"  // 引入 fmt 包，用于格式化输出
    "net"  // 引入 net 包，用于网络操作
    "net/http"  // 引入 http 包，用于处理 HTTP 请求
    "testing"  // 引入 testing 包，用于编写测试
    "time"  // 引入 time 包，用于处理时间

    "github.com/google/uuid"  // 引入第三方库 github.com/google/uuid
    "github.com/ipfs/kubo/test/cli/harness"  // 引入自定义包 harness
    "github.com/ipfs/kubo/test/cli/testutils"  // 引入自定义包 testutils
    "github.com/ipfs/kubo/test/cli/testutils/pinningservice"  // 引入自定义包 pinningservice
    "github.com/stretchr/testify/assert"  // 引入第三方库 github.com/stretchr/testify/assert
    "github.com/stretchr/testify/require"  // 引入第三方库 github.com/stretchr/testify/require
    "github.com/tidwall/gjson"  // 引入第三方库 github.com/tidwall/gjson
    "github.com/tidwall/sjson"  // 引入第三方库 github.com/tidwall/sjson
)

func runPinningService(t *testing.T, authToken string) (*pinningservice.PinningService, string) {
    svc := pinningservice.New()  // 创建 PinningService 对象
    router := pinningservice.NewRouter(authToken, svc)  // 创建路由对象
    server := &http.Server{Handler: router}  // 创建 HTTP 服务器对象
    listener, err := net.Listen("tcp", "127.0.0.1:0")  // 监听指定地址和端口
    require.NoError(t, err)  // 断言错误为空
    go func() {
        err := server.Serve(listener)  // 启动 HTTP 服务器
        if err != nil && !errors.Is(err, net.ErrClosed) && !errors.Is(err, http.ErrServerClosed) {
            t.Logf("Serve error: %s", err)  // 输出错误信息
        }
    }()
    t.Cleanup(func() { listener.Close() })  // 在测试结束时关闭监听器

    return svc, fmt.Sprintf("http://%s/api/v1", listener.Addr().String())  // 返回 PinningService 对象和服务器地址
}

func TestRemotePinning(t *testing.T) {
    t.Parallel()  // 并行执行测试
    authToken := "testauthtoken"  // 设置测试用的认证令牌

    })

    // Pinning.RemoteServices includes API.Key, so we give it the same treatment
    // as Identity,PrivKey to prevent exposing it on the network
}
    t.Run("access token security", func(t *testing.T) {
        // 运行测试用例，检查访问令牌的安全性
        t.Parallel()
        // 创建新的节点并初始化
        node := harness.NewT(t).NewNode().Init()
        // 使用 IPFS 命令添加远程固定服务
        node.IPFS("pin", "remote", "service", "add", "1", "http://example1.com", "testkey")
        // 运行 IPFS 命令获取配置信息
        res := node.RunIPFS("config", "Pinning")
        // 断言结果的退出码为1
        assert.Equal(t, 1, res.ExitCode())
        // 断言标准错误输出包含指定内容
        assert.Contains(t, res.Stderr.String(), "cannot show or change pinning services credentials")
        // 断言标准输出不包含指定内容
        assert.NotContains(t, res.Stdout.String(), "testkey")

        // 运行 IPFS 命令获取指定配置信息
        res = node.RunIPFS("config", "Pinning.RemoteServices.1.API.Key")
        // 断言结果的退出码为1
        assert.Equal(t, 1, res.ExitCode())
        // 断言标准错误输出包含指定内容
        assert.Contains(t, res.Stderr.String(), "cannot show or change pinning services credentials")
        // 断言标准输出不包含指定内容
        assert.NotContains(t, res.Stdout.String(), "testkey")

        // 运行 IPFS 命令获取完整配置信息
        configShow := node.RunIPFS("config", "show").Stdout.String()
        // 断言标准输出不包含指定内容
        assert.NotContains(t, configShow, "testkey")

        t.Run("re-injecting config with 'ipfs config replace' preserves the API keys", func(t *testing.T) {
            // 将完整配置信息写入文件
            node.WriteBytes("config-show", []byte(configShow))
            // 使用 IPFS 命令替换配置信息
            node.IPFS("config", "replace", "config-show")
            // 断言节点配置文件中包含指定内容
            assert.Contains(t, node.ReadFile(node.ConfigFile()), "testkey")
        })

        t.Run("injecting config with 'ipfs config replace' with API keys returns an error", func(t *testing.T) {
            // 删除 Identity.PrivKey 以确保错误由 Pinning.RemoteServices 触发
            configJSON := MustVal(sjson.Delete(configShow, "Identity.PrivKey"))
            // 设置 Pinning.RemoteServices.1.API.Key 的值为 "testkey"
            configJSON = MustVal(sjson.Set(configJSON, "Pinning.RemoteServices.1.API.Key", "testkey"))
            // 将新的配置信息写入文件
            node.WriteBytes("new-config", []byte(configJSON))
            // 运行 IPFS 命令替换配置信息
            res := node.RunIPFS("config", "replace", "new-config")
            // 断言结果的退出码为1
            assert.Equal(t, 1, res.ExitCode())
            // 断言标准错误输出包含指定内容
            assert.Contains(t, res.Stderr.String(), "cannot change remote pinning services api info with `config replace`")
        })
    })
    t.Run("pin remote service ls --stat", func(t *testing.T) {
        // 运行测试用例，测试远程服务列表的状态
        t.Parallel()
        // 创建一个新的节点，并初始化，启动守护进程
        node := harness.NewT(t).NewNode().Init().StartDaemon()
        // 运行 pinningService 函数，返回授权令牌
        _, svcURL := runPinningService(t, authToken)

        // 使用 IPFS 命令将服务添加到远程固定列表
        node.IPFS("pin", "remote", "service", "add", "svc", svcURL, authToken)
        // 使用 IPFS 命令将无效服务添加到远程固定列表
        node.IPFS("pin", "remote", "service", "add", "invalid-svc", svcURL+"/invalidpath", authToken)

        // 获取远程服务列表的状态
        res := node.IPFS("pin", "remote", "service", "ls", "--stat")
        // 断言结果中包含特定的字符串
        assert.Contains(t, res.Stdout.String(), " 0/0/0/0")

        // 获取远程服务列表的状态，并以 JSON 格式编码
        stats := node.IPFS("pin", "remote", "service", "ls", "--stat", "--enc=json").Stdout.String()
        // 断言结果等于特定值
        assert.Equal(t, "valid", gjson.Get(stats, `RemoteServices.#(Service == "svc").Stat.Status`).Str)
        assert.Equal(t, "invalid", gjson.Get(stats, `RemoteServices.#(Service == "invalid-svc").Stat.Status`).Str)

        // 没有 --stat 选项时不返回状态对象
        t.Run("no --stat returns no stat obj", func(t *testing.T) {
            // 运行测试用例，测试没有 --stat 选项时不返回状态对象
            res := node.IPFS("pin", "remote", "service", "ls", "--enc=json")
            // 断言结果中不存在特定的 JSON 路径
            assert.False(t, gjson.Get(res.Stdout.String(), `RemoteServices.#(Service == "svc").Stat`).Exists())
        })
    })

    t.Run("adding service with invalid URL fails", func(t *testing.T) {
        // 运行测试用例，测试使用无效 URL 添加服务失败
        t.Parallel()
        // 创建一个新的节点，并初始化，启动守护进程
        node := harness.NewT(t).NewNode().Init().StartDaemon()

        // 运行 IPFS 命令，尝试添加使用无效 URL 的服务
        res := node.RunIPFS("pin", "remote", "service", "add", "svc", "invalid-service.example.com", "key")
        // 断言结果的退出码为特定值
        assert.Equal(t, 1, res.ExitCode())
        // 断言结果中包含特定的字符串
        assert.Contains(t, res.Stderr.String(), "service endpoint must be a valid HTTP URL")

        // 运行 IPFS 命令，尝试添加使用无效 URL 的服务
        res = node.RunIPFS("pin", "remote", "service", "add", "svc", "xyz://invalid-service.example.com", "key")
        // 断言结果的退出码为特定值
        assert.Equal(t, 1, res.ExitCode())
        // 断言结果中包含特定的字符串
        assert.Contains(t, res.Stderr.String(), "service endpoint must be a valid HTTP URL")
    })
    # 测试未经授权的固定服务调用失败
    t.Run("unauthorized pinning service calls fail", func(t *testing.T) {
        t.Parallel()
        # 创建新的节点并初始化，启动守护进程
        node := harness.NewT(t).NewNode().Init().StartDaemon()
        # 运行固定服务
        _, svcURL := runPinningService(t, authToken)

        # 使用IPFS命令进行远程固定服务调用
        node.IPFS("pin", "remote", "service", "add", "svc", svcURL, "othertoken")

        # 运行IPFS命令，列出远程固定服务
        res := node.RunIPFS("pin", "remote", "ls", "--service=svc")
        # 断言结果的退出码为1
        assert.Equal(t, 1, res.ExitCode())
        # 断言结果的标准错误输出包含"access denied"
        assert.Contains(t, res.Stderr.String(), "access denied")
    })

    # 测试当路径错误时固定服务调用失败
    t.Run("pinning service calls fail when there is a wrong path", func(t *testing.T) {
        t.Parallel()
        # 创建新的节点并初始化，启动守护进程
        node := harness.NewT(t).NewNode().Init().StartDaemon()
        # 运行固定服务
        _, svcURL := runPinningService(t, authToken)
        # 使用IPFS命令进行远程固定服务调用
        node.IPFS("pin", "remote", "service", "add", "svc", svcURL+"/invalid-path", authToken)

        # 运行IPFS命令，列出远程固定服务
        res := node.RunIPFS("pin", "remote", "ls", "--service=svc")
        # 断言结果的退出码为1
        assert.Equal(t, 1, res.ExitCode())
        # 断言结果的标准错误输出包含"404"
        assert.Contains(t, res.Stderr.String(), "404")
    })

    # 测试当DNS解析失败时固定服务调用失败
    t.Run("pinning service calls fail when DNS resolution fails", func(t *testing.T) {
        t.Parallel()
        # 创建新的节点并初始化，启动守护进程
        node := harness.NewT(t).NewNode().Init().StartDaemon()
        # 使用IPFS命令进行远程固定服务调用
        node.IPFS("pin", "remote", "service", "add", "svc", "https://invalid-service.example.com", authToken)

        # 运行IPFS命令，列出远程固定服务
        res := node.RunIPFS("pin", "remote", "ls", "--service=svc")
        # 断言结果的退出码为1
        assert.Equal(t, 1, res.ExitCode())
        # 断言结果的标准错误输出包含"no such host"
        assert.Contains(t, res.Stderr.String(), "no such host")
    })

    # 测试远程固定服务删除
    t.Run("pin remote service rm", func(t *testing.T) {
        t.Parallel()
        # 创建新的节点并初始化，启动守护进程
        node := harness.NewT(t).NewNode().Init().StartDaemon()
        # 使用IPFS命令进行远程固定服务添加
        node.IPFS("pin", "remote", "service", "add", "svc", "https://example.com", authToken)
        # 使用IPFS命令进行远程固定服务删除
        node.IPFS("pin", "remote", "service", "rm", "svc")
        # 运行IPFS命令，列出远程固定服务
        res := node.IPFS("pin", "remote", "service", "ls")
        # 断言结果的标准输出不包含"svc"
        assert.NotContains(t, res.Stdout.String(), "svc")
    })

    })
    t.Run("'ipfs pin remote add' shows a warning message when offline", func(t *testing.T) {
        // 在离线状态下，'ipfs pin remote add' 显示警告消息
        t.Parallel()
        // 创建一个新的节点并初始化
        node := harness.NewT(t).NewNode().Init()
        // 运行固定服务，并返回服务的 URL
        _, svcURL := runPinningService(t, authToken)
        // 使用 IPFS 命令将服务添加到远程固定服务
        node.IPFS("pin", "remote", "service", "add", "svc", svcURL, authToken)

        // 生成随机的 1000 字节数据并将其添加到 IPFS，返回哈希值
        hash := node.IPFSAddStr(string(testutils.RandomBytes(1000)))
        // 使用 IPFS 命令将哈希值添加到远程固定服务，并在后台运行
        res := node.IPFS("pin", "remote", "add", "--service=svc", "--background", hash)
        // 预期的警告消息
        warningMsg := "WARNING: the local node is offline and remote pinning may fail if there is no other provider for this CID"
        // 断言结果中包含预期的警告消息
        assert.Contains(t, res.Stdout.String(), warningMsg)
    })
# 闭合前面的函数定义
```