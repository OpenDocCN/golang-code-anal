# `kubo\core\coreunix\add_test.go`

```
package coreunix

import (
    "bytes" // 导入 bytes 包，用于操作字节
    "context" // 导入 context 包，用于控制程序的上下文
    "io" // 导入 io 包，用于进行输入输出操作
    "math/rand" // 导入 math/rand 包，用于生成随机数
    "os" // 导入 os 包，提供了操作系统功能
    "path/filepath" // 导入 path/filepath 包，用于处理文件路径
    "testing" // 导入 testing 包，用于进行单元测试
    "time" // 导入 time 包，用于处理时间

    "github.com/ipfs/kubo/core" // 导入 core 包
    "github.com/ipfs/kubo/gc" // 导入 gc 包
    "github.com/ipfs/kubo/repo" // 导入 repo 包

    "github.com/ipfs/boxo/blockservice" // 导入 blockservice 包
    blockstore "github.com/ipfs/boxo/blockstore" // 导入 blockstore 包
    "github.com/ipfs/boxo/files" // 导入 files 包
    pi "github.com/ipfs/boxo/filestore/posinfo" // 导入 posinfo 包
    dag "github.com/ipfs/boxo/ipld/merkledag" // 导入 merkledag 包
    blocks "github.com/ipfs/go-block-format" // 导入 go-block-format 包
    "github.com/ipfs/go-cid" // 导入 go-cid 包
    "github.com/ipfs/go-datastore" // 导入 go-datastore 包
    syncds "github.com/ipfs/go-datastore/sync" // 导入 sync 包
    config "github.com/ipfs/kubo/config" // 导入 config 包
    coreiface "github.com/ipfs/kubo/core/coreiface" // 导入 coreiface 包
)

const testPeerID = "QmTFauExutTsy4XP6JbMFcw2Wa9645HJt2bTqL6qYDCKfe" // 定义常量 testPeerID

func TestAddMultipleGCLive(t *testing.T) { // 定义测试函数 TestAddMultipleGCLive
    r := &repo.Mock{ // 创建 repo.Mock 对象并赋值给变量 r
        C: config.Config{ // 设置 repo.Mock 对象的 C 字段
            Identity: config.Identity{ // 设置 config.Config 对象的 Identity 字段
                PeerID: testPeerID, // 设置 PeerID 字段为 testPeerID
            },
        },
        D: syncds.MutexWrap(datastore.NewMapDatastore()), // 设置 repo.Mock 对象的 D 字段
    }
    node, err := core.NewNode(context.Background(), &core.BuildCfg{Repo: r}) // 创建 core.Node 对象并赋值给变量 node
    if err != nil { // 如果 err 不为空
        t.Fatal(err) // 输出错误信息并终止测试
    }

    out := make(chan interface{}, 10) // 创建一个带缓冲区大小为 10 的通道并赋值给变量 out
    adder, err := NewAdder(context.Background(), node.Pinning, node.Blockstore, node.DAG) // 创建 Adder 对象并赋值给变量 adder
    if err != nil { // 如果 err 不为空
        t.Fatal(err) // 输出错误信息并终止测试
    }
    adder.Out = out // 设置 adder 对象的 Out 字段为 out

    // make two files with pipes so we can 'pause' the add for timing of the test
    piper1, pipew1 := io.Pipe() // 创建管道并赋值给变量 piper1 和 pipew1
    hangfile1 := files.NewReaderFile(piper1) // 创建一个读取器文件并赋值给变量 hangfile1

    piper2, pipew2 := io.Pipe() // 创建管道并赋值给变量 piper2 和 pipew2
    hangfile2 := files.NewReaderFile(piper2) // 创建一个读取器文件并赋值给变量 hangfile2

    rfc := files.NewBytesFile([]byte("testfileA")) // 创建一个字节文件并赋值给变量 rfc

    slf := files.NewMapDirectory(map[string]files.Node{ // 创建一个文件映射目录并赋值给变量 slf
        "a": hangfile1, // 设置键值对
        "b": hangfile2, // 设置键值对
        "c": rfc, // 设置键值对
    })
    // 启动一个 goroutine，defer 关闭 out 通道，然后调用 adder.AddAllAndPin 方法，忽略错误以保持代码清晰
    go func() {
        defer close(out)
        _, _ = adder.AddAllAndPin(context.Background(), slf)
        // 忽略错误以保持代码清晰 - 真正的 bug 应该是在添加文件时进行垃圾回收，而不是这个错误
    }()

    // 开始写入第一个文件，但不关闭流
    if _, err := pipew1.Write([]byte("some data for file a")); err != nil {
        t.Fatal(err)
    }

    // 创建一个用于接收 gc 结果的通道 gc1out，并创建一个用于标记 gc1 开始的通道 gc1started
    var gc1out <-chan gc.Result
    gc1started := make(chan struct{})
    // 启动一个 goroutine，defer 关闭 gc1started 通道，然后调用 gc.GC 方法
    go func() {
        defer close(gc1started)
        gc1out = gc.GC(context.Background(), node.Blockstore, node.Repo.Datastore(), node.Pinning, nil)
    }()

    // GC 不应该在文件完全添加之前获取锁
    select {
    case <-gc1started:
        t.Fatal("gc shouldn't have started yet")
    default:
    }

    // 完成写入并解除 gc 阻塞
    pipew1.Close()

    // 在这一点上应该已经获取到锁
    <-gc1started

    // 创建一个用于存储已移除哈希的 map
    removedHashes := make(map[string]struct{})
    // 遍历 gc1out，处理每个结果
    for r := range gc1out {
        if r.Error != nil {
            t.Fatal(err)
        }
        removedHashes[r.KeyRemoved.String()] = struct{}{}
    }

    // 写入第二个文件
    if _, err := pipew2.Write([]byte("some data for file b")); err != nil {
        t.Fatal(err)
    }

    // 创建一个用于接收 gc 结果的通道 gc2out，并创建一个用于标记 gc2 开始的通道 gc2started
    var gc2out <-chan gc.Result
    gc2started := make(chan struct{})
    // 启动一个 goroutine，defer 关闭 gc2started 通道，然后调用 gc.GC 方法
    go func() {
        defer close(gc2started)
        gc2out = gc.GC(context.Background(), node.Blockstore, node.Repo.Datastore(), node.Pinning, nil)
    }()

    // GC 不应该在文件完全添加之前获取锁
    select {
    case <-gc2started:
        t.Fatal("gc shouldn't have started yet")
    default:
    }

    // 关闭 pipew2 流
    pipew2.Close()

    // 等待 gc2 开始
    <-gc2started

    // 遍历 gc2out，处理每个结果
    for r := range gc2out {
        if r.Error != nil {
            t.Fatal(err)
        }
        removedHashes[r.KeyRemoved.String()] = struct{}{}
    }

    // 遍历 out 通道中的事件，检查是否有已移除的哈希
    for o := range out {
        if _, ok := removedHashes[o.(*coreiface.AddEvent).Path.RootCid().String()]; ok {
            t.Fatal("gc'ed a hash we just added")
        }
    }
    // 测试添加垃圾收集的功能
    func TestAddGCLive(t *testing.T) {
        // 创建模拟仓库对象
        r := &repo.Mock{
            // 设置配置信息
            C: config.Config{
                Identity: config.Identity{
                    PeerID: testPeerID, // required by offline node
                },
            },
            // 使用互斥锁包装数据存储
            D: syncds.MutexWrap(datastore.NewMapDatastore()),
        }
        // 创建新的节点
        node, err := core.NewNode(context.Background(), &core.BuildCfg{Repo: r})
        if err != nil {
            t.Fatal(err)
        }

        // 创建输出通道
        out := make(chan interface{})
        // 创建新的添加器
        adder, err := NewAdder(context.Background(), node.Pinning, node.Blockstore, node.DAG)
        if err != nil {
            t.Fatal(err)
        }
        adder.Out = out

        // 创建新的字节文件
        rfa := files.NewBytesFile([]byte("testfileA"))

        // 创建管道以便暂停添加以进行测试的时间
        piper, pipew := io.Pipe()
        hangfile := files.NewReaderFile(piper)

        // 创建新的字节文件
        rfd := files.NewBytesFile([]byte("testfileD"))

        // 创建包含文件节点的映射目录
        slf := files.NewMapDirectory(map[string]files.Node{
            "a": rfa,
            "b": hangfile,
            "d": rfd,
        })

        // 创建完成通道
        addDone := make(chan struct{})
        go func() {
            defer close(addDone)
            defer close(out)
            // 添加所有文件并进行固定
            _, err := adder.AddAllAndPin(context.Background(), slf)
            if err != nil {
                t.Error(err)
            }
        }()

        // 创建已添加哈希的映射
        addedHashes := make(map[string]struct{})
        select {
        case o := <-out:
            addedHashes[o.(*coreiface.AddEvent).Path.RootCid().String()] = struct{}{}
        case <-addDone:
            t.Fatal("add shouldn't complete yet")
        }

        // 创建垃圾收集结果通道
        var gcout <-chan gc.Result
        gcstarted := make(chan struct{})
        go func() {
            defer close(gcstarted)
            // 执行垃圾收集
            gcout = gc.GC(context.Background(), node.Blockstore, node.Repo.Datastore(), node.Pinning, nil)
        }()

        // 垃圾收集不应该在我们让添加完成当前文件之前开始
        if _, err := pipew.Write([]byte("some data for file b")); err != nil {
            t.Fatal(err)
        }

        select {
        case <-gcstarted:
            t.Fatal("gc shouldn't have started yet")
        default:
        }
    }
    // 等待 100 毫秒，确保垃圾回收器能够获取到请求的锁

    // 完成写入并解除对垃圾回收器的阻塞
    pipew.Close()

    // 从 adder 接收下一个对象
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
        // 等待其完成
        c, err := cid.Decode(a.(*coreiface.AddEvent).Path.RootCid().String())
        if err != nil {
            t.Fatal(err)
        }
        last = c
    }

    // 创建一个带有 5 秒超时的上下文
    ctx, cancel := context.WithTimeout(context.Background(), time.Second*5)
    defer cancel()

    // 创建一个 CID 集合
    set := cid.NewSet()
    // 使用 DAG 遍历获取与最后一个 CID 相关的链接
    err = dag.Walk(ctx, dag.GetLinksWithDAG(node.DAG), last, set.Visit)
    if err != nil {
        t.Fatal(err)
    }
// 测试函数，用于测试添加位置信息
func testAddWPosInfo(t *testing.T, rawLeaves bool) {
    // 创建模拟仓库对象
    r := &repo.Mock{
        // 设置配置信息
        C: config.Config{
            Identity: config.Identity{
                PeerID: testPeerID, // 离线节点所需的对等节点 ID
            },
        },
        // 使用互斥锁封装数据存储
        D: syncds.MutexWrap(datastore.NewMapDatastore()),
    }
    // 创建新节点
    node, err := core.NewNode(context.Background(), &core.BuildCfg{Repo: r})
    if err != nil {
        t.Fatal(err)
    }

    // 创建测试块存储
    bs := &testBlockstore{GCBlockstore: node.Blockstore, expectedPath: filepath.Join(os.TempDir(), "foo.txt"), t: t}
    // 创建块服务
    bserv := blockservice.New(bs, node.Exchange)
    // 创建有向无环图服务
    dserv := dag.NewDAGService(bserv)
    // 创建添加器
    adder, err := NewAdder(context.Background(), node.Pinning, bs, dserv)
    if err != nil {
        t.Fatal(err)
    }
    // 创建输出通道
    out := make(chan interface{})
    adder.Out = out
    adder.Progress = true
    adder.RawLeaves = rawLeaves
    adder.NoCopy = true

    // 创建数据
    data := make([]byte, 5*1024*1024)
    rand.New(rand.NewSource(2)).Read(data) // Rand.Read 永远不会返回错误
    fileData := io.NopCloser(bytes.NewBuffer(data))
    fileInfo := dummyFileInfo{"foo.txt", int64(len(data)), time.Now()}
    // 创建文件读取器
    file, _ := files.NewReaderPathFile(filepath.Join(os.TempDir(), "foo.txt"), fileData, &fileInfo)

    // 启动协程执行添加并固定操作
    go func() {
        defer close(adder.Out)
        _, err = adder.AddAllAndPin(context.Background(), file)
        if err != nil {
            t.Error(err)
        }
    }()
    // 等待输出通道关闭
    for range out {
    }

    // 初始化期望值
    exp := 0
    nonOffZero := 0
    if rawLeaves {
        exp = 1
        nonOffZero = 19
    }
    // 检查偏移为零的块数是否符合预期
    if bs.countAtOffsetZero != exp {
        t.Fatalf("expected %d blocks with an offset at zero (one root and one leaf), got %d", exp, bs.countAtOffsetZero)
    }
    // 检查偏移非零的块数是否符合预期
    if bs.countAtOffsetNonZero != nonOffZero {
        // 注意：确切的数字将取决于大小和分片算法的使用
        t.Fatalf("expected %d blocks with an offset > 0, got %d", nonOffZero, bs.countAtOffsetNonZero)
    }
}

// 测试添加位置信息的函数
func TestAddWPosInfo(t *testing.T) {
    # 调用 testAddWPosInfo 函数，传入参数 t 和 false
    testAddWPosInfo(t, false)
// 测试函数，用于测试添加位置信息和原始叶子节点
func TestAddWPosInfoAndRawLeafs(t *testing.T) {
    // 调用 testAddWPosInfo 函数，传入 true 作为参数
    testAddWPosInfo(t, true)
}

// testBlockstore 结构体，包含了 GCBlockstore、expectedPath、t、countAtOffsetZero 和 countAtOffsetNonZero 字段
type testBlockstore struct {
    blockstore.GCBlockstore
    expectedPath         string
    t                    *testing.T
    countAtOffsetZero    int
    countAtOffsetNonZero int
}

// Put 方法，用于向块存储中存储块
func (bs *testBlockstore) Put(ctx context.Context, block blocks.Block) error {
    // 调用 CheckForPosInfo 方法，传入 block 作为参数
    bs.CheckForPosInfo(block)
    // 调用 GCBlockstore 的 Put 方法，传入 ctx 和 block 作为参数
    return bs.GCBlockstore.Put(ctx, block)
}

// PutMany 方法，用于向块存储中存储多个块
func (bs *testBlockstore) PutMany(ctx context.Context, blocks []blocks.Block) error {
    // 遍历 blocks 切片
    for _, blk := range blocks {
        // 调用 CheckForPosInfo 方法，传入 blk 作为参数
        bs.CheckForPosInfo(blk)
    }
    // 调用 GCBlockstore 的 PutMany 方法，传入 ctx 和 blocks 作为参数
    return bs.GCBlockstore.PutMany(ctx, blocks)
}

// CheckForPosInfo 方法，用于检查块是否包含位置信息
func (bs *testBlockstore) CheckForPosInfo(block blocks.Block) {
    // 尝试将 block 转换为 *pi.FilestoreNode 类型，并判断是否成功
    fsn, ok := block.(*pi.FilestoreNode)
    if ok {
        // 获取位置信息
        posInfo := fsn.PosInfo
        // 检查位置信息的完整路径是否与期望路径相符
        if posInfo.FullPath != bs.expectedPath {
            bs.t.Fatal("PosInfo does not have the expected path")
        }
        // 根据偏移量是否为 0，更新计数器
        if posInfo.Offset == 0 {
            bs.countAtOffsetZero += 1
        } else {
            bs.countAtOffsetNonZero += 1
        }
    }
}

// dummyFileInfo 结构体，包含了 name、size 和 modTime 字段
type dummyFileInfo struct {
    name    string
    size    int64
    modTime time.Time
}

// Name 方法，用于返回文件名
func (fi *dummyFileInfo) Name() string       { return fi.name }
// Size 方法，用于返回文件大小
func (fi *dummyFileInfo) Size() int64        { return fi.size }
// Mode 方法，用于返回文件模式
func (fi *dummyFileInfo) Mode() os.FileMode  { return 0 }
// ModTime 方法，用于返回修改时间
func (fi *dummyFileInfo) ModTime() time.Time { return fi.modTime }
// IsDir 方法，用于判断是否为目录
func (fi *dummyFileInfo) IsDir() bool        { return false }
// Sys 方法，用于返回接口
func (fi *dummyFileInfo) Sys() interface{}   { return nil }
```