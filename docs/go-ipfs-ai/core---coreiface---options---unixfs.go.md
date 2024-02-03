# `kubo\core\coreiface\options\unixfs.go`

```go
package options

import (
    "errors"
    "fmt"

    dag "github.com/ipfs/boxo/ipld/merkledag"
    cid "github.com/ipfs/go-cid"
    mh "github.com/multiformats/go-multihash"
)

type Layout int

const (
    BalancedLayout Layout = iota
    TrickleLayout
)

type UnixfsAddSettings struct {
    CidVersion int  // CID 版本号
    MhType     uint64  // 多哈希类型

    Inline       bool  // 是否内联
    InlineLimit  int  // 内联限制
    RawLeaves    bool  // 是否使用原始叶子节点
    RawLeavesSet bool  // 是否设置了原始叶子节点

    Chunker string  // 分块器
    Layout  Layout  // 布局类型

    Pin      bool  // 是否固定
    OnlyHash bool  // 是否仅哈希
    FsCache  bool  // 是否使用文件系统缓存
    NoCopy   bool  // 是否不复制

    Events   chan<- interface{}  // 事件通道
    Silent   bool  // 是否静默
    Progress bool  // 是否显示进度
}

type UnixfsLsSettings struct {
    ResolveChildren   bool  // 是否解析子节点
    UseCumulativeSize bool  // 是否使用累积大小
}

type (
    UnixfsAddOption func(*UnixfsAddSettings) error
    UnixfsLsOption  func(*UnixfsLsSettings) error
)

func UnixfsAddOptions(opts ...UnixfsAddOption) (*UnixfsAddSettings, cid.Prefix, error) {
    options := &UnixfsAddSettings{
        CidVersion: -1,  // CID 版本号初始化为-1
        MhType:     mh.SHA2_256,  // 多哈希类型初始化为 SHA2_256

        Inline:       false,  // 内联初始化为 false
        InlineLimit:  32,  // 内联限制初始化为32
        RawLeaves:    false,  // 原始叶子节点初始化为 false
        RawLeavesSet: false,  // 原始叶子节点设置初始化为 false

        Chunker: "size-262144",  // 分块器初始化为 size-262144
        Layout:  BalancedLayout,  // 布局类型初始化为 BalancedLayout

        Pin:      false,  // 固定初始化为 false
        OnlyHash: false,  // 仅哈希初始化为 false
        FsCache:  false,  // 文件系统缓存初始化为 false
        NoCopy:   false,  // 不复制初始化为 false

        Events:   nil,  // 事件通道初始化为 nil
        Silent:   false,  // 静默初始化为 false
        Progress: false,  // 进度初始化为 false
    }

    for _, opt := range opts {
        err := opt(options)  // 应用选项
        if err != nil {
            return nil, cid.Prefix{}, err  // 如果应用选项出错，返回错误
        }
    }

    // nocopy -> rawblocks
    if options.NoCopy && !options.RawLeaves {
        // fixed?
        if options.RawLeavesSet {
            return nil, cid.Prefix{}, fmt.Errorf("nocopy option requires '--raw-leaves' to be enabled as well")  // 如果设置了原始叶子节点，但不复制选项为真，则返回错误
        }

        // No, satisfy mandatory constraint.
        options.RawLeaves = true  // 否则，设置原始叶子节点为真
    }

    // (hash != "sha2-256") -> CIDv1
    # 如果选项中的哈希类型不是 SHA2_256
    if options.MhType != mh.SHA2_256:
        # 根据 CID 版本进行不同的处理
        switch options.CidVersion:
            # 对于 CID 版本 0，返回错误信息
            case 0:
                return nil, cid.Prefix{}, errors.New("CIDv0 only supports sha2-256")
            # 对于 CID 版本 1 或 -1，将选项中的 CID 版本设置为 1
            case 1, -1:
                options.CidVersion = 1
            # 对于其他 CID 版本，返回错误信息
            default:
                return nil, cid.Prefix{}, fmt.Errorf("unknown CID version: %d", options.CidVersion)
    # 如果选项中的哈希类型是 SHA2_256
    else:
        # 如果选项中的 CID 版本小于 0
        if options.CidVersion < 0:
            # 将选项中的 CID 版本设置为 0（默认为 CIDv0）
            # 默认为 CIDv0
            options.CidVersion = 0

    # 如果选项中的 CID 版本大于 0 并且未设置原始叶子节点
    if options.CidVersion > 0 && !options.RawLeavesSet:
        # 将选项中的原始叶子节点设置为 true
        options.RawLeaves = true

    # 根据选项中的 CID 版本获取前缀
    prefix, err := dag.PrefixForCidVersion(options.CidVersion)
    # 如果出现错误，返回错误信息
    if err != nil:
        return nil, cid.Prefix{}, err

    # 设置前缀中的哈希类型为选项中的哈希类型
    prefix.MhType = options.MhType
    # 设置前缀中的哈希长度为 -1
    prefix.MhLength = -1

    # 返回选项、前缀和空错误信息
    return options, prefix, nil
// UnixfsLsOptions 函数用于创建 UnixfsLsSettings 结构体，并根据传入的选项进行设置
func UnixfsLsOptions(opts ...UnixfsLsOption) (*UnixfsLsSettings, error) {
    // 创建默认的 UnixfsLsSettings 结构体
    options := &UnixfsLsSettings{
        ResolveChildren: true,
    }

    // 遍历传入的选项，并逐个应用到 options 结构体上
    for _, opt := range opts {
        err := opt(options)
        // 如果应用选项时出现错误，则返回错误
        if err != nil {
            return nil, err
        }
    }

    // 返回设置好的 options 结构体
    return options, nil
}

// unixfsOpts 结构体用于定义 Unixfs 相关的选项
type unixfsOpts struct{}

// CidVersion 方法用于设置 CID 的版本
func (unixfsOpts) CidVersion(version int) UnixfsAddOption {
    return func(settings *UnixfsAddSettings) error {
        settings.CidVersion = version
        return nil
    }
}

// Hash 方法用于设置使用的哈希函数
func (unixfsOpts) Hash(mhtype uint64) UnixfsAddOption {
    return func(settings *UnixfsAddSettings) error {
        settings.MhType = mhtype
        return nil
    }
}

// RawLeaves 方法用于设置是否使用原始块作为叶子节点
func (unixfsOpts) RawLeaves(enable bool) UnixfsAddOption {
    return func(settings *UnixfsAddSettings) error {
        settings.RawLeaves = enable
        settings.RawLeavesSet = true
        return nil
    }
}

// Inline 方法用于设置是否将小块内联到 CID 中
func (unixfsOpts) Inline(enable bool) UnixfsAddOption {
    return func(settings *UnixfsAddSettings) error {
        settings.Inline = enable
        return nil
    }
}

// InlineLimit 方法用于设置内联的字节数限制
//
// 注意：虽然没有硬性限制字节数的上限，但应该保持在合理的低值，比如 64
// 实现可能选择将小于此限制的块直接编码到 CID 中，而不是存储和通过哈希寻址
// 默认值为 32 字节
//
// 注意：虽然没有硬性限制字节数的上限，但应该保持在合理的低值，比如 64；实现可能选择
// 将小于此限制的块直接编码到 CID 中，而不是存储和通过哈希寻址
// 设置内联限制，拒绝任何大于该限制的内容
func (unixfsOpts) InlineLimit(limit int) UnixfsAddOption {
    return func(settings *UnixfsAddSettings) error {
        // 设置内联限制
        settings.InlineLimit = limit
        return nil
    }
}

// 指定用于分块算法的设置
//
// 默认值：size-262144，格式：
// size-[bytes] - 将数据分割成大小为n字节的块的简单分块器
// rabin-[min]-[avg]-[max] - Rabin分块器
func (unixfsOpts) Chunker(chunker string) UnixfsAddOption {
    return func(settings *UnixfsAddSettings) error {
        // 设置分块器
        settings.Chunker = chunker
        return nil
    }
}

// 布局告诉添加器如何在叶子节点之间平衡数据。
// options.BalancedLayout是默认值，它针对静态可寻址文件进行了优化。
// options.TrickleLayout 针对流式数据进行了优化
func (unixfsOpts) Layout(layout Layout) UnixfsAddOption {
    return func(settings *UnixfsAddSettings) error {
        // 设置布局
        settings.Layout = layout
        return nil
    }
}

// Pin告诉添加器在添加后递归固定文件根
func (unixfsOpts) Pin(pin bool) UnixfsAddOption {
    return func(settings *UnixfsAddSettings) error {
        // 设置Pin
        settings.Pin = pin
        return nil
    }
}

// HashOnly将使添加器计算数据哈希值，而不将其存储在块存储中或向网络宣布
func (unixfsOpts) HashOnly(hashOnly bool) UnixfsAddOption {
    return func(settings *UnixfsAddSettings) error {
        // 设置仅计算哈希值
        settings.OnlyHash = hashOnly
        return nil
    }
}

// Events指定用于报告正在进行的添加操作的事件的通道
//
// 请注意，如果此通道阻塞，可能会减慢添加器的速度
func (unixfsOpts) Events(sink chan<- interface{}) UnixfsAddOption {
    return func(settings *UnixfsAddSettings) error {
        // 设置事件通道
        settings.Events = sink
        return nil
    }
}

// Silent减少事件输出
func (unixfsOpts) Silent(silent bool) UnixfsAddOption {
    # 返回一个函数，该函数接受一个UnixfsAddSettings类型的参数，并返回一个error类型的值
    return func(settings *UnixfsAddSettings) error {
        # 将函数参数中的Silent字段设置为silent变量的值
        settings.Silent = silent
        # 返回nil，表示没有错误发生
        return nil
    }
// Progress告诉adder是否启用进度事件
func (unixfsOpts) Progress(enable bool) UnixfsAddOption {
    return func(settings *UnixfsAddSettings) error {
        settings.Progress = enable
        return nil
    }
}

// FsCache告诉adder检查文件存储是否存在预先存在的块
//
// 实验性质
func (unixfsOpts) FsCache(enable bool) UnixfsAddOption {
    return func(settings *UnixfsAddSettings) error {
        settings.FsCache = enable
        return nil
    }
}

// NoCopy告诉adder使用filestore添加文件。意味着RawLeaves。
//
// 实验性质
func (unixfsOpts) Nocopy(enable bool) UnixfsAddOption {
    return func(settings *UnixfsAddSettings) error {
        settings.NoCopy = enable
        return nil
    }
}

func (unixfsOpts) ResolveChildren(resolve bool) UnixfsLsOption {
    return func(settings *UnixfsLsSettings) error {
        settings.ResolveChildren = resolve
        return nil
    }
}

func (unixfsOpts) UseCumulativeSize(use bool) UnixfsLsOption {
    return func(settings *UnixfsLsSettings) error {
        settings.UseCumulativeSize = use
        return nil
    }
}
```