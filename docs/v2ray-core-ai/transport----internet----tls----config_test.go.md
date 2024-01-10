# `v2ray-core\transport\internet\tls\config_test.go`

```
package tls_test

import (
    gotls "crypto/tls"  // 导入加密通信包
    "crypto/x509"  // 导入证书相关包
    "testing"  // 导入测试包
    "time"  // 导入时间包

    "v2ray.com/core/common"  // 导入通用包
    "v2ray.com/core/common/protocol/tls/cert"  // 导入证书相关包
    . "v2ray.com/core/transport/internet/tls"  // 导入 TLS 包
)

func TestCertificateIssuing(t *testing.T) {
    certificate := ParseCertificate(cert.MustGenerate(nil, cert.Authority(true), cert.KeyUsage(x509.KeyUsageCertSign)))  // 生成证书并解析
    certificate.Usage = Certificate_AUTHORITY_ISSUE  // 设置证书用途为颁发证书

    c := &Config{  // 创建 TLS 配置
        Certificate: []*Certificate{
            certificate,
        },
    }

    tlsConfig := c.GetTLSConfig()  // 获取 TLS 配置
    v2rayCert, err := tlsConfig.GetCertificate(&gotls.ClientHelloInfo{  // 获取证书
        ServerName: "www.v2ray.com",
    })
    common.Must(err)  // 检查错误

    x509Cert, err := x509.ParseCertificate(v2rayCert.Certificate[0])  // 解析证书
    common.Must(err)  // 检查错误
    if !x509Cert.NotAfter.After(time.Now()) {  // 如果证书过期时间在当前时间之前
        t.Error("NotAfter: ", x509Cert.NotAfter)  // 输出错误信息
    }
}

func TestExpiredCertificate(t *testing.T) {
    caCert := cert.MustGenerate(nil, cert.Authority(true), cert.KeyUsage(x509.KeyUsageCertSign))  // 生成 CA 证书
    expiredCert := cert.MustGenerate(caCert, cert.NotAfter(time.Now().Add(time.Minute*-2)), cert.CommonName("www.v2ray.com"), cert.DNSNames("www.v2ray.com"))  // 生成过期证书

    certificate := ParseCertificate(caCert)  // 解析 CA 证书
    certificate.Usage = Certificate_AUTHORITY_ISSUE  // 设置证书用途为颁发证书

    certificate2 := ParseCertificate(expiredCert)  // 解析过期证书

    c := &Config{  // 创建 TLS 配置
        Certificate: []*Certificate{
            certificate,
            certificate2,
        },
    }

    tlsConfig := c.GetTLSConfig()  // 获取 TLS 配置
    v2rayCert, err := tlsConfig.GetCertificate(&gotls.ClientHelloInfo{  // 获取证书
        ServerName: "www.v2ray.com",
    })
    common.Must(err)  // 检查错误

    x509Cert, err := x509.ParseCertificate(v2rayCert.Certificate[0])  // 解析证书
    common.Must(err)  // 检查错误
    if !x509Cert.NotAfter.After(time.Now()) {  // 如果证书过期时间在当前时间之前
        t.Error("NotAfter: ", x509Cert.NotAfter)  // 输出错误信息
    }
}

func TestInsecureCertificates(t *testing.T) {
    c := &Config{  // 创建 TLS 配置
        AllowInsecureCiphers: true,  // 允许使用不安全的加密算法
    }

    tlsConfig := c.GetTLSConfig()  // 获取 TLS 配置
}
    # 如果 TLS 配置中的加密套件列表长度大于 0
    if len(tlsConfig.CipherSuites) > 0 {
        # 则输出错误信息，包含不期望的 TLS 加密套件列表
        t.Fatal("Unexpected tls cipher suites list: ", tlsConfig.CipherSuites)
    }
# 在测试证书签发的基准测试函数中
func BenchmarkCertificateIssuing(b *testing.B) {
    # 生成一个自签名的证书
    certificate := ParseCertificate(cert.MustGenerate(nil, cert.Authority(true), cert.KeyUsage(x509.KeyUsageCertSign)))
    # 设置证书用途为 AUTHORITY_ISSUE
    certificate.Usage = Certificate_AUTHORITY_ISSUE

    # 创建一个配置对象，包含上面生成的证书
    c := &Config{
        Certificate: []*Certificate{
            certificate,
        },
    }

    # 获取 TLS 配置
    tlsConfig := c.GetTLSConfig()
    # 获取证书列表的长度
    lenCerts := len(tlsConfig.Certificates)

    # 重置计时器
    b.ResetTimer()

    # 循环执行基准测试
    for i := 0; i < b.N; i++ {
        # 获取证书并忽略返回值
        _, _ = tlsConfig.GetCertificate(&gotls.ClientHelloInfo{
            ServerName: "www.v2ray.com",
        })
        # 从 NameToCertificate 中删除指定的证书
        delete(tlsConfig.NameToCertificate, "www.v2ray.com")
        # 截取证书列表，保留前 lenCerts 个证书
        tlsConfig.Certificates = tlsConfig.Certificates[:lenCerts]
    }
}
```