# `kubo\client\rpc\routing.go`

```go
package rpc

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "context"  // 导入 context 包，用于处理上下文
    "encoding/base64"  // 导入 base64 包，用于进行 base64 编码和解码
    "encoding/json"  // 导入 json 包，用于处理 JSON 数据

    "github.com/ipfs/kubo/core/coreiface/options"  // 导入 options 包，用于处理核心接口的选项
    "github.com/libp2p/go-libp2p/core/routing"  // 导入 routing 包，用于处理 libp2p 核心路由
)

type RoutingAPI HttpApi  // 定义 RoutingAPI 结构体，继承自 HttpApi 结构体

func (api *RoutingAPI) Get(ctx context.Context, key string) ([]byte, error) {
    resp, err := api.core().Request("routing/get", key).Send(ctx)  // 发送请求获取指定 key 的数据
    if err != nil {
        return nil, err  // 如果出现错误，返回空值和错误信息
    }
    if resp.Error != nil {
        return nil, resp.Error  // 如果响应中包含错误信息，返回空值和错误信息
    }
    defer resp.Close()  // 延迟关闭响应

    var out routing.QueryEvent  // 定义 routing.QueryEvent 类型的变量 out

    dec := json.NewDecoder(resp.Output)  // 创建 JSON 解码器
    if err := dec.Decode(&out); err != nil {  // 解码响应数据到 out 变量
        return nil, err  // 如果解码出错，返回空值和错误信息
    }

    res, err := base64.StdEncoding.DecodeString(out.Extra)  // 对 out.Extra 进行 base64 解码
    if err != nil {
        return nil, err  // 如果解码出错，返回空值和错误信息
    }

    return res, nil  // 返回解码后的数据和空错误信息
}

func (api *RoutingAPI) Put(ctx context.Context, key string, value []byte, opts ...options.RoutingPutOption) error {
    var cfg options.RoutingPutSettings  // 定义 options.RoutingPutSettings 类型的变量 cfg
    for _, o := range opts {  // 遍历选项参数
        if err := o(&cfg); err != nil {  // 如果处理选项出错，返回错误信息
            return err
        }
    }

    resp, err := api.core().Request("routing/put", key).
        Option("allow-offline", cfg.AllowOffline).
        FileBody(bytes.NewReader(value)).
        Send(ctx)  // 发送请求将指定 key 和 value 存储到路由中
    if err != nil {
        return err  // 如果出现错误，返回错误信息
    }
    if resp.Error != nil {
        return resp.Error  // 如果响应中包含错误信息，返回错误信息
    }
    return nil  // 返回空错误信息
}

func (api *RoutingAPI) core() *HttpApi {
    return (*HttpApi)(api)  // 返回类型转换后的 HttpApi 结构体指针
}
```