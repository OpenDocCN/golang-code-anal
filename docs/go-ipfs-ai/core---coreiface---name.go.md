# `kubo\core\coreiface\name.go`

```go
package iface

import (
    "context"  // 导入上下文包，用于处理请求的取消、超时等操作
    "errors"   // 导入错误包，用于定义和处理错误

    "github.com/ipfs/boxo/ipns"  // 导入IPNS相关的包
    "github.com/ipfs/boxo/path"  // 导入路径相关的包
    "github.com/ipfs/kubo/core/coreiface/options"  // 导入选项相关的包
)

var ErrResolveFailed = errors.New("could not resolve name")  // 定义一个解析失败的错误变量

type IpnsResult struct {
    path.Path  // 定义一个结构体，包含路径和错误信息
    Err error
}

// NameAPI 接口定义了IPNS的接口规范
//
// IPNS是一个PKI命名空间，其中名称是公钥的哈希值，私钥用于发布新的（签名的）值。
// 在发布和解析中，默认使用的名称是节点自己的PeerID，它是公钥的哈希值。
//
// 您可以使用.Key API来列出和生成更多的名称及其相应的密钥。
type NameAPI interface {
    // Publish 用于发布新的IPNS名称
    Publish(ctx context.Context, path path.Path, opts ...options.NamePublishOption) (ipns.Name, error)

    // Resolve 用于尝试解析指定名称的最新版本
    Resolve(ctx context.Context, name string, opts ...options.NameResolveOption) (path.Path, error)

    // Search 是Resolve的一个版本，它在发现路径时输出路径，减少了首个条目的时间
    //
    // 注意：默认情况下，从通道读取的所有路径都被视为不安全，除了最新的（通道读取缓冲区中的最后一个路径）。
    Search(ctx context.Context, name string, opts ...options.NameResolveOption) (<-chan IpnsResult, error)
}
```