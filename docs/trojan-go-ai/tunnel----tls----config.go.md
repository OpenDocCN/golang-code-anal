# `trojan-go\tunnel\tls\config.go`

```go
package tls

import (
    "github.com/p4gefau1t/trojan-go/config"
)

type Config struct {
    RemoteHost string          `json:"remote_addr" yaml:"remote-addr"`  // 远程主机地址
    RemotePort int             `json:"remote_port" yaml:"remote-port"`  // 远程主机端口
    TLS        TLSConfig       `json:"ssl" yaml:"ssl"`                  // TLS 配置
    Websocket  WebsocketConfig `json:"websocket" yaml:"websocket"`      // Websocket 配置
}

type WebsocketConfig struct {
    Enabled bool `json:"enabled" yaml:"enabled"`  // 是否启用 Websocket
}

type TLSConfig struct {
    Verify               bool     `json:"verify" yaml:"verify"`  // 是否验证证书
    VerifyHostName       bool     `json:"verify_hostname" yaml:"verify-hostname"`  // 是否验证主机名
    CertPath             string   `json:"cert" yaml:"cert"`      // 证书路径
    KeyPath              string   `json:"key" yaml:"key"`        // 密钥路径
    KeyPassword          string   `json:"key_password" yaml:"key-password"`  // 密钥密码
    Cipher               string   `json:"cipher" yaml:"cipher"`  // 加密算法
    PreferServerCipher   bool     `json:"prefer_server_cipher" yaml:"prefer-server-cipher"`  // 是否优先使用服务器端加密算法
    SNI                  string   `json:"sni" yaml:"sni"`        // SNI
    HTTPResponseFileName string   `json:"plain_http_response" yaml:"plain-http-response"`  // HTTP 响应文件名
    FallbackHost         string   `json:"fallback_addr" yaml:"fallback-addr"`  // 回退主机地址
    FallbackPort         int      `json:"fallback_port" yaml:"fallback-port"`  // 回退主机端口
    ReuseSession         bool     `json:"reuse_session" yaml:"reuse-session"`  // 是否重用会话
    ALPN                 []string `json:"alpn" yaml:"alpn"`      // ALPN 协议
    Curves               string   `json:"curves" yaml:"curves"`  // 曲线
    Fingerprint          string   `json:"fingerprint" yaml:"fingerprint"`  // 指纹
    KeyLogPath           string   `json:"key_log" yaml:"key-log"`  // 密钥日志路径
    CertCheckRate        int      `json:"cert_check_rate" yaml:"cert-check-rate"`  // 证书检查频率
}

func init() {
    # 注册配置创建器，将名称和创建函数关联起来
    config.RegisterConfigCreator(Name, func() interface{} {
        # 返回一个新的配置对象，包含默认的 TLS 配置
        return &Config{
            TLS: TLSConfig{
                Verify:         true,            # 启用 TLS 验证
                VerifyHostName: true,            # 启用主机名验证
                Fingerprint:    "",              # 设置证书指纹
                ALPN:           []string{"http/1.1"},  # 设置应用层协议
            },
        }
    })
# 闭合前面的函数定义
```