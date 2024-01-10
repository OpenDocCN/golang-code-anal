# `kubo\core\commands\add.go`

```
package commands

import (
    "errors" // 导入 errors 包，用于处理错误
    "fmt" // 导入 fmt 包，用于格式化输出
    "io" // 导入 io 包，用于实现 I/O 操作
    "os" // 导入 os 包，提供操作系统功能
    gopath "path" // 导入 path 包，并重命名为 gopath，用于处理文件路径
    "strings" // 导入 strings 包，用于处理字符串

    "github.com/ipfs/kubo/core/commands/cmdenv" // 导入 cmdenv 包，用于处理命令环境

    "github.com/cheggaaa/pb" // 导入 pb 包，用于显示进度条
    "github.com/ipfs/boxo/files" // 导入 files 包，用于处理文件
    mfs "github.com/ipfs/boxo/mfs" // 导入 mfs 包，用于处理多文件系统
    "github.com/ipfs/boxo/path" // 导入 path 包，用于处理路径
    cmds "github.com/ipfs/go-ipfs-cmds" // 导入 cmds 包，用于处理 IPFS 命令
    ipld "github.com/ipfs/go-ipld-format" // 导入 ipld 包，用于处理 IPLD 格式
    coreiface "github.com/ipfs/kubo/core/coreiface" // 导入 coreiface 包，用于处理核心接口
    "github.com/ipfs/kubo/core/coreiface/options" // 导入 options 包，用于处理选项
    mh "github.com/multiformats/go-multihash" // 导入 mh 包，用于处理多哈希
)

// ErrDepthLimitExceeded indicates that the max depth has been exceeded.
var ErrDepthLimitExceeded = fmt.Errorf("depth limit exceeded") // 定义 ErrDepthLimitExceeded 变量，表示超出最大深度

type AddEvent struct {
    Name  string // 定义 AddEvent 结构体的 Name 字段，表示名称
    Hash  string `json:",omitempty"` // 定义 AddEvent 结构体的 Hash 字段，表示哈希值
    Bytes int64  `json:",omitempty"` // 定义 AddEvent 结构体的 Bytes 字段，表示字节数
    Size  string `json:",omitempty"` // 定义 AddEvent 结构体的 Size 字段，表示大小
}

const (
    quietOptionName       = "quiet" // 定义 quietOptionName 常量，表示安静选项
    quieterOptionName     = "quieter" // 定义 quieterOptionName 常量，表示更安静选项
    silentOptionName      = "silent" // 定义 silentOptionName 常量，表示静音选项
    progressOptionName    = "progress" // 定义 progressOptionName 常量，表示进度选项
    trickleOptionName     = "trickle" // 定义 trickleOptionName 常量，表示涓涓细流选项
    wrapOptionName        = "wrap-with-directory" // 定义 wrapOptionName 常量，表示用目录包装选项
    onlyHashOptionName    = "only-hash" // 定义 onlyHashOptionName 常量，表示仅哈希选项
    chunkerOptionName     = "chunker" // 定义 chunkerOptionName 常量，表示分块选项
    pinOptionName         = "pin" // 定义 pinOptionName 常量，表示固定选项
    rawLeavesOptionName   = "raw-leaves" // 定义 rawLeavesOptionName 常量，表示原始叶子选项
    noCopyOptionName      = "nocopy" // 定义 noCopyOptionName 常量，表示无复制选项
    fstoreCacheOptionName = "fscache" // 定义 fstoreCacheOptionName 常量，表示文件存储缓存选项
    cidVersionOptionName  = "cid-version" // 定义 cidVersionOptionName 常量，表示 CID 版本选项
    hashOptionName        = "hash" // 定义 hashOptionName 常量，表示哈希选项
    inlineOptionName      = "inline" // 定义 inlineOptionName 常量，表示内联选项
    inlineLimitOptionName = "inline-limit" // 定义 inlineLimitOptionName 常量，表示内联限制选项
    toFilesOptionName     = "to-files" // 定义 toFilesOptionName 常量，表示到文件选项
)

const adderOutChanSize = 8 // 定义 adderOutChanSize 常量，表示添加器输出通道大小

var AddCmd = &cmds.Command{ // 定义 AddCmd 变量，表示 IPFS 添加命令
    Helptext: cmds.HelpText{ // 设置帮助文本
        Tagline: "Add a file or directory to IPFS.", // 设置标语
        ShortDescription: ` // 设置简短描述
Adds the content of <path> to IPFS. Use -r to add directories (recursively).
`,
        LongDescription: ` // 设置长描述
Adds the content of <path> to IPFS. Use -r to add directories.
Note that directories are added recursively, to form the IPFS
MerkleDAG.

If the daemon is not running, it will just add locally.
# 如果后台守护进程稍后启动，当重新提供程序运行时，它将在几秒钟后进行广告。

# 使用 wrap 选项 '-w'，将文件（如果使用递归选项，则为多个文件）包装在一个目录中。此目录仅包含已添加的文件，并且意味着文件保留其文件名。例如：

# > ipfs add example.jpg
# 添加了 QmbFMke1KXqnYyBBWxB74N4c5SBnJMVAiMNRcGu6x1AwQH example.jpg
# > ipfs add example.jpg -w
# 添加了 QmbFMke1KXqnYyBBWxB74N4c5SBnJMVAiMNRcGu6x1AwQH example.jpg
# 添加了 QmaG4FuMqEBnQNn3C8XJ5bpW8kLs7zq2ZXgHptJHbKDDVx

# 现在可以在网关中引用已添加的文件，如下所示：

# /ipfs/QmaG4FuMqEBnQNn3C8XJ5bpW8kLs7zq2ZXgHptJHbKDDVx/example.jpg

# 使用 'ipfs add' 导入的文件受到 GC 的保护（隐式 '--pin=true'），但是您需要记住返回的 CID 以便以后获取数据。

# 通过传递 '--to-files'，在 Files API（MFS）中创建一个引用，使将来更容易找到它：

# > ipfs files mkdir -p /myfs/dir
# > ipfs add example.jpg --to-files /myfs/dir/
# > ipfs files ls /myfs/dir/
# example.jpg

# 查看 'ipfs files --help' 以了解更多关于使用 MFS 来跟踪已添加的文件和目录的信息。

# chunker 选项 '-s' 指定了分块策略，决定如何将文件分成块。具有相同内容的块可以进行去重。不同的分块策略将为相同文件产生不同的哈希值。默认是固定块大小为 256 * 1024 字节，'size-262144'。或者，您可以使用 Buzhash 或 Rabin 指纹分块进行内容定义的分块，通过指定 buzhash 或 rabin-[min]-[avg]-[max]（其中 min/avg/max 指的是所需的块大小，以字节为单位），例如 'rabin-262144-524288-1048576'。

# 以下示例使用非常小的字节大小来演示不同分块器在小文件上的属性。您可能会
# 设置大多数文件的块大小为1024倍
# 使用指定的块大小将ipfs-logo.svg添加到IPFS
# 返回ipfs-logo.svg的CID
> ipfs add --chunker=size-2048 ipfs-logo.svg
added QmafrLBfzRLV4XSH1XcaMMeaXEUhDJjmtDfsYU95TrWG87 ipfs-logo.svg
# 使用Rabin算法和指定的参数将ipfs-logo.svg添加到IPFS
# 返回ipfs-logo.svg的CID
> ipfs add --chunker=rabin-512-1024-2048 ipfs-logo.svg
added Qmf1hDN65tR55Ubh2RN1FPxr69xq3giVBz1KApsresY8Gn ipfs-logo.svg

# 可以通过以下命令查看已创建的块
# 返回QmafrLBfzRLV4XSH1XcaMMeaXEUhDJjmtDfsYU95TrWG87的链接
> ipfs object links QmafrLBfzRLV4XSH1XcaMMeaXEUhDJjmtDfsYU95TrWG87
QmY6yj1GsermExDXoosVE3aSPxdMNYr6aKuw3nA8LoWPRS 2059
Qmf7ZQeSxq2fJVJbCmgTrLLVN9tDR9Wy5k75DxQKuz5Gyt 1195
# 返回Qmf1hDN65tR55Ubh2RN1FPxr69xq3giVBz1KApsresY8Gn的链接
> ipfs object links Qmf1hDN65tR55Ubh2RN1FPxr69xq3giVBz1KApsresY8Gn
QmY6yj1GsermExDXoosVE3aSPxdMNYr6aKuw3nA8LoWPRS 2059
QmerURi9k4XzKCaaPbsK6BL5pMEjF7PGphjDvkkjDtsVf3 868
QmQB28iwSriSUSMqG2nXDTLtdPHgWb4rebBrU7Q1j4vxPv 338

# 关于哈希（CID）的确定性和'ipfs add'命令的说明
# 几乎所有此命令提供的标志都会改变最终的CID
# 未来可能会添加新的标志
# 不能保证'ipfs add'的隐式默认值在未来的Kubo版本中保持不变
# 或者其他IPFS软件使用与Kubo相同的导入参数
# 如果需要使用非IPFS介质备份或传输内容寻址数据，可以使用CAR文件来保留CID
# 有关更多信息，请参阅'dag export'和'dag import'
    # 在运行之前执行的函数，用于处理请求和环境
    PreRun: func(req *cmds.Request, env cmds.Environment) error {
        # 从请求选项中获取 quiet 标志，并设置默认值为 false
        quiet, _ := req.Options[quietOptionName].(bool)
        # 从请求选项中获取 quieter 标志，并设置默认值为 false
        quieter, _ := req.Options[quieterOptionName].(bool)
        # 如果 quiet 或 quieter 任一为 true，则将 quiet 设置为 true
        quiet = quiet || quieter

        # 从请求选项中获取 silent 标志，并设置默认值为 false
        silent, _ := req.Options[silentOptionName].(bool)

        # 如果 quiet 或 silent 任一为 true，则直接返回
        if quiet || silent {
            return nil
        }

        # 如果请求选项中没有设置 progress 标志，则将其设置为 true
        _, found := req.Options[progressOptionName].(bool)
        if !found {
            req.Options[progressOptionName] = true
        }

        return nil
    },
    },
    },
    Type: AddEvent{},
# 闭合前面的函数定义
```