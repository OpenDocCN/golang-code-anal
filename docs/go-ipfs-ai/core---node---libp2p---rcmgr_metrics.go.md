# `kubo\core\node\libp2p\rcmgr_metrics.go`

```go
package libp2p

import (
    "errors"  // 导入 errors 包，用于处理错误
    "strconv"  // 导入 strconv 包，用于字符串和数字之间的转换

    "github.com/libp2p/go-libp2p/core/network"  // 导入网络核心包
    "github.com/libp2p/go-libp2p/core/peer"  // 导入对等核心包
    "github.com/libp2p/go-libp2p/core/protocol"  // 导入协议核心包
    rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager"  // 导入资源管理器包

    "github.com/prometheus/client_golang/prometheus"  // 导入 Prometheus 客户端包
)

func mustRegister(c prometheus.Collector) {
    err := prometheus.Register(c)  // 注册指定的 Collector
    are := prometheus.AlreadyRegisteredError{}  // 创建 AlreadyRegisteredError 类型的变量
    if errors.As(err, &are) {  // 如果错误是 AlreadyRegisteredError 类型，则返回
        return
    }
    if err != nil {  // 如果错误不为空，则触发 panic
        panic(err)
    }
}

func createRcmgrMetrics() rcmgr.MetricsReporter {
    const (  // 定义常量
        direction = "direction"  // 方向
        usesFD    = "usesFD"  // 使用文件描述符
        protocol  = "protocol"  // 协议
        service   = "service"  // 服务
    )

    connAllowed := prometheus.NewCounterVec(  // 创建新的 CounterVec 类型指标
        prometheus.CounterOpts{  // 指标选项
            Name: "libp2p_rcmgr_conns_allowed_total",  // 指标名称
            Help: "allowed connections",  // 指标帮助信息
        },
        []string{direction, usesFD},  // 标签
    )
    mustRegister(connAllowed)  // 注册指标

    connBlocked := prometheus.NewCounterVec(  // 创建新的 CounterVec 类型指标
        prometheus.CounterOpts{  // 指标选项
            Name: "libp2p_rcmgr_conns_blocked_total",  // 指标名称
            Help: "blocked connections",  // 指标帮助信息
        },
        []string{direction, usesFD},  // 标签
    )
    mustRegister(connBlocked)  // 注册指标

    streamAllowed := prometheus.NewCounterVec(  // 创建新的 CounterVec 类型指标
        prometheus.CounterOpts{  // 指标选项
            Name: "libp2p_rcmgr_streams_allowed_total",  // 指标名称
            Help: "allowed streams",  // 指标帮助信息
        },
        []string{direction},  // 标签
    )
    mustRegister(streamAllowed)  // 注册指标

    streamBlocked := prometheus.NewCounterVec(  // 创建新的 CounterVec 类型指标
        prometheus.CounterOpts{  // 指标选项
            Name: "libp2p_rcmgr_streams_blocked_total",  // 指标名称
            Help: "blocked streams",  // 指标帮助信息
        },
        []string{direction},  // 标签
    )
    mustRegister(streamBlocked)  // 注册指标

    peerAllowed := prometheus.NewCounter(prometheus.CounterOpts{  // 创建新的 Counter 类型指标
        Name: "libp2p_rcmgr_peers_allowed_total",  // 指标名称
        Help: "allowed peers",  // 指标帮助信息
    })
    mustRegister(peerAllowed)  // 注册指标
}
    # 创建一个名为 peerBlocked 的计数器，用于记录被阻止的对等节点数量
    peerBlocked := prometheus.NewCounter(prometheus.CounterOpts{
        Name: "libp2p_rcmgr_peer_blocked_total",  # 设置计数器的名称
        Help: "blocked peers",  # 设置计数器的帮助信息
    })
    mustRegister(peerBlocked)  # 注册计数器
    
    # 创建一个名为 protocolAllowed 的向量计数器，用于记录附加到特定协议的允许流的数量
    protocolAllowed := prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "libp2p_rcmgr_protocols_allowed_total",  # 设置向量计数器的名称
            Help: "allowed streams attached to a protocol",  # 设置向量计数器的帮助信息
        },
        []string{protocol},  # 设置向量计数器的标签，用于区分不同的协议
    )
    mustRegister(protocolAllowed)  # 注册向量计数器
    
    # 创建一个名为 protocolBlocked 的向量计数器，用于记录附加到特定协议的被阻止流的数量
    protocolBlocked := prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "libp2p_rcmgr_protocols_blocked_total",  # 设置向量计数器的名称
            Help: "blocked streams attached to a protocol",  # 设置向量计数器的帮助信息
        },
        []string{protocol},  # 设置向量计数器的标签，用于区分不同的协议
    )
    mustRegister(protocolBlocked)  # 注册向量计数器
    
    # 创建一个名为 protocolPeerBlocked 的向量计数器，用于记录附加到特定协议的特定对等节点的被阻止流的数量
    protocolPeerBlocked := prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "libp2p_rcmgr_protocols_for_peer_blocked_total",  # 设置向量计数器的名称
            Help: "blocked streams attached to a protocol for a specific peer",  # 设置向量计数器的帮助信息
        },
        []string{protocol},  # 设置向量计数器的标签，用于区分不同的协议
    )
    mustRegister(protocolPeerBlocked)  # 注册向量计数器
    
    # 创建一个名为 serviceAllowed 的向量计数器，用于记录附加到特定服务的允许流的数量
    serviceAllowed := prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "libp2p_rcmgr_services_allowed_total",  # 设置向量计数器的名称
            Help: "allowed streams attached to a service",  # 设置向量计数器的帮助信息
        },
        []string{service},  # 设置向量计数器的标签，用于区分不同的服务
    )
    mustRegister(serviceAllowed)  # 注册向量计数器
    
    # 创建一个名为 serviceBlocked 的向量计数器，用于记录附加到特定服务的被阻止流的数量
    serviceBlocked := prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "libp2p_rcmgr_services_blocked_total",  # 设置向量计数器的名称
            Help: "blocked streams attached to a service",  # 设置向量计数器的帮助信息
        },
        []string{service},  # 设置向量计数器的标签，用于区分不同的服务
    )
    mustRegister(serviceBlocked)  # 注册向量计数器
    
    # 创建一个名为 servicePeerBlocked 的向量计数器，用于记录附加到特定服务的特定对等节点的被阻止流的数量
    servicePeerBlocked := prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "libp2p_rcmgr_service_for_peer_blocked_total",  # 设置向量计数器的名称
            Help: "blocked streams attached to a service for a specific peer",  # 设置向量计数器的帮助信息
        },
        []string{service},  # 设置向量计数器的标签，用于区分不同的服务
    )
    mustRegister(servicePeerBlocked)  # 注册向量计数器
    # 创建一个名为 memoryAllowed 的 Prometheus 计数器，用于记录允许的内存分配总数
    memoryAllowed := prometheus.NewCounter(prometheus.CounterOpts{
        Name: "libp2p_rcmgr_memory_allocations_allowed_total",
        Help: "allowed memory allocations",
    })
    # 注册 memoryAllowed 计数器
    mustRegister(memoryAllowed)
    
    # 创建一个名为 memoryBlocked 的 Prometheus 计数器，用于记录被阻止的内存分配总数
    memoryBlocked := prometheus.NewCounter(prometheus.CounterOpts{
        Name: "libp2p_rcmgr_memory_allocations_blocked_total",
        Help: "blocked memory allocations",
    })
    # 注册 memoryBlocked 计数器
    mustRegister(memoryBlocked)
    
    # 返回包含各种指标的 rcmgrMetrics 结构体
    return rcmgrMetrics{
        connAllowed,
        connBlocked,
        streamAllowed,
        streamBlocked,
        peerAllowed,
        peerBlocked,
        protocolAllowed,
        protocolBlocked,
        protocolPeerBlocked,
        serviceAllowed,
        serviceBlocked,
        servicePeerBlocked,
        memoryAllowed,
        memoryBlocked,
    }
// 确保实现了来自 go-libp2p-resource-manager 的接口
var _ rcmgr.MetricsReporter = rcmgrMetrics{}

// 定义 rcmgrMetrics 结构体
type rcmgrMetrics struct {
    connAllowed         *prometheus.CounterVec
    connBlocked         *prometheus.CounterVec
    streamAllowed       *prometheus.CounterVec
    streamBlocked       *prometheus.CounterVec
    peerAllowed         prometheus.Counter
    peerBlocked         prometheus.Counter
    protocolAllowed     *prometheus.CounterVec
    protocolBlocked     *prometheus.CounterVec
    protocolPeerBlocked *prometheus.CounterVec
    serviceAllowed      *prometheus.CounterVec
    serviceBlocked      *prometheus.CounterVec
    servicePeerBlocked  *prometheus.CounterVec
    memoryAllowed       prometheus.Counter
    memoryBlocked       prometheus.Counter
}

// 根据网络方向返回字符串
func getDirection(d network.Direction) string {
    switch d {
    default:
        return ""
    case network.DirInbound:
        return "inbound"
    case network.DirOutbound:
        return "outbound"
    }
}

// 允许连接
func (r rcmgrMetrics) AllowConn(dir network.Direction, usefd bool) {
    r.connAllowed.WithLabelValues(getDirection(dir), strconv.FormatBool(usefd)).Inc()
}

// 阻止连接
func (r rcmgrMetrics) BlockConn(dir network.Direction, usefd bool) {
    r.connBlocked.WithLabelValues(getDirection(dir), strconv.FormatBool(usefd)).Inc()
}

// 允许流
func (r rcmgrMetrics) AllowStream(_ peer.ID, dir network.Direction) {
    r.streamAllowed.WithLabelValues(getDirection(dir)).Inc()
}

// 阻止流
func (r rcmgrMetrics) BlockStream(_ peer.ID, dir network.Direction) {
    r.streamBlocked.WithLabelValues(getDirection(dir)).Inc()
}

// 允许对等点
func (r rcmgrMetrics) AllowPeer(_ peer.ID) {
    r.peerAllowed.Inc()
}

// 阻止对等点
func (r rcmgrMetrics) BlockPeer(_ peer.ID) {
    r.peerBlocked.Inc()
}

// 允许协议
func (r rcmgrMetrics) AllowProtocol(proto protocol.ID) {
    r.protocolAllowed.WithLabelValues(string(proto)).Inc()
}

// 阻止协议
func (r rcmgrMetrics) BlockProtocol(proto protocol.ID) {
    r.protocolBlocked.WithLabelValues(string(proto)).Inc()
}
# 基于协议和对等节点ID，增加被阻止的协议对等节点计数
func (r rcmgrMetrics) BlockProtocolPeer(proto protocol.ID, _ peer.ID) {
    r.protocolPeerBlocked.WithLabelValues(string(proto)).Inc()
}

# 增加被允许的服务计数
func (r rcmgrMetrics) AllowService(svc string) {
    r.serviceAllowed.WithLabelValues(svc).Inc()
}

# 增加被阻止的服务计数
func (r rcmgrMetrics) BlockService(svc string) {
    r.serviceBlocked.WithLabelValues(svc).Inc()
}

# 基于服务和对等节点ID，增加被阻止的服务对等节点计数
func (r rcmgrMetrics) BlockServicePeer(svc string, _ peer.ID) {
    r.servicePeerBlocked.WithLabelValues(svc).Inc()
}

# 增加被允许的内存计数
func (r rcmgrMetrics) AllowMemory(_ int) {
    r.memoryAllowed.Inc()
}

# 增加被阻止的内存计数
func (r rcmgrMetrics) BlockMemory(_ int) {
    r.memoryBlocked.Inc()
}
```