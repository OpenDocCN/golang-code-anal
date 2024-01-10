# `v2ray-core\features\policy\policy.go`

```
package policy

import (
    "context"
    "runtime"
    "time"

    "v2ray.com/core/common/platform"
    "v2ray.com/core/features"
)

// Timeout contains limits for connection timeout.
type Timeout struct {
    // Timeout for handshake phase in a connection.
    Handshake time.Duration
    // Timeout for connection being idle, i.e., there is no egress or ingress traffic in this connection.
    ConnectionIdle time.Duration
    // Timeout for an uplink only connection, i.e., the downlink of the connection has been closed.
    UplinkOnly time.Duration
    // Timeout for an downlink only connection, i.e., the uplink of the connection has been closed.
    DownlinkOnly time.Duration
}

// Stats contains settings for stats counters.
type Stats struct {
    // Whether or not to enable stat counter for user uplink traffic.
    UserUplink bool
    // Whether or not to enable stat counter for user downlink traffic.
    UserDownlink bool
}

// Buffer contains settings for internal buffer.
type Buffer struct {
    // Size of buffer per connection, in bytes. -1 for unlimited buffer.
    PerConnection int32
}

// SystemStats contains stat policy settings on system level.
type SystemStats struct {
    // Whether or not to enable stat counter for uplink traffic in inbound handlers.
    InboundUplink bool
    // Whether or not to enable stat counter for downlink traffic in inbound handlers.
    InboundDownlink bool
    // Whether or not to enable stat counter for uplink traffic in outbound handlers.
    OutboundUplink bool
    // Whether or not to enable stat counter for downlink traffic in outbound handlers.
    OutboundDownlink bool
}

// System contains policy settings at system level.
type System struct {
    Stats  SystemStats
    Buffer Buffer
}

// Session is session based settings for controlling V2Ray requests. It contains various settings (or limits) that may differ for different users in the context.
type Session struct {
    Timeouts Timeout // Timeout settings
}
    # 创建名为Stats的变量
    Stats    Stats
    # 创建名为Buffer的变量
    Buffer   Buffer
// Manager 是一个提供给定用户的策略的特性
// v2ray:api:stable
type Manager interface {
    features.Feature

    // ForLevel 根据用户级别返回会话策略
    ForLevel(level uint32) Session

    // ForSystem 返回 V2Ray 系统的系统策略
    ForSystem() System
}

// ManagerType 返回 Manager 接口的类型。可用于实现 common.HasType。
// v2ray:api:stable
func ManagerType() interface{} {
    return (*Manager)(nil)
}

// defaultBufferSize 默认缓冲区大小
var defaultBufferSize int32

// 初始化默认缓冲区大小
func init() {
    const key = "v2ray.ray.buffer.size"
    const defaultValue = -17
    size := platform.EnvFlag{
        Name:    key,
        AltName: platform.NormalizeEnvName(key),
    }.GetValueAsInt(defaultValue)

    switch size {
    case 0:
        defaultBufferSize = -1 // 用于管道的无限大小
    case defaultValue: // 环境标志未定义。根据 CPU-arch 使用默认值。
        switch runtime.GOARCH {
        case "arm", "mips", "mipsle":
            defaultBufferSize = 0
        case "arm64", "mips64", "mips64le":
            defaultBufferSize = 4 * 1024 // 低端设备的 4k 缓存
        default:
            defaultBufferSize = 512 * 1024
        }
    default:
        defaultBufferSize = int32(size) * 1024 * 1024
    }
}

// defaultBufferPolicy 返回默认的缓冲策略
func defaultBufferPolicy() Buffer {
    return Buffer{
        PerConnection: defaultBufferSize,
    }
}

// SessionDefault 在未指定用户时返回策略
func SessionDefault() Session {
    # 返回一个Session对象
    return Session{
        # 设置超时时间
        Timeouts: Timeout{
            # 设置握手超时时间，与nginx的client_header_timeout对齐
            # 以确保该值不会暴露服务器身份
            Handshake:      time.Second * 60,
            # 设置连接空闲超时时间
            ConnectionIdle: time.Second * 300,
            # 设置仅上行数据超时时间
            UplinkOnly:     time.Second * 1,
            # 设置仅下行数据超时时间
            DownlinkOnly:   time.Second * 1,
        },
        # 设置统计信息
        Stats: Stats{
            # 设置用户上行数据统计为false
            UserUplink:   false,
            # 设置用户下行数据统计为false
            UserDownlink: false,
        },
        # 设置缓冲策略为默认缓冲策略
        Buffer: defaultBufferPolicy(),
    }
# 定义一个新的类型 policyKey，类型为 int32
type policyKey int32

# 定义一个常量 bufferPolicyKey，类型为 policyKey，值为 0
const (
    bufferPolicyKey policyKey = 0
)

# 定义一个函数 ContextWithBufferPolicy，接收一个 context 和一个 Buffer 对象，返回一个新的 context
func ContextWithBufferPolicy(ctx context.Context, p Buffer) context.Context {
    # 在给定的 context 中设置一个值，键为 bufferPolicyKey，值为 p
    return context.WithValue(ctx, bufferPolicyKey, p)
}

# 定义一个函数 BufferPolicyFromContext，接收一个 context，返回一个 Buffer 对象
func BufferPolicyFromContext(ctx context.Context) Buffer {
    # 从给定的 context 中获取键为 bufferPolicyKey 的值
    pPolicy := ctx.Value(bufferPolicyKey)
    # 如果值为 nil，则返回默认的缓冲策略
    if pPolicy == nil {
        return defaultBufferPolicy()
    }
    # 将获取到的值转换为 Buffer 对象并返回
    return pPolicy.(Buffer)
}
```