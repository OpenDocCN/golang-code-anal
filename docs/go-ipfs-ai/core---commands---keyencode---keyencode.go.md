# `kubo\core\commands\keyencode\keyencode.go`

```go
package keyencode

import (
    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入 ipfs 命令行工具包
    peer "github.com/libp2p/go-libp2p/core/peer"  // 导入 libp2p 核心对等网络包
    mbase "github.com/multiformats/go-multibase"  // 导入 multibase 包
)

const ipnsKeyFormatOptionName = "ipns-base"  // 定义常量，表示 IPNS 键的格式选项名称

var OptionIPNSBase = cmds.StringOption(ipnsKeyFormatOptionName, "Encoding used for keys: Can either be a multibase encoded CID or a base58btc encoded multihash. Takes {b58mh|base36|k|base32|b...}.").WithDefault("base36")  // 定义 IPNS 键的格式选项，包含默认值和描述信息

type KeyEncoder struct {
    baseEnc *mbase.Encoder  // 定义 KeyEncoder 结构体，包含 multibase 编码器
}

func KeyEncoderFromString(formatLabel string) (KeyEncoder, error) {
    switch formatLabel {
    case "b58mh", "v0":  // 如果格式标签是 "b58mh" 或 "v0"
        return KeyEncoder{}, nil  // 返回空的 KeyEncoder 结构体和空错误
    default:  // 否则
        if enc, err := mbase.EncoderByName(formatLabel); err != nil {  // 如果根据格式标签获取编码器出错
            return KeyEncoder{}, err  // 返回空的 KeyEncoder 结构体和错误
        } else {
            return KeyEncoder{&enc}, nil  // 返回包含编码器的 KeyEncoder 结构体和空错误
        }
    }
}

func (enc KeyEncoder) FormatID(id peer.ID) string {
    if enc.baseEnc == nil {  // 如果编码器为空
        return id.String()  // 返回对等网络 ID 的字符串表示
    }
    if s, err := peer.ToCid(id).StringOfBase(enc.baseEnc.Encoding()); err != nil {  // 如果将对等网络 ID 转换为 CID，并使用编码器进行编码出错
        panic(err)  // 抛出异常
    } else {
        return s  // 返回编码后的字符串
    }
}
```