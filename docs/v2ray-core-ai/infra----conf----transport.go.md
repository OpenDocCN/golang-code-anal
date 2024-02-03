# `v2ray-core\infra\conf\transport.go`

```go
package conf

import (
    "v2ray.com/core/common/serial"
    "v2ray.com/core/transport"
    "v2ray.com/core/transport/internet"
)

type TransportConfig struct {
    TCPConfig  *TCPConfig          `json:"tcpSettings"`  // TCP 配置
    KCPConfig  *KCPConfig          `json:"kcpSettings"`  // KCP 配置
    WSConfig   *WebSocketConfig    `json:"wsSettings"`   // WebSocket 配置
    HTTPConfig *HTTPConfig         `json:"httpSettings"` // HTTP 配置
    DSConfig   *DomainSocketConfig `json:"dsSettings"`   // DomainSocket 配置
    QUICConfig *QUICConfig         `json:"quicSettings"` // QUIC 配置
}

// Build implements Buildable.
func (c *TransportConfig) Build() (*transport.Config, error) {
    config := new(transport.Config)

    if c.TCPConfig != nil {
        ts, err := c.TCPConfig.Build()  // 构建 TCP 配置
        if err != nil {
            return nil, newError("failed to build TCP config").Base(err).AtError()  // 返回错误信息
        }
        config.TransportSettings = append(config.TransportSettings, &internet.TransportConfig{
            ProtocolName: "tcp",  // 设置协议名称为 TCP
            Settings:     serial.ToTypedMessage(ts),  // 将配置转换为序列化消息
        })
    }

    if c.KCPConfig != nil {
        ts, err := c.KCPConfig.Build()  // 构建 KCP 配置
        if err != nil {
            return nil, newError("failed to build mKCP config").Base(err).AtError()  // 返回错误信息
        }
        config.TransportSettings = append(config.TransportSettings, &internet.TransportConfig{
            ProtocolName: "mkcp",  // 设置协议名称为 mkcp
            Settings:     serial.ToTypedMessage(ts),  // 将配置转换为序列化消息
        })
    }

    if c.WSConfig != nil {
        ts, err := c.WSConfig.Build()  // 构建 WebSocket 配置
        if err != nil {
            return nil, newError("failed to build WebSocket config").Base(err)  // 返回错误信息
        }
        config.TransportSettings = append(config.TransportSettings, &internet.TransportConfig{
            ProtocolName: "websocket",  // 设置协议名称为 websocket
            Settings:     serial.ToTypedMessage(ts),  // 将配置转换为序列化消息
        })
    }

... (继续注释下面的代码)
    # 如果 HTTPConfig 不为空，则构建 HTTP 配置
    if c.HTTPConfig != nil:
        ts, err := c.HTTPConfig.Build()
        # 如果构建失败，则返回错误
        if err != nil:
            return nil, newError("Failed to build HTTP config.").Base(err)
        # 将构建的配置添加到 TransportSettings 中
        config.TransportSettings = append(config.TransportSettings, &internet.TransportConfig{
            ProtocolName: "http",
            Settings:     serial.ToTypedMessage(ts),
        })

    # 如果 DSConfig 不为空，则构建 DomainSocket 配置
    if c.DSConfig != nil:
        ds, err := c.DSConfig.Build()
        # 如果构建失败，则返回错误
        if err != nil:
            return nil, newError("Failed to build DomainSocket config.").Base(err)
        # 将构建的配置添加到 TransportSettings 中
        config.TransportSettings = append(config.TransportSettings, &internet.TransportConfig{
            ProtocolName: "domainsocket",
            Settings:     serial.ToTypedMessage(ds),
        })

    # 如果 QUICConfig 不为空，则构建 QUIC 配置
    if c.QUICConfig != nil:
        qs, err := c.QUICConfig.Build()
        # 如果构建失败，则返回错误
        if err != nil:
            return nil, newError("Failed to build QUIC config.").Base(err)
        # 将构建的配置添加到 TransportSettings 中
        config.TransportSettings = append(config.TransportSettings, &internet.TransportConfig{
            ProtocolName: "quic",
            Settings:     serial.ToTypedMessage(qs),
        })

    # 返回构建好的配置和 nil 错误
    return config, nil
# 闭合前面的函数定义
```