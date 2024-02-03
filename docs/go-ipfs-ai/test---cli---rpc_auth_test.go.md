# `kubo\test\cli\rpc_auth_test.go`

```go
package cli

import (
    "net/http"
    "testing"

    "github.com/ipfs/kubo/client/rpc/auth"
    "github.com/ipfs/kubo/config"
    "github.com/ipfs/kubo/test/cli/harness"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

const rpcDeniedMsg = "Kubo RPC Access Denied: Please provide a valid authorization token as defined in the API.Authorizations configuration."

func TestRPCAuth(t *testing.T) {
    t.Parallel()

    // 创建并启动受保护的节点
    makeAndStartProtectedNode := func(t *testing.T, authorizations map[string]*config.RPCAuthScope) *harness.Node {
        // 将测试节点的授权信息添加到授权映射中
        authorizations["test-node-starter"] = &config.RPCAuthScope{
            AuthSecret:   "bearer:test-node-starter",
            AllowedPaths: []string{"/api/v0"},
        }

        // 创建新的测试节点
        node := harness.NewT(t).NewNode().Init()
        // 更新节点配置
        node.UpdateConfig(func(cfg *config.Config) {
            cfg.API.Authorizations = authorizations
        })
        // 使用授权令牌启动守护进程
        node.StartDaemonWithAuthorization("Bearer test-node-starter")
        // 返回节点
        return node
    }
    // 创建一个函数，用于生成 HTTP 测试函数，接受认证密钥和头部信息作为参数
    makeHTTPTest := func(authSecret, header string) func(t *testing.T) {
        // 返回一个测试函数
        return func(t *testing.T) {
            // 标记该测试函数可以并行执行
            t.Parallel()
            // 记录认证密钥和头部信息到日志
            t.Log(authSecret, header)

            // 创建并启动受保护节点，传入认证作用域配置
            node := makeAndStartProtectedNode(t, map[string]*config.RPCAuthScope{
                "userA": {
                    AuthSecret:   authSecret,
                    AllowedPaths: []string{"/api/v0/id"},
                },
            })

            // 获取节点的 API 客户端
            apiClient := node.APIClient()
            // 设置 API 客户端的 HTTP 客户端，使用自定义的授权传输器
            apiClient.Client = &http.Client{
                Transport: auth.NewAuthorizedRoundTripper(header, http.DefaultTransport),
            }

            // 使用有效令牌可以访问 /id
            resp := apiClient.Post("/api/v0/id", nil)
            // 断言响应状态码为 200
            assert.Equal(t, 200, resp.StatusCode)

            // 但不能访问 /config/show
            resp = apiClient.Post("/api/v0/config/show", nil)
            // 断言响应状态码为 403
            assert.Equal(t, 403, resp.StatusCode)

            // 创建发送无效访问令牌的客户端
            invalidApiClient := node.APIClient()
            // 设置无效令牌的客户端的 HTTP 客户端，使用自定义的授权传输器
            invalidApiClient.Client = &http.Client{
                Transport: auth.NewAuthorizedRoundTripper("Bearer invalid", http.DefaultTransport),
            }

            // 使用无效令牌不能访问 /id
            errResp := invalidApiClient.Post("/api/v0/id", nil)
            // 断言错误响应状态码为 403
            assert.Equal(t, 403, errResp.StatusCode)

            // 停止节点的守护进程
            node.StopDaemon()
        }
    }
    // 创建一个测试函数，该函数接受一个认证密钥并返回一个测试函数
    makeCLITest := func(authSecret string) func(t *testing.T) {
        return func(t *testing.T) {
            t.Parallel()
    
            // 创建并启动一个受保护的节点，设置用户A的认证范围为访问"/api/v0/id"
            node := makeAndStartProtectedNode(t, map[string]*config.RPCAuthScope{
                "userA": {
                    AuthSecret:   authSecret,
                    AllowedPaths: []string{"/api/v0/id"},
                },
            })
    
            // 可以访问'ipfs id'
            resp := node.RunIPFS("id", "--api-auth", authSecret)
            require.NoError(t, resp.Err)
    
            // 但是不能访问'ipfs config show'
            resp = node.RunIPFS("config", "show", "--api-auth", authSecret)
            require.Error(t, resp.Err)
            require.Contains(t, resp.Stderr.String(), rpcDeniedMsg)
    
            // 停止守护进程
            node.StopDaemon()
        }
    }
    
    // 遍历测试用例，每个测试用例包括名称、认证密钥和标头
    for _, testCase := range []struct {
        name       string
        authSecret string
        header     string
    }{
        {"Bearer (no type)", "myToken", "Bearer myToken"},
        {"Bearer", "bearer:myToken", "Bearer myToken"},
        {"Basic (user:pass)", "basic:user:pass", "Basic dXNlcjpwYXNz"},
        {"Basic (encoded)", "basic:dXNlcjpwYXNz", "Basic dXNlcjpwYXNz"},
    } {
        // 运行CLI上的AllowedPaths测试和HTTP上的AllowedPaths测试
        t.Run("AllowedPaths on CLI "+testCase.name, makeCLITest(testCase.authSecret))
        t.Run("AllowedPaths on HTTP "+testCase.name, makeHTTPTest(testCase.authSecret, testCase.header))
    }
    # 测试用例：当AllowedPaths设置为/api/v0时，给予完全访问权限
    t.Run("AllowedPaths set to /api/v0 Gives Full Access", func(t *testing.T) {
        t.Parallel()

        # 创建并启动受保护节点，设置用户A的RPC权限范围
        node := makeAndStartProtectedNode(t, map[string]*config.RPCAuthScope{
            "userA": {
                AuthSecret:   "bearer:userAToken",
                AllowedPaths: []string{"/api/v0"},
            },
        })

        # 创建API客户端，并设置授权信息
        apiClient := node.APIClient()
        apiClient.Client = &http.Client{
            Transport: auth.NewAuthorizedRoundTripper("Bearer userAToken", http.DefaultTransport),
        }

        # 发送POST请求到指定路径，验证响应状态码为200
        resp := apiClient.Post("/api/v0/id", nil)
        assert.Equal(t, 200, resp.StatusCode)

        # 停止节点
        node.StopDaemon()
    })

    # 测试用例：当API.Authorizations设置为nil时，禁用授权头检查
    t.Run("API.Authorizations set to nil disables Authorization header check", func(t *testing.T) {
        t.Parallel()

        # 创建新节点并初始化
        node := harness.NewT(t).NewNode().Init()
        # 更新配置，将API.Authorizations设置为nil
        node.UpdateConfig(func(cfg *config.Config) {
            cfg.API.Authorizations = nil
        })
        # 启动节点
        node.StartDaemon()

        # 创建API客户端，并发送POST请求到指定路径，验证响应状态码为200
        apiClient := node.APIClient()
        resp := apiClient.Post("/api/v0/id", nil)
        assert.Equal(t, 200, resp.StatusCode)

        # 停止节点
        node.StopDaemon()
    })

    # 测试用例：当API.Authorizations设置为空map时，禁用授权头检查
    t.Run("API.Authorizations set to empty map disables Authorization header check", func(t *testing.T) {
        t.Parallel()

        # 创建新节点并初始化
        node := harness.NewT(t).NewNode().Init()
        # 更新配置，将API.Authorizations设置为空map
        node.UpdateConfig(func(cfg *config.Config) {
            cfg.API.Authorizations = map[string]*config.RPCAuthScope{}
        })
        # 启动节点
        node.StartDaemon()

        # 创建API客户端，并发送POST请求到指定路径，验证响应状态码为200
        apiClient := node.APIClient()
        resp := apiClient.Post("/api/v0/id", nil)
        assert.Equal(t, 200, resp.StatusCode)

        # 停止节点
        node.StopDaemon()
    })
# 闭合前面的函数定义
```