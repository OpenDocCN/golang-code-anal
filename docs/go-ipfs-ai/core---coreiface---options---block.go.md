# `kubo\core\coreiface\options\block.go`

```go
// 导入必要的包
package options

import (
    "fmt"

    cid "github.com/ipfs/go-cid"
    mc "github.com/multiformats/go-multicodec"
    mh "github.com/multiformats/go-multihash"
)

// 定义 BlockPutSettings 结构体，包含 CidPrefix 和 Pin 两个字段
type BlockPutSettings struct {
    CidPrefix cid.Prefix
    Pin       bool
}

// 定义 BlockRmSettings 结构体，包含 Force 字段
type BlockRmSettings struct {
    Force bool
}

// 定义 BlockPutOption 和 BlockRmOption 两个函数类型
type (
    BlockPutOption func(*BlockPutSettings) error
    BlockRmOption  func(*BlockRmSettings) error
)

// BlockPutOptions 函数，接收 BlockPutOption 类型的可变参数，返回 BlockPutSettings 结构体指针和 error
func BlockPutOptions(opts ...BlockPutOption) (*BlockPutSettings, error) {
    // 创建 cidPrefix 变量，类型为 cid.Prefix
    var cidPrefix cid.Prefix

    // 设置 cidPrefix 的各个字段值
    cidPrefix.Version = 1
    cidPrefix.Codec = uint64(mc.Raw)
    cidPrefix.MhType = mh.SHA2_256
    cidPrefix.MhLength = -1

    // 创建 options 变量，类型为 BlockPutSettings 结构体指针
    options := &BlockPutSettings{
        CidPrefix: cidPrefix,
        Pin:       false,
    }

    // 遍历 opts 切片，对 options 进行设置
    for _, opt := range opts {
        err := opt(options)
        if err != nil {
            return nil, err
        }
    }

    return options, nil
}

// BlockRmOptions 函数，接收 BlockRmOption 类型的可变参数，返回 BlockRmSettings 结构体指针和 error
func BlockRmOptions(opts ...BlockRmOption) (*BlockRmSettings, error) {
    // 创建 options 变量，类型为 BlockRmSettings 结构体指针
    options := &BlockRmSettings{
        Force: false,
    }

    // 遍历 opts 切片，对 options 进行设置
    for _, opt := range opts {
        err := opt(options)
        if err != nil {
            return nil, err
        }
    }
    return options, nil
}

// 定义 blockOpts 结构体
type blockOpts struct{}

// 创建 Block 变量，类型为 blockOpts 结构体
var Block blockOpts

// CidCodec 方法，接收 codecName 字符串，返回 BlockPutOption 类型的函数
func (blockOpts) CidCodec(codecName string) BlockPutOption {
    return func(settings *BlockPutSettings) error {
        if codecName == "" {
            return nil
        }
        // 根据 codecName 获取对应的 code
        code, err := codeFromName(codecName)
        if err != nil {
            return err
        }
        // 设置 settings 的 CidPrefix.Codec 字段
        settings.CidPrefix.Codec = uint64(code)
        return nil
    }
}

// Map string to code from go-multicodec
// 根据编解码器名称获取对应的编解码器代码和可能的错误
func codeFromName(codecName string) (mc.Code, error) {
    var cidCodec mc.Code
    err := cidCodec.Set(codecName)
    return cidCodec, err
}

// Format 是 Block.Put 的一个旧选项，用于指定用于序列化对象的多编解码器。
// 仅用于向后兼容。请改用 CidCodec。
func (blockOpts) Format(format string) BlockPutOption {
    return func(settings *BlockPutSettings) error {
        if format == "" {
            return nil
        }
        // 选择 CIDv0 支持以向后兼容
        if format == "v0" {
            settings.CidPrefix.Version = 0
        }

        // 修复 dag-pb（0x70）的旧（无效）名称
        if format == "v0" || format == "protobuf" {
            format = "dag-pb"
        }

        // 修复 dag-cbor（0x71）的无效名称
        if format == "cbor" {
            format = "dag-cbor"
        }

        // 根据传递的“format”名称设置代码
        code, err := codeFromName(format)
        if err != nil {
            return err
        }
        settings.CidPrefix.Codec = uint64(code)

        // 如果是 CIDv0，请确保所有参数兼容
        // （理论上 go-cid 会验证这一点，但我们希望提供更好的错误）
        pref := settings.CidPrefix
        if pref.Version == 0 {
            if pref.Codec != uint64(mc.DagPb) {
                return fmt.Errorf("only dag-pb is allowed with CIDv0")
            }
            if pref.MhType != mh.SHA2_256 || (pref.MhLength != -1 && pref.MhLength != 32) {
                return fmt.Errorf("only sha2-255-32 is allowed with CIDv0")
            }
        }

        return nil
    }
}

// Hash 是 Block.Put 的一个选项，用于指定在对对象进行哈希时使用的多哈希设置。
// 默认为 mh.SHA2_256（0x12）。
// 如果 mhLen 设置为 -1，则将使用哈希的默认长度
func (blockOpts) Hash(mhType uint64, mhLen int) BlockPutOption {
    # 定义一个函数，接受一个BlockPutSettings类型的参数，并返回一个error类型的值
    return func(settings *BlockPutSettings) error {
        # 设置settings的CidPrefix的MhType属性为mhType
        settings.CidPrefix.MhType = mhType
        # 设置settings的CidPrefix的MhLength属性为mhLen
        settings.CidPrefix.MhLength = mhLen
        # 返回一个空的error，表示函数执行成功
        return nil
    }
// Pin 是 Block.Put 的一个选项，用于指定是否（递归地）固定添加的块
func (blockOpts) Pin(pin bool) BlockPutOption {
    return func(settings *BlockPutSettings) error {
        settings.Pin = pin
        return nil
    }
}

// Force 是 Block.Rm 的一个选项，当设置为 true 时，将忽略不存在的块
func (blockOpts) Force(force bool) BlockRmOption {
    return func(settings *BlockRmSettings) error {
        settings.Force = force
        return nil
    }
}
```