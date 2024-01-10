# `kubo\core\corehttp\commands.go`

```
package corehttp

import (
    "errors" // 引入 errors 包，用于创建错误
    "fmt" // 引入 fmt 包，用于格式化输出
    "net" // 引入 net 包，用于网络操作
    "net/http" // 引入 net/http 包，用于 HTTP 服务
    "os" // 引入 os 包，用于操作系统功能
    "strconv" // 引入 strconv 包，用于字符串和基本数据类型之间的转换
    "strings" // 引入 strings 包，用于字符串操作

    cmds "github.com/ipfs/go-ipfs-cmds" // 引入 go-ipfs-cmds 包
    cmdsHttp "github.com/ipfs/go-ipfs-cmds/http" // 引入 go-ipfs-cmds/http 包
    version "github.com/ipfs/kubo" // 引入 kubo 包
    oldcmds "github.com/ipfs/kubo/commands" // 引入 kubo/commands 包
    config "github.com/ipfs/kubo/config" // 引入 kubo/config 包
    "github.com/ipfs/kubo/core" // 引入 kubo/core 包
    corecommands "github.com/ipfs/kubo/core/commands" // 引入 kubo/core/commands 包
    "go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp" // 引入 otelhttp 包
)

var errAPIVersionMismatch = errors.New("api version mismatch") // 创建错误变量 errAPIVersionMismatch

const (
    originEnvKey          = "API_ORIGIN" // 定义常量 originEnvKey
    originEnvKeyDeprecate = `You are using the ` + originEnvKey + `ENV Variable.
This functionality is deprecated, and will be removed in future versions.
Instead, try either adding headers to the config, or passing them via
cli arguments:

    ipfs config API.HTTPHeaders --json '{"Access-Control-Allow-Origin": ["*"]}'
    ipfs daemon
` // 定义常量 originEnvKeyDeprecate
)

// APIPath is the path at which the API is mounted.
const APIPath = "/api/v0" // 定义常量 APIPath

var defaultLocalhostOrigins = []string{ // 定义切片变量 defaultLocalhostOrigins
    "http://127.0.0.1:<port>",
    "https://127.0.0.1:<port>",
    "http://[::1]:<port>",
    "https://[::1]:<port>",
    "http://localhost:<port>",
    "https://localhost:<port>",
}

var companionBrowserExtensionOrigins = []string{ // 定义切片变量 companionBrowserExtensionOrigins
    "chrome-extension://nibjojkomfdiaoajekhjakgkdhaomnch", // ipfs-companion
    "chrome-extension://hjoieblefckbooibpepigmacodalfndh", // ipfs-companion-beta
}

func addCORSFromEnv(c *cmdsHttp.ServerConfig) { // 定义函数 addCORSFromEnv，接收 cmdsHttp.ServerConfig 类型参数
    origin := os.Getenv(originEnvKey) // 获取环境变量 originEnvKey 的值
    if origin != "" { // 如果 origin 不为空
        log.Warn(originEnvKeyDeprecate) // 输出警告信息
        c.AppendAllowedOrigins(origin) // 将 origin 添加到允许的来源列表中
    }
}

func addHeadersFromConfig(c *cmdsHttp.ServerConfig, nc *config.Config) { // 定义函数 addHeadersFromConfig，接收两个参数
    log.Info("Using API.HTTPHeaders:", nc.API.HTTPHeaders) // 输出日志信息
    if acao := nc.API.HTTPHeaders[cmdsHttp.ACAOrigin]; acao != nil { // 如果存在 ACAOrigin
        c.SetAllowedOrigins(acao...) // 设置允许的来源
    }
    if acam := nc.API.HTTPHeaders[cmdsHttp.ACAMethods]; acam != nil { // 如果存在 ACAMethods
        c.SetAllowedMethods(acam...) // 设置允许的方法
    }
    // 遍历 nc.API.HTTPHeaders[cmdsHttp.ACACredentials] 中的值，并设置是否允许凭据
    for _, v := range nc.API.HTTPHeaders[cmdsHttp.ACACredentials] {
        c.SetAllowCredentials(strings.ToLower(v) == "true")
    }

    // 初始化 c.Headers，长度为 nc.API.HTTPHeaders 的长度加一
    c.Headers = make(map[string][]string, len(nc.API.HTTPHeaders)+1)

    // 复制这些值，因为配置是共享的，这个函数会在多个地方同时调用。在原地更新这些值是有竞争条件的。
    for h, v := range nc.API.HTTPHeaders {
        // 将 h 转换为标准的 HTTP 头部键名
        h = http.CanonicalHeaderKey(h)
        switch h {
        // 这些由 CORs 库处理
        case cmdsHttp.ACAOrigin, cmdsHttp.ACAMethods, cmdsHttp.ACACredentials:
            // these are handled by the CORs library.
        default:
            // 将 HTTP 头部键值对添加到 c.Headers 中
            c.Headers[h] = v
        }
    }
    // 设置服务器的 HTTP 头部信息
    c.Headers["Server"] = []string{"kubo/" + version.CurrentVersionNumber}
// 添加默认的跨域资源共享（CORS）配置
func addCORSDefaults(c *cmdsHttp.ServerConfig) {
    // 始终将某些来源列入白名单
    c.AppendAllowedOrigins(defaultLocalhostOrigins...)
    c.AppendAllowedOrigins(companionBrowserExtensionOrigins...)

    // 默认情况下，使用 GET、PUT、POST 方法
    if len(c.AllowedMethods()) == 0 {
        c.SetAllowedMethods(http.MethodGet, http.MethodPost, http.MethodPut)
    }
}

// 修改跨域资源共享（CORS）变量
func patchCORSVars(c *cmdsHttp.ServerConfig, addr net.Addr) {
    // 从地址中获取端口，可能是 IP6 地址
    // TODO: 应该从 multiaddrs 中获取并从中派生端口
    port := ""
    if tcpaddr, ok := addr.(*net.TCPAddr); ok {
        port = strconv.Itoa(tcpaddr.Port)
    } else if udpaddr, ok := addr.(*net.UDPAddr); ok {
        port = strconv.Itoa(udpaddr.Port)
    }

    // 我们正在监听带有端口的 TCP/UDP。("udp!?" 你说？是的... 这种情况确实存在...)
    oldOrigins := c.AllowedOrigins()
    newOrigins := make([]string, len(oldOrigins))
    for i, o := range oldOrigins {
        // TODO: 允许替换 <host>。复杂，IP4 和 IP6 和主机名...
        if port != "" {
            o = strings.Replace(o, "<port>", port, -1)
        }
        newOrigins[i] = o
    }
    c.SetAllowedOrigins(newOrigins...)
}

// 命令选项
func commandsOption(cctx oldcmds.Context, command *cmds.Command, allowGet bool) ServeOption {
    # 定义一个函数，接受一个 IPFS 节点、一个网络监听器和一个 HTTP 服务多路复用器作为参数，返回一个 HTTP 服务多路复用器和一个错误
    return func(n *core.IpfsNode, l net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
        # 创建一个新的 HTTP 服务器配置对象
        cfg := cmdsHttp.NewServerConfig()
        # 设置是否允许 GET 请求
        cfg.AllowGet = allowGet
        # 定义允许的跨域请求方法为 POST
        corsAllowedMethods := []string{http.MethodPost}
        # 如果允许 GET 请求，则添加 GET 请求到允许的跨域请求方法中
        if allowGet {
            corsAllowedMethods = append(corsAllowedMethods, http.MethodGet)
        }

        # 设置允许的跨域请求方法
        cfg.SetAllowedMethods(corsAllowedMethods...)
        # 设置 API 路径
        cfg.APIPath = APIPath
        # 从节点的配置中获取配置信息
        rcfg, err := n.Repo.Config()
        # 如果获取配置信息失败，则返回空和错误
        if err != nil {
            return nil, err
        }

        # 从配置信息中添加头部信息到服务器配置对象中
        addHeadersFromConfig(cfg, rcfg)
        # 从环境变量中添加跨域配置到服务器配置对象中
        addCORSFromEnv(cfg)
        # 添加跨域默认配置到服务器配置对象中
        addCORSDefaults(cfg)
        # 从监听器地址中修补跨域变量到服务器配置对象中
        patchCORSVars(cfg, l.Addr())

        # 创建一个新的命令处理器，使用上下文、命令和服务器配置对象作为参数
        cmdHandler := cmdsHttp.NewHandler(&cctx, command, cfg)

        # 如果 API 配置中有授权信息，则转换为授权映射并添加到命令处理器中
        if len(rcfg.API.Authorizations) > 0 {
            authorizations := convertAuthorizationsMap(rcfg.API.Authorizations)
            cmdHandler = withAuthSecrets(authorizations, cmdHandler)
        }

        # 使用 OpenTelemetry 创建一个新的 HTTP 处理器，并添加到命令处理器中
        cmdHandler = otelhttp.NewHandler(cmdHandler, "corehttp.cmdsHandler")
        # 将命令处理器和 API 路径注册到 HTTP 服务多路复用器中
        mux.Handle(APIPath+"/", cmdHandler)
        # 返回 HTTP 服务多路复用器和空错误
        return mux, nil
    }
// 定义一个结构体 rpcAuthScopeWithUser，包含 config.RPCAuthScope 和 User 字段
type rpcAuthScopeWithUser struct {
    config.RPCAuthScope
    User string
}

// 将传入的 authScopes map 转换为 map[string]rpcAuthScopeWithUser 类型
func convertAuthorizationsMap(authScopes map[string]*config.RPCAuthScope) map[string]rpcAuthScopeWithUser {
    // 创建一个空的 map，用于存储转换后的数据
    authorizations := map[string]rpcAuthScopeWithUser{}
    // 遍历传入的 authScopes map
    for user, authScope := range authScopes {
        // 将 authScope.AuthSecret 转换为 header 值
        expectedHeader := config.ConvertAuthSecret(authScope.AuthSecret)
        // 如果转换后的值不为空
        if expectedHeader != "" {
            // 将转换后的值作为 key，创建 rpcAuthScopeWithUser 对象作为 value 存入 authorizations map
            authorizations[expectedHeader] = rpcAuthScopeWithUser{
                RPCAuthScope: *authScopes[user],
                User:         user,
            }
        }
    }
    // 返回转换后的 authorizations map
    return authorizations
}

// 创建一个带有授权信息的 http.Handler
func withAuthSecrets(authorizations map[string]rpcAuthScopeWithUser, next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // 获取请求头中的 Authorization 值
        authorizationHeader := r.Header.Get("Authorization")
        // 在 authorizations map 中查找对应的授权信息
        auth, ok := authorizations[authorizationHeader]

        // 如果找到对应的授权信息
        if ok {
            // 如果请求的 URL 是 "/api/v0/version"，直接调用下一个处理器
            if r.URL.Path == "/api/v0/version" {
                next.ServeHTTP(w, r)
                return
            }
            // 否则，检查请求的 URL 是否在授权信息的 AllowedPaths 中
            for _, prefix := range auth.AllowedPaths {
                if strings.HasPrefix(r.URL.Path, prefix) {
                    next.ServeHTTP(w, r)
                    return
                }
            }
        }

        // 如果未找到对应的授权信息，返回 403 错误
        http.Error(w, "Kubo RPC Access Denied: Please provide a valid authorization token as defined in the API.Authorizations configuration.", http.StatusForbidden)
    })
}

// CommandsOption 构造一个 ServerOption，用于将命令挂接到 HTTP 服务器上，不允许 GET 请求
func CommandsOption(cctx oldcmds.Context) ServeOption {
    return commandsOption(cctx, corecommands.Root, false)
}

// CommandsROOption 构造一个 ServerOption，用于挂接只读命令
// 返回一个 ServeOption 函数，用于配置允许 HTTP 服务器处理 GET 请求
func CommandsROOption(cctx oldcmds.Context) ServeOption {
    return commandsOption(cctx, corecommands.RootRO, true)
}

// 返回一个 ServeOption 函数，用于检查客户端 IPFS 版本是否匹配，当用户代理字符串不包含 `/kubo/` 或 `/go-ipfs/` 时不执行任何操作
func CheckVersionOption() ServeOption {
    daemonVersion := version.ApiVersion

    return func(n *core.IpfsNode, l net.Listener, parent *http.ServeMux) (*http.ServeMux, error) {
        mux := http.NewServeMux()
        parent.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
            if strings.HasPrefix(r.URL.Path, APIPath) {
                cmdqry := r.URL.Path[len(APIPath):]
                pth := strings.Split(cmdqry, "/")

                // 向后兼容以前的版本检查
                if len(pth) >= 2 && pth[1] != "version" {
                    clientVersion := r.UserAgent()
                    // 如果客户端不是 kubo (go-ipfs)，则跳过检查
                    if (strings.Contains(clientVersion, "/go-ipfs/") || strings.Contains(clientVersion, "/kubo/")) && daemonVersion != clientVersion {
                        http.Error(w, fmt.Sprintf("%s (%s != %s)", errAPIVersionMismatch, daemonVersion, clientVersion), http.StatusBadRequest)
                        return
                    }
                }
            }

            mux.ServeHTTP(w, r)
        })

        return mux, nil
    }
}
```