# `kubo\core\coreapi\dag.go`

```go
package coreapi

import (
    "context"

    dag "github.com/ipfs/boxo/ipld/merkledag"  // 导入 merkledag 包，重命名为 dag
    pin "github.com/ipfs/boxo/pinning/pinner"  // 导入 pinner 包，重命名为 pin
    cid "github.com/ipfs/go-cid"  // 导入 go-cid 包，重命名为 cid
    ipld "github.com/ipfs/go-ipld-format"  // 导入 go-ipld-format 包，重命名为 ipld
    "go.opentelemetry.io/otel/attribute"  // 导入 attribute 包
    "go.opentelemetry.io/otel/trace"  // 导入 trace 包

    "github.com/ipfs/kubo/tracing"  // 导入 tracing 包
)

type dagAPI struct {
    ipld.DAGService  // 定义结构体 dagAPI，包含 ipld.DAGService 接口
    core *CoreAPI  // coreAPI 结构体指针
}

type pinningAdder CoreAPI  // 定义类型 pinningAdder，类型为 CoreAPI

func (adder *pinningAdder) Add(ctx context.Context, nd ipld.Node) error {
    ctx, span := tracing.Span(ctx, "CoreAPI.PinningAdder", "Add", trace.WithAttributes(attribute.String("node", nd.String())))  // 创建追踪 span
    defer span.End()  // 延迟执行 span.End() 方法
    defer adder.blockstore.PinLock(ctx).Unlock(ctx)  // 延迟执行解锁操作

    if err := adder.dag.Add(ctx, nd); err != nil {  // 调用 dag.Add 方法
        return err  // 返回错误
    }

    if err := adder.pinning.PinWithMode(ctx, nd.Cid(), pin.Recursive, ""); err != nil {  // 调用 pinning.PinWithMode 方法
        return err  // 返回错误
    }

    return adder.pinning.Flush(ctx)  // 调用 pinning.Flush 方法
}

func (adder *pinningAdder) AddMany(ctx context.Context, nds []ipld.Node) error {
    ctx, span := tracing.Span(ctx, "CoreAPI.PinningAdder", "AddMany", trace.WithAttributes(attribute.Int("nodes.count", len(nds))))  // 创建追踪 span
    defer span.End()  // 延迟执行 span.End() 方法
    defer adder.blockstore.PinLock(ctx).Unlock(ctx)  // 延迟执行解锁操作

    if err := adder.dag.AddMany(ctx, nds); err != nil {  // 调用 dag.AddMany 方法
        return err  // 返回错误
    }

    cids := cid.NewSet()  // 创建新的 cid 集合

    for _, nd := range nds {  // 遍历 nds 切片
        c := nd.Cid()  // 获取节点的 cid
        if cids.Visit(c) {  // 判断是否已经访问过该 cid
            if err := adder.pinning.PinWithMode(ctx, c, pin.Recursive, ""); err != nil {  // 调用 pinning.PinWithMode 方法
                return err  // 返回错误
            }
        }
    }

    return adder.pinning.Flush(ctx)  // 调用 pinning.Flush 方法
}

func (api *dagAPI) Pinning() ipld.NodeAdder {
    return (*pinningAdder)(api.core)  // 返回 pinningAdder 类型的对象
}

func (api *dagAPI) Session(ctx context.Context) ipld.NodeGetter {
    return dag.NewSession(ctx, api.DAGService)  // 返回新的会话对象
}

var (
    _ ipld.DAGService  = (*dagAPI)(nil)  // 空白标识符，用于断言接口实现
    _ dag.SessionMaker = (*dagAPI)(nil)  // 空白标识符，用于断言接口实现
)
```