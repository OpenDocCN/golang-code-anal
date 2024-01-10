# `kubo\cmd\ipfs\kubo\pinmfs.go`

```
package kubo

import (
    "context"  // 导入上下文包，用于处理请求的取消、超时等操作
    "fmt"  // 导入格式化包，用于格式化输出
    "os"  // 导入操作系统包，用于访问操作系统功能
    "time"  // 导入时间包，用于处理时间相关操作

    "github.com/libp2p/go-libp2p/core/host"  // 导入 libp2p 核心主机包
    peer "github.com/libp2p/go-libp2p/core/peer"  // 导入 libp2p 核心对等体包

    pinclient "github.com/ipfs/boxo/pinning/remote/client"  // 导入远程客户端包
    cid "github.com/ipfs/go-cid"  // 导入 CID 包
    ipld "github.com/ipfs/go-ipld-format"  // 导入 IPLD 格式包
    logging "github.com/ipfs/go-log"  // 导入日志包

    config "github.com/ipfs/kubo/config"  // 导入配置包
    "github.com/ipfs/kubo/core"  // 导入核心包
)

// mfslog is the logger for remote mfs pinning.
var mfslog = logging.Logger("remotepinning/mfs")  // 定义远程 mfs 固定的日志记录器

type lastPin struct {
    Time          time.Time  // 最后固定的时间
    ServiceName   string  // 服务名称
    ServiceConfig config.RemotePinningService  // 远程固定服务的配置
    CID           cid.Cid  // CID
}

func (x lastPin) IsValid() bool {
    return x != lastPin{}  // 判断最后固定是否有效
}

var daemonConfigPollInterval = time.Minute / 2  // 定义守护进程配置轮询间隔

func init() {
    // this environment variable is solely for testing, use at your own risk
    if pollDurStr := os.Getenv("MFS_PIN_POLL_INTERVAL"); pollDurStr != "" {
        d, err := time.ParseDuration(pollDurStr)  // 解析环境变量 MFS_PIN_POLL_INTERVAL 的值为时间间隔
        if err != nil {
            mfslog.Error("error parsing MFS_PIN_POLL_INTERVAL, using default:", err)  // 如果解析出错，则记录错误
        }
        daemonConfigPollInterval = d  // 更新守护进程配置轮询间隔
    }
}

const defaultRepinInterval = 5 * time.Minute  // 定义默认重新固定间隔为 5 分钟

type pinMFSContext interface {
    Context() context.Context  // 定义获取上下文的接口
    GetConfig() (*config.Config, error)  // 定义获取配置的接口
}

type pinMFSNode interface {
    RootNode() (ipld.Node, error)  // 定义获取根节点的接口
    Identity() peer.ID  // 定义获取身份的接口
    PeerHost() host.Host  // 定义获取对等主机的接口
}

type ipfsPinMFSNode struct {
    node *core.IpfsNode  // 定义 IPFS 节点
}

func (x *ipfsPinMFSNode) RootNode() (ipld.Node, error) {
    return x.node.FilesRoot.GetDirectory().GetNode()  // 获取 IPFS 节点的根节点
}

func (x *ipfsPinMFSNode) Identity() peer.ID {
    return x.node.Identity  // 获取 IPFS 节点的身份
}

func (x *ipfsPinMFSNode) PeerHost() host.Host {
    return x.node.PeerHost  // 获取 IPFS 节点的对等主机
}

func startPinMFS(configPollInterval time.Duration, cctx pinMFSContext, node pinMFSNode) {
    errCh := make(chan error)  // 创建错误通道
    go pinMFSOnChange(configPollInterval, cctx, node, errCh)  // 异步启动 MFS 固定的变更监听
}
    # 创建一个匿名的 goroutine 函数
    go func() {
        # 无限循环
        for {
            # 选择不同的 case
            select {
                # 如果 errCh 通道有数据
                case err, isOpen := <-errCh:
                    # 如果通道已关闭，则返回
                    if !isOpen {
                        return
                    }
                    # 否则打印错误信息
                    mfslog.Errorf("%v", err)
                # 如果 cctx.Context().Done() 通道有数据
                case <-cctx.Context().Done():
                    # 返回
                    return
            }
        }
    }()
}

// 在配置轮询间隔内，监控 pinMFS 的变化
func pinMFSOnChange(configPollInterval time.Duration, cctx pinMFSContext, node pinMFSNode, errCh chan<- error) {
    // 延迟关闭错误通道
    defer close(errCh)

    var tmo *time.Timer
    // 延迟函数，用于停止定时器
    defer func() {
        if tmo != nil {
            tmo.Stop()
        }
    }()

    // 记录最后的 pin 信息
    lastPins := map[string]lastPin{}
    for {
        // 轮询休眠
        if tmo == nil {
            tmo = time.NewTimer(configPollInterval)
        } else {
            tmo.Reset(configPollInterval)
        }
        select {
        case <-cctx.Context().Done():
            return
        case <-tmo.C:
        }

        // 重新读取配置，可能已经发生变化
        cfg, err := cctx.GetConfig()
        if err != nil {
            select {
            case errCh <- fmt.Errorf("pinning reading config (%v)", err):
            case <-cctx.Context().Done():
                return
            }
            continue
        }
        mfslog.Debugf("pinning loop is awake, %d remote services", len(cfg.Pinning.RemoteServices))

        // 获取最新的 MFS 根 CID
        rootNode, err := node.RootNode()
        if err != nil {
            select {
            case errCh <- fmt.Errorf("pinning reading MFS root (%v)", err):
            case <-cctx.Context().Done():
                return
            }
            continue
        }
        rootCid := rootNode.Cid()

        // 并行在所有远程服务上进行 pin 操作
        pinAllMFS(cctx.Context(), node, cfg, rootCid, lastPins, errCh)
    }
}

// pinAllMFS 并行在所有远程服务上进行 pin 操作，以克服 DoS 攻击
func pinAllMFS(ctx context.Context, node pinMFSNode, cfg *config.Config, rootCid cid.Cid, lastPins map[string]lastPin, errCh chan<- error) {
    ch := make(chan lastPin, len(cfg.Pinning.RemoteServices))
    // 循环处理所有远程服务
    for i := 0; i < len(cfg.Pinning.RemoteServices); i++ {
        if x := <-ch; x.IsValid() {
            lastPins[x.ServiceName] = x
        }
    }
}

func pinMFS(
    ctx context.Context,
    node pinMFSNode,
    # 定义参数 cid，类型为 cid.Cid
    cid cid.Cid,
    # 定义参数 svcName，类型为 string，表示服务名称
    svcName string,
    # 定义参数 svcConfig，类型为 config.RemotePinningService，表示远程固定服务的配置
    svcConfig config.RemotePinningService,
// 创建一个新的 PIN 客户端，使用给定的 API 端点和密钥
c := pinclient.NewClient(svcConfig.API.Endpoint, svcConfig.API.Key)

// 获取 MFS 的 PIN 名称，如果未设置则使用节点身份的字符串作为默认名称
pinName := svcConfig.Policies.MFS.PinName
if pinName == "" {
    pinName = fmt.Sprintf("policy/%s/mfs", node.Identity().String())
}

// 检查 MFS PIN 是否存在（在所有可能的状态下），并检查其 CID
pinStatuses := []pinclient.Status{pinclient.StatusQueued, pinclient.StatusPinning, pinclient.StatusPinned, pinclient.StatusFailed}
lsPinCh, lsErrCh := c.Ls(ctx, pinclient.PinOpts.FilterName(pinName), pinclient.PinOpts.FilterStatus(pinStatuses...))
existingRequestID := "" // 是否存在任何具有 pinName 的预先存在的 MFS PIN（对于任何 CID）？
pinning := false        // 当前 MFS 的 CID 是否正在被 PIN？
pinTime := time.Now().UTC()
pinStatusMsg := "pinning to %q: received pre-existing %q status for %q (requestid=%q)"
for ps := range lsPinCh {
    existingRequestID = ps.GetRequestId()
    if ps.GetPin().GetCid() == cid && ps.GetStatus() == pinclient.StatusFailed {
        mfslog.Errorf(pinStatusMsg, svcName, pinclient.StatusFailed, cid, existingRequestID)
    } else {
        mfslog.Debugf(pinStatusMsg, svcName, ps.GetStatus(), ps.GetPin().GetCid(), existingRequestID)
    }
    if ps.GetPin().GetCid() == cid && ps.GetStatus() != pinclient.StatusFailed {
        pinning = true
        pinTime = ps.GetCreated().UTC()
        break
    }
}
for range lsPinCh { // 以防前一个循环提前退出
}
if err := <-lsErrCh; err != nil {
    return lastPin{}, fmt.Errorf("error while listing remote pins: %v", err)
}

// 当前 MFS 根的 CID 已经被 PIN，无需操作
if pinning {
    mfslog.Debugf("pinning MFS to %q: pin for %q exists since %s, skipping", svcName, cid, pinTime.String())
    return lastPin{Time: pinTime, ServiceName: svcName, ServiceConfig: svcConfig, CID: cid}, nil
}

// 准备 Pin.name
    # 创建一个选项数组，用于指定添加操作的参数，包括名称
    addOpts := []pinclient.AddOption{pinclient.PinOpts.WithName(pinName)}

    // 准备 Pin.origins
    // 将自己的多地址添加到'origins'数组中，以便固定服务可以将其用作提示，并尝试连接回我们（如果可能的话）
    if node.PeerHost() != nil:
        # 将节点的多地址转换为P2P地址
        addrs, err := peer.AddrInfoToP2pAddrs(host.InfoFromHost(node.PeerHost()))
        if err != nil:
            return lastPin{}, err
        # 将多地址添加到选项数组中
        addOpts = append(addOpts, pinclient.PinOpts.WithOrigins(addrs...))
    // 创建或替换MFS根目录的固定
    if existingRequestID != "":
        mfslog.Debugf("pinning to %q: replacing existing MFS root pin with %q", svcName, cid)
        # 替换现有的MFS根目录固定
        _, err := c.Replace(ctx, existingRequestID, cid, addOpts...)
        if err != nil:
            return lastPin{}, err
    else:
        mfslog.Debugf("pinning to %q: creating a new MFS root pin for %q", svcName, cid)
        # 创建新的MFS根目录固定
        _, err := c.Add(ctx, cid, addOpts...)
        if err != nil:
            return lastPin{}, err
    # 返回最后的固定信息
    return lastPin{Time: pinTime, ServiceName: svcName, ServiceConfig: svcConfig, CID: cid}, nil
# 闭合前面的函数定义
```