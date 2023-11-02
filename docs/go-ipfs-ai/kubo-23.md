# go-ipfs 源码解析 23

# `/opt/kubo/core/commands/pin/remotepin_test.go`

This appears to be a testing function that evaluates a series of input URLs against a known good output. The input URLs are provided as strings, and the expected output for each URL is also provided as a string. If the output for a given input URL is not what is expected, the test will report an error.

The expected output for a valid input URL is prefixed with a URL schema, such as "https://<service-endpoint>". The exact format of the URL schema is not specified, but it should be a valid HTTP URL.

Note that some of the input URLs are missing the protocol and some of the ports, this is because the service endpoint they are requesting may not have those specified.


```
package pin

import (
	"testing"
)

func TestNormalizeEndpoint(t *testing.T) {
	cases := []struct {
		in  string
		err string
		out string
	}{
		{
			in:  "https://1.example.com",
			err: "",
			out: "https://1.example.com",
		},
		{
			in:  "https://2.example.com/",
			err: "",
			out: "https://2.example.com",
		},
		{
			in:  "https://3.example.com/pins/",
			err: "service endpoint should be provided without the /pins suffix",
			out: "",
		},
		{
			in:  "https://4.example.com/pins",
			err: "service endpoint should be provided without the /pins suffix",
			out: "",
		},
		{
			in:  "https://5.example.com/./some//nonsense/../path/../path/",
			err: "",
			out: "https://5.example.com/some/path",
		},
		{
			in:  "https://6.example.com/endpoint/?query=val",
			err: "service endpoint should be provided without any query parameters",
			out: "",
		},
		{
			in:  "http://192.168.0.5:45000/",
			err: "",
			out: "http://192.168.0.5:45000",
		},
		{
			in:  "foo://4.example.com/pins",
			err: "service endpoint must be a valid HTTP URL",
			out: "",
		},
	}

	for _, tc := range cases {
		out, err := normalizeEndpoint(tc.in)
		if err != nil && tc.err != err.Error() {
			t.Errorf("unexpected error for %q: expected %q; got %q", tc.in, tc.err, err)
			continue
		}
		if out != tc.out {
			t.Errorf("unexpected endpoint for %q: expected %q; got %q", tc.in, tc.out, out)
			continue
		}
	}
}

```

# `/opt/kubo/core/commands/unixfs/ls.go`

这段代码定义了一个名为“unixfs”的包。它导入了多个其他包，包括“fmt”用于格式化输出、“io”用于输入/输出操作、“sort”用于排序、“text/tabwriter”用于处理表格数据等。

此外，它还导入了两个自定义的命令类——“cmdenv”和“cmdutils”。“cmdenv”用于设置Linux的命令环境，“cmdutils”用于管理常见的命令行工具。

最后，它导入了两个与“ipfs”相关的包——“merkledag”和“unixfs”。这里我们猜测“ipfs”可能是指一个名为“iparsers”的包，它提供了一些用于解析IPLD的工具。但是，我们无法确定这一点，因为我们缺少关于“ipfs”和“unixfs”的更多上下文信息。


```
package unixfs

import (
	"fmt"
	"io"
	"sort"
	"text/tabwriter"

	cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"
	"github.com/ipfs/kubo/core/commands/cmdutils"

	merkledag "github.com/ipfs/boxo/ipld/merkledag"
	unixfs "github.com/ipfs/boxo/ipld/unixfs"
	cmds "github.com/ipfs/go-ipfs-cmds"
)

```

这段代码定义了三个数据结构类型：LsLink、LsObject 和 LsOutput。

LsLink 结构体包含三个字段：Name、Hash 和 Size，分别表示链接名称、数据哈希值和数据大小。

LsObject 结构体包含三个字段：Hash、Size 和 Type，分别表示对象哈希值、数据大小和对象类型。此外，该结构体还包含一个 LsLink 类型的成员变量，表示该对象包含的链接。

LsOutput 结构体包含三个字段：Arguments、Objects 和 Links，分别表示函数的参数、函数的对象和函数的链接。

这段代码的目的是定义一个链接类型（LsLink）和两个对象类型（LsObject 和 LsOutput）。LsObject 和 LsOutput 都包含一个名为 "Links" 的字段，该字段包含一个 LsLink 类型的成员变量。


```
type LsLink struct {
	Name, Hash string
	Size       uint64
	Type       string
}

type LsObject struct {
	Hash  string
	Size  uint64
	Type  string
	Links []LsLink
}

type LsOutput struct {
	Arguments map[string]string
	Objects   map[string]*LsObject
}

```

这段代码是一个Makefile脚本，它定义了一个名为"LsCmd"的变量，该变量引用了一个名为"cmds.Command"的常量。

通过将以下内容添加到该常量的定义中，定义了该命令的详细信息和帮助信息：

1. 将该命令的名称设置为"List"，类型设置为"Unix filesystem objects"，并标记为"Deprecated"，这意味着该命令将在未来的版本中被删除，因为它的功能已经被弃用。

2. 设置命令的帮助信息的标题为"List directory contents for Unix filesystem objects"，其中"List directory contents"是该命令的主要功能，"for Unix filesystem objects"指定了该命令适用的操作系统和文件系统类型。

3. 通过将"/生成JSON output"设置为true，使该命令在输出时生成JSON格式的输出。

4. 通过将"JSON output tagline"设置为"'The JSON output contains size information.'"，提供了关于该命令输出的JSON格式的简要描述。

5. 通过将"JSON output short description"设置为"'Displays the contents of an IPFS or IPNS object(s) at the given path. The JSON output contains size information. For files, the child size is the total size of the file contents. For directories, the child size is the IPFS link size.'"，提供了关于该命令输出的JSON格式的详细描述。

6. 通过将"Long description"设置为"'This functionality is deprecated, and will be removed in future versions as it duplicates the functionality of 'ipfs ls'. If possible, please use 'ipfs ls' instead.'"，提供了该命令的功能说明，即该命令已经过时，将在未来的版本中被删除，因为它的功能已经被其他命令或库重复实现了。

7. 通过在函数体中执行以下操作，实现了命令的功能：

a. 使用Kubernetes API对象的"cmds.Command"构造命令对象。

b. 设置命令的状态为"Deprecated"，以便在将来删除该命令。

c. 设置命令的帮助信息的标题为"List directory contents for Unix filesystem objects"，并设置为"The JSON output contains size information."，以描述该命令的功能和用途。

d. 设置命令的短描述为"Displays the contents of an IPFS or IPNS object(s) at the given path. The JSON output contains size information. For files, the child size is the total size of the file contents. For directories, the child size is the IPFS link size."，以描述该命令的功能和用途。

e. 设置命令的长描述为"This functionality is deprecated, and will be removed in future versions as it duplicates the functionality of 'ipfs ls'. If possible, please use 'ipfs ls' instead."，以通知用户该命令的功能已经过时，并建议使用其他命令或库来实现相同的功能。


```
var LsCmd = &cmds.Command{
	Status: cmds.Deprecated, // https://github.com/ipfs/kubo/pull/7755
	Helptext: cmds.HelpText{
		Tagline: "List directory contents for Unix filesystem objects. Deprecated: Use 'ipfs ls' and 'ipfs files ls' instead.",
		ShortDescription: `
Displays the contents of an IPFS or IPNS object(s) at the given path.

The JSON output contains size information. For files, the child size
is the total size of the file contents. For directories, the child
size is the IPFS link size.

This functionality is deprecated, and will be removed in future versions as it duplicates the functionality of 'ipfs ls'.
If possible, please use 'ipfs ls' instead.
`,
		LongDescription: `
```

这段代码是一个命令行工具，它使用乔姆·定理（Je皂磨）实现了基于IPFS（InterPlanetary File System）或IPNS（InterPlanetary Name System）对象的文件夹内容显示。

具体来说，这段代码会递归地列出指定路径下的所有IPFS或IPNS对象。对于每个文件，它的子文件大小信息被包含在JSON输出中。对于文件，子文件大小是文件内容的总大小；对于目录，子文件大小是IPFS链接大小。

如果路径是一个无前缀的指定了IPFS或IPNS对象的路径，那么它被认为是/ipfs指定的路径，而不是/ipns。

例如，运行这段代码会输出像这样的结果：


$ ipfs file ls QmW2WQi7j6c7UgJTarActp7tDNikE4B2qXtFCfLPdsgaTQ
cat.jpg
/ipfs/QmW2WQi7j6c7UgJTarActp7tDNikE4B2qXtFCfLPdsgaTQ


这段代码会显示给定的路径下的所有文件夹内容，包括文件和子目录。


```
Displays the contents of an IPFS or IPNS object(s) at the given path.

The JSON output contains size information. For files, the child size
is the total size of the file contents. For directories, the child
size is the IPFS link size.

The path can be a prefixless ref; in this case, we assume it to be an
/ipfs ref and not /ipns.

Example:

    > ipfs file ls QmW2WQi7j6c7UgJTarActp7tDNikE4B2qXtFCfLPdsgaTQ
    cat.jpg
    > ipfs file ls /ipfs/QmW2WQi7j6c7UgJTarActp7tDNikE4B2qXtFCfLPdsgaTQ
    cat.jpg

```

This appears to be a Go program that performs a tree traversal. It takes a root directory and a list of filers to watch for updates.

The program first checks that the input is valid and creates a list of directories and a list of non-directory files. It then sorts these lists based on their contents.

Next, the program watches for updates to the files listed in the `filers` list. When a file is updated, the program checks that it is a new file and not a link. If the file is a new file, the program creates a new directory for it and adds it to the list of directories. If the file is a link, the program updates the `filers` list and the `types` field in the ` out.Args` map to reflect the updated file.

Finally, the program watches for updates to the files and prints any relevant information.


```
This functionality is deprecated, and will be removed in future versions as it duplicates the functionality of 'ipfs ls'.
If possible, please use 'ipfs ls' instead.
`,
	},

	Arguments: []cmds.Argument{
		cmds.StringArg("ipfs-path", true, true, "The path to the IPFS object(s) to list links from.").EnableStdin(),
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		nd, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}

		api, err := cmdenv.GetApi(env, req)
		if err != nil {
			return err
		}

		if err := req.ParseBodyArgs(); err != nil {
			return err
		}

		paths := req.Arguments

		output := LsOutput{
			Arguments: map[string]string{},
			Objects:   map[string]*LsObject{},
		}

		for _, p := range paths {
			ctx := req.Context

			pth, err := cmdutils.PathOrCidPath(p)
			if err != nil {
				return err
			}

			merkleNode, err := api.ResolveNode(ctx, pth)
			if err != nil {
				return err
			}

			c := merkleNode.Cid()

			hash := c.String()
			output.Arguments[p] = hash

			if _, ok := output.Objects[hash]; ok {
				// duplicate argument for an already-listed node
				continue
			}

			ndpb, ok := merkleNode.(*merkledag.ProtoNode)
			if !ok {
				return merkledag.ErrNotProtobuf
			}

			unixFSNode, err := unixfs.FSNodeFromBytes(ndpb.Data())
			if err != nil {
				return err
			}

			t := unixFSNode.Type()

			output.Objects[hash] = &LsObject{
				Hash: c.String(),
				Type: t.String(),
				Size: unixFSNode.FileSize(),
			}

			switch t {
			case unixfs.TFile:
				break
			case unixfs.THAMTShard:
				// We need a streaming ls API for this.
				return fmt.Errorf("cannot list large directories yet")
			case unixfs.TDirectory:
				links := make([]LsLink, len(merkleNode.Links()))
				output.Objects[hash].Links = links
				for i, link := range merkleNode.Links() {
					linkNode, err := link.GetNode(ctx, nd.DAG)
					if err != nil {
						return err
					}
					lnpb, ok := linkNode.(*merkledag.ProtoNode)
					if !ok {
						return merkledag.ErrNotProtobuf
					}

					d, err := unixfs.FSNodeFromBytes(lnpb.Data())
					if err != nil {
						return err
					}
					t := d.Type()
					lsLink := LsLink{
						Name: link.Name,
						Hash: link.Cid.String(),
						Type: t.String(),
					}
					if t == unixfs.TFile {
						lsLink.Size = d.FileSize()
					} else {
						lsLink.Size = link.Size
					}
					links[i] = lsLink
				}
			case unixfs.TSymlink:
				return fmt.Errorf("cannot list symlinks yet")
			default:
				return fmt.Errorf("unrecognized type: %s", t)
			}
		}

		return cmds.EmitOnce(res, &output)
	},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *LsOutput) error {
			tw := tabwriter.NewWriter(w, 1, 2, 1, ' ', 0)

			nonDirectories := []string{}
			directories := []string{}
			for argument, hash := range out.Arguments {
				object, ok := out.Objects[hash]
				if !ok {
					return fmt.Errorf("unresolved hash: %s", hash)
				}

				if object.Type == "Directory" {
					directories = append(directories, argument)
				} else {
					nonDirectories = append(nonDirectories, argument)
				}
			}
			sort.Strings(nonDirectories)
			sort.Strings(directories)

			for _, argument := range nonDirectories {
				fmt.Fprintf(tw, "%s\n", argument)
			}

			seen := map[string]bool{}
			for i, argument := range directories {
				hash := out.Arguments[argument]
				if _, ok := seen[hash]; ok {
					continue
				}
				seen[hash] = true

				object := out.Objects[hash]
				if i > 0 || len(nonDirectories) > 0 {
					fmt.Fprintln(tw)
				}
				if len(out.Arguments) > 1 {
					for _, arg := range directories[i:] {
						if out.Arguments[arg] == hash {
							fmt.Fprintf(tw, "%s:\n", cmdenv.EscNonPrint(arg))
						}
					}
				}
				for _, link := range object.Links {
					fmt.Fprintf(tw, "%s\n", cmdenv.EscNonPrint(link.Name))
				}
			}
			tw.Flush()

			return nil
		}),
	},
	Type: LsOutput{},
}

```

# `/opt/kubo/core/commands/unixfs/unixfs.go`

这段代码是一个 UnixFS 命令行工具的实现，该工具提供了与 IPFS（InterPlanetary File System，IPFS）对象中 Unix 文件系统进行交互的功能。具体来说，这段代码定义了一个名为 "UnixFSCmd" 的命令对象，该对象表示对 UnixFS 工具的支持。

在代码中，首先导入了 ipfs-cmds 包，这是 ipfs 命令行工具的一个依赖项。然后，定义了一个名为 "UnixFSCmd" 的命令对象，该对象包含以下属性和方法：

1. 命令的状态信息：使用 cmds.Deprecated 状态指示符，表示此命令已过时，不再支持使用。
2. 命令的帮助信息：使用 cmds.HelpText 格式定义命令的帮助信息，其中包含命令的标签、短描述和详细描述。
3. 子命令列表：通过 map 类型将 UnixFS 命令列表映射到一个命令对象，这样就可以通过 map 修改子命令的名称。
4. UnixFS 命令列表：通过 UnixFSCmd.Subcommands 字段获取子命令列表，这里定义了一个名为 "ls" 的命令对象，它是 UnixFS 命令列表的一个代表。

整个 UnixFS 命令行工具的作用是帮助用户在 Unix 文件系统中执行各种操作，通过使用 ipfs-cmds 包提供的 UnixFS 命令，可以方便地与 IPFS 对象中 Unix 文件系统进行交互。


```
package unixfs

import (
	cmds "github.com/ipfs/go-ipfs-cmds"
)

var UnixFSCmd = &cmds.Command{
	Status: cmds.Deprecated, // https://github.com/ipfs/kubo/pull/7755
	Helptext: cmds.HelpText{
		Tagline: "Interact with IPFS objects representing Unix filesystems.",
		ShortDescription: `
Old interface to file systems represented by UnixFS.
Superseded by modern alternatives: 'ipfs ls' and 'ipfs files'
`,
	},

	Subcommands: map[string]*cmds.Command{
		"ls": LsCmd,
	},
}

```

# `/opt/kubo/core/coreapi/block.go`

该代码的作用是定义了一个名为 "coreapi" 的包，该包包含了一些与 PIN 相关的工具和类。

具体来说，该包通过导入了一些 PIN 相关的库和常量，包括 "github.com/ipfs/boxo/coreiface" 和 "github.com/ipfs/boxo/coreiface/options"，这些库与 PIN 的子系统相关。此外，该包还通过导入 "github.com/ipfs/go-block-format" 和 "github.com/ipfs/go-cid"，这两个库与 PIN 的数据结构相关。

此外，该包还定义了一些与 PIN 相关的函数和类，例如 "util.NewPinCloser"，该函数通过关闭 PIN 客户端连接并返回一个计时器来关闭连接；"util.NewPinTracer"，该函数通过设置 PIN 跟踪器来跟踪 PIN 操作；"pin.Pin"，该类用于与 PIN 服务器建立连接；"blocks.BlockStore"，该类用于管理 PIN 块存储器中的块。

综上所述，该代码定义了一个 PIN 工具包，用于管理 PIN 相关操作。


```
package coreapi

import (
	"bytes"
	"context"
	"errors"
	"io"

	coreiface "github.com/ipfs/boxo/coreiface"
	caopts "github.com/ipfs/boxo/coreiface/options"
	"github.com/ipfs/boxo/path"
	pin "github.com/ipfs/boxo/pinning/pinner"
	blocks "github.com/ipfs/go-block-format"
	cid "github.com/ipfs/go-cid"
	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/trace"

	util "github.com/ipfs/kubo/blocks/blockstoreutil"
	"github.com/ipfs/kubo/tracing"
)

```

该代码定义了一个名为BlockAPI的接口，该接口实现了一个名为Put的函数，用于在分布式系统中创建新块。

具体来说，该函数接收一个名为ctx的上下文，以及一个名为src的输入 io.Reader 和一个或多个名为opts的caopts.BlockPutOption选项，然后执行以下操作：

1. 获取BlockAPI实例的Put选项，包括设置Cid前缀、Pin标志、以及Pin模式等。
2. 创建一个名为b的块对象，并执行一些设置选项的检查，如设置Pin标志和Pin模式等。
3. 将数据写入src，并计算bcid。
4. 创建一个新的块对象，并添加到api的blockstore中。
5. 如果设置Pin标志，则根据Pin模式旋转块，并使用Pin模式提交块。
6. 返回新创建的块对象的BlockStat，同时输出ln块对象的路径和大小。

该函数的实现需要依赖一些其他的外部工具和数据结构，如caopts.BlockPutOption、blocks.Block、tracing.Span、io.Reader、int类型等。


```
type BlockAPI CoreAPI

type BlockStat struct {
	path path.ImmutablePath
	size int
}

func (api *BlockAPI) Put(ctx context.Context, src io.Reader, opts ...caopts.BlockPutOption) (coreiface.BlockStat, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.BlockAPI", "Put")
	defer span.End()

	settings, err := caopts.BlockPutOptions(opts...)
	if err != nil {
		return nil, err
	}

	data, err := io.ReadAll(src)
	if err != nil {
		return nil, err
	}

	bcid, err := settings.CidPrefix.Sum(data)
	if err != nil {
		return nil, err
	}

	b, err := blocks.NewBlockWithCid(data, bcid)
	if err != nil {
		return nil, err
	}

	if settings.Pin {
		defer api.blockstore.PinLock(ctx).Unlock(ctx)
	}

	err = api.blocks.AddBlock(ctx, b)
	if err != nil {
		return nil, err
	}

	if settings.Pin {
		if err = api.pinning.PinWithMode(ctx, b.Cid(), pin.Recursive); err != nil {
			return nil, err
		}
		if err := api.pinning.Flush(ctx); err != nil {
			return nil, err
		}
	}

	return &BlockStat{path: path.FromCid(b.Cid()), size: len(data)}, nil
}

```

此函数的作用是获取给定路径的块（BlockAPI中的block）。它使用Tracing（追踪）和BlockAPI作为对外接口，通过调用API的`core.BlockAPI.ResolvePath`函数并获取块的根控制器ID（RootCID），然后使用`blocks.GetBlock`函数获取块。如果块获取成功，函数将返回一个内存中的字节切片（io.Reader）和零错误（error）。如果块获取失败，函数将返回一个空值（nil）或错误。


```
func (api *BlockAPI) Get(ctx context.Context, p path.Path) (io.Reader, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.BlockAPI", "Get", trace.WithAttributes(attribute.String("path", p.String())))
	defer span.End()
	rp, _, err := api.core().ResolvePath(ctx, p)
	if err != nil {
		return nil, err
	}

	b, err := api.blocks.GetBlock(ctx, rp.RootCid())
	if err != nil {
		return nil, err
	}

	return bytes.NewReader(b.RawData()), nil
}

```

此函数名为“BlockAPI.Rm”，属于BlockAPI的函数。

它的参数为：

- api：代表BlockAPI的实例
- p：目标路径
- opts ...caopts.BlockRmOption)：选项参数

函数实现：

1. 从BlockAPI的实例中调用核心API的“ResolvePath”方法，将path和opts作为参数。
2. 如果调用“ResolvePath”过程中出现错误，将返回此错误。
3. 设置Block Rm选项（包括力


```
func (api *BlockAPI) Rm(ctx context.Context, p path.Path, opts ...caopts.BlockRmOption) error {
	ctx, span := tracing.Span(ctx, "CoreAPI.BlockAPI", "Rm", trace.WithAttributes(attribute.String("path", p.String())))
	defer span.End()

	rp, _, err := api.core().ResolvePath(ctx, p)
	if err != nil {
		return err
	}

	settings, err := caopts.BlockRmOptions(opts...)
	if err != nil {
		return err
	}
	cids := []cid.Cid{rp.RootCid()}
	o := util.RmBlocksOpts{Force: settings.Force}

	out, err := util.RmBlocks(ctx, api.blockstore, api.pinning, cids, o)
	if err != nil {
		return err
	}

	select {
	case res, ok := <-out:
		if !ok {
			return nil
		}

		remBlock, ok := res.(*util.RemovedBlock)
		if !ok {
			return errors.New("got unexpected output from util.RmBlocks")
		}

		if remBlock.Error != nil {
			return remBlock.Error
		}
		return nil
	case <-ctx.Done():
		return ctx.Err()
	}
}

```

此函数名为`BlockAPI.Stat`，定义在`BlockAPI`类型中。

它的作用是接收一个`BlockAPI`对象和一个路径参数`p`，返回一个表示块的`BlockStat`对象，或者一个非` nil`的错误。

函数内部使用了`tracing`包来跟踪API调用的情况，包括记录请求的路径和参数等信息，以及记录API调用的超时情况。

函数首先获取路径参数`p`的父路径，然后使用`ResolvePath`方法尝试从API中解析出路径`p`，如果解析失败或者返回的错误不影响继续执行，则返回一个` nil`。

接着，函数使用`blocks`字段中的方法尝试从API中获取块，并返回块的`Block`对象的引用。如果获取块的过程中出现错误，则返回一个` nil`。

最后，函数创建一个表示块的`BlockStat`对象，将路径、块的大小设置为`path.FromCid`和`len(b.RawData())`，然后将其返回。


```
func (api *BlockAPI) Stat(ctx context.Context, p path.Path) (coreiface.BlockStat, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.BlockAPI", "Stat", trace.WithAttributes(attribute.String("path", p.String())))
	defer span.End()

	rp, _, err := api.core().ResolvePath(ctx, p)
	if err != nil {
		return nil, err
	}

	b, err := api.blocks.GetBlock(ctx, rp.RootCid())
	if err != nil {
		return nil, err
	}

	return &BlockStat{
		path: path.FromCid(b.Cid()),
		size: len(b.RawData()),
	}, nil
}

```

这段代码定义了三个函数，分别作用于一个名为"BlockStat"的类型和一个名为"BlockAPI"的接口上。

1. "func (bs *BlockStat) Size() int"函数接收一个名为"bs"的指针类型，代表一个BlockStat对象，函数返回bs对象的size值。

2. "func (bs *BlockStat) Path() path.ImmutablePath"函数接收一个名为"bs"的指针类型，代表一个BlockStat对象，函数返回bs对象的path值，该值是一个不可变的path.ImmutablePath类型。

3. "func (api *BlockAPI) core() coreiface.CoreAPI"函数接收一个名为"api"的指针类型，代表一个BlockAPI对象，函数返回一个指向CoreAPI类型"coreiface.CoreAPI"的指针，即api对象的CoreAPI接口。


```
func (bs *BlockStat) Size() int {
	return bs.size
}

func (bs *BlockStat) Path() path.ImmutablePath {
	return bs.path
}

func (api *BlockAPI) core() coreiface.CoreAPI {
	return (*CoreAPI)(api)
}

```

# `/opt/kubo/core/coreapi/coreapi.go`

This is a Go package that implements the Boxo blockchain storage layer. It includes the following components:

* The `store` store provides a BoxoBlockstore implementation.
* The `coreiface` interface provides a BoxoCoreIface implementation.
* The `options` struct provides options for the BoxoCoreIface.
* The `exchange` struct provides an BoxoExchange implementation.
* The `offlinexch` struct provides an BoxoOfflineExchange implementation.
* The `fetcher` struct provides an BoxoFetcher implementation.
* The `dag` struct provides a BoxoDAG implementation.
* The `pathresolver` struct provides a BoxoPathResolver implementation.
* The `pin` struct provides a BoxoPinning implementation.
* The `provider` struct provides a BoxoProvider implementation.
* The `offlineroute` struct provides an BoxoOfflineRoute implementation.
* The `ipld` struct provides an BoxoIPLDImplementation.
* The `pubsub` struct provides a PubSub implementation.
* The `record` struct provides a BoxoRecord implementation.
* The `ci` struct provides a BoxoCryptoImplementation.
* The `p2phost` struct provides a BoxoP2PHost implementation.
* The `peer` struct provides a BoxoPeer implementation.
* The `pstore` struct provides a BoxoPeerStore implementation.
* The `routing` struct provides a BoxoRouting implementation.
* The `madns` struct provides a MadNSDNS implementation.
* The `namesys` struct provides a namesys implementation.
* The `kubo` struct provides a Kubo implementation.
* The `node` struct provides a Node implementation.
* The `repo` struct provides a Repo implementation.


```
/*
**NOTE: this package is experimental.**

Package coreapi provides direct access to the core commands in IPFS. If you are
embedding IPFS directly in your Go program, this package is the public
interface you should use to read and write files or otherwise control IPFS.

If you are running IPFS as a separate process, you should use `client/rpc` to
work with it via HTTP.
*/
package coreapi

import (
	"context"
	"errors"
	"fmt"

	bserv "github.com/ipfs/boxo/blockservice"
	blockstore "github.com/ipfs/boxo/blockstore"
	coreiface "github.com/ipfs/boxo/coreiface"
	"github.com/ipfs/boxo/coreiface/options"
	exchange "github.com/ipfs/boxo/exchange"
	offlinexch "github.com/ipfs/boxo/exchange/offline"
	"github.com/ipfs/boxo/fetcher"
	dag "github.com/ipfs/boxo/ipld/merkledag"
	pathresolver "github.com/ipfs/boxo/path/resolver"
	pin "github.com/ipfs/boxo/pinning/pinner"
	provider "github.com/ipfs/boxo/provider"
	offlineroute "github.com/ipfs/boxo/routing/offline"
	ipld "github.com/ipfs/go-ipld-format"
	pubsub "github.com/libp2p/go-libp2p-pubsub"
	record "github.com/libp2p/go-libp2p-record"
	ci "github.com/libp2p/go-libp2p/core/crypto"
	p2phost "github.com/libp2p/go-libp2p/core/host"
	peer "github.com/libp2p/go-libp2p/core/peer"
	pstore "github.com/libp2p/go-libp2p/core/peerstore"
	routing "github.com/libp2p/go-libp2p/core/routing"
	madns "github.com/multiformats/go-multiaddr-dns"

	"github.com/ipfs/boxo/namesys"
	"github.com/ipfs/kubo/core"
	"github.com/ipfs/kubo/core/node"
	"github.com/ipfs/kubo/repo"
)

```

该代码定义了一个名为 CoreAPI 的结构体，它表示一个 Hyperledger Fabric 节点的核心部分。该结构体包含以下字段：

- nctx：一个上下文上下文，该上下文用于与外部的网络交互。
- identity：一个身份标识符，该标识符用于标识节点。
- privateKey：一个私钥，用于身份验证和加密。

- repo：一个用于存储区块链数据存储库的 repo 客户端。
- blockstore：一个用于存储区块链数据的 blockstore 客户端。
- baseBlocks：一个用于存储基础块的 blockstore 客户端。
- pinning：用于实现身份验证的客户端。

- blocks：用于存储 Hyperledger Fabric 区块的 blocks 客户端。
- dag：用于存储 Hyperledger Fabric 链 DAG 客户端的客户端。
- ipldFetcherFactory：用于存储 Hyperledger Fabric 链 DAG 客户端 fetcher 工厂的客户端。
- unixFSFetcherFactory：用于存储 Hyperledger Fabric 链 DAG 客户端 unixFSPathFetcherFactory 的客户端的客户端。
- peerstore：用于存储 Hyperledger Fabric 节点存储的 peer 存储的客户端。
- peerHost：用于存储 Hyperledger Fabric 节点存储的 peer 存储的主机。
- recordValidator：用于存储 Hyperledger Fabric 区块链的客户端的记录验证器。
- exchange：用于存储 Hyperledger Fabric 链的客户端的客户端。

- namesys：用于存储 Hyperledger Fabric 节点命名系统的客户端。
- routing：用于存储 Hyperledger Fabric 路由的客户端。
- dnsResolver：用于存储 Hyperledger Fabric 节点 DNS 解析器客户端的客户端。
- ipldPathResolver：用于存储 Hyperledger Fabric 链路路径解析器客户端的客户端。
- unixFSPathResolver：用于存储 Hyperledger Fabric 链路路径解析器客户端的客户端。
- provider：用于存储 Hyperledger Fabric 节点提供程序的客户端。
- pubSub：用于存储 Hyperledger Fabric 节点发布订阅消息的客户端的客户端。
- checkPublishAllowed：用于检查是否允许发布所有块。
- checkOnline：用于检查当前是否在线。

- nd：用于存储 Hyperledger Fabric 节点目录节点。
- parentOpts：用于存储 Hyperledger Fabric 节点父节点的选项的客户端。

该代码定义的 Hyperledger Fabric 节点是一个 Hyperledger Fabric 链的节点，该节点使用 Hyperledger Fabric 链作为其数据存储库。该节点还支持与外部的网络交互，并支持存储和检索 Hyperledger Fabric 区块链上的数据。


```
type CoreAPI struct {
	nctx context.Context

	identity   peer.ID
	privateKey ci.PrivKey

	repo       repo.Repo
	blockstore blockstore.GCBlockstore
	baseBlocks blockstore.Blockstore
	pinning    pin.Pinner

	blocks               bserv.BlockService
	dag                  ipld.DAGService
	ipldFetcherFactory   fetcher.Factory
	unixFSFetcherFactory fetcher.Factory
	peerstore            pstore.Peerstore
	peerHost             p2phost.Host
	recordValidator      record.Validator
	exchange             exchange.Interface

	namesys            namesys.NameSystem
	routing            routing.Routing
	dnsResolver        *madns.Resolver
	ipldPathResolver   pathresolver.Resolver
	unixFSPathResolver pathresolver.Resolver

	provider provider.System

	pubSub *pubsub.PubSub

	checkPublishAllowed func() error
	checkOnline         func(allowOffline bool) error

	// ONLY for re-applying options in WithOptions, DO NOT USE ANYWHERE ELSE
	nd         *core.IpfsNode
	parentOpts options.ApiSettings
}

```

这段代码定义了一个名为NewCoreAPI的函数，该函数接收一个IPFS Node对象和一个或多个options.ApiOption选项对象，然后返回一个coreiface.CoreAPI实例。它还定义了一个名为Unixfs的函数，该函数返回一个核心iface.UnixfsAPI接口，该接口实现了go-ipfs Node中的UnixfsAPI。最后，该函数使用WithOptions选项器将一个或多个options.ApiOption选项传递给coreiface.CoreAPI构造函数，以便在创建新实例时初始化它。


```
// NewCoreAPI creates new instance of IPFS CoreAPI backed by go-ipfs Node.
func NewCoreAPI(n *core.IpfsNode, opts ...options.ApiOption) (coreiface.CoreAPI, error) {
	parentOpts, err := options.ApiOptions()
	if err != nil {
		return nil, err
	}

	return (&CoreAPI{nd: n, parentOpts: *parentOpts}).WithOptions(opts...)
}

// Unixfs returns the UnixfsAPI interface implementation backed by the go-ipfs node
func (api *CoreAPI) Unixfs() coreiface.UnixfsAPI {
	return (*UnixfsAPI)(api)
}

```

这段代码定义了三个名为Block、Dag和Name的API接口，分别对应于CoreAPI的Block、Dag和Name方法。

具体来说，当创建一个CoreAPI实例时，可以通过调用Block、Dag或Name方法来获得对应的API接口实现。

例如，在以下代码中，创建了一个名为api的CoreAPI实例，并调用了Block()方法：


blockAPI, err := api.Block()
if err != nil {
	// handle error
}


在这个例子中，api.Block()方法返回了一个BlockAPI类型的接口，可以用来和IPFS节点进行交互，实现对IPFS资源块的写入、读取等操作。

类似地，可以创建Dag和Name的API实例，分别对应于api.Dag和api.Name方法。这些API接口都实现了coreiface.BlockAPI、coreiface.APIDagService和coreiface.NameAPI接口，具有对应的数据存储和操作能力。


```
// Block returns the BlockAPI interface implementation backed by the go-ipfs node
func (api *CoreAPI) Block() coreiface.BlockAPI {
	return (*BlockAPI)(api)
}

// Dag returns the DagAPI interface implementation backed by the go-ipfs node
func (api *CoreAPI) Dag() coreiface.APIDagService {
	return &dagAPI{
		api.dag,
		api,
	}
}

// Name returns the NameAPI interface implementation backed by the go-ipfs node
func (api *CoreAPI) Name() coreiface.NameAPI {
	return (*NameAPI)(api)
}

```

这段代码定义了三个接口，分别是KeyAPI、ObjectAPI和PinAPI，它们都指向了Go-IPFS节点中的KeyAPI、ObjectAPI和PinAPI接口实现。

具体来说，这段代码实现了以下功能：

1. KeyAPI：将一个CoreAPI实例返回，作为实现了KeyAPI接口的实现在这里。
2. ObjectAPI：将一个CoreAPI实例返回，作为实现了ObjectAPI接口的实现在这里。
3. PinAPI：将一个CoreAPI实例返回，作为实现了PinAPI接口的实现在这里。

这些接口都在Go-IPFS节点中用于与IPFS进行交互操作，这里的实现主要涉及到API的使用。


```
// Key returns the KeyAPI interface implementation backed by the go-ipfs node
func (api *CoreAPI) Key() coreiface.KeyAPI {
	return (*KeyAPI)(api)
}

// Object returns the ObjectAPI interface implementation backed by the go-ipfs node
func (api *CoreAPI) Object() coreiface.ObjectAPI {
	return (*ObjectAPI)(api)
}

// Pin returns the PinAPI interface implementation backed by the go-ipfs node
func (api *CoreAPI) Pin() coreiface.PinAPI {
	return (*PinAPI)(api)
}

```

这段代码定义了三个名为"Dht"、"Swarm"和"PubSub"的函数，它们都返回由go-ipfs节点实现的coreiface.DhtAPI、coreiface.SwarmAPI和coreiface.PubSubAPI接口的实现。这些函数都是通过指针变量api来实现的，它们似乎在为api提供Dht、Swarm和PubSub功能的支持。

具体来说，这些函数的作用可能是在某个系统或项目中，通过使用Go-IPFS节点来管理分布式文件系统。这些函数允许通过核心API来注册、发现和使用Swarm、PubSub和Dht API，从而实现更高级别的分布式文件系统管理。


```
// Dht returns the DhtAPI interface implementation backed by the go-ipfs node
func (api *CoreAPI) Dht() coreiface.DhtAPI {
	return (*DhtAPI)(api)
}

// Swarm returns the SwarmAPI interface implementation backed by the go-ipfs node
func (api *CoreAPI) Swarm() coreiface.SwarmAPI {
	return (*SwarmAPI)(api)
}

// PubSub returns the PubSubAPI interface implementation backed by the go-ipfs node
func (api *CoreAPI) PubSub() coreiface.PubSubAPI {
	return (*PubSubAPI)(api)
}

```

This is a Go function that creates a Sub-API service that uses the `Exchange` and `Blockstore` components to implement the necessary functions for the IPNs (Inter-Platform Notebook Service) platform.

The function first checks if the platform is configured to be offline and, if it is, returns an error. It then sets up the necessary configuration settings for the Sub-API, such as the cache size for the IPNS, the DNS resolver, and the name system.

Finally, it sets up the `Exchange` and `Blockstore` components for the Sub-API, and initializes the blockset and the DAG service.

The function returns the initialized Sub-API and a nil error.


```
// Routing returns the RoutingAPI interface implementation backed by the kubo node
func (api *CoreAPI) Routing() coreiface.RoutingAPI {
	return (*RoutingAPI)(api)
}

// WithOptions returns api with global options applied
func (api *CoreAPI) WithOptions(opts ...options.ApiOption) (coreiface.CoreAPI, error) {
	settings := api.parentOpts // make sure to copy
	_, err := options.ApiOptionsTo(&settings, opts...)
	if err != nil {
		return nil, err
	}

	if api.nd == nil {
		return nil, errors.New("cannot apply options to api without node")
	}

	n := api.nd

	subAPI := &CoreAPI{
		nctx: n.Context(),

		identity:   n.Identity,
		privateKey: n.PrivateKey,

		repo:       n.Repo,
		blockstore: n.Blockstore,
		baseBlocks: n.BaseBlocks,
		pinning:    n.Pinning,

		blocks:               n.Blocks,
		dag:                  n.DAG,
		ipldFetcherFactory:   n.IPLDFetcherFactory,
		unixFSFetcherFactory: n.UnixFSFetcherFactory,

		peerstore:          n.Peerstore,
		peerHost:           n.PeerHost,
		namesys:            n.Namesys,
		recordValidator:    n.RecordValidator,
		exchange:           n.Exchange,
		routing:            n.Routing,
		dnsResolver:        n.DNSResolver,
		ipldPathResolver:   n.IPLDPathResolver,
		unixFSPathResolver: n.UnixFSPathResolver,

		provider: n.Provider,

		pubSub: n.PubSub,

		nd:         n,
		parentOpts: settings,
	}

	subAPI.checkOnline = func(allowOffline bool) error {
		if !n.IsOnline && !allowOffline {
			return coreiface.ErrOffline
		}
		return nil
	}

	subAPI.checkPublishAllowed = func() error {
		if n.Mounts.Ipns != nil && n.Mounts.Ipns.IsActive() {
			return errors.New("cannot manually publish while IPNS is mounted")
		}
		return nil
	}

	if settings.Offline {
		cfg, err := n.Repo.Config()
		if err != nil {
			return nil, err
		}

		cs := cfg.Ipns.ResolveCacheSize
		if cs == 0 {
			cs = node.DefaultIpnsCacheSize
		}
		if cs < 0 {
			return nil, fmt.Errorf("cannot specify negative resolve cache size")
		}

		subAPI.routing = offlineroute.NewOfflineRouter(subAPI.repo.Datastore(), subAPI.recordValidator)

		subAPI.namesys, err = namesys.NewNameSystem(subAPI.routing,
			namesys.WithDatastore(subAPI.repo.Datastore()),
			namesys.WithDNSResolver(subAPI.dnsResolver),
			namesys.WithCache(cs))
		if err != nil {
			return nil, fmt.Errorf("error constructing namesys: %w", err)
		}

		subAPI.provider = provider.NewNoopProvider()

		subAPI.peerstore = nil
		subAPI.peerHost = nil
		subAPI.recordValidator = nil
	}

	if settings.Offline || !settings.FetchBlocks {
		subAPI.exchange = offlinexch.Exchange(subAPI.blockstore)
		subAPI.blocks = bserv.New(subAPI.blockstore, subAPI.exchange)
		subAPI.dag = dag.NewDAGService(subAPI.blocks)
	}

	return subAPI, nil
}

```

这段代码定义了一个名为`getSession`的函数，它返回一个名为`api`的`CoreAPI`实例，该实例使用与`api`相同的节点，并使用了一个只读会话。

函数的作用是获取与`api`相同的节点会话，并将其返回。这个会话用于与DAG服务进行交互，并在使用过程中保持一致性，以确保数据一致性。

函数的具体实现包括以下步骤：

1. 从`api`实例中获取`sesAPI`引用。
2. 如果需要，将`dag`设置为新的会话DAG的`NewReadOnlyDagService`，该服务使用相同的数据图和`NewSession`函数，这些参数都来自`api`实例。
3. 返回`api`实例，以便后续调用。

该函数简化了使用DAG服务的方式，使得函数更加容易理解和维护。


```
// getSession returns new api backed by the same node with a read-only session DAG
func (api *CoreAPI) getSession(ctx context.Context) *CoreAPI {
	sesAPI := *api

	// TODO: We could also apply this to api.blocks, and compose into writable api,
	// but this requires some changes in blockservice/merkledag
	sesAPI.dag = dag.NewReadOnlyDagService(dag.NewSession(ctx, api.dag))

	return &sesAPI
}

```

# `/opt/kubo/core/coreapi/dag.go`

这段代码定义了一个名为 "coreapi" 的包。这个包通过导入其它包的方式，将一些库的功能组合在一起，然后导出常用的功能。

具体来说，这个包通过导入以下几个库：

- "github.com/ipfs/boxo/ipld/merkledag"：实现了 BSON11 Merkled Atomic flag。
- "github.com/ipfs/boxo/pinning/pinner"：实现了 BSON11 Pin。
- "github.com/ipfs/boxo/ipld-format"：实现了 BSON11 IPLD 格式。
- "github.com/ipfs/go-cid"：实现了 Go CID。
- "github.com/ipfs/go-ipld-format"：实现了 BSON11 IPLD 格式。
- "go.opentelemetry.io/otel/attribute"：实现了 OpenTelemetry Attribute。
- "go.opentelemetry.io/otel/trace"：实现了 OpenTelemetry Trace。
- "github.com/ipfs/kubo/tracing"：实现了 KubeTracing。

通过导入这些库，我们可以使用它们的接口来创建和操作各种 Merkled Atomic 和 Pin 实例。


```
package coreapi

import (
	"context"

	dag "github.com/ipfs/boxo/ipld/merkledag"
	pin "github.com/ipfs/boxo/pinning/pinner"
	cid "github.com/ipfs/go-cid"
	ipld "github.com/ipfs/go-ipld-format"
	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/trace"

	"github.com/ipfs/kubo/tracing"
)

```

这段代码定义了一个名为 `dagAPI` 的结构体，它包含一个名为 `CoreAPI` 的成员变量，一个名为 `pinningAdder` 的成员变量以及一个名为 `Add` 的成员函数。

`Add` 函数接收一个名为 `nd` 的 `ipld.Node` 参数，然后使用 `tracing.Span` 函数记录一个名为 "CoreAPI.PinningAdder" 的 spans，并设置其 `node` 属性为 `nd` 的 `String()` 形式。函数内部使用 `Add` 函数本身来添加节点，如果遇到错误，则返回。

接着，使用 `span.End()` 函数来关闭 `tracing.Span`，然后使用 `blockstore.PinLock` 函数尝试获取对 `adder.blockstore` 中的 `PinWithMode` 函数的锁，如果失败，则继续执行。最后，使用 `adder.pinning.Flush` 函数来刷新 pin，其中 `Flush` 函数会尝试将所有 pin 都刷新的值都设置为 `ipld.Node` 类型的 `Pin` 结构体，其中 `Pin` 是 `ipld.Node` 类型，它包含一个 `Cid` 字段和一个 `Recursive` 字段，用于在 pin 刷新的模式下工作。


```
type dagAPI struct {
	ipld.DAGService

	core *CoreAPI
}

type pinningAdder CoreAPI

func (adder *pinningAdder) Add(ctx context.Context, nd ipld.Node) error {
	ctx, span := tracing.Span(ctx, "CoreAPI.PinningAdder", "Add", trace.WithAttributes(attribute.String("node", nd.String())))
	defer span.End()
	defer adder.blockstore.PinLock(ctx).Unlock(ctx)

	if err := adder.dag.Add(ctx, nd); err != nil {
		return err
	}

	if err := adder.pinning.PinWithMode(ctx, nd.Cid(), pin.Recursive); err != nil {
		return err
	}

	return adder.pinning.Flush(ctx)
}

```

这段代码定义了一个名为`func`的函数，接收一个名为`adder`的指针和一个包含`ipld.Node`类型的参数`nds`。函数的作用是在给定上下文`ctx`中，对给定的`nds`数组的元素执行异步操作，然后返回操作结果。

具体来说，这段代码实现了如下的操作流程：

1. 在函数开始时，使用`tracing.Span`函数记录操作的跟踪信息，其中`ctx`参数用于传递给`tracing.WithAttributes`的参数，包括`attribute.Int`类型，指定`nodes.count`的值为`len(nds)`。`span`用于跟踪操作的时间和空间消耗。

2. 在操作开始之前，使用`defer`关键字标记，表示需要在操作完成时执行的操作，这里解锁`adder.blockstore.PinLock`操作的锁，并结束`span`。

3. 尝试使用`adder.dag.AddMany`方法，将给定的`nds`数组的元素添加到给定的`ctx`上下文中。如果该操作发生错误，函数返回错误信息。

4. 遍历给定的`nds`数组，对每个元素调用`cid.NewSet`方法创建一个新的`cid.Set`，并检查它是否属于已经存在的`cid.Set`。如果是，说明已经创建过该`cid`，可以继续执行，否则会尝试使用`adder.pinning.PinWithMode`方法将该`cid`与`pin.Recursive`模式相关联，并将结果存储到一个新的`cid`中。

5. 如果所有`cid.Set`都已经处理完毕，使用`adder.pinning.Flush`方法将`adder.pinning.Flush`操作的结果返回，该操作会将所有已经创建好的`cid`立即释放。


```
func (adder *pinningAdder) AddMany(ctx context.Context, nds []ipld.Node) error {
	ctx, span := tracing.Span(ctx, "CoreAPI.PinningAdder", "AddMany", trace.WithAttributes(attribute.Int("nodes.count", len(nds))))
	defer span.End()
	defer adder.blockstore.PinLock(ctx).Unlock(ctx)

	if err := adder.dag.AddMany(ctx, nds); err != nil {
		return err
	}

	cids := cid.NewSet()

	for _, nd := range nds {
		c := nd.Cid()
		if cids.Visit(c) {
			if err := adder.pinning.PinWithMode(ctx, c, pin.Recursive); err != nil {
				return err
			}
		}
	}

	return adder.pinning.Flush(ctx)
}

```

这两段代码定义了一个名为 `func` 的函数，它是 `ipld.NodeAdder` 类型的指针，负责实现 DAG（数据驱动 Application 端微服务）框架中的数据挂载功能。

具体来说，这两段代码实现了以下功能：

1. `Pinning` 函数接收一个名为 `api` 的 `dagAPI` 类型的参数，然后返回一个指向 `ipld.NodeAdder` 类型的指针，该指针实现了 `PinningAdder` 接口，负责将 DAG 中的节点数据挂载到输入的 `api` 参数上。
2. `Session` 函数接收一个名为 `ctx` 的 `context.Context` 类型的参数，然后返回一个名为 `ipld.NodeGetter` 的函数类型，该函数类型实现了 `dag.SessionMaker` 接口，负责创建并返回一个 `Session` 对象，将 `api` 参数作为 `dagService` 字段传递给 `NewSession` 函数，从而创建一个新的 `Session` 对象。


```
func (api *dagAPI) Pinning() ipld.NodeAdder {
	return (*pinningAdder)(api.core)
}

func (api *dagAPI) Session(ctx context.Context) ipld.NodeGetter {
	return dag.NewSession(ctx, api.DAGService)
}

var (
	_ ipld.DAGService  = (*dagAPI)(nil)
	_ dag.SessionMaker = (*dagAPI)(nil)
)

```

# `/opt/kubo/core/coreapi/dht.go`

该代码包是一个名为"coreapi"的包，它导入了以下依赖项：

- blockservice，用于与块存储服务进行交互
- blockstore，用于与本地块存储进行交互
- coreiface，用于与Core Interface进行交互
- caopts，用于配置Core Interface的选项
- offline，用于在网络或设备上管理离线交互
- dag，用于构建分布式数据链路树
- path，用于路径编码
- cid，用于客户端协调器(CID)客户端 ID
- cidutil，用于管理CID客户端ID
- offsets，用于管理文件的本地偏移量
- pelist，用于构建元数据平原
- rest，用于提供WebSocket REST API

该代码可能用于开发块存储服务的SDK或客户端库。


```
package coreapi

import (
	"context"
	"fmt"

	blockservice "github.com/ipfs/boxo/blockservice"
	blockstore "github.com/ipfs/boxo/blockstore"
	coreiface "github.com/ipfs/boxo/coreiface"
	caopts "github.com/ipfs/boxo/coreiface/options"
	offline "github.com/ipfs/boxo/exchange/offline"
	dag "github.com/ipfs/boxo/ipld/merkledag"
	"github.com/ipfs/boxo/path"
	cid "github.com/ipfs/go-cid"
	cidutil "github.com/ipfs/go-cidutil"
	"github.com/ipfs/kubo/tracing"
	peer "github.com/libp2p/go-libp2p/core/peer"
	routing "github.com/libp2p/go-libp2p/core/routing"
	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/trace"
)

```

这段代码定义了一个名为 `DhtAPI` 的接口 `CoreAPI`，该接口实现了 `FindPeer` 函数。函数接收两个参数，一个是 `ctx` 上下文，另一个是 `peer.ID` 类型的变量 `p`。函数返回一个 `peer.AddrInfo` 类型的变量 `pi` 和一个非错误的错误 `err`。

函数的具体实现包括以下步骤：

1. 调用 `api.checkOnline` 函数，检查是否在线。函数的参数 `false` 表示不在线，返回一个非空错误。

2. 如果步骤 1 的操作成功，尝试调用 `api.routing.FindPeer` 函数。该函数接收 `ctx` 和 `peer.ID` 两个参数，并尝试使用这个 `peer.ID` 查找与该 `peer.ID` 对应的 peer。

3. 如果 `api.routing.FindPeer` 函数没有返回错误，步骤 2 的操作成功。步骤 3 尝试返回 `pi` 和一个非错误的结果。如果返回非错误，则将结果赋给 `pi` 并返回。如果返回错误，则将错误作为 `err` 返回。


```
type DhtAPI CoreAPI

func (api *DhtAPI) FindPeer(ctx context.Context, p peer.ID) (peer.AddrInfo, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.DhtAPI", "FindPeer", trace.WithAttributes(attribute.String("peer", p.String())))
	defer span.End()
	err := api.checkOnline(false)
	if err != nil {
		return peer.AddrInfo{}, err
	}

	pi, err := api.routing.FindPeer(ctx, peer.ID(p))
	if err != nil {
		return peer.AddrInfo{}, err
	}

	return pi, nil
}

```

该函数的作用是使用DhtAPI协议中的`FindProviders`函数查找Dht网络中的提供者。它接收一个名为`ctx`的上下文和两个路径参数`p`和`opts`。`opts`参数是一个传递给`caopts.DhtFindProvidersOption`的选项参数数组，用于设置Dht API查找提供者的选项。函数返回一个`<-chan peer.AddrInfo, error>`类型的通道，用于返回发现的可用的提供者地址。如果函数在执行期间出现错误，它将返回一个`error`类型的参数。


```
func (api *DhtAPI) FindProviders(ctx context.Context, p path.Path, opts ...caopts.DhtFindProvidersOption) (<-chan peer.AddrInfo, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.DhtAPI", "FindProviders", trace.WithAttributes(attribute.String("path", p.String())))
	defer span.End()

	settings, err := caopts.DhtFindProvidersOptions(opts...)
	if err != nil {
		return nil, err
	}
	span.SetAttributes(attribute.Int("numproviders", settings.NumProviders))

	err = api.checkOnline(false)
	if err != nil {
		return nil, err
	}

	rp, _, err := api.core().ResolvePath(ctx, p)
	if err != nil {
		return nil, err
	}

	numProviders := settings.NumProviders
	if numProviders < 1 {
		return nil, fmt.Errorf("number of providers must be greater than 0")
	}

	pchan := api.routing.FindProvidersAsync(ctx, rp.RootCid(), numProviders)
	return pchan, nil
}

```

这段代码是一个名为`func`的函数，接受一个名为`api`的DhtAPI实例，以及一个路径参数`path`，和一个可选的`opts`参数，然后返回一个`error`。

函数的作用是使用DhtAPI提供API服务，根据传入的参数，自动调用相关选项，并在调用完成后返回结果。具体的实现步骤如下：

1. 调用`caopts.DhtProvideOptions`函数，将传入的`opts`参数传递给该函数，该函数返回一个设置选项的`settings`变量，需要判断其是否成功。
2. 设置API的`online`状态为`false`，即不在线。
3. 根据传入的`opts`参数，设置`recursive`选项为`true`，表示是否递归提供子节点。
4. 调用`api.checkOnline`函数，检查API是否在线。
5. 如果API在线，则调用`api.core().ResolvePath`函数，根据传入的路径和子节点设置，返回一个根节点`c`。
6. 如果子节点设置为`true`，则递归提供子节点。
7. 如果设置`recursive`为`true`且递归提供了子节点，则返回根节点`c`。
8. 最后，返回一个`error`，如果没有错误，则返回`nil`。


```
func (api *DhtAPI) Provide(ctx context.Context, path path.Path, opts ...caopts.DhtProvideOption) error {
	ctx, span := tracing.Span(ctx, "CoreAPI.DhtAPI", "Provide", trace.WithAttributes(attribute.String("path", path.String())))
	defer span.End()

	settings, err := caopts.DhtProvideOptions(opts...)
	if err != nil {
		return err
	}
	span.SetAttributes(attribute.Bool("recursive", settings.Recursive))

	err = api.checkOnline(false)
	if err != nil {
		return err
	}

	rp, _, err := api.core().ResolvePath(ctx, path)
	if err != nil {
		return err
	}

	c := rp.RootCid()

	has, err := api.blockstore.Has(ctx, c)
	if err != nil {
		return err
	}

	if !has {
		return fmt.Errorf("block %s not found locally, cannot provide", c)
	}

	if settings.Recursive {
		err = provideKeysRec(ctx, api.routing, api.blockstore, []cid.Cid{c})
	} else {
		err = provideKeys(ctx, api.routing, []cid.Cid{c})
	}
	if err != nil {
		return err
	}

	return nil
}

```

这两函数定义了 provideKeys 和 provideKeysRec，它们都是用于在总线路上注册提供者（client）的函数。

在 provideKeys 中，该函数接收一个上下文（Context）、一个路由（r）和一个由客户端ID组成的列表（cids）。然后，它遍历客户端ID列表并将它们传递给路由的 provide 函数。如果提供者返回一个非空错误，它将在函数中返回。否则，它将返回一个非空空集，表示成功提供了客户端ID。

在 provideKeysRec 中，该函数与 provideKeys 类似，但使用了recursive受害者的模式。它接收一个上下文（Context）、一个路由（r）和一个由客户端ID组成的列表（cids）。然而，它使用一个名为 blockservice 的块存储服务作为受害人。

在函数内部，它使用一个名为 dag 的受保护的 DAG 服务创建一个 DAG 服务。它使用 dag.Walk 函数跟踪从受害人处获取到的客户端 ID，并使用 blockservice 的 blockservice.New 函数创建一个块存储服务客户端。然后，它使用 dag.GetLinksDirect 函数获取与每个客户端 ID 相关的链接，并使用 provided.Visitor 函数将其传递给受害人。如果受害人返回一个非空错误，它将在函数中返回。否则，它将返回一个非空空集，表示成功提供了客户端ID。


```
func provideKeys(ctx context.Context, r routing.Routing, cids []cid.Cid) error {
	for _, c := range cids {
		err := r.Provide(ctx, c, true)
		if err != nil {
			return err
		}
	}
	return nil
}

func provideKeysRec(ctx context.Context, r routing.Routing, bs blockstore.Blockstore, cids []cid.Cid) error {
	provided := cidutil.NewStreamingSet()

	errCh := make(chan error)
	go func() {
		dserv := dag.NewDAGService(blockservice.New(bs, offline.Exchange(bs)))
		for _, c := range cids {
			err := dag.Walk(ctx, dag.GetLinksDirect(dserv), c, provided.Visitor(ctx))
			if err != nil {
				errCh <- err
			}
		}
	}()

	for {
		select {
		case k := <-provided.New:
			err := r.Provide(ctx, k, true)
			if err != nil {
				return err
			}
		case err := <-errCh:
			return err
		case <-ctx.Done():
			return ctx.Err()
		}
	}
}

```

该函数是一个指针变量，接收一个名为"api"的DhtAPI类型的大量的int类型参数。

首先，将一个名为"*CoreAPI"的类型别名定义为指针变量。

然后，通过将"*api"复制到一个名为"api"的"DhtAPI"类型变量中，创建了一个指向DhtAPI类型中包含"core"函数的指针。

最后，通过将"*api"复制到一个名为"api"的"CoreAPI"类型变量中，创建了一个指向CoreAPI类型中包含"core"函数的指针。

函数返回的是一个指向CoreAPI类型中包含"core"函数的指针，这个函数接收一个DhtAPI类型的参数，然后执行DhtAPI中包含的"core"函数。


```
func (api *DhtAPI) core() coreiface.CoreAPI {
	return (*CoreAPI)(api)
}

```

# `/opt/kubo/core/coreapi/key.go`

这段代码定义了一个名为 "coreapi" 的包。它导入了来自以下外部库的依赖项：

- "github.com/ipfs/boxo/coreiface"
- "github.com/ipfs/boxo/coreiface/options"
- "github.com/ipfs/boxo/ipns"
- "github.com/ipfs/boxo/path"
- "github.com/ipfs/kubo/tracing"
- "github.com/libp2p/go-libp2p/core/crypto"
- "github.com/libp2p/go-libp2p/core/peer"
- "go.opentelemetry.io/otel/attribute"
- "go.opentelemetry.io/otel/trace"

还可以导入了以下一个非常依赖的外部库的导出：

- "github.com/ipfs/boxo/core/cello"

这个包的主要目的是提供用于与 IPFS 存储桶进行交互的库。它允许用户通过提供配置选项来使用不同的存储桶。


```
package coreapi

import (
	"context"
	"crypto/rand"
	"errors"
	"fmt"
	"sort"

	coreiface "github.com/ipfs/boxo/coreiface"
	caopts "github.com/ipfs/boxo/coreiface/options"
	"github.com/ipfs/boxo/ipns"
	"github.com/ipfs/boxo/path"
	"github.com/ipfs/kubo/tracing"
	crypto "github.com/libp2p/go-libp2p/core/crypto"
	peer "github.com/libp2p/go-libp2p/core/peer"
	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/trace"
)

```

这段代码定义了一个名为KeyAPI的接口类型和两个键值类型Key和Value。

首先定义了一个名为key的结构体类型，该类型包含一个名为name的字符串类型变量、一个名为peerID的peer.ID类型变量和一个名为path的path.Path类型变量。

接着定义了一个名为newKey的函数类型，该函数接收两个参数，一个是字符串类型的name参数和一个peer.ID类型的pid参数。函数返回一个名为key的元组类型和一个名为error的错误类型。函数首先创建一个名为"/ipns/" + ipns.NameFromPeer(pid).String()的路径，如果该路径不存在，则返回nil和错误。然后将创建的路径赋值给key.path变量，并将key的name赋值为传入的name参数，将key的peerID赋值为传入的pid参数。

最后，该函数将返回一个指向key类型的元组，如果没有错误，则该元组包含一个名为key的实参和一个名为error的实参。


```
type KeyAPI CoreAPI

type key struct {
	name   string
	peerID peer.ID
	path   path.Path
}

func newKey(name string, pid peer.ID) (*key, error) {
	p, err := path.NewPath("/ipns/" + ipns.NameFromPeer(pid).String())
	if err != nil {
		return nil, err
	}
	return &key{
		name:   name,
		peerID: pid,
		path:   p,
	}, nil
}

```

这是一段使用Go编程语言编写的函数，定义了名为`key`的`*key`类型。该类型定义了三个函数：`Name`、`Path`和`ID`。下面是这三段函数的实现：
go
// Name returns the key name
func (k *key) Name() string {
	return k.name
}

// Path returns the path of the key.
func (k *key) Path() path.Path {
	return k.path
}

// ID returns key PeerID
func (k *key) ID() peer.ID {
	return k.peerID
}

首先，`key`类型是一个`*key`类型，它表示一个与键相关联的对象。这个类型中定义了三个函数，用于返回与键相关的信息。

第一个函数`Name`函数返回键的名称，它直接将键的名称返回。

第二个函数`Path`函数返回键的路径，它通过`key.path`访问键的路径并返回。

第三个函数`ID`函数返回键的PeerID，它通过`key.peerID`访问键的PeerID并返回。


```
// Name returns the key name
func (k *key) Name() string {
	return k.name
}

// Path returns the path of the key.
func (k *key) Path() path.Path {
	return k.path
}

// ID returns key PeerID
func (k *key) ID() peer.ID {
	return k.peerID
}

```

This is a function that generates a new keystore entry for a given key in the API of the KeyAPI layer of theCoreAPI microservice.

Here is how you can use it:

* The first argument is the name of the keystore entry. This should be unique within the organization. If you want to use a name that is already in use, you should either use the `coreiface.NamePrefix` annotation to use a naming convention, or use the `coreiface.LongRanger` annotation to use a name that is unique within your organization.
* The second argument is an error that may be returned by the API. This error is the result of an error that occurred while generating the keystore entry.
* The `KeyGenerateOptions` struct is used to specify the options for generating a new keystore entry. The `Algorithm` field specifies the type of key that you want to generate. The `Size` field specifies the size of the keystore entry in bytes.
* The `GenerateKeyPairWithReader` function is used to generate a new key pair for the specified algorithm. This function is provided by the `crypto.GenerateKeyPairWithReader` method in the crypto package.
* The `api.repo.Keystore().Get` method is used to retrieve the existing keystore entry for the specified name. If the keystore entry exists, an error will be returned.
* The `crypto.GenerateEd25519Key` function is used to generate a new Ed25519 key for the specified algorithm. This function is provided by the `crypto.GenerateEd25519Key` method in the crypto package.
* The `api.repo.Keystore().Put` method is used to store the new keystore entry in the API. This method is provided by the `api.repo.Keystore` interface in the KeyAPI layer of theCoreAPI microservice.
* The `peer.IDFromPublicKey` function is used to convert the public key to a peer ID. This function is provided by the `peer.IDFromPublicKey` method in the peer package.
* The `newKey` function returns a new keystore entry with the specified name. This function returns a `coreiface.Key` object.


```
// Generate generates new key, stores it in the keystore under the specified
// name and returns a base58 encoded multihash of its public key.
func (api *KeyAPI) Generate(ctx context.Context, name string, opts ...caopts.KeyGenerateOption) (coreiface.Key, error) {
	_, span := tracing.Span(ctx, "CoreAPI.KeyAPI", "Generate", trace.WithAttributes(attribute.String("name", name)))
	defer span.End()

	options, err := caopts.KeyGenerateOptions(opts...)
	if err != nil {
		return nil, err
	}

	if name == "self" {
		return nil, fmt.Errorf("cannot create key with name 'self'")
	}

	_, err = api.repo.Keystore().Get(name)
	if err == nil {
		return nil, fmt.Errorf("key with name '%s' already exists", name)
	}

	var sk crypto.PrivKey
	var pk crypto.PubKey

	switch options.Algorithm {
	case "rsa":
		if options.Size == -1 {
			options.Size = caopts.DefaultRSALen
		}

		priv, pub, err := crypto.GenerateKeyPairWithReader(crypto.RSA, options.Size, rand.Reader)
		if err != nil {
			return nil, err
		}

		sk = priv
		pk = pub
	case "ed25519":
		priv, pub, err := crypto.GenerateEd25519Key(rand.Reader)
		if err != nil {
			return nil, err
		}

		sk = priv
		pk = pub
	default:
		return nil, fmt.Errorf("unrecognized key type: %s", options.Algorithm)
	}

	err = api.repo.Keystore().Put(name, sk)
	if err != nil {
		return nil, err
	}

	pid, err := peer.IDFromPublicKey(pk)
	if err != nil {
		return nil, err
	}

	return newKey(name, pid)
}

```

这段代码定义了一个名为 `List` 的函数，它返回一个键存储在 `keystore` 中的列表。函数的实现包括以下步骤：

1. 从 `repo` 实例的 `Keystore` 中检索键列表。
2. 对列表中的每个键进行排序。
3. 创建一个长度为 `len(keys)+1` 的新列表，用于存储按键值排序后的键。
4. 对每个键进行操作，如果成功，将其添加到新列表中。
5. 如果出现错误，返回一个非空错误对象。

函数的输入参数是一个名为 `ctx` 的 `Context` 实例，它包含了当前的上下文信息。函数的输出是一个由 `coreiface.Key` 类型组成的切片，它包含了按键值排序后的键。


```
// List returns a list keys stored in keystore.
func (api *KeyAPI) List(ctx context.Context) ([]coreiface.Key, error) {
	_, span := tracing.Span(ctx, "CoreAPI.KeyAPI", "List")
	defer span.End()

	keys, err := api.repo.Keystore().List()
	if err != nil {
		return nil, err
	}

	sort.Strings(keys)

	out := make([]coreiface.Key, len(keys)+1)
	out[0], err = newKey("self", api.identity)
	if err != nil {
		return nil, err
	}

	for n, k := range keys {
		privKey, err := api.repo.Keystore().Get(k)
		if err != nil {
			return nil, err
		}

		pubKey := privKey.GetPublic()

		pid, err := peer.IDFromPublicKey(pubKey)
		if err != nil {
			return nil, err
		}

		out[n+1], err = newKey(k, pid)
		if err != nil {
			return nil, err
		}
	}
	return out, nil
}

```

This is a Go function called `RenameKey`. It renames a keystore file, given a new name and an optional `force` flag.

It works as follows:

1. It checks if the key with the name `oldName` exists. If it does, it checks if the `force` flag is set to `true`. If it's not set, it returns an error.
2. If the `force` flag is set to `true`, it checks if the `oldName` is the same as `newName`. If it's not, it returns an error.
3. If `newName` is the same as `oldName`, it creates a new key with the name `newName`. It returns the new key, `false`, and an error if the renaming fails.
4. If `oldName` exists and `force` is not set, it renames the key by deleting the old key and creating a new one with the same name. It returns `true`, the new key, and the error if the renaming fails.
5. If `force` is set, it creates a new key with the name `newName` and renames it by deleting the old key. It returns `true`, the new key, and the error if the renaming fails.
6. If any error occurs, it returns an error.

It is written in the Go programming language and is using the `peer` package to handle the `peer.IDFromPublicKey` function.


```
// Rename renames `oldName` to `newName`. Returns the key and whether another
// key was overwritten, or an error.
func (api *KeyAPI) Rename(ctx context.Context, oldName string, newName string, opts ...caopts.KeyRenameOption) (coreiface.Key, bool, error) {
	_, span := tracing.Span(ctx, "CoreAPI.KeyAPI", "Rename", trace.WithAttributes(attribute.String("oldname", oldName), attribute.String("newname", newName)))
	defer span.End()

	options, err := caopts.KeyRenameOptions(opts...)
	if err != nil {
		return nil, false, err
	}
	span.SetAttributes(attribute.Bool("force", options.Force))

	ks := api.repo.Keystore()

	if oldName == "self" {
		return nil, false, fmt.Errorf("cannot rename key with name 'self'")
	}

	if newName == "self" {
		return nil, false, fmt.Errorf("cannot overwrite key with name 'self'")
	}

	oldKey, err := ks.Get(oldName)
	if err != nil {
		return nil, false, fmt.Errorf("no key named %s was found", oldName)
	}

	pubKey := oldKey.GetPublic()

	pid, err := peer.IDFromPublicKey(pubKey)
	if err != nil {
		return nil, false, err
	}

	// This is important, because future code will delete key `oldName`
	// even if it is the same as newName.
	if newName == oldName {
		k, err := newKey(oldName, pid)
		return k, false, err
	}

	overwrite := false
	if options.Force {
		exist, err := ks.Has(newName)
		if err != nil {
			return nil, false, err
		}

		if exist {
			overwrite = true
			err := ks.Delete(newName)
			if err != nil {
				return nil, false, err
			}
		}
	}

	err = ks.Put(newName, oldKey)
	if err != nil {
		return nil, false, err
	}

	err = ks.Delete(oldName)
	if err != nil {
		return nil, false, err
	}

	k, err := newKey(newName, pid)
	return k, overwrite, err
}

```

此代码是一个名为 `KeyAPI` 的函数，它从 keystore 中移除给定名称的键，并返回键的 IPNS 路径。

函数接受一个名为 `ctx` 的上下文，一个名为 `name` 的字符串参数和一个名为 `api` 的引用参数 `api`。

函数内部首先创建一个名为 `ks` 的 keystore 实例。然后，它检查给定的键名是否为 "self"。如果是，函数返回 `nil`，并输出一条错误消息。

接下来，函数使用 `ks.Get(name)` 从 keystore 中获取给定名称的键。如果获取失败，函数返回 `nil`，并输出一条错误消息。

如果键已经被成功移除，函数使用 `peer.IDFromPublicKey(pubKey)` 获取键的 IPNS 路径。然后，它尝试使用 `ks.Delete(name)` 从 keystore 中删除给定名称的键。如果此操作成功，函数返回一个新的键，该键的 IPNS 路径为 "/removed"。

如果出现任何错误，函数将在栈中记录一条名为 "CoreAPI.KeyAPI.Remove" 的跟踪，以便稍后进行调试。


```
// Remove removes keys from keystore. Returns ipns path of the removed key.
func (api *KeyAPI) Remove(ctx context.Context, name string) (coreiface.Key, error) {
	_, span := tracing.Span(ctx, "CoreAPI.KeyAPI", "Remove", trace.WithAttributes(attribute.String("name", name)))
	defer span.End()

	ks := api.repo.Keystore()

	if name == "self" {
		return nil, fmt.Errorf("cannot remove key with name 'self'")
	}

	removed, err := ks.Get(name)
	if err != nil {
		return nil, fmt.Errorf("no key named %s was found", name)
	}

	pubKey := removed.GetPublic()

	pid, err := peer.IDFromPublicKey(pubKey)
	if err != nil {
		return nil, err
	}

	err = ks.Delete(name)
	if err != nil {
		return nil, err
	}

	return newKey("", pid)
}

```

这是一段使用Go语言编写的函数指针类型，它接收一个名为"api"的KeyAPI类型的参数，并返回一个名为"coreiface.Key"的类型和一个名为"error"的错误类型的值。

具体来说，这段代码实现了一个名为"Self"的函数，它接收一个上下文上下文(Context)，并返回一个键(Key)和一个错误(Error)。函数的实现分为以下几个步骤：

1. 如果传递的参数"api"的标识符为空字符串，函数将返回一个名为"nil"的值，并且抛出一个名为"errors.New"的错误。

2. 如果传递的参数"api"的标识符不为空字符串，函数使用"newKey"函数创建一个名为"self"的自定义键，并将其与传递的参数"api"的标识符连接起来，得到一个新的键。最后，函数返回这个新的键和一个空的键(表示没有错误)。

3. 如果函数在调用时失败，它将抛出一个名为"error"的错误。


```
func (api *KeyAPI) Self(ctx context.Context) (coreiface.Key, error) {
	if api.identity == "" {
		return nil, errors.New("identity not loaded")
	}

	return newKey("self", api.identity)
}

```