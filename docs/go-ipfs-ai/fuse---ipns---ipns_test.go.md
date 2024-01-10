# `kubo\fuse\ipns\ipns_test.go`

```
//go:build !nofuse && !openbsd && !netbsd && !plan9
// +build !nofuse,!openbsd,!netbsd,!plan9

package ipns

import (
    "bytes" // 导入 bytes 包，用于操作字节切片
    "context" // 导入 context 包，用于处理请求的上下文
    "fmt" // 导入 fmt 包，用于格式化输入输出
    "io" // 导入 io 包，提供了基本的接口和函数，用于读写数据
    mrand "math/rand" // 导入 math/rand 包，并重命名为 mrand，用于生成伪随机数
    "os" // 导入 os 包，提供了操作系统功能的函数
    "sync" // 导入 sync 包，提供了并发编程的基本同步原语
    "testing" // 导入 testing 包，用于编写测试函数

    "bazil.org/fuse" // 导入 bazil.org/fuse 包，提供了 FUSE 文件系统的实现

    core "github.com/ipfs/kubo/core" // 导入 github.com/ipfs/kubo/core 包，并重命名为 core
    coreapi "github.com/ipfs/kubo/core/coreapi" // 导入 github.com/ipfs/kubo/core/coreapi 包，并重命名为 coreapi

    fstest "bazil.org/fuse/fs/fstestutil" // 导入 bazil.org/fuse/fs/fstestutil 包，并重命名为 fstest
    u "github.com/ipfs/boxo/util" // 导入 github.com/ipfs/boxo/util 包，并重命名为 u
    racedet "github.com/ipfs/go-detect-race" // 导入 github.com/ipfs/go-detect-race 包，并重命名为 racedet
    ci "github.com/libp2p/go-libp2p-testing/ci" // 导入 github.com/libp2p/go-libp2p-testing/ci 包，并重命名为 ci
)

// maybeSkipFuseTests 根据环境变量判断是否跳过 FUSE 测试
func maybeSkipFuseTests(t *testing.T) {
    if ci.NoFuse() {
        t.Skip("Skipping FUSE tests")
    }
}

// randBytes 生成指定大小的随机字节切片
func randBytes(size int) []byte {
    b := make([]byte, size)
    _, err := io.ReadFull(u.NewTimeSeededRand(), b)
    if err != nil {
        panic(err)
    }
    return b
}

// mkdir 创建目录
func mkdir(t *testing.T, path string) {
    err := os.Mkdir(path, os.ModeDir)
    if err != nil {
        t.Fatal(err)
    }
}

// writeFileOrFail 写入指定大小的文件，如果失败则触发测试失败
func writeFileOrFail(t *testing.T, size int, path string) []byte {
    data, err := writeFile(size, path)
    if err != nil {
        t.Fatal(err)
    }
    return data
}

// writeFile 写入指定大小的文件
func writeFile(size int, path string) ([]byte, error) {
    data := randBytes(size)
    err := os.WriteFile(path, data, 0o666)
    return data, err
}

// verifyFile 验证文件内容是否与期望一致
func verifyFile(t *testing.T, path string, wantData []byte) {
    isData, err := os.ReadFile(path)
    if err != nil {
        t.Fatal(err)
    }
    if len(isData) != len(wantData) {
        t.Fatal("Data not equal - length check failed")
    }
    if !bytes.Equal(isData, wantData) {
        t.Fatal("Data not equal")
    }
}

// checkExists 检查文件或目录是否存在
func checkExists(t *testing.T, path string) {
    _, err := os.Stat(path)
    if err != nil {
        t.Fatal(err)
    }
}

// closeMount 关闭挂载点
func closeMount(mnt *mountWrap) {
    if err := recover(); err != nil {
        log.Error("Recovered panic")
        log.Error(err)
    }
    mnt.Close()
}

// mountWrap 封装了挂载点和文件系统
type mountWrap struct {
    *fstest.Mount
    Fs *FileSystem
}

// Close 关闭挂载点和文件系统
func (m *mountWrap) Close() error {
    m.Fs.Destroy()
    m.Mount.Close()
    return nil
}
// 设置 IPNS 测试环境，返回 IPFS 节点和挂载包装
func setupIpnsTest(t *testing.T, node *core.IpfsNode) (*core.IpfsNode, *mountWrap) {
    t.Helper()
    // 可能跳过 FUSE 测试
    maybeSkipFuseTests(t)

    var err error
    if node == nil {
        // 如果节点为空，创建新节点
        node, err = core.NewNode(context.Background(), &core.BuildCfg{})
        if err != nil {
            t.Fatal(err)
        }

        // 初始化密钥空间
        err = InitializeKeyspace(node, node.PrivateKey)
        if err != nil {
            t.Fatal(err)
        }
    }

    // 创建 CoreAPI
    coreAPI, err := coreapi.NewCoreAPI(node)
    if err != nil {
        t.Fatal(err)
    }

    // 创建文件系统
    fs, err := NewFileSystem(node.Context(), coreAPI, "", "")
    if err != nil {
        t.Fatal(err)
    }
    // 挂载文件系统
    mnt, err := fstest.MountedT(t, fs, nil)
    if err == fuse.ErrOSXFUSENotFound {
        t.Skip(err)
    }
    if err != nil {
        t.Fatalf("error mounting at temporary directory: %v", err)
    }

    return node, &mountWrap{
        Mount: mnt,
        Fs:    fs,
    }
}

// 测试 IPNS 本地链接
func TestIpnsLocalLink(t *testing.T) {
    nd, mnt := setupIpnsTest(t, nil)
    defer mnt.Close()
    name := mnt.Dir + "/local"

    // 检查文件是否存在
    checkExists(t, name)

    // 读取符号链接
    linksto, err := os.Readlink(name)
    if err != nil {
        t.Fatal(err)
    }

    // 检查链接是否有效
    if linksto != nd.Identity.String() {
        t.Fatal("Link invalid")
    }
}

// 测试写入文件并读取
func TestIpnsBasicIO(t *testing.T) {
    if testing.Short() {
        t.SkipNow()
    }
    nd, mnt := setupIpnsTest(t, nil)
    defer closeMount(mnt)

    fname := mnt.Dir + "/local/testfile"
    data := writeFileOrFail(t, 10, fname)

    rbuf, err := os.ReadFile(fname)
    if err != nil {
        t.Fatal(err)
    }

    // 检查读取的数据是否正确
    if !bytes.Equal(rbuf, data) {
        t.Fatal("Incorrect Read!")
    }

    fname2 := mnt.Dir + "/" + nd.Identity.String() + "/testfile"
    rbuf, err = os.ReadFile(fname2)
    if err != nil {
        t.Fatal(err)
    }

    // 检查读取的数据是否正确
    if !bytes.Equal(rbuf, data) {
        t.Fatal("Incorrect Read!")
    }
}

// 测试确保文件更改在 IPNS 挂载期间持续存在
func TestFilePersistence(t *testing.T) {
    // 如果测试运行时间较短，则跳过测试
    if testing.Short() {
        t.SkipNow()
    }
    // 设置 IPNS 测试环境
    node, mnt := setupIpnsTest(t, nil)

    // 设置文件名和数据
    fname := "/local/atestfile"
    data := writeFileOrFail(t, 127, mnt.Dir+fname)

    // 关闭文件系统
    mnt.Close()

    t.Log("Closed, opening new fs")
    // 重新设置 IPNS 测试环境
    _, mnt = setupIpnsTest(t, node)
    defer mnt.Close()

    // 读取文件数据
    rbuf, err := os.ReadFile(mnt.Dir + fname)
    if err != nil {
        t.Fatal(err)
    }

    // 检查文件数据是否一致
    if !bytes.Equal(rbuf, data) {
        t.Fatalf("File data changed between mounts! sizes differ: %d != %d", len(data), len(rbuf))
    }
}

func TestMultipleDirs(t *testing.T) {
    // 设置 IPNS 测试环境
    node, mnt := setupIpnsTest(t, nil)

    t.Log("make a top level dir")
    // 创建顶层目录
    dir1 := "/local/test1"
    mkdir(t, mnt.Dir+dir1)

    // 检查目录是否存在
    checkExists(t, mnt.Dir+dir1)

    t.Log("write a file in it")
    // 在目录中写入文件
    data1 := writeFileOrFail(t, 4000, mnt.Dir+dir1+"/file1")

    // 验证文件是否存在并且数据正确
    verifyFile(t, mnt.Dir+dir1+"/file1", data1)

    t.Log("sub directory")
    // 创建子目录
    mkdir(t, mnt.Dir+dir1+"/dir2")

    // 检查子目录是否存在
    checkExists(t, mnt.Dir+dir1+"/dir2")

    t.Log("file in that subdirectory")
    // 在子目录中写入文件
    data2 := writeFileOrFail(t, 5000, mnt.Dir+dir1+"/dir2/file2")

    // 验证文件是否存在并且数据正确
    verifyFile(t, mnt.Dir+dir1+"/dir2/file2", data2)

    // 关闭文件系统
    mnt.Close()
    t.Log("closing mount, then restarting")

    // 重新设置 IPNS 测试环境
    _, mnt = setupIpnsTest(t, node)

    // 检查目录是否存在
    checkExists(t, mnt.Dir+dir1)

    // 验证文件数据是否正确
    verifyFile(t, mnt.Dir+dir1+"/file1", data1)

    // 验证文件数据是否正确
    verifyFile(t, mnt.Dir+dir1+"/dir2/file2", data2)
    // 关闭文件系统
    mnt.Close()
}

// 测试确保文件系统正确报告文件大小。
func TestFileSizeReporting(t *testing.T) {
    // 如果测试运行时间较短，则跳过测试
    if testing.Short() {
        t.SkipNow()
    }
    // 设置 IPNS 测试环境
    _, mnt := setupIpnsTest(t, nil)
    defer mnt.Close()

    // 设置文件名和数据
    fname := mnt.Dir + "/local/sizecheck"
    data := writeFileOrFail(t, 5555, fname)

    // 获取文件信息
    finfo, err := os.Stat(fname)
    if err != nil {
        t.Fatal(err)
    }

    // 检查文件大小是否正确
    if finfo.Size() != int64(len(data)) {
        t.Fatal("Read incorrect size from stat!")
    }
}

// 测试确保不能创建同名的多个条目。
# 测试重复创建目录时的失败情况
func TestDoubleEntryFailure(t *testing.T) {
    # 如果测试运行时间较短，则跳过该测试
    if testing.Short() {
        t.SkipNow()
    }
    # 设置测试环境并返回测试目录
    _, mnt := setupIpnsTest(t, nil)
    # 延迟关闭测试目录
    defer mnt.Close()

    # 定义目录名
    dname := mnt.Dir + "/local/thisisadir"
    # 创建新目录
    err := os.Mkdir(dname, 0o777)
    # 如果创建目录失败，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    # 再次尝试创建同名目录
    err = os.Mkdir(dname, 0o777)
    # 如果成功创建目录，则输出错误信息并终止测试
    if err == nil {
        t.Fatal("Should have gotten error one creating new directory.")
    }
}

# 测试追加文件内容
func TestAppendFile(t *testing.T) {
    # 如果测试运行时间较短，则跳过该测试
    if testing.Short() {
        t.SkipNow()
    }
    # 设置测试环境并返回测试目录
    _, mnt := setupIpnsTest(t, nil)
    # 延迟关闭测试目录
    defer mnt.Close()

    # 定义文件名
    fname := mnt.Dir + "/local/file"
    # 写入指定大小的文件内容，并返回写入的数据
    data := writeFileOrFail(t, 1300, fname)

    # 以读写追加模式打开文件
    fi, err := os.OpenFile(fname, os.O_RDWR|os.O_APPEND, 0o666)
    # 如果打开文件失败，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    # 生成随机数据
    nudata := randBytes(500)

    # 向文件中写入数据
    n, err := fi.Write(nudata)
    # 如果写入数据失败，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }
    # 关闭文件
    err = fi.Close()
    # 如果关闭文件失败，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    # 检查写入的数据长度是否与预期一致
    if n != len(nudata) {
        t.Fatal("Failed to write enough bytes.")
    }

    # 将新数据追加到原数据中
    data = append(data, nudata...)

    # 读取文件内容
    rbuf, err := os.ReadFile(fname)
    # 如果读取文件内容失败，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }
    # 检查读取的文件内容是否与预期一致
    if !bytes.Equal(rbuf, data) {
        t.Fatal("Data inconsistent!")
    }
}

# 测试并发写入
func TestConcurrentWrites(t *testing.T) {
    # 如果测试运行时间较短，则跳过该测试
    if testing.Short() {
        t.SkipNow()
    }
    # 设置测试环境并返回测试目录
    _, mnt := setupIpnsTest(t, nil)
    # 延迟关闭测试目录
    defer mnt.Close()

    # 定义并发写入的参与者数量
    nactors := 4
    # 每个参与者创建的文件数量
    filesPerActor := 400
    # 每个文件的大小
    fileSize := 2000

    # 创建一个三维字节切片用于存储数据
    data := make([][][]byte, nactors)

    # 如果检测到竞争条件，则修改并发参数
    if racedet.WithRace() {
        nactors = 2
        filesPerActor = 50
    }

    # 创建一个同步等待组
    wg := sync.WaitGroup{}
    // 遍历每个演员
    for i := 0; i < nactors; i++ {
        // 为每个演员创建一个二维字节切片
        data[i] = make([][]byte, filesPerActor)
        // 增加等待组的计数
        wg.Add(1)
        // 启动一个 goroutine 来并发执行文件写入操作
        go func(n int) {
            // 在函数退出时减少等待组的计数
            defer wg.Done()
            // 遍历每个演员需要写入的文件
            for j := 0; j < filesPerActor; j++ {
                // 调用 writeFile 函数写入文件，并获取写入的数据和可能的错误
                out, err := writeFile(fileSize, mnt.Dir+fmt.Sprintf("/local/%dFILE%d", n, j))
                // 如果有错误，记录错误并继续下一个文件的写入
                if err != nil {
                    t.Error(err)
                    continue
                }
                // 将写入的数据保存到对应的位置
                data[n][j] = out
            }
        }(i)
    }
    // 等待所有 goroutine 完成
    wg.Wait()

    // 遍历每个演员的每个文件
    for i := 0; i < nactors; i++ {
        for j := 0; j < filesPerActor; j++ {
            // 如果数据为空，表示之前已经报告了错误，直接跳过
            if data[i][j] == nil {
                // 错误已经报告过，直接跳过
                continue
            }
            // 验证写入的文件内容是否正确
            verifyFile(t, mnt.Dir+fmt.Sprintf("/local/%dFILE%d", i, j), data[i][j])
        }
    }
}
// TestFSThrash 是一个测试函数，用于测试文件系统的性能
func TestFSThrash(t *testing.T) {
    // 创建一个空的文件字典
    files := make(map[string][]byte)

    // 如果测试运行时间较短，则跳过测试
    if testing.Short() {
        t.SkipNow()
    }
    // 设置 IPNS 测试环境
    _, mnt := setupIpnsTest(t, nil)
    // 延迟关闭测试环境
    defer mnt.Close()

    // 设置基础目录
    base := mnt.Dir + "/local"
    dirs := []string{base}
    // 设置目录锁和文件锁
    dirlock := sync.RWMutex{}
    filelock := sync.Mutex{}

    // 设置目录和文件工作线程数
    ndirWorkers := 2
    nfileWorkers := 2

    // 设置目录和文件数量
    ndirs := 100
    nfiles := 200

    // 设置等待组
    wg := sync.WaitGroup{}

    // 创建工作线程用于创建目录
    for i := 0; i < ndirWorkers; i++ {
        wg.Add(1)
        go func(worker int) {
            defer wg.Done()
            for j := 0; j < ndirs; j++ {
                dirlock.RLock()
                n := mrand.Intn(len(dirs))
                dir := dirs[n]
                dirlock.RUnlock()

                newDir := fmt.Sprintf("%s/dir%d-%d", dir, worker, j)
                err := os.Mkdir(newDir, os.ModeDir)
                if err != nil {
                    t.Error(err)
                    continue
                }
                dirlock.Lock()
                dirs = append(dirs, newDir)
                dirlock.Unlock()
            }
        }(i)
    }

    // 创建工作线程用于创建文件
    for i := 0; i < nfileWorkers; i++ {
        wg.Add(1)
        go func(worker int) {
            defer wg.Done()
            for j := 0; j < nfiles; j++ {
                dirlock.RLock()
                n := mrand.Intn(len(dirs))
                dir := dirs[n]
                dirlock.RUnlock()

                newFileName := fmt.Sprintf("%s/file%d-%d", dir, worker, j)

                data, err := writeFile(2000+mrand.Intn(5000), newFileName)
                if err != nil {
                    t.Error(err)
                    continue
                }
                filelock.Lock()
                files[newFileName] = data
                filelock.Unlock()
            }
        }(i)
    }

    // 等待所有工作线程完成
    wg.Wait()
}
    # 遍历文件名和数据的映射关系
    for name, data := range files {
        # 读取文件内容到 out 变量
        out, err := os.ReadFile(name)
        # 如果读取文件出现错误，记录错误信息
        if err != nil {
            t.Error(err)
        }
        
        # 检查读取的文件内容和给定的数据是否相等，如果不相等，记录错误信息
        if !bytes.Equal(data, out) {
            t.Errorf("Data didn't match in %s: expected %v, got %v", name, data, out)
        }
    }
// 测试逐字节写入中等大小文件
func TestMultiWrite(t *testing.T) {
    // 如果是短测试，则跳过
    if testing.Short() {
        t.SkipNow()
    }

    // 设置 IPNS 测试环境
    _, mnt := setupIpnsTest(t, nil)
    // 延迟关闭测试环境
    defer mnt.Close()

    // 创建文件路径
    fpath := mnt.Dir + "/local/file"
    // 创建文件
    fi, err := os.Create(fpath)
    if err != nil {
        t.Fatal(err)
    }

    // 生成随机数据
    data := randBytes(1001)
    // 逐字节写入文件
    for i := 0; i < len(data); i++ {
        n, err := fi.Write(data[i : i+1])
        if err != nil {
            t.Fatal(err)
        }
        if n != 1 {
            t.Fatal("Somehow wrote the wrong number of bytes! (n != 1)")
        }
    }
    // 关闭文件
    fi.Close()

    // 读取文件内容
    rbuf, err := os.ReadFile(fpath)
    if err != nil {
        t.Fatal(err)
    }

    // 检查文件内容是否与写入的数据一致
    if !bytes.Equal(rbuf, data) {
        t.Fatal("File on disk did not match bytes written")
    }
}
```