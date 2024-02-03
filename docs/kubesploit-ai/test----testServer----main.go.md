# `kubesploit\test\testServer\main.go`

```go
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 由自由软件基金会发布，无论是许可证的第3版还是任何以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

package testserver

import (
    // 标准库
    "bytes"
    "crypto/rand"
    "crypto/rsa"
    "crypto/sha256"
    "crypto/tls"
    "crypto/x509"
    "crypto/x509/pkix"
    "encoding/gob"
    "fmt"
    "io/ioutil"
    "log"
    "math/big"
    "net"
    "net/http"
    "strings"
    "testing"
    "time"

    // 第三方库
    "github.com/cretz/gopaque/gopaque"
    "github.com/satori/go.uuid"
    "gopkg.in/square/go-jose.v2"
    "gopkg.in/square/go-jose.v2/jwt"

    // Merlin
    "kubesploit/pkg/agents"
    "kubesploit/pkg/core"
    "kubesploit/pkg/handlers"
    "kubesploit/pkg/messages"
)

func (ts *TestServer) handler(w http.ResponseWriter, r *http.Request) {
    if r.Method == http.MethodGet && r.RequestURI == "/isup" {
        w.WriteHeader(200)
        return
    }

    // 使用Merlin HTTP处理程序
    if r.RequestURI == "/merlin" {
        ts.ctx.AgentHTTP(w, r)
    }

    // 确保消息有JWT
    token := r.Header.Get("Authorization")
    if token == "" {
        w.WriteHeader(404)
        return
    }
    // 读取请求消息直到文件结束
    requestBytes, errRequestBytes := ioutil.ReadAll(r.Body)
    if errRequestBytes != nil {
        fmt.Printf("读取请求消息时出错:\r\n%s\r\n", errRequestBytes.Error())
        w.WriteHeader(500)
        return
    }

    // 解码 gob 数据为 JWE 字符串
    var jweString string
    errDecode := gob.NewDecoder(bytes.NewReader(requestBytes)).Decode(&jweString)
    if errDecode != nil {
        fmt.Printf("从 gob 解码消息为 JWE 字符串时出错:\r\n%s\r\n", errDecode.Error())
        w.WriteHeader(500)
        return
    }

    // 验证 JWT 并获取声明
    var agentID uuid.UUID
    var errValidate error

    hashedKey := sha256.Sum256([]byte("test"))
    key := hashedKey[:]

    agentID, errValidate = validateJWT(strings.Split(token, " ")[1], []byte("xZF7fvaGD6p2yeLyf9i7O9gBBHk05B0u"))
    if errValidate != nil {
        // 使用 PSK 验证 JWT；用于未经身份验证的代理
        hashedKey := sha256.Sum256([]byte("test"))
        key := hashedKey[:]
        agentID, errValidate = validateJWT(strings.Split(token, " ")[1], key)
        if errValidate != nil {
            w.WriteHeader(404)
            return
        }
    }

    if len(agents.GetEncryptionKey(agentID)) > 0 {
        key = agents.GetEncryptionKey(agentID)
    }

    // 解密 JWE
    j, errDecryptPSK := decryptJWE(jweString, key)
    if errDecryptPSK != nil {
        fmt.Printf("服务器解密 JWE 时出错:\r\n%s\r\n", errDecryptPSK.Error())
        w.WriteHeader(500)
        return
    }

    //fmt.Println(fmt.Sprintf("Request: %+v\nBody:%+v", r, j)) // 如果想打印出测试服务器接收到的内容，取消注释此处

    var returnMessage messages.Base
    var err error

    // 基于用户代理的操作
    switch r.UserAgent() {
    case "invalidMessageBaseType":
        returnMessage.Type = "Test"
    }

    // 基于消息类型的操作
    switch j.Type {
    // 根据不同的 case 进行不同的操作
    case "AgentInfo":
        // 更新代理信息
        err = agents.UpdateInfo(j)
    case "AuthInit":
        // 进行认证初始化，并返回消息和可能的错误
        returnMessage, err = agents.OPAQUEAuthenticateInit(j)
    case "AuthComplete":
        // 完成认证，并返回消息和可能的错误
        returnMessage, err = agents.OPAQUEAuthenticateComplete(j)
    case "BadPayload":
        // 设置响应头的内容类型为 application/octet-stream
        w.Header().Set("Content-Type", "application/octet-stream")
        // 使用 GOB 编码器将消息编码为字节流，并写入响应
        errBadPayload := gob.NewEncoder(w).Encode([]byte("Hack the planet!"))
        // 如果编码过程中出现错误，则打印错误信息
        if errBadPayload != nil {
            fmt.Println(errBadPayload.Error())
        }
    default:
        // 默认情况下不执行任何操作
    }

    // 如果在执行 case 语句中出现错误，则记录错误并退出程序
    if err != nil {
        log.Fatal(err)
    }
    // 设置返回消息的 ID 为代理 ID
    returnMessage.ID = agentID
    // 将返回消息编码为 GOB 格式的字节流
    returnMessageBytes := new(bytes.Buffer)
    errReturnMessageBytes := gob.NewEncoder(returnMessageBytes).Encode(returnMessage)
    // 如果编码过程中出现错误，则打印错误信息并返回 500 状态码
    if errReturnMessageBytes != nil {
        fmt.Printf("there was an error encoding the return message into a gob:\r\n%s\r\n", errReturnMessageBytes.Error())
        w.WriteHeader(500)
        return
    }

    // 获取 JWE
    jwe, errJWE := core.GetJWESymetric(returnMessageBytes.Bytes(), key[:])
    // 如果获取 JWE 过程中出现错误，则打印错误信息并返回 500 状态码
    if errJWE != nil {
        fmt.Printf("there was an error encrypting the message into a JWE:\r\n%s\r\n", errJWE.Error())
        w.WriteHeader(500)
        return
    }

    // 设置响应头的内容类型为 application/octet-stream
    w.Header().Set("Content-Type", "application/octet-stream")
    // 使用 GOB 编码器将 JWE 编码为字节流，并写入响应
    errEncode := gob.NewEncoder(w).Encode(jwe)
    // 如果编码过程中出现错误，则打印错误信息并返回 500 状态码
    if errEncode != nil {
        fmt.Printf("there was an error encoding the JWE into a gob:\r\n%s\r\n", errEncode.Error())
        w.WriteHeader(500)
        return
    }
}

// TestServer是一个Web服务器实例，用于便利功能测试需要发送Web请求的代码
type TestServer struct {
    tes *testing.T
    ctx handlers.HTTPContext
}

// 由于tls/pki非常麻烦，因此每次都要生成它们
func generateTLSConfig() *tls.Config {
    // https://golang.org/src/crypto/tls/generate_cert.go 大部分代码来自这里
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
        NotBefore:             time.Now(),
        NotAfter:              time.Now().Add(time.Minute * 20),
        KeyUsage:              x509.KeyUsageKeyEncipherment | x509.KeyUsageDigitalSignature | x509.KeyUsageCertSign,
        ExtKeyUsage:           []x509.ExtKeyUsage{x509.ExtKeyUsageServerAuth},
        BasicConstraintsValid: true,
    }
    // 生成RSA私钥
    priv, err := rsa.GenerateKey(rand.Reader, 2048)
    if err != nil {
        panic(err)
    }
    // 创建证书
    crtBytes, e := x509.CreateCertificate(rand.Reader, &tpl, &tpl, priv.Public(), priv)
    if e != nil {
        panic(e)
    }

    // 创建TLS证书
    crt := tls.Certificate{
        Certificate: [][]byte{crtBytes},
        PrivateKey:  priv,
    }
    /* #nosec G402 */
    // G402: TLS InsecureSkipVerify set true. (Confidence: HIGH, Severity: HIGH) Allowed for testing
    // G402 (CWE-295): TLS MinVersion too low. (Confidence: HIGH, Severity: HIGH)
    // TLS version is not configured to facilitate dynamic JA3 configurations
    # 返回一个包含TLS配置的对象
    return &tls.Config{
        # 设置证书列表，包含一个TLS证书
        Certificates: []tls.Certificate{crt},
        # 设置下一个协议列表，包含"h2"和"hq"
        NextProtos:   []string{"h2", "hq"},
    }
// Start函数在输入端口上启动测试HTTP服务器
func (TestServer) Start(port string, finishedTest, setup chan struct{}, t *testing.T) {
    // 创建一个新的 ServeMux 实例
    s := http.NewServeMux()
    // 创建一个TestServer实例
    ts := TestServer{
        tes: t,
    }
    // 将TestServer实例的handler方法注册为根路径的处理函数
    s.HandleFunc("/", ts.handler)
    // 创建一个http.Server实例
    srv := http.Server{}

    // 生成TLS配置并设置给srv
    srv.TLSConfig = generateTLSConfig()
    // 设置srv的处理器为s
    srv.Handler = s
    // 设置srv的地址
    srv.Addr = "127.0.0.1:" + port

    // 创建一个HTTP上下文
    ctx := handlers.HTTPContext{
        PSK:       "test",
        JWTKey:    []byte(core.RandStringBytesMaskImprSrc(32)),
        OpaqueKey: gopaque.CryptoDefault.NewKey(nil),
    }
    // 将HTTP上下文设置给TestServer实例
    ts.ctx = ctx
    // 启动goroutine来监听端口并提供服务
    go func() {
        ln, e := net.Listen("tcp", srv.Addr)

        // 延迟关闭监听器
        defer func() {
            err := ln.Close()
            if err != nil {
                log.Fatal(err)
            }
        }()

        if e != nil {
            panic(e)
        }
        // 创建TLS监听器
        tlsListener := tls.NewListener(ln, srv.TLSConfig)
        // 使用srv提供服务
        e = srv.Serve(tlsListener)
        if e != nil { //应该由TLS配置设置
            panic(e)
        }
    }()
    // 循环检查服务器是否启动
    for {
        time.Sleep(time.Second * 1)
        /* #nosec G402 */
        // G402: TLS InsecureSkipVerify set true. (Confidence: HIGH, Severity: HIGH) Allowed for testing
        // 创建一个带有InsecureSkipVerify的http.Client实例
        client := &http.Client{
            Transport: &http.Transport{
                TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
            },
        }
        // 发送GET请求到服务器的/isup路径
        resp, err := client.Get("https://localhost:" + port + "/isup")
        if err != nil {
            continue
        }
        // 关闭响应体
        errC := resp.Body.Close()
        if errC != nil {
            log.Fatalf("There was an error closing the body:\r\n%s", errC)
        }
        // 如果响应状态码不是200，则继续循环
        if resp.StatusCode != http.StatusOK {
            continue
        }
        // 到达此处：服务器已经启动运行！
        break
    }

    // 关闭setup通道
    close(setup)
    // 等待finishedTest通道关闭
    <-finishedTest //这是一个非常粗糙的hack :(
}

// decryptJWE接收提供的JWE字符串并使用每个代理密钥解密它
// decryptJWE函数用于解密JWE字符串，并返回解密后的消息和可能的错误
func decryptJWE(jweString string, key []byte) (messages.Base, error) {
    var m messages.Base

    // 将JWE字符串解析为JSONWebEncryption对象
    jwe, errObject := jose.ParseEncrypted(jweString)
    if errObject != nil {
        return m, fmt.Errorf("there was an error parseing the JWE string into a JSONWebEncryption object:\r\n%s", errObject)
    }

    // 解密JWE
    jweMessage, errDecrypt := jwe.Decrypt(key)
    if errDecrypt != nil {
        return m, fmt.Errorf("there was an error decrypting the JWE:\r\n%s", errDecrypt.Error())
    }

    // 将JWE有效载荷解码为messages.Base结构
    errDecode := gob.NewDecoder(bytes.NewReader(jweMessage)).Decode(&m)
    if errDecode != nil {
        return m, fmt.Errorf("there was an error decoding JWE payload message sent by an agent:\r\n%s", errDecode.Error())
    }

    return m, nil
}

// validateJWT函数用于验证提供的JSON Web Token
func validateJWT(agentJWT string, key []byte) (uuid.UUID, error) {
    var agentID uuid.UUID

    claims := jwt.Claims{}

    // 解析以确保它是有效的JWT
    nestedToken, err := jwt.ParseSignedAndEncrypted(agentJWT)
    if err != nil {
        return agentID, fmt.Errorf("there was an error parsing the JWT:\r\n%s", err.Error())
    }

    // 解密JWT
    token, errToken := nestedToken.Decrypt(key)
    if errToken != nil {
        return agentID, fmt.Errorf("there was an error decrypting the JWT:\r\n%s", errToken.Error())
    }

    // 反序列化声明并验证签名
    errClaims := token.Claims(key, &claims)
    if errClaims != nil {
        return agentID, fmt.Errorf("there was an deserializing the JWT claims:\r\n%s", errClaims.Error())
    }

    // 验证声明
    errValidate := claims.Validate(jwt.Expected{
        Time: time.Now(),
    })
    if errValidate != nil {
        return agentID, errValidate
    }
    agentID = uuid.FromStringOrNil(claims.ID)
    return agentID, nil
}
```