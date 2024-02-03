# `kubo\test\cli\harness\node.go`

```go
package harness

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "encoding/json"  // 导入 encoding/json 包，用于 JSON 编解码
    "errors"  // 导入 errors 包，用于错误处理
    "fmt"  // 导入 fmt 包，用于格式化输入输出
    "io"  // 导入 io 包，用于 I/O 操作
    "io/fs"  // 导入 io/fs 包，用于文件系统相关操作
    "net/http"  // 导入 net/http 包，用于 HTTP 相关操作
    "os"  // 导入 os 包，用于操作系统功能
    "os/exec"  // 导入 os/exec 包，用于执行外部命令
    "path/filepath"  // 导入 path/filepath 包，用于处理文件路径
    "runtime"  // 导入 runtime 包，用于运行时信息
    "strconv"  // 导入 strconv 包，用于字符串转换
    "strings"  // 导入 strings 包，用于字符串操作
    "syscall"  // 导入 syscall 包，用于系统调用
    "time"  // 导入 time 包，用于时间操作

    logging "github.com/ipfs/go-log/v2"  // 导入 logging 包，用于日志记录
    "github.com/ipfs/kubo/config"  // 导入 config 包
    serial "github.com/ipfs/kubo/config/serialize"  // 导入 serial 包
    "github.com/libp2p/go-libp2p/core/peer"  // 导入 peer 包
    rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager"  // 导入 rcmgr 包
    "github.com/multiformats/go-multiaddr"  // 导入 multiaddr 包
    manet "github.com/multiformats/go-multiaddr/net"  // 导入 manet 包
)

var log = logging.Logger("testharness")  // 定义全局日志记录器

// Node is a single Kubo node.
// Each node has its own config and can run its own Kubo daemon.
type Node struct {
    ID  int  // 节点 ID
    Dir string  // 节点目录

    APIListenAddr     multiaddr.Multiaddr  // API 监听地址
    GatewayListenAddr multiaddr.Multiaddr  // 网关监听地址
    SwarmAddr         multiaddr.Multiaddr  // Swarm 地址
    EnableMDNS        bool  // 是否启用 MDNS

    IPFSBin string  // IPFS 可执行文件路径
    Runner  *Runner  // 运行器

    Daemon *RunResult  // 运行结果
}

func BuildNode(ipfsBin, baseDir string, id int) *Node {
    dir := filepath.Join(baseDir, strconv.Itoa(id))  // 构建节点目录
    if err := os.MkdirAll(dir, 0o755); err != nil {  // 创建节点目录
        panic(err)  // 发生错误时抛出异常
    }

    env := environToMap(os.Environ())  // 获取环境变量并转换为 map
    env["IPFS_PATH"] = dir  // 设置 IPFS_PATH 环境变量为节点目录

    return &Node{  // 返回节点对象
        ID:      id,  // 设置节点 ID
        Dir:     dir,  // 设置节点目录
        IPFSBin: ipfsBin,  // 设置 IPFS 可执行文件路径
        Runner: &Runner{  // 创建运行器对象
            Env: env,  // 设置环境变量
            Dir: dir,  // 设置目录
        },
    }
}

func (n *Node) WriteBytes(filename string, b []byte) {
    f, err := os.Create(filepath.Join(n.Dir, filename))  // 创建文件
    if err != nil {  // 如果创建文件发生错误
        panic(err)  // 抛出异常
    }
    defer f.Close()  // 延迟关闭文件
    _, err = io.Copy(f, bytes.NewReader(b))  // 将字节写入文件
    if err != nil {  // 如果写入文件发生错误
        panic(err)  // 抛出异常
    }
}

// ReadFile reads the specific file. If it is relative, it is relative the node's root dir.
func (n *Node) ReadFile(filename string) string {
    f := filename  // 设置文件路径
    if !filepath.IsAbs(filename) {  // 如果文件路径不是绝对路径
        f = filepath.Join(n.Dir, filename)  // 将文件路径设置为相对于节点根目录的路径
    }
    b, err := os.ReadFile(f)  // 读取文件内容
    if err != nil {  // 如果读取文件发生错误
        panic(err)  // 抛出异常
    }
    return string(b)  // 返回文件内容
}
# 返回配置文件的完整路径
func (n *Node) ConfigFile() string {
    return filepath.Join(n.Dir, "config")
}

# 读取配置文件并返回配置对象指针
func (n *Node) ReadConfig() *config.Config {
    cfg, err := serial.Load(filepath.Join(n.Dir, "config"))
    if err != nil {
        panic(err)
    }
    return cfg
}

# 将配置对象写入配置文件
func (n *Node) WriteConfig(c *config.Config) {
    err := serial.WriteConfigFile(filepath.Join(n.Dir, "config"), c)
    if err != nil {
        panic(err)
    }
}

# 更新配置文件，通过传入的函数对配置对象进行修改
func (n *Node) UpdateConfig(f func(cfg *config.Config)) {
    cfg := n.ReadConfig()
    f(cfg)
    n.WriteConfig(cfg)
}

# 读取用户资源覆盖配置文件并返回部分限制配置对象指针
func (n *Node) ReadUserResourceOverrides() *rcmgr.PartialLimitConfig {
    var r rcmgr.PartialLimitConfig
    err := serial.ReadConfigFile(filepath.Join(n.Dir, "libp2p-resource-limit-overrides.json"), &r)
    switch err {
    case nil, serial.ErrNotInitialized:
        return &r
    default:
        panic(err)
    }
}

# 将用户提供的资源覆盖配置对象写入配置文件
func (n *Node) WriteUserSuppliedResourceOverrides(c *rcmgr.PartialLimitConfig) {
    err := serial.WriteConfigFile(filepath.Join(n.Dir, "libp2p-resource-limit-overrides.json"), c)
    if err != nil {
        panic(err)
    }
}

# 更新用户提供的资源管理器覆盖配置文件，通过传入的函数对覆盖对象进行修改
func (n *Node) UpdateUserSuppliedResourceManagerOverrides(f func(overrides *rcmgr.PartialLimitConfig)) {
    overrides := n.ReadUserResourceOverrides()
    f(overrides)
    n.WriteUserSuppliedResourceOverrides(overrides)
}

# 运行 IPFS 命令，并返回运行结果对象指针
func (n *Node) IPFS(args ...string) *RunResult {
    res := n.RunIPFS(args...)
    n.Runner.AssertNoError(res)
    return res
}

# 将字符串传输到 IPFS，并返回运行结果对象指针
func (n *Node) PipeStrToIPFS(s string, args ...string) *RunResult {
    return n.PipeToIPFS(strings.NewReader(s), args...)
}

# 将数据流传输到 IPFS，并返回运行结果对象指针
func (n *Node) PipeToIPFS(reader io.Reader, args ...string) *RunResult {
    res := n.RunPipeToIPFS(reader, args...)
    n.Runner.AssertNoError(res)
    return res
}

# 运行数据流传输到 IPFS 的命令，并返回运行结果对象指针
func (n *Node) RunPipeToIPFS(reader io.Reader, args ...string) *RunResult {
    return n.Runner.Run(RunRequest{
        Path:    n.IPFSBin,
        Args:    args,
        CmdOpts: []CmdOpt{RunWithStdin(reader)},
    })
}
// RunIPFS 方法用于运行 IPFS 节点，接受可变数量的参数，并返回运行结果
func (n *Node) RunIPFS(args ...string) *RunResult {
    return n.Runner.Run(RunRequest{
        Path: n.IPFSBin,  // 设置运行 IPFS 节点的路径
        Args: args,  // 设置运行 IPFS 节点的参数
    })
}

// Init 方法用于初始化和配置 IPFS 节点，初始化完成后准备就绪
func (n *Node) Init(ipfsArgs ...string) *Node {
    n.Runner.MustRun(RunRequest{
        Path: n.IPFSBin,  // 设置运行 IPFS 节点的路径
        Args: append([]string{"init"}, ipfsArgs...),  // 设置运行 IPFS 节点的参数
    })

    if n.SwarmAddr == nil {
        swarmAddr, err := multiaddr.NewMultiaddr("/ip4/127.0.0.1/tcp/0")  // 创建 Swarm 地址
        if err != nil {
            panic(err)
        }
        n.SwarmAddr = swarmAddr  // 设置 Swarm 地址
    }

    if n.APIListenAddr == nil {
        apiAddr, err := multiaddr.NewMultiaddr("/ip4/127.0.0.1/tcp/0")  // 创建 API 监听地址
        if err != nil {
            panic(err)
        }
        n.APIListenAddr = apiAddr  // 设置 API 监听地址
    }

    if n.GatewayListenAddr == nil {
        gatewayAddr, err := multiaddr.NewMultiaddr("/ip4/127.0.0.1/tcp/0")  // 创建 Gateway 监听地址
        if err != nil {
            panic(err)
        }
        n.GatewayListenAddr = gatewayAddr  // 设置 Gateway 监听地址
    }

    n.UpdateConfig(func(cfg *config.Config) {
        cfg.Bootstrap = []string{}  // 设置启动节点
        cfg.Addresses.Swarm = []string{n.SwarmAddr.String()}  // 设置 Swarm 地址
        cfg.Addresses.API = []string{n.APIListenAddr.String()}  // 设置 API 监听地址
        cfg.Addresses.Gateway = []string{n.GatewayListenAddr.String()}  // 设置 Gateway 监听地址
        cfg.Swarm.DisableNatPortMap = true  // 禁用 NAT 端口映射
        cfg.Discovery.MDNS.Enabled = n.EnableMDNS  // 设置是否启用 MDNS
    })
    return n
}

// StartDaemonWithReq 方法用于使用给定的请求运行 Kubo 守护进程
// 这会覆盖请求的路径为 Kubo bin 路径
func (n *Node) StartDaemonWithReq(req RunRequest, authorization string) *Node {
    alive := n.IsAlive()  // 检查节点是否已经运行
    if alive {
        log.Panicf("node %d is already running", n.ID)  // 如果节点已经运行，则抛出错误
    }  # 结束 if 语句块

    # 复制 req 对象到 newReq
    newReq := req
    # 设置 newReq 的 Path 属性为 n.IPFSBin
    newReq.Path = n.IPFSBin
    # 将 req.Args 添加到 newReq 的 Args 属性中
    newReq.Args = append([]string{"daemon"}, req.Args...)
    # 设置 newReq 的 RunFunc 属性为 (*exec.Cmd).Start
    newReq.RunFunc = (*exec.Cmd).Start

    # 记录日志，表示正在启动节点 n.ID
    log.Debugf("starting node %d", n.ID)
    # 使用 newReq 执行命令，并将结果赋值给 res
    res := n.Runner.MustRun(newReq)

    # 将 res 赋值给节点的 Daemon 属性
    n.Daemon = res

    # 记录日志，表示节点 n.ID 已启动，正在检查 API
    log.Debugf("node %d started, checking API", n.ID)
    # 等待节点的 API 准备就绪
    n.WaitOnAPI(authorization)
    # 返回节点对象
    return n
# 停止节点的守护进程
func (n *Node) StopDaemon() *Node:
    # 打印调试信息，表示正在停止节点
    log.Debugf("stopping node %d", n.ID)
    # 如果节点的守护进程为空，则打印调试信息并返回节点
    if n.Daemon == nil:
        log.Debugf("didn't stop node %d since no daemon present", n.ID)
        return n
    # 创建一个用于接收信号的通道
    watch := make(chan struct{}, 1)
    # 启动一个 goroutine，等待守护进程的进程结束信号
    go func():
        _, _ = n.Daemon.Cmd.Process.Wait()
        watch <- struct{}{}
    }()
    # 如果运行时环境是 Windows，则执行特定的停止守护进程的逻辑
    if runtime.GOOS == "windows":
        # 如果发送 SIGKILL 信号成功，则返回节点
        if n.signalAndWait(watch, syscall.SIGKILL, 5*time.Second):
            return n
        # 如果超时，则打印错误信息并终止程序
        log.Panicf("timed out stopping node %d with peer ID %s", n.ID, n.PeerID())
    # 否则，发送 SIGTERM 信号并等待
    log.Debugf("signaling node %d with SIGTERM", n.ID)
    if n.signalAndWait(watch, syscall.SIGTERM, 1*time.Second):
        return n
    log.Debugf("signaling node %d with SIGTERM", n.ID)
    if n.signalAndWait(watch, syscall.SIGTERM, 2*time.Second):
        return n
    log.Debugf("signaling node %d with SIGQUIT", n.ID)
    if n.signalAndWait(watch, syscall.SIGQUIT, 5*time.Second):
        return n
    # 使用 Debugf 方法记录信号节点的 ID 和信号类型
    log.Debugf("signaling node %d with SIGKILL", n.ID)
    # 如果信号发送成功并等待了5秒钟，则返回节点
    if n.signalAndWait(watch, syscall.SIGKILL, 5*time.Second):
        return n
    # 如果停止节点超时，则使用 Panicf 方法记录超时信息和节点的 ID 以及对等节点的 ID
    log.Panicf("timed out stopping node %d with peer ID %s", n.ID, n.PeerID())
    # 返回节点
    return n
# 返回节点的 API 地址
func (n *Node) APIAddr() multiaddr.Multiaddr {
    # 尝试获取节点的 API 地址
    ma, err := n.TryAPIAddr()
    if err != nil {
        panic(err)
    }
    return ma
}

# 返回节点的 API URL
func (n *Node) APIURL() string {
    # 获取节点的 API 地址
    apiAddr := n.APIAddr()
    # 将 API 地址转换为网络地址
    netAddr, err := manet.ToNetAddr(apiAddr)
    if err != nil {
        panic(err)
    }
    return "http://" + netAddr.String()
}

# 尝试获取节点的 API 地址
func (n *Node) TryAPIAddr() (multiaddr.Multiaddr, error) {
    # 读取节点的 API 地址
    b, err := os.ReadFile(filepath.Join(n.Dir, "api"))
    if err != nil {
        return nil, err
    }
    # 将读取的 API 地址转换为多地址
    ma, err := multiaddr.NewMultiaddr(string(b))
    if err != nil {
        return nil, err
    }
    return ma, nil
}

# 检查节点的 API 地址是否可用
func (n *Node) checkAPI(authorization string) bool {
    # 获取节点的 API 地址
    apiAddr, err := n.TryAPIAddr()
    if err != nil {
        log.Debugf("node %d API addr not available yet: %s", n.ID, err.Error())
        return false
    }
    # 从 API 地址中获取 IP 地址
    ip, err := apiAddr.ValueForProtocol(multiaddr.P_IP4)
    if err != nil {
        panic(err)
    }
    # 从 API 地址中获取端口号
    port, err := apiAddr.ValueForProtocol(multiaddr.P_TCP)
    if err != nil {
        panic(err)
    }
    # 构建 API 的 URL
    url := fmt.Sprintf("http://%s:%s/api/v0/id", ip, port)
    log.Debugf("checking API for node %d at %s", n.ID, url)

    # 创建 HTTP 请求
    req, err := http.NewRequest(http.MethodPost, url, nil)
    if err != nil {
        panic(err)
    }
    # 设置授权信息
    if authorization != "" {
        req.Header.Set("Authorization", authorization)
    }

    # 发起 HTTP 请求
    httpResp, err := http.DefaultClient.Do(req)
    if err != nil {
        log.Debugf("node %d API check error: %s", err.Error())
        return false
    }
    defer httpResp.Body.Close()
    resp := struct {
        ID string
    }{}

    # 读取 HTTP 响应内容
    respBytes, err := io.ReadAll(httpResp.Body)
    if err != nil {
        log.Debugf("error reading API check response for node %d: %s", n.ID, err.Error())
        return false
    }
    log.Debugf("got API check response for node %d: %s", n.ID, string(respBytes))

    # 解析 JSON 响应
    err = json.Unmarshal(respBytes, &resp)
    # 如果发生错误，则记录错误信息并返回 false
    if err != nil:
        log.Debugf("error decoding API check response for node %d: %s", n.ID, err.Error())
        return false
    # 如果响应中没有包含对等节点 ID，则记录信息并返回 false
    if resp.ID == "":
        log.Debugf("API check response for node %d did not contain a Peer ID", n.ID)
        return false
    # 解码响应中的对等节点 ID
    respPeerID, err := peer.Decode(resp.ID)
    # 如果解码出错，则触发 panic
    if err != nil:
        panic(err)
    # 获取节点的对等节点 ID
    peerID := n.PeerID()
    # 如果响应中的对等节点 ID与节点的对等节点 ID不一致，则触发 panic
    if respPeerID != peerID:
        log.Panicf("expected peer ID %s but got %s", peerID, resp.ID)
    # 记录节点的 API 检查成功信息并返回 true
    log.Debugf("API check for node %d successful", n.ID)
    return true
# 获取节点的对等节点ID
func (n *Node) PeerID() peer.ID {
    # 读取节点的配置信息
    cfg := n.ReadConfig()
    # 解码对等节点ID
    id, err := peer.Decode(cfg.Identity.PeerID)
    if err != nil {
        panic(err)
    }
    return id
}

# 等待节点的API就绪
func (n *Node) WaitOnAPI(authorization string) *Node {
    log.Debugf("waiting on API for node %d", n.ID)
    # 循环检查API是否就绪
    for i := 0; i < 50; i++ {
        if n.checkAPI(authorization) {
            log.Debugf("daemon API found, daemon stdout: %s", n.Daemon.Stdout.String())
            return n
        }
        time.Sleep(400 * time.Millisecond)
    }
    # 如果API未就绪，则输出错误信息并终止程序
    log.Panicf("node %d with peer ID %s failed to come online: \n%s\n\n%s", n.ID, n.PeerID(), n.Daemon.Stderr.String(), n.Daemon.Stdout.String())
    return n
}

# 检查节点是否存活
func (n *Node) IsAlive() bool {
    # 检查节点的守护进程是否存在
    if n.Daemon == nil || n.Daemon.Cmd == nil || n.Daemon.Cmd.Process == nil {
        return false
    }
    log.Debugf("signaling node %d daemon process for liveness check", n.ID)
    # 向节点的守护进程发送信号检查存活状态
    err := n.Daemon.Cmd.Process.Signal(syscall.Signal(0))
    if err == nil {
        log.Debugf("node %d daemon is alive", n.ID)
        return true
    }
    log.Debugf("node %d daemon not alive: %s", err.Error())
    return false
}

# 获取节点的Swarm地址
func (n *Node) SwarmAddrs() []multiaddr.Multiaddr {
    # 运行IPFS命令获取Swarm地址
    res := n.Runner.MustRun(RunRequest{
        Path: n.IPFSBin,
        Args: []string{"swarm", "addrs", "local"},
    })
    out := strings.TrimSpace(res.Stdout.String())
    outLines := strings.Split(out, "\n")
    var addrs []multiaddr.Multiaddr
    # 解析Swarm地址并添加到列表中
    for _, addrStr := range outLines {
        ma, err := multiaddr.NewMultiaddr(addrStr)
        if err != nil {
            panic(err)
        }
        addrs = append(addrs, ma)
    }
    return addrs
}

# 获取带有对等节点ID的Swarm地址
func (n *Node) SwarmAddrsWithPeerIDs() []multiaddr.Multiaddr {
    # 获取IPFS协议名称
    ipfsProtocol := multiaddr.ProtocolWithCode(multiaddr.P_IPFS).Name
    # 获取节点的对等节点ID
    peerID := n.PeerID()
    var addrs []multiaddr.Multiaddr
    // 遍历节点的Swarm地址列表
    for _, ma := range n.SwarmAddrs() {
        // 如果多地址中没有包含peer ID，则将peer ID添加到多地址中
        _, err := ma.ValueForProtocol(multiaddr.P_IPFS)
        // 如果出现协议未找到的错误
        if errors.Is(err, multiaddr.ErrProtocolNotFound) {
            // 创建一个新的组件，包含IPFS协议和peer ID
            comp, err := multiaddr.NewComponent(ipfsProtocol, peerID.String())
            // 如果创建组件时出现错误，则触发panic
            if err != nil {
                panic(err)
            }
            // 将新组件封装到多地址中
            ma = ma.Encapsulate(comp)
        }
        // 将处理后的多地址添加到地址列表中
        addrs = append(addrs, ma)
    }
    // 返回处理后的地址列表
    return addrs
// 返回不带对等节点ID的Swarm地址列表
func (n *Node) SwarmAddrsWithoutPeerIDs() []multiaddr.Multiaddr {
    var addrs []multiaddr.Multiaddr
    for _, ma := range n.SwarmAddrs() {
        var components []multiaddr.Multiaddr
        multiaddr.ForEach(ma, func(c multiaddr.Component) bool {
            if c.Protocol().Code == multiaddr.P_IPFS {
                return true
            }
            components = append(components, &c)
            return true
        })
        ma = multiaddr.Join(components...)
        addrs = append(addrs, ma)
    }
    return addrs
}

// 连接到另一个节点
func (n *Node) Connect(other *Node) *Node {
    n.Runner.MustRun(RunRequest{
        Path: n.IPFSBin,
        Args: []string{"swarm", "connect", other.SwarmAddrsWithPeerIDs()[0].String()},
    })
    return n
}

// 返回节点的对等节点列表
func (n *Node) Peers() []multiaddr.Multiaddr {
    res := n.Runner.MustRun(RunRequest{
        Path: n.IPFSBin,
        Args: []string{"swarm", "peers"},
    })
    var addrs []multiaddr.Multiaddr
    for _, line := range res.Stdout.Lines() {
        ma, err := multiaddr.NewMultiaddr(line)
        if err != nil {
            panic(err)
        }
        addrs = append(addrs, ma)
    }
    return addrs
}

// 与另一个节点建立对等关系
func (n *Node) PeerWith(other *Node) {
    n.UpdateConfig(func(cfg *config.Config) {
        var addrs []multiaddr.Multiaddr
        for _, addrStr := range other.ReadConfig().Addresses.Swarm {
            ma, err := multiaddr.NewMultiaddr(addrStr)
            if err != nil {
                panic(err)
            }
            addrs = append(addrs, ma)
        }

        cfg.Peering.Peers = append(cfg.Peering.Peers, peer.AddrInfo{
            ID:    other.PeerID(),
            Addrs: addrs,
        })
    })
}

// 与另一个节点断开连接
func (n *Node) Disconnect(other *Node) {
    n.IPFS("swarm", "disconnect", "/p2p/"+other.PeerID().String())
}

// 等待网关文件，然后返回其内容或超时
func (n *Node) GatewayURL() string {
    timer := time.NewTimer(1 * time.Second)
    defer timer.Stop()
    # 无限循环，等待定时器通道的信号
    for {
        # 选择执行以下操作：等待定时器通道的信号
        select {
        # 如果收到定时器通道的信号
        case <-timer.C:
            # 抛出超时错误
            panic("timeout waiting for gateway file")
        # 如果没有收到定时器通道的信号
        default:
            # 读取文件内容到字节流 b，同时捕获可能的错误
            b, err := os.ReadFile(filepath.Join(n.Dir, "gateway"))
            # 如果没有错误
            if err == nil {
                # 返回去除空白字符后的文件内容
                return strings.TrimSpace(string(b))
            }
            # 如果出现了错误
            # 如果错误不是文件不存在的错误
            if !errors.Is(err, fs.ErrNotExist) {
                # 抛出错误
                panic(err)
            }
            # 等待 1 毫秒
            time.Sleep(1 * time.Millisecond)
        }
    }
# 返回一个基于节点的网关客户端，使用默认的 HTTP 客户端和节点的网关 URL
func (n *Node) GatewayClient() *HTTPClient {
    return &HTTPClient{
        Client:  http.DefaultClient,
        BaseURL: n.GatewayURL(),
    }
}

# 返回一个基于节点的 API 客户端，使用默认的 HTTP 客户端和节点的 API URL
func (n *Node) APIClient() *HTTPClient {
    return &HTTPClient{
        Client:  http.DefaultClient,
        BaseURL: n.APIURL(),
    }
}
```