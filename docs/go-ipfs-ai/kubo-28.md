# go-ipfs 源码解析 28

# `core/coreunix/add_test.go`

该代码的作用是定义了一个名为 "coreunix" 的包。这个包提供了对 IPFS(InterPlanetary File System) 对象的操作，包括创建、读取、写入和删除子目录等。它还提供了一些与测试相关的工具，如 Math.random() 函数来生成随机数，以及 Filepath.

具体来说，该代码包括以下组件：

1. 通过导入 "coreunix"、"math/rand"、"os" 和 "path/filepath" 包，可以访问 IPFS 对象。

2. 通过导入 "github.com/ipfs/kubo/core"、"github.com/ipfs/kubo/gc" 和 "github.com/ipfs/kubo/repo" 包，可以访问 Kubernetes 对象。

3. 通过导入 "github.com/ipfs/boxo/blockservice"、"github.com/ipfs/boxo/blockstore" 和 "github.com/ipfs/boxo/coreiface" 包，可以访问 Boxo 相关服务。

4. 通过导入 "github.com/ipfs/boxo/files"、"github.com/ipfs/boxo/posinfo" 和 "github.com/ipfs/boxo/ipld/merkledag" 包，可以访问 Boxo 相关文件操作和元数据结构。

5. 通过导入 "github.com/ipfs/boxo/blocks" 和 "github.com/ipfs/go-block-format" 包，可以访问 Boxo 相关块操作和块格式。

6. 通过导入 "github.com/ipfs/go-cid" 和 "github.com/ipfs/kubo/config" 包，可以访问 Kubernetes ConfigMaps 和 Benchmarks。

7. 通过导入 "github.com/ipfs/kubo/coreunix/testing" 包，可以访问测试相关的方法。

8. 通过导入 "github.com/ipfs/boxo/locator" 包，可以访问 Boxo 相关位置服务。

9. 通过导入 "github.com/ipfs/boxo/pkg/粪样" 包，可以访问 Boxo 相关用于测试的模块。

10. 通过导入 "github.com/ipfs/boxo/pkg/haven" 包，可以访问 Boxo 相关用于测试的模块。


```go
package coreunix

import (
	"bytes"
	"context"
	"io"
	"math/rand"
	"os"
	"path/filepath"
	"testing"
	"time"

	"github.com/ipfs/kubo/core"
	"github.com/ipfs/kubo/gc"
	"github.com/ipfs/kubo/repo"

	"github.com/ipfs/boxo/blockservice"
	blockstore "github.com/ipfs/boxo/blockstore"
	coreiface "github.com/ipfs/boxo/coreiface"
	"github.com/ipfs/boxo/files"
	pi "github.com/ipfs/boxo/filestore/posinfo"
	dag "github.com/ipfs/boxo/ipld/merkledag"
	blocks "github.com/ipfs/go-block-format"
	"github.com/ipfs/go-cid"
	"github.com/ipfs/go-datastore"
	syncds "github.com/ipfs/go-datastore/sync"
	config "github.com/ipfs/kubo/config"
)

```

This looks like a function that adds a file to a Pinning node, which is a type of gRPC-based service that uses the pinning operator to store and retrieve the hash of a given asset.

The function takes several arguments, which include the Pinning node's URL, the asset being added, and a data source to read the data from. It starts byGC1, which is the gRPC event that the Pinning node subscribes to for changes to the资产's hash.

The function then reads the data from the data source, finishes reading the data, and starts writing the data to the Pinning node. It uses the gc2 operator to perform the garbage collection, which is a gRPC event that the Pinning node subscribes to for changes in the Pinning node's memory.

Finally, the function checks if the asset's hash has already been added to the Pinning node by checking if the hash is present in the removedHashes map, and then writing the data to the Pinning node's data store.


```go
const testPeerID = "QmTFauExutTsy4XP6JbMFcw2Wa9645HJt2bTqL6qYDCKfe"

func TestAddMultipleGCLive(t *testing.T) {
	r := &repo.Mock{
		C: config.Config{
			Identity: config.Identity{
				PeerID: testPeerID, // required by offline node
			},
		},
		D: syncds.MutexWrap(datastore.NewMapDatastore()),
	}
	node, err := core.NewNode(context.Background(), &core.BuildCfg{Repo: r})
	if err != nil {
		t.Fatal(err)
	}

	out := make(chan interface{}, 10)
	adder, err := NewAdder(context.Background(), node.Pinning, node.Blockstore, node.DAG)
	if err != nil {
		t.Fatal(err)
	}
	adder.Out = out

	// make two files with pipes so we can 'pause' the add for timing of the test
	piper1, pipew1 := io.Pipe()
	hangfile1 := files.NewReaderFile(piper1)

	piper2, pipew2 := io.Pipe()
	hangfile2 := files.NewReaderFile(piper2)

	rfc := files.NewBytesFile([]byte("testfileA"))

	slf := files.NewMapDirectory(map[string]files.Node{
		"a": hangfile1,
		"b": hangfile2,
		"c": rfc,
	})

	go func() {
		defer close(out)
		_, _ = adder.AddAllAndPin(context.Background(), slf)
		// Ignore errors for clarity - the real bug would be gc'ing files while adding them, not this resultant error
	}()

	// Start writing the first file but don't close the stream
	if _, err := pipew1.Write([]byte("some data for file a")); err != nil {
		t.Fatal(err)
	}

	var gc1out <-chan gc.Result
	gc1started := make(chan struct{})
	go func() {
		defer close(gc1started)
		gc1out = gc.GC(context.Background(), node.Blockstore, node.Repo.Datastore(), node.Pinning, nil)
	}()

	// GC shouldn't get the lock until after the file is completely added
	select {
	case <-gc1started:
		t.Fatal("gc shouldn't have started yet")
	default:
	}

	// finish write and unblock gc
	pipew1.Close()

	// Should have gotten the lock at this point
	<-gc1started

	removedHashes := make(map[string]struct{})
	for r := range gc1out {
		if r.Error != nil {
			t.Fatal(err)
		}
		removedHashes[r.KeyRemoved.String()] = struct{}{}
	}

	if _, err := pipew2.Write([]byte("some data for file b")); err != nil {
		t.Fatal(err)
	}

	var gc2out <-chan gc.Result
	gc2started := make(chan struct{})
	go func() {
		defer close(gc2started)
		gc2out = gc.GC(context.Background(), node.Blockstore, node.Repo.Datastore(), node.Pinning, nil)
	}()

	select {
	case <-gc2started:
		t.Fatal("gc shouldn't have started yet")
	default:
	}

	pipew2.Close()

	<-gc2started

	for r := range gc2out {
		if r.Error != nil {
			t.Fatal(err)
		}
		removedHashes[r.KeyRemoved.String()] = struct{}{}
	}

	for o := range out {
		if _, ok := removedHashes[o.(*coreiface.AddEvent).Path.RootCid().String()]; ok {
			t.Fatal("gc'ed a hash we just added")
		}
	}
}

```

This is a function that initializes a Git repository. It does this by first setting up the database, then starting the database writer, and finally starting the database reader. The database writer is responsible for writing data to the specified directory, while the reader reads the data back and updates the database.

The initialization code first creates a new database, then creates a new writer. The writer is responsible for往该目录写入数据， and is started using the `gc` function.

The code then uses the `pipew` function to write some data to the directory, and then waits for the writer to finish.

The next part of the code then starts the writer, and since it's started multiple times, it uses a loop to ensure it has finished writing all the data.

Finally, the code sets up a loop to read the data from the repository, and updates the database c结构层的操作， it also uses the `pipew` function to read the data from the repository and updates the database c结构层的操作.

It also uses `cid.Decode` function to decode the cid.FPDecode error.


```go
func TestAddGCLive(t *testing.T) {
	r := &repo.Mock{
		C: config.Config{
			Identity: config.Identity{
				PeerID: testPeerID, // required by offline node
			},
		},
		D: syncds.MutexWrap(datastore.NewMapDatastore()),
	}
	node, err := core.NewNode(context.Background(), &core.BuildCfg{Repo: r})
	if err != nil {
		t.Fatal(err)
	}

	out := make(chan interface{})
	adder, err := NewAdder(context.Background(), node.Pinning, node.Blockstore, node.DAG)
	if err != nil {
		t.Fatal(err)
	}
	adder.Out = out

	rfa := files.NewBytesFile([]byte("testfileA"))

	// make two files with pipes so we can 'pause' the add for timing of the test
	piper, pipew := io.Pipe()
	hangfile := files.NewReaderFile(piper)

	rfd := files.NewBytesFile([]byte("testfileD"))

	slf := files.NewMapDirectory(map[string]files.Node{
		"a": rfa,
		"b": hangfile,
		"d": rfd,
	})

	addDone := make(chan struct{})
	go func() {
		defer close(addDone)
		defer close(out)
		_, err := adder.AddAllAndPin(context.Background(), slf)
		if err != nil {
			t.Error(err)
		}
	}()

	addedHashes := make(map[string]struct{})
	select {
	case o := <-out:
		addedHashes[o.(*coreiface.AddEvent).Path.RootCid().String()] = struct{}{}
	case <-addDone:
		t.Fatal("add shouldn't complete yet")
	}

	var gcout <-chan gc.Result
	gcstarted := make(chan struct{})
	go func() {
		defer close(gcstarted)
		gcout = gc.GC(context.Background(), node.Blockstore, node.Repo.Datastore(), node.Pinning, nil)
	}()

	// gc shouldn't start until we let the add finish its current file.
	if _, err := pipew.Write([]byte("some data for file b")); err != nil {
		t.Fatal(err)
	}

	select {
	case <-gcstarted:
		t.Fatal("gc shouldn't have started yet")
	default:
	}

	time.Sleep(time.Millisecond * 100) // make sure gc gets to requesting lock

	// finish write and unblock gc
	pipew.Close()

	// receive next object from adder
	o := <-out
	addedHashes[o.(*coreiface.AddEvent).Path.RootCid().String()] = struct{}{}

	<-gcstarted

	for r := range gcout {
		if r.Error != nil {
			t.Fatal(err)
		}
		if _, ok := addedHashes[r.KeyRemoved.String()]; ok {
			t.Fatal("gc'ed a hash we just added")
		}
	}

	var last cid.Cid
	for a := range out {
		// wait for it to finish
		c, err := cid.Decode(a.(*coreiface.AddEvent).Path.RootCid().String())
		if err != nil {
			t.Fatal(err)
		}
		last = c
	}

	ctx, cancel := context.WithTimeout(context.Background(), time.Second*5)
	defer cancel()

	set := cid.NewSet()
	err = dag.Walk(ctx, dag.GetLinksWithDAG(node.DAG), last, set.Visit)
	if err != nil {
		t.Fatal(err)
	}
}

```

This is a Go program that performs a simple operation on a pickle file. The program uses the相通\_block\_scorer function from the GORM-Go library to block and score the file's contents based on its content.

The program first reads the pickle file and then applies the blocker to it. The blocker is defined as follows:

* A new adder that adds the pickle file to the output and then uses the adder's output to add the pickle file to the adder's blockers.
* The adder has a progress bar that reaches the specified limit and a raw leaves flag that prevents the adder from making multiple copies.
* The blocker has a buffer of 512KB and uses the相通\_block\_scorer function to block and score the pickle file based on its content.

The program reads the pickle file and applies the blocker to it. It then reads the file's contents, block it based on the blocker's score, and adds the file to the output.

The program also reads the file's index and usage information and uses this to check if the blocker should add the file to the adder's blockers. If the blocker has a progress bar, the program checks if it has reached the specified limit and should stop. If the blocker has a raw leaves flag, the program checks if the blocker has made multiple copies and should stop.


```go
func testAddWPosInfo(t *testing.T, rawLeaves bool) {
	r := &repo.Mock{
		C: config.Config{
			Identity: config.Identity{
				PeerID: testPeerID, // required by offline node
			},
		},
		D: syncds.MutexWrap(datastore.NewMapDatastore()),
	}
	node, err := core.NewNode(context.Background(), &core.BuildCfg{Repo: r})
	if err != nil {
		t.Fatal(err)
	}

	bs := &testBlockstore{GCBlockstore: node.Blockstore, expectedPath: filepath.Join(os.TempDir(), "foo.txt"), t: t}
	bserv := blockservice.New(bs, node.Exchange)
	dserv := dag.NewDAGService(bserv)
	adder, err := NewAdder(context.Background(), node.Pinning, bs, dserv)
	if err != nil {
		t.Fatal(err)
	}
	out := make(chan interface{})
	adder.Out = out
	adder.Progress = true
	adder.RawLeaves = rawLeaves
	adder.NoCopy = true

	data := make([]byte, 5*1024*1024)
	rand.New(rand.NewSource(2)).Read(data) // Rand.Read never returns an error
	fileData := io.NopCloser(bytes.NewBuffer(data))
	fileInfo := dummyFileInfo{"foo.txt", int64(len(data)), time.Now()}
	file, _ := files.NewReaderPathFile(filepath.Join(os.TempDir(), "foo.txt"), fileData, &fileInfo)

	go func() {
		defer close(adder.Out)
		_, err = adder.AddAllAndPin(context.Background(), file)
		if err != nil {
			t.Error(err)
		}
	}()
	for range out {
	}

	exp := 0
	nonOffZero := 0
	if rawLeaves {
		exp = 1
		nonOffZero = 19
	}
	if bs.countAtOffsetZero != exp {
		t.Fatalf("expected %d blocks with an offset at zero (one root and one leaf), got %d", exp, bs.countAtOffsetZero)
	}
	if bs.countAtOffsetNonZero != nonOffZero {
		// note: the exact number will depend on the size and the sharding algo. used
		t.Fatalf("expected %d blocks with an offset > 0, got %d", nonOffZero, bs.countAtOffsetNonZero)
	}
}

```

这是一个 Go 语言中的测试框架，通过编写测试函数来测试 addWPosInfo 函数的正确性。

func TestAddWPosInfo(t *testing.T) {
	t.Run("with false", func(t *testing.T) {
		testAddWPosInfo(t, false)
	})
	t.Run("with true", func(t *testing.T) {
		testAddWPosInfo(t, true)
	})
}

type testBlockstore struct {
	blockstore.GCBlockstore
	expectedPath         string
	t                    *testing.T
	countAtOffsetZero    int
	countAtOffsetNonZero int
}

func TestAddWPosInfo(t *testing.T) {
	t.Run("with zero offset zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with non-zero offset non-zero", func(t *testing.T) {
		testAddWPosInfo(t, 0)
	})
	t.Run("with zero offset non-zero", func(t *testing.T) {
		


```go
func TestAddWPosInfo(t *testing.T) {
	testAddWPosInfo(t, false)
}

func TestAddWPosInfoAndRawLeafs(t *testing.T) {
	testAddWPosInfo(t, true)
}

type testBlockstore struct {
	blockstore.GCBlockstore
	expectedPath         string
	t                    *testing.T
	countAtOffsetZero    int
	countAtOffsetNonZero int
}

```

这段代码定义了两个函数：Put 和 PutMany。这两个函数用于在测试区块链块存储器（blockstore）中存储多个块（block）。

函数Put的作用是，在给定上下文上下文中，将给定（ctx，block）块存储到块存储器中。首先，调用块存储器中的一个函数CheckForPosInfo，然后使用块存储器中的另一个函数GCBlockstore.Put，将给定块（block）存储到块存储器中。

函数PutMany的作用与函数Put类似，但可以存储多个块。函数接收一个包含多个块的数组（blocks）。然后，遍历数组中的每个块（block），调用块存储器中的CheckForPosInfo函数，然后将块存储到块存储器中。

函数CheckForPosInfo接收一个块（block），并检查给定块的文件存储器（filestore）节点是否具有预期的路径。如果路径不正确，或者文件存储器节点没有指定路径，函数将导致断言错误（t.Fatal）。

函数CheckForPosInfo还统计给定块在尝试中的文件存储器节点位置计数器。如果给定块的文件存储器节点位置计数器不为零，或者尝试将块存储到块存储器中时，函数将统计失败（countAtOffsetNonZero）。


```go
func (bs *testBlockstore) Put(ctx context.Context, block blocks.Block) error {
	bs.CheckForPosInfo(block)
	return bs.GCBlockstore.Put(ctx, block)
}

func (bs *testBlockstore) PutMany(ctx context.Context, blocks []blocks.Block) error {
	for _, blk := range blocks {
		bs.CheckForPosInfo(blk)
	}
	return bs.GCBlockstore.PutMany(ctx, blocks)
}

func (bs *testBlockstore) CheckForPosInfo(block blocks.Block) {
	fsn, ok := block.(*pi.FilestoreNode)
	if ok {
		posInfo := fsn.PosInfo
		if posInfo.FullPath != bs.expectedPath {
			bs.t.Fatal("PosInfo does not have the expected path")
		}
		if posInfo.Offset == 0 {
			bs.countAtOffsetZero += 1
		} else {
			bs.countAtOffsetNonZero += 1
		}
	}
}

```

这段代码定义了一个名为 `dummyFileInfo` 的结构体类型，它包含以下字段：

- `name`：文件名。
- `size`：文件大小，以字节为单位。
- `modTime`：文件修改时间，以 `time.Time` 类型表示。

该结构体类型的实例可以被赋值给一个名为 `fi` 的指针变量，该指针变量可以用来访问和修改实例字段。

该代码的目的是提供一个简单的文件信息结构体，该结构体可以用于操作系统中的文件系统功能，例如文件操作系统的 `Stat` 函数等。


```go
type dummyFileInfo struct {
	name    string
	size    int64
	modTime time.Time
}

func (fi *dummyFileInfo) Name() string       { return fi.name }
func (fi *dummyFileInfo) Size() int64        { return fi.size }
func (fi *dummyFileInfo) Mode() os.FileMode  { return 0 }
func (fi *dummyFileInfo) ModTime() time.Time { return fi.modTime }
func (fi *dummyFileInfo) IsDir() bool        { return false }
func (fi *dummyFileInfo) Sys() interface{}   { return nil }

```

# `core/coreunix/metadata.go`

这段代码定义了一个名为AddMetadataTo的函数，它接收三个参数：一个表示IPFS节点对象的`n`，一个表示元数据的`skey`，以及一个表示元数据的`m`。

函数首先尝试从`skey`中计算出元数据`m`的键值，然后使用`cid`库中的`Decode`函数将键值`skey`转换为`cid`编码。接着，使用`core`库中的`IpfsNode`上下文对象和`DAG`对象，获取`n`的元数据并将其存储在`mdnode`变量中。

接下来，函数使用`ft`库中的`BytesForMetadata`函数将元数据`m`字节编码为字节切片，并将其设置为`mdnode`的`Data`字段。然后，使用`AddNodeLink`方法在`nd`节点链接中添加`file`元数据链，将元数据链的键设置为`mdnode`的`CID`字段，并将其父节点设置为`n`。最后，使用`DAG.Add`方法将`mdnode`节点添加到`n`的元数据链中，并将元数据链的`CID`字段设置为返回的`CID`值。

如果函数在计算或创建元数据链时出现错误，它将返回错误信息并将其作为输入参数传递给调用者。


```go
package coreunix

import (
	dag "github.com/ipfs/boxo/ipld/merkledag"
	ft "github.com/ipfs/boxo/ipld/unixfs"
	cid "github.com/ipfs/go-cid"
	core "github.com/ipfs/kubo/core"
)

func AddMetadataTo(n *core.IpfsNode, skey string, m *ft.Metadata) (string, error) {
	c, err := cid.Decode(skey)
	if err != nil {
		return "", err
	}

	nd, err := n.DAG.Get(n.Context(), c)
	if err != nil {
		return "", err
	}

	mdnode := new(dag.ProtoNode)
	mdata, err := ft.BytesForMetadata(m)
	if err != nil {
		return "", err
	}

	mdnode.SetData(mdata)
	if err := mdnode.AddNodeLink("file", nd); err != nil {
		return "", err
	}

	err = n.DAG.Add(n.Context(), mdnode)
	if err != nil {
		return "", err
	}

	return mdnode.Cid().String(), nil
}

```

该函数的作用是获取一个名为 "skey" 的键对应的值，并返回该值所对应的 Metadata 类型或错误。具体实现包括以下步骤：

1. 使用 cid.Decode 函数将给定的键 "skey" 解码为相应的 CID 编码。如果解码过程中出现错误，函数将返回 nil 和错误信息。
2. 使用 n.DAG.Get 函数获取给定键 "skey" 的节点，并将其封装为一个 DAG 节点对象。
3. 使用 nd.Get 函数获取上面得到的节点对象，并将其转换为一个 DAG 节点的 Proto 对象。
4. 使用 ft.MetadataFromBytes 函数将上面得到的 Proto 对象转换为对应的 Metadata 类型。
5. 如果转换过程中出现错误，函数将返回 dag.ErrNotProtobuf 和错误信息。

因此，该函数的主要作用是获取给定键的值并返回对应的 Metadata 类型或错误。


```go
func Metadata(n *core.IpfsNode, skey string) (*ft.Metadata, error) {
	c, err := cid.Decode(skey)
	if err != nil {
		return nil, err
	}

	nd, err := n.DAG.Get(n.Context(), c)
	if err != nil {
		return nil, err
	}

	pbnd, ok := nd.(*dag.ProtoNode)
	if !ok {
		return nil, dag.ErrNotProtobuf
	}

	return ft.MetadataFromBytes(pbnd.Data())
}

```

# `core/coreunix/metadata_test.go`

该代码是一个 Go 语言项目，它定义了一个名为 "coreunix" 的包。通过导入其他 package，它实现了 Unix 文件系统的功能，包括文件操作、块设备访问和数据存储等。

具体来说，该代码实现了以下功能：

1. 支持块设备访问：通过导入 "ft" 和 "importer" 包，可以实现对块设备的文件系统接口，包括挂载、卸载和读写等操作。
2. 支持文件操作：通过导入 "io" 和 "uio" 包，可以实现对文件的读写和通道操作等。
3. 支持数据存储：通过导入 "bstore"、"chunker"、"offline"、"u"、"cid"、"ds" 和 "dssync" 包，可以实现对磁盘、网络和本地数据存储等的数据存储接口，包括文件系统、网络驱动程序和数据同步等。
4. 支持 Unix 系统：通过导入 "core" 和 "ipld" 包，可以实现对 Unix 系统的支持，包括进程管理、文件权限等。

总体的来说，该代码定义了一个 Unix 文件系统客户端库，可以方便地在各种 Unix 系统上实现文件和数据存储操作。


```go
package coreunix

import (
	"bytes"
	"context"
	"io"
	"testing"

	bserv "github.com/ipfs/boxo/blockservice"
	merkledag "github.com/ipfs/boxo/ipld/merkledag"
	ft "github.com/ipfs/boxo/ipld/unixfs"
	importer "github.com/ipfs/boxo/ipld/unixfs/importer"
	uio "github.com/ipfs/boxo/ipld/unixfs/io"
	core "github.com/ipfs/kubo/core"

	bstore "github.com/ipfs/boxo/blockstore"
	chunker "github.com/ipfs/boxo/chunker"
	offline "github.com/ipfs/boxo/exchange/offline"
	u "github.com/ipfs/boxo/util"
	cid "github.com/ipfs/go-cid"
	ds "github.com/ipfs/go-datastore"
	dssync "github.com/ipfs/go-datastore/sync"
	ipld "github.com/ipfs/go-ipld-format"
)

```

This is a testing function for the rootnode, which is responsible for storing metadata for all nodes in the immutable DAG.

The function takes a piece of data (`data`) and a `chunker` to split it into chunks of data. It then creates a new metadata object (`mdk`) and uses it to build a root node for a test DAG (`ds`) by calling the `core.IpfsNode.BuildDagFromReader` method with the `chunker` and the piece of data.

The function checks if the process was successful by checking if the root node can be retrieved using the `ds.Get` method. If the call to `ds.Get` fails, the function will panic.

Finally, the function reads the output of the root node and checks if it matches the expected data. If the data does not match, the function panics.


```go
func getDagserv(t *testing.T) ipld.DAGService {
	db := dssync.MutexWrap(ds.NewMapDatastore())
	bs := bstore.NewBlockstore(db)
	blockserv := bserv.New(bs, offline.Exchange(bs))
	return merkledag.NewDAGService(blockserv)
}

func TestMetadata(t *testing.T) {
	ctx := context.Background()
	// Make some random node
	ds := getDagserv(t)
	data := make([]byte, 1000)
	_, err := io.ReadFull(u.NewTimeSeededRand(), data)
	if err != nil {
		t.Fatal(err)
	}
	r := bytes.NewReader(data)
	nd, err := importer.BuildDagFromReader(ds, chunker.DefaultSplitter(r))
	if err != nil {
		t.Fatal(err)
	}

	c := nd.Cid()

	m := new(ft.Metadata)
	m.MimeType = "THIS IS A TEST"

	// Such effort, many compromise
	ipfsnode := &core.IpfsNode{DAG: ds}

	mdk, err := AddMetadataTo(ipfsnode, c.String(), m)
	if err != nil {
		t.Fatal(err)
	}

	rec, err := Metadata(ipfsnode, mdk)
	if err != nil {
		t.Fatal(err)
	}
	if rec.MimeType != m.MimeType {
		t.Fatalf("something went wrong in conversion: '%s' != '%s'", rec.MimeType, m.MimeType)
	}

	cdk, err := cid.Decode(mdk)
	if err != nil {
		t.Fatal(err)
	}

	retnode, err := ds.Get(ctx, cdk)
	if err != nil {
		t.Fatal(err)
	}

	rtnpb, ok := retnode.(*merkledag.ProtoNode)
	if !ok {
		t.Fatal("expected protobuf node")
	}

	ndr, err := uio.NewDagReader(ctx, rtnpb, ds)
	if err != nil {
		t.Fatal(err)
	}

	out, err := io.ReadAll(ndr)
	if err != nil {
		t.Fatal(err)
	}

	if !bytes.Equal(out, data) {
		t.Fatal("read incorrect data")
	}
}

```

# `core/mock/mock.go`

这段代码是一个用于测试libp2p包中进行CoreMock的库。其中，CoreMock是用来模拟libp2p的基本功能，而该测试库则是对其进行测试。

具体来说，该测试库包含了一些必要的功能：

1. 模拟Core网络：通过创建一个自定义的Core网络，模拟libp2p中的Core网络功能。
2. 模拟Peer：创建一个自定义的Peer，用于与Core网络进行交互。
3. 模拟P2P网络：通过在Core网络中发送特定的数据，对P2P网络进行测试。
4. 模拟存储：模拟libp2p中的存储功能，包括Peer Store和Data Store。
5. 创建数据存储：使用Data Store创建一个或多个数据存储。
6. 配置Core网络：通过设置Core网络的配置参数，对Core网络进行定制。
7. 运行测试：运行测试以模拟不同的情况，如在不同的网络环境下、不同的Core网络配置等。

通过这些模拟，可以测试libp2p在不同情况下的行为，包括Core网络、Peer、P2P网络、存储等功能。


```go
package coremock

import (
	"context"
	"fmt"
	"io"

	libp2p2 "github.com/ipfs/kubo/core/node/libp2p"

	"github.com/ipfs/kubo/commands"
	"github.com/ipfs/kubo/core"
	"github.com/ipfs/kubo/repo"

	"github.com/ipfs/go-datastore"
	syncds "github.com/ipfs/go-datastore/sync"
	config "github.com/ipfs/kubo/config"

	"github.com/libp2p/go-libp2p"
	testutil "github.com/libp2p/go-libp2p-testing/net"
	"github.com/libp2p/go-libp2p/core/host"
	"github.com/libp2p/go-libp2p/core/peer"
	pstore "github.com/libp2p/go-libp2p/core/peerstore"

	mocknet "github.com/libp2p/go-libp2p/p2p/net/mock"
)

```

该代码定义了一个名为NewMockNode的函数，该函数用于在测试中构造一个IpfsNode实例。函数的核心部分是一个名为core.NewNode的函数，该函数使用libp2p2.HostOption构造一个IpfsNode实例，并设置其Online选项为false，以确保该节点仅作为虚拟节点使用，不与主网络中的节点进行交互。

函数的另一个部分是名为MockHostOption的函数，该函数使用libp2p2.HostOption构造一个虚拟的Host选项，用于设置IpfsNode实例的监听地址。该函数的实现主要通过两个步骤：首先，错误处理函数core.NewNode的返回值，如果出错则返回一个 nil值。其次，遍历并添加IpfsNode实例的监听地址，并将其添加到ps.AddAddrs函数中，以便在测试中正确地设置IpfsNode实例的监听地址。

最后，该代码定义了一个名为NewMockNode测试函数，该函数使用该名为NewMockNode的函数来创建一个IpfsNode实例，并输出其返回值。


```go
// NewMockNode constructs an IpfsNode for use in tests.
func NewMockNode() (*core.IpfsNode, error) {
	// effectively offline, only peer in its network
	return core.NewNode(context.Background(), &core.BuildCfg{
		Online: true,
		Host:   MockHostOption(mocknet.New()),
	})
}

func MockHostOption(mn mocknet.Mocknet) libp2p2.HostOption {
	return func(id peer.ID, ps pstore.Peerstore, opts ...libp2p.Option) (host.Host, error) {
		var cfg libp2p.Config
		if err := cfg.Apply(opts...); err != nil {
			return nil, err
		}

		// The mocknet does not use the provided libp2p.Option. This options include
		// the listening addresses we want our peer listening on. Therefore, we have
		// to manually parse the configuration and add them here.
		ps.AddAddrs(id, cfg.ListenAddrs, pstore.PermanentAddrTTL)
		return mn.AddPeerWithPeerstore(id, ps)
	}
}

```

该代码定义了一个名为 MockCmdsCtx 的函数，它返回一个名为 (commands.Context, error) 的元组。

函数内部使用了多种技巧来实现对代码中模拟的 `commands.Context` 和 `error` 类型进行包装：

1. 首先，通过调用 `testutil.RandIdentity` 函数生成一个随机的身份ID，并检查是否出错。如果出错，函数将返回 (`nil`, `error`)。否则，生成成功，将返回 `(string, nil)`，其中 `string` 是身份ID，`nil` 是返回的 `error` 类型。
2. 其次，定义了一个 `config.Config` 类型，其中包含一个身份认证的配置，例如：
go
config.Config={
	Identity: config.Identity{
		PeerID: p.String(),
	},
},`
3. 然后，定义了一个 `repo.Mock` 类型，其中包含一个 `datastore.NewMapDatastore` 实现的映射，以及一个 `syncds.MutexWrap`实现的互斥锁。
4. 最后，通过调用 `core.NewNode` 和 `core.BuildCfg` 函数，创建了一个 `core.IpfsNode` 类型的实例，并将其作为参数传递给 `commands.Context.ConstructNode` 函数，实现将模拟的 `commands.Context` 包装为 `core.IpfsNode` 类型。


```go
func MockCmdsCtx() (commands.Context, error) {
	// Generate Identity
	ident, err := testutil.RandIdentity()
	if err != nil {
		return commands.Context{}, err
	}
	p := ident.ID()

	conf := config.Config{
		Identity: config.Identity{
			PeerID: p.String(),
		},
	}

	r := &repo.Mock{
		D: syncds.MutexWrap(datastore.NewMapDatastore()),
		C: conf,
	}

	node, err := core.NewNode(context.Background(), &core.BuildCfg{
		Repo: r,
	})
	if err != nil {
		return commands.Context{}, err
	}

	return commands.Context{
		ConfigRoot: "/tmp/.mockipfsconfig",
		ConstructNode: func() (*core.IpfsNode, error) {
			return node, nil
		},
	}, nil
}

```

该函数的作用是创建一个 mocknet 网络节点的实例并返回给调用者。

具体来说，它采取了以下步骤：

1. 创建一个名为 `ds` 的互斥锁，用于在多个客户端之间同步数据。
2. 创建一个 `config.Init` 函数的实例，用于初始化节点配置。该函数接收一个 `io.Discard` 类型的参数，表示异步 discard，以及一个表示最大 peers 的整数。如果初始化失败，函数返回一个非空错误并返回。
3. 创建一个 `config` 结构体，其中包含用于 Swarm 节点地址的字符串。
4. 将 `datastore.NewMapDatastore()` 返回的数据存储器实例分配给 `ds` 变量，用于存储 Swarm 节点数据。
5. 将 `config` 实例的 `addresses.Swarm` 字段设置为 Swarm 节点地址。
6. 调用 `core.NewNode` 函数，该函数接收一个 `context.Context` 类型的上下文，用于在节点创建时设置连接上下文，以及一个表示节点路由的 `libp2p2.DHTServerOption` 类型的实例。
7. 将 `repo.Mock` 实例的 `C` 字段设置为 `*cfg`，用于设置 mocknet 节点的配置，以及将 `ds` 实例设置为数据存储器。
8. 设置 `host` 字段为 `mn.Peers()` 切片中的第一个元素，用于设置主机名称。
9. 返回 `core.IpfsNode` 类型的实例，表示 Swarm 节点。

如果调用 `func MockPublicNode(ctx context.Context, mn mocknet.Mocknet) (*core.IpfsNode, error)` 时出现错误，该函数将返回非空错误并打印堆栈跟踪。


```go
func MockPublicNode(ctx context.Context, mn mocknet.Mocknet) (*core.IpfsNode, error) {
	ds := syncds.MutexWrap(datastore.NewMapDatastore())
	cfg, err := config.Init(io.Discard, 2048)
	if err != nil {
		return nil, err
	}
	count := len(mn.Peers())
	cfg.Addresses.Swarm = []string{
		fmt.Sprintf("/ip4/18.0.%d.%d/tcp/4001", count>>16, count&0xFF),
	}
	cfg.Datastore = config.Datastore{}
	return core.NewNode(ctx, &core.BuildCfg{
		Online:  true,
		Routing: libp2p2.DHTServerOption,
		Repo: &repo.Mock{
			C: *cfg,
			D: ds,
		},
		Host: MockHostOption(mn),
	})
}

```

# `core/node/bitswap.go`

该代码是一个 Go 语言编写的 Node.js package，它实现了名为 "node" 的包。通过导入不同的库，它实现了在 IPFS（InterPlanetary File System）网络中执行 "boxo" 命令行工具的一些功能。下面是实现的一些关键部分的注释：

1. `node` 包的导入：引入了 IPFS 包的 "node" 包，以及 "bitswap"、"network"、"blockstore" 和 "exchange" 包。

2. `bitswap`：通过 "import context" 导入了 "bitswap" 包的上下文。这个库可能提供了一些用于 IPFS 网络操作的函数。

3. `network`：通过 "import *package helpers" 导入了 "network" 包的许多帮助函数。这些函数可能有助于与 IPFS 网络进行交互。

4. `exchange`：通过 "import *package helpers" 导入了 "exchange" 包的许多帮助函数。这些函数可能有助于与 IPFS 网络进行交互。

5. `irouting`：通过 "import *package helpers" 导入了 "irouting" 包的许多帮助函数。这些函数可能有助于与 IPFS 网络进行交互。

6. `host`：通过 "import *package helpers" 导入了 "host" 包的许多帮助函数。这些函数可能有助于与 IPFS 网络进行交互。

7. `core/node/helpers`：这个包的导出函数为 "helpers"，但其中包含的函数并没有具体的名字。这些函数可能是在 IPFS 网络中执行与 "node" 包的上下文相关的操作时需要的。

8. `github.com/ipfs/boxo/bitswap`：通过 "import *package boxo" 导入了 "bitswap" 包。这个包在 IPFS 网络中提供了一些与 "boxo" 命令行工具相关的功能。

9. `github.com/ipfs/boxo/network`：通过 "import *package boxo" 导入了 "network" 包。这个包在 IPFS 网络中提供了一些与 "boxo" 命令行工具相关的功能。

10. `github.com/ipfs/boxo/exchange`：通过 "import *package boxo" 导入了 "exchange" 包。这个包在 IPFS 网络中提供了一些与 "boxo" 命令行工具相关的功能。

11. `github.com/ipfs/kubo/config`：通过 "import *package kubo" 导入了 "config" 包。这个包在 IPFS 网络中提供了一些与 "kubo" 命令行工具相关的功能。

12. `github.com/ipfs/kubo/routing`：通过 "import *package kubo" 导入了 "routing" 包。这个包在 IPFS 网络中提供了一些与 "kubo" 命令行工具相关的功能。

13. `github.com/libp2p/go-libp2p/core/host`：通过 "import *package libp2p/host" 导入了 "libp2p/host" 包的上下文。这个包可能提供了一些用于与 IPFS 网络进行交互的功能。

14. `github.com/libp2p/go-libp2p/core/policy`：通过 "import *package libp2p/policy" 导入了 "libp2p/policy" 包的上下文。这个包可能提供了一些用于与 IPFS 网络进行交互的功能。

15. `github.com/libp2p/go-libp2p/event`：通过 "import *package libp2p/event" 导入了 "libp2p/event" 包的上下文。这个包可能提供了一些用于与 IPFS 网络进行交互的功能。

16. `github.com/ipfs/boxo/util`：这个包的导出函数为 "util"，但其中包含的函数并没有具体的名字。这些函数可能是在 IPFS 网络中执行与 "bitswap" 和 "network" 包的上下文相关的操作时需要的。


```go
package node

import (
	"context"
	"time"

	"github.com/ipfs/boxo/bitswap"
	"github.com/ipfs/boxo/bitswap/network"
	blockstore "github.com/ipfs/boxo/blockstore"
	exchange "github.com/ipfs/boxo/exchange"
	"github.com/ipfs/kubo/config"
	irouting "github.com/ipfs/kubo/routing"
	"github.com/libp2p/go-libp2p/core/host"
	"go.uber.org/fx"

	"github.com/ipfs/kubo/core/node/helpers"
)

```

这段代码定义了一个名为`bitswapOptionsOut`的结构体，它代表了Kubernetes中`config.doc.bitswap.Engine`和`config.doc.bitswap.Task`对象的选项。

具体来说，这段代码指定了以下参数的默认值：

- `DefaultEngineBlockstoreWorkerCount`：发动机块存储器工作者的最大数量，值为128。
- `DefaultTaskWorkerCount`：任务工作者的最大数量，值为8。
- `DefaultEngineTaskWorkerCount`：发动机任务工作者的最大数量，值为8。
- `DefaultMaxOutstandingBytesPerPeer`：每个对等方提供的最大突出内存，值为1 << 20。
- `DefaultProviderSearchDelay`：代理搜索延迟，单位为毫秒，值为1000 * time.Millisecond。

此外，`bitswapOptionsOut`结构体还定义了一个包含一个或多个`bitswap.Option`的`BitswapOpts`字段，但该字段没有定义具体的选项。


```go
// Docs: https://github.com/ipfs/kubo/blob/master/docs/config.md#internalbitswap
const (
	DefaultEngineBlockstoreWorkerCount = 128
	DefaultTaskWorkerCount             = 8
	DefaultEngineTaskWorkerCount       = 8
	DefaultMaxOutstandingBytesPerPeer  = 1 << 20
	DefaultProviderSearchDelay         = 1000 * time.Millisecond
)

type bitswapOptionsOut struct {
	fx.Out

	BitswapOpts []bitswap.Option `group:"bitswap-options,flatten"`
}

```

这段代码定义了一个名为`BitswapOptions`的函数，它从`config.go`文件中读取`Bitswap`的配置选项，并判断是否要提供数据。

函数有两个参数，一个是`*config.Config`类型的变量`cfg`，另一个是`bool`类型的参数`provide`，表示是否提供数据。函数返回一个实现了`bitswapOptionsOut`接口的函数，这个函数会在不提供数据的情况下返回一个空的`bitswapOptionsOut`结构体，否则返回一个包含了所有设置的`bitswap.Option`的`bitswapOptionsOut`结构体。

函数内部先判断`cfg.Internal.Bitswap`是否已经定义好，如果定义好了，就将其赋值给`internalBsCfg`。然后根据`provide`参数的真假，设置`bitswap.Option`中的`provideEnabled``bitswap.ProvideEnabled`选项为`true`，设置`bitswap.ProviderSearchDelay``bitswap.ProviderSearchDelay`选项的值，参考[1]；设置`bitswap.EngineBlockstoreWorkerCount``bitswap.EngineBlockstoreWorkerCount`选项的值，参考[2]；设置`bitswap.TaskWorkerCount``bitswap.TaskWorkerCount`选项的值，参考[3]；最后设置`bitswap.MaxOutstandingBytesPerPeer``bitswap.MaxOutstandingBytesPerPeer`选项的值，参考[4]。

最后，函数返回一个实现了`bitswapOptionsOut`接口的函数，这个函数使用了`bitswap.Option`列表的所有设置，并且如果没有提供数据，则返回一个空的`bitswapOptionsOut`结构体。


```go
// BitswapOptions creates configuration options for Bitswap from the config file
// and whether to provide data.
func BitswapOptions(cfg *config.Config, provide bool) interface{} {
	return func() bitswapOptionsOut {
		var internalBsCfg config.InternalBitswap
		if cfg.Internal.Bitswap != nil {
			internalBsCfg = *cfg.Internal.Bitswap
		}

		opts := []bitswap.Option{
			bitswap.ProvideEnabled(provide),
			bitswap.ProviderSearchDelay(internalBsCfg.ProviderSearchDelay.WithDefault(DefaultProviderSearchDelay)), // See https://github.com/ipfs/go-ipfs/issues/8807 for rationale
			bitswap.EngineBlockstoreWorkerCount(int(internalBsCfg.EngineBlockstoreWorkerCount.WithDefault(DefaultEngineBlockstoreWorkerCount))),
			bitswap.TaskWorkerCount(int(internalBsCfg.TaskWorkerCount.WithDefault(DefaultTaskWorkerCount))),
			bitswap.EngineTaskWorkerCount(int(internalBsCfg.EngineTaskWorkerCount.WithDefault(DefaultEngineTaskWorkerCount))),
			bitswap.MaxOutstandingBytesPerPeer(int(internalBsCfg.MaxOutstandingBytesPerPeer.WithDefault(DefaultMaxOutstandingBytesPerPeer))),
		}

		return bitswapOptionsOut{BitswapOpts: opts}
	}
}

```

这是一个定义了一个名为`onlineExchangeIn`的结构体，其中包含了实现 LibP2P 备份兼容的 `bitswap` 块交换机的选项。

该结构体定义了以下字段：

- `fx.In`字段：该字段是一个输入 libp2p 兼容数据流合约的实例。
- `Mctx`字段：该字段是一个 MetricsCtx 实例，用于管理指标。
- `Host`字段：该字段是目标主机，用于配置 NameNode 和轻量级配置。
- `Rt`字段：该字段是路由器实例，用于为選定的路由器实例提供路由。
- `Bs`字段：该字段是块存储器实例，用于管理 block 存储。
- `BitswapOpts`字段：该字段是一个 bitswap 选项的数组，通过 `bitswap-options` 配置。

此外，该结构体还定义了一个名为 `OnlineExchange`的函数，该函数接收一个 `bitswapIn` 实例和一个生命周期钩子(Lc)，并返回一个 libp2p 兼容的块交换机实例。该函数使用了 `bitswapNetwork` 实例，该实例是从 NameNode 和路由器实例中选择的主机，通过 `bitswap` 管理 libp2p 兼容数据流。函数实现了 `exchange.Interface` 接口，因此可以作为其他 struct 的字段类型。


```go
type onlineExchangeIn struct {
	fx.In

	Mctx        helpers.MetricsCtx
	Host        host.Host
	Rt          irouting.ProvideManyRouter
	Bs          blockstore.GCBlockstore
	BitswapOpts []bitswap.Option `group:"bitswap-options"`
}

// OnlineExchange creates new LibP2P backed block exchange (BitSwap).
// Additional options to bitswap.New can be provided via the "bitswap-options"
// group.
func OnlineExchange() interface{} {
	return func(in onlineExchangeIn, lc fx.Lifecycle) exchange.Interface {
		bitswapNetwork := network.NewFromIpfsHost(in.Host, in.Rt)

		exch := bitswap.New(helpers.LifecycleCtx(in.Mctx, lc), bitswapNetwork, in.Bs, in.BitswapOpts...)
		lc.Append(fx.Hook{
			OnStop: func(ctx context.Context) error {
				return exch.Close()
			},
		})
		return exch
	}
}

```

# `core/node/builder.go`

这段代码是一个基于Go语言构建的Node.js库，它主要用于在IPFS（InterPlanetary File System）网络中实现一个名为“kubo”的P2P网络服务。

具体来说，这段代码实现了以下功能：

1. 定义了一个名为“node”的包。

2. 导入了多个外部库，包括：

	* context：用于创建和取消与上下文的关联。
	* crypto：用于实现加密和解密操作。
	* errors：用于处理错误。
	* node：用于导入与libp2p相关的API。
	* repo：用于导入与kubelet、kubeadm等系统相关的API。
	* ds：用于与DataStore数据库进行交互。
		+ dsync：用于实现对DataStore数据库的同步操作。
		+ cfg：用于设置Kubernetes集群的配置。
		+ peer：用于实现与IPFS网络中的Peer进行通信。

3. 实现了Kubernetes客户端（kubo.go）和Kubernetes服务器（kubeadm.go）的API。

4. 通过使用libp2p的加密功能，实现了数据在Kubernetes集群和IPFS网络之间的加密传输。

5. 通过使用Go-datastore库实现了对DataStore数据库的同步操作。

6. 通过使用Go-datastore库实现的同步操作，可以确保在Kubernetes集群中的操作与其他节点同步。

7. 通过使用libp2p实现的Peer通信，使得Kubernetes集群中的节点可以相互通信，并实现数据在Kubernetes集群和IPFS网络之间的交互。


```go
package node

import (
	"context"
	"crypto/rand"
	"encoding/base64"
	"errors"

	"go.uber.org/fx"

	"github.com/ipfs/kubo/core/node/helpers"
	"github.com/ipfs/kubo/core/node/libp2p"
	"github.com/ipfs/kubo/repo"

	ds "github.com/ipfs/go-datastore"
	dsync "github.com/ipfs/go-datastore/sync"
	cfg "github.com/ipfs/kubo/config"
	"github.com/libp2p/go-libp2p/core/crypto"
	peer "github.com/libp2p/go-libp2p/core/peer"
)

```

这是一段定义了一个名为 `BuildCfg` 的结构体的代码。这个结构体描述了一个用于配置节点运行时设置的选项的容器。以下是这个结构体中所有字段的解释：


// BuildCfg struct定义了一个名为BuildCfg的结构体，它用于配置IPFS节点的创建选项。
type BuildCfg struct {
	// Online字段表示节点是否处于在线状态。如果设置为true，节点将启用网络。
	Online bool

	// ExtraOpts字段是一个包含额外配置选项的map。
	ExtraOpts map[string]bool

	// Permanent字段表示节点是否应该运行更昂贵的进程，以提高长期性能。
	Permanent bool

	// DisableEncryptedConnections字段表示是否禁用连接加密。
	DisableEncryptedConnections bool

	// NilRepo字段表示是否使用一个nil数据存储的repo。
	NilRepo bool

	// Routing字段定义了IPFS路由选项。
	Routing libp2p.RoutingOption

	// Host字段定义了IPFS主机选项。
	Host    libp2p.HostOption

	// Repo字段定义了用于构建IPFS Repo的选项。
	Repo    repo.Repo
}


这个结构体定义了五个字段，包括 `Online`、`ExtraOpts`、`Permanent`、`DisableEncryptedConnections` 和 `NilRepo`。其中，`ExtraOpts` 和 `Routing` 是 map 类型的字段，`Host` 和 `Repo` 是 struct 类型的字段。


```go
type BuildCfg struct {
	// If online is set, the node will have networking enabled
	Online bool

	// ExtraOpts is a map of extra options used to configure the ipfs nodes creation
	ExtraOpts map[string]bool

	// If permanent then node should run more expensive processes
	// that will improve performance in long run
	Permanent bool

	// DisableEncryptedConnections disables connection encryption *entirely*.
	// DO NOT SET THIS UNLESS YOU'RE TESTING.
	DisableEncryptedConnections bool

	// If NilRepo is set, a Repo backed by a nil datastore will be constructed
	NilRepo bool

	Routing libp2p.RoutingOption
	Host    libp2p.HostOption
	Repo    repo.Repo
}

```

这段代码定义了两个函数，分别接收一个名为cfg的BuildCfg结构体和一个key字符串作为参数。

第一个函数名为getOpt，它接收一个key字符串并返回一个布尔值。函数首先检查cfg结构体中是否有extraOpts成员，如果没有，则返回false。如果存在，函数使用key从cfg结构体中提取extraOpts[key]的值，并返回。

第二个函数名为fillDefaults，它接收一个BuildCfg结构体并返回一个错误。函数首先检查cfg结构体中是否设置了一个名为Repo的选项，如果没有设置，则尝试设置一个名为nilrepo的选项。如果既没有设置Repo，也没有设置nilrepo，则函数会创建一个默认的Repo，并将其设置为cfg结构体中的Repo。接下来函数会检查是否设置了一个名为Routing的选项，如果没有设置，则将Routing设置为libp2p.DHTOption。然后函数会检查是否设置了一个名为Host的选项，如果没有设置，则将Host设置为libp2p.DefaultHostOption。最后函数会检查是否发生了其他错误，如果没有发生错误，则返回一个nil表示。如果发生了错误，函数返回一个非nil值，并使用err函数返回该错误。


```go
func (cfg *BuildCfg) getOpt(key string) bool {
	if cfg.ExtraOpts == nil {
		return false
	}

	return cfg.ExtraOpts[key]
}

func (cfg *BuildCfg) fillDefaults() error {
	if cfg.Repo != nil && cfg.NilRepo {
		return errors.New("cannot set a Repo and specify nilrepo at the same time")
	}

	if cfg.Repo == nil {
		var d ds.Datastore
		if cfg.NilRepo {
			d = ds.NewNullDatastore()
		} else {
			d = ds.NewMapDatastore()
		}
		r, err := defaultRepo(dsync.MutexWrap(d))
		if err != nil {
			return err
		}
		cfg.Repo = r
	}

	if cfg.Routing == nil {
		cfg.Routing = libp2p.DHTOption
	}

	if cfg.Host == nil {
		cfg.Host = libp2p.DefaultHostOption
	}

	return nil
}

```

这段代码定义了一个名为`options`的函数，该函数接受一个`BuildCfg`对象和一个`Context`上下文作为参数。函数的作用是在不抛出错误的情况下，根据传入的配置选项创建一个FX选项组，并返回该选项组和相应的配置选项。

函数首先调用一个名为`fillDefaults`的函数，该函数根据`BuildCfg`对象中的默认选项创建一个默认的选项组。如果`fillDefaults`函数在创建选项组时出现错误，函数将抛出`fx.Error`并返回错误和一个空指针。

选项组创建后，函数使用`fx.Provide`函数为每个选项设置提供器。对于`repoOption`，提供者设置了一个`repo.Repo`的提供者，并在停止执行时调用`OnStop`方法关闭仓库。对于`metricsCtx`，提供者设置了一个`helpers.MetricsCtx`的提供者，用于在开始和结束时收集 metrics。对于`hostOption`和`routingOption`，提供者设置了一个`libp2p.HostOption`和一个`libp2p.RoutingOption`的提供者，用于设置主机和路由选项。

最后，函数调用`fx.Options`函数，传递选项组和配置选项作为参数，并返回一个包含选项组和配置选项的`*cfg.Config`。


```go
// options creates fx option group from this build config
func (cfg *BuildCfg) options(ctx context.Context) (fx.Option, *cfg.Config) {
	err := cfg.fillDefaults()
	if err != nil {
		return fx.Error(err), nil
	}

	repoOption := fx.Provide(func(lc fx.Lifecycle) repo.Repo {
		lc.Append(fx.Hook{
			OnStop: func(ctx context.Context) error {
				return cfg.Repo.Close()
			},
		})

		return cfg.Repo
	})

	metricsCtx := fx.Provide(func() helpers.MetricsCtx {
		return helpers.MetricsCtx(ctx)
	})

	hostOption := fx.Provide(func() libp2p.HostOption {
		return cfg.Host
	})

	routingOption := fx.Provide(func() libp2p.RoutingOption {
		return cfg.Routing
	})

	conf, err := cfg.Repo.Config()
	if err != nil {
		return fx.Error(err), nil
	}

	return fx.Options(
		repoOption,
		hostOption,
		routingOption,
		metricsCtx,
	), conf
}

```

这段代码定义了一个名为`defaultRepo`的函数，它接受一个名为`dstore`的参数，代表一个`repo.Datastore`类型的数据存储。

函数内部首先设置了一个名为`c`的配置对象，然后使用`crypto.GenerateKeyPairWithReader`函数生成一个RSA密钥对，并取得私钥。接着使用`peer.IDFromPublicKey`函数尝试从公钥中获取一个ID，如果失败则返回一个错误。

然后函数设置了一些常量，包括`c.Bootstrap`，`c.Addresses.Swarm`和`c.Identity.PeerID`。最后将私钥编码为Base64字符串并将其存储在`c.Identity.PrivKey`中。

函数返回一个名为`repo.Mock`的实例，其中包含一个数据存储`dstore`和一个表示函数执行上下文的`c`。没有错误发生。


```go
func defaultRepo(dstore repo.Datastore) (repo.Repo, error) {
	c := cfg.Config{}
	priv, pub, err := crypto.GenerateKeyPairWithReader(crypto.RSA, 2048, rand.Reader)
	if err != nil {
		return nil, err
	}

	pid, err := peer.IDFromPublicKey(pub)
	if err != nil {
		return nil, err
	}

	privkeyb, err := crypto.MarshalPrivateKey(priv)
	if err != nil {
		return nil, err
	}

	c.Bootstrap = cfg.DefaultBootstrapAddresses
	c.Addresses.Swarm = []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/udp/4001/quic-v1"}
	c.Identity.PeerID = pid.String()
	c.Identity.PrivKey = base64.StdEncoding.EncodeToString(privkeyb)

	return &repo.Mock{
		D: dstore,
		C: c,
	}, nil
}

```

# `core/node/core.go`

该代码是一个 Go 语言编写的 Node.js package，它提供了在 IPFS 区块链上执行操作的库。具体来说，它实现了以下功能：

1.  blockstore：一个定序存储器，用于将数据存储为一系列不可变、顺序的块中。
2. exchange：一个用于与 IPFS 区块链进行交互的库。
3. offline：一个用于在本地存储库中存储数据的库。
4. fetcher：一个用于从 IPFS 区块链上获取数据的库。
5. blockservice：一个用于管理多播客户端的库。
6. filestore：一个用于管理文件系统的库。
7. ipld：一个用于构建 IPFS 块链数据结构的库。
8. merkledag：一个用于管理 Merkle Tree 的库。
9. unixfs：一个用于管理 Unix FSM 操作的库。
10. pathresolver：一个用于解析路径的库。
11. pin：一个用于将数据写入本地存储的库。
12. pinner：一个用于将数据写入 IPFS 区块链的库。
13. ipld：一个用于在 IPFS 区块链上执行操作的库。
14. cid：一个用于创建和检查客户端 ID 的库。
15. datastore：一个用于管理数据存储的库。
16. format：一个用于将 IPFS 数据格式化成为特定格式的库。
17. ipld-format：一个用于格式化 IPFS 数据格式的库。
18. unixfsnode：一个用于管理 Unix FSM 操作的库。
19. dagpb：一个用于与 Go-DAGPB 类型进行交互的库。
20. go-cid：一个用于与 Go-CID 类型进行交互的库。

该代码主要作用是提供一个定序存储器，用于在 IPFS 区块链上执行操作，包括将数据写入和读取。通过使用 blockstore、exchange、offline、fetcher、blockservice、filestore 等库，可以实现与 IPFS 区块链的交互，并支持在本地存储库中存储数据。


```go
package node

import (
	"context"
	"fmt"

	"github.com/ipfs/boxo/blockservice"
	blockstore "github.com/ipfs/boxo/blockstore"
	exchange "github.com/ipfs/boxo/exchange"
	offline "github.com/ipfs/boxo/exchange/offline"
	"github.com/ipfs/boxo/fetcher"
	bsfetcher "github.com/ipfs/boxo/fetcher/impl/blockservice"
	"github.com/ipfs/boxo/filestore"
	"github.com/ipfs/boxo/ipld/merkledag"
	"github.com/ipfs/boxo/ipld/unixfs"
	"github.com/ipfs/boxo/mfs"
	pathresolver "github.com/ipfs/boxo/path/resolver"
	pin "github.com/ipfs/boxo/pinning/pinner"
	"github.com/ipfs/boxo/pinning/pinner/dspinner"
	"github.com/ipfs/go-cid"
	"github.com/ipfs/go-datastore"
	format "github.com/ipfs/go-ipld-format"
	"github.com/ipfs/go-unixfsnode"
	dagpb "github.com/ipld/go-codec-dagpb"
	"go.uber.org/fx"

	"github.com/ipfs/kubo/core/node/helpers"
	"github.com/ipfs/kubo/repo"
)

```

这两段代码定义了两个函数：`BlockService` 和 `Pinning`。它们的作用如下：

1. `BlockService`：

`BlockService` 函数接收三个参数：`lc`、`bs` 和 `rem`。它创建了一个新的 `blockservice` 实例，并为该实例设置了两个 `fx.Lifecycle` 钩子：

a. `OnStop`：在块服务停止时执行的钩子。它通过调用 `blocksvc.Close()` 来关闭块服务。

b. `OnAddBlock`：当块服务接收到一个新的可触达块时执行的钩子。该钩子将在块被添加到块存储器后调用。

2. `Pinning`：

`Pinning` 函数接收三个参数：`bstore`、`ds` 和 `repo`。它返回一个 `pin.Pinner` 实例和一个错误。

a. `Pinning`：创建一个新的 `pin.Pinner` 实例，并设置其 `pinning` 字段为 `nil`。

b. `Pinning`：创建一个新的 `pin.Pinner` 实例，并设置其 `pinning` 字段为 `nil`。接着，它尝试调用 `repo.Datastore()` 的 `Sync` 方法来确保所有块都已经被同步到块存储器中。然后，它继续调用 `repo.Datastore()` 的 `Sync` 方法，以确保所有块都已经被同步到块存储器中。最后，它调用 `ds.Sync` 方法来确保所有块都已经同步到块存储器中。

a. 如果设置的 `ds` 参数为 ` nil`，那么 `Pinning` 将直接返回 `nil`，不会创建新的 `pin.Pinner` 实例。

b. 如果设置的 `ds` 参数为 `blockstore.Blockstore` 实例，那么它将调用 `ds.Sync` 方法来同步块存储器。


```go
// BlockService creates new blockservice which provides an interface to fetch content-addressable blocks
func BlockService(lc fx.Lifecycle, bs blockstore.Blockstore, rem exchange.Interface) blockservice.BlockService {
	bsvc := blockservice.New(bs, rem)

	lc.Append(fx.Hook{
		OnStop: func(ctx context.Context) error {
			return bsvc.Close()
		},
	})

	return bsvc
}

// Pinning creates new pinner which tells GC which blocks should be kept
func Pinning(bstore blockstore.Blockstore, ds format.DAGService, repo repo.Repo) (pin.Pinner, error) {
	rootDS := repo.Datastore()

	syncFn := func(ctx context.Context) error {
		if err := rootDS.Sync(ctx, blockstore.BlockPrefix); err != nil {
			return err
		}
		return rootDS.Sync(ctx, filestore.FilestorePrefix)
	}
	syncDs := &syncDagService{ds, syncFn}

	ctx := context.TODO()

	pinning, err := dspinner.New(ctx, rootDS, syncDs)
	if err != nil {
		return nil, err
	}

	return pinning, nil
}

```

这段代码定义了一个名为 syncDagService 的 struct，它包含两个 field：一个名为 format.DAGService 的 field，另一个名为 merkledag.SessionMaker 的 field。这两个 field 都指向一个名为 syncDagService 的类型，该类型实现了 sync.DAGService 接口。

syncDagService 是一个实现了 sync.DAGService 接口的 struct，它包含一个名为 syncFn 的 field，该字段定义了一个函数，用于在给定上下文上下文时执行。具体来说，这个函数返回一个错误，如果执行失败，将会返回一个符合 sync.DAGService 定义的错误代码的 DAGService 类型的错误。

在给定的代码中，syncDagService 的 syncFn 函数接收一个 context.Context 类型的参数，这个上下文将会被传递给 DAGService 的 Sync 函数。因此，可以认为 syncDagService 的 syncFn 函数是一个使用 DAGService 实现同步数据操作的函数。


```go
var (
	_ merkledag.SessionMaker = new(syncDagService)
	_ format.DAGService      = new(syncDagService)
)

// syncDagService is used by the Pinner to ensure data gets persisted to the underlying datastore
type syncDagService struct {
	format.DAGService
	syncFn func(context.Context) error
}

func (s *syncDagService) Sync(ctx context.Context) error {
	return s.syncFn(ctx)
}

```

这段代码定义了一个名为 `func` 的函数，接收一个名为 `syncDagService` 的 `*` 类型参数 `s`，并返回一个 `Session` 函数。函数内部使用 `merkledag.NewSession` 函数创建一个新的会话，并将 `s.DAGService` 作为参数传递给 `merkledag.NewSession` 函数，以便初始化 DAG 服务。

接下来，定义了一个名为 `FetchersOut` 的类型变量，它包含一个 `fx.Out` 类型的字段 `fx`，以及四个字段 `ipldFetcher`、`unixfsFetcher` 和 `offlineIpldFetcher`，这些字段都是 `fetcher.Factory` 类型的接口，用于从不同的存储介质（如 IPFS、Unix 和磁盘）获取数据。

另外，定义了一个名为 `FetchersIn` 的类型变量，它包含一个 `fx.In` 类型的字段 `fx`，以及四个字段 `ipldFetcher`、`unixfsFetcher` 和 `offlineIpldFetcher`，这些字段也都是 `fetcher.Factory` 类型的接口，用于从不同的存储介质获取数据。

最后，函数内部创建了一个 `ipldFetcher`、`unixfsFetcher` 和 `offlineIpldFetcher` 类型的变量 `fets`，并将它们作为 `fetcher.Factory` 类型的接口，用于从不同存储介质获取数据。这些变量将通过 ` FetchersIn` 类型的实例进行注入，所以 `FetchersIn` 类型变量将自动填充 `fets` 类型的变量。


```go
func (s *syncDagService) Session(ctx context.Context) format.NodeGetter {
	return merkledag.NewSession(ctx, s.DAGService)
}

// FetchersOut allows injection of fetchers.
type FetchersOut struct {
	fx.Out
	IPLDFetcher          fetcher.Factory `name:"ipldFetcher"`
	UnixfsFetcher        fetcher.Factory `name:"unixfsFetcher"`
	OfflineIPLDFetcher   fetcher.Factory `name:"offlineIpldFetcher"`
	OfflineUnixfsFetcher fetcher.Factory `name:"offlineUnixfsFetcher"`
}

// FetchersIn allows using fetchers for other dependencies.
type FetchersIn struct {
	fx.In
	IPLDFetcher          fetcher.Factory `name:"ipldFetcher"`
	UnixfsFetcher        fetcher.Factory `name:"unixfsFetcher"`
	OfflineIPLDFetcher   fetcher.Factory `name:"offlineIpldFetcher"`
	OfflineUnixfsFetcher fetcher.Factory `name:"offlineUnixfsFetcher"`
}

```

该代码定义了一个名为`FetcherConfig`的函数，它返回一个用于构建新的`Fetcher`实例的`fetcher config`。

`Fetcher`是一个`BlockService`类型的接口，代表一个支持异步数据下载的下载器。`FetcherConfig`函数的输入参数`bs`是一个`BlockService`对象，它包含一个`Blockstore`实例，用于存储下载的数据。

`FetcherConfig`函数首先创建一个`IpldFetcher`实例，它使用` blockservice.BlockService`提供的`Fetcher`实例，并实现了其中的一些方法。然后，它创建了一个带有`reifier`选项的`unixfsnode.Reifier`实例，用于将数据存储在本地磁盘上。

接下来，`FetcherConfig`函数使用`blockservice.BlockService`提供的`OfflineIpldFetcher`构造方法，创建了一个带有本地磁盘存储的`IpldFetcher`实例。然后，它使用`unixfsnode.Reifier`将本地磁盘上的数据映射到`UnixFSFetcher`实例中，其中`reifier`选项用于在下载过程中将数据写入本地磁盘。

最后，`FetcherConfig`函数返回一个`Fetcher`实例，其中包含`IPLDFetcher`、`UnixfsFetcher`和`OfflineIPLDFetcher`，分别表示支持异步数据下载、支持数据下载到本地磁盘和支持数据下载到本地磁盘的下载器实例。


```go
// FetcherConfig returns a fetcher config that can build new fetcher instances
func FetcherConfig(bs blockservice.BlockService) FetchersOut {
	ipldFetcher := bsfetcher.NewFetcherConfig(bs)
	ipldFetcher.PrototypeChooser = dagpb.AddSupportToChooser(bsfetcher.DefaultPrototypeChooser)
	unixFSFetcher := ipldFetcher.WithReifier(unixfsnode.Reify)

	// Construct offline versions which we can safely use in contexts where
	// path resolution should not fetch new blocks via exchange.
	offlineBs := blockservice.New(bs.Blockstore(), offline.Exchange(bs.Blockstore()))
	offlineIpldFetcher := bsfetcher.NewFetcherConfig(offlineBs)
	offlineIpldFetcher.PrototypeChooser = dagpb.AddSupportToChooser(bsfetcher.DefaultPrototypeChooser)
	offlineUnixFSFetcher := offlineIpldFetcher.WithReifier(unixfsnode.Reify)

	return FetchersOut{
		IPLDFetcher:          ipldFetcher,
		UnixfsFetcher:        unixFSFetcher,
		OfflineIPLDFetcher:   offlineIpldFetcher,
		OfflineUnixfsFetcher: offlineUnixFSFetcher,
	}
}

```

这段代码定义了一个名为PathResolversOut的结构体，它表示所有用于路径解析的Fetcher的组合。

这个结构体包含四个成员变量，都使用了匿名类型注解，表示它们是类型PathResolversOut的成员。这些成员变量都使用了pathresolver.Resolver类型，它表示用于解析路径的Resolver实例。

在结构体之外，还定义了一个名为PathResolverConfig的函数，它接受一个FetchersIn参数和一个作为其输入的fetchers。这个函数创建了一个PathResolversOut类型，它使用了传入的Fetchers创建了一个PathResolversOut实例，并将其返回。

总结一下，这段代码定义了一个PathResolversOut类型，它包含了多个用于路径解析的Fetcher实例，以及用于创建PathResolversOut实例的函数PathResolverConfig。这个函数允许将Fetcher实例注入PathResolversOut实例，使得PathResolversOut可以更加灵活地设置用于路径解析的Fetcher。


```go
// PathResolversOut allows injection of path resolvers
type PathResolversOut struct {
	fx.Out
	IPLDPathResolver          pathresolver.Resolver `name:"ipldPathResolver"`
	UnixFSPathResolver        pathresolver.Resolver `name:"unixFSPathResolver"`
	OfflineIPLDPathResolver   pathresolver.Resolver `name:"offlineIpldPathResolver"`
	OfflineUnixFSPathResolver pathresolver.Resolver `name:"offlineUnixFSPathResolver"`
}

// PathResolverConfig creates path resolvers with the given fetchers.
func PathResolverConfig(fetchers FetchersIn) PathResolversOut {
	return PathResolversOut{
		IPLDPathResolver:          pathresolver.NewBasicResolver(fetchers.IPLDFetcher),
		UnixFSPathResolver:        pathresolver.NewBasicResolver(fetchers.UnixfsFetcher),
		OfflineIPLDPathResolver:   pathresolver.NewBasicResolver(fetchers.OfflineIPLDFetcher),
		OfflineUnixFSPathResolver: pathresolver.NewBasicResolver(fetchers.OfflineUnixfsFetcher),
	}
}

```

This is a Go function that wraps the Datastore operations of the Dexie sync hook. It is part of a larger system that syncs files from a Dexie storage device to a local file system.

The function takes a context (`ctx`) and a block store block prefix (`blockstore.BlockPrefix`) as input from the caller. It returns the root node of the Merkledag that represents the file system.

If an error is caused by an issue such as a missing file or a file that does not exist, the function returns with an error. If the file is present, the function uses the `repo.Datastore().Get` method to retrieve the file's contents and then writes it to the root node of the Merkledag using the `dsk` path.

The function uses several helper functions to set up the lifecycle of the Merkledag and the file system. These functions include:

* `helpers.LifecycleCtx`: This function sets up the context for the lifecycle of the file system.
* `repo.DatastoreStringGetter`: This function is used to retrieve a file's contents from the file system store.
* `merkledag.NodeWithT反省`: This type is used to represent a node in the Merkledag.
* `errchk.errChk`: This function is used to check for errors in a Recorder.
* `cid.Cast`: This function is used to cast a value to a `Merkledag.Node`.
* `dag.Add`: This function is used to add a file to the root of the Merkledag.
* `dag.Get`: This function is used to retrieve the contents of a file from the root of the Merkledag.
* `cid.Node.回绕`: This function is used to回绕到父节点。

The function returns an error if the file is not found or there is an issue with the file system, or if there is an error when writing to the file system.


```go
// Dag creates new DAGService
func Dag(bs blockservice.BlockService) format.DAGService {
	return merkledag.NewDAGService(bs)
}

// Files loads persisted MFS root
func Files(mctx helpers.MetricsCtx, lc fx.Lifecycle, repo repo.Repo, dag format.DAGService) (*mfs.Root, error) {
	dsk := datastore.NewKey("/local/filesroot")
	pf := func(ctx context.Context, c cid.Cid) error {
		rootDS := repo.Datastore()
		if err := rootDS.Sync(ctx, blockstore.BlockPrefix); err != nil {
			return err
		}
		if err := rootDS.Sync(ctx, filestore.FilestorePrefix); err != nil {
			return err
		}

		if err := rootDS.Put(ctx, dsk, c.Bytes()); err != nil {
			return err
		}
		return rootDS.Sync(ctx, dsk)
	}

	var nd *merkledag.ProtoNode
	ctx := helpers.LifecycleCtx(mctx, lc)
	val, err := repo.Datastore().Get(ctx, dsk)

	switch {
	case err == datastore.ErrNotFound || val == nil:
		nd = unixfs.EmptyDirNode()
		err := dag.Add(ctx, nd)
		if err != nil {
			return nil, fmt.Errorf("failure writing to dagstore: %s", err)
		}
	case err == nil:
		c, err := cid.Cast(val)
		if err != nil {
			return nil, err
		}

		rnd, err := dag.Get(ctx, c)
		if err != nil {
			return nil, fmt.Errorf("error loading filesroot from DAG: %s", err)
		}

		pbnd, ok := rnd.(*merkledag.ProtoNode)
		if !ok {
			return nil, merkledag.ErrNotProtobuf
		}

		nd = pbnd
	default:
		return nil, err
	}

	root, err := mfs.NewRoot(ctx, dag, nd, pf)

	lc.Append(fx.Hook{
		OnStop: func(ctx context.Context) error {
			return root.Close()
		},
	})

	return root, err
}

```