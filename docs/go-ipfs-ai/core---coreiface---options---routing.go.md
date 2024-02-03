# `kubo\core\coreiface\options\routing.go`

```go
package options

// 定义 RoutingPutSettings 结构体，用于存储 RoutingPutOption 的设置
type RoutingPutSettings struct {
    AllowOffline bool
}

// 定义 RoutingPutOption 函数类型，用于设置 RoutingPutSettings
type RoutingPutOption func(*RoutingPutSettings) error

// RoutingPutOptions 函数用于应用 RoutingPutOption 到 RoutingPutSettings
func RoutingPutOptions(opts ...RoutingPutOption) (*RoutingPutSettings, error) {
    // 初始化 RoutingPutSettings 结构体
    options := &RoutingPutSettings{
        AllowOffline: false,
    }

    // 遍历传入的选项并应用到 options
    for _, opt := range opts {
        err := opt(options)
        if err != nil {
            return nil, err
        }
    }

    return options, nil
}

// 定义 putOpts 结构体
type putOpts struct{}

// 定义 Put 变量为 putOpts 类型
var Put putOpts

// AllowOffline 是 Routing.Put 的选项，指定是否允许在节点离线时发布。默认值为 false
func (putOpts) AllowOffline(allow bool) RoutingPutOption {
    // 返回一个函数，用于设置 AllowOffline 选项
    return func(settings *RoutingPutSettings) error {
        settings.AllowOffline = allow
        return nil
    }
}
```