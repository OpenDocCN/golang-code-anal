# `kubo\core\coreapi\pin.go`

```go
package coreapi

import (
    "context"
    "fmt"

    bserv "github.com/ipfs/boxo/blockservice"
    offline "github.com/ipfs/boxo/exchange/offline"
    "github.com/ipfs/boxo/ipld/merkledag"
    "github.com/ipfs/boxo/path"
    pin "github.com/ipfs/boxo/pinning/pinner"
    "github.com/ipfs/go-cid"
    coreiface "github.com/ipfs/kubo/core/coreiface"
    caopts "github.com/ipfs/kubo/core/coreiface/options"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/trace"

    "github.com/ipfs/kubo/tracing"
)

type PinAPI CoreAPI

func (api *PinAPI) Add(ctx context.Context, p path.Path, opts ...caopts.PinAddOption) error {
    // 创建一个新的 span，并设置 span 的名称和属性
    ctx, span := tracing.Span(ctx, "CoreAPI.PinAPI", "Add", trace.WithAttributes(attribute.String("path", p.String())))
    // 在函数返回前结束 span
    defer span.End()

    // 解析路径对应的节点
    dagNode, err := api.core().ResolveNode(ctx, p)
    if err != nil {
        return fmt.Errorf("pin: %s", err)
    }

    // 获取 PinAddOption 的设置
    settings, err := caopts.PinAddOptions(opts...)
    if err != nil {
        return err
    }

    // 设置 span 的属性
    span.SetAttributes(attribute.Bool("recursive", settings.Recursive))

    // 在函数返回前解锁 PinLock
    defer api.blockstore.PinLock(ctx).Unlock(ctx)

    // 对节点进行固化
    err = api.pinning.Pin(ctx, dagNode, settings.Recursive, settings.Name)
    if err != nil {
        return fmt.Errorf("pin: %s", err)
    }

    // 提供节点的数据
    if err := api.provider.Provide(dagNode.Cid()); err != nil {
        return err
    }

    // 刷新固化
    return api.pinning.Flush(ctx)
}

func (api *PinAPI) Ls(ctx context.Context, opts ...caopts.PinLsOption) (<-chan coreiface.Pin, error) {
    // 创建一个新的 span，并设置 span 的名称
    ctx, span := tracing.Span(ctx, "CoreAPI.PinAPI", "Ls")
    // 在函数返回前结束 span
    defer span.End()

    // 获取 PinLsOption 的设置
    settings, err := caopts.PinLsOptions(opts...)
    if err != nil {
        return nil, err
    }

    // 设置 span 的属性
    span.SetAttributes(attribute.String("type", settings.Type))

    // 根据不同的类型进行处理
    switch settings.Type {
    case "all", "direct", "indirect", "recursive":
    default:
        return nil, fmt.Errorf("invalid type '%s', must be one of {direct, indirect, recursive, all}", settings.Type)
    }
    # 调用 api.pinLsAll 方法，传入 ctx, settings.Type, settings.Detailed 作为参数，并返回结果和空值
    return api.pinLsAll(ctx, settings.Type, settings.Detailed), nil
// 检查给定路径是否已经被固定，返回固定类型和是否固定的布尔值
func (api *PinAPI) IsPinned(ctx context.Context, p path.Path, opts ...caopts.PinIsPinnedOption) (string, bool, error) {
    // 创建一个跟踪 span，并在函数结束时结束该 span
    ctx, span := tracing.Span(ctx, "CoreAPI.PinAPI", "IsPinned", trace.WithAttributes(attribute.String("path", p.String())))
    defer span.End()

    // 解析路径并返回解析后的路径、根 CID 和错误信息
    resolved, _, err := api.core().ResolvePath(ctx, p)
    if err != nil {
        return "", false, fmt.Errorf("error resolving path: %s", err)
    }

    // 获取 PinIsPinnedOptions，并返回设置和错误信息
    settings, err := caopts.PinIsPinnedOptions(opts...)
    if err != nil {
        return "", false, err
    }

    // 设置 span 的属性
    span.SetAttributes(attribute.String("withtype", settings.WithType))

    // 将字符串类型转换为固定模式，如果转换失败则返回错误信息
    mode, ok := pin.StringToMode(settings.WithType)
    if !ok {
        return "", false, fmt.Errorf("invalid type '%s', must be one of {direct, indirect, recursive, all}", settings.WithType)
    }

    // 调用 pinning.IsPinnedWithType 方法检查给定路径是否已经被固定
    return api.pinning.IsPinnedWithType(ctx, resolved.RootCid(), mode)
}

// Rm pin rm api
func (api *PinAPI) Rm(ctx context.Context, p path.Path, opts ...caopts.PinRmOption) error {
    // 创建一个跟踪 span，并在函数结束时结束该 span
    ctx, span := tracing.Span(ctx, "CoreAPI.PinAPI", "Rm", trace.WithAttributes(attribute.String("path", p.String())))
    defer span.End()

    // 解析路径并返回解析后的路径、根 CID 和错误信息
    rp, _, err := api.core().ResolvePath(ctx, p)
    if err != nil {
        return err
    }

    // 获取 PinRmOptions，并返回设置和错误信息
    settings, err := caopts.PinRmOptions(opts...)
    if err != nil {
        return err
    }

    // 设置 span 的属性
    span.SetAttributes(attribute.Bool("recursive", settings.Recursive))

    // 注意：取消固定后，固定集将刷新到块存储，因此我们需要锁定以防止并发垃圾回收
    // 在函数结束时解锁 PinLock
    defer api.blockstore.PinLock(ctx).Unlock(ctx)

    // 调用 pinning.Unpin 方法取消固定，并返回错误信息
    if err = api.pinning.Unpin(ctx, rp.RootCid(), settings.Recursive); err != nil {
        return err
    }

    // 调用 pinning.Flush 方法刷新固定
    return api.pinning.Flush(ctx)
}

// 更新固定
func (api *PinAPI) Update(ctx context.Context, from path.Path, to path.Path, opts ...caopts.PinUpdateOption) error {
    # 创建一个新的追踪 span，并设置 span 名称为 "CoreAPI.PinAPI"，操作名称为 "Update"，并添加一些属性
    ctx, span := tracing.Span(ctx, "CoreAPI.PinAPI", "Update", trace.WithAttributes(
        attribute.String("from", from.String()),
        attribute.String("to", to.String()),
    ))
    # 延迟执行 span.End()，确保在函数返回前结束 span
    defer span.End()

    # 使用给定的选项创建 pin 更新设置
    settings, err := caopts.PinUpdateOptions(opts...)
    if err != nil {
        return err
    }

    # 设置 span 的属性，标记是否为取消固定
    span.SetAttributes(attribute.Bool("unpin", settings.Unpin))

    # 解析 from 路径，获取根 CID
    fp, _, err := api.core().ResolvePath(ctx, from)
    if err != nil {
        return err
    }

    # 解析 to 路径，获取根 CID
    tp, _, err := api.core().ResolvePath(ctx, to)
    if err != nil {
        return err
    }

    # 延迟执行 api.blockstore.PinLock(ctx).Unlock(ctx)，确保在函数返回前解锁 pin
    defer api.blockstore.PinLock(ctx).Unlock(ctx)

    # 更新固定状态
    err = api.pinning.Update(ctx, fp.RootCid(), tp.RootCid(), settings.Unpin)
    if err != nil {
        return err
    }

    # 刷新 pinning 状态
    return api.pinning.Flush(ctx)
// 定义结构体 pinStatus，包含错误、CID、状态和坏节点列表
type pinStatus struct {
    err      error
    cid      cid.Cid
    ok       bool
    badNodes []coreiface.BadPinNode
}

// 定义结构体 badNode，用于 PinVerifyRes
type badNode struct {
    path path.ImmutablePath
    err  error
}

// 实现 pinStatus 结构体的 Ok 方法，返回状态是否正常
func (s *pinStatus) Ok() bool {
    return s.ok
}

// 实现 pinStatus 结构体的 BadNodes 方法，返回坏节点列表
func (s *pinStatus) BadNodes() []coreiface.BadPinNode {
    return s.badNodes
}

// 实现 pinStatus 结构体的 Err 方法，返回错误信息
func (s *pinStatus) Err() error {
    return s.err
}

// 实现 badNode 结构体的 Path 方法，返回路径信息
func (n *badNode) Path() path.ImmutablePath {
    return n.path
}

// 实现 badNode 结构体的 Err 方法，返回错误信息
func (n *badNode) Err() error {
    return n.err
}

// 实现 PinAPI 结构体的 Verify 方法，用于验证
func (api *PinAPI) Verify(ctx context.Context) (<-chan coreiface.PinStatus, error) {
    // 创建一个用于存储已访问节点的 map
    visited := make(map[cid.Cid]*pinStatus)
    // 获取块存储
    bs := api.blockstore
    // 创建一个新的 DAG 服务
    DAG := merkledag.NewDAGService(bserv.New(bs, offline.Exchange(bs)))
    // 获取节点链接的函数
    getLinks := merkledag.GetLinksWithDAG(DAG)

    // 定义递归函数 checkPin，用于检查节点的状态
    var checkPin func(root cid.Cid) *pinStatus
    checkPin = func(root cid.Cid) *pinStatus {
        // 创建一个 span 用于追踪
        ctx, span := tracing.Span(ctx, "CoreAPI.PinAPI", "Verify.CheckPin", trace.WithAttributes(attribute.String("cid", root.String())))
        defer span.End()

        // 如果节点已经访问过，则直接返回其状态
        if status, ok := visited[root]; ok {
            return status
        }

        // 获取节点的链接
        links, err := getLinks(ctx, root)
        if err != nil {
            // 如果获取链接出错，则创建一个包含错误信息的 pinStatus
            status := &pinStatus{ok: false, cid: root}
            status.badNodes = []coreiface.BadPinNode{&badNode{path: path.FromCid(root), err: err}}
            visited[root] = status
            return status
        }

        // 创建一个包含节点状态的 pinStatus
        status := &pinStatus{ok: true, cid: root}
        for _, lnk := range links {
            // 递归调用 checkPin 函数，检查链接节点的状态
            res := checkPin(lnk.Cid)
            if !res.ok {
                status.ok = false
                status.badNodes = append(status.badNodes, res.badNodes...)
            }
        }

        visited[root] = status
        return status
    }

    // 创建一个用于存储 PinStatus 的通道
    out := make(chan coreiface.PinStatus)
    # 创建一个匿名的 goroutine 函数
    go func() {
        # 延迟关闭 out 通道，确保在函数退出时关闭通道
        defer close(out)
        # 遍历 api.pinning.RecursiveKeys 返回的通道，获取每个 p
        for p := range api.pinning.RecursiveKeys(ctx, false) {
            # 声明一个指向 pinStatus 结构体的指针 res
            var res *pinStatus
            # 如果 p.Err 不为 nil，则将 err 字段设置为 p.Err
            if p.Err != nil {
                res = &pinStatus{err: p.Err}
            } else {
                # 否则，调用 checkPin 函数检查 p.Pin.Key，并将结果赋给 res
                res = checkPin(p.Pin.Key)
            }
            # 选择性地执行以下两个 case 中的一个
            select {
            # 如果 ctx 被取消，则立即返回
            case <-ctx.Done():
                return
            # 否则，将 res 发送到 out 通道
            case out <- res:
            }
        }
    }()
    # 返回 out 通道和 nil 错误
    return out, nil
// 定义一个结构体，包含 pinType、path、name 和 err 四个字段
type pinInfo struct {
    pinType string
    path    path.ImmutablePath
    name    string
    err     error
}

// 返回 pinInfo 结构体中的 path 字段
func (p *pinInfo) Path() path.ImmutablePath {
    return p.path
}

// 返回 pinInfo 结构体中的 pinType 字段
func (p *pinInfo) Type() string {
    return p.pinType
}

// 返回 pinInfo 结构体中的 name 字段
func (p *pinInfo) Name() string {
    return p.name
}

// 返回 pinInfo 结构体中的 err 字段
func (p *pinInfo) Err() error {
    return p.err
}

// pinLsAll 是一个内部函数，用于返回一个 pin 列表
//
// 调用者必须持续读取结果，直到通道关闭，以防止泄漏获取 pin 的 goroutine
func (api *PinAPI) pinLsAll(ctx context.Context, typeStr string, detailed bool) <-chan coreiface.Pin {
    out := make(chan coreiface.Pin, 1)  // 创建一个带缓冲区的通道

    emittedSet := cid.NewSet()  // 创建一个 cid 集合

    // 定义一个函数 AddToResultKeys，用于向结果集中添加 cid、name 和 typeStr
    AddToResultKeys := func(c cid.Cid, name, typeStr string) error {
        if emittedSet.Visit(c) {  // 如果 cid 已经存在于集合中
            select {
            case out <- &pinInfo{  // 将 pinInfo 结构体的实例发送到通道中
                pinType: typeStr,
                name:    name,
                path:    path.FromCid(c),
            }:
            case <-ctx.Done():  // 如果上下文被取消，则返回上下文的错误
                return ctx.Err()
            }
        }
        return nil
    }

    // 返回通道 out
    return out
}

// 返回 coreiface.CoreAPI 接口
func (api *PinAPI) core() coreiface.CoreAPI {
    return (*CoreAPI)(api)
}
```