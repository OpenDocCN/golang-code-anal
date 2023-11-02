# go-ipfs 源码解析 45

# `gc/gc.go`

这段代码定义了一个名为 "gc" 的包，该包提供 Go-IPFS 中的垃圾回收功能。它主要通过以下几个组件来实现：

1. 导入需要的 "gc" 包。
2. 通过 `import` 语句导入需要的子包，如 `bserv`、`bstore`、`offline` 等。
3. 在包定义中定义了一些常量和变量，如 `context`、`errors`、`fmt` 等。
4. 通过 `import` 语句导入需要的导入。
5. 在 `gc` 包内部，定义了一些方法，如 `NewGarbageCollector`、`Run` 等。
6. 在 `gc` 包内部，定义了一些子模块，如 `blockstore`、`blockstore_輔導`、`ctx_init`、`pin_init` 等。
7. 通过 `import` 语句导入了一些外部的包，如 `cid`、`dstore`、`ipld`、`logging` 等。
8. 在 `gc` 包内部，定义了一些自定义的错误类型，如 `GarbageCollectorError` 等。


```go
// Package gc provides garbage collection for go-ipfs.
package gc

import (
	"context"
	"errors"
	"fmt"
	"strings"

	bserv "github.com/ipfs/boxo/blockservice"
	bstore "github.com/ipfs/boxo/blockstore"
	offline "github.com/ipfs/boxo/exchange/offline"
	dag "github.com/ipfs/boxo/ipld/merkledag"
	pin "github.com/ipfs/boxo/pinning/pinner"
	"github.com/ipfs/boxo/verifcid"
	cid "github.com/ipfs/go-cid"
	dstore "github.com/ipfs/go-datastore"
	ipld "github.com/ipfs/go-ipld-format"
	logging "github.com/ipfs/go-log"
)

```

这段代码定义了一个名为 "gc" 的日志输出变量 log，并使用 Logger.Logger() 函数为其添加了名为 "gc" 的日志级别。这通常用于输出在垃圾回收过程中产生的对象。

定义了一个名为 "Result" 的结构体类型，它包含一个名为 "KeyRemoved" 的整数类型字段，它表示从垃圾回收中移除的对象的键ID，以及一个名为 "Error" 的错误类型字段，它包含一个具体的错误对象。

定义了一个名为 "toRawCids" 的函数，它接收一个名为 "set" 的 cid.Set 类型的参数，并返回一个名为 "newSet" 的 cid.Set 类型和一个名为 "err" 的错误类型的变量。该函数使用 set.ForEach() 函数将传入的 set 中的每个元素，将其键名为 "RawCid" 的新键添加到新集中，并将该键的哈希值添加到新集中。

最后，该代码块没有做任何其他事情，所以它不会产生任何输出。


```go
var log = logging.Logger("gc")

// Result represents an incremental output from a garbage collection
// run.  It contains either an error, or the cid of a removed object.
type Result struct {
	KeyRemoved cid.Cid
	Error      error
}

// converts a set of CIDs with different codecs to a set of CIDs with the raw codec.
func toRawCids(set *cid.Set) (*cid.Set, error) {
	newSet := cid.NewSet()
	err := set.ForEach(func(c cid.Cid) error {
		newSet.Add(cid.NewCidV1(cid.Raw, c.Hash()))
		return nil
	})
	return newSet, err
}

```

This is a Go function that deletes a block of storage in a specific data store based on its CID (Cannon CID). It is using the `go-pointer` library to handle the deletion.

The function takes a `context` and a key (`k`) as input. It checks if the key is present in the data store, and if not, it deletes the block. If the key is present, it checks if the block is valid in the data store, and if not, it returns an error.

It also note that it assumes that all CIDs returned by the `keychannel` are `raw` CIDs, meaning they are not formatted yet.

It uses a loop to iterate over all blocks in the data store, and for each block, it checks if it is valid and if it is not, it returns an error.

It also uses a loop to iterate over all blocks that have been removed, and if it finds a valid block it continues the loop, otherwise it returns.

It also uses a loop to collect the blocks in the data store, and if it finds an error it returns.

It concludes the function by returning the output of the `gcs.CreateBlock` and `gcs.DeleteBlock` method.


```go
// GC performs a mark and sweep garbage collection of the blocks in the blockstore
// first, it creates a 'marked' set and adds to it the following:
// - all recursively pinned blocks, plus all of their descendants (recursively)
// - bestEffortRoots, plus all of its descendants (recursively)
// - all directly pinned blocks
// - all blocks utilized internally by the pinner
//
// The routine then iterates over every block in the blockstore and
// deletes any block that is not found in the marked set.
func GC(ctx context.Context, bs bstore.GCBlockstore, dstor dstore.Datastore, pn pin.Pinner, bestEffortRoots []cid.Cid) <-chan Result {
	ctx, cancel := context.WithCancel(ctx)

	unlocker := bs.GCLock(ctx)

	bsrv := bserv.New(bs, offline.Exchange(bs))
	ds := dag.NewDAGService(bsrv)

	output := make(chan Result, 128)

	go func() {
		defer cancel()
		defer close(output)
		defer unlocker.Unlock(ctx)

		gcs, err := ColoredSet(ctx, pn, ds, bestEffortRoots, output)
		if err != nil {
			select {
			case output <- Result{Error: err}:
			case <-ctx.Done():
			}
			return
		}

		// The blockstore reports raw blocks. We need to remove the codecs from the CIDs.
		gcs, err = toRawCids(gcs)
		if err != nil {
			select {
			case output <- Result{Error: err}:
			case <-ctx.Done():
			}
			return
		}

		keychan, err := bs.AllKeysChan(ctx)
		if err != nil {
			select {
			case output <- Result{Error: err}:
			case <-ctx.Done():
			}
			return
		}

		errors := false
		var removed uint64

	loop:
		for ctx.Err() == nil { // select may not notice that we're "done".
			select {
			case k, ok := <-keychan:
				if !ok {
					break loop
				}
				// NOTE: assumes that all CIDs returned by the keychan are _raw_ CIDv1 CIDs.
				// This means we keep the block as long as we want it somewhere (CIDv1, CIDv0, Raw, other...).
				if !gcs.Has(k) {
					err := bs.DeleteBlock(ctx, k)
					removed++
					if err != nil {
						errors = true
						select {
						case output <- Result{Error: &CannotDeleteBlockError{k, err}}:
						case <-ctx.Done():
							break loop
						}
						// continue as error is non-fatal
						continue loop
					}
					select {
					case output <- Result{KeyRemoved: k}:
					case <-ctx.Done():
						break loop
					}
				}
			case <-ctx.Done():
				break loop
			}
		}
		if errors {
			select {
			case output <- Result{Error: ErrCannotDeleteSomeBlocks}:
			case <-ctx.Done():
				return
			}
		}

		gds, ok := dstor.(dstore.GCDatastore)
		if !ok {
			return
		}

		err = gds.CollectGarbage(ctx)
		if err != nil {
			select {
			case output <- Result{Error: err}:
			case <-ctx.Done():
			}
			return
		}
	}()

	return output
}

```

This is a Go function that implements the IPFS PIN (Plaintext Inlay Node) protocol. It appears to be responsible for adding nodes to the IPFS network and maintaining the desired data structure.

Here's a high-level overview of the function:

1. The function accepts a `map[cid.Cid]` object, which represents the nodes in the IPFS network.
2. The function first checks if the input `c` is valid by checking against a defined allowlist. If the input `c` is invalid, an error is returned.
3. If the `c` is valid, the function attempts to retrieve the links to the node `c`. This is done by calling the `getLinks` function, which takes a context and the `c` object as arguments.
4. If the `getLinks` function returns an error, a more detailed error message is generated and logged.
5. The function then enters a loop that repeatedly waits for new events to occur. If an event does not occur within the given time, the function attempts to retrieve the node with the highest index in the `c` object. This is done by calling the `dag.Walk` function, which is responsible for traversing the node tree and adding nodes to the `c` object.
6. If an error occurs during the `dag.Walk` call, the function attempts to recover by calling the `verboseCidError` function, which attempts to generate a more detailed error message if the input `err` is an error related to the minimum hash length or an insecure hash function.
7. The loop continues until the `ctx.Done()` signal is received, at which point the function returns any errors encountered.

It's worth noting that the `ipfs` package used in this function is Version 0.4.13, which has been deprecated since November 2021. You should consider updating your dependencies to the latest version of this package.


```go
// Descendants recursively finds all the descendants of the given roots and
// adds them to the given cid.Set, using the provided dag.GetLinks function
// to walk the tree.
func Descendants(ctx context.Context, getLinks dag.GetLinks, set *cid.Set, roots <-chan pin.StreamedCid) error {
	verifyGetLinks := func(ctx context.Context, c cid.Cid) ([]*ipld.Link, error) {
		err := verifcid.ValidateCid(verifcid.DefaultAllowlist, c)
		if err != nil {
			return nil, err
		}

		return getLinks(ctx, c)
	}

	verboseCidError := func(err error) error {
		if strings.Contains(err.Error(), verifcid.ErrBelowMinimumHashLength.Error()) ||
			strings.Contains(err.Error(), verifcid.ErrPossiblyInsecureHashFunction.Error()) {
			err = fmt.Errorf("\"%s\"\nPlease run 'ipfs pin verify'"+ // nolint
				" to list insecure hashes. If you want to read them,"+
				" please downgrade your go-ipfs to 0.4.13\n", err)
			log.Error(err)
		}
		return err
	}

	for {
		select {
		case <-ctx.Done():
			return ctx.Err()
		case wrapper, ok := <-roots:
			if !ok {
				return nil
			}
			if wrapper.Err != nil {
				return wrapper.Err
			}

			// Walk recursively walks the dag and adds the keys to the given set
			err := dag.Walk(ctx, verifyGetLinks, wrapper.C, func(k cid.Cid) bool {
				return set.Visit(toCidV1(k))
			}, dag.Concurrent())
			if err != nil {
				err = verboseCidError(err)
				return err
			}
		}
	}
}

```

This is a Go function that appears to fetch all links related to a given CID. It uses the IPLD-iOS SDK to perform the operation and returns either the generated Goal故障报告， the links that were successfully fetched, or an error if an error occurred.

Here is the function signature:

func FetchAndDescendants(ctx context.Context,
	bestEffortGetLinks GoalSeqFn,
	gcs             *path.Path,
	bestEffortRoots  []pin.Cid,
	bestEffortRootsChan *pin.RWCloser,
	err              error) (map[pin.Cid]pin.Cid, error)

The function takes a `context.Context` and a `Go拒不定` function as its arguments,

It has three arguments,

* `bestEffortGetLinks` is a function that returns a `Result` struct that contains a `Error` field if an error occurred or the success status if the operation was successful.
* `gcs`  is a `path.Path` struct that is used to store the root directory of the IPLD-iOS SDK.
* `bestEffortRoots` is a slice of `pin.Cid` structs, which represents the CIDs of the nodes that are considered to have the highest level of performance.
* `bestEffortRootsChan` is a `pin.RWCloser` struct, which is used to manage read-write operations on the `bestEffortRoots` slice.

It returns either a map of `pin.Cid` structs that represent the nodes that were successfully fetched, or an error if an error occurred.

The function first initializes the function's variables,

* `ctx` is the context that the function will use to perform the operation.
* `bestEffortGetLinks` is the function that will be used to fetch the links.
* `gcs` is the root directory of the IPLD-iOS SDK.
* `bestEffortRoots` is a slice of `pin.Cid` structs, which represents the CIDs of the nodes that are considered to have the highest level of performance.
* `bestEffortRootsChan` is a `pin.RWCloser` struct, which is used to manage read-write operations on the `bestEffortRoots` slice.

It then performs the main logic of the function.

* It first checks if there is an error that has been encountered.
* If there is no error, it attempts to fetch all the links related to the CIDs in the `bestEffortRoots` slice using the `bestEffortGetLinks` function.
* If an error occurs while fetching the links, it is handled by the `Descendants` function, which is responsible for cleaning up the scene.
* If the fetch operation is successful, it returns the generated Goal error, the list of links that were successfully fetched, or an error if an error occurred.

It closes the `bestEffortRootsChan` byectimating the ring buffer, which is done using the `<>` operator.


```go
// toCidV1 converts any CIDv0s to CIDv1s.
func toCidV1(c cid.Cid) cid.Cid {
	if c.Version() == 0 {
		return cid.NewCidV1(c.Type(), c.Hash())
	}
	return c
}

// ColoredSet computes the set of nodes in the graph that are pinned by the
// pins in the given pinner.
func ColoredSet(ctx context.Context, pn pin.Pinner, ng ipld.NodeGetter, bestEffortRoots []cid.Cid, output chan<- Result) (*cid.Set, error) {
	// KeySet currently implemented in memory, in the future, may be bloom filter or
	// disk backed to conserve memory.
	errors := false
	gcs := cid.NewSet()
	getLinks := func(ctx context.Context, cid cid.Cid) ([]*ipld.Link, error) {
		links, err := ipld.GetLinks(ctx, ng, cid)
		if err != nil {
			errors = true
			select {
			case output <- Result{Error: &CannotFetchLinksError{cid, err}}:
			case <-ctx.Done():
				return nil, ctx.Err()
			}
		}
		return links, nil
	}
	rkeys := pn.RecursiveKeys(ctx)
	err := Descendants(ctx, getLinks, gcs, rkeys)
	if err != nil {
		errors = true
		select {
		case output <- Result{Error: err}:
		case <-ctx.Done():
			return nil, ctx.Err()
		}
	}

	bestEffortGetLinks := func(ctx context.Context, cid cid.Cid) ([]*ipld.Link, error) {
		links, err := ipld.GetLinks(ctx, ng, cid)
		if err != nil && !ipld.IsNotFound(err) {
			errors = true
			select {
			case output <- Result{Error: &CannotFetchLinksError{cid, err}}:
			case <-ctx.Done():
				return nil, ctx.Err()
			}
		}
		return links, nil
	}
	bestEffortRootsChan := make(chan pin.StreamedCid)
	go func() {
		defer close(bestEffortRootsChan)
		for _, root := range bestEffortRoots {
			select {
			case <-ctx.Done():
				return
			case bestEffortRootsChan <- pin.StreamedCid{C: root}:
			}
		}
	}()
	err = Descendants(ctx, bestEffortGetLinks, gcs, bestEffortRootsChan)
	if err != nil {
		errors = true
		select {
		case output <- Result{Error: err}:
		case <-ctx.Done():
			return nil, ctx.Err()
		}
	}

	dkeys := pn.DirectKeys(ctx)
	for k := range dkeys {
		if k.Err != nil {
			return nil, k.Err
		}
		gcs.Add(toCidV1(k.C))
	}

	ikeys := pn.InternalPins(ctx)
	err = Descendants(ctx, getLinks, gcs, ikeys)
	if err != nil {
		errors = true
		select {
		case output <- Result{Error: err}:
		case <-ctx.Done():
			return nil, ctx.Err()
		}
	}

	if errors {
		return nil, ErrCannotFetchAllLinks
	}

	return gcs, nil
}

```

这段代码定义了两个错误对象：ErrCannotFetchAllLinks和ErrCannotDeleteSomeBlocks。它们都提供了错误信息，并返回一个错误对象（error）给调用者。

ErrCannotFetchAllLinks错误在尝试创建一个标记集合时失败，并返回一个名为“无法获取所有链接”的错误。

ErrCannotDeleteSomeBlocks错误在尝试删除已经被标记为删除的块时失败，并返回一个名为“无法完成删除某些块”的错误。

ErrcannotcantFetchLinksError是一个结构体，它包含一个名为“cid”的元组字段，该字段指定了要获取链接的块的CID，以及一个名为“Err”的错误字段，其中包含错误发生的详细信息，该信息可以在GC输出中看到。


```go
// ErrCannotFetchAllLinks is returned as the last Result in the GC output
// channel when there was an error creating the marked set because of a
// problem when finding descendants.
var ErrCannotFetchAllLinks = errors.New("garbage collection aborted: could not retrieve some links")

// ErrCannotDeleteSomeBlocks is returned when removing blocks marked for
// deletion fails as the last Result in GC output channel.
var ErrCannotDeleteSomeBlocks = errors.New("garbage collection incomplete: could not delete some blocks")

// CannotFetchLinksError provides detailed information about which links
// could not be fetched and can appear as a Result in the GC output channel.
type CannotFetchLinksError struct {
	Key cid.Cid
	Err error
}

```

这段代码定义了一个名为Error的接口，该接口包含一个名为CannotFetchLinksError的类型，该类型包含一个与错误相关的消息。然后，该接口实现了一个名为Error的类型，该类型包含一个CannotFetchLinksError类型的实例，该实例包含一个键（CannotFetchLinksError）和一个错误消息（fmt.Sprintf("could not retrieve links for %s: %s", e.Key, e.Err)。

同时，该代码定义了一个名为CannotDeleteBlockError的类型，该类型包含一个与错误相关的消息，该消息可以出现在垃圾回收（GC）输出中。在该类型中，使用了一个名为Error的接口，该接口包含一个CannotDeleteBlockError类型的实例，该实例包含一个键（CannotDeleteBlockError）和一个错误消息（fmt.Sprintf("could not delete blocks for %s: %s", e.Key, e.Err)。


```go
// Error implements the error interface for this type with a useful
// message.
func (e *CannotFetchLinksError) Error() string {
	return fmt.Sprintf("could not retrieve links for %s: %s", e.Key, e.Err)
}

// CannotDeleteBlockError provides detailed information about which
// blocks could not be deleted and can appear as a Result in the GC output
// channel.
type CannotDeleteBlockError struct {
	Key cid.Cid
	Err error
}

// Error implements the error interface for this type with a
```

这是一段 Go 语言中的函数指针。函数指针是一种特殊的函数，它存储了一个接收者类型的参数和一个错误类型的参数。接收者类型的参数是一个指针，而不是一个普通变量。

在这段代码中，函数指针 `e` 接收两个参数：一个是字符串 `e.Key`，另一个是错误类型的参数 `e.Err`。函数指针使用 `fmt.Sprintf` 函数来创建一个字符串，该字符串包含错误消息。

函数指针返回一个字符串，该字符串格式化显示了 `e.Key` 和 `e.Err` 的值。


```go
// useful message.
func (e *CannotDeleteBlockError) Error() string {
	return fmt.Sprintf("could not remove %s: %s", e.Key, e.Err)
}

```

# `gc/gc_test.go`

该代码是一个 Go 语言项目，名为 "gc"，包含以下主要部分：

1. 导入 "github.com/ipfs/boxo/blockservice" 和 "github.com/ipfs/boxo/blockstore"，这两个库与链存储和节点存储相关。
2. 导入 "github.com/ipfs/boxo/exchange/offline"，这个库与链的 off-chain 交易相关。
3. 导入 "github.com/ipfs/boxo/ipld/merkledag"，这个库与 Merkle 树相关。
4. 导入 "github.com/ipfs/boxo/ipld/test"，这个库与测试相关。
5. 导入 "github.com/ipfs/boxo/pinning/pinner"，这个库与键链中的节点相关。
6. 导入 "github.com/ipfs/boxo/pinning/pinner/dspinner"，这个库与键链中的节点相关。
7. 导入 "github.com/ipfs/go-cid"，这个库与万事可乐 (IPFS) 客户端相关。
8. 导入 "github.com/ipfs/go-datastore"，这个库与万事可乐 (IPFS) 相关数据存储服务。
9. 导入 "github.com/multiformats/go-multihash"，这个库与 Merkle 树相关。
10. 导入 "github.com/stretchr/testify/assert"，这个库用于测试。

根据上述分析，此代码的主要作用是测试万事可乐 (IPFS) 客户端与相关库的交互作用。


```go
package gc

import (
	"context"
	"testing"

	"github.com/ipfs/boxo/blockservice"
	"github.com/ipfs/boxo/blockstore"
	"github.com/ipfs/boxo/exchange/offline"
	"github.com/ipfs/boxo/ipld/merkledag"
	mdutils "github.com/ipfs/boxo/ipld/merkledag/test"
	pin "github.com/ipfs/boxo/pinning/pinner"
	"github.com/ipfs/boxo/pinning/pinner/dspinner"
	"github.com/ipfs/go-cid"
	"github.com/ipfs/go-datastore"
	dssync "github.com/ipfs/go-datastore/sync"
	"github.com/multiformats/go-multihash"
	"github.com/stretchr/testify/require"
)

```

This is a test function for a "DAG consistency checker" that checks that the output of the consistency checker is correct. The function uses the "multihash" package to implement a hash table and the "daggen" package to generate DAG nodes.

The function takes in several parameters:

* `ctx`: the context used by the `daggen` package
* `bs`: the Buffer Simulator from the "multihash" package
* `ds`: the DAG Simulator from the "multihash" package
* `pinner`: a instance of the "pinner" package for pinning
* `bestEffortRoots`: a slice of DAG nodes with "best effort" roots

The function checks that the output of the consistency checker is a valid hash table that contains all expected keys and values.

Here are the steps that the function follows:

1. Generates a DAG tree with "best effort" roots
2. Checks that the output of the consistency checker contains all expected keys and values
3. The output of the consistency checker should have the expected number of keys and values
4. The output of the consistency checker should contain the expected keys and values in the correct order
5. The output of the consistency checker should not have any missing keys or values
6. The output of the consistency checker should not have any keys or values that have not been processed yet

The function uses the "pinner" package to pin the DAG nodes with "recursive" mode, which guarantees that the output of the consistency checker will be GCed (Generated from Memory).


```go
func TestGC(t *testing.T) {
	ctx := context.Background()

	ds := dssync.MutexWrap(datastore.NewMapDatastore())
	bs := blockstore.NewGCBlockstore(blockstore.NewBlockstore(ds), blockstore.NewGCLocker())
	bserv := blockservice.New(bs, offline.Exchange(bs))
	dserv := merkledag.NewDAGService(bserv)
	pinner, err := dspinner.New(ctx, ds, dserv)
	require.NoError(t, err)

	daggen := mdutils.NewDAGGenerator()

	var expectedKept []multihash.Multihash
	var expectedDiscarded []multihash.Multihash

	// add some pins
	for i := 0; i < 5; i++ {
		// direct
		root, _, err := daggen.MakeDagNode(dserv.Add, 0, 1)
		require.NoError(t, err)
		err = pinner.PinWithMode(ctx, root, pin.Direct)
		require.NoError(t, err)
		expectedKept = append(expectedKept, root.Hash())

		// recursive
		root, allCids, err := daggen.MakeDagNode(dserv.Add, 5, 2)
		require.NoError(t, err)
		err = pinner.PinWithMode(ctx, root, pin.Recursive)
		require.NoError(t, err)
		expectedKept = append(expectedKept, toMHs(allCids)...)
	}

	err = pinner.Flush(ctx)
	require.NoError(t, err)

	// add more dags to be GCed
	for i := 0; i < 5; i++ {
		_, allCids, err := daggen.MakeDagNode(dserv.Add, 5, 2)
		require.NoError(t, err)
		expectedDiscarded = append(expectedDiscarded, toMHs(allCids)...)
	}

	// and some other as "best effort roots"
	var bestEffortRoots []cid.Cid
	for i := 0; i < 5; i++ {
		root, allCids, err := daggen.MakeDagNode(dserv.Add, 5, 2)
		require.NoError(t, err)
		bestEffortRoots = append(bestEffortRoots, root)
		expectedKept = append(expectedKept, toMHs(allCids)...)
	}

	ch := GC(ctx, bs, ds, pinner, bestEffortRoots)
	var discarded []multihash.Multihash
	for res := range ch {
		require.NoError(t, res.Error)
		discarded = append(discarded, res.KeyRemoved.Hash())
	}

	allKeys, err := bs.AllKeysChan(ctx)
	require.NoError(t, err)
	var kept []multihash.Multihash
	for key := range allKeys {
		kept = append(kept, key.Hash())
	}

	require.ElementsMatch(t, expectedDiscarded, discarded)
	require.ElementsMatch(t, expectedKept, kept)
}

```

该函数将一个元素列表中的 CID 值转换为相应的 Multihash 值，并将结果存储在一个名为 "res" 的多哈希数组中。

函数接受一个名为 "cids" 的元素列表，其中每个元素都是 CID 类型。函数遍历 "cids" 列表中的每个元素，将其 CID 值存储在名为 "res" 的多哈希数组中的对应索引位置。函数返回 "res" 数组，其中每个元素都是 Multihash 类型。

从函数的实现中可以看出，该函数主要用于将给定的元素列表的每个元素与一个唯一的 Multihash 值相关联，以便后续的操作和比较。


```go
func toMHs(cids []cid.Cid) []multihash.Multihash {
	res := make([]multihash.Multihash, len(cids))
	for i, c := range cids {
		res[i] = c.Hash()
	}
	return res
}

```

## init system integration

go-ipfs can be started by your operating system's native init system.

- [systemd](#systemd)
- [LSB init script](#initd)
- [Upstart/startup job](#upstart)
- [launchd](#launchd)

### systemd

For `systemd`, the best approach is to run the daemon in a user session. Here is a sample service file:

```gosystemd
[Unit]
Description=IPFS daemon

[Service]
# Environment="IPFS_PATH=/data/ipfs"  # optional path to ipfs init directory if not default ($HOME/.ipfs)
ExecStart=/usr/local/bin/ipfs daemon
Restart=on-failure

[Install]
WantedBy=default.target
```

To run this in your user session, save it as `~/.config/systemd/user/ipfs.service` (creating directories as necessary). Once you run `ipfs init` to create your IPFS settings, you can control the daemon using the following commands:

* `systemctl --user start ipfs` - start the daemon
* `systemctl --user stop ipfs` - stop the daemon
* `systemctl --user status ipfs` - get status of the daemon
* `systemctl --user enable ipfs` - enable starting the daemon at boot
* `systemctl --user disable ipfs` - disable starting the daemon at boot

*Note:* If you want this `--user` service to run at system boot, you must [`enable-linger`](http://www.freedesktop.org/software/systemd/man/loginctl.html) on the account that runs the service:

```go
# loginctl enable-linger [user]
```
Read more about `--user` services here: [wiki.archlinux.org:Systemd ](https://wiki.archlinux.org/index.php/Systemd/User#Automatic_start-up_of_systemd_user_instances)

### initd

- Here is a full-featured sample service file: https://github.com/dylanPowers/ipfs-linux-service/blob/master/init.d/ipfs
- Use `service` or your distribution's equivalent to control the service.

##  upstart

- And below is a very basic sample upstart job. **Note the username jbenet**.

```go
cat /etc/init/ipfs.conf
```
```go
description "ipfs: interplanetary filesystem"

start on (local-filesystems and net-device-up IFACE!=lo)
stop on runlevel [!2345]

limit nofile 524288 1048576
limit nproc 524288 1048576
setuid jbenet
chdir /home/jbenet
respawn
exec ipfs daemon
```

Another version is available here:

```gosh
ipfs cat /ipfs/QmbYCwVeA23vz6mzAiVQhJNa2JSiRH4ebef1v2e5EkDEZS/ipfs.conf >/etc/init/ipfs.conf
```

For both, edit to replace occurrences of `jbenet` with whatever user you want it to run as:

```gosh
sed -i s/jbenet/<chosen-username>/ /etc/init/ipfs.conf
```

Once you run `ipfs init` to create your IPFS settings, you can control the daemon using the `init.d` commands:

```gosh
sudo service ipfs start
sudo service ipfs stop
sudo service ipfs restart
...
```

## launchd

Similar to `systemd`, on macOS you can run `go-ipfs` via a user LaunchAgent.

- Create `~/Library/LaunchAgents/io.ipfs.go-ipfs.plist`:

```goxml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>KeepAlive</key>
        <true/>
        <key>Label</key>
        <string>io.ipfs.go-ipfs</string>
        <key>ProcessType</key>
        <string>Background</string>
        <key>ProgramArguments</key>
        <array>
                <string>/bin/sh</string>
                <string>-c</string>
                <string>~/go/bin/ipfs daemon</string>
        </array>
        <key>RunAtLoad</key>
        <true/>
</dict>
</plist>
```
The reason for running `ipfs` under a shell is to avoid needing to hard-code the user's home directory in the job.

- To start the job, run `launchctl load ~/Library/LaunchAgents/io.ipfs.go-ipfs.plist`

Notes:

- To check that the job is running, run `launchctl list | grep ipfs`.
- IPFS should now start whenever you log in (and exit when you log out).
- [LaunchControl](http://www.soma-zone.com/LaunchControl/) is a GUI tool which simplifies management of LaunchAgents.


# ipfs launchd agent

A bare-bones launchd agent file for ipfs. To have launchd automatically run the ipfs daemon for you, run `./misc/launchd/install.sh`



# `p2p/listener.go`

这段代码定义了一个名为p2p的包，其中定义了一个名为Listener的接口。

Listener接口表示一个用于监听连接并将其转发到目标的主机。通过import语句，该包引入了erros、sync和protocol库。

该代码中的Listener接口定义了三个方法：Protocol、ListenAddress和TargetAddress。其中，Protocol()返回所监听的协议ID,ListenAddress()返回用于监听的地址，TargetAddress()返回目标地址。这三个方法都是通过ma.Multiaddr类型定义的，表示该接口支持的目标地址是任意ma.Multiaddr类型。

该接口还定义了一个名为key()的额外方法，用于获取监听器的关键，这个方法在整个接口中可见，即使它没有被实现。

最后，该接口还定义了一个close()方法，用于关闭监听器。这个方法不会影响已经连接的流，可以在关闭监听器之前或同时调用。


```go
package p2p

import (
	"errors"
	"sync"

	p2phost "github.com/libp2p/go-libp2p/core/host"
	net "github.com/libp2p/go-libp2p/core/network"
	"github.com/libp2p/go-libp2p/core/protocol"
	ma "github.com/multiformats/go-multiaddr"
)

// Listener listens for connections and proxies them to a target.
type Listener interface {
	Protocol() protocol.ID
	ListenAddress() ma.Multiaddr
	TargetAddress() ma.Multiaddr

	key() protocol.ID

	// close closes the listener. Does not affect child streams
	close()
}

```

这段代码定义了一个名为 Listeners 的 struct，表示一个管理一组 Listener 实体的抽象类。Listeners 实现了两个关键方法：newListenersLocal 和 newListenersP2P。

newListenersLocal 的实现创建了一个名为 Listeners 的 struct，该 struct 包含一个 map 类型的 Listeners 对象。map 类型代表一组键值对，键是协议 ID，值是实现了 Listener 接口的Listener 对象。

newListenersP2P 的实现创建了一个名为 Listeners 的 struct，该 struct 实现了两个名为 newListenersLocal 和 newListenersP2P 的方法。newListenersLocal 和 newListenersP2P 都接受一个参数，分别是主机（host）和一个用于 P2P 通信的主机（p2phost）。

newListenersLocal 的实现创建了一个名为 Listeners 的 struct，该 struct 包含一个 map 类型的 Listeners 对象。map 类型代表一组键值对，键是协议 ID，值是实现了 Listener 接口的Listener 对象。实现了 Listener 接口的Listener 对象通过 handleStream 函数与从主机到客户端的 P2P 连接建立连接。

newListenersP2P 的实现创建了一个名为 Listeners 的 struct，该 struct 包含一个 map 类型的 Listeners 对象。map 类型代表一组键值对，键是协议 ID，值是实现了 Listener 接口的Listener 对象。实现了 Listener 接口的Listener 对象通过 handleStream 函数与从客户端到主机的 P2P 连接建立连接。


```go
// Listeners manages a group of Listener implementations,
// checking for conflicts and optionally dispatching connections.
type Listeners struct {
	sync.RWMutex

	Listeners map[protocol.ID]Listener
}

func newListenersLocal() *Listeners {
	return &Listeners{
		Listeners: map[protocol.ID]Listener{},
	}
}

func newListenersP2P(host p2phost.Host) *Listeners {
	reg := &Listeners{
		Listeners: map[protocol.ID]Listener{},
	}

	host.SetStreamHandlerMatch("/x/", func(p protocol.ID) bool {
		reg.RLock()
		defer reg.RUnlock()

		_, ok := reg.Listeners[p]
		return ok
	}, func(stream net.Stream) {
		reg.RLock()
		defer reg.RUnlock()

		l := reg.Listeners[stream.Protocol()]
		if l != nil {
			go l.(*remoteListener).handleStream(stream)
		}
	})

	return reg
}

```

这段代码定义了两个函数，分别注册和关闭注册表中的监听器。

函数1：`Register`
此函数接收一个`Listener`实例作为参数，并将其注册到注册表中。如果注册失败，函数返回一个`error`。首先，函数创建一个名为`r.Lock`的互斥锁，以确保在函数内部对注册表的修改操作互斥可见。然后，函数遍历已注册的监听器，检查它们是否已注册。如果是，函数返回一个`error`。否则，函数创建一个新监听器实例，并将其添加到注册表中。最后，函数解锁互斥锁并返回操作结果。

函数2：`Close`
此函数接收一个函数作为参数，该函数接收一个`Listener`实例作为参数。如果函数返回`true`，则关闭所有监听器，否则循环并关闭最后一个监听器。首先，函数创建一个名为`todo`的缓冲区，用于存储未关闭的监听器。然后，函数获取所有已注册的监听器并遍历它们。对于每个已注册的监听器，如果函数返回`true`，则删除它并从`todo`缓冲区中添加它。否则，函数关闭该监听器。最后，函数遍历`todo`缓冲区并关闭所有监听器。函数返回`todo`中监听器的数量。


```go
// Register registers listenerInfo into this registry and starts it.
func (r *Listeners) Register(l Listener) error {
	r.Lock()
	defer r.Unlock()

	if _, ok := r.Listeners[l.key()]; ok {
		return errors.New("listener already registered")
	}

	r.Listeners[l.key()] = l
	return nil
}

func (r *Listeners) Close(matchFunc func(listener Listener) bool) int {
	todo := make([]Listener, 0)
	r.Lock()
	for _, l := range r.Listeners {
		if !matchFunc(l) {
			continue
		}

		if _, ok := r.Listeners[l.key()]; ok {
			delete(r.Listeners, l.key())
			todo = append(todo, l)
		}
	}
	r.Unlock()

	for _, l := range todo {
		l.close()
	}

	return len(todo)
}

```

# `p2p/local.go`

这段代码定义了一个名为"p2p"的包，其中定义了一个名为"localListener"的函数。该函数使用Go标准库中的"context"和"time"函数以及第三方库"github.com/jbenet/go-temp-err-catcher"、"github.com/libp2p/go-libp2p/core/network"、"github.com/libp2p/go-libp2p/core/peer"、"github.com/multiformats/go-multiaddr"和"github.com/multiformats/go-multiaddr/net"。

函数接受一个名为"context"的上下文，然后使用"time.Duration"类型的变量"duration"来记录进入函数的时间和当前时间之间的差值。接下来，使用"go.驳ify.Func"类型将一个接收者设置为"localListener"函数的回调函数，并使用Go标准库中的"os"函数创建一个当前目录并设置为回调函数的执行上下文。

然后，使用"github.com/jbenet/go-temp-err-catcher"库中的函数"ten世的恩惠"来捕获并处理在函数中发生的错误，并使用"context.With"+"context.WithCh"+"context.WithError""随着"context.With"短路到上下文"WithError"中。

接下来，使用"github.com/libp2p/go-libp2p/core/network"库中的"ListenAll"函数创建一个TCP套接字并绑定到"localhost:2380"端口，用于接收通过TCP套接字发送的流量。然后，使用"ma"库中的"multiAddr"类型将TCP套接字中的本地地址添加到"multiAddr"结构体中，并将其设置为"net.IPv4:127.0.0.1:10000"。

最后，使用"github.com/multiformats/go-multiaddr/net"库中的"M十个地址"创建一个TCP套接字并绑定到"localhost:2380"端口，将上面创建的TCP套接字中的本地地址添加到"multiAddr"结构体中，并将其设置为"net.IPv4:127.0.0.1:10000"。然后，使用Go标准库中的"context.With"+"context.WithCh"+"context.WithError""随着"context.With"短路到上下文"WithError"中。

函数最后返回，没有做任何其他事情。


```go
package p2p

import (
	"context"
	"time"

	tec "github.com/jbenet/go-temp-err-catcher"
	net "github.com/libp2p/go-libp2p/core/network"
	"github.com/libp2p/go-libp2p/core/peer"
	"github.com/libp2p/go-libp2p/core/protocol"
	ma "github.com/multiformats/go-multiaddr"
	manet "github.com/multiformats/go-multiaddr/net"
)

// localListener manet streams and proxies them to libp2p services.
```

这段代码定义了一个名为 `localListener` 的 struct，表示一个本地 P2P 流到远程监听者的流。该 struct 包含以下字段：

- `ctx`: 当前上下文上下文。
- `p2p`: 指向 P2P 对象的指针。
- `proto`: P2P 协议的 ID。
- `laddr`: 远程监听者的地址，是一个 `ma.Multiaddr` 类型的变量。
- `peer`: 对等程序的 ID，是一个 `peer.ID` 类型的变量。
- `listener`: 一个自定义的 `manet.Listener` 类型的变量，用于监听本地连接。
- `ForwardLocal` 函数： 将创建一个新的 P2P 流到远程监听者，并返回监听器和错误。

`ForwardLocal` 函数接收两个参数，一个是 `ctx` 上下文上下文，另一个是远程监听者的对等程序的 ID 和 P2P 协议的 ID。函数首先创建一个 `localListener` 类型的实例，并将其注册到本地监听器上下文中。然后，函数使用 `p2p.ListenersLocal.Register` 方法将远程监听器注册到本地监听器上下文中。最后，函数使用go 子句来启动本地监听器对象的 `acceptConns` 方法，从而等待连接的到来。

如果需要输出 `ForwardLocal` 函数的源代码，可以使用以下方式：


func (p2p *P2P) ForwardLocal(ctx context.Context, peer peer.ID, proto protocol.ID, bindAddr ma.Multiaddr) (Listener, error) {
	listener := &localListener{
		ctx:   ctx,
		p2p:   p2p,
		proto: proto,
		peer:  peer,
		listener: manet.Listener(func(ch manet.Listener, a ma.Multiaddr) (net.Listener, error) {
			return true, nil
		}),
	}

	maListener, err := manet.Listen(bindAddr)
	if err != nil {
		return nil, err
	}

	listener.listener = maListener
	listener.laddr = maListener.Multiaddr()

	if err := p2p.ListenersLocal.Register(listener); err != nil {
		return nil, err
	}

	go listener.acceptConns()

	return listener, nil
}


```go
type localListener struct {
	ctx context.Context

	p2p *P2P

	proto protocol.ID
	laddr ma.Multiaddr
	peer  peer.ID

	listener manet.Listener
}

// ForwardLocal creates new P2P stream to a remote listener.
func (p2p *P2P) ForwardLocal(ctx context.Context, peer peer.ID, proto protocol.ID, bindAddr ma.Multiaddr) (Listener, error) {
	listener := &localListener{
		ctx:   ctx,
		p2p:   p2p,
		proto: proto,
		peer:  peer,
	}

	maListener, err := manet.Listen(bindAddr)
	if err != nil {
		return nil, err
	}

	listener.listener = maListener
	listener.laddr = maListener.Multiaddr()

	if err := p2p.ListenersLocal.Register(listener); err != nil {
		return nil, err
	}

	go listener.acceptConns()

	return listener, nil
}

```

这两段代码是讨论网络中的对等网络（peer-to-peer network）中的本地监听器（local listener）函数。该函数接受连接并返回一个网络流和一个错误。

函数参数：

* `l`：一个指向 `localListener` 的引用。
* `ctx`：一个 `context.Context`，表示当前操作的上下文。
* `time.Second*30`：延迟，以秒为单位。
* `cancel`：一个 `context.Cancel` 返回的 `context.Context` 表示取消延迟。

函数实现：

1. 在 `func (l *localListener) dial(ctx context.Context) (net.Stream, error)` 中，首先创建一个 `context.WithTimeout` 函数，设置一个以 `time.Second*30` 为延迟的延迟。`WithTimeout` 函数返回一个新的 `context.Context`，其中包含延迟。
2. 在 `cctx, cancel := context.WithTimeout(ctx, time.Second*30)` 中，创建了一个新的 `context.Cancel` 返回的 `context.Context`，该 `context.Context` 以延迟为设置，并且传递给 `WithTimeout` 函数。
3. 接着，在 `l.p2p.peerHost.NewStream(cctx, l.peer, l.proto)` 中，创建一个新的网络流，并使用 `l.p2p.peerHost` 字段获取该网络的 `localStream` 字段，然后使用 `NewStream` 函数创建一个新的网络流。最后，将 `cctx` 和 `l.peer` 和 `l.proto` 传递给 `NewStream` 函数。
4. 在 `func (l *localListener) acceptConns()` 中，设置一个循环，等待新的连接。
5. 在循环中，使用 `l.listener.Accept()` 函数获取当前正在等待的连接。
6. 如果连接出现错误，如 `tec.ErrIsTemporary(err)` 返回一个非空错误，函数将返回，并跳过当前循环。
7. 否则，函数将继续运行，并设置一个Go函数 `l.setupStream(local)`，用于设置本地套接字。


```go
func (l *localListener) dial(ctx context.Context) (net.Stream, error) {
	cctx, cancel := context.WithTimeout(ctx, time.Second*30) // TODO: configurable?
	defer cancel()

	return l.p2p.peerHost.NewStream(cctx, l.peer, l.proto)
}

func (l *localListener) acceptConns() {
	for {
		local, err := l.listener.Accept()
		if err != nil {
			if tec.ErrIsTemporary(err) {
				continue
			}
			return
		}

		go l.setupStream(local)
	}
}

```

这段代码定义了一个名为 `func` 的函数，接收一个名为 `localListener` 的第二个参数，以及一个名为 `manet.Conn` 的本地连接对象。

函数首先尝试使用 `localListener` 初始化连接到远程服务器 `l.peer` 的 `manet.Conn`，如果初始化失败，则关闭本地连接并输出一条警告信息。如果初始化成功，函数创建一个名为 `Stream` 的 `manet.Stream` 对象，设置其协议为 `l.proto`，并设置其来源地址为 `local.RemoteMultiaddr` 和目标地址为 `l.peer`，同时设置 `Local` 和 `Remote` 字段为 `localListener` 和 `manet.Conn`，最后将 `Stream` 注册到 `localListener` 的 `p2p.Streams` 注册表中。

函数的作用是创建一个远程流并将其注册到本地监听器中，以便在需要时可以开始监听远程服务器发送的数据。


```go
func (l *localListener) setupStream(local manet.Conn) {
	remote, err := l.dial(l.ctx)
	if err != nil {
		local.Close()
		log.Warnf("failed to dial to remote %s/%s", l.peer, l.proto)
		return
	}

	stream := &Stream{
		Protocol: l.proto,

		OriginAddr: local.RemoteMultiaddr(),
		TargetAddr: l.TargetAddress(),
		peer:       l.peer,

		Local:  local,
		Remote: remote,

		Registry: l.p2p.Streams,
	}

	l.p2p.Streams.Register(stream)
}

```

这是一段 Go 语言中的函数指针，定义了一个名为 "func" 的函数，接收一个名为 "localListener" 的类型为 "localListener" 的参数。

函数的作用是关闭监听器(由 "l.listener" 引用)，并返回监听器使用的协议 ID(由 "l.proto" 引用)。

函数还返回监听器的发送方地址(由 "l.laddr" 引用)，以及由 "l.peer" 引用的目标地址(目标服务器)。


```go
func (l *localListener) close() {
	l.listener.Close()
}

func (l *localListener) Protocol() protocol.ID {
	return l.proto
}

func (l *localListener) ListenAddress() ma.Multiaddr {
	return l.laddr
}

func (l *localListener) TargetAddress() ma.Multiaddr {
	addr, err := ma.NewMultiaddr(maPrefix + l.peer.String())
	if err != nil {
		panic(err)
	}
	return addr
}

```

该函数名为 `func`，接受一个名为 `l` 的本地监听器 `localListener`，并返回一个名为 `protocol.ID` 的协议 ID。

函数的作用是获取并返回一个协议 ID，用于标识网络消息的发送方或接收方。

函数的实现首先获取并获取远程监听器的连接地址，然后使用该地址的 `String()` 方法获取一个字符串，作为协议 ID 的值，最后将该值作为 `protocol.ID` 返回。


```go
func (l *localListener) key() protocol.ID {
	return protocol.ID(l.ListenAddress().String())
}

```

# `p2p/p2p.go`

该代码是一个名为"p2p"的包，它定义了一个名为"P2P"的结构体，用于表示当前正在运行的流/监听器。

该结构体包含以下字段：

- ListenersLocal：一个包含当前正在运行的流的监听器。
- ListenersP2P：一个包含正在等待连接的流的监听器。
- Streams：一个注册当前正在运行的流的名称。

该结构体还包含一个名为"identity"的标识字段，该字段包含一个指向一个peer.ID的指针，用于标识自己。另外，该结构体还包含一个指向一个p2phost.Host的指针和一个指向一个pstore.Peerstore的指针，用于连接到远程主机和远程存储。

最后，该结构体还定义了一个名为"log"的Logger，用于输出当前的日志信息。


```go
package p2p

import (
	logging "github.com/ipfs/go-log"
	p2phost "github.com/libp2p/go-libp2p/core/host"
	"github.com/libp2p/go-libp2p/core/peer"
	pstore "github.com/libp2p/go-libp2p/core/peerstore"
	"github.com/libp2p/go-libp2p/core/protocol"
)

var log = logging.Logger("p2p-mount")

// P2P structure holds information on currently running streams/Listeners.
type P2P struct {
	ListenersLocal *Listeners
	ListenersP2P   *Listeners
	Streams        *StreamRegistry

	identity  peer.ID
	peerHost  p2phost.Host
	peerstore pstore.Peerstore
}

```

该代码定义了一个名为 `New` 的函数，该函数创建了一个新的 P2P 结构体。

该函数需要三个参数，分别为 `identity`、`peerHost` 和 `peerstore`。其中，`identity` 是用于标识该 P2P 实例的唯一 ID，`peerHost` 是 P2P 实例的远程主机，`peerstore` 是用于存储 P2P 实例的远程存储库。

函数的实现中，首先定义了一个名为 `ListenersLocal` 的函数，该函数返回一个 `ListenersLocal` 类型的指针，该指针实现了 `Listeners` 接口的 `Local` 方法。接着定义了一个名为 `ListenersP2P` 的函数，该函数返回一个 `ListenersP2P` 类型的指针，该指针实现了 `Listeners` 接口的 `P2P` 方法。

接下来，定义了一个名为 `Streams` 的结构体，该结构体包含一个 `Streams` 字段和一个 `ConnManager` 字段和一个 `conns` 字段。其中，`Streams` 字段是一个 `map[uint64]*Stream` 类型的字段，`ConnManager` 字段是一个 `peer.ID` 类型字段，`conns` 字段是一个 `map[peer.ID]int` 类型的字段。

最后，该函数返回了一个新的 `P2P` 实例，该实例包含上述定义的 `ListenersLocal`、`ListenersP2P` 和 `Streams` 字段。


```go
// New creates new P2P struct.
func New(identity peer.ID, peerHost p2phost.Host, peerstore pstore.Peerstore) *P2P {
	return &P2P{
		identity:  identity,
		peerHost:  peerHost,
		peerstore: peerstore,

		ListenersLocal: newListenersLocal(),
		ListenersP2P:   newListenersP2P(peerHost),

		Streams: &StreamRegistry{
			Streams:     map[uint64]*Stream{},
			ConnManager: peerHost.ConnManager(),
			conns:       map[peer.ID]int{},
		},
	}
}

```

这段代码定义了一个名为`CheckProtocExists`的函数，其接收一个名为`proto`的协议ID，并返回`true`或`false`。函数的作用是检查在`mux handler`中，是否已注册了一个名为`proto`的协议处理程序。

具体来说，函数首先获取一个`protos`变量，该变量存储了`mux handler`中的所有协议。然后，函数遍历`protos`，检查当前是否等于`proto`。如果是，则返回`true`，否则继续遍历。最后，如果所有的`proto`处理程序都已注册，则返回`false`。


```go
// CheckProtoExists checks whether a proto handler is registered to
// mux handler.
func (p2p *P2P) CheckProtoExists(proto protocol.ID) bool {
	protos := p2p.peerHost.Mux().Protocols()

	for _, p := range protos {
		if p != proto {
			continue
		}
		return true
	}
	return false
}

```

# `p2p/remote.go`

这段代码定义了一个名为 "p2p" 的包，其中包含了一些定义、函数和变量。以下是该包的主要作用：

1. 定义了一个名为 "maPrefix" 的常量，该常量使用 libp2p 协议的 IPFS 方法将 Multi-Authority Prefix 格式化，并将其前缀添加到包名中。

2. 定义了一个名为 "remoteListener" 的函数，该函数接受 libp2p 流，将其转发到名为 "manet" 的名为 "manet" 的网络接口上，然后将其传递给该接口的 "listen" 函数。

3. 在 "remoteListener" 函数中，通过调用 "net" 包中的 "net.ListenEndpoint" 函数，创建一个 Endpoint 类型，将其设置为远程监听器地址，然后在该端口的 "listen" 函数中添加一个 "ma" 协议的流量。

4. 在 "remoteListener" 函数中，通过调用 "fmt" 函数中的 "Printf" 函数，将远程监听器 IP 地址和端口格式化并输出到控制台。


```go
package p2p

import (
	"context"
	"fmt"

	net "github.com/libp2p/go-libp2p/core/network"
	"github.com/libp2p/go-libp2p/core/protocol"
	ma "github.com/multiformats/go-multiaddr"
	manet "github.com/multiformats/go-multiaddr/net"
)

var maPrefix = "/" + ma.ProtocolWithCode(ma.P_IPFS).Name + "/"

// remoteListener accepts libp2p streams and proxies them to a manet host.
```

这是一个 Go 语言中的 struct 类型，表示一个用于远程连接的 P2P（Peer-to-Peer）监听器。

struct 类型 remoteListener 包含以下字段：

* p2p：一个指向 P2P 结构的指针，用于管理与远程设备的通信。
* protocol.ID：应用程序协议标识符（通常是 0x12345678 或 0xabcdefgh）。
* addr：一个 ma.Multiaddr 类型的地址，用于将传入的连接地址映射到远程设备。
* reportRemote：一个布尔值，用于设置或取消发送 "<base58 远程对等方 ID>" 消息。当设置为 true 时，会在将数据转发前发送此消息。

remoteListener 结构体可以用来创建和管理 P2P 监听器，以接收和发送数据到远程设备。


```go
type remoteListener struct {
	p2p *P2P

	// Application proto identifier.
	proto protocol.ID

	// Address to proxy the incoming connections to
	addr ma.Multiaddr

	// reportRemote if set to true makes the handler send '<base58 remote peerid>\n'
	// to target before any data is forwarded
	reportRemote bool
}

// ForwardRemote creates new p2p listener.
```

该函数定义了一个名为"ForwardRemote"的函数，接收一个名为"P2P"的上下文对象、一个名为"proto"的协议ID和一个名为"addr"的"Multiaddr"结构体，以及一个名为"reportRemote"的布尔值。

函数的主要作用是创建一个名为"listener"的"Listener"对象，使用接收到的"P2P"上下文对象将消息发送到接收者。函数还可以设置接收者报告发送的远程消息的"reportRemote"设置为true或false。

具体来说，函数首先创建一个名为"listener"的"Listener"对象，该对象的初始值为接收者。"p2p"上下文对象被传递给该对象的"p2p"字段，接收者设置为接收者。"proto"字段用于设置接收者正在等待的消息的协议ID。"addr"字段用于设置接收者地址。

接下来，函数使用"p2p.ListenersP2P.Register"方法将接收者注册到"P2P"上下文对象的"ListenersP2P"链表中。如果此注册操作出现错误，函数将返回一个非空错误对象。

最后，函数返回名为"listener"的"Listener"对象和一个非空错误对象。


```go
func (p2p *P2P) ForwardRemote(ctx context.Context, proto protocol.ID, addr ma.Multiaddr, reportRemote bool) (Listener, error) {
	listener := &remoteListener{
		p2p: p2p,

		proto: proto,
		addr:  addr,

		reportRemote: reportRemote,
	}

	if err := p2p.ListenersP2P.Register(listener); err != nil {
		return nil, err
	}

	return listener, nil
}

```

该函数接收一个远程监听器 (l *remoteListener)和一个远程网络流 (remote net.Stream)。它的作用是连接远程监听器与远程服务器，并监听来自服务器的数据流。

具体来说，函数首先尝试使用manet.Dial函数与远程服务器建立连接。如果连接建立成功，则函数会从远程服务器获取对端计算机的IP地址。接下来，函数会连接远程服务器，并获取与之连接的客户端的IP地址。

如果函数在获取客户端IP地址时出现错误，函数会将远程服务器关闭，并返回。

如果函数配置了l.reportRemote为true，函数会将远程服务器和客户端的IP地址打印到本地。

函数还将ma.NewMultiaddr作为参数，尝试使用动态主机设置协议 (DHCP) 获取一个客户端到服务器的多地址。如果函数在创建MA时出现错误，函数会将远程服务器关闭，并返回。

最后，函数使用l.p2p.Streams注册到远程服务器。注册后，函数将开始监听来自服务器的数据流，并将这些数据流传递给l.p2p.Streams的注册函数。


```go
func (l *remoteListener) handleStream(remote net.Stream) {
	local, err := manet.Dial(l.addr)
	if err != nil {
		_ = remote.Reset()
		return
	}

	peer := remote.Conn().RemotePeer()

	if l.reportRemote {
		if _, err := fmt.Fprintf(local, "%s\n", peer); err != nil {
			_ = remote.Reset()
			return
		}
	}

	peerMa, err := ma.NewMultiaddr(maPrefix + peer.String())
	if err != nil {
		_ = remote.Reset()
		return
	}

	stream := &Stream{
		Protocol: l.proto,

		OriginAddr: peerMa,
		TargetAddr: l.addr,
		peer:       peer,

		Local:  local,
		Remote: remote,

		Registry: l.p2p.Streams,
	}

	l.p2p.Streams.Register(stream)
}

```

这段代码定义了两个函数，分别用于返回一个远程监听者的协议ID和目标地址。

第一个函数 `func (l *remoteListener) Protocol() protocol.ID` 接收一个远程监听者 `l`，并返回其协议ID。函数的实现比较简单，直接将远程监听者的 `proto` 字段作为参数返回即可。

第二个函数 `func (l *remoteListener) ListenAddress() ma.Multiaddr` 同样接收远程监听者 `l`，并返回其目标地址。函数的实现比较复杂，需要创建一个 `ma.Multiaddr` 对象，其中 `maPrefix` 是前缀，用于标识目标地址的前缀。函数首先尝试从远程监听者的 `p2p.identity` 字段中构建目标地址，如果失败则 `panic` 并输出错误信息。最后，函数返回 `l.addr`，即远程监听者的目标地址。


```go
func (l *remoteListener) Protocol() protocol.ID {
	return l.proto
}

func (l *remoteListener) ListenAddress() ma.Multiaddr {
	addr, err := ma.NewMultiaddr(maPrefix + l.p2p.identity.String())
	if err != nil {
		panic(err)
	}
	return addr
}

func (l *remoteListener) TargetAddress() ma.Multiaddr {
	return l.addr
}

```

这段代码定义了一个名为`func`的函数，接收一个名为`l`的远程监听器参数，并返回一个名为`remoteListener`的远程监听器类型。

进一步地，

func (l *remoteListener) key() protocol.ID {

return l.proto

}函数返回一个名为`protocol.ID`的类型，它是远程监听器`l`的协议ID。

函数内部使用了`*remoteListener` 获取远程监听器`l`的引用，然后通过远程监听器`l`的`proto`字段获取其协议ID，最后返回该协议ID。


```go
func (l *remoteListener) close() {}

func (l *remoteListener) key() protocol.ID {
	return l.proto
}

```

# `p2p/stream.go`

这段代码定义了一个名为"p2p"的包，其中包含了以下几个主要组件：

1. `import`语句，引入了所需的库，包括：`io`、`sync`、`github.com/libp2p/go-libp2p/core/connmgr`、`github.com/libp2p/go-libp2p/core/network`、`github.com/libp2p/go-libp2p/core/peer`、`github.com/libp2p/go-libp2p/core/protocol`、`github.com/multiformats/go-multiaddr`和`github.com/multiformats/go-multiaddr/net`。

2. `const`语句，定义了一个名为"cmgrTag"的常量，值为"stream-fwd"。

3. `package`语句，定义了该包的名称。

4. `import`语句，引入了`github.com/multiformats/go-multiaddr`库，定义了一个名为"manet"的包。

5. `const`语句，定义了一个名为"cmgrTag"的常量，值为"stream-fwd"。

6. `package`语句，定义了该包的名称。

7. `import`语句，引入了`github.com/libp2p/go-libp2p/core/connmgr`库，定义了一个名为"ifconnmgr"的包。

8. `import`语句，引入了`github.com/libp2p/go-libp2p/core/network`库，定义了一个名为"net"的包。

9. `import`语句，引入了`github.com/libp2p/go-libp2p/core/peer`库，定义了一个名为"peer"的包。

10. `import`语句，引入了`github.com/libp2p/go-libp2p/core/protocol`库，定义了一个名为"protocol"的包。

11. `导入`语句，引入了`github.com/multiformats/go-multiaddr`库，定义了一个名为"manet"的包。


```go
package p2p

import (
	"io"
	"sync"

	ifconnmgr "github.com/libp2p/go-libp2p/core/connmgr"
	net "github.com/libp2p/go-libp2p/core/network"
	peer "github.com/libp2p/go-libp2p/core/peer"
	protocol "github.com/libp2p/go-libp2p/core/protocol"
	ma "github.com/multiformats/go-multiaddr"
	manet "github.com/multiformats/go-multiaddr/net"
)

const cmgrTag = "stream-fwd"

```

该代码定义了一个名为 Stream 的结构体，用于表示P2P流道的信息。

该 Stream 结构体包含以下字段：

- id：流道的ID，用于标识该流道。
- Protocol：该流道的协议ID，如HTTP、FTP等。
- OriginAddr：该流道的发送方地址，是一个多地址。
- TargetAddr：该流道的接收方地址，是一个多地址。
- peer：该流道的对等方ID，是一个P2PID结构体。
- Local：该流道的本地连接。
- Remote：该流道的远程连接。
- Registry：该流道注册表，用于存储该流道的元数据。

该结构体定义了一个 StreamRegistry，可以用来注册、获取和设置Stream的元数据。


```go
// Stream holds information on active incoming and outgoing p2p streams.
type Stream struct {
	id uint64

	Protocol protocol.ID

	OriginAddr ma.Multiaddr
	TargetAddr ma.Multiaddr
	peer       peer.ID

	Local  manet.Conn
	Remote net.Stream

	Registry *StreamRegistry
}

```

这段代码定义了两个函数，名为 `close()` 和 `reset()`，以及一个名为 `startStreaming()` 的函数。

`close()` 函数的作用是关闭与 `Stream` 相关的注册表，并释放资源。具体实现包括关闭注册表、调用注册表关闭函数以及调用 `reset()` 函数。

`reset()` 函数的作用是重新初始化与 `Stream` 相关的注册表，并释放资源。具体实现包括调用注册表关闭函数、调用 `close()` 函数以及调用 `reset()` 函数。

`startStreaming()` 函数的作用是启动一个流式传输的过程。具体实现包括开始接收本地数据、开始发送本地数据以及关闭与 `Stream` 相关的注册表。


```go
// close stream endpoints and deregister it.
func (s *Stream) close() {
	s.Registry.Close(s)
}

// reset closes stream endpoints and deregisters it.
func (s *Stream) reset() {
	s.Registry.Reset(s)
}

func (s *Stream) startStreaming() {
	go func() {
		_, err := io.Copy(s.Local, s.Remote)
		if err != nil {
			s.reset()
		} else {
			s.close()
		}
	}()

	go func() {
		_, err := io.Copy(s.Remote, s.Local)
		if err != nil {
			s.reset()
		} else {
			s.close()
		}
	}()
}

```

该代码定义了一个名为 StreamRegistry 的结构体，用于管理 streams。这个结构体包含一个名为 `Streams` 的 map，用于存储正在到来的和出圈的代理流，每个 key 都是一个 `uint64` 类型的 ID，用于标识不同的 stream。它还包含一个名为 `conns` 的 map，用于存储与每个连接相关的 `int` 类型的 ID，以及一个名为 `nextID` 的 `uint64` 类型的值，用于跟踪下一个注册的 ID。

这个结构体还包含一个名为 `Registration` 的方法，用于将指定的代理流注册到注册表中。这个方法首先获取注册表中的 `connmgr` 实例，然后使用 `TagPeer` 方法标记指定流与指定连接，最后增加该流与指定连接的连接计数，并启动该流的 streaming。

另外，这个结构体还包含一个名为 `Lock` 的方法，用于获取对注册表中流的状态进行互斥锁，以确保在多个调用 `Registration` 方法的情况下，只能够有一个实例在同时执行。


```go
// StreamRegistry is a collection of active incoming and outgoing proto app streams.
type StreamRegistry struct {
	sync.Mutex

	Streams map[uint64]*Stream
	conns   map[peer.ID]int
	nextID  uint64

	ifconnmgr.ConnManager
}

// Register registers a stream to the registry.
func (r *StreamRegistry) Register(streamInfo *Stream) {
	r.Lock()
	defer r.Unlock()

	r.ConnManager.TagPeer(streamInfo.peer, cmgrTag, 20)
	r.conns[streamInfo.peer]++

	streamInfo.id = r.nextID
	r.Streams[r.nextID] = streamInfo
	r.nextID++

	streamInfo.startStreaming()
}

```

此代码定义了一个名为`Deregister`的函数，该函数从注册表中删除指定ID的流。函数的参数`r`是一个`StreamRegistry`类型的引用，它锁定了`rg`并从注册表中删除指定的流。以下是函数的实际用途的简要说明：

1. 首先，函数会尝试从`rg`中检索指定ID的流。如果不能找到，函数将返回，因为已经确认该流已经从注册表中移除。
2. 如果找到了指定ID的流，函数将获取该流的对等方（`peer`属性）并减少与该对等方的连接数量。如果当前连接数量已经等于1，函数将删除该对等方并通知`rg`的`connMgr`函数。
3. 最后，函数从`rg`中删除指定的流，并通知`rg`的`connMgr`函数。

函数的实现充分说明了其作用：即从注册表中删除指定的流。


```go
// Deregister deregisters stream from the registry.
func (r *StreamRegistry) Deregister(streamID uint64) {
	r.Lock()
	defer r.Unlock()

	s, ok := r.Streams[streamID]
	if !ok {
		return
	}
	p := s.peer
	r.conns[p]--
	if r.conns[p] < 1 {
		delete(r.conns, p)
		r.ConnManager.UntagPeer(p, cmgrTag)
	}

	delete(r.Streams, streamID)
}

```

这两函数一起管理一个StreamRegistry，负责关闭Registry中注册的流，并输出Stream关闭的本地和远程端点。

在函数Close中，首先关闭Registry中注册的流所在的本地端点，然后关闭Registry中注册的流所在的远程端点。接着，调用Registry中注册的流所在的远程端的关闭操作，并关闭Registry中注册的流所在的本地端点。最后，输出Registry中注册的流。

在函数Reset中，首先关闭Registry中注册的流所在的本地端点，然后调用Registry中注册的流所在的远程端的Reset操作，使得远程端的点准备就绪。接着，调用Registry中注册的流所在的本地端的Reset操作，使得本地端的点准备就绪。最后，输出Registry中注册的流。


```go
// Close stream endpoints and deregister it.
func (r *StreamRegistry) Close(s *Stream) {
	_ = s.Local.Close()
	_ = s.Remote.Close()
	s.Registry.Deregister(s.id)
}

// Reset closes stream endpoints and deregisters it.
func (r *StreamRegistry) Reset(s *Stream) {
	_ = s.Local.Close()
	_ = s.Remote.Reset()
	s.Registry.Deregister(s.id)
}

```