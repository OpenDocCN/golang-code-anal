# `v2ray-core\app\dispatcher\sniffer.go`

```
// +build !confonly

package dispatcher

import (
    "v2ray.com/core/common"
    "v2ray.com/core/common/protocol/bittorrent"
    "v2ray.com/core/common/protocol/http"
    "v2ray.com/core/common/protocol/tls"
)

// SniffResult 定义了协议嗅探结果的接口
type SniffResult interface {
    Protocol() string
    Domain() string
}

// protocolSniffer 定义了协议嗅探函数的类型
type protocolSniffer func([]byte) (SniffResult, error)

// Sniffer 定义了嗅探器的结构
type Sniffer struct {
    sniffer []protocolSniffer
}

// NewSniffer 创建一个新的嗅探器对象
func NewSniffer() *Sniffer {
    return &Sniffer{
        sniffer: []protocolSniffer{
            func(b []byte) (SniffResult, error) { return http.SniffHTTP(b) },
            func(b []byte) (SniffResult, error) { return tls.SniffTLS(b) },
            func(b []byte) (SniffResult, error) { return bittorrent.SniffBittorrent(b) },
        },
    }
}

// errUnknownContent 定义了未知内容错误
var errUnknownContent = newError("unknown content")

// Sniff 根据给定的数据进行嗅探
func (s *Sniffer) Sniff(payload []byte) (SniffResult, error) {
    var pendingSniffer []protocolSniffer
    for _, s := range s.sniffer {
        result, err := s(payload)
        if err == common.ErrNoClue {
            pendingSniffer = append(pendingSniffer, s)
            continue
        }

        if err == nil && result != nil {
            return result, nil
        }
    }

    if len(pendingSniffer) > 0 {
        s.sniffer = pendingSniffer
        return nil, common.ErrNoClue
    }

    return nil, errUnknownContent
}
```