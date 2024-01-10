# `kubo\core\commands\cmdenv\cidbase.go`

```
package cmdenv

import (
    "fmt"
    "strings"

    cid "github.com/ipfs/go-cid"
    cidenc "github.com/ipfs/go-cidutil/cidenc"
    cmds "github.com/ipfs/go-ipfs-cmds"
    mbase "github.com/multiformats/go-multibase"
)

var (
    OptionCidBase              = cmds.StringOption("cid-base", "Multibase encoding used for version 1 CIDs in output.")
    OptionUpgradeCidV0InOutput = cmds.BoolOption("upgrade-cidv0-in-output", "Upgrade version 0 to version 1 CIDs in output.")
)

// GetCidEncoder processes the `cid-base` and `output-cidv1` options and
// returns a encoder to use based on those parameters.
func GetCidEncoder(req *cmds.Request) (cidenc.Encoder, error) {
    return getCidBase(req, true)
}

// GetLowLevelCidEncoder is like GetCidEncoder but meant to be used by
// lower level commands.  It differs from GetCidEncoder in that CIDv0
// are not, by default, auto-upgraded to CIDv1.
func GetLowLevelCidEncoder(req *cmds.Request) (cidenc.Encoder, error) {
    return getCidBase(req, false)
}

func getCidBase(req *cmds.Request, autoUpgrade bool) (cidenc.Encoder, error) {
    base, _ := req.Options[OptionCidBase.Name()].(string)  // 获取请求中的 cid-base 选项值
    upgrade, upgradeDefined := req.Options[OptionUpgradeCidV0InOutput.Name()].(bool)  // 获取请求中的 upgrade-cidv0-in-output 选项值

    e := cidenc.Default()  // 创建一个默认的 CID 编码器

    if base != "" {  // 如果 cid-base 选项有值
        var err error
        e.Base, err = mbase.EncoderByName(base)  // 根据选项值设置编码器的基础编码
        if err != nil {
            return e, err  // 如果出现错误，返回错误信息
        }
        if autoUpgrade {
            e.Upgrade = true  // 如果需要自动升级，设置编码器的升级标志为 true
        }
    }

    if upgradeDefined {  // 如果定义了升级选项
        e.Upgrade = upgrade  // 设置编码器的升级标志为选项值
    }

    return e, nil  // 返回编码器和空错误
}

// CidBaseDefined returns true if the `cid-base` option is specified
// on the command line
func CidBaseDefined(req *cmds.Request) bool {
    base, _ := req.Options["cid-base"].(string)  // 获取请求中的 cid-base 选项值
    return base != ""  // 如果选项值不为空，返回 true
}

// CidEncoderFromPath creates a new encoder that is influenced from
// the encoded Cid in a Path.  For CidV0 the multibase from the base
// encoder is used and automatic upgrades are disabled.  For CidV1 the
// 从 CID 中提取 multibase，并启用升级。
//
// 这个逻辑是有意模糊的，将匹配任何形式为 `CidLike`、`CidLike/...` 或 `/namespace/CidLike/...` 的内容。
//
// 例如：
//
// * Qm...
// * Qm.../...
// * /ipfs/Qm...
// * /ipns/bafybeiahnxfi7fpmr5wtxs2imx4abnyn7fdxeiox7xxjem6zuiioqkh6zi/...
// * /bzz/bafybeiahnxfi7fpmr5wtxs2imx4abnyn7fdxeiox7xxjem6zuiioqkh6zi/...
func CidEncoderFromPath(p string) (cidenc.Encoder, error) {
    components := strings.SplitN(p, "/", 4)

    var maybeCid string
    if components[0] != "" {
        // 没有前导斜杠，第一个组件可能是类似 CID 的内容。
        maybeCid = components[0]
    } else if len(components) < 3 {
        // 组件不足以包含 CID。
        return cidenc.Encoder{}, fmt.Errorf("no cid in path: %s", p)
    } else {
        maybeCid = components[2]
    }
    c, err := cid.Decode(maybeCid)
    if err != nil {
        // 好的，不是类似 CID 的东西。保持当前编码器。
        return cidenc.Encoder{}, fmt.Errorf("no cid in path: %s", p)
    }
    if c.Version() == 0 {
        // 版本 0，使用 base58 非升级编码器。
        return cidenc.Default(), nil
    }

    // 版本 1+，提取 multibase 编码。
    encoding, _, err := mbase.Decode(maybeCid)
    if err != nil {
        // 这应该是不可能的，我们已经解码了 CID。
        panic(fmt.Sprintf("BUG: failed to get multibase decoder for CID %s", maybeCid))
    }

    return cidenc.Encoder{Base: mbase.MustNewEncoder(encoding), Upgrade: true}, nil
}
```