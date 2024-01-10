# `grype\cmd\grype\cli\options\registry.go`

```
package options

import (
    "os"

    "github.com/anchore/clio"
    "github.com/anchore/stereoscope/pkg/image"
)

type RegistryCredentials struct {
    Authority string `yaml:"authority" json:"authority" mapstructure:"authority"`
    // IMPORTANT: do not show the username, password, or token in any output (sensitive information)
    Username secret `yaml:"username" json:"username" mapstructure:"username"`
    Password secret `yaml:"password" json:"password" mapstructure:"password"`
    Token    secret `yaml:"token" json:"token" mapstructure:"token"`

    TLSCert string `yaml:"tls-cert,omitempty" json:"tls-cert,omitempty" mapstructure:"tls-cert"`
    TLSKey  string `yaml:"tls-key,omitempty" json:"tls-key,omitempty" mapstructure:"tls-key"`
}

type registry struct {
    InsecureSkipTLSVerify bool                  `yaml:"insecure-skip-tls-verify" json:"insecure-skip-tls-verify" mapstructure:"insecure-skip-tls-verify"`
    InsecureUseHTTP       bool                  `yaml:"insecure-use-http" json:"insecure-use-http" mapstructure:"insecure-use-http"`
    Auth                  []RegistryCredentials `yaml:"auth" json:"auth" mapstructure:"auth"`
    CACert                string                `yaml:"ca-cert" json:"ca-cert" mapstructure:"ca-cert"`
}

var _ clio.PostLoader = (*registry)(nil)

func (cfg *registry) PostLoad() error {
    // there may be additional credentials provided by env var that should be appended to the set of credentials
    authority, username, password, token, tlsCert, tlsKey :=
        os.Getenv("GRYPE_REGISTRY_AUTH_AUTHORITY"),  // 获取环境变量中的注册表权限信息
        os.Getenv("GRYPE_REGISTRY_AUTH_USERNAME"),  // 获取环境变量中的注册表用户名
        os.Getenv("GRYPE_REGISTRY_AUTH_PASSWORD"),  // 获取环境变量中的注册表密码
        os.Getenv("GRYPE_REGISTRY_AUTH_TOKEN"),     // 获取环境变量中的注册表令牌
        os.Getenv("GRYPE_REGISTRY_AUTH_TLS_CERT"),  // 获取环境变量中的注册表 TLS 证书
        os.Getenv("GRYPE_REGISTRY_AUTH_TLS_KEY")    // 获取环境变量中的注册表 TLS 密钥
    # 检查是否存在非空的凭据
    if hasNonEmptyCredentials(username, password, token, tlsCert, tlsKey) {
        # 注意：我们在凭据之前添加，以便环境变量优先于磁盘配置。
        # 将凭据添加到 cfg.Auth 中，环境变量优先于磁盘配置
        cfg.Auth = append([]RegistryCredentials{
            {
                Authority: authority,
                Username:  secret(username),
                Password:  secret(password),
                Token:     secret(token),
                TLSCert:   tlsCert,
                TLSKey:    tlsKey,
            },
        }, cfg.Auth...)
    }
    # 返回空值
    return nil
# 检查是否存在非空的凭据，包括用户名、密码、令牌、TLS 证书和 TLS 密钥
func hasNonEmptyCredentials(username, password, token, tlsCert, tlsKey string) bool {
    # 检查是否存在用户名和密码
    hasUserPass := username != "" && password != ""
    # 检查是否存在令牌
    hasToken := token != ""
    # 检查是否存在 TLS 材料
    hasTLSMaterial := tlsCert != "" && tlsKey != ""
    # 返回是否存在非空凭据
    return hasUserPass || hasToken || hasTLSMaterial
}

# 将 registry 结构转换为 image.RegistryOptions 结构
func (cfg *registry) ToOptions() *image.RegistryOptions {
    # 创建一个与 cfg.Auth 长度相同的认证数组
    var auth = make([]image.RegistryCredentials, len(cfg.Auth))
    # 遍历 cfg.Auth，将每个认证信息转换为 image.RegistryCredentials 结构
    for i, a := range cfg.Auth {
        auth[i] = image.RegistryCredentials{
            Authority:  a.Authority,
            Username:   a.Username.String(),
            Password:   a.Password.String(),
            Token:      a.Token.String(),
            ClientCert: a.TLSCert,
            ClientKey:  a.TLSKey,
        }
    }
    # 返回 image.RegistryOptions 结构
    return &image.RegistryOptions{
        InsecureSkipTLSVerify: cfg.InsecureSkipTLSVerify,
        InsecureUseHTTP:       cfg.InsecureUseHTTP,
        Credentials:           auth,
        CAFileOrDir:           cfg.CACert,
    }
}
```