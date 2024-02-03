# `kubesploit\pkg\util\tls_test.go`

```go
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。有关更多详细信息，请参见GNU通用公共许可证。

// 您应该已经收到GNU通用公共许可证的副本。
// 如果没有，请参见<http://www.gnu.org/licenses/>。

package util

import (
    // 标准库
    "bytes"
    "crypto"
    "crypto/ecdsa"
    "crypto/elliptic"
    "crypto/rand"
    "crypto/rsa"
    "crypto/sha256"
    "crypto/tls"
    "crypto/x509"
    "crypto/x509/pkix"
    "fmt"
    "math/big"
    "testing"
    "time"
)

// 'util'包中导出函数的测试文件
// TestTLSCertGeneration测试从'util'包生成证书
func TestTLSCertGeneration(t *testing.T) {
    // 设置
    // 序列号
    serial := big.NewInt(1337)
    // 主题
    cnString := "It's in that place where I put that thing that time"
    subj := pkix.Name{
        CommonName: cnString,
    }
    // DNS名称
    dnsName := "HackThePlanet.org"
    dnsNames := []string{dnsName}
    // 时间（之前和之后）
    notBefore := time.Now().AddDate(0, 0, -5) // 5天前
    notAfter := time.Now().AddDate(13, 3, 7)  //13年，3个月，7天
    // 私钥
    ecpk, err := ecdsa.GenerateKey(elliptic.P521(), rand.Reader)
    // 如果发生错误，则输出错误信息并终止测试
    if err != nil {
        t.Fatal("Couldn't generate EC key", err)
    }
    // 使用 EC 私钥创建私钥对象
    pk := crypto.PrivateKey(ecpk)

    // 创建证书
    certSetVals, err := GenerateTLSCert(serial, &subj, dnsNames, &notBefore, &notAfter, pk, false)
    // 如果生成证书时发生错误，则输出错误信息并终止测试
    if err != nil {
        t.Fatal("Certificate generation[1] error:" + err.Error())
    }

    // 测试
    x5certSetVals, err := x509.ParseCertificate(certSetVals.Certificate[0])
    // 如果解析 X509 证书时发生错误，则输出错误信息并终止测试
    if err != nil {
        t.Fatal("Could not parse X509 certificate")
    }
    // 检查序列号是否匹配
    if x5certSetVals.SerialNumber.Cmp(serial) != 0 {
        t.Error("Serial number mismatch")
    }
    // 检查主题是否匹配
    if x5certSetVals.Subject.CommonName != cnString {
        t.Error("cn mismatch in subject: \n" + x5certSetVals.Subject.CommonName + "\n should be \n" + cnString)
    }

    // 检查 DNS 名称是否匹配
    if len(x5certSetVals.DNSNames) < 1 || x5certSetVals.DNSNames[0] != dnsName {
        t.Error(fmt.Sprintf("dnsnames failed assignment: should be a length 1 string slice with the only "+
            "contents:\n%s\nbut is:\n%v", dnsName, x5certSetVals.DNSNames))
    }

    // 检查时间是否匹配
    expectYear, expectMonth, expectDay := notBefore.Date()
    certYear, certMonth, certDay := x5certSetVals.NotBefore.Date()
    if expectYear != certYear || expectMonth != certMonth || expectDay != certDay {
        t.Error(fmt.Errorf(
            "before date invalid:\nYear:%v (expected %v)\nMonth:%v (expected %v)\nDay:%v (expected %v)",
            certYear,
            expectYear,
            certMonth,
            expectMonth,
            certDay,
            expectDay,
        ))
    }
    expectYear, expectMonth, expectDay = notAfter.Date()
    certYear, certMonth, certDay = x5certSetVals.NotAfter.Date()
    # 检查预期日期与证书日期是否匹配，如果不匹配则输出错误信息
    if expectYear != certYear || expectMonth != certMonth || expectDay != certDay:
        t.Error(fmt.Errorf(
            "after date invalid:\nYear:%v (expected %v)\nMonth:%v (expected %v)\nDay:%v (expected %v)",
            certYear,
            expectYear,
            certMonth,
            expectMonth,
            certDay,
            expectDay,
        ))

    # 检查证书私钥的曲线名称是否为 "P-521"，如果不是则输出错误信息
    if certSetVals.PrivateKey.(*ecdsa.PrivateKey).Params().Name != "P-521":
        t.Error("Incorrect curve name: " + certSetVals.PrivateKey.(*ecdsa.PrivateKey).Params().Name)

    # TODO 这应该是一个独立的测试用例
    # 测试未设置值是否能够随机化该属性
    x5Certs := []*x509.Certificate{}
    tlsCerts := []*tls.Certificate{}
    for i := 0; i < 10; i++:
        # 生成 TLS 证书，用于测试加密/解密是否正常
        certRand1, err := GenerateTLSCert(nil, nil, nil, nil, nil,
            nil, true)
        if err != nil:
            t.Fatal("Certificate generation[2] error:" + err.Error())
        tlsCerts = append(tlsCerts, certRand1)
        # 解析 TLS 证书的 X.509 证书
        x5CertRand1, err := x509.ParseCertificate(certRand1.Certificate[0])
        if err != nil:
            t.Fatal("Certificate generation[5] error:" + err.Error())
        x5Certs = append(x5Certs, x5CertRand1)
    // 遍历 x5Certs 数组，获取索引 i 和值 cer
    for i, cer := range x5Certs {
        // 检查各个值的独立性

        // 测试生成的时间是否准确

        // timeBefore 必须在今天之前
        if cer.NotBefore.After(time.Now()) {
            t.Error("Generated time incorrect:", cer.NotBefore, "(today: ", time.Now(), ")")
        }
        // timeBefore 不能比一年前还要久
        if cer.NotBefore.Before(time.Now().AddDate(-1, 0, 0)) {
            t.Error("Generated time before too long ago: ", cer.NotBefore, time.Now())
        }
        // timeAfter 必须在 timeBefore 之后两年
        certYear, certMonth, certDay := cer.NotBefore.Date()
        acertYear, acertMonth, acertDay := cer.NotAfter.Date()
        ecertYear, ecertMonth, ecertDay := cer.NotBefore.AddDate(2, 0, 0).Date()
        if acertYear != ecertYear || certDay != ecertDay || certMonth != ecertMonth {
            t.Error("Generated times for cert after inconsistent. Got:", acertYear, acertMonth, acertDay,
                "Expected:", certYear+2, certMonth, certDay)
        }

        // 将证书与其他证书进行比较
        for ii, cer2 := range x5Certs {

            if i == ii {
                continue // 不要比较相同的值
            }
            // 检查序列号是否不同
            if cer.SerialNumber.Cmp(cer2.SerialNumber) == 0 { // 相同的值 :(
                t.Error(fmt.Errorf("serial numbers generated are the same: %d and %d: %v",
                    i,
                    ii,
                    cer.SerialNumber.Int64()))
            }
        }
    }

    k1 := tlsCerts[0].PrivateKey.(*rsa.PrivateKey)
    k1pub := k1.Public().(*rsa.PublicKey)
    k2 := tlsCerts[1].PrivateKey.(*rsa.PrivateKey)
    k2pub := k2.Public().(*rsa.PublicKey)

    // 测试证书的私钥/公钥是否正确（可用于加密/解密操作）
    plain1 := []byte("plainmessage1")
    plain2 := []byte("plainmsg2")
    // 使用 RSA 公钥 k1pub 对 plain1 进行 OAEP 加密，返回密文 ct1 和可能的错误 err
    ct1, err := rsa.EncryptOAEP(sha256.New(), rand.Reader, k1pub, plain1, []byte{})
    // 如果出现错误，则输出错误信息
    if err != nil {
        t.Error("Error during encrypt/decrypt verification 1: ", err)
    }
    // 使用 RSA 公钥 k2pub 对 plain2 进行 OAEP 加密，返回密文 ct2 和可能的错误 err
    ct2, err := rsa.EncryptOAEP(sha256.New(), rand.Reader, k2pub, plain2, []byte{})
    // 如果出现错误，则输出错误信息
    if err != nil {
        t.Error("Error during encrypt/decrypt verification 2: ", err)
    }

    // 使用 RSA 私钥 k1 对密文 ct1 进行 OAEP 解密，返回明文 dec1 和可能的错误 err
    dec1, err := rsa.DecryptOAEP(sha256.New(), rand.Reader, k1, ct1, []byte{})
    // 如果出现错误，则输出错误信息
    if err != nil {
        t.Error("Error during encrypt/decrypt verification 3: ", err)
    }

    // 使用 RSA 私钥 k2 对密文 ct2 进行 OAEP 解密，返回明文 dec2 和可能的错误 err
    dec2, err := rsa.DecryptOAEP(sha256.New(), rand.Reader, k2, ct2, []byte{})
    // 如果出现错误，则输出错误信息
    if err != nil {
        t.Error("Error during encrypt/decrypt verification 4: ", err)
    }

    // 检查解密后的明文是否与原始明文相等，如果不相等则输出错误信息
    if !bytes.Equal(dec1, plain1) || !bytes.Equal(dec2, plain2) {
        t.Error("Error during encrypt/decrypt verification 5: decrypted values don't match (",
            string(plain1),
            string(dec1),
            "), (",
            string(plain2),
            string(dec2),
            ")",
        )
    }

    // 待办事项：测试生成的证书是否可以用于 TLS 操作
# 闭合前面的函数定义
```