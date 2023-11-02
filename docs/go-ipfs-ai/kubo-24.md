# go-ipfs 源码解析 24

# `/opt/kubo/core/coreapi/name.go`

该代码是一个 Go 语言的库，名为 "coreapi"，它实现了通过 localhost:9000 进行点对点连接到 IPFS 网络的功能。IPFS（InterPlanetary File System）是一个点对点分布式文件系统，旨在提供一种对网络上的任意节点都有机会访问的文件系统。

具体来说，该代码库实现了以下功能：

1. 创建一个名为 "coreapi" 的包，其中定义了与 IPFS 网络的交互操作。
2. 导入了 IPFS 相关的一些接口，如 "path"，"coreiface"，"options"，"caopts"，"tracing"，"attribute"，以及 "peer"。
3. 通过引入一些自己的定义，如 "keystore"，"namesys"，"time"，"context"，以及 "namesys"，"caopts"，"tracing"，"attribute"，使得代码能够使用这些接口。
4. 通过使用 IPFS 提供的 "namesys" 接口创建一个自定义的 "names" 空间。
5. 通过使用 IPFS 提供的 "caopts" 接口设置一个默认的认证选项。
6. 通过使用 IPFS 提供的 "tracing" 接口记录与 IPFS 网络交互的每个请求的跟踪。
7. 通过使用 IPFS 提供的 "attribute" 接口获取与每个连接相关的元数据。
8. 通过使用 IPFS 提供的 "peer" 接口与 IPFS 网络上的其他节点建立连接。


```
package coreapi

import (
	"context"
	"fmt"
	"strings"
	"time"

	"github.com/ipfs/boxo/ipns"
	keystore "github.com/ipfs/boxo/keystore"
	"github.com/ipfs/boxo/namesys"
	"github.com/ipfs/kubo/tracing"
	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/trace"

	coreiface "github.com/ipfs/boxo/coreiface"
	caopts "github.com/ipfs/boxo/coreiface/options"
	"github.com/ipfs/boxo/path"
	ci "github.com/libp2p/go-libp2p/core/crypto"
	peer "github.com/libp2p/go-libp2p/core/peer"
)

```

This is a Go function that handles the publish attribute for a node. Here's how it works:

1. The function takes an `attribute` object and a `publicationPath` parameter.
2. It uses the `WithAttributes` method from the `namesys` package to set the publication path attribute.
3. It checks if the publishAllowed attribute is set to `true` and returns the function error if an error occurs.
4. If the publishAllowed is set to `true`, it creates the necessary options for the publish operation using the `caopts` and `namesys` packages.
5. It sets the appropriate attributes for the publish operation using the `namesys` package.
6. It checks if the `TTL` attribute is set and sets the appropriate attributes for the publish operation using the `namesys` package.
7. It uses the `api.checkOnline` and `api.checkOffline` methods to check if the node is online and allow offline publishing, respectively.
8. It uses the `keylookup` function from the `golang.org/x/crypto/keystore` package to look up the private key for the repository.
9. It uses the `namesys.Publish` method to publish the node, passing in the necessary options as an argument.
10. It returns the `ipns.Name` object.


```
type NameAPI CoreAPI

// Publish announces new IPNS name and returns the new IPNS entry.
func (api *NameAPI) Publish(ctx context.Context, p path.Path, opts ...caopts.NamePublishOption) (ipns.Name, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.NameAPI", "Publish", trace.WithAttributes(attribute.String("path", p.String())))
	defer span.End()

	if err := api.checkPublishAllowed(); err != nil {
		return ipns.Name{}, err
	}

	options, err := caopts.NamePublishOptions(opts...)
	if err != nil {
		return ipns.Name{}, err
	}
	span.SetAttributes(
		attribute.Bool("allowoffline", options.AllowOffline),
		attribute.String("key", options.Key),
		attribute.Float64("validtime", options.ValidTime.Seconds()),
	)
	if options.TTL != nil {
		span.SetAttributes(attribute.Float64("ttl", options.TTL.Seconds()))
	}

	err = api.checkOnline(options.AllowOffline)
	if err != nil {
		return ipns.Name{}, err
	}

	k, err := keylookup(api.privateKey, api.repo.Keystore(), options.Key)
	if err != nil {
		return ipns.Name{}, err
	}

	eol := time.Now().Add(options.ValidTime)

	publishOptions := []namesys.PublishOption{
		namesys.PublishWithEOL(eol),
		namesys.PublishWithIPNSOption(ipns.WithV1Compatibility(options.CompatibleWithV1)),
	}

	if options.TTL != nil {
		publishOptions = append(publishOptions, namesys.PublishWithTTL(*options.TTL))
	}

	err = api.namesys.Publish(ctx, k, p, publishOptions...)
	if err != nil {
		return ipns.Name{}, err
	}

	pid, err := peer.IDFromPrivateKey(k)
	if err != nil {
		return ipns.Name{}, err
	}

	return ipns.NameFromPeer(pid), nil
}

```

此函数的作用是实现一个名为"CoreAPI.NameAPI"的API，用于在Core API中搜索指定名称的IPNS实例。它接受一个可选的"opts"参数数组，其中包含一些用于指定名称解析选项的选项。

函数首先在内部进行一些设置，包括在请求上下文中的跟踪，以帮助调试API调用。然后，它设置一些请求选项的属性和设置超时时间。

接下来，它检查API是否可用并设置名称解析选项。如果设置超时时间后仍然无法访问API，它将返回一个空通道和一个错误。

然后，它使用name参数创建一个路径，并使用resolvers函数尝试从命名存储中解析名称。如果解析失败，它将重试所有解析选项并继续尝试，直到成功或所有的尝试都失败。成功解析名称后，它将返回IPNS结果的通道，并将其分配给out通道。最后，它确保out通道有足够的数据来处理返回的IPNS结果，然后关闭它。


```
func (api *NameAPI) Search(ctx context.Context, name string, opts ...caopts.NameResolveOption) (<-chan coreiface.IpnsResult, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.NameAPI", "Search", trace.WithAttributes(attribute.String("name", name)))
	defer span.End()

	options, err := caopts.NameResolveOptions(opts...)
	if err != nil {
		return nil, err
	}

	span.SetAttributes(attribute.Bool("cache", options.Cache))

	err = api.checkOnline(true)
	if err != nil {
		return nil, err
	}

	var resolver namesys.Resolver = api.namesys
	if !options.Cache {
		resolver, err = namesys.NewNameSystem(api.routing,
			namesys.WithDatastore(api.repo.Datastore()),
			namesys.WithDNSResolver(api.dnsResolver))
		if err != nil {
			return nil, err
		}
	}

	if !strings.HasPrefix(name, "/ipns/") {
		name = "/ipns/" + name
	}

	p, err := path.NewPath(name)
	if err != nil {
		return nil, err
	}

	out := make(chan coreiface.IpnsResult)
	go func() {
		defer close(out)
		for res := range resolver.ResolveAsync(ctx, p, options.ResolveOpts...) {
			select {
			case out <- coreiface.IpnsResult{Path: res.Path, Err: res.Err}:
			case <-ctx.Done():
				return
			}
		}
	}()

	return out, nil
}

```

这段代码的作用是实现了一个名为 `NameAPI` 的 API，用于查找指定给定名称的最新版本及其路径。它接受一个 `name` 参数和一个或多个 `opts...caopts.NameResolveOption`，然后返回指定名称的路径和错误。

代码首先定义了一个名为 `Resolve` 的函数，该函数使用一个名为 `api` 的上下文参数，一个名为 `name` 的字符串参数，和一个或多个名为 `opts...caopts.NameResolveOption` 的选项参数。函数内部使用一个名为 `ctx` 的上下文参数，一个名为 `span` 的跟踪上下文，和一个名为 `defer` 的延迟函数。

函数内部首先设置一个名为 `ctx` 的上下文并启用跟踪，然后设置一个名为 `cancel` 的上下文并启用取消。接下来，使用 `api.Search` 函数搜索指定名称的结果，并将结果存储在名为 `res` 的变量中。然后，在循环中处理每个结果，设置路径 `p` 和错误 `res.Err`，并检查是否已发生错误。最后，如果所有结果都处理完毕，函数返回路径 `p` 和错误 `res.Err`。


```
// Resolve attempts to resolve the newest version of the specified name and
// returns its path.
func (api *NameAPI) Resolve(ctx context.Context, name string, opts ...caopts.NameResolveOption) (path.Path, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.NameAPI", "Resolve", trace.WithAttributes(attribute.String("name", name)))
	defer span.End()

	ctx, cancel := context.WithCancel(ctx)
	defer cancel()

	results, err := api.Search(ctx, name, opts...)
	if err != nil {
		return nil, err
	}

	err = coreiface.ErrResolveFailed
	var p path.Path

	for res := range results {
		p, err = res.Path, res.Err
		if err != nil {
			break
		}
	}

	return p, err
}

```

此代码定义了一个名为`keylookup`的函数，它接收一个名为`self`的参数，一个名为`kstore`的第二个参数，以及一个字符串`k`。函数返回一个名为`ci.PrivKey`的值，或者一个名为`error`的错误。

函数的作用是查找一个私钥与给定的`k`名称，或者在失败时返回一个错误。首先，函数通过`k == "self"`判断输入的`k`是否为"self"（私有 key 的名称），如果是，则返回输入的`self`，否则继续查找。接下来，函数通过`kstore.Get(k)`尝试从`keystore`中查找给定`k`的私钥，如果查找成功，则返回该私钥，否则返回一个错误。接着，函数通过`kstore.List()`获取`keystore`中的所有键列表，如果返回过程中出现错误，则返回一个错误。最后，函数通过`peer.Decode(k)`和`peer.IDFromPrivateKey`分别尝试从`peer`中根据给定的`k`名称获取对等方ID和私钥，如果失败，则返回一个错误。如果对等方ID等于给定的ID，则返回该私钥，否则继续查找。


```
func keylookup(self ci.PrivKey, kstore keystore.Keystore, k string) (ci.PrivKey, error) {
	////////////////////
	// Lookup by name //
	////////////////////

	// First, lookup self.
	if k == "self" {
		return self, nil
	}

	// Then, look in the keystore.
	res, err := kstore.Get(k)
	if res != nil {
		return res, nil
	}

	if err != nil && err != keystore.ErrNoSuchKey {
		return nil, err
	}

	keys, err := kstore.List()
	if err != nil {
		return nil, err
	}

	//////////////////
	// Lookup by ID //
	//////////////////
	targetPid, err := peer.Decode(k)
	if err != nil {
		return nil, keystore.ErrNoSuchKey
	}

	// First, check self.
	pid, err := peer.IDFromPrivateKey(self)
	if err != nil {
		return nil, fmt.Errorf("failed to determine peer ID for private key: %w", err)
	}
	if pid == targetPid {
		return self, nil
	}

	// Then, look in the keystore.
	for _, key := range keys {
		privKey, err := kstore.Get(key)
		if err != nil {
			return nil, err
		}

		pid, err := peer.IDFromPrivateKey(privKey)
		if err != nil {
			return nil, err
		}

		if targetPid == pid {
			return privKey, nil
		}
	}

	return nil, fmt.Errorf("no key by the given name or PeerID was found")
}

```

# `/opt/kubo/core/coreapi/object.go`

该代码的作用是定义了一个名为 "coreapi" 的包，其中定义了一些与 ByteArrayView、Encoding 和 errors 等相关的函数和类型。

具体来说，这些函数和类型用于实现了一些常见的操作，如创建 ByteArrayView、将 ByteArrayView 中的内容转换为 JSON 编码的字符串、解码 JSON 编码的字符串并获取 ByteArrayView、将 ByteArrayView 中的内容转换为 XML 编码的字符串、解码 XML 编码的字符串并获取 ByteArrayView、将 ByteArrayView 中的内容转换为 XML 编码的字符串并获取 ByteArrayView、创建 Merkledag 和 ByteArrayView、创建 UnixFS 和 ByteArrayView、创建 Pin 用于 ByteArrayView 的访问、创建 CID 和 ByteArrayView 中的内容、获取 ByteArrayView 中的内容等等。

该代码还定义了一些错误类型，如 errors.AlreadyExists 和 errors.NotFound，用于处理文件或 ByteArrayView 中不存在指定内容的情况。


```
package coreapi

import (
	"bytes"
	"context"
	"encoding/base64"
	"encoding/json"
	"encoding/xml"
	"errors"
	"fmt"
	"io"

	coreiface "github.com/ipfs/boxo/coreiface"
	caopts "github.com/ipfs/boxo/coreiface/options"
	dag "github.com/ipfs/boxo/ipld/merkledag"
	"github.com/ipfs/boxo/ipld/merkledag/dagutils"
	ft "github.com/ipfs/boxo/ipld/unixfs"
	"github.com/ipfs/boxo/path"
	pin "github.com/ipfs/boxo/pinning/pinner"
	cid "github.com/ipfs/go-cid"
	ipld "github.com/ipfs/go-ipld-format"
	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/trace"

	"github.com/ipfs/kubo/tracing"
)

```

这段代码定义了一个名为`ObjectAPI`的类型，其中包含一个名为`Link`的结构体类型，一个名为`Node`的结构体类型，以及一个名为`New`的函数。

`New`函数接受一个可选的`opts`参数，它是一个包含`ObjectAPI`结构体和一些其他选项的`caopts.ObjectNewOption`类型。函数在内部使用`tracing.Span`函数来记录一个名为“CoreAPI.ObjectAPI.New”的跨系统调用，并返回一个`ipld.Node`值和一个错误。

如果调用`New`函数时没有提供`opts`参数，则函数将返回`nil`，否则它将尝试根据传入的`opts`来创建一个`Node`实例。如果创建过程中出现错误，函数将返回一个非`nil`错误。

`Link`结构体包含一个名为`Name`的字符串和一个名为`Hash`的字符串，以及一个名为`Size`的`uint64`类型的字段。

`Node`结构体包含一个名为`Links`的数组，包含一个名为`Data`的字符串和一个名为`Size`的`uint64`类型的字段。

`ObjectAPI`是一个定义了`Node`和`Link`结构体的类型，以及一个名为`New`的函数。这个类型和函数都在定义了它们的行为之后被用于创建新的`Node`实例和返回它们。


```
const inputLimit = 2 << 20

type ObjectAPI CoreAPI

type Link struct {
	Name, Hash string
	Size       uint64
}

type Node struct {
	Links []Link
	Data  string
}

func (api *ObjectAPI) New(ctx context.Context, opts ...caopts.ObjectNewOption) (ipld.Node, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "New")
	defer span.End()

	options, err := caopts.ObjectNewOptions(opts...)
	if err != nil {
		return nil, err
	}

	var n ipld.Node
	switch options.Type {
	case "empty":
		n = new(dag.ProtoNode)
	case "unixfs-dir":
		n = ft.EmptyDirNode()
	default:
		return nil, fmt.Errorf("unknown node type: %s", options.Type)
	}

	err = api.dag.Add(ctx, n)
	if err != nil {
		return nil, err
	}
	return n, nil
}

```

This is a function that takes a piece of data in a given data type and returns a serialized representation of the data in the specified data type. It uses the built-in `json`, `xml`, and `protobuf` decoders to handle different data types.

The function takes a string `data` as input, and returns a serialized representation of the data in the specified data type. The serialized representation is returned in the following formats:

* If the data type is `json`, the serialized representation is returned as a JSON object.
* If the data type is `xml`, the serialized representation is returned as an XML document.
* If the data type is `protobuf`, the serialized representation is returned as a Protobuf message.
* If the data type is `unknown`, an error is thrown.

If the serialization fails due to an error, the function returns an error. If the data type cannot be serialized, an error is also thrown.

The function also allows for pinning of the data by default, which means that the data can be pinned to further improve performance. The pinning is done recursively, so that the entire data structure can be pinned.


```
func (api *ObjectAPI) Put(ctx context.Context, src io.Reader, opts ...caopts.ObjectPutOption) (path.ImmutablePath, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "Put")
	defer span.End()

	options, err := caopts.ObjectPutOptions(opts...)
	if err != nil {
		return path.ImmutablePath{}, err
	}
	span.SetAttributes(
		attribute.Bool("pin", options.Pin),
		attribute.String("datatype", options.DataType),
		attribute.String("inputenc", options.InputEnc),
	)

	data, err := io.ReadAll(io.LimitReader(src, inputLimit+10))
	if err != nil {
		return path.ImmutablePath{}, err
	}

	var dagnode *dag.ProtoNode
	switch options.InputEnc {
	case "json":
		node := new(Node)
		decoder := json.NewDecoder(bytes.NewReader(data))
		decoder.DisallowUnknownFields()
		err = decoder.Decode(node)
		if err != nil {
			return path.ImmutablePath{}, err
		}

		dagnode, err = deserializeNode(node, options.DataType)
		if err != nil {
			return path.ImmutablePath{}, err
		}

	case "protobuf":
		dagnode, err = dag.DecodeProtobuf(data)

	case "xml":
		node := new(Node)
		err = xml.Unmarshal(data, node)
		if err != nil {
			return path.ImmutablePath{}, err
		}

		dagnode, err = deserializeNode(node, options.DataType)
		if err != nil {
			return path.ImmutablePath{}, err
		}

	default:
		return path.ImmutablePath{}, errors.New("unknown object encoding")
	}

	if err != nil {
		return path.ImmutablePath{}, err
	}

	if options.Pin {
		defer api.blockstore.PinLock(ctx).Unlock(ctx)
	}

	err = api.dag.Add(ctx, dagnode)
	if err != nil {
		return path.ImmutablePath{}, err
	}

	if options.Pin {
		if err := api.pinning.PinWithMode(ctx, dagnode.Cid(), pin.Recursive); err != nil {
			return path.ImmutablePath{}, err
		}

		err = api.pinning.Flush(ctx)
		if err != nil {
			return path.ImmutablePath{}, err
		}
	}

	return path.FromCid(dagnode.Cid()), nil
}

```

这段代码定义了两个函数，一个是 `func (api *ObjectAPI) Get(ctx context.Context, path path.Path) (ipld.Node, error)`，另一个是 `func (api *ObjectAPI) Data(ctx context.Context, path path.Path) (io.Reader, error)`。

这两个函数接收一个 `api` 对象和一个 `path` 参数，返回一个 `ipld.Node` 类型的数据，或者一个错误的错误。在函数内部，使用了 `api.core()` 函数来获取数据，这个函数可能从外部库或者数据库中获取数据，并返回一个 `ipld.Node` 类型的数据。如果这个函数失败，那么会返回一个错误的错误。

`api.Get` 函数的参数包括一个 `ctx` 和一个 `path` 参数。`ctx` 是传递给函数的上下文，`path` 是请求的数据路径。函数返回一个 `ipld.Node` 类型的数据，如果这个数据可用，否则返回一个错误的错误。

`api.Data` 函数的参数与 `api.Get` 函数类似，只不过返回的是一个 `io.Reader` 类型的数据。函数返回一个 `ipld.Node` 类型的数据，如果这个数据可用，否则返回一个错误的错误。


```
func (api *ObjectAPI) Get(ctx context.Context, path path.Path) (ipld.Node, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "Get", trace.WithAttributes(attribute.String("path", path.String())))
	defer span.End()
	return api.core().ResolveNode(ctx, path)
}

func (api *ObjectAPI) Data(ctx context.Context, path path.Path) (io.Reader, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "Data", trace.WithAttributes(attribute.String("path", path.String())))
	defer span.End()

	nd, err := api.core().ResolveNode(ctx, path)
	if err != nil {
		return nil, err
	}

	pbnd, ok := nd.(*dag.ProtoNode)
	if !ok {
		return nil, dag.ErrNotProtobuf
	}

	return bytes.NewReader(pbnd.Data()), nil
}

```

此函数的作用是获取指定路径下的所有链接。它使用了Go标准库中的tracing包来记录API调用过程的日志。

具体来说，函数接收两个参数：一个指向ObjectAPI类型对象的引用和一个路径参数。函数内部使用tracing.Span函数来记录请求的日志，其中path参数也被传递给tracing.WithAttributes函数，用于将请求相关的信息添加到日志中。

函数首先使用api.core().ResolveNode函数尝试从API中解析出指定的节点，如果失败则返回一个非空链表和错误。如果成功，函数使用nd.Links函数获取指定节点下的所有链接，并将它们存储在一个数组中。最后，函数返回这个数组，或是nil，取决于是否发生了错误。

整个函数的实现主要目的是从API中获取指定路径下的链接，并返回它们。


```
func (api *ObjectAPI) Links(ctx context.Context, path path.Path) ([]*ipld.Link, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "Links", trace.WithAttributes(attribute.String("path", path.String())))
	defer span.End()

	nd, err := api.core().ResolveNode(ctx, path)
	if err != nil {
		return nil, err
	}

	links := nd.Links()
	out := make([]*ipld.Link, len(links))
	for n, l := range links {
		out[n] = (*ipld.Link)(l)
	}

	return out, nil
}

```

此函数名为 `func (api *ObjectAPI) Stat(ctx context.Context, path path.Path) (*coreiface.ObjectStat, error)`，它接收一个名为 `api` 的指针参数和一个名为 `path` 的路径参数。

该函数内部包含三个步骤：

1. 调用父函数 `api.core().ResolveNode(ctx, path)`，并获取节点对象 `nd`。
2. 调用子函数 `nd.Stat()`，获取节点统计信息 `stat`。
3. 创建一个名为 `out` 的 `coreiface.ObjectStat` 结构体，设置其 `Cid`、`NumLinks`、`BlockSize`、`LinksSize`、`DataSize` 和 `CumulativeSize` 字段，然后将其赋值给 `out`。
4. 返回 `out` 和 ` nil` 作为结果。

函数的作用是获取给定路径的节点统计信息，例如文件的大小、链接数等，并将其返回。在函数内部，首先通过调用 `api.core().ResolveNode(ctx, path)` 获取节点对象 `nd`，然后通过调用 `nd.Stat()` 获取节点统计信息 `stat`。最后，创建一个 `coreiface.ObjectStat` 结构体，并设置其字段的值，然后将其返回。


```
func (api *ObjectAPI) Stat(ctx context.Context, path path.Path) (*coreiface.ObjectStat, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "Stat", trace.WithAttributes(attribute.String("path", path.String())))
	defer span.End()

	nd, err := api.core().ResolveNode(ctx, path)
	if err != nil {
		return nil, err
	}

	stat, err := nd.Stat()
	if err != nil {
		return nil, err
	}

	out := &coreiface.ObjectStat{
		Cid:            nd.Cid(),
		NumLinks:       stat.NumLinks,
		BlockSize:      stat.BlockSize,
		LinksSize:      stat.LinksSize,
		DataSize:       stat.DataSize,
		CumulativeSize: stat.CumulativeSize,
	}

	return out, nil
}

```

这段代码是一个名为`func`的函数，接收一个名为`api`的指针参数，代表一个名为`ObjectAPI`的类。这个函数的作用是在给定的路径下创建一个新的链接。

函数的具体实现包括以下步骤：

1. 设置链的名称，如果未指定名称，则使用链的名称作为名称。
2. 设置链的目标路径，如果未指定目标路径，则使用链的目标路径作为目标路径。
3. 如果定义了`opts...caopts.ObjectAddLinkOption`，则将其设置为`opts...`。
4. 如果出现错误，则返回路径`nil`和错误信息。
5. 如果链已经定义，则创建一个新的链接节点`dag.Node`，并将其插入到链的节点中。
6. 对链的节点进行`e.InsertNodeAtPath`操作，并使用`e.Finalize`方法对链的节点进行最终化。
7. 返回链的起始点，如果成功创建链接则返回，否则返回错误信息。


```
func (api *ObjectAPI) AddLink(ctx context.Context, base path.Path, name string, child path.Path, opts ...caopts.ObjectAddLinkOption) (path.ImmutablePath, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "AddLink", trace.WithAttributes(
		attribute.String("base", base.String()),
		attribute.String("name", name),
		attribute.String("child", child.String()),
	))
	defer span.End()

	options, err := caopts.ObjectAddLinkOptions(opts...)
	if err != nil {
		return path.ImmutablePath{}, err
	}
	span.SetAttributes(attribute.Bool("create", options.Create))

	baseNd, err := api.core().ResolveNode(ctx, base)
	if err != nil {
		return path.ImmutablePath{}, err
	}

	childNd, err := api.core().ResolveNode(ctx, child)
	if err != nil {
		return path.ImmutablePath{}, err
	}

	basePb, ok := baseNd.(*dag.ProtoNode)
	if !ok {
		return path.ImmutablePath{}, dag.ErrNotProtobuf
	}

	var createfunc func() *dag.ProtoNode
	if options.Create {
		createfunc = ft.EmptyDirNode
	}

	e := dagutils.NewDagEditor(basePb, api.dag)

	err = e.InsertNodeAtPath(ctx, name, childNd, createfunc)
	if err != nil {
		return path.ImmutablePath{}, err
	}

	nnode, err := e.Finalize(ctx, api.dag)
	if err != nil {
		return path.ImmutablePath{}, err
	}

	return path.FromCid(nnode.Cid()), nil
}

```

此函数名为 `RmLink`，它接收一个名为 `api` 的对象指针，以及一个根路径 `base` 和一个链接字符串 `link`。函数内部执行以下操作：

1. 使用 `tracing.Span` 记录请求的上下文信息，并设置请求属于 "CoreAPI.ObjectAPI" 主题，并附上请求参数的 `base` 和 `link` 属性。
2. 尝试使用 `api.core()` 函数的 `ResolveNode` 函数查找根路径，并记录结果。
3. 如果查找失败，函数返回一个 ImmutablePath 对象和一个错误。
4. 如果成功，函数使用 `dagutils.NewDagEditor` 创建一个 `DagEditor` 对象，然后使用 `RmLink` 函数操作，最后使用 `Finalize` 函数提交操作结果。
5. 返回路径的 ImmutablePath 对象，如果没有错误。


```
func (api *ObjectAPI) RmLink(ctx context.Context, base path.Path, link string) (path.ImmutablePath, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "RmLink", trace.WithAttributes(
		attribute.String("base", base.String()),
		attribute.String("link", link)),
	)
	defer span.End()

	baseNd, err := api.core().ResolveNode(ctx, base)
	if err != nil {
		return path.ImmutablePath{}, err
	}

	basePb, ok := baseNd.(*dag.ProtoNode)
	if !ok {
		return path.ImmutablePath{}, dag.ErrNotProtobuf
	}

	e := dagutils.NewDagEditor(basePb, api.dag)

	err = e.RmLink(ctx, link)
	if err != nil {
		return path.ImmutablePath{}, err
	}

	nnode, err := e.Finalize(ctx, api.dag)
	if err != nil {
		return path.ImmutablePath{}, err
	}

	return path.FromCid(nnode.Cid()), nil
}

```

此代码定义了两个函数：`AppendData` 和 `SetData`，它们都是用于在指定的路径下向 API 对象中设置数据。

这两个函数都使用了相同的 API 对象 `ObjectAPI`，以及一个名为 `ctx` 的上下文，和一个名为 `path` 的路径参数。这两个函数的参数也不同，`AppendData` 需要一个名为 `r` 的输入参数，而 `SetData` 需要一个名为 `r` 的输入参数。

这两个函数的实现都在同一个名为 `coreAPI.ObjectAPI` 的跟踪中进行，其中包括一些记录到跟踪中的元数据。这两个函数的实现都在同一个名为 `AppendData` 的函数中，该函数使用 `api.core().ResolveNode` 和 `api.dag.Add` 函数来处理数据路径和 API 对象。

这两个函数一起构成了一个完整的 API 对象的读取和写入数据的功能，允许用户在指定的路径下读取数据并将其写入到 API 对象中。


```
func (api *ObjectAPI) AppendData(ctx context.Context, path path.Path, r io.Reader) (path.ImmutablePath, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "AppendData", trace.WithAttributes(attribute.String("path", path.String())))
	defer span.End()

	return api.patchData(ctx, path, r, true)
}

func (api *ObjectAPI) SetData(ctx context.Context, path path.Path, r io.Reader) (path.ImmutablePath, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "SetData", trace.WithAttributes(attribute.String("path", path.String())))
	defer span.End()

	return api.patchData(ctx, path, r, false)
}

func (api *ObjectAPI) patchData(ctx context.Context, p path.Path, r io.Reader, appendData bool) (path.ImmutablePath, error) {
	nd, err := api.core().ResolveNode(ctx, p)
	if err != nil {
		return path.ImmutablePath{}, err
	}

	pbnd, ok := nd.(*dag.ProtoNode)
	if !ok {
		return path.ImmutablePath{}, dag.ErrNotProtobuf
	}

	data, err := io.ReadAll(r)
	if err != nil {
		return path.ImmutablePath{}, err
	}

	if appendData {
		data = append(pbnd.Data(), data...)
	}
	pbnd.SetData(data)

	err = api.dag.Add(ctx, pbnd)
	if err != nil {
		return path.ImmutablePath{}, err
	}

	return path.FromCid(pbnd.Cid()), nil
}

```

这段代码定义了一个名为func的函数，接收两个参数，一个是指向ObjectAPI类型的整型变量api，另一个是两个路径参数before和after。函数返回一个数组，包含对比前后的coreiface.ObjectChange结构体。

函数的核心逻辑如下：

1. 获取before和after节点，分别从api的core层中解析获取。
2. 使用dagutils.Diff函数计算前后两个节点之间的差异，并返回差异数组。
3. 遍历变化数组，将差异封装为coreiface.ObjectChange结构体并返回。

函数还使用了tracing包来记录函数的上下文信息，以便在函数调用时进行跟踪。函数使用了withAttributes和trace.WithAttributes()方法来设置跟踪参数，并使用了ctx.Span()方法来记录函数的入时间和出时间。函数还使用了defer.Drop()方法来通知函数调用者在函数返回前释放资源。


```
func (api *ObjectAPI) Diff(ctx context.Context, before path.Path, after path.Path) ([]coreiface.ObjectChange, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.ObjectAPI", "Diff", trace.WithAttributes(
		attribute.String("before", before.String()),
		attribute.String("after", after.String()),
	))
	defer span.End()

	beforeNd, err := api.core().ResolveNode(ctx, before)
	if err != nil {
		return nil, err
	}

	afterNd, err := api.core().ResolveNode(ctx, after)
	if err != nil {
		return nil, err
	}

	changes, err := dagutils.Diff(ctx, api.dag, beforeNd, afterNd)
	if err != nil {
		return nil, err
	}

	out := make([]coreiface.ObjectChange, len(changes))
	for i, change := range changes {
		out[i] = coreiface.ObjectChange{
			Type: coreiface.ChangeType(change.Type),
			Path: change.Path,
		}

		if change.Before.Defined() {
			out[i].Before = path.FromCid(change.Before)
		}

		if change.After.Defined() {
			out[i].After = path.FromCid(change.After)
		}
	}

	return out, nil
}

```

这两段代码定义了两个函数：

1. `func (api *ObjectAPI) core() coreiface.CoreAPI`：将 `api` 指向的 `ObjectAPI` 对象包装成一个 `coreiface.CoreAPI` 类型，从而实现将 `api` 的数据输出给其他部分的功能。

2. `func deserializeNode(nd *Node, dataFieldEncoding string) (*dag.ProtoNode, error)`：将 `nd` 指向的 `Node` 对象中的 `dataFieldEncoding` 字段值解析为 `dataFieldEncoding` 常量的对应编码方式，然后尝试使用该编码方式将 `nd` 的数据加载到 `dag.Node` 对象中。这个函数可能会返回一个 `dag.Node` 对象，或者一个错误的 `error` 消息。


```
func (api *ObjectAPI) core() coreiface.CoreAPI {
	return (*CoreAPI)(api)
}

func deserializeNode(nd *Node, dataFieldEncoding string) (*dag.ProtoNode, error) {
	dagnode := new(dag.ProtoNode)
	switch dataFieldEncoding {
	case "text":
		dagnode.SetData([]byte(nd.Data))
	case "base64":
		data, err := base64.StdEncoding.DecodeString(nd.Data)
		if err != nil {
			return nil, err
		}
		dagnode.SetData(data)
	default:
		return nil, fmt.Errorf("unknown data field encoding")
	}

	links := make([]*ipld.Link, len(nd.Links))
	for i, link := range nd.Links {
		c, err := cid.Decode(link.Hash)
		if err != nil {
			return nil, err
		}
		links[i] = &ipld.Link{
			Name: link.Name,
			Size: link.Size,
			Cid:  c,
		}
	}
	if err := dagnode.SetLinks(links); err != nil {
		return nil, err
	}

	return dagnode, nil
}

```

# `/opt/kubo/core/coreapi/path.go`

这段代码定义了一个名为 "coreapi" 的包。它导入了以下依赖项：

- "context"
- "errors"
- "fmt"

- "github.com/ipfs/boxo/namesys"
- "github.com/ipfs/kubo/tracing"

- "github.com/ipfs/otel/attribute"
- "github.com/ipfs/otel/trace"

- "github.com/ipfs/boxo/coreiface"
- "github.com/ipfs/boxo/path"
- "github.com/ipfs/boxo/resolver"
- "github.com/ipfs/boxo/ipld"

package "coreapi" 

通过导入这些依赖项，可以访问 "coreapi" 包中定义的函数和类型。这个包的具体作用没有在代码中详细说明，它可能是用于一个名为 "coreapi" 的项目的开发，或者是出于某种需要而定义的。


```
package coreapi

import (
	"context"
	"errors"
	"fmt"

	"github.com/ipfs/boxo/namesys"
	"github.com/ipfs/kubo/tracing"

	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/trace"

	coreiface "github.com/ipfs/boxo/coreiface"
	"github.com/ipfs/boxo/path"
	ipfspathresolver "github.com/ipfs/boxo/path/resolver"
	ipld "github.com/ipfs/go-ipld-format"
)

```

此代码定义了一个名为 `ResolveNode` 的函数，它使用 `Unixfs` 解析器来查找路径 `p` 并返回其解析后的 Node。函数的实现如下：
go
func (api *CoreAPI) ResolveNode(ctx context.Context, p path.Path) (ipld.Node, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI", "ResolveNode", trace.WithAttributes(attribute.String("path", p.String()))
	defer span.End()

	rp, _, err := api.ResolvePath(ctx, p)
	if err != nil {
		return nil, err
	}

	node, err := api.dag.Get(ctx, rp.RootCid())
	if err != nil {
		return nil, err
	}
	return node, nil
}

此函数的核心逻辑是使用 `api.ResolvePath` 函数查找给定路径的根节点，然后使用 `api.dag.Get` 函数获取该根节点的 `CID` 值并返回。如果解析或获取 Node 过程中出现错误，函数将返回 ` nil` 并捕获相应的错误。函数使用了 `attribute.String("path", p.String())` 类型注解来传递给 `api.ResolvePath` 函数的路径参数 `p`，以便在日志中记录该参数。函数使用了 `trace.WithAttributes` 用于记录函数运行时生成的跟踪属性，例如 `ipld.Node` 类型，以便在将来的调试中跟踪函数的执行情况。


```
// ResolveNode resolves the path `p` using Unixfs resolver, gets and returns the
// resolved Node.
func (api *CoreAPI) ResolveNode(ctx context.Context, p path.Path) (ipld.Node, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI", "ResolveNode", trace.WithAttributes(attribute.String("path", p.String())))
	defer span.End()

	rp, _, err := api.ResolvePath(ctx, p)
	if err != nil {
		return nil, err
	}

	node, err := api.dag.Get(ctx, rp.RootCid())
	if err != nil {
		return nil, err
	}
	return node, nil
}

```

This is a function in the CoreAPI package that resolves a path using the ResolvePath function. It takes a context.Context, a path.Path object and returns the path.ImmutablePath, a slice of strings, and an error.

The function first checks if there is an error with the namesys.Resolve function. If there is an error, it returns a path.ImmutablePath, a slice of strings, and an error. If there is no error, it creates a new path.ImmutablePath by following the ResolveToLastNode function of the ipfspathresolver.Resolver type, and then returns it.

It is important to note that the function only resolves paths that are either IPLD or Unix FSPaths. It will not be able to resolve paths that are not IPLD or Unix FSPaths.


```
// ResolvePath resolves the path `p` using Unixfs resolver, returns the
// resolved path.
func (api *CoreAPI) ResolvePath(ctx context.Context, p path.Path) (path.ImmutablePath, []string, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI", "ResolvePath", trace.WithAttributes(attribute.String("path", p.String())))
	defer span.End()

	res, err := namesys.Resolve(ctx, api.namesys, p)
	if errors.Is(err, namesys.ErrNoNamesys) {
		return path.ImmutablePath{}, nil, coreiface.ErrOffline
	} else if err != nil {
		return path.ImmutablePath{}, nil, err
	}
	p = res.Path

	var resolver ipfspathresolver.Resolver
	switch p.Namespace() {
	case path.IPLDNamespace:
		resolver = api.ipldPathResolver
	case path.IPFSNamespace:
		resolver = api.unixFSPathResolver
	default:
		return path.ImmutablePath{}, nil, fmt.Errorf("unsupported path namespace: %s", p.Namespace())
	}

	imPath, err := path.NewImmutablePath(p)
	if err != nil {
		return path.ImmutablePath{}, nil, err
	}

	node, remainder, err := resolver.ResolveToLastNode(ctx, imPath)
	if err != nil {
		return path.ImmutablePath{}, nil, err
	}

	segments := []string{p.Namespace(), node.String()}
	segments = append(segments, remainder...)

	p, err = path.NewPathFromSegments(segments...)
	if err != nil {
		return path.ImmutablePath{}, nil, err
	}

	imPath, err = path.NewImmutablePath(p)
	if err != nil {
		return path.ImmutablePath{}, nil, err
	}

	return imPath, remainder, nil
}

```

# `/opt/kubo/core/coreapi/pin.go`

这段代码定义了一个名为 "coreapi" 的包，它包含了使用 IPFS(InterPlanetary File System)进行分布式存储的相关组件。它从以下依赖中使用了一些：

- "github.com/ipfs/boxo/blockservice" 和 "github.com/ipfs/boxo/coreiface" 用于从 IPFS 存储桶中读取和写入块信息。
- "github.com/ipfs/boxo/coreiface/options" 和 "github.com/ipfs/boxo/path" 用于配置 IPFS 存储桶的选项和路径。
- "github.com/ipfs/boxo/exchange/offline" 和 "github.com/ipfs/boxo/ipld/merkledag" 用于实现与本地文件系统之间的交互。
- "github.com/ipfs/boxo/ipld/pin" 和 "github.com/ipfs/boxo/path" 用于挂载和创建 IPFS 路径。
- "github.com/ipfs/boxo/path/walker" 和 "github.com/ipfs/boxo/path/data" 用于遍历和访问 IPFS 路径。
- "github.com/ipfs/boxo/transport/typic" 和 "github.com/ipfs/boxo/transport/json" 用于实现 IPFS 的传输层。
- "github.com/ipfs/boxo/as" 和 "github.com/ipfs/boxo/crypto" 用于实现更高级别的 Ascii 到 IPFS 映射。

此包的目的是提供用于管理 IPFS 存储桶的组件，可以用于创建和管理 IPFS 路径，并支持与本地文件系统进行交互。此外，它还支持使用 OpenTelemetry 进行跟踪和日志记录。


```
package coreapi

import (
	"context"
	"fmt"

	bserv "github.com/ipfs/boxo/blockservice"
	coreiface "github.com/ipfs/boxo/coreiface"
	caopts "github.com/ipfs/boxo/coreiface/options"
	offline "github.com/ipfs/boxo/exchange/offline"
	"github.com/ipfs/boxo/ipld/merkledag"
	"github.com/ipfs/boxo/path"
	pin "github.com/ipfs/boxo/pinning/pinner"
	"github.com/ipfs/go-cid"
	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/trace"

	"github.com/ipfs/kubo/tracing"
)

```

这段代码定义了一个名为 PinAPI 的接口，该接口包含一个名为 Add 的方法，用于将一个特定的路径 "path"（在此示例中为 "/api/v1/graph/1234567890/node134567890"）添加到系统中。

该方法接受一个名为 opts 的 caopts 参数，该参数是一个或多个传递给 PinAPI.Add 方法的选项。在方法内部，首先会使用 PinAPI.core() 函数查找给定路径的节点，然后设置具有 "recursive" 属性的选项，设置此选项为 true 时，将开始递归查找所有与给定路径相对应的节点。然后设置 caopts 中的 Recursive 选项为 true，设置结束条件为没有错误，然后继续添加给定路径的节点。

接着，使用 api.pinning.Pin 函数将给定路径的节点添加到系统中，并递归地处理子节点。然后使用 api.provider.Provide 函数提供给系统使用，无论是否有错误。最后，使用 api.pinning.Flush 函数输出最后的日志。


```
type PinAPI CoreAPI

func (api *PinAPI) Add(ctx context.Context, p path.Path, opts ...caopts.PinAddOption) error {
	ctx, span := tracing.Span(ctx, "CoreAPI.PinAPI", "Add", trace.WithAttributes(attribute.String("path", p.String())))
	defer span.End()

	dagNode, err := api.core().ResolveNode(ctx, p)
	if err != nil {
		return fmt.Errorf("pin: %s", err)
	}

	settings, err := caopts.PinAddOptions(opts...)
	if err != nil {
		return err
	}

	span.SetAttributes(attribute.Bool("recursive", settings.Recursive))

	defer api.blockstore.PinLock(ctx).Unlock(ctx)

	err = api.pinning.Pin(ctx, dagNode, settings.Recursive)
	if err != nil {
		return fmt.Errorf("pin: %s", err)
	}

	if err := api.provider.Provide(dagNode.Cid()); err != nil {
		return err
	}

	return api.pinning.Flush(ctx)
}

```

此函数的作用是调用PinAPI的`pinLsAll`方法，使用传递给该函数的`opts`选项设置选项，然后返回API调用返回的响应。

具体来说，函数接收一个`PinAPI`类型的参数`api`，然后使用`tracing.Span`函数记录一个名为"CoreAPI.PinAPI"的跨临界区，并使用`caopts.PinLsOptions`函数将传递给`opts`的选项设置为`PinLsOption`类型。如果设置选项有错误，函数将返回一个非空错误。

接下来，函数将`settings`设置为`PinLsOption`类型的参数，根据传递的`opts`设置设置属性的`attribute.String`设置其值为`settings.Type`。然后，函数根据`settings.Type`选择正确的选项，并使用`api.pinLsAll`函数的`ctx`和`settings`作为参数调用`pinLsAll`方法。

最后，函数使用`span.End`来关闭记录的跨临界区，并返回设置的`attribute.String`值，如果没有错误，则返回`nil`。


```
func (api *PinAPI) Ls(ctx context.Context, opts ...caopts.PinLsOption) (<-chan coreiface.Pin, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.PinAPI", "Ls")
	defer span.End()

	settings, err := caopts.PinLsOptions(opts...)
	if err != nil {
		return nil, err
	}

	span.SetAttributes(attribute.String("type", settings.Type))

	switch settings.Type {
	case "all", "direct", "indirect", "recursive":
	default:
		return nil, fmt.Errorf("invalid type '%s', must be one of {direct, indirect, recursive, all}", settings.Type)
	}

	return api.pinLsAll(ctx, settings.Type), nil
}

```

此函数的作用是验证传入的路径和选项中是否存在与API的配置文件中指定的选项中包含的设置。如果存在，则返回相应的设置，否则返回false和错误。

具体来说，函数接收一个指向PinAPI结构的指针（api），以及一个路径（p）和一些选项（opts...caopts.PinIsPinnedOption）。函数先在子句中使用tracing记录开启一个名为"CoreAPI.PinAPI"的跟踪，并将path和opts作为附件跟踪。然后使用api的core()函数的resolvePath函数查找path指定的路径，并将结果存储在resolved变量中。

接下来，函数创建一个名为settings的选项切片，并使用它设置生成的设置（通过...caopts.PinIsPinnedOption传递的）。然后设置跟踪使用生成的设置的属性，并使用api的pinning.IsPinnedWithType函数在ctx中检查路径指定的设置是否与api的配置文件中指定的设置匹配。

最后，函数返回设置是否成功，成功则返回true，否则返回false和错误。


```
func (api *PinAPI) IsPinned(ctx context.Context, p path.Path, opts ...caopts.PinIsPinnedOption) (string, bool, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.PinAPI", "IsPinned", trace.WithAttributes(attribute.String("path", p.String())))
	defer span.End()

	resolved, _, err := api.core().ResolvePath(ctx, p)
	if err != nil {
		return "", false, fmt.Errorf("error resolving path: %s", err)
	}

	settings, err := caopts.PinIsPinnedOptions(opts...)
	if err != nil {
		return "", false, err
	}

	span.SetAttributes(attribute.String("withtype", settings.WithType))

	mode, ok := pin.StringToMode(settings.WithType)
	if !ok {
		return "", false, fmt.Errorf("invalid type '%s', must be one of {direct, indirect, recursive, all}", settings.WithType)
	}

	return api.pinning.IsPinnedWithType(ctx, resolved.RootCid(), mode)
}

```

这段代码定义了一个名为 `Rm` 的函数，属于名为 `PinAPI` 的枚举类型。

该函数接收一个名为 `ctx` 的上下文，一个名为 `p` 的路径参数，以及一个包含 `opts` 的选项参数。`opts` 是一个 `PinRmOption` 数组，其中包含了各种选项，例如 `Recursive` 选项，它指定了是否进行递归操作。

函数内部首先尝试使用 `api.core().ResolvePath` 函数来查找路径 `p` 所对应的对象，如果失败，则返回错误。然后，根据 `opts...` 循环遍历 `opts` 中的每个选项，并设置相应的设置。设置完选项后，继续调用 `api.pinning.Unpin` 和 `api.pinning.Flush` 函数，最后释放锁。

函数的作用是帮助您通过 `PinAPI` 实现对指定路径 `p` 的 Unpin（解除 pin） 和 Flush（刷到块存储器）操作。


```
// Rm pin rm api
func (api *PinAPI) Rm(ctx context.Context, p path.Path, opts ...caopts.PinRmOption) error {
	ctx, span := tracing.Span(ctx, "CoreAPI.PinAPI", "Rm", trace.WithAttributes(attribute.String("path", p.String())))
	defer span.End()

	rp, _, err := api.core().ResolvePath(ctx, p)
	if err != nil {
		return err
	}

	settings, err := caopts.PinRmOptions(opts...)
	if err != nil {
		return err
	}

	span.SetAttributes(attribute.Bool("recursive", settings.Recursive))

	// Note: after unpin the pin sets are flushed to the blockstore, so we need
	// to take a lock to prevent a concurrent garbage collection
	defer api.blockstore.PinLock(ctx).Unlock(ctx)

	if err = api.pinning.Unpin(ctx, rp.RootCid(), settings.Recursive); err != nil {
		return err
	}

	return api.pinning.Flush(ctx)
}

```

此函数名为 `Update`，属于 `PinAPI` 类型。它接收一个名为 `api` 的引用，以及一个名为 `from` 的路径参数和一个名为 `to` 的路径参数，还可以传递多个名为 `opts` 的参数。函数的作用是更新指定路径的 `from` 和 `to` 路径下的 `pin` 对象，并可选地设置 `unpin` 选项。

具体来说，函数首先根据传递的 `opts` 参数创建一个 `PinUpdateOptions` 对象，然后使用 `span` 记录函数调用，并设置一些跟踪属性。接着使用 `api.core().ResolvePath` 函数求出 `from` 和 `to` 路径的根 `cid`，并创建一个 `ResolvePathResult` 对象，将其作为跟踪的 `result` 字段。

接着，函数调用 `api.pinning.Update` 函数更新指定路径下的 `pin` 对象，并设置 `unpin` 选项（如果已存在）。最后，函数调用 `api.pinning.Flush` 函数清除之前的跟踪，并返回更新操作的错误。


```
func (api *PinAPI) Update(ctx context.Context, from path.Path, to path.Path, opts ...caopts.PinUpdateOption) error {
	ctx, span := tracing.Span(ctx, "CoreAPI.PinAPI", "Update", trace.WithAttributes(
		attribute.String("from", from.String()),
		attribute.String("to", to.String()),
	))
	defer span.End()

	settings, err := caopts.PinUpdateOptions(opts...)
	if err != nil {
		return err
	}

	span.SetAttributes(attribute.Bool("unpin", settings.Unpin))

	fp, _, err := api.core().ResolvePath(ctx, from)
	if err != nil {
		return err
	}

	tp, _, err := api.core().ResolvePath(ctx, to)
	if err != nil {
		return err
	}

	defer api.blockstore.PinLock(ctx).Unlock(ctx)

	err = api.pinning.Update(ctx, fp.RootCid(), tp.RootCid(), settings.Unpin)
	if err != nil {
		return err
	}

	return api.pinning.Flush(ctx)
}

```

这段代码定义了一个名为`pinStatus`的结构体，它表示了在Pin网络中`/dev/pin`设备的`/私/signal`输出端点上的状态信息。

下面是对该结构体的定义：

go
type pinStatus struct {
	err      error
	cid      cid.Cid
	ok       bool
	badNodes []coreiface.BadPinNode
}


该结构体定义了`err`、`cid`、`ok`和`badNodes`这四个成员变量。其中：

* `err`成员变量表示与`/dev/pin`设备进行交互时发生的错误。
* `cid`成员变量是一个`cid.Cid`类型的变量，它表示了与`/dev/pin`设备进行交互时使用的唯一ID。
* `ok`成员变量是一个布尔类型的变量，它表示`/dev/pin`设备上`/私/signal`输出端点是否正常工作。
* `badNodes`成员变量是一个由`coreiface.BadPinNode`类型组成的数组，它表示了在`/dev/pin`设备上，由于一些错误导致无法正常工作的`/私/signal`输出端点。

另外，该结构体还包含一个名为`Ok()`的成员函数，它的作用是返回`pinStatus`结构体中`ok`成员变量的值。


```
type pinStatus struct {
	err      error
	cid      cid.Cid
	ok       bool
	badNodes []coreiface.BadPinNode
}

// BadNode is used in PinVerifyRes
type badNode struct {
	path path.ImmutablePath
	err  error
}

func (s *pinStatus) Ok() bool {
	return s.ok
}

```

这是一段用于检测是否有 BadNodes 的函数，如果有则返回。函数接收一个指向 pinStatus 类型的参数 s，并返回 s.badNodes 类型的数组。

函数接收一个指向 badNode 类型的参数 n，并返回 n.err 类型的错误。

函数内部，针 n 包含一个指向 badNode 的类型，函数返回这个针的路径，通过这个路径可以恢复 badNode 的某些元数据。


```
func (s *pinStatus) BadNodes() []coreiface.BadPinNode {
	return s.badNodes
}

func (s *pinStatus) Err() error {
	return s.err
}

func (n *badNode) Path() path.ImmutablePath {
	return n.path
}

func (n *badNode) Err() error {
	return n.err
}

```

This is a function that creates a `pinStatus` struct to hold information about the current pin. It has an `ok` field that indicates whether the pin is valid, as well as a `badNodes` field for each node that is considered bad (e.g. because it is not being used or has failed). It also has a `cid` field that identifies the pin.

The function has an additional layer of logging using the `trace.WithAttributes` method to add an `cid` attribute to the current pin's `badNodes` field.

The function checks the pin by making a recursive call to `getLinks` and then either using or creating a new `pinStatus` object based on the result of that call. If the pin is valid, the function returns it. If it is not valid, the function creates a new `pinStatus` object with the appropriate fields set.

The function also has a `Visited` function that adds the current pin to the `visited` map so that it can be safely checked again in the future.


```
func (api *PinAPI) Verify(ctx context.Context) (<-chan coreiface.PinStatus, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.PinAPI", "Verify")
	defer span.End()

	visited := make(map[cid.Cid]*pinStatus)
	bs := api.blockstore
	DAG := merkledag.NewDAGService(bserv.New(bs, offline.Exchange(bs)))
	getLinks := merkledag.GetLinksWithDAG(DAG)

	var checkPin func(root cid.Cid) *pinStatus
	checkPin = func(root cid.Cid) *pinStatus {
		ctx, span := tracing.Span(ctx, "CoreAPI.PinAPI", "Verify.CheckPin", trace.WithAttributes(attribute.String("cid", root.String())))
		defer span.End()

		if status, ok := visited[root]; ok {
			return status
		}

		links, err := getLinks(ctx, root)
		if err != nil {
			status := &pinStatus{ok: false, cid: root}
			status.badNodes = []coreiface.BadPinNode{&badNode{path: path.FromCid(root), err: err}}
			visited[root] = status
			return status
		}

		status := &pinStatus{ok: true, cid: root}
		for _, lnk := range links {
			res := checkPin(lnk.Cid)
			if !res.ok {
				status.ok = false
				status.badNodes = append(status.badNodes, res.badNodes...)
			}
		}

		visited[root] = status
		return status
	}

	out := make(chan coreiface.PinStatus)
	go func() {
		defer close(out)
		for p := range api.pinning.RecursiveKeys(ctx) {
			var res *pinStatus
			if p.Err != nil {
				res = &pinStatus{err: p.Err}
			} else {
				res = checkPin(p.C)
			}
			select {
			case <-ctx.Done():
				return
			case out <- res:
			}
		}
	}()

	return out, nil
}

```

该代码定义了一个名为 "pinInfo" 的结构体类型，该类型包含三个字段：pinType、path 和 err。

pinInfo.pinType 字段表示了该结构的 pin 类型，path.ImmutablePath 字段表示了该结构的路径，而 err 字段则表示了该结构所代表的错误。

该代码还定义了三个方法：

* pinInfo.Path() 方法返回了该结构体中 pinType 字段的值，作为 path.ImmutablePath 类型的路径。
* pinInfo.Type() 方法返回了该结构体中 pinType 字段的值，作为 string 类型的字符串。
* pinInfo.Err() 方法返回了该结构体中 err 字段的值，作为 error 类型的字面值。


```
type pinInfo struct {
	pinType string
	path    path.ImmutablePath
	err     error
}

func (p *pinInfo) Path() path.ImmutablePath {
	return p.path
}

func (p *pinInfo) Type() string {
	return p.pinType
}

func (p *pinInfo) Err() error {
	return p.err
}

```

This is a function that implements the `pinInfo` struct to return a PinInfo object.

This function takes in a stream of CIDs, which are emitted by the node, and a set of directories that are忽视 by the node.

It returns a PinInfo object, which contains information about the pins that have been processed and the status of those pins.

The function works as follows:

1. It initializes a set of emitted CIDs and a set of directories to ignore.
2. It walks through the directories, skipping over any CIDs that have already been emitted.
3. For each directory, it attempts to add the CID to the set of emitted CIDs.
4. If the CID is successfully added to the set of emitted CIDs, it skips over the next set of CIDs.
5. If the CID is not successfully added to the set of emitted CIDs, it emits an error and skips over the next set of CIDs.
6. It重复 steps 3-5 for each directory.
7. If the function is called with `"indirect"` or `"all"` as its argument, it walks through the CIDs and adds them to the set of emitted CIDs.

This function is useful for implementing a pinning system for Indirect Proof of Stake (PoS) networks.


```
// pinLsAll is an internal function for returning a list of pins
//
// The caller must keep reading results until the channel is closed to prevent
// leaking the goroutine that is fetching pins.
func (api *PinAPI) pinLsAll(ctx context.Context, typeStr string) <-chan coreiface.Pin {
	out := make(chan coreiface.Pin, 1)

	emittedSet := cid.NewSet()

	AddToResultKeys := func(c cid.Cid, typeStr string) error {
		if emittedSet.Visit(c) {
			select {
			case out <- &pinInfo{
				pinType: typeStr,
				path:    path.FromCid(c),
			}:
			case <-ctx.Done():
				return ctx.Err()
			}
		}
		return nil
	}

	go func() {
		defer close(out)

		var rkeys []cid.Cid
		var err error
		if typeStr == "recursive" || typeStr == "all" {
			for streamedCid := range api.pinning.RecursiveKeys(ctx) {
				if streamedCid.Err != nil {
					out <- &pinInfo{err: streamedCid.Err}
					return
				}
				if err = AddToResultKeys(streamedCid.C, "recursive"); err != nil {
					out <- &pinInfo{err: err}
					return
				}
				rkeys = append(rkeys, streamedCid.C)
			}
		}
		if typeStr == "direct" || typeStr == "all" {
			for streamedCid := range api.pinning.DirectKeys(ctx) {
				if streamedCid.Err != nil {
					out <- &pinInfo{err: streamedCid.Err}
					return
				}
				if err = AddToResultKeys(streamedCid.C, "direct"); err != nil {
					out <- &pinInfo{err: err}
					return
				}
			}
		}
		if typeStr == "indirect" {
			// We need to first visit the direct pins that have priority
			// without emitting them

			for streamedCid := range api.pinning.DirectKeys(ctx) {
				if streamedCid.Err != nil {
					out <- &pinInfo{err: streamedCid.Err}
					return
				}
				emittedSet.Add(streamedCid.C)
			}

			for streamedCid := range api.pinning.RecursiveKeys(ctx) {
				if streamedCid.Err != nil {
					out <- &pinInfo{err: streamedCid.Err}
					return
				}
				emittedSet.Add(streamedCid.C)
				rkeys = append(rkeys, streamedCid.C)
			}
		}
		if typeStr == "indirect" || typeStr == "all" {
			walkingSet := cid.NewSet()
			for _, k := range rkeys {
				err = merkledag.Walk(
					ctx, merkledag.GetLinksWithDAG(api.dag), k,
					func(c cid.Cid) bool {
						if !walkingSet.Visit(c) {
							return false
						}
						if emittedSet.Has(c) {
							return true // skipped
						}
						err := AddToResultKeys(c, "indirect")
						if err != nil {
							out <- &pinInfo{err: err}
							return false
						}
						return true
					},
					merkledag.SkipRoot(), merkledag.Concurrent(),
				)
				if err != nil {
					out <- &pinInfo{err: err}
					return
				}
			}
		}
	}()

	return out
}

```

该函数`func`接收一个名为`api`的`PinAPI`类型参数，并返回一个指向`CoreAPI`类型的指针。

`coreiface.CoreAPI`是一个接口，定义了`CoreAPI`类型的接口，该接口包含了与系统交互的所有核心功能。

函数体中，首先将`api`作为参数传递给`(*CoreAPI)(api)`，然后返回该返回值的类型指针，即`CoreAPI`。

由于`api`是`PinAPI`类型，它实现了`CoreAPI`接口，因此`(*CoreAPI)(api)`返回的结果就是`api`的`CoreAPI`类型。通过将`api`作为参数传递给`func`，函数可以安全地操作`PinAPI`类型的`api`，同时又可以安全地操作`CoreAPI`类型的`api`。


```
func (api *PinAPI) core() coreiface.CoreAPI {
	return (*CoreAPI)(api)
}

```