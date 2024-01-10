# `kubo\routing\composer.go`

```
package routing

import (
    "context"  // 导入上下文包，用于处理请求的上下文信息
    "errors"   // 导入错误包，用于处理错误信息

    "github.com/hashicorp/go-multierror"  // 导入多错误包，用于处理多个错误信息
    "github.com/ipfs/go-cid"  // 导入CID包，用于处理CID（Content Identifier）
    routinghelpers "github.com/libp2p/go-libp2p-routing-helpers"  // 导入路由助手包，用于辅助路由功能
    "github.com/libp2p/go-libp2p/core/peer"  // 导入对等节点包，用于处理对等节点信息
    "github.com/libp2p/go-libp2p/core/routing"  // 导入路由包，用于处理路由功能
    "github.com/multiformats/go-multihash"  // 导入多哈希包，用于处理多种哈希算法
)

var (
    _ routinghelpers.ProvideManyRouter = &Composer{}  // Composer类型实现了ProvideManyRouter接口
    _ routing.Routing                  = &Composer{}  // Composer类型实现了Routing接口
)

type Composer struct {
    GetValueRouter      routing.Routing  // 获取值的路由
    PutValueRouter      routing.Routing  // 存储值的路由
    FindPeersRouter     routing.Routing  // 查找对等节点的路由
    FindProvidersRouter routing.Routing  // 查找提供者的路由
    ProvideRouter       routing.Routing  // 提供路由
}

func (c *Composer) Provide(ctx context.Context, cid cid.Cid, provide bool) error {
    log.Debug("composer: calling provide: ", cid)  // 记录调用提供函数的日志信息
    err := c.ProvideRouter.Provide(ctx, cid, provide)  // 调用提供路由的提供函数
    if err != nil {
        log.Debug("composer: calling provide: ", cid, " error: ", err)  // 记录提供函数调用出错的日志信息
    }

    return err  // 返回错误信息
}

func (c *Composer) ProvideMany(ctx context.Context, keys []multihash.Multihash) error {
    log.Debug("composer: calling provide many: ", len(keys))  // 记录调用提供多个函数的日志信息
    pmr, ok := c.ProvideRouter.(routinghelpers.ProvideManyRouter)  // 判断提供路由是否实现了提供多个路由接口
    if !ok {
        log.Debug("composer: provide many is not implemented on the actual router")  // 记录提供多个函数未在实际路由上实现的日志信息
        return nil  // 返回空值
    }

    err := pmr.ProvideMany(ctx, keys)  // 调用提供多个函数
    if err != nil {
        log.Debug("composer: calling provide many error: ", err)  // 记录调用提供多个函数出错的日志信息
    }

    return err  // 返回错误信息
}

func (c *Composer) Ready() bool {
    log.Debug("composer: calling ready")  // 记录调用准备函数的日志信息
    pmr, ok := c.ProvideRouter.(routinghelpers.ReadyAbleRouter)  // 判断提供路由是否实现了准备路由接口
    if !ok {
        return true  // 如果未实现准备路由接口，则返回true
    }

    ready := pmr.Ready()  // 调用准备函数

    log.Debug("composer: calling ready result: ", ready)  // 记录调用准备函数的结果日志信息

    return ready  // 返回准备结果
}

func (c *Composer) FindProvidersAsync(ctx context.Context, cid cid.Cid, count int) <-chan peer.AddrInfo {
    log.Debug("composer: calling findProvidersAsync: ", cid)  // 记录调用查找提供者异步函数的日志信息
    return c.FindProvidersRouter.FindProvidersAsync(ctx, cid, count)  // 返回查找提供者异步函数的结果
}
func (c *Composer) FindPeer(ctx context.Context, pid peer.ID) (peer.AddrInfo, error) {
    // 调试日志：调用 findPeer 方法，传入参数 pid
    log.Debug("composer: calling findPeer: ", pid)
    // 调用 FindPeersRouter 的 FindPeer 方法，传入参数 ctx 和 pid
    addr, err := c.FindPeersRouter.FindPeer(ctx, pid)
    // 如果出现错误，记录错误日志
    if err != nil {
        log.Debug("composer: calling findPeer error: ", pid, addr.String(), err)
    }
    // 返回地址信息和错误
    return addr, err
}

func (c *Composer) PutValue(ctx context.Context, key string, val []byte, opts ...routing.Option) error {
    // 调试日志：调用 putValue 方法，传入参数 key 和 val 的长度
    log.Debug("composer: calling putValue: ", key, len(val))
    // 调用 PutValueRouter 的 PutValue 方法，传入参数 ctx、key、val 和 opts
    err := c.PutValueRouter.PutValue(ctx, key, val, opts...)
    // 如果出现错误，记录错误日志
    if err != nil {
        log.Debug("composer: calling putValue error: ", key, len(val), err)
    }
    // 返回错误
    return err
}

func (c *Composer) GetValue(ctx context.Context, key string, opts ...routing.Option) ([]byte, error) {
    // 调试日志：调用 getValue 方法，传入参数 key
    log.Debug("composer: calling getValue: ", key)
    // 调用 GetValueRouter 的 GetValue 方法，传入参数 ctx、key 和 opts
    val, err := c.GetValueRouter.GetValue(ctx, key, opts...)
    // 如果出现错误，记录错误日志
    if err != nil {
        log.Debug("composer: calling getValue error: ", key, len(val), err)
    }
    // 返回值和错误
    return val, err
}

func (c *Composer) SearchValue(ctx context.Context, key string, opts ...routing.Option) (<-chan []byte, error) {
    // 调试日志：调用 searchValue 方法，传入参数 key
    log.Debug("composer: calling searchValue: ", key)
    // 调用 GetValueRouter 的 SearchValue 方法，传入参数 ctx、key 和 opts
    ch, err := c.GetValueRouter.SearchValue(ctx, key, opts...)

    // 避免在不支持 SearchValue 方法的实现上出现空通道
    if errors.Is(err, routing.ErrNotFound) && ch == nil {
        out := make(chan []byte)
        close(out)
        return out, err
    }

    // 如果出现错误，记录错误日志
    if err != nil {
        log.Debug("composer: calling searchValue error: ", key, err)
    }

    // 返回通道和错误
    return ch, err
}

func (c *Composer) Bootstrap(ctx context.Context) error {
    // 调试日志：调用 bootstrap 方法
    log.Debug("composer: calling bootstrap")
    // 调用 FindPeersRouter 的 Bootstrap 方法，传入参数 ctx
    errfp := c.FindPeersRouter.Bootstrap(ctx)
    // 调用 FindProvidersRouter 的 Bootstrap 方法，传入参数 ctx
    errfps := c.FindProvidersRouter.Bootstrap(ctx)
    // 调用 GetValueRouter 的 Bootstrap 方法，传入参数 ctx
    errgv := c.GetValueRouter.Bootstrap(ctx)
    // 调用 PutValueRouter 的 Bootstrap 方法，传入参数 ctx
    errpv := c.PutValueRouter.Bootstrap(ctx)
    // 调用 ProvideRouter 的 Bootstrap 方法，传入参数 ctx
    errp := c.ProvideRouter.Bootstrap(ctx)
    // 将所有错误合并为一个错误
    err := multierror.Append(errfp, errfps, errgv, errpv, errp)
    # 如果错误不为空，则记录调用引导错误并返回错误
    if err != nil:
        log.Debug("composer: calling bootstrap error: ", err)
    # 返回错误
    return err
# 闭合前面的函数定义
```