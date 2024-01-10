# `kubo\client\rpc\dht.go`

```
package rpc

import (
    "context"
    "encoding/json"

    "github.com/ipfs/boxo/path"  // 导入路径相关的包
    caopts "github.com/ipfs/kubo/core/coreiface/options"  // 导入选项相关的包
    "github.com/libp2p/go-libp2p/core/peer"  // 导入对等节点相关的包
    "github.com/libp2p/go-libp2p/core/routing"  // 导入路由相关的包
)

type DhtAPI HttpApi  // 定义 DhtAPI 结构体，继承自 HttpApi

func (api *DhtAPI) FindPeer(ctx context.Context, p peer.ID) (peer.AddrInfo, error) {
    var out struct {  // 定义结构体 out
        Type      routing.QueryEventType  // 定义 Type 字段，表示查询事件类型
        Responses []peer.AddrInfo  // 定义 Responses 字段，表示对等节点地址信息的切片
    }
    resp, err := api.core().Request("dht/findpeer", p.String()).Send(ctx)  // 发送请求，查找对等节点
    if err != nil {  // 如果发生错误
        return peer.AddrInfo{}, err  // 返回空的对等节点地址信息和错误
    }
    if resp.Error != nil {  // 如果响应中包含错误
        return peer.AddrInfo{}, resp.Error  // 返回空的对等节点地址信息和响应中的错误
    }
    defer resp.Close()  // 延迟关闭响应
    dec := json.NewDecoder(resp.Output)  // 创建 JSON 解码器
    for {  // 无限循环
        if err := dec.Decode(&out); err != nil {  // 解码响应到结构体 out
            return peer.AddrInfo{}, err  // 如果发生错误，返回空的对等节点地址信息和错误
        }
        if out.Type == routing.FinalPeer {  // 如果查询事件类型为 FinalPeer
            return out.Responses[0], nil  // 返回第一个响应的对等节点地址信息和空错误
        }
    }
}

func (api *DhtAPI) FindProviders(ctx context.Context, p path.Path, opts ...caopts.DhtFindProvidersOption) (<-chan peer.AddrInfo, error) {
    options, err := caopts.DhtFindProvidersOptions(opts...)  // 获取查找提供者的选项
    if err != nil {  // 如果发生错误
        return nil, err  // 返回空的通道和错误
    }

    rp, _, err := api.core().ResolvePath(ctx, p)  // 解析路径
    if err != nil {  // 如果发生错误
        return nil, err  // 返回空的通道和错误
    }

    resp, err := api.core().Request("dht/findprovs", rp.RootCid().String()).  // 发送请求，查找提供者
        Option("num-providers", options.NumProviders).  // 设置选项，指定提供者数量
        Send(ctx)
    if err != nil {  // 如果发生错误
        return nil, err  // 返回空的通道和错误
    }
    if resp.Error != nil {  // 如果响应中包含错误
        return nil, resp.Error  // 返回空的通道和响应中的错误
    }
    res := make(chan peer.AddrInfo)  // 创建对等节点地址信息的通道
    # 创建一个匿名函数并立即执行
    go func() {
        # 延迟关闭响应体
        defer resp.Close()
        # 延迟关闭结果通道
        defer close(res)
        # 创建一个 JSON 解码器，从响应体中读取数据
        dec := json.NewDecoder(resp.Output)

        # 无限循环，读取并处理 JSON 数据
        for {
            # 定义一个结构体用于存储 JSON 数据
            var out struct {
                Extra     string
                Type      routing.QueryEventType
                Responses []peer.AddrInfo
            }

            # 解码 JSON 数据到结构体中
            if err := dec.Decode(&out); err != nil {
                return // todo: handle this somehow
            }
            # 如果查询事件类型为错误，则返回
            if out.Type == routing.QueryError {
                return // usually a 'not found' error
                # todo: handle other errors
            }
            # 如果查询事件类型为提供者，则将结果发送到结果通道中
            if out.Type == routing.Provider {
                for _, pi := range out.Responses {
                    select {
                    case res <- pi:
                    case <-ctx.Done():
                        return
                    }
                }
            }
        }
    }()

    # 返回结果通道和空值
    return res, nil
# 定义 DhtAPI 结构体的 Provide 方法，接收路径和选项参数，返回错误信息
func (api *DhtAPI) Provide(ctx context.Context, p path.Path, opts ...caopts.DhtProvideOption) error {
    # 解析选项参数，获取选项和错误信息
    options, err := caopts.DhtProvideOptions(opts...)
    # 如果有错误信息，返回错误
    if err != nil {
        return err
    }

    # 解析路径，获取解析后的路径、根 CID 和错误信息
    rp, _, err := api.core().ResolvePath(ctx, p)
    # 如果有错误信息，返回错误
    if err != nil {
        return err
    }

    # 发起 DHT 提供请求，提供根 CID 字符串，并设置递归选项，执行请求
    return api.core().Request("dht/provide", rp.RootCid().String()).
        Option("recursive", options.Recursive).
        Exec(ctx, nil)
}

# 定义 DhtAPI 结构体的 core 方法，返回 HttpApi 结构体指针
func (api *DhtAPI) core() *HttpApi {
    return (*HttpApi)(api)
}
```