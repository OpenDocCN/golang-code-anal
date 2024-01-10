# `kubo\config\routing.go`

```
package config

import (
    "encoding/json"  // 导入 JSON 编解码包
    "fmt"  // 导入格式化输出包
    "runtime"  // 导入运行时包
)

// Routing 定义了 libp2p 路由的配置选项
type Routing struct {
    // Type 设置默认的守护程序路由模式
    //
    // 可以是 "auto", "autoclient", "dht", "dhtclient", "dhtserver", "none", 或 "custom" 中的一个
    // 当未设置或设置为 "auto" 时，使用 DHT 和隐式路由
    // 当设置为 "custom" 时，使用用户提供的 Routing.Routers
    Type *OptionalString `json:",omitempty"`

    AcceleratedDHTClient bool  // 加速的 DHT 客户端

    Routers Routers  // 路由器

    Methods Methods  // 方法
}

type Router struct {
    // 路由器类型 ID。参见 RouterType 以获取更多信息
    Type RouterType  // 路由器类型

    // Parameters 是该路由器可能需要的额外配置
    // HTTP 路由器的常见参数是 "Endpoint"
    Parameters interface{}  // 参数
}

type (
    Routers map[string]RouterParser  // 路由器映射
    Methods map[MethodName]Method  // 方法映射
)

func (m Methods) Check() error {
    // 检查支持的方法
    for _, mn := range MethodNameList {
        _, ok := m[mn]
        if !ok {
            return fmt.Errorf("method name %q is missing from Routing.Methods config param", mn)
        }
    }

    // 检查不支持的方法
    for k := range m {
        seen := false
        for _, mn := range MethodNameList {
            if mn == k {
                seen = true
                break
            }
        }

        if seen {
            continue
        }

        return fmt.Errorf("method name %q is not a supported method on Routing.Methods config param", k)
    }

    return nil
}

type RouterParser struct {
    Router  // 路由器
}

func (r *RouterParser) UnmarshalJSON(b []byte) error {
    out := Router{}
    out.Parameters = &json.RawMessage{}
    if err := json.Unmarshal(b, &out); err != nil {
        return err
    }
    raw := out.Parameters.(*json.RawMessage)

    var p interface{}
    switch out.Type {
    case RouterTypeHTTP:
        p = &HTTPRouterParams{}
    case RouterTypeDHT:
        p = &DHTRouterParams{}
    # 根据路由器类型选择不同的参数结构体
    case RouterTypeSequential:
        # 如果路由器类型为顺序，则创建顺序路由器参数结构体
        p = &ComposableRouterParams{}
    case RouterTypeParallel:
        # 如果路由器类型为并行，则创建并行路由器参数结构体
        p = &ComposableRouterParams{}
    }

    # 将 JSON 数据解析到参数结构体中
    if err := json.Unmarshal(*raw, &p); err != nil {
        # 如果解析失败，则返回错误
        return err
    }

    # 将解析后的参数赋值给路由器对象
    r.Router.Type = out.Type
    r.Router.Parameters = p

    # 返回空错误，表示操作成功
    return nil
// Type is the routing type.
// Depending of the type we need to instantiate different Routing implementations.
// 路由类型
// 根据类型实例化不同的路由实现

type RouterType string

const (
    RouterTypeHTTP       RouterType = "http"       // HTTP JSON API for delegated routing systems (IPIP-337).
    RouterTypeDHT        RouterType = "dht"        // DHT router.
    RouterTypeSequential RouterType = "sequential" // Router helper to execute several routers sequentially.
    RouterTypeParallel   RouterType = "parallel"   // Router helper to execute several routers in parallel.
)

type DHTMode string

const (
    DHTModeServer DHTMode = "server"
    DHTModeClient DHTMode = "client"
    DHTModeAuto   DHTMode = "auto"
)

type MethodName string

const (
    MethodNameProvide       MethodName = "provide"
    MethodNameFindProviders MethodName = "find-providers"
    MethodNameFindPeers     MethodName = "find-peers"
    MethodNameGetIPNS       MethodName = "get-ipns"
    MethodNamePutIPNS       MethodName = "put-ipns"
)

var MethodNameList = []MethodName{MethodNameProvide, MethodNameFindPeers, MethodNameFindProviders, MethodNameGetIPNS, MethodNamePutIPNS}

type HTTPRouterParams struct {
    // Endpoint is the URL where the routing implementation will point to get the information.
    // Endpoint 是路由实现将指向以获取信息的 URL。
    Endpoint string

    // MaxProvideBatchSize determines the maximum amount of CIDs sent per batch.
    // Servers might not accept more than 100 elements per batch. 100 elements by default.
    // MaxProvideBatchSize 确定每批发送的 CID 的最大数量。
    // 服务器可能不会接受超过每批 100 个元素。默认为 100 个元素。
    MaxProvideBatchSize int

    // MaxProvideConcurrency determines the number of threads used when providing content. GOMAXPROCS by default.
    // MaxProvideConcurrency 确定提供内容时使用的线程数。默认为 GOMAXPROCS。
    MaxProvideConcurrency int
}

func (hrp *HTTPRouterParams) FillDefaults() {
    if hrp.MaxProvideBatchSize == 0 {
        hrp.MaxProvideBatchSize = 100
    }

    if hrp.MaxProvideConcurrency == 0 {
        hrp.MaxProvideConcurrency = runtime.GOMAXPROCS(0)
    }
}

type DHTRouterParams struct {
    Mode                 DHTMode
    AcceleratedDHTClient bool `json:",omitempty"`
    PublicIPNetwork      bool
    // DHT 路由参数
    // 模式
    // 加速 DHT 客户端
    // 公共 IP 网络
}
# 结构体定义：可组合的路由参数，包含路由配置和超时时间
type ComposableRouterParams struct {
    # 路由配置列表
    Routers []ConfigRouter
    # 可选的超时时间
    Timeout *OptionalDuration `json:",omitempty"`
}

# 结构体定义：路由配置，包含路由名称、超时时间、是否忽略错误、执行后的可选时间
type ConfigRouter struct {
    # 路由名称
    RouterName   string
    # 超时时间
    Timeout      Duration
    # 是否忽略错误
    IgnoreErrors bool
    # 可选的执行后时间
    ExecuteAfter *OptionalDuration `json:",omitempty"`
}

# 结构体定义：方法，包含路由名称
type Method struct {
    # 路由名称
    RouterName string
}
```