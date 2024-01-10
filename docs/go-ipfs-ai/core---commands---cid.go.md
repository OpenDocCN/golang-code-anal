# `kubo\core\commands\cid.go`

```
package commands

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"  // 导入 io 包，用于实现 I/O 操作
    "sort"  // 导入 sort 包，用于对数据进行排序
    "strings"  // 导入 strings 包，用于处理字符串
    "unicode"  // 导入 unicode 包，用于处理 Unicode 字符

    verifcid "github.com/ipfs/boxo/verifcid"  // 导入 verifcid 包
    cid "github.com/ipfs/go-cid"  // 导入 cid 包
    cidutil "github.com/ipfs/go-cidutil"  // 导入 cidutil 包
    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入 cmds 包
    ipldmulticodec "github.com/ipld/go-ipld-prime/multicodec"  // 导入 ipldmulticodec 包
    mbase "github.com/multiformats/go-multibase"  // 导入 mbase 包
    mc "github.com/multiformats/go-multicodec"  // 导入 mc 包
    mhash "github.com/multiformats/go-multihash"  // 导入 mhash 包
)

var CidCmd = &cmds.Command{  // 定义 CidCmd 变量，类型为 *cmds.Command
    Helptext: cmds.HelpText{  // 设置 Helptext 属性
        Tagline: "Convert and discover properties of CIDs",  // 设置 Tagline 属性
    },
    Subcommands: map[string]*cmds.Command{  // 设置 Subcommands 属性，类型为 map[string]*cmds.Command
        "format": cidFmtCmd,  // 设置 "format" 键对应的值为 cidFmtCmd
        "base32": base32Cmd,  // 设置 "base32" 键对应的值为 base32Cmd
        "bases":  basesCmd,  // 设置 "bases" 键对应的值为 basesCmd
        "codecs": codecsCmd,  // 设置 "codecs" 键对应的值为 codecsCmd
        "hashes": hashesCmd,  // 设置 "hashes" 键对应的值为 hashesCmd
    },
    Extra: CreateCmdExtras(SetDoesNotUseRepo(true)),  // 设置 Extra 属性
}

const (  // 定义常量块
    cidFormatOptionName    = "f"  // 设置 cidFormatOptionName 常量
    cidVerisonOptionName   = "v"  // 设置 cidVerisonOptionName 常量
    cidCodecOptionName     = "mc"  // 设置 cidCodecOptionName 常量
    cidMultibaseOptionName = "b"  // 设置 cidMultibaseOptionName 常量
)

var cidFmtCmd = &cmds.Command{  // 定义 cidFmtCmd 变量，类型为 *cmds.Command
    Helptext: cmds.HelpText{  // 设置 Helptext 属性
        Tagline: "Format and convert a CID in various useful ways.",  // 设置 Tagline 属性
        LongDescription: `  // 设置 LongDescription 属性
Format and converts <cid>'s in various useful ways.

The optional format string is a printf style format string:
` + cidutil.FormatRef,  // 拼接 LongDescription 属性值
    },
    Arguments: []cmds.Argument{  // 设置 Arguments 属性，类型为 []cmds.Argument
        cmds.StringArg("cid", true, true, "CIDs to format.").EnableStdin(),  // 添加 StringArg 到 Arguments
    },
    Options: []cmds.Option{  // 设置 Options 属性，类型为 []cmds.Option
        cmds.StringOption(cidFormatOptionName, "Printf style format string.").WithDefault("%s"),  // 添加 StringOption 到 Options
        cmds.StringOption(cidVerisonOptionName, "CID version to convert to."),  // 添加 StringOption 到 Options
        cmds.StringOption(cidCodecOptionName, "CID multicodec to convert to."),  // 添加 StringOption 到 Options
        cmds.StringOption(cidMultibaseOptionName, "Multibase to display CID in."),  // 添加 StringOption 到 Options
    },
    # 定义一个函数 Run，接收请求、响应和环境参数，并返回错误
    Run: func(req *cmds.Request, resp cmds.ResponseEmitter, env cmds.Environment) error {
        # 从请求参数中获取格式字符串
        fmtStr, _ := req.Options[cidFormatOptionName].(string)
        # 从请求参数中获取 CID 版本字符串
        verStr, _ := req.Options[cidVerisonOptionName].(string)
        # 从请求参数中获取编解码器字符串
        codecStr, _ := req.Options[cidCodecOptionName].(string)
        # 从请求参数中获取多重基字符串
        baseStr, _ := req.Options[cidMultibaseOptionName].(string)

        # 初始化 CID 格式选项
        opts := cidFormatOpts{}

        # 检查格式字符串中是否包含 %
        if strings.IndexByte(fmtStr, '%') == -1 {
            return fmt.Errorf("invalid format string: %q", fmtStr)
        }
        # 将格式字符串赋值给 opts.fmtStr
        opts.fmtStr = fmtStr

        # 如果编解码器字符串不为空
        if codecStr != "" {
            # 创建编解码器对象并设置编解码器字符串
            var codec mc.Code
            err := codec.Set(codecStr)
            if err != nil {
                return err
            }
            # 将编解码器值转换为 uint64 类型并赋值给 opts.newCodec
            opts.newCodec = uint64(codec)
        } # 否则，将其保留为 0（不是有效的 IPLD 编解码器）

        # 根据 CID 版本字符串进行不同的处理
        switch verStr {
        case "":
            # 如果多重基字符串不为空，则设置版本转换函数为 toCidV1
            if baseStr != "" {
                opts.verConv = toCidV1
            }
        case "0":
            # 如果新编解码器不为 0 且不为 dag-pb，则返回错误
            if opts.newCodec != 0 && opts.newCodec != cid.DagProtobuf {
                return fmt.Errorf("cannot convert to CIDv0 with any codec other than dag-pb")
            }
            # 如果多重基字符串不为空且不为 base58btc，则返回错误
            if baseStr != "" && baseStr != "base58btc" {
                return fmt.Errorf("cannot convert to CIDv0 with any multibase other than the implicit base58btc")
            }
            # 设置版本转换函数为 toCidV0
            opts.verConv = toCidV0
        case "1":
            # 设置版本转换函数为 toCidV1
            opts.verConv = toCidV1
        default:
            return fmt.Errorf("invalid cid version: %q", verStr)
        }

        # 如果多重基字符串不为空
        if baseStr != "" {
            # 根据多重基字符串获取编码器对象
            encoder, err := mbase.EncoderByName(baseStr)
            if err != nil {
                return err
            }
            # 获取编码器的编码方式并赋值给 opts.newBase
            opts.newBase = encoder.Encoding()
        } else {
            # 否则，将 opts.newBase 设置为未知编码方式
            opts.newBase = mbase.Encoding(-1)
        }

        # 调用 emitCids 函数并返回结果
        return emitCids(req, resp, opts)
    },
    # 定义 PostRun 属性为 cmds.PostRunMap 类型的映射
    PostRun: cmds.PostRunMap{
        # 将 CLI 属性映射为 streamResult 函数，该函数将结果写入输出流
        cmds.CLI: streamResult(func(v interface{}, out io.Writer) nonFatalError {
            # 将接口类型转换为 CidFormatRes 类型
            r := v.(*CidFormatRes)
            # 如果存在错误信息，则返回非致命错误
            if r.ErrorMsg != "" {
                return nonFatalError(fmt.Sprintf("%s: %s", r.CidStr, r.ErrorMsg))
            }
            # 向输出流写入格式化后的结果
            fmt.Fprintf(out, "%s\n", r.Formatted)
            # 返回空字符串表示无错误
            return ""
        }),
    },
    # 定义 Type 属性为 CidFormatRes 类型的空对象
    Type: CidFormatRes{},
// 定义了一个结构体 CidFormatRes，包含了原始的 Cid 字符串、格式化后的结果和错误信息
type CidFormatRes struct {
    CidStr    string // 原始的 Cid 字符串
    Formatted string // 格式化后的结果
    ErrorMsg  string // 错误信息
}

// 定义了一个命令 base32Cmd，用于将 CIDs 转换为 Base32 CID 版本 1
var base32Cmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Convert CIDs to Base32 CID version 1.", // 帮助文本的概述
        ShortDescription: `
'ipfs cid base32' normalizes passed CIDs to their canonical case-insensitive encoding.
Useful when processing third-party CIDs which could come with arbitrary formats.
`, // 帮助文本的简短描述
    },
    Arguments: []cmds.Argument{
        cmds.StringArg("cid", true, true, "CIDs to convert.").EnableStdin(), // 接受要转换的 CIDs 参数
    },
    Run: func(req *cmds.Request, resp cmds.ResponseEmitter, env cmds.Environment) error {
        opts := cidFormatOpts{ // 设置 CID 格式化的选项
            fmtStr:  "%s",
            newBase: mbase.Encoding(mbase.Base32),
            verConv: toCidV1,
        }
        return emitCids(req, resp, opts) // 调用 emitCids 函数处理请求
    },
    PostRun: cidFmtCmd.PostRun, // 运行后的处理函数
    Type:    cidFmtCmd.Type, // 类型
}

// 定义了一个结构体 cidFormatOpts，包含了格式化字符串、新的编码方式、版本转换函数和新的编解码器
type cidFormatOpts struct {
    fmtStr   string
    newBase  mbase.Encoding
    verConv  func(cid cid.Cid) (cid.Cid, error)
    newCodec uint64
}

// 定义了一个 argumentIterator 结构体，用于迭代处理参数
type argumentIterator struct {
    args []string
    body cmds.StdinArguments
}

// 定义了 argumentIterator 结构体的 next 方法，用于获取下一个参数
func (i *argumentIterator) next() (string, bool) {
    if len(i.args) > 0 {
        arg := i.args[0]
        i.args = i.args[1:]
        return arg, true
    }
    if i.body == nil || !i.body.Scan() {
        return "", false
    }
    return strings.TrimSpace(i.body.Argument()), true
}

// 定义了 argumentIterator 结构体的 err 方法，用于获取错误信息
func (i *argumentIterator) err() error {
    if i.body == nil {
        return nil
    }
    return i.body.Err()
}

// 定义了 emitCids 函数，用于处理请求并发射结果
func emitCids(req *cmds.Request, resp cmds.ResponseEmitter, opts cidFormatOpts) error {
    itr := argumentIterator{req.Arguments, req.BodyArgs()} // 创建 argumentIterator 实例
    var emitErr error
    # 当没有错误发生时循环执行以下操作
    for emitErr == nil:
        # 从迭代器中获取下一个 CID 字符串和是否成功的标志
        cidStr, ok := itr.next()
        # 如果获取失败，则跳出循环
        if !ok:
            break
        # 创建一个 CID 格式的结果对象
        res := &CidFormatRes{CidStr: cidStr}
        # 解码 CID 字符串为 CID 对象
        c, err := cid.Decode(cidStr)
        # 如果解码出错，则记录错误信息并发送结果
        if err != nil:
            res.ErrorMsg = err.Error()
            emitErr = resp.Emit(res)
            continue

        # 如果指定了新的编解码器并且与原 CID 对象的编解码器不同，则创建一个新的 CID 对象
        if opts.newCodec != 0 && opts.newCodec != c.Type():
            c = cid.NewCidV1(opts.newCodec, c.Hash())

        # 如果存在版本转换函数，则进行版本转换
        if opts.verConv != nil:
            c, err = opts.verConv(c)
            # 如果转换出错，则记录错误信息并发送结果
            if err != nil:
                res.ErrorMsg = err.Error()
                emitErr = resp.Emit(res)
                continue

        # 获取新的基数
        base := opts.newBase
        # 如果基数未指定，则根据 CID 版本自动选择基数
        if base == -1:
            if c.Version() == 0:
                base = mbase.Base58BTC
            else:
                base, _ = cid.ExtractEncoding(cidStr)

        # 格式化 CID 对象为字符串
        str, err := cidutil.Format(opts.fmtStr, base, c)
        # 如果格式化字符串出错，则直接返回错误
        if _, ok := err.(cidutil.FormatStringError); ok:
            # 如果格式化字符串出错，则直接返回错误
            return err
        # 如果没有错误，则记录格式化后的字符串
        if err != nil:
            res.ErrorMsg = err.Error()
        else:
            res.Formatted = str
        # 发送结果
        emitErr = resp.Emit(res)
    # 如果在循环中发生错误，则直接返回错误
    if emitErr != nil:
        return emitErr
    # 获取迭代器的错误信息
    err := itr.err()
    # 如果迭代器有错误，则直接返回错误
    if err != nil:
        return err
    # 如果没有错误，则返回空
    return nil
}



func toCidV0(c cid.Cid) (cid.Cid, error) {
    // 如果传入的 CID 类型不是 DagProtobuf，则返回空的 CID 和错误信息
    if c.Type() != cid.DagProtobuf {
        return cid.Cid{}, fmt.Errorf("can't convert non-dag-pb nodes to cidv0")
    }
    // 根据传入的 CID 的哈希值创建一个 CIDv0 对象并返回
    return cid.NewCidV0(c.Hash()), nil
}



func toCidV1(c cid.Cid) (cid.Cid, error) {
    // 根据传入的 CID 的类型和哈希值创建一个 CIDv1 对象并返回
    return cid.NewCidV1(c.Type(), c.Hash()), nil
}



type CodeAndName struct {
    Code int
    Name string
}



const (
    prefixOptionName  = "prefix"
    numericOptionName = "numeric"
)



var basesCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "List available multibase encodings.",
        ShortDescription: `
'ipfs cid bases' relies on https://github.com/multiformats/go-multibase
`,
    },
    Options: []cmds.Option{
        cmds.BoolOption(prefixOptionName, "also include the single letter prefixes in addition to the code"),
        cmds.BoolOption(numericOptionName, "also include numeric codes"),
    },
    Run: func(req *cmds.Request, resp cmds.ResponseEmitter, env cmds.Environment) error {
        var res []CodeAndName
        // 使用 EncodingToStr 方法，将编码和名称添加到结果数组中
        for code, name := range mbase.EncodingToStr {
            res = append(res, CodeAndName{int(code), name})
        }
        // 发送结果数组到响应流中
        return cmds.EmitOnce(resp, res)
    },
    # 定义 Encoders 字段，包含不同类型的编码器
    Encoders: cmds.EncoderMap{
        # 文本类型的编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, val []CodeAndName) error {
            # 获取是否包含前缀的选项
            prefixes, _ := req.Options[prefixOptionName].(bool)
            # 获取是否包含数字的选项
            numeric, _ := req.Options[numericOptionName].(bool)
            # 对 val 进行多重排序
            sort.Sort(multibaseSorter{val})
            # 遍历 val 数组
            for _, v := range val {
                # 获取编码值
                code := v.Code
                # 如果编码值小于 32 或者大于等于 127，则将其替换为空格
                if code < 32 || code >= 127 {
                    # 不显示不可打印的前缀
                    code = ' '
                }
                # 根据不同选项输出不同格式的内容
                switch {
                case prefixes && numeric:
                    fmt.Fprintf(w, "%c %7d  %s\n", code, v.Code, v.Name)
                case prefixes:
                    fmt.Fprintf(w, "%c  %s\n", code, v.Name)
                case numeric:
                    fmt.Fprintf(w, "%7d  %s\n", v.Code, v.Name)
                default:
                    fmt.Fprintf(w, "%s\n", v.Name)
                }
            }
            # 返回空值
            return nil
        }),
    },
    # 定义 Type 字段，包含 CodeAndName 类型的数组
    Type: []CodeAndName{},
// 定义常量，用于存储选项名称
const (
    codecsNumericOptionName   = "numeric"  // 数字选项名称
    codecsSupportedOptionName = "supported"  // 支持选项名称
)

// 定义命令对象
var codecsCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "List available CID multicodecs.",  // 命令简介
        ShortDescription: `
'ipfs cid codecs' relies on https://github.com/multiformats/go-multicodec  // 命令简短描述
`,
    },
    Options: []cmds.Option{  // 定义命令选项
        cmds.BoolOption(codecsNumericOptionName, "n", "also include numeric codes"),  // 布尔选项，包含数字代码
        cmds.BoolOption(codecsSupportedOptionName, "s", "list only codecs supported by go-ipfs commands"),  // 布尔选项，只列出 go-ipfs 命令支持的编解码器
    },
    Run: func(req *cmds.Request, resp cmds.ResponseEmitter, env cmds.Environment) error {  // 定义命令执行函数
        listSupported, _ := req.Options[codecsSupportedOptionName].(bool)  // 获取是否列出支持的编解码器选项的值
        supportedCodecs := make(map[uint64]struct{})  // 创建支持的编解码器映射
        if listSupported {  // 如果需要列出支持的编解码器
            for _, code := range ipldmulticodec.ListEncoders() {  // 遍历编码器列表
                supportedCodecs[code] = struct{}{}  // 将编码器添加到支持的编解码器映射中
            }
            for _, code := range ipldmulticodec.ListDecoders() {  // 遍历解码器列表
                supportedCodecs[code] = struct{}{}  // 将解码器添加到支持的编解码器映射中
            }
            // 添加 libp2p-key
            supportedCodecs[uint64(mc.Libp2pKey)] = struct{}{}  // 将 libp2p-key 添加到支持的编解码器映射中
        }

        var res []CodeAndName  // 定义结果数组
        for _, code := range mc.KnownCodes() {  // 遍历已知编码器列表
            if code.Tag() == "ipld" {  // 如果编码器标签为 "ipld"
                if listSupported {  // 如果需要列出支持的编解码器
                    if _, ok := supportedCodecs[uint64(code)]; !ok {  // 如果编码器不在支持的编解码器映射中
                        continue  // 继续下一次循环
                    }
                }
                res = append(res, CodeAndName{int(code), mc.Code(code).String()})  // 将编码器代码和名称添加到结果数组中
            }
        }
        return cmds.EmitOnce(resp, res)  // 发送结果数组到响应流
    },
    # 创建一个编码器映射，将文本命令映射到特定的编码器函数
    Encoders: cmds.EncoderMap{
        # 将文本命令映射到一个特定的编码器函数，该函数接受请求、写入器和值作为参数
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, val []CodeAndName) error {
            # 从请求选项中获取编解码器的数值选项，如果不存在则默认为布尔值
            numeric, _ := req.Options[codecsNumericOptionName].(bool)
            # 对值进行排序
            sort.Sort(codeAndNameSorter{val})
            # 遍历值数组，根据数值选项格式化输出到写入器
            for _, v := range val {
                if numeric:
                    # 如果数值选项为真，则按照指定格式输出编码和名称
                    fmt.Fprintf(w, "%5d  %s\n", v.Code, v.Name)
                else:
                    # 如果数值选项为假，则按照指定格式输出名称
                    fmt.Fprintf(w, "%s\n", v.Name)
                }
            }
            # 返回空值
            return nil
        }),
    },
    # 初始化一个空的 CodeAndName 数组
    Type: []CodeAndName{},
// 定义 hashesCmd 命令，用于列出可用的多哈希
var hashesCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "List available multihashes.", // 帮助文本的一句简短描述
        ShortDescription: `
'ipfs cid hashes' relies on https://github.com/multiformats/go-multihash
`, // 帮助文本的简短描述
    },
    Options: codecsCmd.Options, // 设置命令的选项
    Run: func(req *cmds.Request, resp cmds.ResponseEmitter, env cmds.Environment) error {
        var res []CodeAndName // 定义存储结果的数组
        // 使用 mhash.Codes，以防将来某个时候给定代码有多个名称
        for code, name := range mhash.Codes {
            if !verifcid.DefaultAllowlist.IsAllowed(code) { // 如果给定代码不在允许列表中，则跳过
                continue
            }
            res = append(res, CodeAndName{int(code), name}) // 将代码和名称添加到结果数组中
        }
        return cmds.EmitOnce(resp, res) // 发送结果数组到响应流
    },
    Encoders: codecsCmd.Encoders, // 设置命令的编码器
    Type:     codecsCmd.Type, // 设置命令的类型
}

// 定义 multibaseSorter 结构体
type multibaseSorter struct {
    data []CodeAndName // 存储数据的数组
}

// 实现排序接口的 Len 方法
func (s multibaseSorter) Len() int      { return len(s.data) }

// 实现排序接口的 Swap 方法
func (s multibaseSorter) Swap(i, j int) { s.data[i], s.data[j] = s.data[j], s.data[i] }

// 实现排序接口的 Less 方法
func (s multibaseSorter) Less(i, j int) bool {
    a := unicode.ToLower(rune(s.data[i].Code)) // 将代码转换为小写
    b := unicode.ToLower(rune(s.data[j].Code)) // 将代码转换为小写
    if a != b {
        return a < b // 如果 a 不等于 b，则返回 a 是否小于 b
    }
    // 小写字母应该在大写字母之前
    return s.data[i].Code > s.data[j].Code // 返回代码是否逆序排列
}

// 定义 codeAndNameSorter 结构体
type codeAndNameSorter struct {
    data []CodeAndName // 存储数据的数组
}

// 实现排序接口的 Len 方法
func (s codeAndNameSorter) Len() int           { return len(s.data) }

// 实现排序接口的 Swap 方法
func (s codeAndNameSorter) Swap(i, j int)      { s.data[i], s.data[j] = s.data[j], s.data[i] }

// 实现排序接口的 Less 方法
func (s codeAndNameSorter) Less(i, j int) bool { return s.data[i].Code < s.data[j].Code } // 返回代码是否正序排列
```