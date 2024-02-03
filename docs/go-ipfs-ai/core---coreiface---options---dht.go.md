# `kubo\core\coreiface\options\dht.go`

```go
package options

// DhtProvideSettings 定义了 Dht.Provide 方法的选项
type DhtProvideSettings struct {
    Recursive bool
}

// DhtFindProvidersSettings 定义了 Dht.FindProviders 方法的选项
type DhtFindProvidersSettings struct {
    NumProviders int
}

// DhtProvideOption 和 DhtFindProvidersOption 是函数类型，用于设置 DhtProvideSettings 和 DhtFindProvidersSettings 的选项
type (
    DhtProvideOption       func(*DhtProvideSettings) error
    DhtFindProvidersOption func(*DhtFindProvidersSettings) error
)

// DhtProvideOptions 用于设置 Dht.Provide 方法的选项
func DhtProvideOptions(opts ...DhtProvideOption) (*DhtProvideSettings, error) {
    options := &DhtProvideSettings{
        Recursive: false,
    }

    for _, opt := range opts {
        err := opt(options)
        if err != nil {
            return nil, err
        }
    }
    return options, nil
}

// DhtFindProvidersOptions 用于设置 Dht.FindProviders 方法的选项
func DhtFindProvidersOptions(opts ...DhtFindProvidersOption) (*DhtFindProvidersSettings, error) {
    options := &DhtFindProvidersSettings{
        NumProviders: 20,
    }

    for _, opt := range opts {
        err := opt(options)
        if err != nil {
            return nil, err
        }
    }
    return options, nil
}

// dhtOpts 结构体用于定义 Dht 的选项
type dhtOpts struct{}

// Dht 是 dhtOpts 结构体的实例
var Dht dhtOpts

// Recursive 是 Dht.Provide 方法的选项，指定是否递归提供给定路径
func (dhtOpts) Recursive(recursive bool) DhtProvideOption {
    return func(settings *DhtProvideSettings) error {
        settings.Recursive = recursive
        return nil
    }
}

// NumProviders 是 Dht.FindProviders 方法的选项，指定要查找的对等体数量，默认为 20
func (dhtOpts) NumProviders(numProviders int) DhtFindProvidersOption {
    return func(settings *DhtFindProvidersSettings) error {
        settings.NumProviders = numProviders
        return nil
    }
}
```