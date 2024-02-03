# `kubo\config\autonat.go`

```go
package config

import (
    "fmt"
)

// AutoNATServiceMode configures the ipfs node's AutoNAT service.
type AutoNATServiceMode int

const (
    // AutoNATServiceUnset indicates that the user has not set the
    // AutoNATService mode.
    //
    // When unset, nodes configured to be public DHT nodes will _also_
    // perform limited AutoNAT dialbacks.
    AutoNATServiceUnset AutoNATServiceMode = iota
    // AutoNATServiceEnabled indicates that the user has enabled the
    // AutoNATService.
    AutoNATServiceEnabled
    // AutoNATServiceDisabled indicates that the user has disabled the
    // AutoNATService.
    AutoNATServiceDisabled
)

func (m *AutoNATServiceMode) UnmarshalText(text []byte) error {
    // 根据传入的文本内容设置 AutoNATServiceMode 的值
    switch string(text) {
    case "":
        *m = AutoNATServiceUnset
    case "enabled":
        *m = AutoNATServiceEnabled
    case "disabled":
        *m = AutoNATServiceDisabled
    default:
        return fmt.Errorf("unknown autonat mode: %s", string(text))
    }
    return nil
}

func (m AutoNATServiceMode) MarshalText() ([]byte, error) {
    // 根据 AutoNATServiceMode 的值返回对应的文本内容
    switch m {
    case AutoNATServiceUnset:
        return nil, nil
    case AutoNATServiceEnabled:
        return []byte("enabled"), nil
    case AutoNATServiceDisabled:
        return []byte("disabled"), nil
    default:
        return nil, fmt.Errorf("unknown autonat mode: %d", m)
    }
}

// AutoNATConfig configures the node's AutoNAT subsystem.
type AutoNATConfig struct {
    // ServiceMode configures the node's AutoNAT service mode.
    ServiceMode AutoNATServiceMode `json:",omitempty"`

    // Throttle configures AutoNAT dialback throttling.
    //
    // If unset, the conservative libp2p defaults will be unset. To help the
    // network, please consider setting this and increasing the limits.
    //
    // By default, the limits will be a total of 30 dialbacks, with a
    // per-peer max of 3 peer, resetting every minute.
    Throttle *AutoNATThrottleConfig `json:",omitempty"`
}
// AutoNATThrottleConfig 结构体用于配置自动NAT限制
type AutoNATThrottleConfig struct {
    // GlobalLimit 和 PeerLimit 分别设置全局和每个对等体的拨号限制
    // AutoNAT服务每个时间间隔只会执行指定数量的拨号
    //
    // 将任一值设置为0将禁用相应的限制
    GlobalLimit, PeerLimit int

    // Interval 指定此节点应该多久重置一次全局/对等体拨号限制
    //
    // 当未设置时，默认为1分钟
    Interval OptionalDuration `json:",omitempty"`
}
```