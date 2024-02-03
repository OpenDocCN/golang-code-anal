# `v2ray-core\common\protocol\tls\cert\privateKey.go`

```go
package cert

import (
    "crypto/x509/pkix"  // 导入 pkix 包，用于处理证书相关的数据结构
    "encoding/asn1"     // 导入 asn1 包，用于处理 ASN.1 编解码
    "math/big"          // 导入 big 包，用于处理大整数
)

type ecPrivateKey struct {
    Version       int
    PrivateKey    []byte
    NamedCurveOID asn1.ObjectIdentifier `asn1:"optional,explicit,tag:0"`  // 椭圆曲线私钥的命名曲线 OID
    PublicKey     asn1.BitString        `asn1:"optional,explicit,tag:1"`  // 椭圆曲线私钥的公钥
}

type pkcs8 struct {
    Version    int
    Algo       pkix.AlgorithmIdentifier  // PKCS#8 私钥的算法标识符
    PrivateKey []byte
    // optional attributes omitted.  // 省略了可选的属性
}

type pkcs1AdditionalRSAPrime struct {
    Prime *big.Int  // RSA 私钥的附加素数

    // We ignore these values because rsa will calculate them.
    Exp   *big.Int  // 我们忽略这些值，因为 RSA 将会计算它们
    Coeff *big.Int  // 我们忽略这些值，因为 RSA 将会计算它们
}

type pkcs1PrivateKey struct {
    Version int
    N       *big.Int  // RSA 私钥的模数
    E       int       // RSA 私钥的公钥指数
    D       *big.Int  // RSA 私钥的私钥指数
    P       *big.Int  // RSA 私钥的第一个质数因子
    Q       *big.Int  // RSA 私钥的第二个质数因子
    // We ignore these values, if present, because rsa will calculate them.
    Dp   *big.Int `asn1:"optional"`  // 我们忽略这些值，如果存在的话，因为 RSA 将会计算它们
    Dq   *big.Int `asn1:"optional"`  // 我们忽略这些值，如果存在的话，因为 RSA 将会计算它们
    Qinv *big.Int `asn1:"optional"`  // 我们忽略这些值，如果存在的话，因为 RSA 将会计算它们

    AdditionalPrimes []pkcs1AdditionalRSAPrime `asn1:"optional,omitempty"`  // RSA 私钥的附加素数，可选的
}
```