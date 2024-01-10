# `kubo\core\coreiface\options\global.go`

```
// 定义 ApiSettings 结构体，包含 Offline 和 FetchBlocks 两个布尔类型字段
type ApiSettings struct {
    Offline     bool
    FetchBlocks bool
}

// 定义 ApiOption 函数类型，用于修改 ApiSettings 结构体
type ApiOption func(*ApiSettings) error

// ApiOptions 函数接收多个 ApiOption 参数，并返回 ApiSettings 结构体指针和错误
func ApiOptions(opts ...ApiOption) (*ApiSettings, error) {
    // 初始化 options 结构体指针，设置默认值
    options := &ApiSettings{
        Offline:     false,
        FetchBlocks: true,
    }

    // 调用 ApiOptionsTo 函数，将 options 结构体指针和 opts 参数传入
    return ApiOptionsTo(options, opts...)
}

// ApiOptionsTo 函数接收 ApiSettings 结构体指针和多个 ApiOption 参数，并返回 ApiSettings 结构体指针和错误
func ApiOptionsTo(options *ApiSettings, opts ...ApiOption) (*ApiSettings, error) {
    // 遍历 opts 参数，依次调用每个 ApiOption 函数，修改 options 结构体
    for _, opt := range opts {
        err := opt(options)
        if err != nil {
            return nil, err
        }
    }
    return options, nil
}

// 定义 apiOpts 结构体
type apiOpts struct{}

// 定义 Api 结构体变量
var Api apiOpts

// Offline 方法接收一个布尔类型参数，返回一个 ApiOption 函数，用于修改 Offline 字段
func (apiOpts) Offline(offline bool) ApiOption {
    return func(settings *ApiSettings) error {
        settings.Offline = offline
        return nil
    }
}

// FetchBlocks 方法接收一个布尔类型参数，返回一个 ApiOption 函数，用于修改 FetchBlocks 字段
func (apiOpts) FetchBlocks(fetch bool) ApiOption {
    return func(settings *ApiSettings) error {
        settings.FetchBlocks = fetch
        return nil
    }
}
```