# `kubesploit\pkg\util\tls.go`

```go
// Kubesploit 是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit 是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，不附带任何保证；包括适销性或特定用途适用性的暗示保证。有关更多详细信息，请参见GNU通用公共许可证。

// 您应该已经收到GNU通用公共许可证的副本。
// 如果没有，请参见<http://www.gnu.org/licenses/>。

package util

import (
    // 标准库
    "crypto"
    "crypto/ecdsa"
    "crypto/elliptic"
    "crypto/rand"
    "crypto/rsa"
    "crypto/sha256"
    "crypto/tls"
    "crypto/x509"
    "crypto/x509/pkix"
    "encoding/hex"
    "fmt"
    "math/big"
    "os"
    "time"
)

/*
GenerateTLSCert将生成一个新的证书。参数中的nil值将被替换为随机值或空值。

如果makeRsa设置为true，则生成的密钥是RSA密钥（默认为EC）。

如果notBefore和notAfter传入nil日期，则会在过去一年内选择一个随机日期。

如果notAfter传入nil日期，则日期将设置为在提供的（或生成的）notBefore参数之后2年。

请确保privkey是一个合适的私钥。这个值的go实现很具有挑战性，所以在函数定义中无法进行类型断言。
*/
# 生成 TLS 证书
func GenerateTLSCert(serial *big.Int, subject *pkix.Name, dnsNames []string, notBefore, notAfter *time.Time, privKey crypto.PrivateKey, makeRsa bool) (*tls.Certificate, error):
    # 从 https://golang.org/src/crypto/tls/generate_cert.go 大部分内容来自此处
    var err error

    if serial == nil:
        # 生成一个 128 位的随机序列号
        serialNumberLimit := new(big.Int).Lsh(big.NewInt(1), 128) //128 bits tops
        serial, err = rand.Int(rand.Reader, serialNumberLimit)
        if err != nil:
            return nil, err

    if subject == nil: # 使用指针更容易与 nil 进行比较
        subject = &pkix.Name{} # todo: 生成随机主题属性？
    
    # todo: 生成随机名称？

    if notBefore == nil:
        # 生成一个随机的起始日期
        randDay, err := rand.Int(rand.Reader, big.NewInt(360)) #not 365, playing it safe... time and computers are hard
        if err != nil:
            return nil, err
        b4 := time.Now().AddDate(0, 0, -1*int(randDay.Int64())) #random date sometime in the last year
        notBefore = &b4

    if notAfter == nil:
        # 设置证书的过期日期为起始日期的两年后
        aft := notBefore.AddDate(2, 0, 0) //2 years after the notbefore date
        notAfter = &aft

    # 创建证书模板
    tpl := x509.Certificate{
        SerialNumber:          serial,
        Subject:               *subject,
        DNSNames:              dnsNames,
        NotBefore:             *notBefore,
        NotAfter:              *notAfter,
        KeyUsage:              x509.KeyUsageKeyEncipherment | x509.KeyUsageDigitalSignature | x509.KeyUsageCertSign,
        ExtKeyUsage:           []x509.ExtKeyUsage{x509.ExtKeyUsageServerAuth},
        BasicConstraintsValid: true,
    }
    # 如果私钥为空
    if privKey == nil:
        # 如果需要生成 RSA 密钥对
        if makeRsa:
            # 使用随机数生成器和 4096 位长度生成 RSA 密钥对
            privKey, err = rsa.GenerateKey(rand.Reader, 4096)
            # 如果生成失败，返回错误
            if err != nil:
                return nil, err
        else:
            # 否则，使用椭圆曲线 P384 生成 ECDSA 密钥对
            privKey, err = ecdsa.GenerateKey(elliptic.P384(), rand.Reader) #maybe check to see if P384 is the right choice (would want to be the most common choice for ec curves)
            # 如果生成失败，返回错误
            if err != nil:
                return nil, err

    # 使用模板和私钥创建证书
    crtBytes, err := x509.CreateCertificate(rand.Reader, &tpl, &tpl, getPublicKey(privKey), privKey)
    # 如果创建失败，返回错误
    if err != nil:
        return nil, err

    # 返回 TLS 证书对象
    return &tls.Certificate{
        Certificate: [][]byte{crtBytes},
        PrivateKey:  privKey,
    }, nil
// GetTLSCertificates函数：解析PEM编码的x.509证书和密钥文件路径作为字符串，并返回一个tls对象
func GetTLSCertificates(certificate string, key string) (*tls.Certificate, error) {
    var cer tls.Certificate
    var err error

    // 检查磁盘上是否存在x.509证书文件
    _, errCrt := os.Stat(certificate)
    if errCrt != nil {
        return &cer, fmt.Errorf("there was an error importing the SSL/TLS x509 certificate:\r\n%s", errCrt.Error())
    }

    // 检查磁盘上是否存在x.509密钥文件
    _, errKey := os.Stat(key)
    if errKey != nil {
        return &cer, fmt.Errorf("there was an error importing the SSL/TLS x509 key:: %s", errKey.Error())
    }

    cer, err = tls.LoadX509KeyPair(certificate, key)
    if err != nil {
        return &cer, fmt.Errorf("there was an error importing the SSL/TLS x509 key pair\r\n%s", err.Error())
    }

    if len(cer.Certificate) < 1 || cer.PrivateKey == nil {
        return &cer, fmt.Errorf("unable to import certificate because the certificate structure was empty")
    }
    return &cer, nil
}

// CheckInsecureFingerprint函数：计算传入证书的SHA256哈希，并确定是否与Merlin存储库中公开分发的密钥对匹配。任何人都可以解密TLS流量
func CheckInsecureFingerprint(certificate tls.Certificate) (bool, error) {
    // 解析为X.509格式
    x509Certificate, errX509 := x509.ParseCertificate(certificate.Certificate[0])
    if errX509 != nil {
        return false, fmt.Errorf("there was an error parsing the tls.Certificate structure into a x509.Certificate"+
            " structure:\r\n%s", errX509.Error())
    }

    // 创建指纹
    S256 := sha256.Sum256(x509Certificate.Raw)
    sha256Fingerprint := hex.EncodeToString(S256[:])

    // merlinCRT是Merlin分发的公共x.509证书的SHA1指纹的字符串表示形式
    // 将指纹字符串赋值给变量merlinCRT
    merlinCRT := "4af9224c77821bc8a46503cfc2764b94b1fc8aa2521afc627e835f0b3c449f50"

    // 检查公钥的SHA1指纹是否与Merlin测试中分发的证书匹配
    if merlinCRT == sha256Fingerprint {
        // 如果匹配，则返回true和空错误
        return true, nil
    }
    // 如果不匹配，则返回false和空错误
    return false, nil
// getPublicKey函数接受一个私钥，并从中提取公钥。
// https://golang.org/src/crypto/tls/generate_cert.go
func getPublicKey(priv interface{}) interface{} {
    // 使用类型断言判断私钥的类型
    switch k := priv.(type) {
    // 如果是RSA私钥，则返回对应的公钥
    case *rsa.PrivateKey:
        return &k.PublicKey
    // 如果是ECDSA私钥，则返回对应的公钥
    case *ecdsa.PrivateKey:
        return &k.PublicKey
    // 其他情况返回空值
    default:
        return nil
    }
}
```