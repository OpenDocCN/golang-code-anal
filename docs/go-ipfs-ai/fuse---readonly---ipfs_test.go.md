# `kubo\fuse\readonly\ipfs_test.go`

```
// 如果不是在 nofuse、openbsd、netbsd、plan9 环境下构建，则包含此文件
// 导入 readonly 包
package readonly

// 导入所需的包
import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "context"  // 导入 context 包，用于控制程序的上下文
    "errors"  // 导入 errors 包，用于处理错误
    "fmt"  // 导入 fmt 包，用于格式化输入输出
    "io"  // 导入 io 包，用于进行 I/O 操作
    "math/rand"  // 导入 math/rand 包，用于生成随机数
    "os"  // 导入 os 包，提供了对操作系统功能的访问
    gopath "path"  // 导入 path 包，用于处理文件路径
    "strings"  // 导入 strings 包，用于处理字符串
    "sync"  // 导入 sync 包，提供了同步原语的支持
    "testing"  // 导入 testing 包，用于编写测试函数

    "bazil.org/fuse"  // 导入 fuse 包，用于文件系统实现
    core "github.com/ipfs/kubo/core"  // 导入 core 包，用于 IPFS 核心功能
    coreapi "github.com/ipfs/kubo/core/coreapi"  // 导入 coreapi 包，用于 IPFS 核心 API
    coremock "github.com/ipfs/kubo/core/mock"  // 导入 coremock 包，用于 IPFS 核心模拟

    fstest "bazil.org/fuse/fs/fstestutil"  // 导入 fstest 包，用于文件系统测试
    chunker "github.com/ipfs/boxo/chunker"  // 导入 chunker 包，用于分块处理
    "github.com/ipfs/boxo/files"  // 导入 files 包，用于文件操作
    dag "github.com/ipfs/boxo/ipld/merkledag"  // 导入 dag 包，用于 MerkleDAG 操作
    importer "github.com/ipfs/boxo/ipld/unixfs/importer"  // 导入 importer 包，用于导入操作
    uio "github.com/ipfs/boxo/ipld/unixfs/io"  // 导入 uio 包，用于 UnixFS I/O 操作
    "github.com/ipfs/boxo/path"  // 导入 path 包，用于路径操作
    u "github.com/ipfs/boxo/util"  // 导入 util 包，提供了一些实用函数
    ipld "github.com/ipfs/go-ipld-format"  // 导入 ipld 包，用于 IPLD 格式操作
    ci "github.com/libp2p/go-libp2p-testing/ci"  // 导入 ci 包，用于持续集成测试
)

// 可能跳过 FUSE 测试
func maybeSkipFuseTests(t *testing.T) {
    if ci.NoFuse() {
        t.Skip("Skipping FUSE tests")
    }
}

// 生成指定大小的随机对象
func randObj(t *testing.T, nd *core.IpfsNode, size int64) (ipld.Node, []byte) {
    buf := make([]byte, size)  // 创建指定大小的字节缓冲区
    _, err := io.ReadFull(u.NewTimeSeededRand(), buf)  // 从随机源中读取数据填充缓冲区
    if err != nil {
        t.Fatal(err)  // 如果出现错误，记录错误并终止测试
    }
    read := bytes.NewReader(buf)  // 创建字节缓冲区的读取器
    obj, err := importer.BuildTrickleDagFromReader(nd.DAG, chunker.DefaultSplitter(read))  // 从读取器构建 TrickleDAG 对象
    if err != nil {
        t.Fatal(err)  // 如果出现错误，记录错误并终止测试
    }

    return obj, buf  // 返回 TrickleDAG 对象和字节缓冲区
}

// 设置 IPFS 测试环境
func setupIpfsTest(t *testing.T, node *core.IpfsNode) (*core.IpfsNode, *fstest.Mount) {
    t.Helper()  // 标记该函数是测试辅助函数
    maybeSkipFuseTests(t)  // 可能跳过 FUSE 测试

    var err error
    if node == nil {
        node, err = coremock.NewMockNode()  // 创建一个新的模拟节点
        if err != nil {
            t.Fatal(err)  // 如果出现错误，记录错误并终止测试
        }
    }

    fs := NewFileSystem(node)  // 创建新的文件系统
    mnt, err := fstest.MountedT(t, fs, nil)  // 挂载文件系统
    if err == fuse.ErrOSXFUSENotFound {
        t.Skip(err)  // 如果是因为找不到 OSXFUSE 而出现错误，则跳过测试
    }
    if err != nil {
        t.Fatalf("error mounting temporary directory: %v", err)  // 如果出现其他错误，记录错误并终止测试
    }

    return node, mnt  // 返回节点和挂载点
}

// 测试写入对象并通过 FUSE 读取
func TestIpfsBasicRead(t *testing.T) {
    # 如果是测试短路状态，立即跳过当前测试
    if testing.Short() {
        t.SkipNow()
    }
    # 设置 IPFS 测试环境，并获取节点和挂载点
    nd, mnt := setupIpfsTest(t, nil)
    # 延迟关闭挂载点
    defer mnt.Close()

    # 生成随机对象，并返回文件信息和数据
    fi, data := randObj(t, nd, 10000)
    # 获取文件的 CID
    k := fi.Cid()
    # 生成文件名
    fname := gopath.Join(mnt.Dir, k.String())
    # 读取文件内容到缓冲区
    rbuf, err := os.ReadFile(fname)
    # 如果读取出错，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    # 比较读取的内容和原始数据是否相等，如果不相等，输出错误信息并终止测试
    if !bytes.Equal(rbuf, data) {
        t.Fatal("Incorrect Read!")
    }
}
// 获取节点路径
func getPaths(t *testing.T, ipfs *core.IpfsNode, name string, n *dag.ProtoNode) []string {
    // 如果节点没有链接，则返回节点名称
    if len(n.Links()) == 0 {
        return []string{name}
    }
    var out []string
    // 遍历节点链接
    for _, lnk := range n.Links() {
        // 获取链接节点
        child, err := lnk.GetNode(ipfs.Context(), ipfs.DAG)
        if err != nil {
            t.Fatal(err)
        }
        // 判断节点类型
        childpb, ok := child.(*dag.ProtoNode)
        if !ok {
            t.Fatal(dag.ErrNotProtobuf)
        }
        // 递归获取路径
        sub := getPaths(t, ipfs, gopath.Join(name, lnk.Name), childpb)
        out = append(out, sub...)
    }
    return out
}

// 对系统进行大量并发读取以测试压力
func TestIpfsStressRead(t *testing.T) {
    // 如果测试时间较短，则跳过测试
    if testing.Short() {
        t.SkipNow()
    }
    // 设置IPFS测试环境
    nd, mnt := setupIpfsTest(t, nil)
    defer mnt.Close()

    // 创建IPFS核心API
    api, err := coreapi.NewCoreAPI(nd)
    if err != nil {
        t.Fatal(err)
    }

    var nodes []ipld.Node
    var paths []string

    nobj := 50
    ndiriter := 50

    // 创建大量对象
    for i := 0; i < nobj; i++ {
        fi, _ := randObj(t, nd, rand.Int63n(50000))
        nodes = append(nodes, fi)
        paths = append(paths, fi.Cid().String())
    }

    // 创建大量目录
    for i := 0; i < ndiriter; i++ {
        db := uio.NewDirectory(nd.DAG)
        for j := 0; j < 1+rand.Intn(10); j++ {
            name := fmt.Sprintf("child%d", j)
            // 添加子节点
            err := db.AddChild(nd.Context(), name, nodes[rand.Intn(len(nodes))])
            if err != nil {
                t.Fatal(err)
            }
        }
        newdir, err := db.GetNode()
        if err != nil {
            t.Fatal(err)
        }
        // 添加目录节点
        err = nd.DAG.Add(nd.Context(), newdir)
        if err != nil {
            t.Fatal(err)
        }
        nodes = append(nodes, newdir)
        // 获取目录路径
        npaths := getPaths(t, nd, newdir.Cid().String(), newdir.(*dag.ProtoNode))
        paths = append(paths, npaths...)
    }

    // 并发读取
    wg := sync.WaitGroup{}
    // 创建一个用于传递错误的通道
    errs := make(chan error)

    // 循环4次，每次增加等待组的计数
    for s := 0; s < 4; s++ {
        wg.Add(1)
        // 启动一个并发的匿名函数
        go func() {
            // 在函数结束时减少等待组的计数
            defer wg.Done()

            // 循环2000次
            for i := 0; i < 2000; i++ {
                // 从路径列表中随机选择一个路径，创建一个路径对象
                item, err := path.NewPath(paths[rand.Intn(len(paths))])
                // 如果出现错误，将错误发送到errs通道并继续下一次循环
                if err != nil {
                    errs <- err
                    continue
                }

                // 获取相对路径并拼接成完整文件路径
                relpath := strings.Replace(item.String(), item.Namespace(), "", 1)
                fname := gopath.Join(mnt.Dir, relpath)

                // 读取文件内容到rbuf
                rbuf, err := os.ReadFile(fname)
                // 如果出现错误，将错误发送到errs通道
                if err != nil {
                    errs <- err
                }

                // 创建一个上下文，用于控制Unixfs的Get操作
                ctx, cancelFunc := context.WithCancel(context.Background())

                // 从Unixfs获取数据
                read, err := api.Unixfs().Get(ctx, item)
                // 如果出现错误，将错误发送到errs通道
                if err != nil {
                    errs <- err
                }

                // 读取文件数据
                data, err := io.ReadAll(read.(files.File))
                // 如果出现错误，将错误发送到errs通道
                if err != nil {
                    errs <- err
                }

                // 取消上下文
                cancelFunc()

                // 比较读取的数据和文件内容是否一致，如果不一致，将错误发送到errs通道
                if !bytes.Equal(rbuf, data) {
                    errs <- errors.New("incorrect read")
                }
            }
        }()
    }

    // 启动一个并发的匿名函数，等待所有任务完成后关闭errs通道
    go func() {
        wg.Wait()
        close(errs)
    }()

    // 从errs通道中读取错误，如果有错误则调用t.Fatal()函数
    for err := range errs {
        if err != nil {
            t.Fatal(err)
        }
    }
// 测试写入文件并读取回来。
func TestIpfsBasicDirRead(t *testing.T) {
    if testing.Short() {
        t.SkipNow()
    }
    nd, mnt := setupIpfsTest(t, nil)
    defer mnt.Close()

    // 创建一个 'file'
    fi, data := randObj(t, nd, 10000)

    // 创建一个目录并将文件放入其中
    db := uio.NewDirectory(nd.DAG)
    err := db.AddChild(nd.Context(), "actual", fi)
    if err != nil {
        t.Fatal(err)
    }

    d1nd, err := db.GetNode()
    if err != nil {
        t.Fatal(err)
    }

    err = nd.DAG.Add(nd.Context(), d1nd)
    if err != nil {
        t.Fatal(err)
    }

    dirname := gopath.Join(mnt.Dir, d1nd.Cid().String())
    fname := gopath.Join(dirname, "actual")
    rbuf, err := os.ReadFile(fname)
    if err != nil {
        t.Fatal(err)
    }

    dirents, err := os.ReadDir(dirname)
    if err != nil {
        t.Fatal(err)
    }
    if len(dirents) != 1 {
        t.Fatal("Bad directory entry count")
    }
    if dirents[0].Name() != "actual" {
        t.Fatal("Bad directory entry")
    }

    if !bytes.Equal(rbuf, data) {
        t.Fatal("Incorrect Read!")
    }
}

// 测试确保文件系统正确报告文件大小。
func TestFileSizeReporting(t *testing.T) {
    if testing.Short() {
        t.SkipNow()
    }
    nd, mnt := setupIpfsTest(t, nil)
    defer mnt.Close()

    fi, data := randObj(t, nd, 10000)
    k := fi.Cid()

    fname := gopath.Join(mnt.Dir, k.String())

    finfo, err := os.Stat(fname)
    if err != nil {
        t.Fatal(err)
    }

    if finfo.Size() != int64(len(data)) {
        t.Fatal("Read incorrect size from stat!")
    }
}
```