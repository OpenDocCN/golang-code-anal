# `kubo\plugin\plugins\peerlog\peerlog.go`

```
package peerlog

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "sync/atomic"  // 导入 sync/atomic 包，用于原子操作
    "time"  // 导入 time 包，用于时间相关操作

    logging "github.com/ipfs/go-log"  // 导入 logging 包，用于日志记录
    core "github.com/ipfs/kubo/core"  // 导入 core 包，用于核心功能
    plugin "github.com/ipfs/kubo/plugin"  // 导入 plugin 包，用于插件管理
    event "github.com/libp2p/go-libp2p/core/event"  // 导入 event 包，用于事件处理
    network "github.com/libp2p/go-libp2p/core/network"  // 导入 network 包，用于网络相关操作
    "github.com/libp2p/go-libp2p/core/peer"  // 导入 peer 包，用于对等节点操作
    "github.com/libp2p/go-libp2p/core/peerstore"  // 导入 peerstore 包，用于对等节点存储
    "go.uber.org/zap"  // 导入 zap 包，用于日志记录
)

var log = logging.Logger("plugin/peerlog")  // 定义全局变量 log，用于记录日志

type eventType int  // 定义 eventType 类型为 int

var (
    // size of the event queue buffer.
    eventQueueSize = 64 * 1024  // 定义事件队列缓冲区大小为 64 * 1024
    // number of events to drop when busy.
    busyDropAmount = eventQueueSize / 8  // 定义繁忙时要丢弃的事件数量为事件队列缓冲区大小的八分之一
)

const (
    eventConnect eventType = iota  // 定义 eventConnect 常量为 iota（0）
    eventIdentify  // 定义 eventIdentify 常量为 iota（1）
)

type plEvent struct {
    kind eventType  // 定义 plEvent 结构体，包含 kind 和 peer 两个字段
    peer peer.ID
}

// Log all the PeerIDs. This is considered internal, unsupported, and may break at any point.
//
// Usage:
//
//    GOLOG_FILE=~/peer.log IPFS_LOGGING_FMT=json ipfs daemon
//
// Output:
//
//    {"level":"info","ts":"2020-02-10T13:54:26.639Z","logger":"plugin/peerlog","caller":"peerlog/peerlog.go:51","msg":"connected","peer":"QmS2H72gdrekXJggGdE9SunXPntBqdkJdkXQJjuxcH8Cbt"}
//    {"level":"info","ts":"2020-02-10T13:54:59.095Z","logger":"plugin/peerlog","caller":"peerlog/peerlog.go:56","msg":"identified","peer":"QmS2H72gdrekXJggGdE9SunXPntBqdkJdkXQJjuxcH8Cbt","agent":"go-ipfs/0.5.0/"}
type peerLogPlugin struct {
    enabled      bool  // 定义 peerLogPlugin 结构体，包含 enabled、droppedCount 和 events 三个字段
    droppedCount uint64
    events       chan plEvent
}

var _ plugin.PluginDaemonInternal = (*peerLogPlugin)(nil)  // 定义 plugin.PluginDaemonInternal 接口的实现

// Plugins is exported list of plugins that will be loaded.
var Plugins = []plugin.Plugin{  // 定义 Plugins 列表，包含 peerLogPlugin 结构体
    &peerLogPlugin{},
}

// Name returns the plugin's name, satisfying the plugin.Plugin interface.
func (*peerLogPlugin) Name() string {  // 定义 peerLogPlugin 结构体的 Name 方法，返回插件的名称
    return "peerlog"
}

// Version returns the plugin's version, satisfying the plugin.Plugin interface.
func (*peerLogPlugin) Version() string {  // 定义 peerLogPlugin 结构体的 Version 方法，返回插件的版本
    return "0.1.0"
}

// extractEnabled extracts the "Enabled" field from the plugin config.
// 不要将此作为先例，这仅适用于此插件，因为它是内部功能，不受支持。
// 对于受支持的功能，我们应该重新设计插件 API 以支持包含默认禁用的插件的用例。
// 提取已启用的插件
func extractEnabled(config interface{}) bool {
    // 默认情况下插件是禁用的，除非 Enabled=true
    if config == nil {
        return false
    }
    mapIface, ok := config.(map[string]interface{})
    if !ok {
        return false
    }
    enabledIface, ok := mapIface["Enabled"]
    if !ok || enabledIface == nil {
        return false
    }
    enabled, ok := enabledIface.(bool)
    if !ok {
        return false
    }
    return enabled
}

// 初始化插件
func (pl *peerLogPlugin) Init(env *plugin.Environment) error {
    pl.events = make(chan plEvent, eventQueueSize)
    pl.enabled = extractEnabled(env.Config)
    return nil
}

// 收集事件
func (pl *peerLogPlugin) collectEvents(node *core.IpfsNode) {
    ctx := node.Context()

    busyCounter := 0
    dlog := log.Desugar()
    for {
        // 处理丢失的事件
        dropped := atomic.SwapUint64(&pl.droppedCount, 0)  // 交换并获取丢失事件的计数值
        if dropped > 0 {  // 如果有丢失的事件
            busyCounter++  // 忙碌计数器加一

            // 等待一段时间，让系统有机会赶上日志记录
            select {
            case <-time.After(time.Duration(busyCounter) * time.Second):
            case <-ctx.Done():
                return
            }

            // 从事件队列中取出1/8的事件，以避免立即再次出现丢失事件的情况
        loop:
            for i := 0; i < busyDropAmount; i++ {
                select {
                case <-pl.events:
                    dropped++
                default:
                    break loop
                }
            }

            // 加上在此期间丢失的事件
            dropped += atomic.SwapUint64(&pl.droppedCount, 0)

            // 报告已丢失的事件
            dlog.Error("dropped events", zap.Uint64("count", dropped))
        } else {
            busyCounter = 0  // 重置忙碌计数器
        }

        var e plEvent
        select {
        case <-ctx.Done():
            return
        case e = <-pl.events:
        }

        peerID := zap.String("peer", e.peer.String())

        switch e.kind {
        case eventConnect:
            dlog.Info("connected", peerID)
        case eventIdentify:
            agent, err := node.Peerstore.Get(e.peer, "AgentVersion")
            switch err {
            case nil:
            case peerstore.ErrNotFound:
                continue
            default:
                dlog.Error("failed to get agent version", zap.Error(err))
                continue
            }

            agentS, ok := agent.(string)
            if !ok {
                continue
            }
            dlog.Info("identified", peerID, zap.String("agent", agentS))
        }
    }
# 定义一个名为 emit 的方法，用于向事件通道发送事件
func (pl *peerLogPlugin) emit(evt eventType, p peer.ID) {
    # 通过 select 语句向事件通道发送事件，如果通道已满则增加 droppedCount 计数
    select {
    case pl.events <- plEvent{kind: evt, peer: p}:
    default:
        atomic.AddUint64(&pl.droppedCount, 1)
    }
}

# 定义一个名为 Start 的方法，用于启动插件
func (pl *peerLogPlugin) Start(node *core.IpfsNode) error {
    # 如果插件未启用，则直接返回
    if !pl.enabled {
        return nil
    }

    # 确保来自该插件的日志会被打印，而不受全局 IPFS_LOGGING 值的影响
    if err := logging.SetLogLevel("plugin/peerlog", "info"); err != nil {
        return fmt.Errorf("failed to set log level: %w", err)
    }

    # 订阅 PeerIdentificationCompleted 事件
    sub, err := node.PeerHost.EventBus().Subscribe(new(event.EvtPeerIdentificationCompleted))
    if err != nil {
        return fmt.Errorf("failed to subscribe to identify notifications")
    }

    # 定义一个 NotifyBundle，处理连接事件
    var notifee network.NotifyBundle
    notifee.ConnectedF = func(net network.Network, conn network.Conn) {
        pl.emit(eventConnect, conn.RemotePeer())
    }
    node.PeerHost.Network().Notify(&notifee)

    # 启动一个 goroutine 处理订阅的事件
    go func() {
        defer sub.Close()
        for e := range sub.Out() {
            switch e := e.(type) {
            case event.EvtPeerIdentificationCompleted:
                pl.emit(eventIdentify, e.Peer)
            }
        }
    }()

    # 启动一个 goroutine 收集事件
    go pl.collectEvents(node)

    return nil
}

# 定义一个名为 Close 的方法，用于关闭插件
func (*peerLogPlugin) Close() error {
    return nil
}
```