# `kubo\core\coreapi\path.go`

```go
package coreapi

import (
    "context" // 导入上下文包，用于处理请求的上下文信息
    "errors" // 导入错误包，用于处理错误信息
    "fmt" // 导入格式化包，用于格式化输出

    "github.com/ipfs/boxo/namesys" // 导入命名系统包，用于处理命名解析
    "github.com/ipfs/kubo/tracing" // 导入追踪包，用于追踪请求

    "go.opentelemetry.io/otel/attribute" // 导入属性包，用于处理属性信息
    "go.opentelemetry.io/otel/trace" // 导入追踪包，用于追踪请求

    "github.com/ipfs/boxo/path" // 导入路径包，用于处理路径相关操作
    ipfspathresolver "github.com/ipfs/boxo/path/resolver" // 导入路径解析器包，用于解析路径
    ipld "github.com/ipfs/go-ipld-format" // 导入IPLD包，用于处理IPLD格式
    coreiface "github.com/ipfs/kubo/core/coreiface" // 导入核心接口包，用于定义核心接口
)

// ResolveNode resolves the path `p` using Unixfs resolver, gets and returns the
// resolved Node.
func (api *CoreAPI) ResolveNode(ctx context.Context, p path.Path) (ipld.Node, error) {
    ctx, span := tracing.Span(ctx, "CoreAPI", "ResolveNode", trace.WithAttributes(attribute.String("path", p.String()))) // 创建追踪上下文和追踪span
    defer span.End() // 在函数返回时结束span

    rp, _, err := api.ResolvePath(ctx, p) // 解析路径
    if err != nil {
        return nil, err
    }

    node, err := api.dag.Get(ctx, rp.RootCid()) // 获取节点
    if err != nil {
        return nil, err
    }
    return node, nil
}

// ResolvePath resolves the path `p` using Unixfs resolver, returns the
// resolved path.
func (api *CoreAPI) ResolvePath(ctx context.Context, p path.Path) (path.ImmutablePath, []string, error) {
    ctx, span := tracing.Span(ctx, "CoreAPI", "ResolvePath", trace.WithAttributes(attribute.String("path", p.String()))) // 创建追踪上下文和追踪span
    defer span.End() // 在函数返回时结束span

    res, err := namesys.Resolve(ctx, api.namesys, p) // 解析路径
    if errors.Is(err, namesys.ErrNoNamesys) {
        return path.ImmutablePath{}, nil, coreiface.ErrOffline
    } else if err != nil {
        return path.ImmutablePath{}, nil, err
    }
    p = res.Path // 更新路径为解析后的路径

    var resolver ipfspathresolver.Resolver // 声明路径解析器
    switch p.Namespace() {
    case path.IPLDNamespace:
        resolver = api.ipldPathResolver // 使用IPLD路径解析器
    case path.IPFSNamespace:
        resolver = api.unixFSPathResolver // 使用IPFS路径解析器
    default:
        return path.ImmutablePath{}, nil, fmt.Errorf("unsupported path namespace: %s", p.Namespace()) // 返回不支持的路径命名空间错误
    }

    imPath, err := path.NewImmutablePath(p) // 创建不可变路径
    if err != nil {
        return path.ImmutablePath{}, nil, err
    }
    # 使用 resolver.ResolveToLastNode 方法解析给定的 imPath，返回节点、剩余部分和错误信息
    node, remainder, err := resolver.ResolveToLastNode(ctx, imPath)
    # 如果有错误发生，则返回空的 ImmutablePath 对象、nil 和错误信息
    if err != nil {
        return path.ImmutablePath{}, nil, err
    }

    # 创建一个字符串切片，包含 p 的命名空间和节点的字符串表示
    segments := []string{p.Namespace(), node.String()}
    # 将剩余部分添加到切片中
    segments = append(segments, remainder...)

    # 使用 path.NewPathFromSegments 方法根据切片创建一个新的 Path 对象
    p, err = path.NewPathFromSegments(segments...)
    # 如果有错误发生，则返回空的 ImmutablePath 对象、nil 和错误信息
    if err != nil {
        return path.ImmutablePath{}, nil, err
    }

    # 使用 path.NewImmutablePath 方法根据 p 创建一个新的 ImmutablePath 对象
    imPath, err = path.NewImmutablePath(p)
    # 如果有错误发生，则返回空的 ImmutablePath 对象、nil 和错误信息
    if err != nil {
        return path.ImmutablePath{}, nil, err
    }

    # 返回创建的 ImmutablePath 对象、剩余部分和 nil（表示没有错误发生）
    return imPath, remainder, nil
# 闭合前面的函数定义
```