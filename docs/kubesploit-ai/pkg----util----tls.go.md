# `kubesploit\pkg\util\tls.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd.

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 无论是许可证的第3版还是任何以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。请参阅GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

// 包名为util
package util
# 导入标准库中的各种加密和证书相关的包
import (
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
	"math.big"
	"os"
	"time"
)

'''
GenerateTLSCert函数将生成一个新的证书。参数中的空值将被替换为随机或空白值。
'''
/*
如果 makeRsa 设置为 true，则生成的密钥是 RSA 密钥（默认为 EC）。

如果 notBefore 和 notAfter 传入 nil 日期，则在过去一年中选择一个随机日期。

如果 notAfter 传入 nil 日期，则日期设置为在 notBefore 参数中提供（或生成）的日期之后的 2 年。

请确保 privkey 是一个合适的私钥。go 实现这个值是具有挑战性的，因此在函数定义中不能进行类型断言。
*/
func GenerateTLSCert(serial *big.Int, subject *pkix.Name, dnsNames []string, notBefore, notAfter *time.Time, privKey crypto.PrivateKey, makeRsa bool) (*tls.Certificate, error) {
	//https://golang.org/src/crypto/tls/generate_cert.go 大部分取自这里
	var err error

	if serial == nil {
		serialNumberLimit := new(big.Int).Lsh(big.NewInt(1), 128) //128 bits tops
		serial, err = rand.Int(rand.Reader, serialNumberLimit)
		if err != nil {
			return nil, err
		}
	}
*/
// 如果 subject 为空，则创建一个空的 pkix.Name 对象
if subject == nil { // 使用指针更容易与 nil 进行比较
    subject = &pkix.Name{} // todo: 生成随机的 subject 属性？
}

// todo: 生成随机的名称？

// 如果 notBefore 为空
if notBefore == nil {

    randDay, err := rand.Int(rand.Reader, big.NewInt(360)) // 不是 365，为了安全起见... 时间和计算机都很难
    if err != nil {
        return nil, err
    }

    b4 := time.Now().AddDate(0, 0, -1*int(randDay.Int64())) // 在过去一年内的某个随机日期
    notBefore = &b4
}

// 如果 notAfter 为空
if notAfter == nil {
    aft := notBefore.AddDate(2, 0, 0) // 在 notBefore 日期之后的 2 年
```
		// 将指向aft的指针赋值给notAfter变量
		notAfter = &aft
	}

	// 创建x509.Certificate结构体实例tpl
	tpl := x509.Certificate{
		// 设置证书序列号
		SerialNumber:          serial,
		// 设置证书主题
		Subject:               *subject,
		// 设置证书的DNS名称
		DNSNames:              dnsNames,
		// 设置证书的生效时间
		NotBefore:             *notBefore,
		// 设置证书的失效时间
		NotAfter:              *notAfter,
		// 设置证书的密钥用途
		KeyUsage:              x509.KeyUsageKeyEncipherment | x509.KeyUsageDigitalSignature | x509.KeyUsageCertSign,
		// 设置证书的扩展密钥用途
		ExtKeyUsage:           []x509.ExtKeyUsage{x509.ExtKeyUsageServerAuth},
		// 设置证书的基本约束是否有效
		BasicConstraintsValid: true,
	}
	// 如果私钥为空
	if privKey == nil {
		// 如果需要生成RSA密钥
		if makeRsa {
			// 生成RSA密钥
			privKey, err = rsa.GenerateKey(rand.Reader, 4096)
			// 如果生成密钥过程中出现错误，则返回nil和错误信息
			if err != nil {
				return nil, err
			}
		} else {
// 使用椭圆曲线 P384 生成 ECDSA 私钥，可能需要检查 P384 是否是最常见的曲线选择
privKey, err = ecdsa.GenerateKey(elliptic.P384(), rand.Reader)
if err != nil {
    return nil, err
}

// 使用私钥创建 x.509 证书
crtBytes, err := x509.CreateCertificate(rand.Reader, &tpl, &tpl, getPublicKey(privKey), privKey)
if err != nil {
    return nil, err
}

// 返回包含证书和私钥的 tls.Certificate 对象
return &tls.Certificate{
    Certificate: [][]byte{crtBytes},
    PrivateKey:  privKey,
}, nil
}

// GetTLSCertificates 解析 PEM 编码的 x.509 证书和密钥文件路径，并返回一个 tls 对象
func GetTLSCertificates(certificate string, key string) (*tls.Certificate, error) {
// 声明一个用于存储 x.509 证书的变量
var cer tls.Certificate
// 声明一个用于存储错误信息的变量
var err error

// 检查 x.509 证书文件是否存在
_, errCrt := os.Stat(certificate)
if errCrt != nil {
    return &cer, fmt.Errorf("there was an error importing the SSL/TLS x509 certificate:\r\n%s", errCrt.Error())
}

// 检查 x.509 密钥文件是否存在
_, errKey := os.Stat(key)
if errKey != nil {
    return &cer, fmt.Errorf("there was an error importing the SSL/TLS x509 key:: %s", errKey.Error())
}

// 加载 x.509 证书和密钥对
cer, err = tls.LoadX509KeyPair(certificate, key)
if err != nil {
    return &cer, fmt.Errorf("there was an error importing the SSL/TLS x509 key pair\r\n%s", err.Error())
}
// 如果证书结构中的证书长度小于1或私钥为空，则返回错误
if len(cer.Certificate) < 1 || cer.PrivateKey == nil {
    return &cer, fmt.Errorf("unable to import certificate because the certificate structure was empty")
}
// 返回证书结构和空错误
return &cer, nil
}

// CheckInsecureFingerprint 计算传入证书的SHA256哈希，并确定是否与Merlin存储库中公开分发的密钥对匹配。任何人都可以解密TLS流量
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
	// merlinCRT是用于Merlin测试的公共x.509证书的SHA1指纹的字符串表示
	merlinCRT := "4af9224c77821bc8a46503cfc2764b94b1fc8aa2521afc627e835f0b3c449f50"

	// 检查公钥SHA1指纹是否与Merlin测试中分发的证书匹配
	if merlinCRT == sha256Fingerprint {
		return true, nil
	}
	return false, nil
}

// getPublicKey接受一个私钥，并从中提供公钥。
// https://golang.org/src/crypto/tls/generate_cert.go
func getPublicKey(priv interface{}) interface{} {
	switch k := priv.(type) {
	case *rsa.PrivateKey:
		return &k.PublicKey
	case *ecdsa.PrivateKey:
		return &k.PublicKey
	default:
		return nil
这部分代码缺少具体的语句和功能，无法添加注释。
```