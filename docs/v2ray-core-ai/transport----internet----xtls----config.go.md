# `v2ray-core\transport\internet\xtls\config.go`

```go
// +build !confonly

package xtls

import (
    "crypto/x509"  // 导入加密/解密相关的包
    "sync"  // 导入同步相关的包
    "time"  // 导入时间相关的包

    xtls "github.com/xtls/go"  // 导入 xtls 包

    "v2ray.com/core/common/net"  // 导入网络相关的包
    "v2ray.com/core/common/protocol/tls/cert"  // 导入 TLS 证书相关的包
    "v2ray.com/core/transport/internet"  // 导入网络传输相关的包
)

var (
    globalSessionCache = xtls.NewLRUClientSessionCache(128)  // 创建一个全局的客户端会话缓存
)

// ParseCertificate converts a cert.Certificate to Certificate.
func ParseCertificate(c *cert.Certificate) *Certificate {
    certPEM, keyPEM := c.ToPEM()  // 将 cert.Certificate 转换为 PEM 格式的证书和密钥
    return &Certificate{
        Certificate: certPEM,  // 返回包含证书的 Certificate 对象
        Key:         keyPEM,  // 返回包含密钥的 Certificate 对象
    }
}

func (c *Config) loadSelfCertPool() (*x509.CertPool, error) {
    root := x509.NewCertPool()  // 创建一个新的证书池
    for _, cert := range c.Certificate {  // 遍历配置中的证书
        if !root.AppendCertsFromPEM(cert.Certificate) {  // 将证书添加到证书池中
            return nil, newError("failed to append cert").AtWarning()  // 如果添加失败，则返回错误
        }
    }
    return root, nil  // 返回证书池
}

// BuildCertificates builds a list of TLS certificates from proto definition.
func (c *Config) BuildCertificates() []xtls.Certificate {
    certs := make([]xtls.Certificate, 0, len(c.Certificate))  // 创建一个空的证书列表
    for _, entry := range c.Certificate {  // 遍历配置中的证书
        if entry.Usage != Certificate_ENCIPHERMENT {  // 如果证书用途不是加密
            continue  // 则跳过该证书
        }
        keyPair, err := xtls.X509KeyPair(entry.Certificate, entry.Key)  // 使用证书和密钥创建密钥对
        if err != nil {  // 如果创建失败
            newError("ignoring invalid X509 key pair").Base(err).AtWarning().WriteToLog()  // 记录警告日志
            continue  // 继续下一个证书
        }
        certs = append(certs, keyPair)  // 将密钥对添加到证书列表中
    }
    return certs  // 返回证书列表
}

func isCertificateExpired(c *xtls.Certificate) bool {
    if c.Leaf == nil && len(c.Certificate) > 0 {  // 如果证书的叶子节点为空且证书长度大于0
        if pc, err := x509.ParseCertificate(c.Certificate[0]); err == nil {  // 解析证书的第一个元素
            c.Leaf = pc  // 将解析结果赋值给证书的叶子节点
        }
    }

    // If leaf is not there, the certificate is probably not used yet. We trust user to provide a valid certificate.
    return c.Leaf != nil && c.Leaf.NotAfter.Before(time.Now().Add(-time.Minute))  // 返回证书是否过期的判断结果
}

func issueCertificate(rawCA *Certificate, domain string) (*xtls.Certificate, error) {
    # 使用原始 CA 证书和密钥解析出父证书
    parent, err := cert.ParseCertificate(rawCA.Certificate, rawCA.Key)
    # 如果解析出错，则返回错误信息
    if err != nil:
        return nil, newError("failed to parse raw certificate").Base(err)
    # 使用父证书生成新的证书，包括指定的域名和 DNS 名称
    newCert, err := cert.Generate(parent, cert.CommonName(domain), cert.DNSNames(domain))
    # 如果生成证书出错，则返回错误信息
    if err != nil:
        return nil, newError("failed to generate new certificate for ", domain).Base(err)
    # 将新生成的证书和密钥转换成 PEM 格式
    newCertPEM, newKeyPEM := newCert.ToPEM()
    # 使用证书和密钥创建 X.509 证书和密钥对
    cert, err := xtls.X509KeyPair(newCertPEM, newKeyPEM)
    # 返回创建的证书和可能的错误
    return &cert, err
# 获取自定义CA证书的函数
func (c *Config) getCustomCA() []*Certificate {
    # 创建一个空的证书切片，长度为0，容量为c.Certificate的长度
    certs := make([]*Certificate, 0, len(c.Certificate))
    # 遍历c.Certificate切片中的证书
    for _, certificate := range c.Certificate {
        # 如果证书的用途是Certificate_AUTHORITY_ISSUE，则将证书添加到certs切片中
        if certificate.Usage == Certificate_AUTHORITY_ISSUE {
            certs = append(certs, certificate)
        }
    }
    # 返回包含自定义CA证书的切片
    return certs
}

# 获取GetCertificate函数的函数
func getGetCertificateFunc(c *xtls.Config, ca []*Certificate) func(hello *xtls.ClientHelloInfo) (*xtls.Certificate, error) {
    # 创建一个读写锁
    var access sync.RWMutex
    # 定义一个函数，接收一个指向ClientHelloInfo结构体的指针参数，返回一个指向Certificate结构体和error接口的指针
    return func(hello *xtls.ClientHelloInfo) (*xtls.Certificate, error) {
        # 获取客户端请求的域名
        domain := hello.ServerName
        # 初始化证书过期状态为false
        certExpired := false

        # 读取锁定访问
        access.RLock()
        # 从NameToCertificate映射中查找域名对应的证书
        certificate, found := c.NameToCertificate[domain]
        # 读取解锁访问
        access.RUnlock()

        # 如果找到了对应的证书
        if found {
            # 如果证书未过期，则返回该证书
            if !isCertificateExpired(certificate) {
                return certificate, nil
            }
            # 否则标记证书已过期
            certExpired = true
        }

        # 如果证书已过期
        if certExpired {
            # 创建一个新的证书切片
            newCerts := make([]xtls.Certificate, 0, len(c.Certificates))

            # 写入锁定访问
            access.Lock()
            # 遍历所有证书
            for _, certificate := range c.Certificates {
                # 如果证书未过期，则加入到新的证书切片中
                if !isCertificateExpired(&certificate) {
                    newCerts = append(newCerts, certificate)
                }
            }

            # 更新证书切片
            c.Certificates = newCerts
            # 写入解锁访问
            access.Unlock()
        }

        # 声明一个指向Certificate结构体的指针
        var issuedCertificate *xtls.Certificate

        # 从现有的CA证书中创建一个新的证书
        for _, rawCert := range ca {
            # 如果CA证书的用途是颁发证书
            if rawCert.Usage == Certificate_AUTHORITY_ISSUE {
                # 调用issueCertificate函数颁发新证书
                newCert, err := issueCertificate(rawCert, domain)
                # 如果颁发证书出错，则记录日志并继续下一个CA证书
                if err != nil {
                    newError("failed to issue new certificate for ", domain).Base(err).WriteToLog()
                    continue
                }

                # 写入锁定访问
                access.Lock()
                # 将新证书加入到证书切片中
                c.Certificates = append(c.Certificates, *newCert)
                # 获取最后一个证书的指针
                issuedCertificate = &c.Certificates[len(c.Certificates)-1]
                # 写入解锁访问
                access.Unlock()
                # 跳出循环
                break
            }
        }

        # 如果未成功颁发证书，则返回错误
        if issuedCertificate == nil {
            return nil, newError("failed to create a new certificate for ", domain)
        }

        # 写入锁定访问
        access.Lock()
        # 重新构建NameToCertificate映射
        c.BuildNameToCertificate()
        # 写入解锁访问
        access.Unlock()

        # 返回颁发的证书
        return issuedCertificate, nil
    }
}
// 解析服务器名称
func (c *Config) parseServerName() string {
    return c.ServerName
}

// 将 Config 转换为 xtls.Config
func (c *Config) GetXTLSConfig(opts ...Option) *xtls.Config {
    // 获取根证书
    root, err := c.getCertPool()
    if err != nil {
        newError("failed to load system root certificate").AtError().Base(err).WriteToLog()
    }

    // 创建 xtls.Config 对象
    config := &xtls.Config{
        ClientSessionCache:     globalSessionCache,
        RootCAs:                root,
        InsecureSkipVerify:     c.AllowInsecure,
        NextProtos:             c.NextProtocol,
        SessionTicketsDisabled: c.DisableSessionResumption,
    }
    if c == nil {
        return config
    }

    // 应用额外的选项
    for _, opt := range opts {
        opt(config)
    }

    // 构建证书
    config.Certificates = c.BuildCertificates()
    config.BuildNameToCertificate()

    // 获取自定义 CA
    caCerts := c.getCustomCA()
    if len(caCerts) > 0 {
        config.GetCertificate = getGetCertificateFunc(config, caCerts)
    }

    // 解析服务器名称
    if sn := c.parseServerName(); len(sn) > 0 {
        config.ServerName = sn
    }

    // 设置默认的 ALPN 协议
    if len(config.NextProtos) == 0 {
        config.NextProtos = []string{"h2", "http/1.1"}
    }

    return config
}

// 用于构建 XTLS 配置的选项
type Option func(*xtls.Config)

// 设置 XTLS 配置中的服务器名称
func WithDestination(dest net.Destination) Option {
    return func(config *xtls.Config) {
        if dest.Address.Family().IsDomain() && config.ServerName == "" {
            config.ServerName = dest.Address.Domain()
        }
    }
}

// 设置 XTLS 配置中的 ALPN 值
func WithNextProto(protocol ...string) Option {
    return func(config *xtls.Config) {
        if len(config.NextProtos) == 0 {
            config.NextProtos = protocol
        }
    }
}

// 从流设置中获取 Config。如果找不到则返回 nil
func ConfigFromStreamSettings(settings *internet.MemoryStreamConfig) *Config {
    if settings == nil {
        return nil
    }
    // 尝试将settings.SecuritySettings转换为*Config类型的变量config，同时判断是否转换成功
    config, ok := settings.SecuritySettings.(*Config)
    // 如果转换失败，则返回空值
    if !ok {
        return nil
    }
    // 如果转换成功，则返回config变量
    return config
# 闭合前面的函数定义
```