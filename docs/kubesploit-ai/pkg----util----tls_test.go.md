# `kubesploit\pkg\util\tls_test.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd.

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何担保；包括适销性或特定用途的隐含担保。请参阅GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本。
// 如果没有，请参阅<http://www.gnu.org/licenses/>。

// 包声明
package util
// 导入标准库中的模块
import (
	"bytes"  // 字节操作
	"crypto"  // 加密算法
	"crypto/ecdsa"  // 椭圆曲线数字签名算法
	"crypto/elliptic"  // 椭圆曲线加密
	"crypto/rand"  // 生成随机数
	"crypto/rsa"  // RSA加密算法
	"crypto/sha256"  // SHA256哈希算法
	"crypto/tls"  // TLS协议
	"crypto/x509"  // X.509证书标准
	"crypto/x509/pkix"  // PKIX证书标准
	"fmt"  // 格式化输出
	"math/big"  // 大数运算
	"testing"  // 测试框架
	"time"  // 时间操作
)

// 'util'包中导出函数的测试文件
// TestTLSCertGeneration 测试util包中的证书生成函数
func TestTLSCertGeneration(t *testing.T) {
	// 设置测试环境

	// 生成证书序列号
	serial := big.NewInt(1337)

	// 设置证书的主体信息
	cnString := "It's in that place where I put that thing that time"
	subj := pkix.Name{
		CommonName: cnString,
	}

	// 设置证书的 DNS 名称
	dnsName := "HackThePlanet.org"
	dnsNames := []string{dnsName}

	// 设置证书的有效期
	notBefore := time.Now().AddDate(0, 0, -5) // 5天前
	notAfter := time.Now().AddDate(13, 3, 7)  //13年3个月7天后

	// 生成私钥
	ecpk, err := ecdsa.GenerateKey(elliptic.P521(), rand.Reader)
	if err != nil {
		t.Fatal("Couldn't generate EC key", err)
	}
}
	// 使用椭圆曲线私钥创建私钥对象
	pk := crypto.PrivateKey(ecpk)

	// 创建证书
	certSetVals, err := GenerateTLSCert(serial, &subj, dnsNames, &notBefore, &notAfter, pk, false)
	if err != nil {
		t.Fatal("Certificate generation[1] error:" + err.Error())
	}

	// 测试
	// 解析 X509 证书
	x5certSetVals, err := x509.ParseCertificate(certSetVals.Certificate[0])
	if err != nil {
		t.Fatal("Could not parse X509 certificate")
	}
	// 检查序列号
	if x5certSetVals.SerialNumber.Cmp(serial) != 0 {
		t.Error("Serial number mismatch")
	}
	// 检查主题
	if x5certSetVals.Subject.CommonName != cnString {
		t.Error("cn mismatch in subject: \n" + x5certSetVals.Subject.CommonName + "\n should be \n" + cnString)
	}
	}

	// 检查证书中的 DNSNames 是否符合预期
	if len(x5certSetVals.DNSNames) < 1 || x5certSetVals.DNSNames[0] != dnsName {
		// 如果 DNSNames 的长度小于1或者第一个元素不等于指定的 dnsName，则输出错误信息
		t.Error(fmt.Sprintf("dnsnames failed assignment: should be a length 1 string slice with the only "+
			"contents:\n%s\nbut is:\n%v", dnsName, x5certSetVals.DNSNames))
	}

	// 检查证书的有效期是否符合预期
	expectYear, expectMonth, expectDay := notBefore.Date()
	certYear, certMonth, certDay := x5certSetVals.NotBefore.Date()
	if expectYear != certYear || expectMonth != certMonth || expectDay != certDay {
		// 如果证书的有效期不符合预期，则输出错误信息
		t.Error(fmt.Errorf(
			"before date invalid:\nYear:%v (expected %v)\nMonth:%v (expected %v)\nDay:%v (expected %v)",
			certYear,
			expectYear,
			certMonth,
			expectMonth,
			certDay,
			expectDay,
```

		))
	}
	expectYear, expectMonth, expectDay = notAfter.Date()  # 获取证书过期日期的年、月、日
	certYear, certMonth, certDay = x5certSetVals.NotAfter.Date()  # 获取证书实际过期日期的年、月、日
	if expectYear != certYear || expectMonth != certMonth || expectDay != certDay:  # 检查证书实际过期日期是否与预期日期相符
		t.Error(fmt.Errorf(
			"after date invalid:\nYear:%v (expected %v)\nMonth:%v (expected %v)\nDay:%v (expected %v)",
			certYear,
			expectYear,
			certMonth,
			expectMonth,
			certDay,
			expectDay,
		))  # 如果日期不符合预期，输出错误信息
	}

	//privKey
	if certSetVals.PrivateKey.(*ecdsa.PrivateKey).Params().Name != "P-521" {  # 检查私钥的椭圆曲线名称是否为 "P-521"
		t.Error("Incorrect curve name: " + certSetVals.PrivateKey.(*ecdsa.PrivateKey).Params().Name)  # 如果椭圆曲线名称不符合预期，输出错误信息
	}
// TODO this should be its own test case
// 创建空的 x509 证书切片和 TLS 证书切片
x5Certs := []*x509.Certificate{}
tlsCerts := []*tls.Certificate{}
// 循环10次，生成随机的 TLS 证书，并添加到 tlsCerts 切片中
for i := 0; i < 10; i++ {
    certRand1, err := GenerateTLSCert(nil, nil, nil, nil, nil, nil, true) //making rsa to test enc/dec good
    if err != nil {
        t.Fatal("Certificate generation[2] error:" + err.Error())
    }
    tlsCerts = append(tlsCerts, certRand1)
    // 将 TLS 证书的第一个证书解析为 x509 证书，并添加到 x5Certs 切片中
    x5CertRand1, err := x509.ParseCertificate(certRand1.Certificate[0])
    if err != nil {
        t.Fatal("Certificate generation[5] error:" + err.Error())
    }
    x5Certs = append(x5Certs, x5CertRand1)
}

// 遍历 x5Certs 切片中的 x509 证书
for i, cer := range x5Certs {
    // 在这里执行 x509 证书的相关操作
}
		// 检查单独的数值

		// 测试生成的时间是否准确

		// timeBefore 必须在今天之前
		if cer.NotBefore.After(time.Now()) {
			t.Error("生成的时间不正确:", cer.NotBefore, "(今天是: ", time.Now(), ")")
		}
		// timeBefore 不能比一年前还要久
		if cer.NotBefore.Before(time.Now().AddDate(-1, 0, 0)) {
			t.Error("生成的时间太久远了: ", cer.NotBefore, time.Now())
		}
		// timeAfter 必须在 timeBefore 之后的两年
		certYear, certMonth, certDay := cer.NotBefore.Date()
		acertYear, acertMonth, acertDay := cer.NotAfter.Date()
		ecertYear, ecertMonth, ecertDay := cer.NotBefore.AddDate(2, 0, 0).Date()
		if acertYear != ecertYear || certDay != ecertDay || certMonth != ecertMonth {
			t.Error("生成的证书时间不一致。得到的时间为:", acertYear, acertMonth, acertDay,
				"期望的时间为:", certYear+2, certMonth, certDay)
		}
// 对比证书与其他证书
for ii, cer2 := range x5Certs {
    // 如果是同一个证书，则跳过
    if i == ii {
        continue
    }
    // 检查证书序列号是否不同
    if cer.SerialNumber.Cmp(cer2.SerialNumber) == 0 {
        t.Error(fmt.Errorf("生成的序列号相同：%d 和 %d：%v",
            i,
            ii,
            cer.SerialNumber.Int64()))
    }
}

// 获取第一个 TLS 证书的私钥
k1 := tlsCerts[0].PrivateKey.(*rsa.PrivateKey)
// 获取第一个 TLS 证书私钥对应的公钥
k1pub := k1.Public().(*rsa.PublicKey)
// 获取第二个 TLS 证书的私钥
k2 := tlsCerts[1].PrivateKey.(*rsa.PrivateKey)
	// 将 k2 转换为 rsa.PublicKey 类型，并赋值给 k2pub
	k2pub := k2.Public().(*rsa.PublicKey)

	// 测试证书的私钥/公钥是否正确（可用于加密/解密操作）
	// 创建两个待加密的消息
	plain1 := []byte("plainmessage1")
	plain2 := []byte("plainmsg2")

	// 使用 RSA 公钥加密消息 plain1
	ct1, err := rsa.EncryptOAEP(sha256.New(), rand.Reader, k1pub, plain1, []byte{})
	if err != nil {
		t.Error("Error during encrypt/decrypt verification 1: ", err)
	}

	// 使用 RSA 公钥加密消息 plain2
	ct2, err := rsa.EncryptOAEP(sha256.New(), rand.Reader, k2pub, plain2, []byte{})
	if err != nil {
		t.Error("Error during encrypt/decrypt verification 2: ", err)
	}

	// 使用 RSA 私钥解密消息 ct1
	dec1, err := rsa.DecryptOAEP(sha256.New(), rand.Reader, k1, ct1, []byte{})
	if err != nil {
		t.Error("Error during encrypt/decrypt verification 3: ", err)
	}
	// 使用 RSA 的 OAEP 模式进行解密，使用 SHA256 哈希算法，从 k2 和 ct2 解密得到明文 dec2
	dec2, err := rsa.DecryptOAEP(sha256.New(), rand.Reader, k2, ct2, []byte{})
	// 如果解密过程中出现错误，输出错误信息
	if err != nil {
		t.Error("Error during encrypt/decrypt verification 4: ", err)
	}

	// 检查解密后的值是否与原始明文匹配，如果不匹配，输出错误信息
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
}
```