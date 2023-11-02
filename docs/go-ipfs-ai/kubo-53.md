# go-ipfs 源码解析 53

# `routing/error.go`

这段代码定义了一个名为 "routing" 的包，其中包含了一些用于创建动态路由参数的函数和结构体。

具体来说，这段代码实现了一个名为 "NewParamNeededError" 的函数，它接收两个参数：一个字符串参数 "param" 和一个名为 "routing" 的路由器类型参数。函数返回一个 "ParamNeededError" 类型的结构体，其中包含 "param" 和 "routing" 字段，分别表示错误参数名称和错误路由器类型。

此外，这段代码还定义了一个名为 "fmt" 的包，其中包含了一项fmt函数，用于将错误消息打印为字符串格式。


```go
package routing

import (
	"fmt"

	"github.com/ipfs/kubo/config"
)

type ParamNeededError struct {
	ParamName  string
	RouterType config.RouterType
}

func NewParamNeededErr(param string, routing config.RouterType) error {
	return &ParamNeededError{
		ParamName:  param,
		RouterType: routing,
	}
}

```

该函数名为`func (e *ParamNeededError) Error() string`，接受一个名为`e`的参数，并返回一个字符串类型的错误信息。

函数体中，首先通过`fmt.Sprintf`函数创建一个字符串切片，其中包含两个格式化字符串："configuration param '%v' is needed for %v delegated routing types"，`%v`表示要格式化的变量，`delegated routing types`表示路由类型别名，替换为具体的参数名。第一个格式化字符串`configuration param '%v' is needed for %v delegated routing types`中，`configuration param '%v'`表示需要解析的参数名，`is needed for %v`表示需要参数的路由类型，`delegated routing types`表示需要使用代理路由类型的路由类型。而第二个格式化字符串`fmt.Sprintf("configuration param '%v' is needed for %v delegated routing types", e.ParamName, e.RouterType)`，则直接使用`%v`和`%v`来表示需要解析的参数名和路由类型。

最后，将创建好的格式化字符串返回，并使用`e.Error()`方法将其返回的字符串作为参数传入，返回新的错误对象，即`e.Error()`的返回值。


```go
func (e *ParamNeededError) Error() string {
	return fmt.Sprintf("configuration param '%v' is needed for %v delegated routing types", e.ParamName, e.RouterType)
}

```

# `routing/wrapper.go`

这段代码定义了一个名为“routing”的包，其中包含了一个名为“ProvideManyRouter”的接口，以及一个实现了“ProvideManyRouter”接口的名为“httpRoutingWrapper”的类。

具体来说，这段代码以下面几种方式实现了“ProvideManyRouter”接口：

1. 定义了一个名为“_”的变量，其值为实现了“ProvideManyRouter”接口的类型的别名，即“&httpRoutingWrapper{}”。
2. 定义了一个名为“_”的变量，其值为实现了“ProvideManyRouter”接口的类型的别名，即“&httpRoutingWrapper{}”。
3. 定义了一个名为“_”的变量，其值为实现了“ProvideManyRouter”接口的类型的别名，即“&httpRoutingWrapper{}”。
4. 定义了一个名为“_”的变量，其值为实现了“ProvideManyRouter”接口的类型的别名，即“&httpRoutingWrapper{}”。
5. 定义了一个名为“routing”的函数，其接收一个实现了“ProvideManyRouter”接口的类型的参数，并返回一个实现了“ProvideManyRouter”接口的类型的实例。
6. 定义了一个名为“httpRoutingWrapper”的函数，其接收一个实现“ProvideManyRouter”接口的类型的参数，并返回一个实现了“ProvideManyRouter”接口的类型的实例。

这段代码定义了一个名为“routing”的包，其中包含了一个名为“ProvideManyRouter”的接口，以及多个名为“httpRoutingWrapper”的类，这些类实现了“ProvideManyRouter”接口。


```go
package routing

import (
	"context"

	routinghelpers "github.com/libp2p/go-libp2p-routing-helpers"
	"github.com/libp2p/go-libp2p/core/routing"
)

type ProvideManyRouter interface {
	routinghelpers.ProvideManyRouter
	routing.Routing
}

var (
	_ routing.Routing                  = &httpRoutingWrapper{}
	_ routinghelpers.ProvideManyRouter = &httpRoutingWrapper{}
)

```

这段代码定义了一个名为httpRoutingWrapper的结构体，它是一个用于构建路由的wrapper，通过它，可以实现将http的delegated路由包装到一起的功能。

具体来说，它包含以下字段：

* routing.ContentRouting：表示内容路由，用于将请求和响应之间的内容相关路由。
* routing.PeerRouting：表示对等路由，用于将请求和响应之间的对等关系路由。
* routing.ValueStore：表示将路由信息存储到上下文中的功能。
* routinghelpers.ProvideManyRouter：表示提供多个路由器的功能。

另外，还包含一个名为Bootstrap的函数，用于启动整个路由器链的加载过程。


```go
// httpRoutingWrapper is a wrapper needed to construct the routing.Routing interface from
// http delegated routing.
type httpRoutingWrapper struct {
	routing.ContentRouting
	routing.PeerRouting
	routing.ValueStore
	routinghelpers.ProvideManyRouter
}

func (c *httpRoutingWrapper) Bootstrap(ctx context.Context) error {
	return nil
}

```

# `tar/format.go`

这段代码定义了一个名为 "tarfmt" 的包，它主要用于处理 tarball（tarball 是 GitHub Boxo 项目中的一个依赖包）文件。这个包实现了 tarball 文件的标准化，以便在不同的系统上进行解包和打包。

具体来说，这段代码做了以下几件事情：

1. 导入了一些必要的依赖：首先，它导入了 "archive/tar"、"bytes"、"context" 和 "errors" 等库，这些库对于 tarball 文件的处理非常有用。

2. 定义了一个名为 "dag" 的指针：这个指针以树状结构表示了一个 tarball 文件的元数据，它包含了这个 tarball 文件的一些信息，如它所属的目录、解包工具、目标 tarball 文件等等。

3. 定义了一个名为 "merkledag" 的函数：这个函数实现了 tarball 文件的元数据写入、读取和删除操作。这个函数使用了 Merkle 冲突检测算法，确保写入的元数据树是完整的。

4. 定义了一个名为 "importer" 的函数：这个函数实现了 tarball 文件的导入操作。它接收一个 tarball 文件的路径，然后使用 unixfs 库读取 tarball 文件，并返回一个 "Chunker" 对象的实例。

5. 定义了一个名为 "uio" 的函数：这个函数实现了 unixfs 库的 io 操作。它可以读写文件并执行各种 io 操作，如复制、移动、权限等等。

6. 定义了一个名为 "chunker" 的函数：这个函数实现了 chunker 函数，用于将 tarball 文件中的数据块分割成更小的数据块并输出。

7. 定义了一个名为 "ipld" 的函数：这个函数实现了 ipld 函数，用于将 tarball 文件中的元数据转换为 ipld 格式并输出。

8. 定义了一个名为 "logging" 的函数：这个函数实现了 logging 函数，用于记录操作过程中的错误信息。

9. 将所有定义的函数组合在一起，并定义了一个 "main" 函数，它接收一个参数（表示要处理的 tarball 文件的路径），然后执行相应的函数，最后将结果输出。

通过这些函数，"tarfmt" 包可以帮助用户处理 tarball 文件，使其具有标准化的一些特点，从而使其在不同的系统上具有更好的兼容性。


```go
package tarfmt

import (
	"archive/tar"
	"bytes"
	"context"
	"errors"
	"io"
	"path"
	"strings"

	dag "github.com/ipfs/boxo/ipld/merkledag"
	"github.com/ipfs/boxo/ipld/merkledag/dagutils"
	importer "github.com/ipfs/boxo/ipld/unixfs/importer"
	uio "github.com/ipfs/boxo/ipld/unixfs/io"

	chunker "github.com/ipfs/boxo/chunker"
	ipld "github.com/ipfs/go-ipld-format"
	logging "github.com/ipfs/go-log"
)

```

这段代码定义了一个名为 "tarfmt" 的日志输出器，并实现了两个函数：

1. `marshalHeader`：将给定的 `tar.Header` 对象压缩打包并写入字节切片 `[]byte` 中的字节数组。

2. `zeroBlock`：创建一个固定大小的字节数组，用于在 `marshalHeader` 函数中写入 `tar.Header` 对象的 `零` 字段的字节数组。

`var log = logging.Logger("tarfmt")` 创建了一个名为 "tarfmt" 的日志输出器，并将输出到标准输出通道 `os.Stderr`。

`var blockSize = 512` 定义了一个名为 `blockSize` 的变量，其值为 512。

`var zeroBlock = make([]byte, blockSize)` 创建了一个名为 `zeroBlock` 的变量，其值是一个大小为 `blockSize` 的字节数组。

`func marshalHeader(h *tar.Header) ([]byte, error)` 函数接收一个 `tar.Header` 对象 `h`，并返回其 `[]byte` 字节数组和压缩打包的错误。

该函数首先创建一个名为 `buf` 的字节缓冲区 `new(bytes.Buffer)`，并创建一个名为 `w` 的 `tar.NewWriter` 函数。然后，该函数使用 `w.WriteHeader(h)` 将 `h` 写入 `buf` 缓冲区，并返回 `buf.Bytes()` 和错误。如果错误，函数返回 `nil` 和错误。

如果 `w.WriteHeader(h)` 成功，函数返回 `buf.Bytes()` 和错误。因为 `blockSize` 变量是固定的，所以每次调用 `marshalHeader` 函数时，`zeroBlock` 字节数组中的所有元素都将被复用。


```go
var log = logging.Logger("tarfmt")

var (
	blockSize = 512
	zeroBlock = make([]byte, blockSize)
)

func marshalHeader(h *tar.Header) ([]byte, error) {
	buf := new(bytes.Buffer)
	w := tar.NewWriter(buf)
	err := w.WriteHeader(h)
	if err != nil {
		return nil, err
	}
	return buf.Bytes(), nil
}

```

这段代码的作用是将一个Tar文件中的目录节点导入到DAGService中，并返回根节点。Tar文件是一个二进制文件，其中包含一些链接文件、目录节点和其他元数据。在使用Tar文件时，需要先将它读入一个TarReader对象中，然后使用DAGService中的Add、Insert和Finalize方法将目录节点导入到DAGService中。

具体来说，代码中实现了一个名为ImportTar的函数，它接受一个IOReader对象和一个DAGService对象作为参数。函数在读取Tar文件内容后，创建了一个根目录节点，然后遍历Tar文件内容，将每个目录节点导入到DAGService中。在导入每个目录节点时，函数会检查其大小并计算其所需的存储空间，然后添加到DAGService中。如果函数在遍历过程中遇到错误，例如Tar文件不存在或读取错误，函数将返回错误并停止执行。最终，函数返回根目录节点，它将作为DAGService中的根节点，用于导出目录树。


```go
// ImportTar imports a tar file into the given DAGService and returns the root
// node.
func ImportTar(ctx context.Context, r io.Reader, ds ipld.DAGService) (*dag.ProtoNode, error) {
	tr := tar.NewReader(r)

	root := new(dag.ProtoNode)
	root.SetData([]byte("ipfs/tar"))

	e := dagutils.NewDagEditor(root, ds)

	for {
		h, err := tr.Next()
		if err != nil {
			if err == io.EOF {
				break
			}
			return nil, err
		}

		header := new(dag.ProtoNode)

		headerBytes, err := marshalHeader(h)
		if err != nil {
			return nil, err
		}

		header.SetData(headerBytes)

		if h.Size > 0 {
			spl := chunker.NewRabin(tr, uint64(chunker.DefaultBlockSize))
			nd, err := importer.BuildDagFromReader(ds, spl)
			if err != nil {
				return nil, err
			}

			err = header.AddNodeLink("data", nd)
			if err != nil {
				return nil, err
			}
		}

		err = ds.Add(ctx, header)
		if err != nil {
			return nil, err
		}

		path := escapePath(h.Name)
		err = e.InsertNodeAtPath(context.Background(), path, header, func() *dag.ProtoNode { return new(dag.ProtoNode) })
		if err != nil {
			return nil, err
		}
	}

	return e.Finalize(ctx, ds)
}

```

此代码定义了一个名为"escapePath"的函数，以及一个名为"tarReader"的结构体。函数的作用是将路径中的每个元素转换为"-"并在其前面添加一个"-"，以便在结构中使用"data"特殊链接，而无需担心"data"链接的结构。

"tarReader"结构体包含一个或多个"-"链接、一个IPLDagService实例以及一个内部变量，用于跟踪当前的上下文上下文。

具体来说，函数首先通过使用strings.Trim函数从路径中删除"/"分隔符，并将结果存储在名为"elems"的字符串中。然后，使用for循环遍历每个元素，将其转换为"-"并在前面添加一个"-"，最后将结果存储在名为"path"的字符串中。

"tarReader"结构体中的"-"链接用于在结构中引用"data"链接，这可以通过将一个IPLDagService实例传递给结构体来实现。此外，结构体还包含一个名为"ds"的内部变量，用于跟踪当前的上下文上下文。另外，结构体还包含一个名为"childRead"的内部变量，用于跟踪当前的子进程，以及一个名为"hdrBuf"的内部变量，用于存储当前的DAG头缓冲区。

最后，函数使用path.Join函数将所有元素连接起来，形成一个完整的路径，并返回该路径。


```go
// adds a '-' to the beginning of each path element so we can use 'data' as a
// special link in the structure without having to worry about.
func escapePath(pth string) string {
	elems := strings.Split(strings.Trim(pth, "/"), "/")
	for i, e := range elems {
		elems[i] = "-" + e
	}
	return path.Join(elems...)
}

type tarReader struct {
	links []*ipld.Link
	ds    ipld.DAGService

	childRead *tarReader
	hdrBuf    *bytes.Reader
	fileRead  *countReader
	pad       int

	ctx context.Context
}

```

This appears to be a function that reads a file and returns the number of bytes read and a pointer to the end of the file. It does this by reading the file in chunks, padding the data to make it 512 bytes in size, and using the `tar` command to read the file system.

The function takes a single argument, `tr`, which is a struct containing information about the file. The `tr` struct has the following fields:

* `fileRead`: a `countReader` object containing the file input.
* `fileRead.Close`: a call to `fileRead.Close` that should be called when the file is closed.
* `tr.fileRead`: a pointer to the file input.
* `tr.ctx`: the context for the file input.
* `tr.ds`: the data start address for the file input.

The function first checks if the `fileRead` field is not `nil`. If it is `nil`, an error is returned. If it is not `nil`, the function then reads the file in chunks and returns the number of bytes read and a pointer to the end of the file.

If the `fileRead` field is `nil`, it is not possible to read the file because there is no file to read. If this happens, the function returns an error.

If the `fileRead` field is not `nil`, the function creates a `countReader` object containing the file input and a `tar` reader object. The `tar` reader reads the file system and passes the data to the `countReader`. The `countReader` object closes the file when it is closed.

The function then returns the number of bytes read and a pointer to the end of the file. If the `fileRead` field is `nil` and the `tar` reader has successfully read the file, the function will return the number of bytes read. If the `tar` reader has encountered an error, the function will return an error.

Overall, the function appears to be reading a file and returning the number of bytes read and a pointer to the end of the file.


```go
func (tr *tarReader) Read(b []byte) (int, error) {
	// if we have a header to be read, it takes priority
	if tr.hdrBuf != nil {
		n, err := tr.hdrBuf.Read(b)
		if err == io.EOF {
			tr.hdrBuf = nil
			return n, nil
		}
		return n, err
	}

	// no header remaining, check for recursive
	if tr.childRead != nil {
		n, err := tr.childRead.Read(b)
		if err == io.EOF {
			tr.childRead = nil
			return n, nil
		}
		return n, err
	}

	// check for filedata to be read
	if tr.fileRead != nil {
		n, err := tr.fileRead.Read(b)
		if err == io.EOF {
			nr := tr.fileRead.n
			tr.pad = (blockSize - (nr % blockSize)) % blockSize
			tr.fileRead.Close()
			tr.fileRead = nil
			return n, nil
		}
		return n, err
	}

	// filedata reads must be padded out to 512 byte offsets
	if tr.pad > 0 {
		n := copy(b, zeroBlock[:tr.pad])
		tr.pad -= n
		return n, nil
	}

	if len(tr.links) == 0 {
		return 0, io.EOF
	}

	next := tr.links[0]
	tr.links = tr.links[1:]

	headerNd, err := next.GetNode(tr.ctx, tr.ds)
	if err != nil {
		return 0, err
	}

	hndpb, ok := headerNd.(*dag.ProtoNode)
	if !ok {
		return 0, dag.ErrNotProtobuf
	}

	tr.hdrBuf = bytes.NewReader(hndpb.Data())

	dataNd, err := hndpb.GetLinkedProtoNode(tr.ctx, tr.ds, "data")
	if err != nil && !errors.Is(err, dag.ErrLinkNotFound) {
		return 0, err
	}

	if err == nil {
		dr, err := uio.NewDagReader(tr.ctx, dataNd, tr.ds)
		if err != nil {
			log.Error("dagreader error: ", err)
			return 0, err
		}

		tr.fileRead = &countReader{r: dr}
	} else if len(headerNd.Links()) > 0 {
		tr.childRead = &tarReader{
			links: headerNd.Links(),
			ds:    tr.ds,
			ctx:   tr.ctx,
		}
	}

	return tr.Read(b)
}

```

该函数 `ExportTar` 将传入的 `DAG` 对象导出为 tar 文件。它与 `ImportTar` 的作用是相反的，因为 `ImportTar` 将 tar 文件导入到内存中，而该函数将 DAG 导出为 tar 文件。

函数接收两个参数：

- `ctx`: 上下文信息，用于度量导入/导出操作的进度。
- `root`: DAG 对象的根节点。
- `ds`: IPLD-DAGService，用于与 IPFS 存储桶进行通信。

函数首先检查传入的 DAG 对象是否为 "ipfs/tar" 类型，如果是，则函数会返回一个 tar 文件的读取器和一个错误。如果不是，函数会返回 nil 和一个错误。

函数的实现包括以下步骤：

1. 如果 DAG 对象为 "ipfs/tar"，则创建一个 `countReader` 并将其赋值为 `n = 0` 和一个 `links` 列表，用于跟踪 DAG 中的链接。
2. 调用 `ds.Get(ctx, root.Name(), nil)` 方法获取 DAG 对象的根节点，并将其添加到 `links` 列表中。
3. 创建一个 `tarReader` 并将其设置为 `links`、`ds` 和 `ctx`。
4. 返回 `tarReader` 的 `Read` 方法，作为 tar 文件的读取器。

该函数的作用是将 DAG 对象导出为 tar 文件，并返回一个 tar 文件的读取器和错误。


```go
// ExportTar exports the passed DAG as a tar file. This function is the inverse
// of ImportTar.
func ExportTar(ctx context.Context, root *dag.ProtoNode, ds ipld.DAGService) (io.Reader, error) {
	if string(root.Data()) != "ipfs/tar" {
		return nil, errors.New("not an IPFS tarchive")
	}
	return &tarReader{
		links: root.Links(),
		ds:    ds,
		ctx:   ctx,
	}, nil
}

type countReader struct {
	r io.ReadCloser
	n int
}

```

这段代码定义了两个函数，一个是 `func (r *countReader) Read(b []byte) (int, error)`，另一个是 `func (r *countReader) Close() error`。

这两个函数的功能如下：

1. `func (r *countReader) Read(b []byte) (int, error)`：

这个函数接收一个 `countReader` 类型的指针 `r` 和一个字节数组 `b`。它使用 `r.r.Read` 方法来读取 `b` 数组中的数据，并将读取到的数据计数器 `r.n` 加一。它返回读取到的数据个数 `n` 和读取错误 `err` 中的一个。

2. `func (r *countReader) Close() error`：

这个函数只是简单地关闭 `r.r` 对象的读取操作。它返回一个错误，如果不成功关闭 `r.r` 对象，则会返回一个错误。


```go
func (r *countReader) Read(b []byte) (int, error) {
	n, err := r.r.Read(b)
	r.n += n
	return n, err
}

func (r *countReader) Close() error {
	return r.r.Close()
}

```


## Sharness test command coverage

Module     |     Online Test |    Offline Test |
-----------|-----------------|-----------------|
object     |           t0051 |           t0051
ls         |           t0045 |           t0045
cat        |           t0040 |
dht        |                 |
bitswap    |                 |
block      |                 |           t0050
daemon     |           t0030 |             N/A
init       |             N/A |           t0020
add        |           t0040 |
config     |           t0021 |           t0021
version    |           t0060 |           t0010
ping       |                 |
diag       |                 |
mount      |           t0030 |
name       |           t0110 |           t0100
pin        |           t0080 |
get        |           t0090 |           t0090
refs       |           t0080 |
repo gc    |           t0080 |
id         |                 |
bootstrap  |           t0120 |           t0120
swarm      |                 |
update     |                 |
commands   |                 |


this is an ipfs integration test

**requirements**

* Docker
* fig
* Go

* ipfs image named "zaqwsx_ipfs-test-img"

```go
make setup
fig build 
fig up
```


this is a bootstrap peer with an empty bootstrap list

it listens on 4011 and 4012


**requirements**

* docker container with bootstrap node linked as "bootstrap" with ID QmNXuBh8HFsWq68Fid8dMbGNQTh7eG6hV9rr1fQyfmfomE
* file in data volume, internally mapped to /data/file


# `test/api-startup/main.go`

该代码的作用是测量HTTP服务器监听的两个不同端口（端口号5001和8080）的响应时间。

具体来说，该代码创建了两个通道：`when` 和 `second`。`when` 通道用于存储每个时间戳（或时间点）的`time.Time`类型数据。`second` 通道则用于存储`when` 通道中的数据，并使用`time.Duration`类型将其转换为`time.Time`类型。

接着，代码使用一个`sync.WaitGroup`来等待两个 goroutine（或协程）的完成。每个`goroutine`都使用一个`http.Get`函数来向指定的端口发送请求，并使用一个`time.Now`函数来记录请求的时间。如果请求成功，则使用`log.Println`函数打印端口号、请求时间和请求状态码。如果请求失败，则退出记录，并阻塞 `when` 通道，让其他 goroutine有机会完成。

最后，代码等待两个 goroutine 完成，然后从 `when` 通道中获取数据，并打印从第一个数据中减去第二个数据的结果。


```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"sync"
	"time"
)

func main() {
	when := make(chan time.Time, 2)
	var wg sync.WaitGroup
	wg.Add(2)
	for _, port := range []string{"5001", "8080"} {
		go func(port string) {
			defer wg.Done()
			for {
				r, err := http.Get(fmt.Sprintf("http://127.0.0.1:%s", port))
				if err != nil {
					continue
				}
				t := time.Now()
				when <- t
				log.Println(port, t, r.StatusCode)
				break
			}
		}(port)
	}
	wg.Wait()
	first := <-when
	second := <-when
	log.Println(second.Sub(first))
}

```

# `test/bench/bench_cli_ipfs_add/main.go`

这段代码是一个 Go 语言编写的程序，主要作用是测试一个名为 "test-mode.sh" 的脚本是否符合预期。如果脚本运行成功，则会输出 "./test-mode.sh: test-mode.sh" 的错误信息；如果脚本运行失败，则会输出更详细的错误信息。

具体来说，这段代码以下几个部分组成：

1. 导入一些必要的包：
	* "flag": 用于解析命令行参数
	* "fmt": 用于输出格式化信息
	* "log": 用于输出日志信息
	* "os": 用于操作操作系统
	* "os/exec": 用于执行操作系统命令
	* "path": 用于操作文件路径
	* "testing": 用于测试

2. 导入额外的包：
	* "github.com/ipfs/kubo/thirdparty/unit": 用于测试第三个库
	* "github.com/ipfs/kubo/config": 用于测试第三个库
	* "github.com/jbenet/go-random": 用于测试第三个库

3. 定义了一个名为 "test-mode.sh" 的脚本：
	* "./test-mode.sh": 在当前目录下创建一个名为 "test-mode.sh" 的脚本
	* "test-mode.sh": 在脚本中，首先导入了必要的包
	* "fmt.Printf": 输出了一个带有两个参数的格式化字符串
	* "log.Printf": 输出了一个带有两个参数的格式化字符串
	* "os.Exit": 退出程序并返回 1，如果测试失败
	* "test-mode.sh": 在中间部分，使用 "github.com/ipfs/kubo/thirdparty/unit/run" 函数下载一些内容
	* "github.com/ipfs/kubo/config/的开源代码，下载了一个"kubo-api.yaml"文件并打印其中的 "Endpoints" 字段
	* "github.com/ipfs/kubo/config/的开源代码，下载了一个"kubo-context.yaml"文件并打印其中的 "Preprocessors" 字段
	* "github.com/ipfs/kubo/config/第三方库，下载了一个"containerd.yaml"文件并打印其中的 "Path" 字段
	* "github.com/ipfs/kubo/config/第三方库，下载了一个"kubelet.yaml"文件并打印其中的 "Endpoints" 字段
	* "github.com/ipfs/kubo/config/第三方库，下载了一个"kubeadm.yaml"文件并打印其中的 "Endpoints" 字段
	* "github.com/ipfs/kubo/config/第三方库，下载了一个"kubelet-proxy.yaml"文件并打印其中的 "Endpoints" 字段
	* "github.com/ipfs/kubo/config/第三方库，下载了一个"kubo-api-request.yaml"文件并打印其中的 "Endpoints" 字段
	* "github.com/ipfs/kubo/config/第三方库，下载了一个"kubo-api-response.yaml"文件并打印其中的 "Endpoints" 字段
	* "github.com/ipfs/kubo/config/第三方库，下载了一个"kubelet-api.yaml"文件并打印其中的 "Endpoints" 字段
	* "github.com/ipfs/kubo/config/第三方库，下载了一个"kubelet-context.yaml"文件并打印其中的 "Endpoints" 字段
	* "github.com/ipfs/kubo/config/第三方库，下载了一个"kubeadm-node-config.yaml"文件并打印其中的 "Endpoints" 字段
	* "github.com/ipfs/kubo/config/第三方库，下载了一个"kubelet


```go
package main

import (
	"flag"
	"fmt"
	"log"
	"os"
	"os/exec"
	"path"
	"testing"

	"github.com/ipfs/kubo/thirdparty/unit"

	config "github.com/ipfs/kubo/config"
	random "github.com/jbenet/go-random"
)

```

这段代码定义了三个变量，分别是`debug`、`online`和`amount`。

`debug`和`online`都是布尔类型的变量，分别设置为`false`和`false`。这两个变量的作用是判断是否启用调试输出和运行在线 benchmarks。

`amount`是一个浮点数类型的变量，也被设置为`10 * unit.MB`。这个变量用于存储要基准测试的容量，每次基准测试的容量都会乘以`2`。

接着在`compareResults`函数中，变量`amount`被用来循环，每次循环都会调用`benchmarkAdd`函数来添加基准测试。函数接收一个`int64`类型的参数，表示每次添加的容量。

`benchmarkAdd`函数的具体实现不在这段代码中给出，但是可以通过包装一些常见的基准测试，比如`http基准测试`、`io/ioutil`基准测试等。

`if results, err := benchmarkAdd(int64(amount)); err != nil { // TODO compare`这一行代码表示，如果在`benchmarkAdd`函数中出现错误，那么将返回这个错误。

如果上述代码中出现的错误仍然存在，那么将打印一个错误消息并退出程序。


```go
var (
	debug  = flag.Bool("debug", false, "direct ipfs output to console")
	online = flag.Bool("online", false, "run the benchmarks with a running daemon")
)

func main() {
	flag.Parse()
	if err := compareResults(); err != nil {
		log.Fatal(err)
	}
}

func compareResults() error {
	var amount unit.Information
	for amount = 10 * unit.MB; amount > 0; amount = amount * 2 {
		if results, err := benchmarkAdd(int64(amount)); err != nil { // TODO compare
			return err
		} else {
			log.Println(amount, "\t", results)
		}
	}
	return nil
}

```

This is a Go program that performs a "benchmark" of a simple FUSE-based file system, using the FUSE-lib and Go's `exec` package. It creates a temporary directory, writes a random amount of entropy to the files in it, and then attempts to add a file to the file system using the `ipfs add` command.

The program has a number of features and options that can be used to control and debug the benchmark. For example, the `-o` flag can be used to specify a different output directory for the benchmark output, or the `-t` flag can be used to specify a timeout for waiting for the `ipfs add` command to finish.

The program also includes a number of helper functions and variables that are used to perform the benchmark, such as writing a random amount of entropy using the `random` package, closing the file after writing to it using the `f.Close()` method, and waiting for the `ipfs add` command to finish using the `select` package.

Overall, this program provides a simple and convenient way to perform a benchmark of a FUSE-based file system in Go.


```go
func benchmarkAdd(amount int64) (*testing.BenchmarkResult, error) {
	var benchmarkError error
	results := testing.Benchmark(func(b *testing.B) {
		b.SetBytes(amount)
		for i := 0; i < b.N; i++ {
			b.StopTimer()
			tmpDir := b.TempDir()

			env := append(
				[]string{fmt.Sprintf("%s=%s", config.EnvDir, path.Join(tmpDir, config.DefaultPathName))}, // first in order to override
				os.Environ()...,
			)
			setupCmd := func(cmd *exec.Cmd) {
				cmd.Env = env
				if *debug {
					cmd.Stdout = os.Stdout
					cmd.Stderr = os.Stderr
				}
			}

			initCmd := exec.Command("ipfs", "init", "-b=2048")
			setupCmd(initCmd)
			if err := initCmd.Run(); err != nil {
				benchmarkError = err
				b.Fatal(err)
			}

			const seed = 1
			f, err := os.CreateTemp("", "")
			if err != nil {
				benchmarkError = err
				b.Fatal(err)
			}
			defer os.Remove(f.Name())

			if err := random.WritePseudoRandomBytes(amount, f, seed); err != nil {
				benchmarkError = err
				b.Fatal(err)
			}
			if err := f.Close(); err != nil {
				benchmarkError = err
				b.Fatal(err)
			}

			func() {
				// FIXME online mode isn't working. client complains that it cannot open leveldb
				if *online {
					daemonCmd := exec.Command("ipfs", "daemon")
					setupCmd(daemonCmd)
					if err := daemonCmd.Start(); err != nil {
						benchmarkError = err
						b.Fatal(err)
					}
					defer func() {
						_ = daemonCmd.Process.Signal(os.Interrupt)
						_ = daemonCmd.Wait()
					}()
				}

				b.StartTimer()
				addCmd := exec.Command("ipfs", "add", f.Name())
				setupCmd(addCmd)
				if err := addCmd.Run(); err != nil {
					benchmarkError = err
					b.Fatal(err)
				}
				b.StopTimer()
			}()
		}
	})
	if benchmarkError != nil {
		return nil, benchmarkError
	}
	return &results, nil
}

```

# `test/bench/offline_add/main.go`

这段代码是一个 Go 语言编写的测试程序，它包括以下主要部分：

1. 导入所需的包：fmt、log、os、os/exec 和 testing。
2. 导入来自第三方库的 config、random 和 unit。
3. 设置 Go 语言工作目录为包含测试文件夹的目录。
4. 实例化第三方库的 config 配置文件。
5. 通过 config.Load失活一个随机对象。
6. 对测试对象执行一些测试。
7. 将测试结果保存到名为 "test-results.txt" 的文件中。




```go
package main

import (
	"fmt"
	"log"
	"os"
	"os/exec"
	"path"
	"testing"

	"github.com/ipfs/kubo/thirdparty/unit"

	config "github.com/ipfs/kubo/config"
	random "github.com/jbenet/go-random"
)

```

这段代码定义了一个名为main的函数，该函数中包含一个名为compareResults的匿名函数。compareResults函数的作用是使用 benchmarkAdd函数来比较两个不同单位的数值，如果比较结果出现错误或者出现任何错误，函数将返回一个错误。

具体来说，compareResults函数首先定义了一个名为amount的变量，该变量是一个整数类型的变量，其值为10 * unit.MB。compareResults函数接下来使用for循环来不断将amount变量乘以2，直到amount的值大于0。在每一次循环中，函数使用基准函数benchmarkAdd函数来获取与amount变量相等的单位数量，并将结果存储在results变量中。如果该函数在尝试获取结果过程中出现错误，则函数将返回一个错误。否则，函数将打印出amount变量和结果，并返回 nil。

这段代码的主要目的是比较两个不同单位的数值，基准函数是用来获取每个单位数量并打印在控制台上的。


```go
func main() {
	if err := compareResults(); err != nil {
		log.Fatal(err)
	}
}

func compareResults() error {
	var amount unit.Information
	for amount = 10 * unit.MB; amount > 0; amount = amount * 2 {
		if results, err := benchmarkAdd(int64(amount)); err != nil { // TODO compare
			return err
		} else {
			log.Println(amount, "\t", results)
		}
	}
	return nil
}

```

该代码定义了一个名为"benchmarkAdd"的函数，接受一个整数参数"amount"，该函数返回一个名为"testing.BenchmarkResult"的类型和一个名为"error"的类型的指针。

函数内部使用了一个名为"testing.Benchmark"的函数，该函数包含一个内部函数"b"，该函数接受一个"testing.B"类型的参数，并覆盖了该参数的"SetBytes"和"N"方法。

在内部函数"b"中，首先设置一个名为"amount"的整数类型的临时变量，并使用for循环遍历该变量。在每次循环中，函数会执行一个名为"stopTimer"的静态方法，该方法暂停当前计时器的运行，并设置一个名为"env"的数组，其中包含一个指向"config.EnvDir"和"path.Join(tmpDir, config.DefaultPathName)"的路径。

接下来，设置一个名为"setupCmd"的函数，该函数接受一个执行命令"ipfs"和一个执行命令"init"的参数，并将"setupCmd"的输出赋值给一个名为"cmd"的执行命令。然后，设置一个名为"seed"的整数类型的临时变量，并创建一个名为"f"的临时文件。

接下来，使用"random.WritePseudoRandomBytes"函数生成一个随机字节数组，随机数组的元素数量为"amount"，然后将该字节数组写入到"f"中，并设置随机数种子为"seed"。最后，使用"f.Close"方法关闭"f"，这样所有生成的随机数据都会被写入到"f"。

最后，设置一个名为"b.StartTimer"的计时器，并使用"exec.Command"函数执行"ipfs add"命令，将"amount"参数作为命令行参数传入。设置"setupCmd"为"cmd"的执行命令，将"f"作为执行命令的文件名，并覆盖"amount"参数为所需的"amount"，将"seed"参数设置为所需的随机数种子。最后，使用"b.StopTimer"方法停止计时器，并在所有计时器都停止时返回"testing.BenchmarkResult"。


```go
func benchmarkAdd(amount int64) (*testing.BenchmarkResult, error) {
	results := testing.Benchmark(func(b *testing.B) {
		b.SetBytes(amount)
		for i := 0; i < b.N; i++ {
			b.StopTimer()
			tmpDir := b.TempDir()

			env := append(os.Environ(), fmt.Sprintf("%s=%s", config.EnvDir, path.Join(tmpDir, config.DefaultPathName)))
			setupCmd := func(cmd *exec.Cmd) {
				cmd.Env = env
			}

			cmd := exec.Command("ipfs", "init", "-b=2048")
			setupCmd(cmd)
			if err := cmd.Run(); err != nil {
				b.Fatal(err)
			}

			const seed = 1
			f, err := os.CreateTemp("", "")
			if err != nil {
				b.Fatal(err)
			}
			defer os.Remove(f.Name())

			err = random.WritePseudoRandomBytes(amount, f, seed)
			if err != nil {
				b.Fatal(err)
			}
			if err := f.Close(); err != nil {
				b.Fatal(err)
			}

			b.StartTimer()
			cmd = exec.Command("ipfs", "add", f.Name())
			setupCmd(cmd)
			if err := cmd.Run(); err != nil {
				b.Fatal(err)
			}
			b.StopTimer()
		}
	})
	return &results, nil
}

```

# `test/cli/backup_bootstrap_test.go`

This is a Go program that starts a Heartbeat server and twodaemon nodes using the Raft protocol. It starts the nodes with a bootstrap interval, which is a time interval between the start of the bootstrap process and the start of the nodes.

The program starts by defining the configuration options that can be used when starting the nodes. The `NewOptionalDuration` function creates an optional duration with a minimum of 250 milliseconds.

The program then starts the nodes by calling the `StartDaemons` method on each node. This method stops the nodes, sets the daemon to start, and starts the node as a daemon. The `StopDaemons` method is used to stop the nodes, and the `ForEachPar` function is used to iterate over the nodes and stop them.

The `Connect` method is used to connect two nodes. This method takes two arguments, the node to connect to and the connection timeout (in milliseconds).

The `StartDaemon` method is used to start a daemon on a node. This method takes one argument, the daemon to start.

The program uses the `assert` types to check that the nodes have the expected number of peers and stops them when they are not connected.

The program is using the Raft protocol to connect the nodes and start the nodes. The `NewOptionalDuration` function is used to create an optional duration with a minimum of 250 milliseconds. The `StartDaemons` method is used to start the nodes, the `StopDaemons` method is used to stop the nodes, and the `Connect` method is used to connect two nodes. The `StartDaemon` method is used to start a daemon on a node and the `assert` types are used to check that the nodes have the expected number of peers and stops them when they are not connected.


```go
package cli

import (
	"fmt"
	"testing"
	"time"

	"github.com/ipfs/kubo/config"
	"github.com/ipfs/kubo/test/cli/harness"
	"github.com/stretchr/testify/assert"
)

func TestBackupBootstrapPeers(t *testing.T) {
	nodes := harness.NewT(t).NewNodes(3).Init()
	nodes.ForEachPar(func(n *harness.Node) {
		n.UpdateConfig(func(cfg *config.Config) {
			cfg.Bootstrap = []string{}
			cfg.Addresses.Swarm = []string{fmt.Sprintf("/ip4/127.0.0.1/tcp/%d", harness.NewRandPort())}
			cfg.Discovery.MDNS.Enabled = false
			cfg.Internal.BackupBootstrapInterval = config.NewOptionalDuration(250 * time.Millisecond)
		})
	})

	// Start all nodes and ensure they all have no peers.
	nodes.StartDaemons()
	nodes.ForEachPar(func(n *harness.Node) {
		assert.Len(t, n.Peers(), 0)
	})

	// Connect nodes 0 and 1, ensure they know each other.
	nodes[0].Connect(nodes[1])
	assert.Len(t, nodes[0].Peers(), 1)
	assert.Len(t, nodes[1].Peers(), 1)
	assert.Len(t, nodes[2].Peers(), 0)

	// Wait a bit to ensure that 0 and 1 saved their temporary bootstrap backups.
	time.Sleep(time.Millisecond * 500)
	nodes.StopDaemons()

	// Start 1 and 2. 2 does not know anyone yet.
	nodes[1].StartDaemon()
	nodes[2].StartDaemon()
	assert.Len(t, nodes[1].Peers(), 0)
	assert.Len(t, nodes[2].Peers(), 0)

	// Connect 1 and 2, ensure they know each other.
	nodes[1].Connect(nodes[2])
	assert.Len(t, nodes[1].Peers(), 1)
	assert.Len(t, nodes[2].Peers(), 1)

	// Start 0, wait a bit. Should connect to 1, and then discover 2 via the
	// backup bootstrap peers.
	nodes[0].StartDaemon()
	time.Sleep(time.Millisecond * 500)

	// Check if they're all connected.
	assert.Len(t, nodes[0].Peers(), 2)
	assert.Len(t, nodes[1].Peers(), 2)
	assert.Len(t, nodes[2].Peers(), 2)
}

```

# `test/cli/basic_commands_test.go`

该代码是一个 Go 语言编写的 CLI 工具链，用于测试 IPFS（InterPlanetary File System）库的客户端和服务端。它主要用于测试 ipfs-cli 工具链的功能，以验证它是否符合 Go 语言的规范。

具体来说，该代码包括以下几个主要部分：

1. 导入需要使用的标准：fmt、regexp、strings 和 testing。

2. 导入 ipfs-cli 的相关包： Harness 和 testutils。

3. 定义了一个名为 "testtest" 的函数，该函数的作用是提供一些通用的功能，如打印错误信息、测试 foo 是否为真、模拟一些 HTTP 请求等。

4. 定义了一个名为 "testipfs" 的函数，该函数的作用是测试 ipfs-cli 和 ipfs-kube 是否能够正常工作。其中，ipfs-kube 是一个假设的 Kubernetes 服务，用于测试 ipfs-cli 的客户端是否能够通过 ipfs-kube 访问文件系统。

5. 导入了 ipfs 和 ipfs-kube 两个测试。

6. 定义了一个名为 "testmain" 的函数，该函数的作用是使用 go test 运行测试。

7. 输出 ipfs-cli 和 ipfs-kube 的版本信息，以确保它们在 CLI 工具链中定义正确。

8. 通过调用 "testipfs" 函数，测试 ipfs-cli 是否能够通过 ipfs-kube 访问文件系统，并测试是否能够正常工作。


```go
package cli

import (
	"fmt"
	"regexp"
	"strings"
	"testing"

	"github.com/blang/semver/v4"
	"github.com/ipfs/kubo/test/cli/harness"
	. "github.com/ipfs/kubo/test/cli/testutils"
	"github.com/stretchr/testify/assert"
	gomod "golang.org/x/mod/module"
)

```

该代码的作用是定义了一个名为`versionRegexp`的 regex 变量，该变量使用了MustCompile()方法进行匹配。其匹配的正则表达式为`^ipfs version (.+)$`，其中`^`匹配字符串的开始位置，`$`匹配字符串的结束位置，`ipfs`匹配字符串中的ipfs，`version`匹配字符串中的version，`.+`匹配字符串中的至少一个字符，`$`匹配字符串的结束位置。

接着，该代码定义了一个名为`parseVersionOutput`的函数，该函数接受一个字符串参数`s`，并返回一个`semver.Version`类型的值。该函数使用versionRegexp.MustCompile()方法来查找字符串中匹配正则表达式的第一部分，如果匹配成功，则返回`semver.Version`类型的值。如果匹配失败，则返回一个 panic()异常。

最后，该代码在同一目录下创建了一个名为`test.txt`的文件，并将其中的内容替换为`It works!`。然后，该代码使用` harness.NewT`()方法创建了一个测试 harness，并使用`t.Parallel()`()方法来保证测试的并行执行。该测试 harness会随机写入一些测试文件到该目录下，然后运行该代码，并输出测试结果。如果测试成功，则输出"/path/to/test.txt: It works!"。


```go
var versionRegexp = regexp.MustCompile(`^ipfs version (.+)$`)

func parseVersionOutput(s string) semver.Version {
	versString := versionRegexp.FindStringSubmatch(s)[1]
	v, err := semver.Parse(versString)
	if err != nil {
		panic(err)
	}
	return v
}

func TestCurDirIsWritable(t *testing.T) {
	t.Parallel()
	h := harness.NewT(t)
	h.WriteFile("test.txt", "It works!")
}

```

这两段代码是在测试IPFS（InterPlanetary File System）的命令行工具`ipfs`。

首先，这两段代码都是基于`t.Parallel()`来并行执行测试的。`t.Parallel()`是Go测试框架中的一个内置函数，用于并行执行多个测试。

第一个测试用例：`func TestIPFSVersionCommandMatchesFlag(t *testing.T) {...}`

这段代码的作用是测试`ipfs`命令行工具中关于`version`标志的命令。具体来说，这段代码创建一个新的`node`实例，运行`ipfs version --all`命令，并输出结果。然后，通过`strings.TrimSpace()`方法去除结果中的换行符和空格，得到一个只包含`version`字样的字符串。接下来，通过`parseVersionOutput()`函数解析并打印`version`命令输出的版本信息。最后，通过`assert.Equal()`函数测试两个`version`值是否相等。

第二个测试用例：`func TestIPFSVersionAll(t *testing.T) {...}`

这段代码的作用是测试`ipfs`命令行工具中所有可用的`version`标志。具体来说，这段代码创建一个新的`node`实例，运行`ipfs version --all`命令，并输出结果。然后，通过`strings.TrimSpace()`方法去除结果中的换行符和空格，得到一个只包含`version`字样的字符串。接下来，通过`assert.Contains()`函数测试四个输出是否都包含`Kubo`、`Repo`、`System`和`Golang`。


```go
func TestIPFSVersionCommandMatchesFlag(t *testing.T) {
	t.Parallel()
	node := harness.NewT(t).NewNode()
	commandVersionStr := node.IPFS("version").Stdout.String()
	commandVersionStr = strings.TrimSpace(commandVersionStr)
	commandVersion := parseVersionOutput(commandVersionStr)

	flagVersionStr := node.IPFS("--version").Stdout.String()
	flagVersionStr = strings.TrimSpace(flagVersionStr)
	flagVersion := parseVersionOutput(flagVersionStr)

	assert.Equal(t, commandVersion, flagVersion)
}

func TestIPFSVersionAll(t *testing.T) {
	t.Parallel()
	node := harness.NewT(t).NewNode()
	res := node.IPFS("version", "--all").Stdout.String()
	res = strings.TrimSpace(res)
	assert.Contains(t, res, "Kubo version")
	assert.Contains(t, res, "Repo version")
	assert.Contains(t, res, "System version")
	assert.Contains(t, res, "Golang version")
}

```

这段代码是一个名为 "TestIPFSVersionDeps" 的测试函数，用于测试 IPFS(InterPlanetary File System) 的版本依赖关系。

它具体做了以下几件事情：

1. 创建一个新的 Harness.Node 实例，用于测试 IPFS 是否正常工作。
2. 通过调用 `node.IPFS("version", "deps").Stdout.String()` 方法，获取 IPFS 版本依赖关系中所有的输出内容。
3. 通过 `strings.TrimSpace()` 函数，去除输出内容中的所有空格。
4. 通过 `SplitLines(res)` 函数，将输出内容拆分成多个并行的行。
5. 通过循环遍历分行的内容，提取其中的模块版本信息。
6. 对于每个提取到的模块版本信息，使用 `assert.NoError()` 函数进行错误检查，确保 IPFS 版本的依赖关系构建没有问题。

整个函数的目的是验证 IPFS 版本依赖关系的构建没有问题，并确保可以通过该函数轻松地追踪和测试不同的依赖关系。


```go
func TestIPFSVersionDeps(t *testing.T) {
	t.Parallel()
	node := harness.NewT(t).NewNode()
	res := node.IPFS("version", "deps").Stdout.String()
	res = strings.TrimSpace(res)
	lines := SplitLines(res)

	assert.Equal(t, "github.com/ipfs/kubo@(devel)", lines[0])

	for _, depLine := range lines[1:] {
		split := strings.Split(depLine, " => ")
		for _, moduleVersion := range split {
			splitModVers := strings.Split(moduleVersion, "@")
			modPath := splitModVers[0]
			modVers := splitModVers[1]
			assert.NoError(t, gomod.Check(modPath, modVers), "path: %s, version: %s", modPath, modVers)
		}
	}
}

```

这道题的目的是测试 IPFS 命令行工具中的所有子命令，并验证它们是否都能正确地接受帮助信息。以下是代码的作用：

1. 首先，我们创建一个名为 `TestIPFSCommands` 的函数，它使用 `testing.T` 类型中的 `Parallel` 选项，以确保测试代码在一个新的人生平流水线中运行，并尝试在多个操作系统上运行。

2. 在 `TestIPFSCommands` 函数中，我们创建了一个新的 ` harness.NewT` 实例和一个名为 `node` 的 `NewNode` 实例。

3. 接下来，我们使用 `node.IPFSCommands` 函数获取当前节点上的所有 `IPFSCommands` 子命令。

4. 我们使用 `assert.Contains` 函数验证是否包含所有期望的子命令。具体来说，我们使用 `assert.Contains` 函数检查 `cmds` 是否包含字符串 `"ipfs add"`、`"ipfs daemon"` 和 `"ipfs update"`。

5. 接下来，我们使用 `assert.Contains` 函数验证是否包含所有期望的子命令。具体来说，我们使用 `assert.Contains` 函数检查是否包含字符串 `"ipfs add"`、`"ipfs daemon"` 和 `"ipfs update"`。

6. 最后，我们在 `for` 循环中尝试使用每个子命令。具体来说，我们使用 `node.IPFS` 函数尝试运行 `ipfs add`、`ipfs daemon` 和 `ipfs update` 命令，并使用 `fmt.Sprintf` 函数将命令行输出格式化为 `"command %q accepts help"` 的字符串。

7. 我们使用 `t.Run` 函数运行 `for` 循环，并使用 `fmt.Sprintf` 函数将输出格式化为 `"command %q accepts help"` 的字符串。

8. 最后，我们使用 ` harness.NewT` 实例的 `Setup` 函数设置 `node` 实例，并返回 `node` 实例。


```go
func TestIPFSCommands(t *testing.T) {
	t.Parallel()
	node := harness.NewT(t).NewNode()
	cmds := node.IPFSCommands()
	assert.Contains(t, cmds, "ipfs add")
	assert.Contains(t, cmds, "ipfs daemon")
	assert.Contains(t, cmds, "ipfs update")
}

func TestAllSubcommandsAcceptHelp(t *testing.T) {
	t.Parallel()
	node := harness.NewT(t).NewNode()
	for _, cmd := range node.IPFSCommands() {
		cmd := cmd
		t.Run(fmt.Sprintf("command %q accepts help", cmd), func(t *testing.T) {
			t.Parallel()
			splitCmd := strings.Split(cmd, " ")[1:]
			node.IPFS(StrCat("help", splitCmd)...)
			node.IPFS(StrCat(splitCmd, "--help")...)
		})
	}
}

```

该测试用例函数的作用是测试所有 root 命令是否在帮助文档中提到了。它使用了一个名为 ` harness.NewT` 的函数创建了一个根节点，并使用 `node.IPFSCommands()` 函数获取了该节点中的所有命令。

然后，它遍历所有命令，并将每个命令拆分成字段。如果拆分后的长度为 2，则说明这是一个 root 命令。否则，如果该命令不在 `notInHelp`  map 中，则测试函数将该命令包含在帮助文档中。

最后，它使用 `node.IPFS("--help").Stdout.String()` 函数获取根节点中的 `--help` 命令的输出，并使用一系列 `assert` 语句来验证帮助文档是否包含根命令。


```go
func TestAllRootCommandsAreMentionedInHelpText(t *testing.T) {
	t.Parallel()
	node := harness.NewT(t).NewNode()
	cmds := node.IPFSCommands()
	var rootCmds []string
	for _, cmd := range cmds {
		splitCmd := strings.Split(cmd, " ")
		if len(splitCmd) == 2 {
			rootCmds = append(rootCmds, splitCmd[1])
		}
	}

	// a few base commands are not expected to be in the help message
	// but we default to requiring them to be in the help message, so that we
	// have to make an conscious decision to exclude them
	notInHelp := map[string]bool{
		"object":   true,
		"shutdown": true,
		"tar":      true,
		"urlstore": true,
		"dns":      true,
	}

	helpMsg := strings.TrimSpace(node.IPFS("--help").Stdout.String())
	for _, rootCmd := range rootCmds {
		if _, ok := notInHelp[rootCmd]; ok {
			continue
		}
		assert.Contains(t, helpMsg, fmt.Sprintf("  %s", rootCmd))
	}
}

```

This appears to be a Go test that checks if certain IPFS commands follow the documentation width limit. The `allowList` and `denyList` variables are used to store the list of commands that are allowed and the list of commands that are denied, respectively. The `node.IPFSCommands()` function is called to get a list of all the available IPFS commands, and then the test checks if each command is in the `allowList` or `denyList`. If the command is in the `allowList`, the test prints a message and does not proceed. If the command is in the `denyList`, the test prints a message and proceeds with the test.


```go
func TestCommandDocsWidth(t *testing.T) {
	t.Parallel()
	node := harness.NewT(t).NewNode()

	// require new commands to explicitly opt in to longer lines
	allowList := map[string]bool{
		"ipfs add":                      true,
		"ipfs block put":                true,
		"ipfs daemon":                   true,
		"ipfs config profile":           true,
		"ipfs pin remote service":       true,
		"ipfs name pubsub":              true,
		"ipfs object patch":             true,
		"ipfs swarm connect":            true,
		"ipfs p2p forward":              true,
		"ipfs p2p close":                true,
		"ipfs swarm disconnect":         true,
		"ipfs swarm addrs listen":       true,
		"ipfs dag resolve":              true,
		"ipfs dag get":                  true,
		"ipfs object stat":              true,
		"ipfs pin remote add":           true,
		"ipfs config show":              true,
		"ipfs config edit":              true,
		"ipfs pin remote rm":            true,
		"ipfs pin remote ls":            true,
		"ipfs pin verify":               true,
		"ipfs dht get":                  true,
		"ipfs pin remote service add":   true,
		"ipfs file ls":                  true,
		"ipfs pin update":               true,
		"ipfs pin rm":                   true,
		"ipfs p2p":                      true,
		"ipfs resolve":                  true,
		"ipfs dag stat":                 true,
		"ipfs name publish":             true,
		"ipfs object diff":              true,
		"ipfs object patch add-link":    true,
		"ipfs name":                     true,
		"ipfs object patch append-data": true,
		"ipfs object patch set-data":    true,
		"ipfs dht put":                  true,
		"ipfs diag profile":             true,
		"ipfs diag cmds":                true,
		"ipfs swarm addrs local":        true,
		"ipfs files ls":                 true,
		"ipfs stats bw":                 true,
		"ipfs urlstore add":             true,
		"ipfs swarm peers":              true,
		"ipfs pubsub sub":               true,
		"ipfs repo fsck":                true,
		"ipfs files write":              true,
		"ipfs swarm limit":              true,
		"ipfs commands completion fish": true,
		"ipfs key export":               true,
		"ipfs routing get":              true,
		"ipfs refs":                     true,
		"ipfs refs local":               true,
		"ipfs cid base32":               true,
		"ipfs pubsub pub":               true,
		"ipfs repo ls":                  true,
		"ipfs routing put":              true,
		"ipfs key import":               true,
		"ipfs swarm peering add":        true,
		"ipfs swarm peering rm":         true,
		"ipfs swarm peering ls":         true,
		"ipfs update":                   true,
		"ipfs swarm stats":              true,
	}
	for _, cmd := range node.IPFSCommands() {
		if _, ok := allowList[cmd]; ok {
			continue
		}
		t.Run(fmt.Sprintf("command %q conforms to docs width limit", cmd), func(t *testing.T) {
			splitCmd := strings.Split(cmd, " ")
			resStr := node.IPFS(StrCat(splitCmd[1:], "--help")...)
			res := strings.TrimSpace(resStr.Stdout.String())
			for _, line := range SplitLines(res) {
				assert.LessOrEqualf(t, len(line), 80, "expected width %d < 80 for %q", len(line), cmd)
			}
		})
	}
}

```

这道题目是关于 `func TestAllCommandsFailWhenPassedBadFlag` 和 `func TestCommandsFlags` 的测试，主要作用是测试 `func TestAllCommandsFailWhenPassedBadFlag` 函数，即在给定的测试场景中，测试 `func TestAllCommandsFailWhenPassedBadFlag` 函数的行为。

具体来说，这段代码主要实现了以下功能：

1. 导入了 `testing` 包；
2. 创建了一个新的 ` harness.Node` 对象，用于测试；
3. 通过 `node.IPFS()` 方法，给 ` harness.Node` 对象一个测试命令，并输出结果；
4. 通过循环遍历 `node.IPFSCommands()` 方法返回的所有命令，并调用 `t.Run()` 方法，对每个命令的参数进行测试；
5. 对于每个命令，先将命令参数使用 `strings.Split()` 方法分割出来，再将所有参数拼接成一个字符串；
6. 通过调用 `node.RunIPFS()` 方法，对分好字的命令参数进行执行；
7. 通过调用 `assert.Equal()` 方法，测试结果是否符合预期，如果不符合，则输出详细信息，以便更好地定位问题。

总的作用就是测试 `func TestAllCommandsFailWhenPassedBadFlag` 函数的行为，并确保它能够在所有测试场景中正常运行。


```go
func TestAllCommandsFailWhenPassedBadFlag(t *testing.T) {
	t.Parallel()
	node := harness.NewT(t).NewNode()

	for _, cmd := range node.IPFSCommands() {
		t.Run(fmt.Sprintf("command %q fails when passed a bad flag", cmd), func(t *testing.T) {
			splitCmd := strings.Split(cmd, " ")
			res := node.RunIPFS(StrCat(splitCmd, "--badflag")...)
			assert.Equal(t, 1, res.Cmd.ProcessState.ExitCode())
		})
	}
}

func TestCommandsFlags(t *testing.T) {
	t.Parallel()
	node := harness.NewT(t).NewNode()
	resStr := node.IPFS("commands", "--flags").Stdout.String()
	assert.Contains(t, resStr, "ipfs pin add --recursive / ipfs pin add -r")
	assert.Contains(t, resStr, "ipfs id --format / ipfs id -f")
	assert.Contains(t, resStr, "ipfs repo gc --quiet / ipfs repo gc -q")
}

```

# `test/cli/completion_test.go`

这段代码是一个用于测试 Bash 命令行工具的工具。具体来说，它使用 Github.com/ipfs/kubo库实现的命令行工具 Harness，并实现了以下主要功能：

1. 定义了一个名为 "TestBashCompletion" 的测试函数；
2. 通过调用 Github.com/ipfs/kubo/test/cli/harness.NewT、github.com/ipfs/kubo/test/cli/testutils.github.com/stretchr/testify/assert 和github.com/ipfs/kubo/test/cli/testutils.github.com/ipfs/kubo测试了一个 Bash 命令行工具的完成情况；
3. 通过调用 Github.com/ipfs/kubo/test/cli/harness.NewNode 和 Harness.newNode 实现了一个测试环境和一个根节点；
4. 通过调用 Harness.writeToTemp 写入了一个 Bash 命令行工具的完成内容到临时文件中，并使用 Harness.sh 将该文件系统的 stdout 内容读取到控制台，同时记录错误；
5. 通过调用 assert.NoError 和 assertion error 验证错误信息，如果错误信息为空，则表示验证成功。


```go
package cli

import (
	"fmt"
	"testing"

	"github.com/ipfs/kubo/test/cli/harness"
	. "github.com/ipfs/kubo/test/cli/testutils"
	"github.com/stretchr/testify/assert"
)

func TestBashCompletion(t *testing.T) {
	t.Parallel()
	h := harness.NewT(t)
	node := h.NewNode()

	res := node.IPFS("commands", "completion", "bash")

	length := len(res.Stdout.String())
	if length < 100 {
		t.Fatalf("expected a long Bash completion file, but got one of length %d", length)
	}

	t.Run("completion file can be loaded in bash", func(t *testing.T) {
		RequiresLinux(t)

		completionFile := h.WriteToTemp(res.Stdout.String())
		res = h.Sh(fmt.Sprintf("source %s && type -t _ipfs", completionFile))
		assert.NoError(t, res.Err)
	})
}

```

该代码是一个名为 "TestZshCompletion" 的测试函数，用于测试 Bash 命令行界面中的命令完成功能。

以下是代码的功能解释：

1. 构造函数：该函数使用 "func" 关键字定义了一个名为 "TestZshCompletion" 的函数，函数内部通过调用 "testing.T" 类型的参数 "t"。

2. 创建数据结构：函数定义了一个名为 "h" 的 " harness"，然后通过 "NewT" 和 "


```go
func TestZshCompletion(t *testing.T) {
	t.Parallel()
	h := harness.NewT(t)
	node := h.NewNode()

	res := node.IPFS("commands", "completion", "zsh")

	length := len(res.Stdout.String())
	if length < 100 {
		t.Fatalf("expected a long Bash completion file, but got one of length %d", length)
	}

	t.Run("completion file can be loaded in bash", func(t *testing.T) {
		RequiresLinux(t)

		completionFile := h.WriteToTemp(res.Stdout.String())
		res = h.Runner.Run(harness.RunRequest{
			Path: "zsh",
			Args: []string{"-c", fmt.Sprintf("autoload -Uz compinit && compinit && source %s && echo -E $_comps[ipfs]", completionFile)},
		})

		assert.NoError(t, res.Err)
		assert.NotEmpty(t, res.Stdout.String())
	})
}

```

# `test/cli/content_blocking_test.go`

该代码是一个 Go 语言编写的命令行工具包，名为 "cli"。它主要用于测试 libp2p 库的用法。

具体来说，该代码实现了以下功能：

1. 定义了一个名为 "test" 的函数，它通过 `net/http` 库向测试服务器发送请求来测试 libp2p 库的 HTTP 功能。
2. 定义了一个名为 "main" 的函数，它使用 `strings` 库将测试服务器上的 HTTP 请求 URI 进行转义，并使用 `path/filepath` 库获取测试文件的绝对路径，以便将测试文件放在测试服务器上的指定目录下。
3. 定义了一个名为 " harness" 的函数，它使用 `libp2p` 库的 `peer` 函数与测试服务器上的节点建立连接，并使用 `libp2phttp` 库的 `fetch` 函数发送 HTTP 请求，以测试 libp2p 库的 HTTP 功能。
4. 定义了一个名为 "assert" 的函数，它使用 `testing` 库的 `assert` 函数对测试结果进行断言，例如对输出日志中的内容进行断言，以确保 libp2p 库能够正确地输出日志。

该代码的作用是提供一个用于测试 libp2p 库命令行工具功能的工具包，通过简单的函数设计和测试，使得开发人员可以更容易地测试和验证 libp2p 库的不同功能。


```go
package cli

import (
	"context"
	"fmt"
	"io"
	"log"
	"net/http"
	"net/url"
	"os"
	"path/filepath"
	"strings"
	"testing"

	"github.com/ipfs/kubo/test/cli/harness"
	"github.com/libp2p/go-libp2p"
	"github.com/libp2p/go-libp2p/core/peer"
	libp2phttp "github.com/libp2p/go-libp2p/p2p/http"
	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/require"
)

```

This is a Go test that checks if the libp2p HTTP Host is able to serve allowed and blocked CID files.

The `clientHost` is a connection to a local libp2p HTTP Host, and it is used to serve the tests.

The `libp2phttp.Host` is used to create a connection to the HTTP Host.

The `libp2p.NamespacedClient` is used to make the HTTP request to the connected HTTP Host.

The tests serve all allowed CID files and block the directories that are blocked.

The `assertions` are using the `require.NoError` and `assert.Equal` from the `testing.T` package.

The `bodyExpl` is a helper that checks the content of the blocked file.

The `bodyExpl.isBlockContent` is used to check if the content is blocked.


```go
func TestContentBlocking(t *testing.T) {
	// NOTE: we can't run this with t.Parallel() because we set IPFS_NS_MAP
	// and running in parallel could impact other tests

	const blockedMsg = "blocked and cannot be provided"
	const statusExpl = "specific HTTP error code is expected"
	const bodyExpl = "Error message informing about content block is expected"

	h := harness.NewT(t)

	// Init IPFS_PATH
	node := h.NewNode().Init("--empty-repo", "--profile=test")

	// Create CIDs we use in test
	h.WriteFile("blocked-dir/subdir/indirectly-blocked-file.txt", "indirectly blocked file content")
	parentDirCID := node.IPFS("add", "--raw-leaves", "-Q", "-r", filepath.Join(h.Dir, "blocked-dir")).Stdout.Trimmed()

	h.WriteFile("directly-blocked-file.txt", "directly blocked file content")
	blockedCID := node.IPFS("add", "--raw-leaves", "-Q", filepath.Join(h.Dir, "directly-blocked-file.txt")).Stdout.Trimmed()

	h.WriteFile("not-blocked-file.txt", "not blocked file content")
	allowedCID := node.IPFS("add", "--raw-leaves", "-Q", filepath.Join(h.Dir, "not-blocked-file.txt")).Stdout.Trimmed()

	// Create denylist at $IPFS_PATH/denylists/test.deny
	denylistTmp := h.WriteToTemp("name: test list\n---\n" +
		"//QmX9dhRcQcKUw3Ws8485T5a9dtjrSCQaUAHnG4iK9i4ceM\n" + // Double hash (sha256) CID block: base58btc(sha256-multihash(QmVTF1yEejXd9iMgoRTFDxBv7HAz9kuZcQNBzHrceuK9HR))
		"//gW813G35CnLsy7gRYYHuf63hrz71U1xoLFDVeV7actx6oX\n" + // Double hash (blake3) Path block under blake3 root CID: base58btc(blake3-multihash(gW7Nhu4HrfDtphEivm3Z9NNE7gpdh5Tga8g6JNZc1S8E47/path))
		"//8526ba05eec55e28f8db5974cc891d0d92c8af69d386fc6464f1e9f372caf549\n" + // Legacy CID double-hash block: sha256(bafkqahtcnrxwg23fmqqgi33vmjwgk2dbonuca3dfm5qwg6jamnuwicq/)
		"//e5b7d2ce2594e2e09901596d8e1f29fa249b74c8c9e32ea01eda5111e4d33f07\n" + // Legacy Path double-hash block: sha256(bafyaagyscufaqalqaacauaqiaejao43vmjygc5didacauaqiae/subpath)
		"/ipfs/" + blockedCID + "\n" + // block specific CID
		"/ipfs/" + parentDirCID + "/subdir*\n" + // block only specific subpath
		"/ipns/blocked-cid.example.com\n" +
		"/ipns/blocked-dnslink.example.com\n")

	if err := os.MkdirAll(filepath.Join(node.Dir, "denylists"), 0o777); err != nil {
		log.Panicf("failed to create denylists dir: %s", err.Error())
	}
	if err := os.Rename(denylistTmp, filepath.Join(node.Dir, "denylists", "test.deny")); err != nil {
		log.Panicf("failed to create test denylist: %s", err.Error())
	}

	// Add two entries to namesys resolution cache
	// /ipns/blocked-cid.example.com point at a blocked CID (to confirm blocking impacts /ipns resolution)
	// /ipns/blocked-dnslink.example.com with safe CID (to test blocking of /ipns/ paths)
	os.Setenv("IPFS_NS_MAP", "blocked-cid.example.com:/ipfs/"+blockedCID+",blocked-dnslink.example.com/ipns/QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn")
	defer os.Unsetenv("IPFS_NS_MAP")

	// Enable GatewayOverLibp2p as we want to test denylist there too
	node.IPFS("config", "--json", "Experimental.GatewayOverLibp2p", "true")

	// Start daemon, it should pick up denylist from $IPFS_PATH/denylists/test.deny
	node.StartDaemon() // we need online mode for GatewayOverLibp2p tests
	client := node.GatewayClient()

	// First, confirm gateway works
	t.Run("Gateway Allows CID that is not blocked", func(t *testing.T) {
		t.Parallel()
		resp := client.Get("/ipfs/" + allowedCID)
		assert.Equal(t, http.StatusOK, resp.StatusCode)
		assert.Equal(t, "not blocked file content", resp.Body)
	})

	// Then, does the most basic blocking case work?
	t.Run("Gateway Denies directly blocked CID", func(t *testing.T) {
		t.Parallel()
		resp := client.Get("/ipfs/" + blockedCID)
		assert.Equal(t, http.StatusGone, resp.StatusCode, statusExpl)
		assert.NotEqual(t, "directly blocked file content", resp.Body)
		assert.Contains(t, resp.Body, blockedMsg, bodyExpl)
	})

	// Confirm parent of blocked subpath is not blocked
	t.Run("Gateway Allows parent Path that is not blocked", func(t *testing.T) {
		t.Parallel()
		resp := client.Get("/ipfs/" + parentDirCID)
		assert.Equal(t, http.StatusOK, resp.StatusCode)
	})

	// Ok, now the full list of test cases we want to cover in both CLI and Gateway
	testCases := []struct {
		name string
		path string
	}{
		{
			name: "directly blocked CID",
			path: "/ipfs/" + blockedCID,
		},
		{
			name: "indirectly blocked file (on a blocked subpath)",
			path: "/ipfs/" + parentDirCID + "/subdir/indirectly-blocked-file.txt",
		},
		{
			name: "/ipns path that resolves to a blocked CID",
			path: "/ipns/blocked-cid.example.com",
		},
		{
			name: "/ipns Path that is blocked by DNSLink name",
			path: "/ipns/blocked-dnslink.example.com",
		},
		{
			name: "double-hash CID block (sha256-multihash)",
			path: "/ipfs/QmVTF1yEejXd9iMgoRTFDxBv7HAz9kuZcQNBzHrceuK9HR",
		},
		{
			name: "double-hash Path block (blake3-multihash)",
			path: "/ipfs/bafyb4ieqht3b2rssdmc7sjv2cy2gfdilxkfh7623nvndziyqnawkmo266a/path",
		},
		{
			name: "legacy CID double-hash block (sha256)",
			path: "/ipfs/bafkqahtcnrxwg23fmqqgi33vmjwgk2dbonuca3dfm5qwg6jamnuwicq",
		},

		{
			name: "legacy Path double-hash block (sha256)",
			path: "/ipfs/bafyaagyscufaqalqaacauaqiaejao43vmjygc5didacauaqiae/subpath",
		},
	}

	// Which specific cliCmds we test against testCases
	cliCmds := [][]string{
		{"block", "get"},
		{"block", "stat"},
		{"dag", "get"},
		{"dag", "export"},
		{"dag", "stat"},
		{"cat"},
		{"ls"},
		{"get"},
		{"refs"},
	}

	expectedMsg := blockedMsg
	for _, testCase := range testCases {

		// Confirm that denylist is active for every command in 'cliCmds' x 'testCases'
		for _, cmd := range cliCmds {
			cmd := cmd
			cliTestName := fmt.Sprintf("CLI '%s' denies %s", strings.Join(cmd, " "), testCase.name)
			t.Run(cliTestName, func(t *testing.T) {
				t.Parallel()
				args := append(cmd, testCase.path)
				errMsg := node.RunIPFS(args...).Stderr.Trimmed()
				if !strings.Contains(errMsg, expectedMsg) {
					t.Errorf("Expected STDERR error message %q, but got: %q", expectedMsg, errMsg)
				}
			})
		}

		// Confirm that denylist is active for every content path in 'testCases'
		gwTestName := fmt.Sprintf("Gateway denies %s", testCase.name)
		t.Run(gwTestName, func(t *testing.T) {
			resp := client.Get(testCase.path)
			assert.Equal(t, http.StatusGone, resp.StatusCode, statusExpl)
			assert.Contains(t, resp.Body, blockedMsg, bodyExpl)
		})

	}

	// Extra edge cases on subdomain gateway

	t.Run("Gateway Denies /ipns Path that is blocked by DNSLink name (subdomain redirect)", func(t *testing.T) {
		t.Parallel()

		gwURL, _ := url.Parse(node.GatewayURL())
		resp := client.Get("/ipns/blocked-dnslink.example.com", func(r *http.Request) {
			r.Host = "localhost:" + gwURL.Port()
		})

		assert.Equal(t, http.StatusGone, resp.StatusCode, statusExpl)
		assert.Contains(t, resp.Body, blockedMsg, bodyExpl)
	})

	t.Run("Gateway Denies /ipns Path that is blocked by DNSLink name (subdomain, no TLS)", func(t *testing.T) {
		t.Parallel()

		gwURL, _ := url.Parse(node.GatewayURL())
		resp := client.Get("/", func(r *http.Request) {
			r.Host = "blocked-dnslink.example.com.ipns.localhost:" + gwURL.Port()
		})

		assert.Equal(t, http.StatusGone, resp.StatusCode, statusExpl)
		assert.Contains(t, resp.Body, blockedMsg, bodyExpl)
	})

	t.Run("Gateway Denies /ipns Path that is blocked by DNSLink name (subdomain, inlined for TLS)", func(t *testing.T) {
		t.Parallel()

		gwURL, _ := url.Parse(node.GatewayURL())
		resp := client.Get("/", func(r *http.Request) {
			// Inlined DNSLink to fit in single DNS label for TLS interop:
			// https://specs.ipfs.tech/http-gateways/subdomain-gateway/#host-request-header
			r.Host = "blocked--dnslink-example-com.ipns.localhost:" + gwURL.Port()
		})

		assert.Equal(t, http.StatusGone, resp.StatusCode, statusExpl)
		assert.Contains(t, resp.Body, blockedMsg, bodyExpl)
	})

	// We need to confirm denylist is active when gateway is run in NoFetch
	// mode (which usually swaps blockservice to a read-only one, and that swap
	// may cause denylists to not be applied, as it is a separate code path)
	t.Run("GatewayNoFetch", func(t *testing.T) {
		// NOTE: we don't run this in parallel, as it requires restart with different config

		// Switch gateway to NoFetch mode
		node.StopDaemon()
		node.IPFS("config", "--json", "Gateway.NoFetch", "true")
		node.StartDaemon()

		// update client, as the port of test node might've changed after restart
		client = node.GatewayClient()

		// First, confirm gateway works
		t.Run("Allows CID that is not blocked", func(t *testing.T) {
			resp := client.Get("/ipfs/" + allowedCID)
			assert.Equal(t, http.StatusOK, resp.StatusCode)
			assert.Equal(t, "not blocked file content", resp.Body)
		})

		// Then, does the most basic blocking case work?
		t.Run("Denies directly blocked CID", func(t *testing.T) {
			resp := client.Get("/ipfs/" + blockedCID)
			assert.Equal(t, http.StatusGone, resp.StatusCode, statusExpl)
			assert.NotEqual(t, "directly blocked file content", resp.Body)
			assert.Contains(t, resp.Body, blockedMsg, bodyExpl)
		})

		// Restore default
		node.StopDaemon()
		node.IPFS("config", "--json", "Gateway.NoFetch", "false")
		node.StartDaemon()
		client = node.GatewayClient()
	})

	// We need to confirm denylist is active on the
	// trustless gateway exposed over libp2p
	// when Experimental.GatewayOverLibp2p=true
	// (https://github.com/ipfs/kubo/blob/master/docs/experimental-features.md#http-gateway-over-libp2p)
	// NOTE: this type fo gateway is hardcoded to be NoFetch: it does not fetch
	// data that is not in local store, so we only need to run it once: a
	// simple smoke-test for allowed CID and blockedCID.
	t.Run("GatewayOverLibp2p", func(t *testing.T) {
		t.Parallel()

		// Create libp2p client that connects to our node over
		// /http1.1 and then talks gateway semantics over the /ipfs/gateway sub-protocol
		clientHost, err := libp2p.New(libp2p.NoListenAddrs)
		require.NoError(t, err)
		err = clientHost.Connect(context.Background(), peer.AddrInfo{
			ID:    node.PeerID(),
			Addrs: node.SwarmAddrs(),
		})
		require.NoError(t, err)

		libp2pClient, err := (&libp2phttp.Host{StreamHost: clientHost}).NamespacedClient("/ipfs/gateway", peer.AddrInfo{ID: node.PeerID()})
		require.NoError(t, err)

		t.Run("Serves Allowed CID", func(t *testing.T) {
			t.Parallel()
			resp, err := libp2pClient.Get(fmt.Sprintf("/ipfs/%s?format=raw", allowedCID))
			require.NoError(t, err)
			defer resp.Body.Close()
			assert.Equal(t, http.StatusOK, resp.StatusCode)
			body, err := io.ReadAll(resp.Body)
			require.NoError(t, err)
			require.Equal(t, string(body), "not blocked file content", bodyExpl)
		})

		t.Run("Denies Blocked CID", func(t *testing.T) {
			t.Parallel()
			resp, err := libp2pClient.Get(fmt.Sprintf("/ipfs/%s?format=raw", blockedCID))
			require.NoError(t, err)
			defer resp.Body.Close()
			assert.Equal(t, http.StatusGone, resp.StatusCode, statusExpl)
			body, err := io.ReadAll(resp.Body)
			require.NoError(t, err)
			assert.NotEqual(t, string(body), "directly blocked file content")
			assert.Contains(t, string(body), blockedMsg, bodyExpl)
		})
	})
}

```