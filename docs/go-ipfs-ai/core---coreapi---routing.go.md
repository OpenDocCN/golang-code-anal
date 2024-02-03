# `kubo\core\coreapi\routing.go`

```go
package coreapi

import (
    "context" // 导入上下文包，用于处理请求的上下文信息
    "errors" // 导入错误包，用于处理错误信息
    "strings" // 导入字符串包，用于处理字符串操作

    coreiface "github.com/ipfs/kubo/core/coreiface" // 导入核心接口包
    caopts "github.com/ipfs/kubo/core/coreiface/options" // 导入核心接口选项包
    peer "github.com/libp2p/go-libp2p/core/peer" // 导入对等节点包
)

type RoutingAPI CoreAPI // 定义 RoutingAPI 类型为 CoreAPI

func (r *RoutingAPI) Get(ctx context.Context, key string) ([]byte, error) {
    if !r.nd.IsOnline { // 如果节点不在线
        return nil, coreiface.ErrOffline // 返回空值和离线错误
    }

    dhtKey, err := normalizeKey(key) // 对键进行规范化处理
    if err != nil {
        return nil, err // 如果出错，返回空值和错误
    }

    return r.routing.GetValue(ctx, dhtKey) // 返回路由获取的值
}

func (r *RoutingAPI) Put(ctx context.Context, key string, value []byte, opts ...caopts.RoutingPutOption) error {
    options, err := caopts.RoutingPutOptions(opts...) // 获取路由放置选项
    if err != nil {
        return err // 如果出错，返回错误
    }

    err = r.checkOnline(options.AllowOffline) // 检查是否允许离线操作
    if err != nil {
        return err // 如果出错，返回错误
    }

    dhtKey, err := normalizeKey(key) // 对键进行规范化处理
    if err != nil {
        return err // 如果出错，返回错误
    }

    return r.routing.PutValue(ctx, dhtKey, value) // 路由放置值
}

func normalizeKey(s string) (string, error) {
    parts := strings.Split(s, "/") // 使用斜杠分割键
    if len(parts) != 3 || // 如果分割后的部分不等于3
        parts[0] != "" || // 或者第一个部分不为空
        !(parts[1] == "ipns" || parts[1] == "pk") { // 或者第二个部分不是 "ipns" 或 "pk"
        return "", errors.New("invalid key") // 返回空字符串和无效键错误
    }

    k, err := peer.Decode(parts[2]) // 解码第三部分
    if err != nil {
        return "", err // 如果出错，返回错误
    }
    return strings.Join(append(parts[:2], string(k)), "/"), nil // 返回连接后的键和空值
}
```