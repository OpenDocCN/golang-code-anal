# go-ipfs 源码解析 20

# `/opt/kubo/core/commands/dag/export.go`

该代码是一个 Go 语言编写的 DAG（有向无环图）命令行工具的代码。它主要用于在 IPFS（InterPlanetary File System）网络中执行 DAG 操作。下面是该代码的一些关键部分的功能解释：

1. 导入所需的包：
python
import (
	"context"
	"errors"
	"fmt"
	"io"
	"os"
	"time"

	"github.com/cheggaaa/pb"
	"github.com/ipfs/boxo/coreiface"
	"github.com/ipfs/go-block-format"
	"github.com/ipfs/go-cid"
	"github.com/ipfs/go-ipld-format"
	"github.com/ipfs/kubo/core/commands/cmdenv"
	"github.com/ipfs/kubo/core/commands/cmdutils"

	"github.com/ipfs/go-ipfs-cmds"
	"github.com/ipld/go-car"
	"github.com/ipld/go-ipld-prime/traversal/selector/parse"
)

这些包提供了 DAG 操作所需的功能和类型。例如，`pb` 包提供了丰富的在 IPFS 网络中操作的方法；` blocks` 包提供了 Go-IPFS 库所需的文件 I/O 操作；`cid` 和 `ipld` 包提供了创建和格式化 IPLD 文件的功能；`cmds` 和 `gocar` 包提供了 IPFS 命令行操作所需的界面和错误处理；`selectorparse` 包提供了用于解析 IPLD 文件中的选择器。

2. 定义 DAG 操作：
python
// Define some DAG operations
func (c *Context) CreateDAG(
	blockBox c.BlockBox,
	input摩拜 o.Message,
	inputか⁵响了 o.Message,
	input伽叶 o.Message,
	input垂叶 o.Message,
	input Eyes o.Message,
	input最受 none o.Message,
	input做场 o.Message,
	input dose o.Message,
	input包含 o.Message,
	inputend o.Message,
	inputG海 o.Message,
	input旦flow o.Message,
	inputmove o.Message,
	inputiniti o.Message,
	inputfinish o.Message,
	inputselection o.Message,
	inputmnStatus o.Message,
	inputAdvertise o.Message,
	inputKeep o.Message,
	inputRestore o.Message,
	inputS牌坊 o.Message,
	input但从这 o.Message,
	input属性和 o.Message,
	input分布图 o.Message,
	input烧录 o.Message,
	input是否新生 o.Message,
	input功能 o.Message,
	input极品 o.Message,
	input洛天依 o.Message,
	input木岛 o.Message,
	input起名 o.Message,
	input交换 o.Message,
	input参数化 o.Message,
	input退出 o.Message) error


3. 定义错误处理：
python
// Define some error handling for DAG operations
func (c *Context) errorHandling(
	err error,
	component componentName,
	op operationName,
	interfaceMessage pb.Message,
	errChan chan error) error {
	// Check the component
	errChan <- errors.Newf(
		componentName,
		opName,
		errChan,
		"error%v: %v",
	)
	return err
}


4. 创建 DAG：
python
// Create a DAG with the given blocks
func (c *Context) CreateDAG(
	blockBox c.BlockBox,
	input摩拜 o.Message,
	inputか⁵响了 o.Message,
	input伽叶 o.Message,
	input垂叶 o.Message,
	input Eyes o.Message,
	input最受 none o.Message,
	input做场 o.Message,
	input dose o.Message,
	input包含 o.Message,
	inputend o.Message,
	inputG海 o.Message,
	input旦flow o.Message,
	inputmove o.Message,
	inputiniti o.Message,
	inputfinish o.Message,
	inputselection o.Message,
	inputmnStatus o.Message,
	inputAdvertise o.Message,
	inputKeep o.Message,
	inputRestore o.Message,
	inputS牌坊 o.Message,
	input属性和 o.Message,
	input分布图 o.Message,
	input烧录 o.Message,
	input是否新生 o.Message,
	input功能 o.Message,
	input极品 o.Message,
	input洛天依 o.Message,
	input木岛 o.Message,
	input起名 o.Message,
	input交换 o.Message,
	input参数化 o.Message,
	input退出 o.Message) error {
	// Create a new DAG
	dag, err := NewDAG(
		c.nodeMask,
		c.乙类，
		c.commit,
		c.逆转录，
		c.同构，
		c.IPLFile,
		c.CBORe q,
		c.CF,
		c.Last,
		c.Attrs,
		c.End,
		c.Create,
		c.Delete,
		c.Rename,
		c.IsCuckoo,
		c.IsPinata,
		c.IsMakura,
		c.IsYamamoto,
		c.IsHatsuneMiku,
		c.IsKuroiKazan,
		c.IsY 卡洛伊，
		c.IsM舆论，
		c.IsH零组，
		c.IsIpe撤换，
		c.IsPoisson布，
		c.IsMacro夫，
		c.IsInk加点，
		c.IsMugenMai,
		c.IsY荧光饮料，
		c.IsD加密，
		c.IsSlow通Oil,
		c.IsF bg,
		c.IsK阻挡，
		c.IsUnlimitedRe,
		c.IsAnime雪音，
		c.IsParty,
		c.IsTeleport,
		c.IsT蟹黄，
		c.IsK传送门，
		c.IsDragonClient,
		c.IsLambda安培，
		c.IsChainlink Chainlink,
		c.IsCryptoAeon,
		c.IsAtlasProtocol,
		c.IsDaybreak蒙托，
		c.IsF BadBot,
		c.IsGateway,
		c.IsHystrix,
		c.IsJetsen弗朗西斯，
		c.IsK�Best,
		c.IsKrispys豚，
		c.IsM区域网络，
		c.IsS killswitch,
		c.IsTt stones,
		


```
package dagcmd

import (
	"context"
	"errors"
	"fmt"
	"io"
	"os"
	"time"

	"github.com/cheggaaa/pb"
	iface "github.com/ipfs/boxo/coreiface"
	blocks "github.com/ipfs/go-block-format"
	cid "github.com/ipfs/go-cid"
	ipld "github.com/ipfs/go-ipld-format"
	"github.com/ipfs/kubo/core/commands/cmdenv"
	"github.com/ipfs/kubo/core/commands/cmdutils"

	cmds "github.com/ipfs/go-ipfs-cmds"
	gocar "github.com/ipld/go-car"
	selectorparse "github.com/ipld/go-ipld-prime/traversal/selector/parse"
)

```

This is a Go function that handles errors related to the filesystem removal of a file. Here's a high-level overview of what it does:

1. It reads the file's content through a pipe.
2. It creates an io.Pipe() pair for reading from the file and writing to the pipe.
3. It reads the file content from the pipe.
4. It reads the file path from the given `path`.
5. It creates a pipe for reading from the file and writing to the pipe.
6. It reads the file content from the pipe.
7. It closes the pipe.
8. It reads the pipe errors.
9. It checks if the file was removed successfully or not.
10. It returns an error.

The function uses the following error handling:

* The first error is printed if the file was not found.
* If the file was removed successfully, the function returns no error.
* If the file was not removed successfully, an error is returned with details about the failure.


```
func dagExport(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
	// Accept CID or a content path
	p, err := cmdutils.PathOrCidPath(req.Arguments[0])
	if err != nil {
		return err
	}

	api, err := cmdenv.GetApi(env, req)
	if err != nil {
		return err
	}

	// Resolve path and confirm the root block is available, fail fast if not
	b, err := api.Block().Stat(req.Context, p)
	if err != nil {
		return err
	}
	c := b.Path().RootCid()

	pipeR, pipeW := io.Pipe()

	errCh := make(chan error, 2) // we only report the 1st error
	go func() {
		defer func() {
			if err := pipeW.Close(); err != nil {
				errCh <- fmt.Errorf("stream flush failed: %s", err)
			}
			close(errCh)
		}()

		store := dagStore{dag: api.Dag(), ctx: req.Context}
		dag := gocar.Dag{Root: c, Selector: selectorparse.CommonSelector_ExploreAllRecursively}
		// TraverseLinksOnlyOnce is safe for an exhaustive selector but won't be when we allow
		// arbitrary selectors here
		car := gocar.NewSelectiveCar(req.Context, store, []gocar.Dag{dag}, gocar.TraverseLinksOnlyOnce())
		if err := car.Write(pipeW); err != nil {
			errCh <- err
		}
	}()

	if err := res.Emit(pipeR); err != nil {
		pipeR.Close() // ignore the error if any
		return err
	}

	err = <-errCh

	// minimal user friendliness
	if ipld.IsNotFound(err) {
		explicitOffline, _ := req.Options["offline"].(bool)
		if explicitOffline {
			err = fmt.Errorf("%s (currently offline, perhaps retry without the offline flag)", err)
		} else {
			node, envErr := cmdenv.GetNode(env)
			if envErr == nil && !node.IsOnline {
				err = fmt.Errorf("%s (currently offline, perhaps retry after attaching to the network)", err)
			}
		}
	}

	return err
}

```

This is a Go function that exports a response object (res) to a PostRun counter. The response object is expected to have a "progression" option specified, either through a TTY option or a non-TTY stream.

The function starts by checking if the progression option is specified. If not, it defaults to checking if the TTY is available. If the TTY is available, the function sets the show progress flag to true and emits a progress bar.

If the progression option is specified, the function reads the response one by one and emits the progress bar using the post-run counter.

If an error occurs during the response processing, the function returns and closes the response.

The function also handles an encoded response that is passed to PostRun. If the response is not expected to be a stream, the function returns an error.


```
func finishCLIExport(res cmds.Response, re cmds.ResponseEmitter) error {
	var showProgress bool
	val, specified := res.Request().Options[progressOptionName]
	if !specified {
		// default based on TTY availability
		errStat, _ := os.Stderr.Stat()
		if (errStat.Mode() & os.ModeCharDevice) != 0 {
			showProgress = true
		}
	} else if val.(bool) {
		showProgress = true
	}

	// simple passthrough, no progress
	if !showProgress {
		return cmds.Copy(re, res)
	}

	bar := pb.New64(0).SetUnits(pb.U_BYTES)
	bar.Output = os.Stderr
	bar.ShowSpeed = true
	bar.ShowElapsedTime = true
	bar.RefreshRate = 500 * time.Millisecond
	bar.Start()

	var processedOneResponse bool
	for {
		v, err := res.Next()
		if err == io.EOF {

			// We only write the final bar update on success
			// On error it looks too weird
			bar.Finish()

			return re.Close()
		} else if err != nil {
			return re.CloseWithError(err)
		} else if processedOneResponse {
			return re.CloseWithError(errors.New("unexpected multipart response during emit, please file a bugreport"))
		}

		r, ok := v.(io.Reader)
		if !ok {
			// some sort of encoded response, this should not be happening
			return errors.New("unexpected non-stream passed to PostRun: please file a bugreport")
		}

		processedOneResponse = true

		if err := re.Emit(bar.NewProxyReader(r)); err != nil {
			return err
		}
	}
}

```

这段代码定义了一个名为 `dagStore` 的结构体，它包含一个名为 `dag` 的 `APIDagService` 类型字段和一个名为 `ctx` 的 `Context` 类型字段。

`dagStore` 结构体代表了一个可以操作分布式算法数据图形（DAG）的数据库。`dag` 字段是一个引用，指向一个实现了 `APIDagService` 接口的组件，这个组件可以用来进行分布式算法数据的拉取和更新。`ctx` 字段是一个指向 `Context` 类型的字段，用于在调用 `dag` 字段的方法时提供上下文。

`dagStore.Get` 方法使用 `dag` 字段的 `Get` 方法来获取指定 `ctx` 上下文中指定的 `Cid` 类型的数据。这个方法返回一个 `blocks.Block` 类型的数据，代表了一个数据块，以及一个可能的错误。


```
// FIXME(@Jorropo): https://github.com/ipld/go-car/issues/315
type dagStore struct {
	dag iface.APIDagService
	ctx context.Context
}

func (ds dagStore) Get(_ context.Context, c cid.Cid) (blocks.Block, error) {
	return ds.dag.Get(ds.ctx, c)
}

```

# `/opt/kubo/core/commands/dag/get.go`

该代码的作用是定义了一个名为 "dagcmd" 的包。这个包通过使用 "fmt"、"io"、"github.com/ipfs/boxo/path"、"github.com/ipfs/go-ipld-legacy"、"github.com/ipfs/kubo/core/commands/cmdenv"、"github.com/ipfs/kubo/core/commands/cmdutils"、"github.com/ipfs/kubo/core/commands/ipsf湿质"、"github.com/ipfs/kubo/core/params"、"github.com/ipfs/kubo/private/kthash/sortings" 和 "github.com/ipfs/kubo/private/kthash/tree" 这几个库来实现。

具体来说，这个包的作用是创建和操作 DAG（有向无环图）数据结构。它包括对 DAG 数据的读取、写入、修改等操作。通过使用 "ipldlegacy"、"github.com/ipfs/boxo/path" 和 "github.com/ipfs/kubo/core/commands/ipsf湿质" 等库，可以方便地处理 DAG 数据。


```
package dagcmd

import (
	"fmt"
	"io"

	"github.com/ipfs/boxo/path"
	ipldlegacy "github.com/ipfs/go-ipld-legacy"
	"github.com/ipfs/kubo/core/commands/cmdenv"
	"github.com/ipfs/kubo/core/commands/cmdutils"

	"github.com/ipld/go-ipld-prime"
	"github.com/ipld/go-ipld-prime/multicodec"
	"github.com/ipld/go-ipld-prime/traversal"
	mc "github.com/multiformats/go-multicodec"

	cmds "github.com/ipfs/go-ipfs-cmds"
)

```

This appears to be a function definition for an `ipldlegacy.MessageDS` interface.  It takes in a request object and an environment object, and returns a response object.

The function takes a single argument, which is a path to a CSV file.  It parses the path and returns a universal node representing the root of the IPLD tree rooted at that path.

If the universal node is not found, an error is thrown.

The function also defines a codec for encoding/decoding the universal node, which is specified by the `output-codec` option of the request.

The function is part of a library that provides a unified interface for working with IPLD (Inter-Platform Portable Document) data, such as晴空（zhangkong.example.com，开出枝）..


```
func dagGet(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
	api, err := cmdenv.GetApi(env, req)
	if err != nil {
		return err
	}

	codecStr, _ := req.Options["output-codec"].(string)
	var codec mc.Code
	if err := codec.Set(codecStr); err != nil {
		return err
	}

	p, err := cmdutils.PathOrCidPath(req.Arguments[0])
	if err != nil {
		return err
	}

	rp, remainder, err := api.ResolvePath(req.Context, p)
	if err != nil {
		return err
	}

	obj, err := api.Dag().Get(req.Context, rp.RootCid())
	if err != nil {
		return err
	}

	universal, ok := obj.(ipldlegacy.UniversalNode)
	if !ok {
		return fmt.Errorf("%T is not a valid IPLD node", obj)
	}

	finalNode := universal.(ipld.Node)

	if len(remainder) > 0 {
		remainderPath := ipld.ParsePath(path.SegmentsToString(remainder...))

		finalNode, err = traversal.Get(finalNode, remainderPath)
		if err != nil {
			return err
		}
	}

	encoder, err := multicodec.LookupEncoder(uint64(codec))
	if err != nil {
		return fmt.Errorf("invalid encoding: %s - %s", codec, err)
	}

	r, w := io.Pipe()
	go func() {
		defer w.Close()
		if err := encoder(finalNode, w); err != nil {
			_ = w.CloseWithError(err)
		}
	}()

	return res.Emit(r)
}

```

# `/opt/kubo/core/commands/dag/import.go`

该代码是一个 Go 语言编写的 DAG（有向无环图）命令行工具 "boxo" 的依赖关系文件。通过导入不同的库 "github.com/ipfs/boxo/coreiface/options" 和 "github.com/ipfs/boxo/files"，该代码定义了一个 DAG 对象模型的函数和变量。

具体来说，该代码实现了一个 DAG 模型的基本功能，包括：

1. 定义 DAG 对象的函数，如 `CreateDAG` 和 `GetDAG` 等。
2. 定义 DAG 对象的变量，如 `dag` 和 `腰带` 等。
3. 导入 DAG 文件的路径，通过 `options.Args` 参数实现。
4. 支持块文件的输入和输出，通过 `blocks` 和 `cid` 包的导入实现。
5. 支持 IPFS-CID（IPFS 对象索引编码）文件的输入和输出，通过 `ipld` 和 `ipldlegacy` 包的导入实现。
6. 支持 Car v2 格式的输入和输出，通过 `gocarv2` 包的导入实现。
7. 支持 Kubernetes（K8s）命令行界面（CMD）的输入和输出，通过 `cmdenv` 和 `cmdutils` 包的导入实现。

该代码是 `boxo` 命令行工具的主要依赖项之一，可以在不需要配置文件的情况下，对 DAG 对象执行各种操作。


```
package dagcmd

import (
	"errors"
	"fmt"
	"io"

	"github.com/ipfs/boxo/coreiface/options"
	"github.com/ipfs/boxo/files"
	blocks "github.com/ipfs/go-block-format"
	cid "github.com/ipfs/go-cid"
	cmds "github.com/ipfs/go-ipfs-cmds"
	ipld "github.com/ipfs/go-ipld-format"
	ipldlegacy "github.com/ipfs/go-ipld-legacy"
	gocarv2 "github.com/ipld/go-car/v2"

	"github.com/ipfs/kubo/core/commands/cmdenv"
	"github.com/ipfs/kubo/core/commands/cmdutils"
)

```

This is a function that attempts to pin a root block to a file in a car file repository. It uses the `rootmeta` package to get the root metadata for each block, which is then used to read the block's contents and apply the `pin` method from the `node.Pinning` struct.

The function takes an optional `doPinRoots` flag, which is a boolean value that is set to `true` to enable this feature. If this flag is set to `true`, the function will attempt to pin all blocks in the repository.

The function returns an error if any errors occur, or an error if the pinning operation fails. If the pinning operation was successful, the function returns no error. If the repository is too large, the function may also return a timeout error.

The function also includes some additional logging and error handling. If the repository is not well-formed, the function will attempt to notify the user. If there are any errors during the pinning process, the function will log the error and return an error.


```
func dagImport(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
	node, err := cmdenv.GetNode(env)
	if err != nil {
		return err
	}

	api, err := cmdenv.GetApi(env, req)
	if err != nil {
		return err
	}

	blockDecoder := ipldlegacy.NewDecoder()

	// on import ensure we do not reach out to the network for any reason
	// if a pin based on what is imported + what is in the blockstore
	// isn't possible: tough luck
	api, err = api.WithOptions(options.Api.Offline(true))
	if err != nil {
		return err
	}

	doPinRoots, _ := req.Options[pinRootsOptionName].(bool)

	// grab a pinlock ( which doubles as a GC lock ) so that regardless of the
	// size of the streamed-in cars nothing will disappear on us before we had
	// a chance to roots that may show up at the very end
	// This is especially important for use cases like dagger:
	//    ipfs dag import $( ... | ipfs-dagger --stdout=carfifos )
	//
	if doPinRoots {
		unlocker := node.Blockstore.PinLock(req.Context)
		defer unlocker.Unlock(req.Context)
	}

	// this is *not* a transaction
	// it is simply a way to relieve pressure on the blockstore
	// similar to pinner.Pin/pinner.Flush
	batch := ipld.NewBatch(req.Context, api.Dag())

	roots := cid.NewSet()
	var blockCount, blockBytesCount uint64

	// remember last valid block and provide a meaningful error message
	// when a truncated/mangled CAR is being imported
	importError := func(previous blocks.Block, current blocks.Block, err error) error {
		if current != nil {
			return fmt.Errorf("import failed at block %q: %w", current.Cid(), err)
		}
		if previous != nil {
			return fmt.Errorf("import failed after block %q: %w", previous.Cid(), err)
		}
		return fmt.Errorf("import failed: %w", err)
	}

	it := req.Files.Entries()
	for it.Next() {
		file := files.FileFromEntry(it)
		if file == nil {
			return errors.New("expected a file handle")
		}

		// import blocks
		err = func() error {
			// wrap a defer-closer-scope
			//
			// every single file in it() is already open before we start
			// just close here sooner rather than later for neatness
			// and to surface potential errors writing on closed fifos
			// this won't/can't help with not running out of handles
			defer file.Close()

			var previous blocks.Block

			car, err := gocarv2.NewBlockReader(file)
			if err != nil {
				return err
			}

			for _, c := range car.Roots {
				roots.Add(c)
			}

			for {
				block, err := car.Next()
				if err != nil && err != io.EOF {
					return importError(previous, block, err)
				} else if block == nil {
					break
				}
				if err := cmdutils.CheckBlockSize(req, uint64(len(block.RawData()))); err != nil {
					return importError(previous, block, err)
				}

				// the double-decode is suboptimal, but we need it for batching
				nd, err := blockDecoder.DecodeNode(req.Context, block)
				if err != nil {
					return importError(previous, block, err)
				}

				if err := batch.Add(req.Context, nd); err != nil {
					return importError(previous, block, err)
				}
				blockCount++
				blockBytesCount += uint64(len(block.RawData()))
				previous = block
			}
			return nil
		}()
		if err != nil {
			return err
		}
	}

	if err := batch.Commit(); err != nil {
		return err
	}

	// It is not guaranteed that a root in a header is actually present in the same ( or any )
	// .car file. This is the case in version 1, and ideally in further versions too.
	// Accumulate any root CID seen in a header, and supplement its actual node if/when encountered
	// We will attempt a pin *only* at the end in case all car files were well-formed.

	// opportunistic pinning: try whatever sticks
	if doPinRoots {
		err = roots.ForEach(func(c cid.Cid) error {
			ret := RootMeta{Cid: c}

			// This will trigger a full read of the DAG in the pinner, to make sure we have all blocks.
			// Ideally we would do colloring of the pinning state while importing the blocks
			// and ensure the gray bucket is empty at the end (or use the network to download missing blocks).
			if block, err := node.Blockstore.Get(req.Context, c); err != nil {
				ret.PinErrorMsg = err.Error()
			} else if nd, err := blockDecoder.DecodeNode(req.Context, block); err != nil {
				ret.PinErrorMsg = err.Error()
			} else if err := node.Pinning.Pin(req.Context, nd, true); err != nil {
				ret.PinErrorMsg = err.Error()
			} else if err := node.Pinning.Flush(req.Context); err != nil {
				ret.PinErrorMsg = err.Error()
			}

			return res.Emit(&CarImportOutput{Root: &ret})
		})
		if err != nil {
			return err
		}
	}

	stats, _ := req.Options[statsOptionName].(bool)
	if stats {
		err = res.Emit(&CarImportOutput{
			Stats: &CarImportStats{
				BlockCount:      blockCount,
				BlockBytesCount: blockBytesCount,
			},
		})
		if err != nil {
			return err
		}
	}

	return nil
}

```

# `/opt/kubo/core/commands/dag/put.go`

这段代码定义了一个名为 "dagcmd" 的包。它从以下依赖中导入了一些必要的库：

- "github.com/ipfs/go-block-format" 用于 DAG 数据的块格式支持
- "github.com/ipfs/go-cid" 用于 CID 格式的子元素引用支持
- "github.com/ipfs/go-ipld-legacy" 用于 IPLD 旧版格式的支持
- "github.com/ipfs/kubo/core/commands/cmdenv" 用于 Kubo 命令的 Core-DAG 环境
- "github.com/ipfs/kubo/core/commands/cmdutils" 用于命令的运行时参数处理
- "github.com/ipfs/go-ipld-prime/multicodec" 用于多链路 DAG 数据编码
- "github.com/ipfs/boxo/files" 用于文件系统操作
- "github.com/ipfs/go-ipfs-cmds" 用于 IPFS 命令行工具的创建
- "github.com/ipfs/go-ipld-format" 用于 IPLD 格式的支持
- "github.com/multiformats/go-multicodec" 用于多链路 DAG 数据编码

该代码还定义了一个名为 "basicnode" 的类，用于 IPLD 基本节点操作。

最后，该代码通过导入一些库，定义了一些常量和函数，用于在 IPFS 系统中执行 DAG 命令。


```
package dagcmd

import (
	"bytes"
	"fmt"

	blocks "github.com/ipfs/go-block-format"
	"github.com/ipfs/go-cid"
	ipldlegacy "github.com/ipfs/go-ipld-legacy"
	"github.com/ipfs/kubo/core/commands/cmdenv"
	"github.com/ipfs/kubo/core/commands/cmdutils"
	"github.com/ipld/go-ipld-prime/multicodec"
	basicnode "github.com/ipld/go-ipld-prime/node/basic"

	"github.com/ipfs/boxo/files"
	cmds "github.com/ipfs/go-ipfs-cmds"
	ipld "github.com/ipfs/go-ipld-format"
	mc "github.com/multiformats/go-multicodec"

	// Expected minimal set of available format/ienc codecs.
	_ "github.com/ipld/go-codec-dagpb"
	_ "github.com/ipld/go-ipld-prime/codec/cbor"
	_ "github.com/ipld/go-ipld-prime/codec/dagcbor"
	_ "github.com/ipld/go-ipld-prime/codec/dagjson"
	_ "github.com/ipld/go-ipld-prime/codec/json"
	_ "github.com/ipld/go-ipld-prime/codec/raw"
)

```

This appears to be a Go function that adds a file to a Build元旦离线 operation. It takes a single argument, which is an `ipld.NodeAdder` object, and optionally takes another argument, which is an API endpoint to use for named pipes.

The function reads the contents of the file and adds it to a `ipldlegacy.LegacyNode` object, which is then added to a batch using the `ipld.NodeAdder`. The batch is then committed to the API endpoint, and the function returns None if the operation was successful.

If an error occurs during the operation, such as a file not being found or an error in the encoder or blockCoder, the function returns it.


```
func dagPut(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
	api, err := cmdenv.GetApi(env, req)
	if err != nil {
		return err
	}

	inputCodec, _ := req.Options["input-codec"].(string)
	storeCodec, _ := req.Options["store-codec"].(string)
	hash, _ := req.Options["hash"].(string)
	dopin, _ := req.Options["pin"].(bool)

	var icodec mc.Code
	if err := icodec.Set(inputCodec); err != nil {
		return err
	}
	var scodec mc.Code
	if err := scodec.Set(storeCodec); err != nil {
		return err
	}
	var mhType mc.Code
	if err := mhType.Set(hash); err != nil {
		return err
	}

	cidPrefix := cid.Prefix{
		Version:  1,
		Codec:    uint64(scodec),
		MhType:   uint64(mhType),
		MhLength: -1,
	}

	decoder, err := multicodec.LookupDecoder(uint64(icodec))
	if err != nil {
		return err
	}
	encoder, err := multicodec.LookupEncoder(uint64(scodec))
	if err != nil {
		return err
	}

	var adder ipld.NodeAdder = api.Dag()
	if dopin {
		adder = api.Dag().Pinning()
	}
	b := ipld.NewBatch(req.Context, adder)

	it := req.Files.Entries()
	for it.Next() {
		file := files.FileFromEntry(it)
		if file == nil {
			return fmt.Errorf("expected a regular file")
		}

		node := basicnode.Prototype.Any.NewBuilder()
		if err := decoder(node, file); err != nil {
			return err
		}
		n := node.Build()

		bd := bytes.NewBuffer([]byte{})
		if err := encoder(n, bd); err != nil {
			return err
		}

		blockCid, err := cidPrefix.Sum(bd.Bytes())
		if err != nil {
			return err
		}
		blk, err := blocks.NewBlockWithCid(bd.Bytes(), blockCid)
		if err != nil {
			return err
		}
		ln := ipldlegacy.LegacyNode{
			Block: blk,
			Node:  n,
		}

		if err := cmdutils.CheckBlockSize(req, uint64(bd.Len())); err != nil {
			return err
		}

		if err := b.Add(req.Context, &ln); err != nil {
			return err
		}

		cid := ln.Cid()
		if err := res.Emit(&OutputObject{Cid: cid}); err != nil {
			return err
		}
	}
	if it.Err() != nil {
		return it.Err()
	}

	if err := b.Commit(); err != nil {
		return err
	}

	return nil
}

```

# `/opt/kubo/core/commands/dag/resolve.go`

这段代码定义了一个名为 `dagResolve` 的函数，属于名为 `dagcmd` 的包。函数接受一个名为 `req` 的 cmds.Request 对象，一个名为 `res` 的 cmds.ResponseEmitter 对象和一个名为 `env` 的 cmds.Environment 对象作为参数。函数的作用是：通过调用 api.ResolvePath 函数，解决指定路径的两大问题，然后返回调用结果。

函数的具体实现包括以下步骤：

1. 检查请求参数中的路径是否合法，如果错误则返回。
2. 如果已经成功调用 api.ResolvePath 函数，获取路径对象的根节点 ID。
3. 创建一个名为 `remainder` 的切片，用于存储路径中未解析的部分，如果出错则返回。
4. 调用 api.ResolvePath 函数，提供指定路径和剩余部分，并获取解决后的路径对象。
5. 创建一个名为 `resolvedPath` 的字符串切片，包含路径对象解析后的结果。
6. 使用 res.Once` 方法确保只调用一次函数内部的方法。
7. 返回调用结果，使用 res.Emit` 方法确保只调用一次函数内部的方法。


```
package dagcmd

import (
	"github.com/ipfs/boxo/path"
	"github.com/ipfs/kubo/core/commands/cmdenv"
	"github.com/ipfs/kubo/core/commands/cmdutils"

	cmds "github.com/ipfs/go-ipfs-cmds"
)

func dagResolve(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
	api, err := cmdenv.GetApi(env, req)
	if err != nil {
		return err
	}

	p, err := cmdutils.PathOrCidPath(req.Arguments[0])
	if err != nil {
		return err
	}

	rp, remainder, err := api.ResolvePath(req.Context, p)
	if err != nil {
		return err
	}

	return cmds.EmitOnce(res, &ResolveOutput{
		Cid:     rp.RootCid(),
		RemPath: path.SegmentsToString(remainder...),
	})
}

```

# `/opt/kubo/core/commands/dag/stat.go`

这段代码是一个 Go 语言编写的 DAG（有向无环图）命令行工具的源代码。具体来说，它实现了通过 DAG 数据结构对存储在本地文件夹中的数据进行树状结构化，并提供了命令行界面来展示和操作这些树状结构。

首先，它导入了必要的库，包括：

* `fmt`：用于输出格式化信息
* `io`：用于文件 I/O
* `os`：用于操作系统功能
* `mdag`：用于 DAG 树数据结构操作
* `traverse`：用于 DAG 树的遍历
* `cid`：用于创建和管理目录的 Go 库
* `cmds`：用于 Git 仓库的 CLI 工具
* `e`：用于 errors 的包

接下来，它定义了一些常量和函数：

* `version`：定义了工具的版本
* `分数线`：定义了 DAG 树中的分数线，用于显示 DAG 树中的路径
* `handleSample`：定义了一个函数，用于从本地文件夹中读取数据，并创建 DAG 树结构
* `buildDAG`：定义了一个函数，用于从本地文件夹中读取数据，并创建 DAG 树结构
* `listDAG`：定义了一个函数，用于从本地文件夹中读取数据，并创建 DAG 树结构
* `showDAG`：定义了一个函数，用于打印 DAG 树
* `handleCreate`：定义了一个函数，用于创建一个新的目录
* `handlePut`：定义了一个函数，用于将数据保存到目录中

最后，它实现了主要的命令行函数：

* `buildDAGInplace`：用于从本地文件夹中读取数据，并创建 DAG 树结构
* `buildDAGFromFile`：用于从用户提供的文件中读取数据，并创建 DAG 树结构
* `listDAGInplace`：用于从本地文件夹中读取数据，并创建 DAG 树结构
* `listDAGFromFile`：用于从用户提供的文件中读取数据，并创建 DAG 树结构
* `showDAGInplace`：用于从本地文件夹中读取数据，并创建 DAG 树结构
* `showDAGFromFile`：用于从用户提供的文件中读取数据，并创建 DAG 树结构
* `handleCreateDAG`：用于创建一个新的目录
* `handlePutDAG`：用于将数据保存到目录中
* `handleDeleteDAG`：用于删除指定的 DAG 树结构

通过这些函数，使用者可以对本地文件夹中的数据进行树状结构化，并从不同的路径引用这些数据。


```
package dagcmd

import (
	"fmt"
	"io"
	"os"

	mdag "github.com/ipfs/boxo/ipld/merkledag"
	"github.com/ipfs/boxo/ipld/merkledag/traverse"
	cid "github.com/ipfs/go-cid"
	cmds "github.com/ipfs/go-ipfs-cmds"
	"github.com/ipfs/kubo/core/commands/cmdenv"
	"github.com/ipfs/kubo/core/commands/cmdutils"
	"github.com/ipfs/kubo/core/commands/e"
)

```

This is a Go function that traverses a DAG and calculates the statistics of the DAG. The function takes a request object with a root DID and


```
// TODO cache every cid traversal in a dp cache
// if the cid exists in the cache, don't traverse it, and use the cached result
// to compute the new state

func dagStat(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
	progressive := req.Options[progressOptionName].(bool)
	api, err := cmdenv.GetApi(env, req)
	if err != nil {
		return err
	}
	nodeGetter := mdag.NewSession(req.Context, api.Dag())

	cidSet := cid.NewSet()
	dagStatSummary := &DagStatSummary{DagStatsArray: []*DagStat{}}
	for _, a := range req.Arguments {
		p, err := cmdutils.PathOrCidPath(a)
		if err != nil {
			return err
		}
		rp, remainder, err := api.ResolvePath(req.Context, p)
		if err != nil {
			return err
		}
		if len(remainder) > 0 {
			return fmt.Errorf("cannot return size for anything other than a DAG with a root CID")
		}

		obj, err := nodeGetter.Get(req.Context, rp.RootCid())
		if err != nil {
			return err
		}
		dagstats := &DagStat{Cid: rp.RootCid()}
		dagStatSummary.appendStats(dagstats)
		err = traverse.Traverse(obj, traverse.Options{
			DAG:   nodeGetter,
			Order: traverse.DFSPre,
			Func: func(current traverse.State) error {
				currentNodeSize := uint64(len(current.Node.RawData()))
				dagstats.Size += currentNodeSize
				dagstats.NumBlocks++
				if !cidSet.Has(current.Node.Cid()) {
					dagStatSummary.incrementTotalSize(currentNodeSize)
				}
				dagStatSummary.incrementRedundantSize(currentNodeSize)
				cidSet.Add(current.Node.Cid())
				if progressive {
					if err := res.Emit(dagStatSummary); err != nil {
						return err
					}
				}
				return nil
			},
			ErrFunc:        nil,
			SkipDuplicates: true,
		})
		if err != nil {
			return fmt.Errorf("error traversing DAG: %w", err)
		}
	}

	dagStatSummary.UniqueBlocks = cidSet.Len()
	dagStatSummary.calculateSummary()

	if err := res.Emit(dagStatSummary); err != nil {
		return err
	}
	return nil
}

```

此代码定义了一个名为 finishCLIStat 的函数，接收两个参数，一个是结果输出类 cmds.Response，另一个是输出生成器类 cmds.ResponseEmitter。函数的作用是输出 DagStatSummary 类型的数据。

函数内部，首先定义了一个名为 dagStats 的 pointer，类型为 *DagStatSummary。然后使用一个 for 循环，遍历结果输出类 res 的 Next() 方法返回的结果。在循环中，首先检查结果是否为错误，若为错误则返回。若结果为成功，则根据结果值类型进行判断，若为 *DagStatSummary，则将 dagStats 指向结果，并输出相关信息；若不是，则返回错误。

在循环体内部，定义了一个 switch 语句，根据输入结果的类型进行判断。若结果为 *DagStatSummary，则输出相关信息；若不是，则返回错误。在循环结束后，函数使用 responseEmitter 参数的 Emit() 方法将 dagStats 输出。


```
func finishCLIStat(res cmds.Response, re cmds.ResponseEmitter) error {
	var dagStats *DagStatSummary
	for {
		v, err := res.Next()
		if err != nil {
			if err == io.EOF {
				break
			}
			return err
		}
		switch out := v.(type) {
		case *DagStatSummary:
			dagStats = out
			if dagStats.Ratio == 0 {
				length := len(dagStats.DagStatsArray)
				if length > 0 {
					currentStat := dagStats.DagStatsArray[length-1]
					fmt.Fprintf(os.Stderr, "CID: %s, Size: %d, NumBlocks: %d\n", currentStat.Cid, currentStat.Size, currentStat.NumBlocks)
				}
			}
		default:
			return e.TypeErr(out, v)

		}
	}
	return re.Emit(dagStats)
}

```

# `/opt/kubo/core/commands/e/error.go`

这段代码定义了一个名为“e”的包，其中包含一个名为“TypeErr”的函数，以及一个名为“_”的变量。

函数“TypeErr”接受两个参数，一个是 expected 类型，另一个是实际的类型。函数返回一个带着错误信息的字符串，其中包含两个错误类型：expected 和 actual。

变量“_”是一个被声明为“nil”的变量，但没有初始化。

此外，函数“compile time type check that HandlerError is an error”对HandlerError类型进行类型检查，确保该类型实现了Error接口。这个函数创建了一个名为“HandlerError”的类型，如果该类型实现了Error接口，那么创建的实例就是一个实现了该接口的错误对象。


```
package e

import (
	"fmt"
	"runtime/debug"
)

// TypeErr returns an error with a string that explains what error was expected and what was received.
func TypeErr(expected, actual interface{}) error {
	return fmt.Errorf("expected type %T, got %T", expected, actual)
}

// compile time type check that HandlerError is an error
var _ error = New(nil)

```

这段代码定义了一个名为 HandlerError 的结构体，该结构体包含一个名为 Err 的错误和一个名为 Stack 的 Stack 跟踪。

接着，该代码定义了一个名为 Error 的函数，该函数将返回一个 HandlerError 类型的错误对象，使用了 %v 形式的占位符来格式化错误信息，即 "%s in: %s"。

然后，该代码定义了一个名为 New 的函数，该函数接收一个名为 err 的错误对象，并返回一个新的 HandlerError 类型的对象，其中错误信息从 err.Err 和 Stack 跟踪中提取出来，并使用 debug.Stack() 函数获取 Stack 跟踪。

最后，该代码没有定义其他的函数或变量，但是使用了 HandlerError 的定义，因此在输出堆栈跟踪时会自动添加错误信息。


```
// HandlerError adds a stack trace to an error
type HandlerError struct {
	Err   error
	Stack []byte
}

// Error makes HandlerError implement error
func (err HandlerError) Error() string {
	return fmt.Sprintf("%s in:\n%s", err.Err.Error(), err.Stack)
}

// New returns a new HandlerError
func New(err error) HandlerError {
	return HandlerError{Err: err, Stack: debug.Stack()}
}

```

# `/opt/kubo/core/commands/keyencode/keyencode.go`

这段代码定义了一个名为“keyencode”的包，并导入了三个依赖项：

1. “github.com/ipfs/go-ipfs-cmds” 库，用于在 IPFS 客户端中执行命令行操作。
2. “github.com/libp2p/go-libp2p/core/peer” 库，用于与 IPFS 网络中的节点通信。
3. “github.com/multiformats/go-multibase” 库，用于在 IPFS 或其他数据系统中提供跨链的 JSON-API。

接下来，定义了一个名为“KeyEncoder”的结构体，该结构体包含一个“baseEnc”字段，该字段是一个“mbase.Encoder”类型的变量。

然后，通过创建一个名为“OptionIPNSBase”的命令行选项，指定了一个名为“ipns-base”的选项，并使用字符串格式 "{b58mh|base36|k|base32|b...}" 来描述该选项的值。该选项是多字节编码的 CID 或 base58btc 编码的多哈希。设置默认值为 “base36”。

最后，定义了一个“KeyEncoder”结构体，该结构体将“baseEnc”字段以及一个名为“peer.Stream”的字段作为其依赖项。


```
package keyencode

import (
	cmds "github.com/ipfs/go-ipfs-cmds"
	peer "github.com/libp2p/go-libp2p/core/peer"
	mbase "github.com/multiformats/go-multibase"
)

const ipnsKeyFormatOptionName = "ipns-base"

var OptionIPNSBase = cmds.StringOption(ipnsKeyFormatOptionName, "Encoding used for keys: Can either be a multibase encoded CID or a base58btc encoded multihash. Takes {b58mh|base36|k|base32|b...}.").WithDefault("base36")

type KeyEncoder struct {
	baseEnc *mbase.Encoder
}

```

该函数`KeyEncoderFromString`的作用是从给定的字符串`formatLabel`中解析出`KeyEncoder`类型，并返回其值。如果字符串`formatLabel`无法转换为有效的`KeyEncoder`类型，函数将返回一个`nil`值。

具体实现中，函数首先定义了一个`switch`语句，用于根据给定的`formatLabel`分支出不同的处理逻辑。如果分支出的处理逻辑出现错误，函数将返回一个`nil`值。否则，如果存在一个可以解析为`KeyEncoder`的`enc`变量和一个有效的`baseEnc`变量，函数将返回一个`KeyEncoder`实例，并将其`baseEnc`字段赋值为`enc`，此时函数返回的`KeyEncoder`实例包含已经解析好的`KeyEncoder`类型及其`baseEnc`字段。

函数`(KeyEncoder) FormatID`的作用是将一个`peer.ID`对象`id`转换为字符串形式，并将转换后的字符串返回。函数首先检查给定的`enc`字段是否为`nil`，如果是，函数将直接返回`id`的`String()`方法返回的`string`。否则，函数将尝试使用`peer.ToCid()`函数将`ID`对象转换为`ToCid()`函数返回的`ToCid`对象，并使用`ToCid().StringOfBase()`函数将`ToCid`对象的字符串编码为`baseEnc`字段中的字符串。此时，如果编码过程中出现错误，函数将返回一个`err`。否则，函数将返回`id`的`String()`方法返回的`string`。


```
func KeyEncoderFromString(formatLabel string) (KeyEncoder, error) {
	switch formatLabel {
	case "b58mh", "v0":
		return KeyEncoder{}, nil
	default:
		if enc, err := mbase.EncoderByName(formatLabel); err != nil {
			return KeyEncoder{}, err
		} else {
			return KeyEncoder{&enc}, nil
		}
	}
}

func (enc KeyEncoder) FormatID(id peer.ID) string {
	if enc.baseEnc == nil {
		return id.String()
	}
	if s, err := peer.ToCid(id).StringOfBase(enc.baseEnc.Encoding()); err != nil {
		panic(err)
	} else {
		return s
	}
}

```

# `/opt/kubo/core/commands/name/ipns.go`

这段代码是一个 Go 语言package，它定义了一系列用于从 IPFS 存储桶中读取和写入文件以及设置目录的命令。它主要作用于在 IPFS 存储桶中进行文件操作。

具体来说，这段代码实现了以下功能：

1. 定义了两个名为 "options" 和 "namesys" 的选项类，用于设置和改进 IPFS 存储桶的选项。
2. 引入了 "errors"、"fmt"、"io"、"strings" 和 "time" 等标准库。
3. 实现了从 IPFS 存储桶中读取和写入文件以及设置目录的命令。具体来说，实现了以下命令：
	* `boxo i`：从 IPFS 存储桶中读取目录 "i" 中的所有文件。
	* `boxo o`：从 IPFS 存储桶中读取目录 "o" 中的所有文件。
	* `boxo r`：从 IPFS 存储桶中读取目录 "r" 中的所有文件。
	* `boxo w`：从 IPFS 存储桶中写入目录 "w" 中的所有文件。
	* `boxo top`：从 IPFS 存储桶中读取目录 "top" 中的所有文件。
	* `boxo b`：从 IPFS 存储桶中读取目录 "b" 中的所有文件。
	* `boxo t`：从 IPFS 存储桶中读取目录 "t" 中的所有文件。
	* `boxo new`：创建一个新目录。
	* `boxo edit`：编辑一个指定目录。
	* `boxo ls`：列出指定目录中的所有文件和子目录。
4. 设置了一个名为 "logging" 的选项类，用于记录操作日志。
5. 设置了一个名为 "cmdenv" 的命令类，用于管理 IPFS 存储桶的连接和配置。


```
package name

import (
	"errors"
	"fmt"
	"io"
	"strings"
	"time"

	options "github.com/ipfs/boxo/coreiface/options"
	"github.com/ipfs/boxo/namesys"
	"github.com/ipfs/boxo/path"
	cmds "github.com/ipfs/go-ipfs-cmds"
	logging "github.com/ipfs/go-log"
	cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"
)

```

此代码的作用是创建一个名为 "core/commands/ipns" 的 logging logger，用于输出在 ipns 命令行工具中发生的命令行日志。

它定义了一个名为 ResolvedPath 的结构体，其中包含用于 ipns 命令行工具查找名称的路径。

它还定义了几个常量，包括 recursiveOptionName、nocacheOptionName、dhtRecordCountOptionName 和 dhtTimeoutOptionName，这些常量用于指定 ipns 命令行工具是使用递归方式还是直接访问 DNS 服务器。

最后，它创建了一个名为 IpnsCmd 的结构体，该结构体代表 ipns 命令行工具的上下文，包括命令行帮助文本、短描述和内置命令。


```
var log = logging.Logger("core/commands/ipns")

type ResolvedPath struct {
	Path string
}

const (
	recursiveOptionName      = "recursive"
	nocacheOptionName        = "nocache"
	dhtRecordCountOptionName = "dht-record-count"
	dhtTimeoutOptionName     = "dht-timeout"
	streamOptionName         = "stream"
)

var IpnsCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Resolve IPNS names.",
		ShortDescription: `
```

这段代码定义了一个名为IPNS的PKI命名空间，其中名称是公共键的哈希值。私有密钥允许发布新的（签名）值。在发布和解析中，使用默认名称是节点自身的PeerID，这是节点公钥的哈希值。

此代码的作用是定义了一个用于存储和发布IPNS命名空间中的公共钥哈希值的命名空间。它还允许使用'ipfs key'命令来列出并生成更多名称及其对应的密钥。


```
IPNS is a PKI namespace, where names are the hashes of public keys, and
the private key enables publishing new (signed) values. In both publish
and resolve, the default name used is the node's own PeerID,
which is the hash of its public key.
`,
		LongDescription: `
IPNS is a PKI namespace, where names are the hashes of public keys, and
the private key enables publishing new (signed) values. In both publish
and resolve, the default name used is the node's own PeerID,
which is the hash of its public key.

You can use the 'ipfs key' commands to list and generate more names and their
respective keys.

Examples:

```

这段代码是一个基于 IPFS(InterPlanetary File System)的命令行工具，用于在 IPFS 系统中查找并返回指定名称的值。具体来说，它具有以下功能：

1. 解决名字的值：

  > ipfs name resolve：将输入名字作为参数，输出该名字对应的 ISDR(InterPlanetary Document Selector) 值，用于标识该名字在 IPFS 系统中的位置。

2. 解决另一个名字的值：

  > ipfs name resolve QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ：与上述类似，将输入名字转换为 ISDR 值，用于标识该名字在 IPFS 系统中的位置。

3. 解决dnslink：

  > ipfs name resolve ipfs.io：将输入dnslink（如 "ipfs.io"）转换为 ISDR 值，用于标识该dnslink在 IPFS 系统中的位置。


```
Resolve the value of your name:

  > ipfs name resolve
  /ipfs/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy

Resolve the value of another name:

  > ipfs name resolve QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ
  /ipfs/QmSiTko9JZyabH56y2fussEt1A5oDqsFXB3CkvAqraFryz

Resolve the value of a dnslink:

  > ipfs name resolve ipfs.io
  /ipfs/QmaBvfZooxWkrv7D3r8LS9moNjzD2o525XMZze69hhoxf5

```

This is a Go function that resolves a DHT timeout value by naming it after a device, such as an "ipns" device. It takes an optional "recursive" parameter, which is a boolean value that indicates whether the function should continue to resolve the name by递归查找子节点，而不是一旦达到深度限制就停止递归。 It also takes an optional "namesys.ResolveWithDhtTimeout" option, which is a method that the "ipns" device uses to resolve a name.

The function first checks if the given name is already a valid IPCN device name. If it is not, it attempts to resolve the name using the "ipns.ResolveWithDhtTimeout" option, and then follows the IPCN path to the device. If the device is not found, the function returns an error.

The function also resolves a short path to the device by名称， by appending a "/ipns/" suffix to the given name.

Finally, the function emits any changes to the output and returns an error if any errors occur during the resolution process.

The function uses the "cmds.EmitOnce" method to handle the output of the "api.Name().Search" method, which returns a slice of resolve entries. The function iterates over the entries and emits each one using "cmds.ResolvedPath.Emit" method.


```
`,
	},

	Arguments: []cmds.Argument{
		cmds.StringArg("name", false, false, "The IPNS name to resolve. Defaults to your node's peerID."),
	},
	Options: []cmds.Option{
		cmds.BoolOption(recursiveOptionName, "r", "Resolve until the result is not an IPNS name.").WithDefault(true),
		cmds.BoolOption(nocacheOptionName, "n", "Do not use cached entries."),
		cmds.UintOption(dhtRecordCountOptionName, "dhtrc", "Number of records to request for DHT resolution.").WithDefault(uint(namesys.DefaultResolverDhtRecordCount)),
		cmds.StringOption(dhtTimeoutOptionName, "dhtt", "Max time to collect values during DHT resolution eg \"30s\". Pass 0 for no timeout.").WithDefault(namesys.DefaultResolverDhtTimeout.String()),
		cmds.BoolOption(streamOptionName, "s", "Stream entries as they are found."),
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		api, err := cmdenv.GetApi(env, req)
		if err != nil {
			return err
		}

		nocache, _ := req.Options["nocache"].(bool)

		var name string
		if len(req.Arguments) == 0 {
			self, err := api.Key().Self(req.Context)
			if err != nil {
				return err
			}
			name = self.ID().String()
		} else {
			name = req.Arguments[0]
		}

		recursive, _ := req.Options[recursiveOptionName].(bool)
		rc, rcok := req.Options[dhtRecordCountOptionName].(uint)
		dhtt, dhttok := req.Options[dhtTimeoutOptionName].(string)
		stream, _ := req.Options[streamOptionName].(bool)

		opts := []options.NameResolveOption{
			options.Name.Cache(!nocache),
		}

		if !recursive {
			opts = append(opts, options.Name.ResolveOption(namesys.ResolveWithDepth(1)))
		}
		if rcok {
			opts = append(opts, options.Name.ResolveOption(namesys.ResolveWithDhtRecordCount(rc)))
		}
		if dhttok {
			d, err := time.ParseDuration(dhtt)
			if err != nil {
				return err
			}
			if d < 0 {
				return errors.New("DHT timeout value must be >= 0")
			}
			opts = append(opts, options.Name.ResolveOption(namesys.ResolveWithDhtTimeout(d)))
		}

		if !strings.HasPrefix(name, "/ipns/") {
			name = "/ipns/" + name
		}

		if !stream {
			output, err := api.Name().Resolve(req.Context, name, opts...)
			if err != nil && (recursive || err != namesys.ErrResolveRecursion) {
				return err
			}

			pth, err := path.NewPath(output.String())
			if err != nil {
				return err
			}

			return cmds.EmitOnce(res, &ResolvedPath{pth.String()})
		}

		output, err := api.Name().Search(req.Context, name, opts...)
		if err != nil {
			return err
		}

		for v := range output {
			if v.Err != nil && (recursive || v.Err != namesys.ErrResolveRecursion) {
				return v.Err
			}
			if err := res.Emit(&ResolvedPath{v.Path.String()}); err != nil {
				return err
			}

		}

		return nil
	},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, rp *ResolvedPath) error {
			_, err := fmt.Fprintln(w, rp.Path)
			return err
		}),
	},
	Type: ResolvedPath{},
}

```

# `/opt/kubo/core/commands/name/ipnsps.go`

这段代码定义了一个名为“ipnsPubsubState”的结构体，表示 IPNS 发布/订阅状态。它可能用于表示一些关于 IPNS 发布/订阅操作的信息。


package name

import (
	"fmt"
	"io"
	"strings"

	cmds "github.com/ipfs/go-ipfs-cmds"
	"github.com/ipfs/kubo/core/commands/cmdenv"
	ke "github.com/ipfs/kubo/core/commands/keyencode"
	record "github.com/libp2p/go-libp2p-record"
	"github.com/libp2p/go-libp2p/core/peer"
)


它导入了几个必要的库，包括 `fmt`、`io`、`strings` 和 `ke`、`record` 和 `cmds`。


type ipnsPubsubState struct {
	Enabled bool
}


这个结构体定义了一个名为 `ipnsPubsubState` 的变量，它是一个布尔值，表示是否启用 IPNS 发布/订阅。

fmt.Println(ipnsPubsubState)


这个函数用于输出 `ipnsPubsubState` 的值。

ipnsPubsubState := true


注意：在运行此代码之前，请确保您的 `libp2p` 安装正确。



```
package name

import (
	"fmt"
	"io"
	"strings"

	cmds "github.com/ipfs/go-ipfs-cmds"
	"github.com/ipfs/kubo/core/commands/cmdenv"
	ke "github.com/ipfs/kubo/core/commands/keyencode"
	record "github.com/libp2p/go-libp2p-record"
	"github.com/libp2p/go-libp2p/core/peer"
)

type ipnsPubsubState struct {
	Enabled bool
}

```

此代码定义了两个数据结构类型：ipnsPubsubCancel和stringList。

ipnsPubsubCancel表示一个用于取消IPNS pubsub订阅关系的布尔值类型。

stringList表示一个字符串列表类型，可以存储多个字符串。

接下来的行定义了一个名为"IpnsPubsubCmd"的命令类型变量，它是一个具有实验性status的cmds.Command对象。该命令用于管理IPNS pubsub系统，并输出帮助文本"IPNS pubsub management"，描述了如何使用该命令。


```
type ipnsPubsubCancel struct {
	Canceled bool
}

type stringList struct {
	Strings []string
}

// IpnsPubsubCmd is the subcommand that allows us to manage the IPNS pubsub system
var IpnsPubsubCmd = &cmds.Command{
	Status: cmds.Experimental,
	Helptext: cmds.HelpText{
		Tagline: "IPNS pubsub management",
		ShortDescription: `
Manage and inspect the state of the IPNS pubsub resolver.

```

这段代码是一个命令行工具的配置文件，其中包含了该工具的所有可用命令和其作用。

`Note: this command is experimental and subject to change as the system is refined` 是该命令的说明，告知开发人员该命令是实验性的，可能会在未来进行更改。

`",
	},
	Subcommands: map[string]*cmds.Command{
		"state":  ipnspsStateCmd,
		"subs":   ipnspsSubsCmd,
		"cancel": ipnspsCancelCmd,
	},
}`

该配置文件定义了一个名为 "ipnsps" 的命令行工具，它可以用于查询 IPNS 发布/订阅的状态。

`var ipnspsStateCmd = &cmds.Command{
	Status: cmds.Experimental,
	Helptext: cmds.HelpText{
		Tagline: "Query the state of IPNS pubsub.",
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		n, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}

		return cmds.EmitOnce(res, &ipnsPubsubState{n.PSRouter != nil})
	},
	Type: ipnsPubsubState{},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, ips *ipnsPubsubState) error {
			var state string
			if ips.Enabled {
				state = "enabled"
			} else {
				state = "disabled"
			}

			_, err := fmt.Fprintln(w, state)
			return err
		}),
	},
}`

`ipnspsStateCmd` 是一个实现了 `ipnsps` 命令的 `ipnsps.Command` 实例。它的 `Run` 函数用于实际运行命令，如果出现错误，将返回一个 `ipnsps.ResponseEmitter`。该 `Command` 的 `Status` 字段指定了该命令的实验性状态，`Helptext` 字段定义了该命令的可用帮助文本，`Type` 字段指定了命令的数据类型，`Encoders` 字段定义了命令可以使用的编码器。


```
Note: this command is experimental and subject to change as the system is refined
`,
	},
	Subcommands: map[string]*cmds.Command{
		"state":  ipnspsStateCmd,
		"subs":   ipnspsSubsCmd,
		"cancel": ipnspsCancelCmd,
	},
}

var ipnspsStateCmd = &cmds.Command{
	Status: cmds.Experimental,
	Helptext: cmds.HelpText{
		Tagline: "Query the state of IPNS pubsub.",
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		n, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}

		return cmds.EmitOnce(res, &ipnsPubsubState{n.PSRouter != nil})
	},
	Type: ipnsPubsubState{},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, ips *ipnsPubsubState) error {
			var state string
			if ips.Enabled {
				state = "enabled"
			} else {
				state = "disabled"
			}

			_, err := fmt.Fprintln(w, state)
			return err
		}),
	},
}

```

该代码定义了一个名为`ipnspsSubsCmd`的命令对象，属于`cmds.Command`类型。它的状态为`cmds.Experimental`，表示这是一个实验性的命令。

该命令的帮助文本为"Show current name subscriptions."，即显示当前名称订阅者。

该命令有一个`ke.OptionIPNSBase`选项，用于指定与IPNS基名称相关的键。

该命令的一个`Run`函数用于处理请求并返回响应。函数的参数包括：

- `req`：请求对象
- `res`：响应对象
- `env`：上下文对象

函数首先尝试从请求选项中从命名键中解析`ke.OptionIPNSBase.Name()`，然后尝试从PSRouter中获取订阅者列表。如果失败，函数将返回错误信息。

如果PSRouter中存在订阅者，函数将返回一个字符串列表，其中包含每个订阅者的路径。


```
var ipnspsSubsCmd = &cmds.Command{
	Status: cmds.Experimental,
	Helptext: cmds.HelpText{
		Tagline: "Show current name subscriptions.",
	},
	Options: []cmds.Option{
		ke.OptionIPNSBase,
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		keyEnc, err := ke.KeyEncoderFromString(req.Options[ke.OptionIPNSBase.Name()].(string))
		if err != nil {
			return err
		}

		n, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}

		if n.PSRouter == nil {
			return cmds.Errorf(cmds.ErrClient, "IPNS pubsub subsystem is not enabled")
		}
		var paths []string
		for _, key := range n.PSRouter.GetSubscriptions() {
			ns, k, err := record.SplitKey(key)
			if err != nil || ns != "ipns" {
				// Not necessarily an error.
				continue
			}
			pid, err := peer.IDFromBytes([]byte(k))
			if err != nil {
				log.Errorf("ipns key not a valid peer ID: %s", err)
				continue
			}
			paths = append(paths, "/ipns/"+keyEnc.FormatID(pid))
		}

		return cmds.EmitOnce(res, &stringList{paths})
	},
	Type: stringList{},
	Encoders: cmds.EncoderMap{
		cmds.Text: stringListEncoder(),
	},
}

```

This is a command line interface (CLI) for a NodeJS program that allows you to cancel a name subscription in the IPNs (Inter-Platform纳曾是） PubSub subsystem.

The `ipnsPubsubCancel` struct represents the CLI command. It has a `status` field of `cmds.Experimental`, a `helptext` field that displays a tooltip with the description of the command, and a `run` field that contains the command's code.

The `run` field has a function that takes a `cmds.Request` object and a `cmds.ResponseEmitter` object, as well as an environment object provided by the `cmds.Environment` type. The function starts by getting a node by its PSRouter and then checks if the PSRouter is not nil. If it is not, the function returns an error. If the PSRouter is not a psutil.Listeners.PSReader or is a psutil.Listeners.PSWriter, the function will return an error.

The function then takes the name of the subscription to cancel as an argument and removes the "/ipns/<subscription>" prefix from it. It then attempts to cancel the subscription by calling the `Cancel()` method on the PSRouter object with the given subscription name.

If the cancellation was successful, the response emitter will return an `ipnsPubsubCancel` object with a `ok` field.


```
var ipnspsCancelCmd = &cmds.Command{
	Status: cmds.Experimental,
	Helptext: cmds.HelpText{
		Tagline: "Cancel a name subscription.",
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		n, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}

		if n.PSRouter == nil {
			return cmds.Errorf(cmds.ErrClient, "IPNS pubsub subsystem is not enabled")
		}

		name := req.Arguments[0]
		name = strings.TrimPrefix(name, "/ipns/")
		pid, err := peer.Decode(name)
		if err != nil {
			return cmds.Errorf(cmds.ErrClient, err.Error())
		}

		ok, err := n.PSRouter.Cancel("/ipns/" + string(pid))
		if err != nil {
			return err
		}
		return cmds.EmitOnce(res, &ipnsPubsubCancel{ok})
	},
	Arguments: []cmds.Argument{
		cmds.StringArg("name", true, false, "Name to cancel the subscription for."),
	},
	Type: ipnsPubsubCancel{},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, ipc *ipnsPubsubCancel) error {
			var state string
			if ipc.Canceled {
				state = "canceled"
			} else {
				state = "no subscription"
			}

			_, err := fmt.Fprintln(w, state)
			return err
		}),
	},
}

```

这段代码定义了一个名为`stringListEncoder`的函数，它接收一个名为`req`的请求参数和一个名为`w`的输出写入缓冲区`<- 这是万里合约中的一个特殊构造函数，代表一个万里合约的请求，同时提供了写入数据到给定缓冲区的接口。另一个参数`list`代表一个字符串列表`<stringList>`，可能是请求中传递给万里合约的一个或多个字符串参数。函数返回一个没有输出错误并且返回`nil`的`<go-cmp>`值，表示没有返回任何值。

函数体首先定义了一个内部函数`func(req *cmds.Request, w io.Writer, list *stringList) error`，这个内部函数接收一个`<cmds.Request>`类型的参数`req`，一个`<io.Writer>`类型的参数`w`和一个`<stringList>`类型的参数`list`。这个内部函数通过一个循环遍历`list`中的字符串，将每个字符串写入`w`缓冲区中。在写入数据之前，函数检查写入缓冲区操作是否成功。如果写入操作出现错误，函数返回一个非`nil`的`<go-cmp>`值，表示函数返回一个非`nil`的错误值。如果写入操作成功，函数返回`nil`。

然后，在`stringListEncoder`函数体中，定义了一个接收一个`<cmds.Request>`类型的参数`req`和一个`<go.StringList>`类型的参数`list`的内部函数`func(req *cmds.Request, w io.Writer, list *stringList) error`。这个内部函数与上面定义的内部函数`func(req *cmds.Request, w io.Writer, list *stringList) error`合并，共同构成了万里合约的`stringListEncoder`函数。

`stringListEncoder`函数返回一个没有输出错误并且返回`nil`的`<go-cmp>`值，表示函数没有返回任何值。


```
func stringListEncoder() cmds.EncoderFunc {
	return cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, list *stringList) error {
		for _, s := range list.Strings {
			_, err := fmt.Fprintln(w, s)
			if err != nil {
				return err
			}
		}

		return nil
	})
}

```