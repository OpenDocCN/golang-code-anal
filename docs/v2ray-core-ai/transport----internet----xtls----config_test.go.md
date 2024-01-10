# `v2ray-core\transport\internet\xtls\config_test.go`

```
package xtls_test

import (
    "crypto/x509"  // 导入crypto/x509包，用于操作和解析X.509证书
    "testing"  // 导入testing包，用于编写测试函数
    "time"  // 导入time包，用于处理时间相关操作

    xtls "github.com/xtls/go"  // 导入xtls包，并重命名为xtls

    "v2ray.com/core/common"  // 导入v2ray.com/core/common包
    "v2ray.com/core/common/protocol/tls/cert"  // 导入v2ray.com/core/common/protocol/tls/cert包
    . "v2ray.com/core/transport/internet/xtls"  // 导入v2ray.com/core/transport/internet/xtls包，并使用"."来省略包名
)

func TestCertificateIssuing(t *testing.T) {
    certificate := ParseCertificate(cert.MustGenerate(nil, cert.Authority(true), cert.KeyUsage(x509.KeyUsageCertSign)))  // 解析生成的证书
    certificate.Usage = Certificate_AUTHORITY_ISSUE  // 设置证书用途为颁发证书

    c := &Config{  // 创建Config对象
        Certificate: []*Certificate{  // 设置Certificate字段为包含certificate的切片
            certificate,
        },
    }

    xtlsConfig := c.GetXTLSConfig()  // 获取XTLS配置
    v2rayCert, err := xtlsConfig.GetCertificate(&xtls.ClientHelloInfo{  // 获取证书
        ServerName: "www.v2fly.org",  // 设置服务器名称
    })
    common.Must(err)  // 检查错误

    x509Cert, err := x509.ParseCertificate(v2rayCert.Certificate[0])  // 解析证书
    common.Must(err)  // 检查错误
    if !x509Cert.NotAfter.After(time.Now()) {  // 如果证书过期时间在当前时间之前
        t.Error("NotAfter: ", x509Cert.NotAfter)  // 输出错误信息
    }
}

func TestExpiredCertificate(t *testing.T) {
    caCert := cert.MustGenerate(nil, cert.Authority(true), cert.KeyUsage(x509.KeyUsageCertSign))  // 生成CA证书
    expiredCert := cert.MustGenerate(caCert, cert.NotAfter(time.Now().Add(time.Minute*-2)), cert.CommonName("www.v2fly.org"), cert.DNSNames("www.v2fly.org"))  // 生成过期证书

    certificate := ParseCertificate(caCert)  // 解析CA证书
    certificate.Usage = Certificate_AUTHORITY_ISSUE  // 设置证书用途为颁发证书

    certificate2 := ParseCertificate(expiredCert)  // 解析过期证书

    c := &Config{  // 创建Config对象
        Certificate: []*Certificate{  // 设置Certificate字段为包含certificate和certificate2的切片
            certificate,
            certificate2,
        },
    }

    xtlsConfig := c.GetXTLSConfig()  // 获取XTLS配置
    v2rayCert, err := xtlsConfig.GetCertificate(&xtls.ClientHelloInfo{  // 获取证书
        ServerName: "www.v2fly.org",  // 设置服务器名称
    })
    common.Must(err)  // 检查错误

    x509Cert, err := x509.ParseCertificate(v2rayCert.Certificate[0])  // 解析证书
    common.Must(err)  // 检查错误
    if !x509Cert.NotAfter.After(time.Now()) {  // 如果证书过期时间在当前时间之前
        t.Error("NotAfter: ", x509Cert.NotAfter)  // 输出错误信息
    }
}

func TestInsecureCertificates(t *testing.T) {
    c := &Config{  // 创建Config对象
        AllowInsecureCiphers: true,  // 设置允许使用不安全的密码
    }

    xtlsConfig := c.GetXTLSConfig()  // 获取XTLS配置
}
    # 如果加密套件列表的长度大于0
    if len(xtlsConfig.CipherSuites) > 0 {
        # 抛出致命错误，显示意外的 TLS 加密套件列表
        t.Fatal("Unexpected tls cipher suites list: ", xtlsConfig.CipherSuites)
    }
# 在测试中对证书签发进行基准测试
func BenchmarkCertificateIssuing(b *testing.B) {
    # 生成一个自签名证书
    certificate := ParseCertificate(cert.MustGenerate(nil, cert.Authority(true), cert.KeyUsage(x509.KeyUsageCertSign)))
    # 设置证书用途为授权签发
    certificate.Usage = Certificate_AUTHORITY_ISSUE

    # 创建一个配置对象，包含上面生成的证书
    c := &Config{
        Certificate: []*Certificate{
            certificate,
        },
    }

    # 获取配置对象的 XTLS 配置
    xtlsConfig := c.GetXTLSConfig()
    # 获取 XTLS 配置中证书的数量
    lenCerts := len(xtlsConfig.Certificates)

    # 重置计时器
    b.ResetTimer()

    # 循环执行基准测试
    for i := 0; i < b.N; i++ {
        # 获取指定服务器名称的证书
        _, _ = xtlsConfig.GetCertificate(&xtls.ClientHelloInfo{
            ServerName: "www.v2fly.org",
        })
        # 从名称到证书的映射中删除指定服务器名称的证书
        delete(xtlsConfig.NameToCertificate, "www.v2fly.org")
        # 重置 XTLS 配置中的证书列表
        xtlsConfig.Certificates = xtlsConfig.Certificates[:lenCerts]
    }
}
```