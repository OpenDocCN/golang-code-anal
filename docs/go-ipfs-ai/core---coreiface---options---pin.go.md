# `kubo\core\coreiface\options\pin.go`

```go
// 导入 fmt 包
import "fmt"

// PinAddSettings 表示 PinAPI.Add 的设置
type PinAddSettings struct {
    Recursive bool
    Name      string
}

// PinLsSettings 表示 PinAPI.Ls 的设置
type PinLsSettings struct {
    Type     string
    Detailed bool
}

// PinIsPinnedSettings 表示 PinAPI.IsPinned 的设置
type PinIsPinnedSettings struct {
    WithType string
}

// PinRmSettings 表示 PinAPI.Rm 的设置
type PinRmSettings struct {
    Recursive bool
}

// PinUpdateSettings 表示 PinAPI.Update 的设置
type PinUpdateSettings struct {
    Unpin bool
}

// PinAddOption 是 PinAPI.Add 的选项签名
type PinAddOption func(*PinAddSettings) error

// PinLsOption 是 PinAPI.Ls 的选项签名
type PinLsOption func(*PinLsSettings) error

// PinIsPinnedOption 是 PinAPI.IsPinned 的选项签名
type PinIsPinnedOption func(*PinIsPinnedSettings) error

// PinRmOption 是 PinAPI.Rm 的选项签名
type PinRmOption func(*PinRmSettings) error

// PinUpdateOption 是 PinAPI.Update 的选项签名
type PinUpdateOption func(*PinUpdateSettings) error

// PinAddOptions 将一系列 PinAddOption 编译成一个可用的 PinAddSettings，并设置默认值
func PinAddOptions(opts ...PinAddOption) (*PinAddSettings, error) {
    options := &PinAddSettings{
        Recursive: true,
    }

    for _, opt := range opts {
        err := opt(options)
        if err != nil {
            return nil, err
        }
    }

    return options, nil
}

// PinLsOptions 将一系列 PinLsOption 编译成一个可用的 PinLsSettings，并设置默认值
func PinLsOptions(opts ...PinLsOption) (*PinLsSettings, error) {
    options := &PinLsSettings{
        Type: "all",
    }

    for _, opt := range opts {
        err := opt(options)
        if err != nil {
            return nil, err
        }
    }

    return options, nil
}
// 将一系列 PinIsPinnedOption 编译成一个可用的 PinIsPinnedSettings，并设置默认值
func PinIsPinnedOptions(opts ...PinIsPinnedOption) (*PinIsPinnedSettings, error) {
    // 设置默认的 WithType 为 "all"
    options := &PinIsPinnedSettings{
        WithType: "all",
    }

    // 遍历选项并应用到 options
    for _, opt := range opts {
        err := opt(options)
        if err != nil {
            return nil, err
        }
    }

    return options, nil
}

// 将一系列 PinRmOption 编译成一个可用的 PinRmSettings，并设置默认值
func PinRmOptions(opts ...PinRmOption) (*PinRmSettings, error) {
    // 设置默认的 Recursive 为 true
    options := &PinRmSettings{
        Recursive: true,
    }

    // 遍历选项并应用到 options
    for _, opt := range opts {
        if err := opt(options); err != nil {
            return nil, err
        }
    }

    return options, nil
}

// 将一系列 PinUpdateOption 编译成一个可用的 PinUpdateSettings，并设置默认值
func PinUpdateOptions(opts ...PinUpdateOption) (*PinUpdateSettings, error) {
    // 设置默认的 Unpin 为 true
    options := &PinUpdateSettings{
        Unpin: true,
    }

    // 遍历选项并应用到 options
    for _, opt := range opts {
        err := opt(options)
        if err != nil {
            return nil, err
        }
    }

    return options, nil
}

// 定义 pinOpts 结构体
type pinOpts struct {
    Ls       pinLsOpts
    IsPinned pinIsPinnedOpts
}

// 提供对 Pin API 的所有选项的访问
var Pin pinOpts

// 定义 pinLsOpts 结构体
type pinLsOpts struct{}

// All 是 Pin.Ls 的一个选项，将使其返回所有的 pins，这是默认值
func (pinLsOpts) All() PinLsOption {
    return Pin.Ls.pinType("all")
}

// Recursive 是 Pin.Ls 的一个选项，将使其只返回递归的 pins
func (pinLsOpts) Recursive() PinLsOption {
    return Pin.Ls.pinType("recursive")
}

// Direct 是 Pin.Ls 的一个选项，将使其只返回直接的（非递归的）pins
func (pinLsOpts) Direct() PinLsOption {
    return Pin.Ls.pinType("direct")
}
// Indirect 是 Pin.Ls 的一个选项，它将使其仅返回间接引用的 pin
// （由其他递归固定对象引用的对象）
func (pinLsOpts) Indirect() PinLsOption {
    return Pin.Ls.pinType("indirect")
}

// Type 是 Pin.Ls 的一个选项，它将使其仅返回给定类型的 pin
//
// 支持的值：
//   - "direct" - 直接固定的对象
//   - "recursive" - 递归 pin 的根
//   - "indirect" - 间接固定的对象（由递归固定的对象引用）
//   - "all" - 所有固定的对象（默认）
func (pinLsOpts) Type(typeStr string) (PinLsOption, error) {
    switch typeStr {
    case "all", "direct", "indirect", "recursive":
        return Pin.Ls.pinType(typeStr), nil
    default:
        return nil, fmt.Errorf("invalid type '%s', must be one of {direct, indirect, recursive, all}", typeStr)
    }
}

// pinType 是 Pin.Ls 的一个选项，允许指定应返回哪些 pin 类型
//
// 支持的值：
//   - "direct" - 直接固定的对象
//   - "recursive" - 递归 pin 的根
//   - "indirect" - 间接固定的对象（由递归固定的对象引用）
//   - "all" - 所有固定的对象（默认）
func (pinLsOpts) pinType(t string) PinLsOption {
    return func(settings *PinLsSettings) error {
        settings.Type = t
        return nil
    }
}

// Detailed 是 Pin.Ls 的一个选项，用于设置是否返回详细信息，例如 pin 名称和模式。
func (pinLsOpts) Detailed(detailed bool) PinLsOption {
    return func(settings *PinLsSettings) error {
        settings.Detailed = detailed
        return nil
    }
}

type pinIsPinnedOpts struct{}

// All 是 Pin.IsPinned 的一个选项，它将使其在所有类型的 pin 中搜索。
// 这是默认设置
func (pinIsPinnedOpts) All() PinIsPinnedOption {
    return Pin.IsPinned.pinType("all")
}

// Recursive 是 Pin.IsPinned 的一个选项，它将使其仅在递归 pin 中搜索
// 返回一个递归选项，用于指定是否递归搜索
func (pinIsPinnedOpts) Recursive() PinIsPinnedOption {
    return Pin.IsPinned.pinType("recursive")
}

// Direct 是 Pin.IsPinned 的一个选项，将使其仅在直接（非递归）固定中搜索
func (pinIsPinnedOpts) Direct() PinIsPinnedOption {
    return Pin.IsPinned.pinType("direct")
}

// Indirect 是 Pin.IsPinned 的一个选项，将使其仅搜索间接固定（由其他递归固定对象引用的对象）
func (pinIsPinnedOpts) Indirect() PinIsPinnedOption {
    return Pin.IsPinned.pinType("indirect")
}

// Type 是 Pin.IsPinned 的一个选项，将使其仅搜索给定类型的固定。
//
// 支持的值：
//   - "direct" - 直接固定的对象
//   - "recursive" - 递归固定的根
//   - "indirect" - 间接固定的对象（由递归固定对象引用）
//   - "all" - 所有固定的对象（默认）
func (pinIsPinnedOpts) Type(typeStr string) (PinIsPinnedOption, error) {
    switch typeStr {
    case "all", "direct", "indirect", "recursive":
        return Pin.IsPinned.pinType(typeStr), nil
    default:
        return nil, fmt.Errorf("invalid type '%s', must be one of {direct, indirect, recursive, all}", typeStr)
    }
}

// pinType 是 Pin.IsPinned 的一个选项，允许指定给定固定的固定类型，加快搜索速度。
//
// 支持的值：
//   - "direct" - 直接固定的对象
//   - "recursive" - 递归固定的根
//   - "indirect" - 间接固定的对象（由递归固定对象引用）
//   - "all" - 所有固定的对象（默认）
func (pinIsPinnedOpts) pinType(t string) PinIsPinnedOption {
    return func(settings *PinIsPinnedSettings) error {
        settings.WithType = t
        return nil
    }
}

// Recursive 是 Pin.Add 的一个选项，指定是否固定整个对象树还是只固定一个对象。默认值：true
func (pinOpts) Recursive(recursive bool) PinAddOption {
    # 返回一个函数，该函数接受一个PinAddSettings类型的参数，并返回一个error类型的值
    return func(settings *PinAddSettings) error {
        # 将函数参数中的Recursive属性设置为指定的值
        settings.Recursive = recursive
        # 返回空值，表示没有错误发生
        return nil
    }
// Name 是 Pin.Add 的一个选项，用于指定要添加到 pin 的可选名称。
func (pinOpts) Name(name string) PinAddOption {
    return func(settings *PinAddSettings) error {
        settings.Name = name
        return nil
    }
}

// RmRecursive 是 Pin.Rm 的一个选项，用于指定是否递归取消固定指定对象链接的对象。这不会删除其他递归 pin 引用的间接 pin。
func (pinOpts) RmRecursive(recursive bool) PinRmOption {
    return func(settings *PinRmSettings) error {
        settings.Recursive = recursive
        return nil
    }
}

// Unpin 是 Pin.Update 的一个选项，用于指定是否删除旧的 pin。默认为 true。
func (pinOpts) Unpin(unpin bool) PinUpdateOption {
    return func(settings *PinUpdateSettings) error {
        settings.Unpin = unpin
        return nil
    }
}
```