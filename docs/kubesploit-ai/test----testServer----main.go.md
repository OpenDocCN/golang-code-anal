# `kubesploit\test\testServer\main.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd.

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 由自由软件基金会发布，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

// 包声明
package testserver
# 导入标准库中的模块
import (
	// 标准库
	"bytes"  # 用于操作字节的模块
	"crypto/rand"  # 用于生成随机数的加密安全随机数生成器
	"crypto/rsa"  # 提供了 RSA 算法的实现
	"crypto/sha256"  # 提供了 SHA-256 哈希算法的实现
	"crypto/tls"  # 提供了 TLS 协议的实现
	"crypto/x509"  # 提供了 X.509 证书的解析和生成
	"crypto/x509/pkix"  # 提供了 X.509 证书的 PKIX 数据结构
	"encoding/gob"  # 用于编码和解码数据的包
	"fmt"  # 用于格式化输出的包
	"io/ioutil"  # 用于读取文件内容的包
	"log"  # 用于记录日志的包
	"math/big"  # 提供了大整数的算术运算
	"net"  # 提供了网络操作的支持
	"net/http"  # 提供了 HTTP 客户端和服务器的实现
	"strings"  # 提供了对字符串的操作
	"testing"  # 提供了单元测试的支持
	"time"  # 提供了时间操作的支持
// 导入第三方库
"github.com/cretz/gopaque/gopaque"  // 导入名为 gopaque 的包
"github.com/satori/go.uuid"  // 导入名为 uuid 的包
"gopkg.in/square/go-jose.v2"  // 导入名为 jose 的包
"gopkg.in/square/go-jose.v2/jwt"  // 导入名为 jwt 的包

// 导入 Merlin 库
"kubesploit/pkg/agents"  // 导入名为 agents 的包
"kubesploit/pkg/core"  // 导入名为 core 的包
"kubesploit/pkg/handlers"  // 导入名为 handlers 的包
"kubesploit/pkg/messages"  // 导入名为 messages 的包
)

// 定义 TestServer 结构体的 handler 方法
func (ts *TestServer) handler(w http.ResponseWriter, r *http.Request) {
    // 如果请求方法为 GET 并且请求 URI 为 "/isup"，则返回状态码 200
    if r.Method == http.MethodGet && r.RequestURI == "/isup" {
        w.WriteHeader(200)
        return
    }

    // 使用 Merlin HTTP 处理程序
// 如果请求的 URI 是 "/merlin"，则调用 AgentHTTP 方法处理请求
if r.RequestURI == "/merlin" {
    ts.ctx.AgentHTTP(w, r)
}

// 确保消息中包含 JWT
token := r.Header.Get("Authorization")
if token == "" {
    // 如果没有 JWT，返回 404 错误
    w.WriteHeader(404)
    return
}

// 读取请求消息直到 EOF
requestBytes, errRequestBytes := ioutil.ReadAll(r.Body)
if errRequestBytes != nil {
    // 如果读取请求消息出错，打印错误信息并返回 500 错误
    fmt.Printf("there was an error reading the request message:\r\n%s\r\n", errRequestBytes.Error())
    w.WriteHeader(500)
    return
}

// 解码 gob 格式为 JWE 字符串
var jweString string
	// 使用gob解码requestBytes中的数据，并将结果存储到jweString中
	errDecode := gob.NewDecoder(bytes.NewReader(requestBytes)).Decode(&jweString)
	// 如果解码过程中出现错误，打印错误信息并返回500状态码
	if errDecode != nil {
		fmt.Printf("there was an error decoding the message from gob to JWE string:\r\n%s\r\n", errDecode.Error())
		w.WriteHeader(500)
		return
	}

	// 计算字符串"test"的SHA256哈希值，并将结果存储到hashedKey中
	hashedKey := sha256.Sum256([]byte("test"))
	// 从hashedKey中获取前32个字节，作为加密密钥key
	key := hashedKey[:]

	// 使用validateJWT函数验证JWT，并获取其中的claims和agentID
	var agentID uuid.UUID
	var errValidate error
	// 调用validateJWT函数，将token和key作为参数传入，并将结果存储到agentID和errValidate中
	agentID, errValidate = validateJWT(strings.Split(token, " ")[1], []byte("xZF7fvaGD6p2yeLyf9i7O9gBBHk05B0u"))
	// 如果验证过程中出现错误，使用PSK接口再次验证JWT
	if errValidate != nil {
		// 重新计算字符串"test"的SHA256哈希值，并将结果存储到hashedKey中
		hashedKey := sha256.Sum256([]byte("test"))
		// 从hashedKey中获取前32个字节，作为加密密钥key
		key := hashedKey[:]
		// 再次调用validateJWT函数，将token和key作为参数传入，并将结果存储到agentID和errValidate中
		agentID, errValidate = validateJWT(strings.Split(token, " ")[1], key)
		// 如果验证出现错误，返回 404 错误
		if errValidate != nil {
			w.WriteHeader(404)
			return
		}
	}

	// 如果代理的加密密钥长度大于 0，获取加密密钥
	if len(agents.GetEncryptionKey(agentID)) > 0 {
		key = agents.GetEncryptionKey(agentID)
	}

	// 解密 JWE
	j, errDecryptPSK := decryptJWE(jweString, key)
	// 如果解密出现错误，打印错误信息并返回 500 错误
	if errDecryptPSK != nil {
		fmt.Printf("there was an error decrypting the JWE on the server:\r\n%s\r\n", errDecryptPSK.Error())
		w.WriteHeader(500)
		return
	}

	// 如果需要打印出测试服务器接收到的请求信息，取消注释下面的代码
	//fmt.Println(fmt.Sprintf("Request: %+v\nBody:%+v", r, j)) //uncomment here if you want to print out exactly what the test server receives
// 声明变量returnMessage为messages.Base类型，声明变量err为error类型
var returnMessage messages.Base
var err error

// 根据用户代理进行不同的操作
switch r.UserAgent() {
case "invalidMessageBaseType":
    // 设置returnMessage的Type属性为"Test"
    returnMessage.Type = "Test"
}

// 根据消息类型进行不同的操作
switch j.Type {
case "AgentInfo":
    // 调用agents.UpdateInfo(j)方法，并将返回的错误赋值给err
    err = agents.UpdateInfo(j)
case "AuthInit":
    // 调用agents.OPAQUEAuthenticateInit(j)方法，并将返回的消息和错误赋值给returnMessage和err
    returnMessage, err = agents.OPAQUEAuthenticateInit(j)
case "AuthComplete":
    // 调用agents.OPAQUEAuthenticateComplete(j)方法，并将返回的消息和错误赋值给returnMessage和err
    returnMessage, err = agents.OPAQUEAuthenticateComplete(j)
case "BadPayload":
    // 设置响应头的Content-Type为"application/octet-stream"
    w.Header().Set("Content-Type", "application/octet-stream")
    // 创建一个gob编码器，将[]byte("Hack the planet!")编码后写入响应体，并将错误赋值给errBadPayload
    errBadPayload := gob.NewEncoder(w).Encode([]byte("Hack the planet!"))
}
		// 如果 errBadPayload 不为 nil，则打印错误信息
		if errBadPayload != nil {
			fmt.Println(errBadPayload.Error())
		}
	// 默认情况下，不执行任何操作
	default:

	}

	// 如果 err 不为 nil，则记录错误并退出程序
	if err != nil {
		log.Fatal(err)
	}
	// 将 returnMessage 的 ID 设置为 agentID
	returnMessage.ID = agentID
	// 将 messages.Base 编码为一个 gob
	returnMessageBytes := new(bytes.Buffer)
	errReturnMessageBytes := gob.NewEncoder(returnMessageBytes).Encode(returnMessage)
	// 如果编码过程中出现错误，则打印错误信息并返回 500 状态码
	if errReturnMessageBytes != nil {
		fmt.Printf("there was an error encoding the return message into a gob:\r\n%s\r\n", errReturnMessageBytes.Error())
		w.WriteHeader(500)
		return
	}
// 获取 JWE
// 使用给定的密钥对返回的消息进行对称加密，得到 JWE
jwe, errJWE := core.GetJWESymetric(returnMessageBytes.Bytes(), key[:])
// 如果加密过程中出现错误，打印错误信息并返回 500 状态码
if errJWE != nil {
    fmt.Printf("there was an error encrypting the message into a JWE:\r\n%s\r\n", errJWE.Error())
    w.WriteHeader(500)
    return
}

// 将 JWE 编码为 GOB 格式并发送给代理
w.Header().Set("Content-Type", "application/octet-stream")
errEncode := gob.NewEncoder(w).Encode(jwe)
// 如果编码过程中出现错误，打印错误信息并返回 500 状态码
if errEncode != nil {
    fmt.Printf("there was an error encoding the JWE into a gob:\r\n%s\r\n", errEncode.Error())
    w.WriteHeader(500)
    return
}

// TestServer 是一个 Web 服务器实例，用于便于发送 Web 请求进行代码的功能测试
type TestServer struct {
// 定义一个测试结构体和一个HTTP上下文结构体

// 由于tls/pki非常麻烦，因此每次都生成它们
func generateTLSConfig() *tls.Config {
	// 从https://golang.org/src/crypto/tls/generate_cert.go中大部分内容来自这里
	// 生成一个128位的序列号限制
	serialNumberLimit := new(big.Int).Lsh(big.NewInt(1), 128)
	// 生成一个随机的序列号
	serialNumber, err := rand.Int(rand.Reader, serialNumberLimit)
	if err != nil {
		log.Fatalf("failed to generate serial number: %s", err)
	}
	// 创建一个x509证书模板
	tpl := x509.Certificate{
		SerialNumber: serialNumber,
		Subject: pkix.Name{
			CommonName:   "127.0.0.1",
			Organization: []string{"Joey is the best hacker in Hackers"},
		},
		IPAddresses:           []net.IP{net.IPv4(127, 0, 0, 1)},
		DNSNames:              []string{"127.0.0.1", "localhost"},
		# 设置证书的生效时间为当前时间
		NotBefore:             time.Now(),
		# 设置证书的过期时间为当前时间后的20分钟
		NotAfter:              time.Now().Add(time.Minute * 20),
		# 设置证书的密钥用途，包括加密、数字签名和证书签名
		KeyUsage:              x509.KeyUsageKeyEncipherment | x509.KeyUsageDigitalSignature | x509.KeyUsageCertSign,
		# 设置证书的扩展密钥用途，包括服务器认证
		ExtKeyUsage:           []x509.ExtKeyUsage{x509.ExtKeyUsageServerAuth},
		# 设置证书的基本约束为有效
		BasicConstraintsValid: true,
	}
	# 生成 RSA 密钥对
	priv, err := rsa.GenerateKey(rand.Reader, 2048)
	# 如果生成密钥对出现错误，则抛出异常
	if err != nil {
		panic(err)
	}
	# 创建证书的字节流
	crtBytes, e := x509.CreateCertificate(rand.Reader, &tpl, &tpl, priv.Public(), priv)
	# 如果创建证书出现错误，则抛出异常
	if e != nil {
		panic(e)
	}

	# 创建 TLS 证书对象
	crt := tls.Certificate{
		# 设置证书的字节流
		Certificate: [][]byte{crtBytes},
		# 设置证书的私钥
		PrivateKey:  priv,
	}
	# 忽略安全扫描警告
	/* #nosec G402 */
// 返回一个配置了证书和协议的 TLS 配置对象
return &tls.Config{
    Certificates: []tls.Certificate{crt}, // 设置证书
    NextProtos:   []string{"h2", "hq"},   // 设置支持的协议
}

// Start 方法用于启动测试 HTTP 服务器，监听指定端口
func (TestServer) Start(port string, finishedTest, setup chan struct{}, t *testing.T) {
    s := http.NewServeMux() // 创建一个新的 ServeMux 对象
    ts := TestServer{        // 创建一个 TestServer 对象
        tes: t,              // 设置测试对象
    }
    s.HandleFunc("/", ts.handler) // 设置根路径的处理函数为 ts.handler
    srv := http.Server{}         // 创建一个 HTTP 服务器对象

    srv.TLSConfig = generateTLSConfig() // 设置服务器的 TLS 配置
    srv.Handler = s                     // 设置服务器的处理器为 s
}
# 设置服务器地址
srv.Addr = "127.0.0.1:" + port

# 创建 HTTP 上下文对象
ctx := handlers.HTTPContext{
    PSK:       "test",  # 设置预共享密钥
    JWTKey:    []byte(core.RandStringBytesMaskImprSrc(32)),  # 生成随机的 32 字节密钥
    OpaqueKey: gopaque.CryptoDefault.NewKey(nil),  # 使用默认的加密算法生成密钥
}
ts.ctx = ctx  # 将上下文对象赋值给某个变量

# 启动一个 goroutine，监听指定地址的 TCP 连接
go func() {
    ln, e := net.Listen("tcp", srv.Addr)  # 监听指定地址的 TCP 连接

    # 延迟执行关闭连接的操作
    defer func() {
        err := ln.Close()  # 关闭连接
        if err != nil {
            log.Fatal(err)  # 如果关闭连接出错，则记录日志并退出程序
        }
    }()

    if e != nil {
        panic(e)  # 如果监听出错，则抛出异常
		}
		// 创建一个基于TLS的监听器
		tlsListener := tls.NewListener(ln, srv.TLSConfig)
		// 使用TLS监听器来提供服务
		e = srv.Serve(tlsListener)
		// 如果出现错误，抛出异常
		if e != nil { //should be set by the tls config
			panic(e)
		}
	}()
	// 持续循环
	for {
		// 暂停1秒
		time.Sleep(time.Second * 1)
		/* #nosec G402 */
		// G402: TLS InsecureSkipVerify set true. (Confidence: HIGH, Severity: HIGH) Allowed for testing
		// 创建一个HTTP客户端，跳过TLS验证
		client := &http.Client{
			Transport: &http.Transport{
				TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
			},
		}
		// 发送GET请求到指定的URL
		resp, err := client.Get("https://localhost:" + port + "/isup")
		// 如果出现错误，继续循环
		if err != nil {
			continue
		}
		// 关闭响应体，释放资源
		errC := resp.Body.Close()
		// 如果关闭响应体出现错误，记录错误信息并终止程序
		if errC != nil {
			log.Fatalf("There was an error closing the body:\r\n%s", errC)
		}
		// 如果响应状态码不是 200，继续循环
		if resp.StatusCode != http.StatusOK {
			continue
		}
		// 到达此处：服务器已经正常运行
		break
	}

	// 关闭 setup 通道
	close(setup)
	// 等待 finishedTest 通道的消息，这是一个不太好的解决办法
	<-finishedTest //this is an ultra gross hack :(
}

// decryptJWE 使用提供的 JWE 字符串和代理密钥解密
func decryptJWE(jweString string, key []byte) (messages.Base, error) {
	var m messages.Base

	// 将 JWE 字符串解析为 JSONWebEncryption
// 解析加密的 JWE 字符串，返回解析后的对象和可能的错误
jwe, errObject := jose.ParseEncrypted(jweString)
// 如果解析出错，返回错误信息
if errObject != nil {
    return m, fmt.Errorf("there was an error parseing the JWE string into a JSONWebEncryption object:\r\n%s", errObject)
}

// 解密 JWE
jweMessage, errDecrypt := jwe.Decrypt(key)
// 如果解密出错，返回错误信息
if errDecrypt != nil {
    return m, fmt.Errorf("there was an error decrypting the JWE:\r\n%s", errDecrypt.Error())
}

// 将 JWE 载荷解码为 messages.Base 结构
errDecode := gob.NewDecoder(bytes.NewReader(jweMessage)).Decode(&m)
// 如果解码出错，返回错误信息
if errDecode != nil {
    return m, fmt.Errorf("there was an error decoding JWE payload message sent by an agent:\r\n%s", errDecode.Error())
}

// 返回解码后的消息和空错误
return m, nil
// validateJWT 验证提供的 JSON Web Token
func validateJWT(agentJWT string, key []byte) (uuid.UUID, error) {
    var agentID uuid.UUID

    // 创建一个空的声明结构体
    claims := jwt.Claims{}

    // 解析以确保它是一个有效的 JWT
    nestedToken, err := jwt.ParseSignedAndEncrypted(agentJWT)
    if err != nil {
        return agentID, fmt.Errorf("解析 JWT 时出错:\r\n%s", err.Error())
    }

    // 解密 JWT
    token, errToken := nestedToken.Decrypt(key)
    if errToken != nil {
        return agentID, fmt.Errorf("解密 JWT 时出错:\r\n%s", errToken.Error())
    }

    // 反序列化声明并验证签名
    errClaims := token.Claims(key, &claims)
	// 如果 JWT 的声明出现错误，则返回错误信息
	if errClaims != nil {
		return agentID, fmt.Errorf("there was an deserializing the JWT claims:\r\n%s", errClaims.Error())
	}

	// 验证声明的有效性
	errValidate := claims.Validate(jwt.Expected{
		Time: time.Now(),
	})
	// 如果验证失败，则返回错误信息
	if errValidate != nil {
		return agentID, errValidate
	}
	// 从声明中获取代理ID
	agentID = uuid.FromStringOrNil(claims.ID)
	// 返回代理ID和空错误
	return agentID, nil
}
```