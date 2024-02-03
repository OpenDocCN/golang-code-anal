# `trojan-go\api\service\config.go`

```go
package service

import "github.com/p4gefau1t/trojan-go/config"

const Name = "API_SERVICE"

type SSLConfig struct {
    Enabled        bool     `json:"enabled" yaml:"enabled"`  // SSL 配置是否启用
    CertPath       string   `json:"cert" yaml:"cert"`        // SSL 证书路径
    KeyPath        string   `json:"key" yaml:"key"`          // SSL 私钥路径
    VerifyClient   bool     `json:"verify_client" yaml:"verify-client"`  // 是否验证客户端证书
    ClientCertPath []string `json:"client_cert" yaml:"client-cert"`      // 客户端证书路径
}

type APIConfig struct {
    Enabled bool      `json:"enabled" yaml:"enabled"`  // API 是否启用
    APIHost string    `json:"api_addr" yaml:"api-addr"`  // API 主机地址
    APIPort int       `json:"api_port" yaml:"api-port"`  // API 端口
    SSL     SSLConfig `json:"ssl" yaml:"ssl"`  // SSL 配置
}

type Config struct {
    API APIConfig `json:"api" yaml:"api"`  // API 配置
}

func init() {
    // 注册配置创建函数
    config.RegisterConfigCreator(Name, func() interface{} {
        return new(Config)
    })
}
```