# `kubo\core\corehttp\gateway.go`

```go
package corehttp

import (
    "context"  // 导入上下文包，用于处理请求的上下文信息
    "errors"   // 导入错误包，用于处理错误信息
    "fmt"      // 导入格式化包，用于格式化输出
    "io"       // 导入输入输出包，用于处理输入输出操作
    "net"      // 导入网络包，用于网络操作
    "net/http" // 导入 HTTP 包，用于处理 HTTP 请求
    "time"     // 导入时间包，用于处理时间

    "github.com/ipfs/boxo/blockservice"  // 导入 IPFS 包，用于区块服务
    "github.com/ipfs/boxo/exchange/offline"  // 导入 IPFS 包，用于离线交换
    "github.com/ipfs/boxo/files"  // 导入 IPFS 包，用于文件操作
    "github.com/ipfs/boxo/gateway"  // 导入 IPFS 包，用于网关操作
    "github.com/ipfs/boxo/namesys"  // 导入 IPFS 包，用于命名系统
    "github.com/ipfs/boxo/path"  // 导入 IPFS 包，用于路径操作
    offlineroute "github.com/ipfs/boxo/routing/offline"  // 导入 IPFS 包，用于离线路由
    "github.com/ipfs/go-cid"  // 导入 IPFS 包，用于 CID 操作
    version "github.com/ipfs/kubo"  // 导入 IPFS 包，用于版本信息
    "github.com/ipfs/kubo/config"  // 导入 IPFS 包，用于配置信息
    "github.com/ipfs/kubo/core"  // 导入 IPFS 包，用于核心功能
    iface "github.com/ipfs/kubo/core/coreiface"  // 导入 IPFS 包，用于核心接口
    "github.com/ipfs/kubo/core/node"  // 导入 IPFS 包，用于节点操作
    "github.com/libp2p/go-libp2p/core/routing"  // 导入 libp2p 包，用于路由操作
    "go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp"  // 导入 OpenTelemetry 包，用于 HTTP 监控
)

func GatewayOption(paths ...string) ServeOption {
    return func(n *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
        config, err := getGatewayConfig(n)  // 获取网关配置信息
        if err != nil {
            return nil, err
        }

        backend, err := newGatewayBackend(n)  // 创建网关后端
        if err != nil {
            return nil, err
        }

        handler := gateway.NewHandler(config, backend)  // 创建网关处理程序
        handler = otelhttp.NewHandler(handler, "Gateway")  // 使用 OpenTelemetry 包装网关处理程序

        for _, p := range paths {
            mux.Handle(p+"/", handler)  // 处理路径请求
        }

        return mux, nil
    }
}

func HostnameOption() ServeOption {
    return func(n *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
        config, err := getGatewayConfig(n)  // 获取网关配置信息
        if err != nil {
            return nil, err
        }

        backend, err := newGatewayBackend(n)  // 创建网关后端
        if err != nil {
            return nil, err
        }

        childMux := http.NewServeMux()  // 创建子 ServeMux
        mux.Handle("/", gateway.NewHostnameHandler(config, backend, childMux))  // 处理根路径请求
        return childMux, nil
    }
}

func VersionOption() ServeOption {
    # 定义一个函数，接受一个IpfsNode指针、net.Listener和http.ServeMux指针作为参数，并返回一个http.ServeMux指针和一个错误
    return func(_ *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
        # 为指定路径"/version"注册一个处理函数，该函数接受一个http.ResponseWriter和一个http.Request作为参数
        mux.HandleFunc("/version", func(w http.ResponseWriter, r *http.Request) {
            # 向客户端写入当前提交的版本信息
            fmt.Fprintf(w, "Commit: %s\n", version.CurrentCommit)
            # 向客户端写入客户端版本信息
            fmt.Fprintf(w, "Client Version: %s\n", version.GetUserAgentVersion())
        })
        # 返回处理完"/version"路径后的http.ServeMux指针和一个空的错误
        return mux, nil
    }
}
# 定义一个名为Libp2pGatewayOption的函数，返回ServeOption类型
func Libp2pGatewayOption() ServeOption {
    # 返回一个函数，该函数接受IpfsNode指针、net.Listener和http.ServeMux指针，并返回http.ServeMux指针和error
    return func(n *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
        # 创建一个新的blockservice，使用IpfsNode的Blockstore和离线交换
        bserv := blockservice.New(n.Blocks.Blockstore(), offline.Exchange(n.Blocks.Blockstore()))

        # 创建一个新的gateway的BlocksBackend
        backend, err := gateway.NewBlocksBackend(bserv,
            # 使用离线路径解析器，GatewayOverLibp2p仅返回本地块存储中的内容（与Gateway.NoFetch=true相同）
            gateway.WithResolver(n.OfflineUnixFSPathResolver),
        )
        if err != nil {
            return nil, err
        }

        # 创建一个gateway配置
        gwConfig := gateway.Config{
            DeserializedResponses: false,
            NoDNSLink:             true,
            PublicGateways:        nil,
            Menu:                  nil,
        }

        # 创建一个新的handler，使用gateway配置和离线网关错误包装器
        handler := gateway.NewHandler(gwConfig, &offlineGatewayErrWrapper{gwimpl: backend})
        # 使用otelhttp包装handler，设置名称为"Libp2p-Gateway"
        handler = otelhttp.NewHandler(handler, "Libp2p-Gateway")

        # 将handler绑定到指定的路径
        mux.Handle("/ipfs/", handler)

        # 返回更新后的http.ServeMux和nil
        return mux, nil
    }
}

# 定义一个名为newGatewayBackend的函数，接受一个IpfsNode指针，并返回gateway.IPFSBackend和error
func newGatewayBackend(n *core.IpfsNode) (gateway.IPFSBackend, error) {
    # 获取IpfsNode的配置
    cfg, err := n.Repo.Config()
    if err != nil {
        return nil, err
    }

    # 获取IpfsNode的Blocks、Routing、Namesys和UnixFSPathResolver
    bserv := n.Blocks
    var vsRouting routing.ValueStore = n.Routing
    nsys := n.Namesys
    pathResolver := n.UnixFSPathResolver
}
    # 如果配置中指定不进行数据获取
    if cfg.Gateway.NoFetch {
        # 创建一个新的区块服务，使用离线交换机来交换区块存储
        bserv = blockservice.New(bserv.Blockstore(), offline.Exchange(bserv.Blockstore()))

        # 获取IPNS解析缓存大小
        cs := cfg.Ipns.ResolveCacheSize
        # 如果未指定缓存大小，则使用默认的IPNS缓存大小
        if cs == 0 {
            cs = node.DefaultIpnsCacheSize
        }
        # 如果缓存大小为负数，则返回错误
        if cs < 0 {
            return nil, fmt.Errorf("cannot specify negative resolve cache size")
        }

        # 创建离线路由器，使用节点数据存储和记录验证器
        vsRouting = offlineroute.NewOfflineRouter(n.Repo.Datastore(), n.RecordValidator)
        # 创建名称系统，使用离线路由器、数据存储、DNS解析器和缓存大小
        nsys, err = namesys.NewNameSystem(vsRouting,
            namesys.WithDatastore(n.Repo.Datastore()),
            namesys.WithDNSResolver(n.DNSResolver),
            namesys.WithCache(cs))
        # 如果创建名称系统时出现错误，则返回错误
        if err != nil {
            return nil, fmt.Errorf("error constructing namesys: %w", err)
        }

        # Gateway.NoFetch=true需要离线路径解析器，以避免在路径遍历期间获取丢失的区块
        pathResolver = n.OfflineUnixFSPathResolver
    }

    # 创建区块后端，使用区块服务、数值存储、名称系统和路径解析器
    backend, err := gateway.NewBlocksBackend(bserv,
        gateway.WithValueStore(vsRouting),
        gateway.WithNameSystem(nsys),
        gateway.WithResolver(pathResolver),
    )
    # 如果创建区块后端时出现错误，则返回错误
    if err != nil {
        return nil, err
    }
    # 返回离线网关错误包装后的结果
    return &offlineGatewayErrWrapper{gwimpl: backend}, nil
// 定义一个结构体，包含一个实现了IPFSBackend接口的gateway对象
type offlineGatewayErrWrapper struct {
    gwimpl gateway.IPFSBackend
}

// 定义一个函数，用于处理离线错误，如果错误是ErrOffline，则返回ServiceUnavailable错误
func offlineErrWrap(err error) error {
    if errors.Is(err, iface.ErrOffline) {
        return fmt.Errorf("%s : %w", err.Error(), gateway.ErrServiceUnavailable)
    }
    return err
}

// 实现Get方法，调用gwimpl的Get方法，处理离线错误
func (o *offlineGatewayErrWrapper) Get(ctx context.Context, path path.ImmutablePath, ranges ...gateway.ByteRange) (gateway.ContentPathMetadata, *gateway.GetResponse, error) {
    md, n, err := o.gwimpl.Get(ctx, path, ranges...)
    err = offlineErrWrap(err)
    return md, n, err
}

// 实现GetAll方法，调用gwimpl的GetAll方法，处理离线错误
func (o *offlineGatewayErrWrapper) GetAll(ctx context.Context, path path.ImmutablePath) (gateway.ContentPathMetadata, files.Node, error) {
    md, n, err := o.gwimpl.GetAll(ctx, path)
    err = offlineErrWrap(err)
    return md, n, err
}

// 实现GetBlock方法，调用gwimpl的GetBlock方法，处理离线错误
func (o *offlineGatewayErrWrapper) GetBlock(ctx context.Context, path path.ImmutablePath) (gateway.ContentPathMetadata, files.File, error) {
    md, n, err := o.gwimpl.GetBlock(ctx, path)
    err = offlineErrWrap(err)
    return md, n, err
}

// 实现Head方法，调用gwimpl的Head方法，处理离线错误
func (o *offlineGatewayErrWrapper) Head(ctx context.Context, path path.ImmutablePath) (gateway.ContentPathMetadata, *gateway.HeadResponse, error) {
    md, n, err := o.gwimpl.Head(ctx, path)
    err = offlineErrWrap(err)
    return md, n, err
}

// 实现ResolvePath方法，调用gwimpl的ResolvePath方法，处理离线错误
func (o *offlineGatewayErrWrapper) ResolvePath(ctx context.Context, path path.ImmutablePath) (gateway.ContentPathMetadata, error) {
    md, err := o.gwimpl.ResolvePath(ctx, path)
    err = offlineErrWrap(err)
    return md, err
}

// 实现GetCAR方法，调用gwimpl的GetCAR方法，处理离线错误
func (o *offlineGatewayErrWrapper) GetCAR(ctx context.Context, path path.ImmutablePath, params gateway.CarParams) (gateway.ContentPathMetadata, io.ReadCloser, error) {
    md, data, err := o.gwimpl.GetCAR(ctx, path, params)
    err = offlineErrWrap(err)
    return md, data, err
}

// 实现IsCached方法，调用gwimpl的IsCached方法
func (o *offlineGatewayErrWrapper) IsCached(ctx context.Context, path path.Path) bool {
    return o.gwimpl.IsCached(ctx, path)
}
func (o *offlineGatewayErrWrapper) GetIPNSRecord(ctx context.Context, c cid.Cid) ([]byte, error) {
    // 调用底层网关实现的 GetIPNSRecord 方法，获取 IPNS 记录
    rec, err := o.gwimpl.GetIPNSRecord(ctx, c)
    // 对错误进行包装处理
    err = offlineErrWrap(err)
    return rec, err
}

func (o *offlineGatewayErrWrapper) ResolveMutable(ctx context.Context, path path.Path) (path.ImmutablePath, time.Duration, time.Time, error) {
    // 调用底层网关实现的 ResolveMutable 方法，解析可变路径
    imPath, ttl, lastMod, err := o.gwimpl.ResolveMutable(ctx, path)
    // 对错误进行包装处理
    err = offlineErrWrap(err)
    return imPath, ttl, lastMod, err
}

func (o *offlineGatewayErrWrapper) GetDNSLinkRecord(ctx context.Context, s string) (path.Path, error) {
    // 调用底层网关实现的 GetDNSLinkRecord 方法，获取 DNSLink 记录
    p, err := o.gwimpl.GetDNSLinkRecord(ctx, s)
    // 对错误进行包装处理
    err = offlineErrWrap(err)
    return p, err
}

// 确保 offlineGatewayErrWrapper 实现了 IPFSBackend 接口
var _ gateway.IPFSBackend = (*offlineGatewayErrWrapper)(nil)

// 默认路径列表
var defaultPaths = []string{"/ipfs/", "/ipns/", "/api/", "/p2p/"}

// 子域网关规范
var subdomainGatewaySpec = &gateway.PublicGateway{
    Paths:         defaultPaths,
    UseSubdomains: true,
}

// 默认已知网关映射
var defaultKnownGateways = map[string]*gateway.PublicGateway{
    "localhost": subdomainGatewaySpec,
}

func getGatewayConfig(n *core.IpfsNode) (gateway.Config, error) {
    // 获取节点的配置信息
    cfg, err := n.Repo.Config()
    if err != nil {
        return gateway.Config{}, err
    }

    // 解析配置头并添加默认的访问控制头
    headers := make(map[string][]string, len(cfg.Gateway.HTTPHeaders))
    for h, v := range cfg.Gateway.HTTPHeaders {
        headers[http.CanonicalHeaderKey(h)] = v
    }
    gateway.AddAccessControlHeaders(headers)

    // 初始化网关配置，使用空的 PublicGateways，稍后处理
    gwCfg := gateway.Config{
        Headers:               headers,
        DeserializedResponses: cfg.Gateway.DeserializedResponses.WithDefault(config.DefaultDeserializedResponses),
        DisableHTMLErrors:     cfg.Gateway.DisableHTMLErrors.WithDefault(config.DefaultDisableHTMLErrors),
        NoDNSLink:             cfg.Gateway.NoDNSLink,
        PublicGateways:        map[string]*gateway.PublicGateway{},
    }
}
    // 添加默认的隐式已知网关，比如本地主机上的子域网关。
    for hostname, gw := range defaultKnownGateways {
        // 将默认已知网关添加到公共网关配置中
        gwCfg.PublicGateways[hostname] = gw
    }

    // 如果存在，应用来自 cfg.Gateway.PublicGateways 的值。
    for hostname, gw := range cfg.Gateway.PublicGateways {
        if gw == nil {
            // 如果存在隐式默认值，则删除它们。当想要禁用本地主机上的子域网关等时，这很有用。
            delete(gwCfg.PublicGateways, hostname)
            continue
        }

        // 将 cfg.Gateway.PublicGateways 中的值应用到公共网关配置中
        gwCfg.PublicGateways[hostname] = &gateway.PublicGateway{
            Paths:                 gw.Paths,
            NoDNSLink:             gw.NoDNSLink,
            UseSubdomains:         gw.UseSubdomains,
            InlineDNSLink:         gw.InlineDNSLink.WithDefault(config.DefaultInlineDNSLink),
            DeserializedResponses: gw.DeserializedResponses.WithDefault(gwCfg.DeserializedResponses),
        }
    }

    // 返回公共网关配置和空错误
    return gwCfg, nil
# 闭合前面的函数定义
```