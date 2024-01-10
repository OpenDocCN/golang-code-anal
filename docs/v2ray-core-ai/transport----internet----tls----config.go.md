# `v2ray-core\transport\internet\tls\config.go`

```
// +build !confonly

package tls

import (
    "crypto/tls" // 导入加密传输层协议包
    "crypto/x509" // 导入加密证书包
    "strings" // 导入字符串处理包
    "sync" // 导入同步包
    "time" // 导入时间包

    "v2ray.com/core/common/net" // 导入网络通用包
    "v2ray.com/core/common/protocol/tls/cert" // 导入 TLS 证书协议包
    "v2ray.com/core/transport/internet" // 导入网络传输包
)

var (
    globalSessionCache = tls.NewLRUClientSessionCache(128) // 创建一个全局的 TLS 客户端会话缓存
)

const exp8357 = "experiment:8357" // 定义一个实验性常量

// ParseCertificate converts a cert.Certificate to Certificate.
// 将 cert.Certificate 转换为 Certificate
func ParseCertificate(c *cert.Certificate) *Certificate {
    certPEM, keyPEM := c.ToPEM() // 获取证书和密钥的 PEM 格式
    return &Certificate{
        Certificate: certPEM, // 返回包含证书和密钥的 Certificate 结构体
        Key:         keyPEM,
    }
}

func (c *Config) loadSelfCertPool() (*x509.CertPool, error) {
    root := x509.NewCertPool() // 创建一个新的证书池
    for _, cert := range c.Certificate { // 遍历配置中的证书
        if !root.AppendCertsFromPEM(cert.Certificate) { // 将证书添加到证书池中
            return nil, newError("failed to append cert").AtWarning() // 如果添加失败，则返回错误
        }
    }
    return root, nil // 返回证书池
}

// BuildCertificates builds a list of TLS certificates from proto definition.
// 从协议定义中构建 TLS 证书列表
func (c *Config) BuildCertificates() []tls.Certificate {
    certs := make([]tls.Certificate, 0, len(c.Certificate)) // 创建一个 TLS 证书列表
    for _, entry := range c.Certificate { // 遍历配置中的证书
        if entry.Usage != Certificate_ENCIPHERMENT { // 如果证书用途不是加密
            continue // 则跳过该证书
        }
        keyPair, err := tls.X509KeyPair(entry.Certificate, entry.Key) // 使用证书和密钥创建 TLS 密钥对
        if err != nil { // 如果创建失败
            newError("ignoring invalid X509 key pair").Base(err).AtWarning().WriteToLog() // 记录警告日志
            continue // 继续下一个证书
        }
        certs = append(certs, keyPair) // 将创建的密钥对添加到证书列表中
    }
    return certs // 返回证书列表
}

func isCertificateExpired(c *tls.Certificate) bool {
    if c.Leaf == nil && len(c.Certificate) > 0 { // 如果证书的 Leaf 字段为空且证书长度大于 0
        if pc, err := x509.ParseCertificate(c.Certificate[0]); err == nil { // 解析证书的第一个元素
            c.Leaf = pc // 将解析结果赋值给证书的 Leaf 字段
        }
    }

    // If leaf is not there, the certificate is probably not used yet. We trust user to provide a valid certificate.
    // 如果 Leaf 字段为空，则证书可能尚未使用。我们相信用户提供了有效的证书。
    return c.Leaf != nil && c.Leaf.NotAfter.Before(time.Now().Add(-time.Minute)) // 返回证书是否过期
}

func issueCertificate(rawCA *Certificate, domain string) (*tls.Certificate, error) {
    // 使用原始 CA 证书和密钥解析父证书
    parent, err := cert.ParseCertificate(rawCA.Certificate, rawCA.Key)
    // 如果解析出错，则返回错误信息
    if err != nil {
        return nil, newError("failed to parse raw certificate").Base(err)
    }
    // 使用父证书生成新的证书，包括指定的域名和 DNS 名称
    newCert, err := cert.Generate(parent, cert.CommonName(domain), cert.DNSNames(domain))
    // 如果生成出错，则返回错误信息
    if err != nil {
        return nil, newError("failed to generate new certificate for ", domain).Base(err)
    }
    // 将新生成的证书和密钥转换成 PEM 格式
    newCertPEM, newKeyPEM := newCert.ToPEM()
    // 使用新生成的证书和密钥创建 X.509 证书和密钥对
    cert, err := tls.X509KeyPair(newCertPEM, newKeyPEM)
    // 返回证书和可能的错误
    return &cert, err
# 获取自定义CA证书的函数
func (c *Config) getCustomCA() []*Certificate {
    # 创建一个空的证书切片，长度为c.Certificate的长度
    certs := make([]*Certificate, 0, len(c.Certificate))
    # 遍历c.Certificate中的每个证书
    for _, certificate := range c.Certificate {
        # 如果证书的用途是Certificate_AUTHORITY_ISSUE，则将其添加到certs切片中
        if certificate.Usage == Certificate_AUTHORITY_ISSUE {
            certs = append(certs, certificate)
        }
    }
    # 返回包含自定义CA证书的切片
    return certs
}

# 获取获取证书函数的函数
func getGetCertificateFunc(c *tls.Config, ca []*Certificate) func(hello *tls.ClientHelloInfo) (*tls.Certificate, error) {
    # 创建一个读写锁
    var access sync.RWMutex
    # 定义一个函数，接收一个tls.ClientHelloInfo类型的参数，返回一个*tls.Certificate和error类型的结果
    return func(hello *tls.ClientHelloInfo) (*tls.Certificate, error) {
        # 获取客户端请求的域名
        domain := hello.ServerName
        # 标记证书是否已过期
        certExpired := false

        # 读取锁，用于并发安全地访问NameToCertificate映射
        access.RLock()
        # 在NameToCertificate映射中查找对应域名的证书
        certificate, found := c.NameToCertificate[domain]
        # 释放读取锁
        access.RUnlock()

        # 如果找到了对应域名的证书
        if found {
            # 如果证书未过期，则直接返回该证书
            if !isCertificateExpired(certificate) {
                return certificate, nil
            }
            # 标记证书已过期
            certExpired = true
        }

        # 如果证书已过期
        if certExpired {
            # 创建一个新的证书切片，用于存储未过期的证书
            newCerts := make([]tls.Certificate, 0, len(c.Certificates))

            # 写入锁，用于并发安全地修改Certificates切片
            access.Lock()
            # 遍历Certificates切片中的证书
            for _, certificate := range c.Certificates {
                # 如果证书未过期，则加入到新的证书切片中
                if !isCertificateExpired(&certificate) {
                    newCerts = append(newCerts, certificate)
                }
            }

            # 将新的证书切片赋值给Certificates
            c.Certificates = newCerts
            # 释放写入锁
            access.Unlock()
        }

        # 声明一个指向tls.Certificate类型的指针，用于存储颁发的证书
        var issuedCertificate *tls.Certificate

        # 从现有的CA证书中创建一个新的证书
        for _, rawCert := range ca {
            # 如果CA证书的用途是颁发证书
            if rawCert.Usage == Certificate_AUTHORITY_ISSUE {
                # 调用issueCertificate函数颁发新证书
                newCert, err := issueCertificate(rawCert, domain)
                # 如果颁发证书出现错误
                if err != nil {
                    # 记录日志并继续下一次循环
                    newError("failed to issue new certificate for ", domain).Base(err).WriteToLog()
                    continue
                }

                # 写入锁，用于并发安全地修改Certificates切片
                access.Lock()
                # 将新颁发的证书加入到Certificates切片中
                c.Certificates = append(c.Certificates, *newCert)
                # 指向新颁发的证书
                issuedCertificate = &c.Certificates[len(c.Certificates)-1]
                # 释放写入锁
                access.Unlock()
                # 跳出循环
                break
            }
        }

        # 如果未颁发新证书，则返回错误
        if issuedCertificate == nil {
            return nil, newError("failed to create a new certificate for ", domain)
        }

        # 写入锁，用于并发安全地修改NameToCertificate映射
        access.Lock()
        # 重新构建NameToCertificate映射
        c.BuildNameToCertificate()
        # 释放写入锁
        access.Unlock()

        # 返回颁发的证书
        return issuedCertificate, nil
    }
// 检查配置是否为实验8357
func (c *Config) IsExperiment8357() bool {
    return strings.HasPrefix(c.ServerName, exp8357)
}

// 解析服务器名称
func (c *Config) parseServerName() string {
    if c.IsExperiment8357() {
        return c.ServerName[len(exp8357):]
    }

    return c.ServerName
}

// 将配置转换为tls.Config
func (c *Config) GetTLSConfig(opts ...Option) *tls.Config {
    // 获取根证书池
    root, err := c.getCertPool()
    if err != nil {
        newError("failed to load system root certificate").AtError().Base(err).WriteToLog()
    }

    // 创建tls.Config对象
    config := &tls.Config{
        ClientSessionCache:     globalSessionCache,
        RootCAs:                root,
        InsecureSkipVerify:     c.AllowInsecure,
        NextProtos:             c.NextProtocol,
        SessionTicketsDisabled: c.DisableSessionResumption,
    }
    // 如果配置为空，则返回config
    if c == nil {
        return config
    }

    // 应用选项
    for _, opt := range opts {
        opt(config)
    }

    // 构建证书
    config.Certificates = c.BuildCertificates()
    config.BuildNameToCertificate()

    // 获取自定义CA证书
    caCerts := c.getCustomCA()
    if len(caCerts) > 0 {
        config.GetCertificate = getGetCertificateFunc(config, caCerts)
    }

    // 解析服务器名称并设置到config中
    if sn := c.parseServerName(); len(sn) > 0 {
        config.ServerName = sn
    }

    // 如果NextProtos为空，则设置默认值
    if len(config.NextProtos) == 0 {
        config.NextProtos = []string{"h2", "http/1.1"}
    }

    return config
}

// 用于构建TLS配置的选项
type Option func(*tls.Config)

// 设置TLS配置中的服务器名称
func WithDestination(dest net.Destination) Option {
    return func(config *tls.Config) {
        if dest.Address.Family().IsDomain() && config.ServerName == "" {
            config.ServerName = dest.Address.Domain()
        }
    }
}

// 设置TLS配置中的ALPN值
func WithNextProto(protocol ...string) Option {
    return func(config *tls.Config) {
        if len(config.NextProtos) == 0 {
            config.NextProtos = protocol
        }
    }
}
// 从流设置中获取配置信息，如果找不到则返回 nil
func ConfigFromStreamSettings(settings *internet.MemoryStreamConfig) *Config {
    // 如果设置为空，则返回 nil
    if settings == nil {
        return nil
    }
    // 从安全设置中获取配置信息
    config, ok := settings.SecuritySettings.(*Config)
    // 如果类型断言失败，则返回 nil
    if !ok {
        return nil
    }
    // 返回获取到的配置信息
    return config
}
```