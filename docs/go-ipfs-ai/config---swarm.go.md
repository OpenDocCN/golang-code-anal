# `kubo\config\swarm.go`

```go
package config

type SwarmConfig struct {
    // AddrFilters specifies a set libp2p addresses that we should never
    // dial or receive connections from.
    AddrFilters []string

    // DisableBandwidthMetrics disables recording of bandwidth metrics for a
    // slight reduction in memory usage. You probably don't need to set this
    // flag.
    DisableBandwidthMetrics bool

    // DisableNatPortMap turns off NAT port mapping (UPnP, etc.).
    DisableNatPortMap bool

    // DisableRelay explicitly disables the relay transport.
    //
    // Deprecated: This flag is deprecated and is overridden by
    // `Swarm.Transports.Relay` if specified.
    DisableRelay bool `json:",omitempty"`

    // EnableRelayHop makes this node act as a public relay v1
    //
    // Deprecated: The circuit v1 protocol is deprecated.
    // Use `Swarm.RelayService` to configure the circuit v2 relay.
    EnableRelayHop bool `json:",omitempty"`

    // EnableAutoRelay enables the "auto relay user" feature.
    // Node will find and use advertised public relays when it determines that
    // it's not reachable from the public internet.
    //
    // Deprecated: This flag is deprecated and is overridden by
    // `Swarm.RelayClient.Enabled` if specified.
    EnableAutoRelay bool `json:",omitempty"`

    // RelayClient controls the client side of "auto relay" feature.
    // When enabled, the node will use relays if it is not publicly reachable.
    RelayClient RelayClient

    // RelayService.* controls the "relay service".
    // When enabled, node will provide a limited relay service to other peers.
    RelayService RelayService

    // EnableHolePunching enables the hole punching service.
    EnableHolePunching Flag `json:",omitempty"`

    // Transports contains flags to enable/disable libp2p transports.
    Transports Transports

    // ConnMgr configures the connection manager.
    ConnMgr ConnMgr

    // ResourceMgr configures the libp2p Network Resource Manager
}
    # 创建一个名为ResourceMgr的ResourceMgr对象
    ResourceMgr ResourceMgr
}

type RelayClient struct {
    // 启用自动中继功能：如果节点无法公开访问，则将使用中继
    Enabled Flag `json:",omitempty"`

    // StaticRelays 配置在节点无法公开访问时要使用的静态中继。如果设置，自动中继将不会尝试查找任何其他中继服务器。
    StaticRelays []string `json:",omitempty"`
}

// RelayService 配置了电路 v2 中继的资源。
// 对于每个字段，go-ipfs 中都会定义一个合理的默认值。
type RelayService struct {
    // 启用有限的中继服务供其他对等方使用（电路 v2 中继）。
    Enabled Flag `json:",omitempty"`

    // ConnectionDurationLimit 是在重置中继连接之前的时间限制。
    ConnectionDurationLimit *OptionalDuration `json:",omitempty`
    // ConnectionDataLimit 是在重置连接之前中继的数据限制（每个方向）。
    ConnectionDataLimit *OptionalInteger `json:",omitempty`

    // ReservationTTL 是新的（或刷新的）预留的持续时间。
    ReservationTTL *OptionalDuration `json:",omitempty`

    // MaxReservations 是活动中继插槽的最大数量。
    MaxReservations *OptionalInteger `json:",omitempty`
    // MaxCircuits 是每个对等方的最大开放中继连接数；默认为 16。
    MaxCircuits *OptionalInteger `json:",omitempty`
    // BufferSize 是中继连接缓冲区的大小。
    BufferSize *OptionalInteger `json:",omitempty`

    // MaxReservationsPerPeer 是源自同一对等方的预留的最大数量。
    MaxReservationsPerPeer *OptionalInteger `json:",omitempty`
    // MaxReservationsPerIP 是源自同一 IP 地址的预留的最大数量。
    MaxReservationsPerIP *OptionalInteger `json:",omitempty`
    // MaxReservationsPerASN 是源自同一 ASN 的预留的最大数量。
    MaxReservationsPerASN *OptionalInteger `json:",omitempty`
}
type Transports struct {
    // Network specifies the base transports we'll use for dialing. To
    // listen on a transport, add the transport to your Addresses.Swarm.
    Network struct {
        // All default to on.
        QUIC         Flag `json:",omitempty"`  // QUIC transport flag
        TCP          Flag `json:",omitempty"`  // TCP transport flag
        Websocket    Flag `json:",omitempty"`  // Websocket transport flag
        Relay        Flag `json:",omitempty"`  // Relay transport flag
        WebTransport Flag `json:",omitempty"`  // WebTransport transport flag
        // except WebRTCDirect which is experimental and opt-in.
        WebRTCDirect Flag `json:",omitempty"`  // WebRTCDirect transport flag
    }

    // Security specifies the transports used to encrypt insecure network
    // transports.
    Security struct {
        // Defaults to 100.
        TLS Priority `json:",omitempty"`  // TLS security priority
        // Defaults to 200.
        SECIO Priority `json:",omitempty"`  // SECIO security priority
        // Defaults to 300.
        Noise Priority `json:",omitempty"`  // Noise security priority
    }

    // Multiplexers specifies the transports used to multiplex multiple
    // connections over a single duplex connection.
    Multiplexers struct {
        // Defaults to 100.
        Yamux Priority `json:",omitempty"`  // Yamux multiplexer priority
        // Defaults to -1.
        Mplex Priority `json:",omitempty"`  // Mplex multiplexer priority
    }
}

// ConnMgr defines configuration options for the libp2p connection manager.
type ConnMgr struct {
    Type        *OptionalString   `json:",omitempty"`  // Connection manager type
    LowWater    *OptionalInteger  `json:",omitempty"`  // Low water mark for connection manager
    HighWater   *OptionalInteger  `json:",omitempty"`  // High water mark for connection manager
    GracePeriod *OptionalDuration `json:",omitempty"`  // Grace period for connection manager
}

// ResourceMgr defines configuration options for the libp2p Network Resource Manager
// <https://github.com/libp2p/go-libp2p/tree/master/p2p/host/resource-manager#readme>
type ResourceMgr struct {
    // Enables the Network Resource Manager feature, default to on.
    Enabled Flag        `json:",omitempty"`  // Network Resource Manager enabled flag
    Limits  swarmLimits `json:",omitempty"`  // Swarm limits for Network Resource Manager

    MaxMemory          *OptionalString  `json:",omitempty"`  // Maximum memory for Network Resource Manager
    MaxFileDescriptors *OptionalInteger `json:",omitempty"`  // Maximum file descriptors for Network Resource Manager
}
    // 定义一个可以绕过正常系统限制的多地址列表（但仍受允许列表范围限制）。方便配置，围绕 https://pkg.go.dev/github.com/libp2p/go-libp2p/p2p/host/resource-manager#Allowlist.Add
    Allowlist []string `json:",omitempty"`
# 定义常量 ResourceMgrSystemScope，表示资源管理器的系统范围
ResourceMgrSystemScope         = "system"
# 定义常量 ResourceMgrTransientScope，表示资源管理器的瞬态范围
ResourceMgrTransientScope      = "transient"
# 定义常量 ResourceMgrServiceScopePrefix，表示资源管理器的服务范围前缀
ResourceMgrServiceScopePrefix  = "svc:"
# 定义常量 ResourceMgrProtocolScopePrefix，表示资源管理器的协议范围前缀
ResourceMgrProtocolScopePrefix = "proto:"
# 定义常量 ResourceMgrPeerScopePrefix，表示资源管理器的对等范围前缀
ResourceMgrPeerScopePrefix     = "peer:"
```