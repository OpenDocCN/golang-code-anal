# `kubo\core\coreapi\name.go`

```go
package coreapi

import (
    "context" // 引入上下文包，用于处理请求的上下文信息
    "fmt" // 引入格式化包，用于格式化输出
    "strings" // 引入字符串包，用于处理字符串
    "time" // 引入时间包，用于处理时间

    "github.com/ipfs/boxo/ipns" // 引入IPNS包
    keystore "github.com/ipfs/boxo/keystore" // 引入密钥库包
    "github.com/ipfs/boxo/namesys" // 引入命名系统包
    "github.com/ipfs/kubo/tracing" // 引入追踪包
    "go.opentelemetry.io/otel/attribute" // 引入属性包
    "go.opentelemetry.io/otel/trace" // 引入追踪包

    "github.com/ipfs/boxo/path" // 引入路径包
    coreiface "github.com/ipfs/kubo/core/coreiface" // 引入核心接口包
    caopts "github.com/ipfs/kubo/core/coreiface/options" // 引入选项包
    ci "github.com/libp2p/go-libp2p/core/crypto" // 引入加密包
    peer "github.com/libp2p/go-libp2p/core/peer" // 引入对等节点包
)

type NameAPI CoreAPI // 定义NameAPI类型为CoreAPI类型

// Publish announces new IPNS name and returns the new IPNS entry.
func (api *NameAPI) Publish(ctx context.Context, p path.Path, opts ...caopts.NamePublishOption) (ipns.Name, error) {
    ctx, span := tracing.Span(ctx, "CoreAPI.NameAPI", "Publish", trace.WithAttributes(attribute.String("path", p.String()))) // 创建追踪span
    defer span.End() // 延迟结束span

    if err := api.checkPublishAllowed(); err != nil { // 检查是否允许发布
        return ipns.Name{}, err // 返回空的IPNS名称和错误
    }

    options, err := caopts.NamePublishOptions(opts...) // 获取发布选项
    if err != nil {
        return ipns.Name{}, err // 返回空的IPNS名称和错误
    }
    span.SetAttributes( // 设置span的属性
        attribute.Bool("allowoffline", options.AllowOffline),
        attribute.String("key", options.Key),
        attribute.Float64("validtime", options.ValidTime.Seconds()),
    )
    if options.TTL != nil {
        span.SetAttributes(attribute.Float64("ttl", options.TTL.Seconds())) // 如果TTL不为空，设置span的TTL属性
    }

    err = api.checkOnline(options.AllowOffline) // 检查是否在线
    if err != nil {
        return ipns.Name{}, err // 返回空的IPNS名称和错误
    }

    k, err := keylookup(api.privateKey, api.repo.Keystore(), options.Key) // 查找密钥
    if err != nil {
        return ipns.Name{}, err // 返回空的IPNS名称和错误
    }

    eol := time.Now().Add(options.ValidTime) // 计算过期时间

    publishOptions := []namesys.PublishOption{ // 创建发布选项
        namesys.PublishWithEOL(eol), // 设置过期时间
        namesys.PublishWithIPNSOption(ipns.WithV1Compatibility(options.CompatibleWithV1)), // 设置IPNS选项
    }
    # 如果TTL选项不为空，则将PublishWithTTL(*options.TTL)添加到publishOptions切片中
    if options.TTL != nil {
        publishOptions = append(publishOptions, namesys.PublishWithTTL(*options.TTL))
    }

    # 使用API对象的namesys.Publish方法发布给定的密钥和路径，并传入publishOptions切片中的选项
    err = api.namesys.Publish(ctx, k, p, publishOptions...)
    # 如果出现错误，则返回空的IPNS名称和错误
    if err != nil {
        return ipns.Name{}, err
    }

    # 使用给定的私钥生成对等节点的ID
    pid, err := peer.IDFromPrivateKey(k)
    # 如果出现错误，则返回空的IPNS名称和错误
    if err != nil {
        return ipns.Name{}, err
    }

    # 根据对等节点的ID生成IPNS名称，并返回该名称和空的错误
    return ipns.NameFromPeer(pid), nil
// Search方法用于在给定的上下文中搜索指定名称的IPNS结果，并返回IPNS结果的通道和错误信息
func (api *NameAPI) Search(ctx context.Context, name string, opts ...caopts.NameResolveOption) (<-chan coreiface.IpnsResult, error) {
    // 创建一个新的span来跟踪Search方法的执行
    ctx, span := tracing.Span(ctx, "CoreAPI.NameAPI", "Search", trace.WithAttributes(attribute.String("name", name)))
    // 在方法执行结束时结束span
    defer span.End()

    // 解析NameResolveOption参数
    options, err := caopts.NameResolveOptions(opts...)
    if err != nil {
        return nil, err
    }

    // 设置span的属性，表示是否使用缓存
    span.SetAttributes(attribute.Bool("cache", options.Cache))

    // 检查API是否在线
    err = api.checkOnline(true)
    if err != nil {
        return nil, err
    }

    // 根据是否使用缓存选择相应的解析器
    var resolver namesys.Resolver = api.namesys
    if !options.Cache {
        resolver, err = namesys.NewNameSystem(api.routing,
            namesys.WithDatastore(api.repo.Datastore()),
            namesys.WithDNSResolver(api.dnsResolver))
        if err != nil {
            return nil, err
        }
    }

    // 如果名称不以"/ipns/"开头，则添加"/ipns/"前缀
    if !strings.HasPrefix(name, "/ipns/") {
        name = "/ipns/" + name
    }

    // 创建Path对象
    p, err := path.NewPath(name)
    if err != nil {
        return nil, err
    }

    // 创建用于返回IPNS结果的通道
    out := make(chan coreiface.IpnsResult)
    // 启动goroutine来异步解析IPNS结果
    go func() {
        defer close(out)
        for res := range resolver.ResolveAsync(ctx, p, options.ResolveOpts...) {
            select {
            case out <- coreiface.IpnsResult{Path: res.Path, Err: res.Err}:
            case <-ctx.Done():
                return
            }
        }
    }()

    return out, nil
}

// Resolve方法尝试解析指定名称的最新版本，并返回其路径
func (api *NameAPI) Resolve(ctx context.Context, name string, opts ...caopts.NameResolveOption) (path.Path, error) {
    // 创建一个新的span来跟踪Resolve方法的执行
    ctx, span := tracing.Span(ctx, "CoreAPI.NameAPI", "Resolve", trace.WithAttributes(attribute.String("name", name)))
    // 在方法执行结束时结束span
    defer span.End()

    // 创建一个新的上下文，并在方法执行结束时取消
    ctx, cancel := context.WithCancel(ctx)
    defer cancel()

    // 调用Search方法来搜索指定名称的IPNS结果
    results, err := api.Search(ctx, name, opts...)
    if err != nil {
        return nil, err
    }

    // 初始化错误信息和Path对象
    err = coreiface.ErrResolveFailed
    var p path.Path
    # 遍历结果集中的每个结果
    for res := range results {
        # 获取结果的路径和错误信息
        p, err = res.Path, res.Err
        # 如果有错误，则跳出循环
        if err != nil {
            break
        }
    }
    # 返回最后一个结果的路径和错误信息
    return p, err
func keylookup(self ci.PrivKey, kstore keystore.Keystore, k string) (ci.PrivKey, error) {
    ////////////////////
    // Lookup by name //
    ////////////////////

    // 首先，按名称查找
    // 如果名称为"self"，则返回self
    if k == "self" {
        return self, nil
    }

    // 然后，在密钥存储中查找
    res, err := kstore.Get(k)
    if res != nil {
        return res, nil
    }

    // 如果出现错误并且错误不是ErrNoSuchKey，则返回错误
    if err != nil && err != keystore.ErrNoSuchKey {
        return nil, err
    }

    // 获取密钥列表
    keys, err := kstore.List()
    if err != nil {
        return nil, err
    }

    //////////////////
    // Lookup by ID //
    //////////////////
    // 通过ID查找
    targetPid, err := peer.Decode(k)
    if err != nil {
        return nil, keystore.ErrNoSuchKey
    }

    // 首先，检查self
    pid, err := peer.IDFromPrivateKey(self)
    if err != nil {
        return nil, fmt.Errorf("failed to determine peer ID for private key: %w", err)
    }
    // 如果self的PeerID与目标PeerID相同，则返回self
    if pid == targetPid {
        return self, nil
    }

    // 然后，在密钥存储中查找
    for _, key := range keys {
        privKey, err := kstore.Get(key)
        if err != nil {
            return nil, err
        }

        pid, err := peer.IDFromPrivateKey(privKey)
        if err != nil {
            return nil, err
        }

        // 如果目标PeerID与当前密钥的PeerID相同，则返回当前密钥
        if targetPid == pid {
            return privKey, nil
        }
    }

    // 如果没有找到给定名称或PeerID的密钥，则返回错误
    return nil, fmt.Errorf("no key by the given name or PeerID was found")
}
```