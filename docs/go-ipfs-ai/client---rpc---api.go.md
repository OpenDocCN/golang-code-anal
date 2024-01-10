# `kubo\client\rpc\api.go`

```
package rpc

import (
    "context" // 上下文包，用于控制请求的取消、超时等
    "encoding/json" // JSON 编解码包
    "errors" // 错误处理包
    "fmt" // 格式化包，用于打印输出
    "net/http" // HTTP 客户端包
    "os" // 操作系统函数包
    "path/filepath" // 文件路径操作包
    "strings" // 字符串操作包
    "sync" // 同步包，用于并发控制
    "time" // 时间包

    "github.com/blang/semver/v4" // 语义化版本包
    "github.com/ipfs/boxo/ipld/merkledag" // IPFS 相关包
    "github.com/ipfs/go-cid" // IPFS CID 包
    legacy "github.com/ipfs/go-ipld-legacy" // IPFS 旧版 IPLD 包
    ipfs "github.com/ipfs/kubo" // IPFS 包
    iface "github.com/ipfs/kubo/core/coreiface" // IPFS 接口包
    caopts "github.com/ipfs/kubo/core/coreiface/options" // IPFS 选项包
    dagpb "github.com/ipld/go-codec-dagpb" // DAGPB 编解码包
    _ "github.com/ipld/go-ipld-prime/codec/dagcbor" // DAGCBOR 编解码包
    "github.com/ipld/go-ipld-prime/node/basicnode" // 基本节点包
    "github.com/mitchellh/go-homedir" // 用户目录操作包
    ma "github.com/multiformats/go-multiaddr" // 多地址包
    manet "github.com/multiformats/go-multiaddr/net" // 多地址网络包
)

const (
    DefaultPathName = ".ipfs" // 默认 IPFS 路径名
    DefaultPathRoot = "~/" + DefaultPathName // 默认 IPFS 根路径
    DefaultApiFile  = "api" // 默认 API 文件名
    EnvDir          = "IPFS_PATH" // 环境变量中 IPFS 路径名
)

// ErrApiNotFound if we fail to find a running daemon.
var ErrApiNotFound = errors.New("ipfs api address could not be found") // 未找到运行中的守护进程错误

// HttpApi implements github.com/ipfs/interface-go-ipfs-core/CoreAPI using
// IPFS HTTP API.
//
// For interface docs see
// https://godoc.org/github.com/ipfs/interface-go-ipfs-core#CoreAPI
type HttpApi struct {
    url         string // API 地址
    httpcli     http.Client // HTTP 客户端
    Headers     http.Header // HTTP 请求头
    applyGlobal func(*requestBuilder) // 应用全局函数
    ipldDecoder *legacy.Decoder // IPLD 解码器
    versionMu   sync.Mutex // 版本锁
    version     *semver.Version // 语义化版本
}

// NewLocalApi tries to construct new HttpApi instance communicating with local
// IPFS daemon
//
// Daemon api address is pulled from the $IPFS_PATH/api file.
// If $IPFS_PATH env var is not present, it defaults to ~/.ipfs.
func NewLocalApi() (*HttpApi, error) {
    baseDir := os.Getenv(EnvDir) // 获取环境变量中的 IPFS 路径
    if baseDir == "" {
        baseDir = DefaultPathRoot // 如果环境变量中没有设置 IPFS 路径，则使用默认根路径
    }

    return NewPathApi(baseDir) // 返回指定路径的 API 实例
}

// NewPathApi constructs new HttpApi by pulling api address from specified
// ipfspath. Api file should be located at $ipfspath/api.
// NewPathApi creates a new HttpApi object based on the given IPFS path.
func NewPathApi(ipfspath string) (*HttpApi, error) {
    // 获取 API 地址
    a, err := ApiAddr(ipfspath)
    if err != nil {
        // 如果出现错误，检查是否为文件不存在错误，如果是则返回 ErrApiNotFound
        if os.IsNotExist(err) {
            err = ErrApiNotFound
        }
        return nil, err
    }
    // 根据 API 地址创建新的 HttpApi 对象
    return NewApi(a)
}

// ApiAddr 从指定的 IPFS 路径中读取 API 文件地址
func ApiAddr(ipfspath string) (ma.Multiaddr, error) {
    // 将 IPFS 路径扩展为绝对路径
    baseDir, err := homedir.Expand(ipfspath)
    if err != nil {
        return nil, err
    }

    // 获取 API 文件的完整路径
    apiFile := filepath.Join(baseDir, DefaultApiFile)

    // 读取 API 文件内容
    api, err := os.ReadFile(apiFile)
    if err != nil {
        return nil, err
    }

    // 创建 Multiaddr 对象并返回
    return ma.NewMultiaddr(strings.TrimSpace(string(api)))
}

// NewApi 使用指定的端点构造 HttpApi 对象
func NewApi(a ma.Multiaddr) (*HttpApi, error) {
    // 创建自定义的 http 客户端
    c := &http.Client{
        Transport: &http.Transport{
            Proxy:             http.ProxyFromEnvironment,
            DisableKeepAlives: true,
        },
    }

    return NewApiWithClient(a, c)
}

// NewApiWithClient 使用指定的端点和自定义的 http 客户端构造 HttpApi 对象
func NewApiWithClient(a ma.Multiaddr, c *http.Client) (*HttpApi, error) {
    // 解析 Multiaddr 获取 URL 和错误信息
    _, url, err := manet.DialArgs(a)
    if err != nil {
        return nil, err
    }

    // 如果 Multiaddr 包含协议信息，则根据协议信息确定使用 http 还是 https
    proto := "http://"
    protocols := a.Protocols()
    for _, p := range protocols {
        if p.Code == ma.P_HTTPS || p.Code == ma.P_TLS {
            proto = "https://"
            break
        }
    }

    // 使用指定的 URL 和 http 客户端创建 HttpApi 对象
    return NewURLApiWithClient(proto+url, c)
}

// NewURLApiWithClient 使用指定的 URL 和 http 客户端构造 HttpApi 对象
func NewURLApiWithClient(url string, c *http.Client) (*HttpApi, error) {
    // 省略部分代码
}
    // 创建一个名为decoder的legacy解码器实例
    decoder := legacy.NewDecoder()
    // 为了匹配merkledag库的行为，添加对这些编解码器的支持
    // 注意：为了匹配先前的行为，手动包含go-ipld-prime CBOR解码器
    // TODO: 允许调用者通过全局变量而不是通过配置来配置使用的编解码器注册表
    decoder.RegisterCodec(cid.DagProtobuf, dagpb.Type.PBNode, merkledag.ProtoNodeConverter)
    decoder.RegisterCodec(cid.Raw, basicnode.Prototype.Bytes, merkledag.RawNodeConverter)

    // 创建一个名为api的HttpApi实例
    api := &HttpApi{
        url:         url,
        httpcli:     *c,
        Headers:     make(map[string][]string),
        applyGlobal: func(*requestBuilder) {},
        ipldDecoder: decoder,
    }

    // 禁止重定向
    api.httpcli.CheckRedirect = func(_ *http.Request, _ []*http.Request) error {
        return fmt.Errorf("unexpected redirect")
    }

    // 返回api实例和nil
    return api, nil
# 定义一个方法，接收一系列的选项，并返回一个核心API接口和一个错误
func (api *HttpApi) WithOptions(opts ...caopts.ApiOption) (iface.CoreAPI, error) {
    # 使用给定的选项创建API配置
    options, err := caopts.ApiOptions(opts...)
    # 如果创建配置时出现错误，则返回空和错误
    if err != nil {
        return nil, err
    }

    # 创建一个新的HttpApi对象，继承自原始的api对象
    subApi := &HttpApi{
        url:     api.url,
        httpcli: api.httpcli,
        Headers: api.Headers,
        applyGlobal: func(req *requestBuilder) {
            # 如果选项中包含离线标志，则在请求中添加离线选项
            if options.Offline {
                req.Option("offline", options.Offline)
            }
        },
        ipldDecoder: api.ipldDecoder,
    }

    # 返回新创建的子API对象和空错误
    return subApi, nil
}

# 定义一个方法，接收一个命令和一系列参数，并返回一个请求构建器
func (api *HttpApi) Request(command string, args ...string) RequestBuilder {
    # 创建一个空的头部map
    headers := make(map[string]string)
    # 如果原始API对象的头部不为空，则将其复制到新的头部map中
    if api.Headers != nil {
        for k := range api.Headers {
            headers[k] = api.Headers.Get(k)
        }
    }
    # 返回一个新的请求构建器对象
    return &requestBuilder{
        command: command,
        args:    args,
        shell:   api,
        headers: headers,
    }
}

# 以下为一系列方法，每个方法都将原始API对象转换为特定的接口类型并返回
# 具体的方法作用不在此处详细解释
func (api *HttpApi) Unixfs() iface.UnixfsAPI {
    return (*UnixfsAPI)(api)
}

func (api *HttpApi) Block() iface.BlockAPI {
    return (*BlockAPI)(api)
}

func (api *HttpApi) Dag() iface.APIDagService {
    return (*HttpDagServ)(api)
}

func (api *HttpApi) Name() iface.NameAPI {
    return (*NameAPI)(api)
}

func (api *HttpApi) Key() iface.KeyAPI {
    return (*KeyAPI)(api)
}

func (api *HttpApi) Pin() iface.PinAPI {
    return (*PinAPI)(api)
}

func (api *HttpApi) Object() iface.ObjectAPI {
    return (*ObjectAPI)(api)
}

func (api *HttpApi) Dht() iface.DhtAPI {
    return (*DhtAPI)(api)
}

func (api *HttpApi) Swarm() iface.SwarmAPI {
    return (*SwarmAPI)(api)
}

func (api *HttpApi) PubSub() iface.PubSubAPI {
    return (*PubsubAPI)(api)
}

func (api *HttpApi) Routing() iface.RoutingAPI {
    return (*RoutingAPI)(api)
}

# 定义一个方法，用于加载远程版本信息，返回一个版本号和一个错误
func (api *HttpApi) loadRemoteVersion() (*semver.Version, error) {
    # 加锁，确保在方法执行期间不会被其他线程修改
    api.versionMu.Lock()
    # 在方法执行结束后解锁
    defer api.versionMu.Unlock()
    # 如果 API 的版本为空
    if api.version == nil {
        # 创建一个带有截止时间的上下文，并设置超时时间为30秒
        ctx, cancel := context.WithDeadline(context.Background(), time.Now().Add(time.Second*30))
        # 在函数返回时调用取消函数
        defer cancel()

        # 发送带有上下文的版本请求，并接收响应
        resp, err := api.Request("version").Send(ctx)
        # 如果发生错误，返回空和错误信息
        if err != nil {
            return nil, err
        }
        # 如果响应中包含错误信息，返回空和响应中的错误信息
        if resp.Error != nil {
            return nil, resp.Error
        }
        # 在函数返回时关闭响应
        defer resp.Close()
        # 创建一个新的 JSON 解码器，用于解码响应的输出
        dec := json.NewDecoder(resp.Output)
        # 解码 JSON 数据到 VersionInfo 结构体中
        if err := dec.Decode(&out); err != nil {
            return nil, err
        }

        # 解析远程版本号并存储到 remoteVersion 中
        remoteVersion, err := semver.New(out.Version)
        # 如果解析出错，返回空和错误信息
        if err != nil {
            return nil, err
        }

        # 将远程版本号存储到 API 对象中的 version 字段
        api.version = remoteVersion
    }

    # 返回 API 对象中的版本号和空
    return api.version, nil
# 闭合前面的函数定义
```