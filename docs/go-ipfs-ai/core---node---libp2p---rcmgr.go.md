# `kubo\core\node\libp2p\rcmgr.go`

```go
package libp2p

import (
    "context"  // 上下文包，用于控制函数调用的超时、取消和截止
    "encoding/json"  // 用于 JSON 数据的编码和解码
    "fmt"  // 用于格式化输出
    "os"  // 提供对操作系统功能的访问
    "path/filepath"  // 用于操作文件路径的函数

    "github.com/benbjohnson/clock"  // 时钟库
    logging "github.com/ipfs/go-log/v2"  // 日志库
    "github.com/libp2p/go-libp2p"  // libp2p 核心库
    "github.com/libp2p/go-libp2p/core/network"  // libp2p 网络核心库
    "github.com/libp2p/go-libp2p/core/peer"  // libp2p 对等核心库
    "github.com/libp2p/go-libp2p/core/protocol"  // libp2p 协议核心库
    rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager"  // libp2p 主机资源管理库
    "github.com/multiformats/go-multiaddr"  // 多地址库
    "go.uber.org/fx"  // 用于编写模块化应用程序的库

    "github.com/ipfs/kubo/config"  // kubo 配置库
    "github.com/ipfs/kubo/core/node/helpers"  // kubo 节点助手库
    "github.com/ipfs/kubo/repo"  // kubo 存储库
)

var rcmgrLogger = logging.Logger("rcmgr")  // 创建 rcmgrLogger 日志记录器

const NetLimitTraceFilename = "rcmgr.json.gz"  // 设置网络限制跟踪文件名

var ErrNoResourceMgr = fmt.Errorf("missing ResourceMgr: make sure the daemon is running with Swarm.ResourceMgr.Enabled")  // 创建错误变量

func ResourceManager(cfg config.SwarmConfig, userResourceOverrides rcmgr.PartialLimitConfig) interface{} {  // 创建 ResourceManager 函数
    return func(mctx helpers.MetricsCtx, lc fx.Lifecycle, repo repo.Repo) (network.ResourceManager, Libp2pOpts, error) {  // 返回一个函数
        var manager network.ResourceManager  // 声明 network.ResourceManager 变量
        var opts Libp2pOpts  // 声明 Libp2pOpts 变量

        enabled := cfg.ResourceMgr.Enabled.WithDefault(true)  // 从配置中获取 ResourceMgr 是否启用，默认为 true

        //  ENV overrides Config (if present)
        switch os.Getenv("LIBP2P_RCMGR") {  // 获取环境变量 LIBP2P_RCMGR 的值
        case "0", "false":  // 如果值为 "0" 或 "false"
            enabled = false  // 则禁用 ResourceMgr
        case "1", "true":  // 如果值为 "1" 或 "true"
            enabled = true  // 则启用 ResourceMgr
        }

        if enabled {  // 如果 ResourceMgr 启用
            log.Debug("libp2p resource manager is enabled")  // 输出日志信息

            repoPath, err := config.PathRoot()  // 获取 IPFS_PATH
            if err != nil {  // 如果出错
                return nil, opts, fmt.Errorf("opening IPFS_PATH: %w", err)  // 返回错误信息
            }

            limitConfig, msg, err := LimitConfig(cfg, userResourceOverrides)  // 获取资源限制配置
            if err != nil {  // 如果出错
                return nil, opts, fmt.Errorf("creating final Resource Manager config: %w", err)  // 返回错误信息
            }

            if !isPartialConfigEmpty(userResourceOverrides) {  // 如果用户资源配置不为空
                rcmgrLogger.Info(`  // 输出日志信息
// 检查部分配置是否为空
func isPartialConfigEmpty(cfg rcmgr.PartialLimitConfig) bool {
    // 创建一个空的资源限制配置
    var emptyResourceConfig rcmgr.ResourceLimits
    // 检查各个字段是否为空，如果有任何一个字段不为空，则返回 false
    if cfg.System != emptyResourceConfig ||
        cfg.Transient != emptyResourceConfig ||
        cfg.AllowlistedSystem != emptyResourceConfig ||
        cfg.AllowlistedTransient != emptyResourceConfig ||
        cfg.ServiceDefault != emptyResourceConfig ||
        cfg.ServicePeerDefault != emptyResourceConfig ||
        cfg.ProtocolDefault != emptyResourceConfig ||
        cfg.ProtocolPeerDefault != emptyResourceConfig ||
        cfg.PeerDefault != emptyResourceConfig ||
        cfg.Conn != emptyResourceConfig ||
        cfg.Stream != emptyResourceConfig {
        return false
    }
    // 遍历 Service 字段，如果有任何一个字段不为空，则返回 false
    for _, v := range cfg.Service {
        if v != emptyResourceConfig {
            return false
        }
    }
    // 遍历 ServicePeer 字段，如果有任何一个字段不为空，则返回 false
    for _, v := range cfg.ServicePeer {
        if v != emptyResourceConfig {
            return false
        }
    }
    // 遍历 Protocol 字段，如果有任何一个字段不为空，则返回 false
    for _, v := range cfg.Protocol {
        if v != emptyResourceConfig {
            return false
        }
    }
    // 遍历 ProtocolPeer 字段，如果有任何一个字段不为空，则返回 false
    for _, v := range cfg.ProtocolPeer {
        if v != emptyResourceConfig {
            return false
        }
    }
    // 遍历 Peer 字段，如果有任何一个字段不为空，则返回 false
    for _, v := range cfg.Peer {
        if v != emptyResourceConfig {
            return false
        }
    }
    // 如果所有字段都为空，则返回 true
    return true
}

// LimitConfig 返回计算出的默认限制和用户提供的覆盖限制的并集
func LimitConfig(cfg config.SwarmConfig, userResourceOverrides rcmgr.PartialLimitConfig) (limitConfig rcmgr.ConcreteLimitConfig, logMessageForStartup string, err error) {
    // 创建默认限制配置
    limitConfig, msg, err := createDefaultLimitConfig(cfg)
    // 如果创建默认限制配置时出现错误，则返回空的限制配置和错误信息
    if err != nil {
        return rcmgr.ConcreteLimitConfig{}, msg, err
    }

    // 默认值和使用指定的 userResourceOverrides 进行覆盖的逻辑在 docs/libp2p-resource-management.md 中有详细说明
    // 任何更改都应该在那里反映出来。

    // 这实际上用用户资源覆盖文件中的任何非“useDefault”值来覆盖计算出的默认LimitConfig。
    // 由于构建的工作方式，userResourceOverrides中的任何rcmgr.Default值都将被计算出的默认值覆盖。
    limitConfig = userResourceOverrides.Build(limitConfig)

    // 返回覆盖后的limitConfig和消息，没有错误
    return limitConfig, msg, nil
// ResourceLimitsAndUsage 结构体定义了资源限制和使用情况，包括内存、文件描述符、连接数、流量等
type ResourceLimitsAndUsage struct {
    // Memory 表示内存限制
    Memory               rcmgr.LimitVal64
    // MemoryUsage 表示内存使用情况
    MemoryUsage          int64
    // FD 表示文件描述符限制
    FD                   rcmgr.LimitVal
    // FDUsage 表示文件描述符使用情况
    FDUsage              int
    // Conns 表示连接数限制
    Conns                rcmgr.LimitVal
    // ConnsUsage 表示连接数使用情况
    ConnsUsage           int
    // ConnsInbound 表示入站连接数限制
    ConnsInbound         rcmgr.LimitVal
    // ConnsInboundUsage 表示入站连接数使用情况
    ConnsInboundUsage    int
    // ConnsOutbound 表示出站连接数限制
    ConnsOutbound        rcmgr.LimitVal
    // ConnsOutboundUsage 表示出站连接数使用情况
    ConnsOutboundUsage   int
    // Streams 表示流量限制
    Streams              rcmgr.LimitVal
    // StreamsUsage 表示流量使用情况
    StreamsUsage         int
    // StreamsInbound 表示入站流量限制
    StreamsInbound       rcmgr.LimitVal
    // StreamsInboundUsage 表示入站流量使用情况
    StreamsInboundUsage  int
    // StreamsOutbound 表示出站流量限制
    StreamsOutbound      rcmgr.LimitVal
    // StreamsOutboundUsage 表示出站流量使用情况
    StreamsOutboundUsage int
}

// ToResourceLimits 方法将 ResourceLimitsAndUsage 转换为 rcmgr.ResourceLimits 类型
func (u ResourceLimitsAndUsage) ToResourceLimits() rcmgr.ResourceLimits {
    return rcmgr.ResourceLimits{
        Memory:          u.Memory,
        FD:              u.FD,
        Conns:           u.Conns,
        ConnsInbound:    u.ConnsInbound,
        ConnsOutbound:   u.ConnsOutbound,
        Streams:         u.Streams,
        StreamsInbound:  u.StreamsInbound,
        StreamsOutbound: u.StreamsOutbound,
    }
}

// LimitsConfigAndUsage 结构体定义了限制配置和使用情况，包括系统级别、临时级别、服务、协议和对等体
type LimitsConfigAndUsage struct {
    // System 表示系统级别的资源限制和使用情况
    System    ResourceLimitsAndUsage                 `json:",omitempty"`
    // Transient 表示临时级别的资源限制和使用情况
    Transient ResourceLimitsAndUsage                 `json:",omitempty"`
    // Services 表示服务级别的资源限制和使用情况
    Services  map[string]ResourceLimitsAndUsage      `json:",omitempty"`
    // Protocols 表示协议级别的资源限制和使用情况
    Protocols map[protocol.ID]ResourceLimitsAndUsage `json:",omitempty"`
    // Peers 表示对等体级别的资源限制和使用情况
    Peers     map[peer.ID]ResourceLimitsAndUsage     `json:",omitempty"`
}

// MarshalJSON 方法将 LimitsConfigAndUsage 结构体转换为 JSON 格式
func (u LimitsConfigAndUsage) MarshalJSON() ([]byte, error) {
    // 创建一个 map，将对等体的 ID 编码后作为 key，对应的资源限制和使用情况作为 value
    encodedPeerMap := make(map[string]ResourceLimitsAndUsage, len(u.Peers))
    for p, v := range u.Peers {
        encodedPeerMap[p.String()] = v
    }

    // 定义别名 Alias，用于 JSON 编码
    type Alias LimitsConfigAndUsage
    # 返回一个 JSON 格式的数据，使用 Marshal 方法将结构体转换为 JSON 格式
    return json.Marshal(&struct {
        # 定义一个匿名结构体，包含一个别名为 Alias 的指针和一个 Peers 字段，类型为 map，键为字符串，值为 ResourceLimitsAndUsage 结构体
        *Alias
        Peers map[string]ResourceLimitsAndUsage `json:",omitempty"`
    }{
        # 将 u 转换为 Alias 指针类型，并赋值给匿名结构体的 Alias 字段
        Alias: (*Alias)(&u),
        # 将 encodedPeerMap 赋值给匿名结构体的 Peers 字段
        Peers: encodedPeerMap,
    })
# 将 LimitsConfigAndUsage 结构体转换为 PartialLimitConfig 结构体
func (u LimitsConfigAndUsage) ToPartialLimitConfig() (result rcmgr.PartialLimitConfig) {
    # 将 System 字段转换为 ResourceLimits 结构体
    result.System = u.System.ToResourceLimits()
    # 将 Transient 字段转换为 ResourceLimits 结构体
    result.Transient = u.Transient.ToResourceLimits()

    # 初始化 Service 字段为一个空的 map
    result.Service = make(map[string]rcmgr.ResourceLimits, len(u.Services))
    # 遍历 Services 字段，将每个值转换为 ResourceLimits 结构体，并存入 result.Service 中
    for s, l := range u.Services {
        result.Service[s] = l.ToResourceLimits()
    }
    # 初始化 Protocol 字段为一个空的 map
    result.Protocol = make(map[protocol.ID]rcmgr.ResourceLimits, len(u.Protocols))
    # 遍历 Protocols 字段，将每个值转换为 ResourceLimits 结构体，并存入 result.Protocol 中
    for p, l := range u.Protocols {
        result.Protocol[p] = l.ToResourceLimits()
    }
    # 初始化 Peer 字段为一个空的 map
    result.Peer = make(map[peer.ID]rcmgr.ResourceLimits, len(u.Peers))
    # 遍历 Peers 字段，将每个值转换为 ResourceLimits 结构体，并存入 result.Peer 中
    for p, l := range u.Peers {
        result.Peer[p] = l.ToResourceLimits()
    }

    # 返回转换后的结果
    return
}

# 将 ConcreteLimitConfig 和 ResourceManagerStat 结构体合并为 LimitsConfigAndUsage 结构体
func MergeLimitsAndStatsIntoLimitsConfigAndUsage(l rcmgr.ConcreteLimitConfig, stats rcmgr.ResourceManagerStat) LimitsConfigAndUsage {
    # 将 ConcreteLimitConfig 结构体转换为 PartialLimitConfig 结构体
    limits := l.ToPartialLimitConfig()

    # 返回合并后的 LimitsConfigAndUsage 结构体
    return LimitsConfigAndUsage{
        System:    mergeResourceLimitsAndScopeStatToResourceLimitsAndUsage(limits.System, stats.System),
        Transient: mergeResourceLimitsAndScopeStatToResourceLimitsAndUsage(limits.Transient, stats.Transient),
        Services:  mergeLimitsAndStatsMapIntoLimitsConfigAndUsageMap(limits.Service, stats.Services),
        Protocols: mergeLimitsAndStatsMapIntoLimitsConfigAndUsageMap(limits.Protocol, stats.Protocols),
        Peers:     mergeLimitsAndStatsMapIntoLimitsConfigAndUsageMap(limits.Peer, stats.Peers),
    }
}

# 将两个 map 合并为 ResourceLimitsAndUsage 的 map
func mergeLimitsAndStatsMapIntoLimitsConfigAndUsageMap[K comparable](limits map[K]rcmgr.ResourceLimits, stats map[K]network.ScopeStat) map[K]ResourceLimitsAndUsage {
    # 初始化结果 map
    r := make(map[K]ResourceLimitsAndUsage, maxInt(len(limits), len(stats)))
    # 遍历 stats map
    for p, s := range stats {
        # 初始化 l 为 ResourceLimits 结构体
        var l rcmgr.ResourceLimits
        # 如果 limits 不为空，且包含当前 key，则将其值赋给 l
        if limits != nil {
            if rl, ok := limits[p]; ok {
                l = rl
            }
        }
        # 将 l 和 s 合并为 ResourceLimitsAndUsage 结构体，并存入结果 map 中
        r[p] = mergeResourceLimitsAndScopeStatToResourceLimitsAndUsage(l, s)
    }
}
    # 遍历 limits 中的元素和索引
    for p, s := range limits {
        # 检查 stats 中是否存在索引为 p 的元素
        if _, ok := stats[p]; ok {
            # 如果存在，则跳过当前循环，继续下一个元素
            continue // we already processed this element in the loop above
        }

        # 将 s 和空的 network.ScopeStat 对象传递给 mergeResourceLimitsAndScopeStatToResourceLimitsAndUsage 函数，并将结果存储在 r 中的索引为 p 的位置
        r[p] = mergeResourceLimitsAndScopeStatToResourceLimitsAndUsage(s, network.ScopeStat{})
    }
    # 返回结果字典 r
    return r
}

// 定义一个函数，用于比较两个整数并返回较大的那个
func maxInt(x, y int) int {
    if x > y {
        return x
    }
    return y
}

// 将资源限制和范围统计合并为资源限制和使用情况
func mergeResourceLimitsAndScopeStatToResourceLimitsAndUsage(rl rcmgr.ResourceLimits, ss network.ScopeStat) ResourceLimitsAndUsage {
    return ResourceLimitsAndUsage{
        Memory:               rl.Memory,
        MemoryUsage:          ss.Memory,
        FD:                   rl.FD,
        FDUsage:              ss.NumFD,
        Conns:                rl.Conns,
        ConnsUsage:           ss.NumConnsOutbound + ss.NumConnsInbound,
        ConnsOutbound:        rl.ConnsOutbound,
        ConnsOutboundUsage:   ss.NumConnsOutbound,
        ConnsInbound:         rl.ConnsInbound,
        ConnsInboundUsage:    ss.NumConnsInbound,
        Streams:              rl.Streams,
        StreamsUsage:         ss.NumStreamsOutbound + ss.NumStreamsInbound,
        StreamsOutbound:      rl.StreamsOutbound,
        StreamsOutboundUsage: ss.NumStreamsOutbound,
        StreamsInbound:       rl.StreamsInbound,
        StreamsInboundUsage:  ss.NumStreamsInbound,
    }
}

// 定义 ResourceInfos 类型的切片
type ResourceInfos []ResourceInfo

// 定义 ResourceInfo 结构体
type ResourceInfo struct {
    ScopeName    string
    LimitName    string
    LimitValue   rcmgr.LimitVal64
    CurrentUsage int64
}

// LimitConfigsToInfo 函数获取限制和统计信息，并生成要打印的范围和限制列表
func LimitConfigsToInfo(stats LimitsConfigAndUsage) ResourceInfos {
    result := ResourceInfos{}

    // 将系统范围和瞬态范围的资源限制和使用情况转换为资源信息并添加到结果中
    result = append(result, resourceLimitsAndUsageToResourceInfo(config.ResourceMgrSystemScope, stats.System)...)
    result = append(result, resourceLimitsAndUsageToResourceInfo(config.ResourceMgrTransientScope, stats.Transient)...)

    // 遍历服务范围的限制和使用情况，转换为资源信息并添加到结果中
    for i, s := range stats.Services {
        result = append(result, resourceLimitsAndUsageToResourceInfo(
            config.ResourceMgrServiceScopePrefix+i,
            s,
        )...)
    }
}
    # 遍历 stats.Protocols 中的元素和索引
    for i, p := range stats.Protocols {
        # 将 resourceLimitsAndUsageToResourceInfo 函数返回的结果追加到 result 中
        result = append(result, resourceLimitsAndUsageToResourceInfo(
            config.ResourceMgrProtocolScopePrefix+string(i),
            p,
        )...)
    }

    # 遍历 stats.Peers 中的元素和索引
    for i, p := range stats.Peers {
        # 将 resourceLimitsAndUsageToResourceInfo 函数返回的结果追加到 result 中
        result = append(result, resourceLimitsAndUsageToResourceInfo(
            config.ResourceMgrPeerScopePrefix+i.String(),
            p,
        )...)
    }

    # 返回最终的结果
    return result
# 定义常量，表示资源限制的名称
const (
    limitNameMemory          = "Memory"  # 内存限制
    limitNameFD              = "FD"  # 文件描述符限制
    limitNameConns           = "Conns"  # 连接数限制
    limitNameConnsInbound    = "ConnsInbound"  # 入站连接数限制
    limitNameConnsOutbound   = "ConnsOutbound"  # 出站连接数限制
    limitNameStreams         = "Streams"  # 流限制
    limitNameStreamsInbound  = "StreamsInbound"  # 入站流限制
    limitNameStreamsOutbound = "StreamsOutbound"  # 出站流限制
)

# 定义资源限制的名称列表
var limits = []string{
    limitNameMemory,
    limitNameFD,
    limitNameConns,
    limitNameConnsInbound,
    limitNameConnsOutbound,
    limitNameStreams,
    limitNameStreamsInbound,
    limitNameStreamsOutbound,
}

# 定义函数，将资源限制和使用情况转换为资源信息
func resourceLimitsAndUsageToResourceInfo(scopeName string, stats ResourceLimitsAndUsage) ResourceInfos {
    # 初始化结果字典
    result := ResourceInfos{}
    遍历限制列表，每次循环都会得到一个限制值
    for _, l := range limits {
        创建 ResourceInfo 结构体对象 ri，设置 ScopeName 属性为 scopeName
        ri := ResourceInfo{
            ScopeName: scopeName,
        }
        根据不同的限制值进行不同的处理
        switch l {
        case limitNameMemory:
            设置 ri 的 LimitName 为 limitNameMemory
            ri.LimitName = limitNameMemory
            设置 ri 的 LimitValue 为 stats.Memory
            ri.LimitValue = stats.Memory
            设置 ri 的 CurrentUsage 为 stats.MemoryUsage
            ri.CurrentUsage = stats.MemoryUsage
        case limitNameFD:
            设置 ri 的 LimitName 为 limitNameFD
            ri.LimitName = limitNameFD
            设置 ri 的 LimitValue 为 rcmgr.LimitVal64(stats.FD)
            ri.LimitValue = rcmgr.LimitVal64(stats.FD)
            设置 ri 的 CurrentUsage 为 int64(stats.FDUsage)
            ri.CurrentUsage = int64(stats.FDUsage)
        case limitNameConns:
            设置 ri 的 LimitName 为 limitNameConns
            ri.LimitName = limitNameConns
            设置 ri 的 LimitValue 为 rcmgr.LimitVal64(stats.Conns)
            ri.LimitValue = rcmgr.LimitVal64(stats.Conns)
            设置 ri 的 CurrentUsage 为 int64(stats.ConnsUsage)
            ri.CurrentUsage = int64(stats.ConnsUsage)
        case limitNameConnsInbound:
            设置 ri 的 LimitName 为 limitNameConnsInbound
            ri.LimitName = limitNameConnsInbound
            设置 ri 的 LimitValue 为 rcmgr.LimitVal64(stats.ConnsInbound)
            ri.LimitValue = rcmgr.LimitVal64(stats.ConnsInbound)
            设置 ri 的 CurrentUsage 为 int64(stats.ConnsInboundUsage)
            ri.CurrentUsage = int64(stats.ConnsInboundUsage)
        case limitNameConnsOutbound:
            设置 ri 的 LimitName 为 limitNameConnsOutbound
            ri.LimitName = limitNameConnsOutbound
            设置 ri 的 LimitValue 为 rcmgr.LimitVal64(stats.ConnsOutbound)
            ri.LimitValue = rcmgr.LimitVal64(stats.ConnsOutbound)
            设置 ri 的 CurrentUsage 为 int64(stats.ConnsOutboundUsage)
            ri.CurrentUsage = int64(stats.ConnsOutboundUsage)
        case limitNameStreams:
            设置 ri 的 LimitName 为 limitNameStreams
            ri.LimitName = limitNameStreams
            设置 ri 的 LimitValue 为 rcmgr.LimitVal64(stats.Streams)
            ri.LimitValue = rcmgr.LimitVal64(stats.Streams)
            设置 ri 的 CurrentUsage 为 int64(stats.StreamsUsage)
            ri.CurrentUsage = int64(stats.StreamsUsage)
        case limitNameStreamsInbound:
            设置 ri 的 LimitName 为 limitNameStreamsInbound
            ri.LimitName = limitNameStreamsInbound
            设置 ri 的 LimitValue 为 rcmgr.LimitVal64(stats.StreamsInbound)
            ri.LimitValue = rcmgr.LimitVal64(stats.StreamsInbound)
            设置 ri 的 CurrentUsage 为 int64(stats.StreamsInboundUsage)
            ri.CurrentUsage = int64(stats.StreamsInboundUsage)
        case limitNameStreamsOutbound:
            设置 ri 的 LimitName 为 limitNameStreamsOutbound
            ri.LimitName = limitNameStreamsOutbound
            设置 ri 的 LimitValue 为 rcmgr.LimitVal64(stats.StreamsOutbound)
            ri.LimitValue = rcmgr.LimitVal64(stats.StreamsOutbound)
            设置 ri 的 CurrentUsage 为 int64(stats.StreamsOutboundUsage)
            ri.CurrentUsage = int64(stats.StreamsOutboundUsage)
        }

        如果 LimitValue 为无限制或默认限制，则忽略，不添加到结果中
        if ri.LimitValue == rcmgr.Unlimited64 || ri.LimitValue == rcmgr.DefaultLimit64 {
            // ignore unlimited and unset limits to remove noise from output.
            continue
        }

        将 ResourceInfo 对象 ri 添加到结果列表中
        result = append(result, ri)
    }

    返回结果列表
    return result
func ensureConnMgrMakeSenseVsResourceMgr(concreteLimits rcmgr.ConcreteLimitConfig, cfg config.SwarmConfig) error {
    // 检查是否需要确保连接管理器与资源管理器的配置合理性
    if cfg.ConnMgr.Type.WithDefault(config.DefaultConnMgrType) == "none" || len(cfg.ResourceMgr.Allowlist) != 0 {
        // 如果连接管理器类型为"none"，或者资源管理器的允许列表不为空，则返回nil
        // 如果设置了允许列表，用户可能正在执行某种形式的DoS防御。
        // 在这种情况下，我们不希望修改System.ConnsInbound，因为它可能是合理的，并且保持为"blockAll"，
        // 以便只有在多地址的允许列表内的连接才能建立。
        return nil
    }

    rcm := concreteLimits.ToPartialLimitConfig()

    highWater := cfg.ConnMgr.HighWater.WithDefault(config.DefaultConnMgrHighWater)
    if (rcm.System.Conns > rcmgr.DefaultLimit || rcm.System.Conns == rcmgr.BlockAllLimit) && int64(rcm.System.Conns) <= highWater {
        // 如果资源管理器的System.Conns大于默认限制或者等于BlockAllLimit，并且小于等于ConnMgr.HighWater，则返回错误
        // nolint
        return fmt.Errorf(`
Unable to initialize libp2p due to conflicting resource manager limit configuration.
resource manager System.Conns (%d) must be bigger than ConnMgr.HighWater (%d)
See: https://github.com/ipfs/kubo/blob/master/docs/libp2p-resource-management.md#how-does-the-resource-manager-resourcemgr-relate-to-the-connection-manager-connmgr
`, rcm.System.Conns, highWater)
    }
    if (rcm.System.ConnsInbound > rcmgr.DefaultLimit || rcm.System.ConnsInbound == rcmgr.BlockAllLimit) && int64(rcm.System.ConnsInbound) <= highWater {
        // 如果资源管理器的System.ConnsInbound大于默认限制或者等于BlockAllLimit，并且小于等于ConnMgr.HighWater，则返回错误
        // nolint
        return fmt.Errorf(`
Unable to initialize libp2p due to conflicting resource manager limit configuration.
resource manager System.ConnsInbound (%d) must be bigger than ConnMgr.HighWater (%d)
See: https://github.com/ipfs/kubo/blob/master/docs/libp2p-resource-management.md#how-does-the-resource-manager-resourcemgr-relate-to-the-connection-manager-connmgr
`, rcm.System.ConnsInbound, highWater)
    }
}
    # 如果系统流的数量大于默认限制或者系统流的数量等于阻止所有流的限制并且系统流的数量小于等于高水位
    if rcm.System.Streams > rcmgr.DefaultLimit || rcm.System.Streams == rcmgr.BlockAllLimit && int64(rcm.System.Streams) <= highWater {
        # nolint
        # 返回格式化的错误信息
        return fmt.Errorf(`
// 如果资源管理器的 System.Streams 小于 ConnMgr.HighWater，则无法初始化 libp2p，需要调整资源管理器的配置
// 参考链接：https://github.com/ipfs/kubo/blob/master/docs/libp2p-resource-management.md#how-does-the-resource-manager-resourcemgr-relate-to-the-connection-manager-connmgr
`, rcm.System.Streams, highWater)

// 如果资源管理器的 System.StreamsInbound 大于 rcmgr.DefaultLimit 或等于 rcmgr.BlockAllLimit，并且小于等于 highWater，则无法初始化 libp2p，需要调整资源管理器的配置
// 参考链接：https://github.com/ipfs/kubo/blob/master/docs/libp2p-resource-management.md#how-does-the-resource-manager-resourcemgr-relate-to-the-connection-manager-connmgr
`, rcm.System.StreamsInbound, highWater)
```