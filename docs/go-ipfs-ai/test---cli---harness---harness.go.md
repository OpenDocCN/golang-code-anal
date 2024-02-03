# `kubo\test\cli\harness\harness.go`

```go
package harness

import (
    "errors"  // 导入错误处理包
    "fmt"  // 导入格式化包
    "os"  // 导入操作系统包
    "path/filepath"  // 导入文件路径包
    "strings"  // 导入字符串处理包
    "testing"  // 导入测试包
    "time"  // 导入时间包

    logging "github.com/ipfs/go-log/v2"  // 导入日志包
    . "github.com/ipfs/kubo/test/cli/testutils"  // 导入测试工具包
    "github.com/libp2p/go-libp2p/core/peer"  // 导入对等网络包
    "github.com/multiformats/go-multiaddr"  // 导入多地址包
)

// Harness tracks state for a test, such as temp dirs and IFPS nodes, and cleans them up after the test.
type Harness struct {
    Dir       string  // 测试目录
    IPFSBin   string  // IPFS二进制文件路径
    Runner    *Runner  // 运行器
    NodesRoot string  // 节点根目录
    Nodes     Nodes  // 节点
}

// TODO: use zaptest.NewLogger(t) instead
func EnableDebugLogging() {
    err := logging.SetLogLevel("testharness", "DEBUG")  // 设置日志级别为DEBUG
    if err != nil {
        panic(err)  // 如果设置日志级别失败，抛出异常
    }
}

// NewT constructs a harness that cleans up after the given test is done.
func NewT(t *testing.T, options ...func(h *Harness)) *Harness {
    h := New(options...)  // 创建一个新的测试环境
    t.Cleanup(h.Cleanup)  // 在测试结束时清理环境
    return h  // 返回测试环境
}

func New(options ...func(h *Harness)) *Harness {
    h := &Harness{Runner: &Runner{Env: osEnviron()}}  // 创建测试环境

    // walk up to find the root dir, from which we can locate the binary
    wd, err := os.Getwd()  // 获取当前工作目录
    if err != nil {
        panic(err)  // 如果获取失败，抛出异常
    }
    goMod := FindUp("go.mod", wd)  // 查找包含"go.mod"文件的根目录
    if goMod == "" {
        panic("unable to find root dir")  // 如果找不到根目录，抛出异常
    }
    rootDir := filepath.Dir(goMod)  // 获取根目录路径
    h.IPFSBin = filepath.Join(rootDir, "cmd", "ipfs", "ipfs")  // 设置IPFS二进制文件路径

    // setup working dir
    tmpDir, err := os.MkdirTemp("", "")  // 创建临时工作目录
    if err != nil {
        log.Panicf("error creating temp dir: %s", err)  // 如果创建临时目录失败，抛出异常
    }
    h.Dir = tmpDir  // 设置测试目录
    h.Runner.Dir = h.Dir  // 设置运行器的目录

    h.NodesRoot = filepath.Join(h.Dir, ".nodes")  // 设置节点根目录

    // apply any customizations
    // this should happen after all initialization
    for _, o := range options {
        o(h)  // 应用任何自定义设置
    }

    return h  // 返回测试环境
}

func osEnviron() map[string]string {
    m := map[string]string{}  // 创建环境变量映射
    for _, entry := range os.Environ() {
        split := strings.Split(entry, "=")  // 分割环境变量键值对
        m[split[0]] = split[1]  // 添加到环境变量映射中
    }
    return m  // 返回环境变量映射
}

func (h *Harness) NewNode() *Node {
    # 获取节点列表的长度，并赋值给 nodeID
    nodeID := len(h.Nodes)
    # 使用给定的 IPFSBin 和 NodesRoot 构建一个节点
    node := BuildNode(h.IPFSBin, h.NodesRoot, nodeID)
    # 将新构建的节点添加到节点列表中
    h.Nodes = append(h.Nodes, node)
    # 返回新构建的节点
    return node
// NewNodes 创建指定数量的新节点，并返回节点数组
func (h *Harness) NewNodes(count int) Nodes {
    var newNodes []*Node
    for i := 0; i < count; i++ {
        newNodes = append(newNodes, h.NewNode())
    }
    return newNodes
}

// WriteToTemp 将给定内容写入一个保证唯一的临时文件，并返回其路径
func (h *Harness) WriteToTemp(contents string) string {
    f := h.TempFile()
    _, err := f.WriteString(contents)
    if err != nil {
        log.Panicf("writing to temp file: %s", err.Error())
    }
    err = f.Close()
    if err != nil {
        log.Panicf("closing temp file: %s", err.Error())
    }
    return f.Name()
}

// TempFile 创建一个新的唯一临时文件
func (h *Harness) TempFile() *os.File {
    f, err := os.CreateTemp(h.Dir, "")
    if err != nil {
        log.Panicf("creating temp file: %s", err.Error())
    }
    return f
}

// WriteFile 根据文件名和内容写入文件
// 文件名必须是相对路径，否则会引发 panic
func (h *Harness) WriteFile(filename, contents string) {
    if filepath.IsAbs(filename) {
        log.Panicf("%s must be a relative path", filename)
    }
    absPath := filepath.Join(h.Runner.Dir, filename)
    err := os.MkdirAll(filepath.Dir(absPath), 0o777)
    if err != nil {
        log.Panicf("creating intermediate dirs for %q: %s", filename, err.Error())
    }
    err = os.WriteFile(absPath, []byte(contents), 0o644)
    if err != nil {
        log.Panicf("writing %q (%q): %s", filename, absPath, err.Error())
    }
}

// WaitForFile 等待文件出现，超时时间为指定时长
func WaitForFile(path string, timeout time.Duration) error {
    start := time.Now()
    timer := time.NewTimer(timeout)
    ticker := time.NewTicker(1 * time.Millisecond)
    defer timer.Stop()
    defer ticker.Stop()
    # 无限循环，等待两个通道的消息
    for {
        # 从定时器通道接收消息
        select {
        case <-timer.C:
            # 返回超时错误信息
            return fmt.Errorf("timeout waiting for %s after %v", path, time.Since(start))
        # 从定时器通道接收消息
        case <-ticker.C:
            # 检查文件状态
            _, err := os.Stat(path)
            # 如果没有错误，返回空
            if err == nil {
                return nil
            }
            # 如果文件不存在，继续循环
            if errors.Is(err, os.ErrNotExist) {
                continue
            }
            # 返回等待文件时发生的错误
            return fmt.Errorf("error waiting for %s: %w", path, err)
        }
    }
// 创建目录的方法，接受一个或多个路径参数
func (h *Harness) Mkdirs(paths ...string) {
    // 遍历所有路径
    for _, path := range paths {
        // 如果路径是绝对路径，则抛出错误
        if filepath.IsAbs(path) {
            log.Panicf("%s must be a relative path when making dirs", path)
        }
        // 将路径与运行目录拼接成绝对路径
        absPath := filepath.Join(h.Runner.Dir, path)
        // 递归创建目录，权限为 0o777
        err := os.MkdirAll(absPath, 0o777)
        // 如果创建目录出错，则抛出错误
        if err != nil {
            log.Panicf("recursively making dirs under %s: %s", absPath, err)
        }
    }
}

// 执行 shell 命令的方法，返回运行结果
func (h *Harness) Sh(expr string) *RunResult {
    // 调用 Runner 的 Run 方法执行 bash 命令
    return h.Runner.Run(RunRequest{
        Path: "bash",
        Args: []string{"-c", expr},
    })
}

// 清理方法，停止节点守护进程并删除临时目录
func (h *Harness) Cleanup() {
    // 输出日志，表示正在清理集群
    log.Debugf("cleaning up cluster")
    // 停止所有节点的守护进程
    h.Nodes.StopDaemons()
    // 输出日志，表示正在删除临时目录
    log.Debugf("removing harness dir")
    // 递归删除临时目录
    err := os.RemoveAll(h.Dir)
    // 如果删除出错，则抛出错误
    if err != nil {
        log.Panicf("removing temp dir %s: %s", h.Dir, err)
    }
}

// 从给定的 multiaddr 中提取 peer ID，如果不包含 peer ID 则抛出错误
func (h *Harness) ExtractPeerID(m multiaddr.Multiaddr) peer.ID {
    var peerIDStr string
    // 遍历 multiaddr 的组件
    multiaddr.ForEach(m, func(c multiaddr.Component) bool {
        // 如果组件的协议码是 P2P，则提取 peer ID
        if c.Protocol().Code == multiaddr.P_P2P {
            peerIDStr = c.Value()
        }
        return true
    })
    // 如果未提取到 peer ID，则抛出错误
    if peerIDStr == "" {
        panic(multiaddr.ErrProtocolNotFound)
    }
    // 解码 peer ID 字符串为 peer.ID 对象
    peerID, err := peer.Decode(peerIDStr)
    // 如果解码出错，则抛出错误
    if err != nil {
        panic(err)
    }
    // 返回提取到的 peer ID
    return peerID
}
```