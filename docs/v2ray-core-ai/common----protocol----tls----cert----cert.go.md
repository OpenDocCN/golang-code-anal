# `v2ray-core\common\protocol\tls\cert\cert.go`

```
package cert

import (
    "crypto/ecdsa"  // 导入椭圆曲线数字签名算法
    "crypto/ed25519"  // 导入Ed25519数字签名算法
    "crypto/elliptic"  // 导入椭圆曲线加密算法
    "crypto/rand"  // 导入随机数生成包
    "crypto/rsa"  // 导入RSA算法
    "crypto/x509"  // 导入X.509证书标准
    "encoding/asn1"  // 导入ASN.1编解码包
    "encoding/pem"  // 导入PEM编解码包
    "math/big"  // 导入大数包
    "time"  // 导入时间包

    "v2ray.com/core/common"  // 导入v2ray通用包
)

//go:generate go run v2ray.com/core/common/errors/errorgen

type Certificate struct {
    Certificate []byte  // 证书的ASN.1 DER格式
    PrivateKey []byte  // 私钥的ASN.1 DER格式
}

func ParseCertificate(certPEM []byte, keyPEM []byte) (*Certificate, error) {
    certBlock, _ := pem.Decode(certPEM)  // 解码证书PEM数据
    if certBlock == nil {
        return nil, newError("failed to decode certificate")  // 如果解码失败，返回错误
    }
    keyBlock, _ := pem.Decode(keyPEM)  // 解码私钥PEM数据
    if keyBlock == nil {
        return nil, newError("failed to decode key")  // 如果解码失败，返回错误
    }
    return &Certificate{
        Certificate: certBlock.Bytes,  // 返回证书的字节数据
        PrivateKey:  keyBlock.Bytes,  // 返回私钥的字节数据
    }, nil
}

func (c *Certificate) ToPEM() ([]byte, []byte) {
    return pem.EncodeToMemory(&pem.Block{Type: "CERTIFICATE", Bytes: c.Certificate}),  // 将证书转换为PEM格式
        pem.EncodeToMemory(&pem.Block{Type: "RSA PRIVATE KEY", Bytes: c.PrivateKey})  // 将私钥转换为PEM格式
}

type Option func(*x509.Certificate)

func Authority(isCA bool) Option {
    return func(cert *x509.Certificate) {
        cert.IsCA = isCA  // 设置证书是否为CA
    }
}

func NotBefore(t time.Time) Option {
    return func(c *x509.Certificate) {
        c.NotBefore = t  // 设置证书的生效时间
    }
}

func NotAfter(t time.Time) Option {
    return func(c *x509.Certificate) {
        c.NotAfter = t  // 设置证书的失效时间
    }
}

func DNSNames(names ...string) Option {
    return func(c *x509.Certificate) {
        c.DNSNames = names  // 设置证书的DNS名称
    }
}

func CommonName(name string) Option {
    return func(c *x509.Certificate) {
        c.Subject.CommonName = name  // 设置证书的通用名称
    }
}

func KeyUsage(usage x509.KeyUsage) Option {
    return func(c *x509.Certificate) {
        c.KeyUsage = usage  // 设置证书的密钥用途
    }
}

func Organization(org string) Option {
    return func(c *x509.Certificate) {
        c.Subject.Organization = []string{org}  // 设置证书的组织
    }
}
func MustGenerate(parent *Certificate, opts ...Option) *Certificate {
    // 调用 Generate 函数生成证书，并处理可能的错误
    cert, err := Generate(parent, opts...)
    common.Must(err) // 调用 common 包中的 Must 函数处理错误
    return cert // 返回生成的证书
}

func publicKey(priv interface{}) interface{} {
    // 根据私钥类型返回对应的公钥
    switch k := priv.(type) {
    case *rsa.PrivateKey:
        return &k.PublicKey
    case *ecdsa.PrivateKey:
        return &k.PublicKey
    case ed25519.PrivateKey:
        return k.Public().(ed25519.PublicKey)
    default:
        return nil
    }
}

func Generate(parent *Certificate, opts ...Option) (*Certificate, error) {
    var (
        pKey      interface{} // 定义变量 pKey 用于存储父证书的私钥
        parentKey interface{} // 定义变量 parentKey 用于存储父证书的公钥
        err       error // 定义变量 err 用于存储错误信息
    )
    // 使用椭圆曲线 P-256 生成自签名证书的私钥
    selfKey, err := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
    if err != nil {
        return nil, newError("failed to generate self private key").Base(err) // 返回生成私钥失败的错误信息
    }
    parentKey = selfKey // 将自签名证书的私钥作为父证书的公钥
    if parent != nil {
        // 解析父证书的私钥
        if _, e := asn1.Unmarshal(parent.PrivateKey, &ecPrivateKey{}); e == nil {
            pKey, err = x509.ParseECPrivateKey(parent.PrivateKey)
        } else if _, e := asn1.Unmarshal(parent.PrivateKey, &pkcs8{}); e == nil {
            pKey, err = x509.ParsePKCS8PrivateKey(parent.PrivateKey)
        } else if _, e := asn1.Unmarshal(parent.PrivateKey, &pkcs1PrivateKey{}); e == nil {
            pKey, err = x509.ParsePKCS1PrivateKey(parent.PrivateKey)
        }
        if err != nil {
            return nil, newError("failed to parse parent private key").Base(err) // 返回解析父证书私钥失败的错误信息
        }
        parentKey = pKey // 将解析得到的父证书私钥作为父证书的公钥
    }

    serialNumberLimit := new(big.Int).Lsh(big.NewInt(1), 128) // 生成序列号的上限
    serialNumber, err := rand.Int(rand.Reader, serialNumberLimit) // 生成证书的序列号
    if err != nil {
        return nil, newError("failed to generate serial number").Base(err) // 返回生成序列号失败的错误信息
    }
    # 创建一个 x509 证书模板，设置证书的基本信息
    template := &x509.Certificate{
        SerialNumber:          serialNumber,  # 设置证书的序列号
        NotBefore:             time.Now().Add(time.Hour * -1),  # 设置证书的生效时间为当前时间的前一小时
        NotAfter:              time.Now().Add(time.Hour),  # 设置证书的过期时间为当前时间的后一小时
        KeyUsage:              x509.KeyUsageKeyEncipherment | x509.KeyUsageDigitalSignature,  # 设置证书的密钥用途
        ExtKeyUsage:           []x509.ExtKeyUsage{x509.ExtKeyUsageServerAuth},  # 设置证书的扩展密钥用途
        BasicConstraintsValid: true,  # 设置证书的基本约束有效性
    }
    
    # 遍历选项列表，对证书模板进行进一步设置
    for _, opt := range opts:
        opt(template)
    
    # 初始化父证书为模板
    parentCert := template
    # 如果存在父证书，则解析父证书的证书数据
    if parent != nil:
        pCert, err := x509.ParseCertificate(parent.Certificate)
        if err != nil:
            return nil, newError("failed to parse parent certificate").Base(err)  # 返回解析父证书失败的错误
        parentCert = pCert  # 将解析后的父证书赋值给 parentCert
    
    # 创建证书的 DER 编码字节流
    derBytes, err := x509.CreateCertificate(rand.Reader, template, parentCert, publicKey(selfKey), parentKey)
    if err != nil:
        return nil, newError("failed to create certificate").Base(err)  # 返回创建证书失败的错误
    
    # 将私钥编码为 PKCS#8 格式
    privateKey, err := x509.MarshalPKCS8PrivateKey(selfKey)
    if err != nil:
        return nil, newError("Unable to marshal private key").Base(err)  # 返回私钥编码失败的错误
    
    # 返回证书和私钥
    return &Certificate{
        Certificate: derBytes,
        PrivateKey:  privateKey,
    }, nil
# 闭合前面的函数定义
```