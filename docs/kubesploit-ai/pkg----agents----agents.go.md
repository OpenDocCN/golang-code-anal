# `kubesploit\pkg\agents\agents.go`

```go
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是任何以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。有关更多详细信息，请参见GNU通用公共许可证。

// 您应该已经收到了GNU通用公共许可证的副本。
// 如果没有，请参见<http://www.gnu.org/licenses/>。

package agents

import (
    // 标准库
    "bytes"
    "crypto/rand"
    "crypto/rsa"
    "crypto/sha256"
    "encoding/base64"
    "errors"
    "fmt"
    "io"
    "io/ioutil"
    "os"
    "path"
    "path/filepath"
    "strconv"
    "strings"
    "time"

    // 第三方库
    "github.com/cretz/gopaque/gopaque"
    "github.com/fatih/color"
    "github.com/olekukonko/tablewriter"
    "github.com/satori/go.uuid"
    "go.dedis.ch/kyber/v3"

    // Merlin
    messageAPI "kubesploit/pkg/api/messages"
    "kubesploit/pkg/core"
    "kubesploit/pkg/logging"
    "kubesploit/pkg/messages"
)

// 全局变量

// Agents包含所有被其他模块访问的实例化代理对象
var Agents = make(map[uuid.UUID]*agent)

type agent struct {
    ID               uuid.UUID
    Platform         string
    Architecture     string
    UserName         string
    UserGUID         string
    HostName         string
    Ips              []string
    Pid              int
    agentLog         *os.File
    # 用于传递 Job 结构体的通道
    channel          chan []Job
    # 初始检查时间
    InitialCheckIn   time.Time
    # 状态检查时间
    StatusCheckIn    time.Time
    # 版本号
    Version          string
    # 构建版本号
    Build            string
    # 等待时间
    WaitTime         string
    # 填充最大值
    PaddingMax       int
    # 最大重试次数
    MaxRetry         int
    # 失败检查次数
    FailedCheckin    int
    # 时间偏移
    Skew             int64
    # 协议类型
    Proto            string
    # 终止日期
    KillDate         int64
    # RSA 私钥
    RSAKeys          *rsa.PrivateKey                // RSA Private/Public key pair; Private key used to decrypt messages
    # RSA 公钥
    PublicKey        rsa.PublicKey                  // Public key used to encrypt messages
    # 用于执行对称加密操作的密钥
    secret           []byte                         // secret is used to perform symmetric encryption operations
    # OPAQUE 服务器认证信息
    OPAQUEServerAuth gopaque.ServerAuth             // OPAQUE Server Authentication information used to derive shared secret
    # OPAQUE 服务器注册信息
    OPAQUEServerReg  gopaque.ServerRegister         // OPAQUE server registration information
    # OPAQUE 记录
    OPAQUERecord     gopaque.ServerRegisterComplete // Holds the OPAQUE kU, EnvU, PrivS, PubU
    # 代理的 JA3 签名
    JA3              string                         // The JA3 signature applied to the agent's TLS client
// KeyExchange函数用于在服务器和代理之间交换公钥
func KeyExchange(m messages.Base) (messages.Base, error) {
    // 如果调试模式开启，则输出进入agents.KeyExchange函数的调试信息
    if core.Debug {
        message("debug", "Entering into agents.KeyExchange function")
    }

    // 创建服务器密钥消息
    serverKeyMessage := messages.Base{
        ID:      m.ID,
        Version: 1.0,
        Type:    "KeyExchange",
        Padding: core.RandStringBytesMaskImprSrc(4096),
    }

    // 确保代理已经进行过身份验证
    if !isAgent(m.ID) {
        return serverKeyMessage, fmt.Errorf("the agent does not exist")
    }

    // 记录接收到的新代理密钥交换
    logging.Server(fmt.Sprintf("Received new agent key exchange from %s", m.ID))

    // 获取密钥交换的数据
    ke := m.Payload.(messages.KeyExchange)

    // 如果调试模式开启，则输出接收到的来自代理的新公钥信息
    if core.Debug {
        message("debug", fmt.Sprintf("Received new public key from %s:\r\n%v", m.ID, ke.PublicKey))
    }

    // 更新服务器密钥消息的ID和Payload
    serverKeyMessage.ID = Agents[m.ID].ID
    Agents[m.ID].PublicKey = ke.PublicKey

    // 生成密钥对
    privateKey, rsaErr := rsa.GenerateKey(rand.Reader, 4096)
    if rsaErr != nil {
        return serverKeyMessage, fmt.Errorf("there was an error generating the RSA key pair:\r\n%s", rsaErr.Error())
    }

    // 更新代理的RSA密钥
    Agents[m.ID].RSAKeys = privateKey

    // 如果调试模式开启，则输出服务器的公钥信息
    if core.Debug {
        message("debug", fmt.Sprintf("Server's Public Key: %v", Agents[m.ID].RSAKeys.PublicKey))
    }

    // 创建密钥交换消息
    pk := messages.KeyExchange{
        PublicKey: Agents[m.ID].RSAKeys.PublicKey,
    }

    // 更新服务器密钥消息的ID和Payload
    serverKeyMessage.ID = m.ID
    serverKeyMessage.Payload = pk

    // 如果调试模式开启，则输出离开agents.KeyExchange函数的调试信息和服务器密钥消息的内容
    if core.Debug {
        message("debug", "Leaving agents.KeyExchange returning without error")
        message("debug", fmt.Sprintf("serverKeyMessage: %v", serverKeyMessage))
    }
    // 返回服务器密钥消息和无错误
    return serverKeyMessage, nil
}

// OPAQUERegistrationInit函数用于使用OPAQUE密码认证密钥交换（PAKE）协议注册代理
func OPAQUERegistrationInit(m messages.Base, opaqueServerKey kyber.Scalar) (messages.Base, error) {
    // 如果调试模式开启，输出进入 agents.OPAQUERegistrationInit 函数的调试信息
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

    // 检查该代理是否已经在服务器上注册
    if isAgent(m.ID) {
        return returnMessage, fmt.Errorf("the %s agent has already been registered", m.ID.String())
    }

    // 创建服务器注册对象和用户注册初始化对象
    serverReg := gopaque.NewServerRegister(gopaque.CryptoDefault, opaqueServerKey)
    var userRegInit gopaque.UserRegisterInit

    // 从字节流中解析 OPAQUE 用户注册初始化消息
    errUserRegInit := userRegInit.FromBytes(gopaque.CryptoDefault, m.Payload.([]byte))
    if errUserRegInit != nil {
        return returnMessage, fmt.Errorf("there was an error unmarshalling the OPAQUE user register initialization message from bytes:\r\n%s", errUserRegInit.Error())
    }

    // 检查 OPAQUE 用户ID 是否与 Merlin 消息ID匹配
    if !bytes.Equal(userRegInit.UserID, m.ID.Bytes()) {
        if core.Verbose {
            message("note", fmt.Sprintf("OPAQUE UserID: %v", userRegInit.UserID))
            message("note", fmt.Sprintf("Merlin Message UserID: %v", m.ID.Bytes()))
        }
        return returnMessage, errors.New("the OPAQUE UserID doesn't match the Merlin message ID")
    }

    // 初始化服务器注册对象
    serverRegInit := serverReg.Init(&userRegInit)

    // 将服务器注册初始化消息转换为字节流
    serverRegInitBytes, errServerRegInitBytes := serverRegInit.ToBytes()
    if errServerRegInitBytes != nil {
        return returnMessage, fmt.Errorf("there was an error marshalling the OPAQUE server registration initialization message to bytes:\r\n%s", errServerRegInitBytes.Error())
    }

    // 将服务器注册初始化消息添加到返回消息的载荷中
    returnMessage.Payload = serverRegInitBytes

    // 创建新的代理并将其添加到全局映射中
    agent, agentErr := newAgent(m.ID)
    if agentErr != nil {
        return returnMessage, fmt.Errorf("there was an error creating a new agent instance for %s:\r\n%s", m.ID.String(), agentErr.Error())
    }
    agent.OPAQUEServerReg = *serverReg

    // 将代理添加到全局映射中
    # 将 agent 对象存储到 Agents 字典中，键为 m.ID
    Agents[m.ID] = &agent

    # 记录日志，表示接收到了 agent 的 OPAQUE 注册初始化消息
    Log(m.ID, "Received agent OPAQUE register initialization message")

    # 如果处于调试模式，则输出调试信息
    if core.Debug {
        message("debug", "Leaving agents.OPAQUERegistrationInit function without error")
    }

    # 返回 returnMessage 和空错误
    return returnMessage, nil
// OPAQUERegistrationComplete用于完成OPAQUE用户注册并存储加密的信封EnvU
func OPAQUERegistrationComplete(m messages.Base) (messages.Base, error) {
    if core.Debug {
        message("debug", "Entering into agents.OPAQUERegistrationComplete function")
    }

    returnMessage := messages.Base{
        ID:      m.ID,
        Version: 1.0,
        Type:    "RegComplete",
        Padding: core.RandStringBytesMaskImprSrc(4096),
    }

    // 检查该代理是否已知于服务器
    if !isAgent(m.ID) {
        return returnMessage, fmt.Errorf("the %s agent has not completed OPAQUE user registration intialization", m.ID.String())
    }

    var userRegComplete gopaque.UserRegisterComplete

    errUserRegComplete := userRegComplete.FromBytes(gopaque.CryptoDefault, m.Payload.([]byte))
    if errUserRegComplete != nil {
        return returnMessage, fmt.Errorf("there was an error unmarshalling the OPAQUE user register complete message from bytes:\r\n%s", errUserRegComplete.Error())
    }

    Agents[m.ID].OPAQUERecord = *Agents[m.ID].OPAQUEServerReg.Complete(&userRegComplete)

    // 检查梅林UserID是否与OPAQUE UserID匹配
    if !bytes.Equal(m.ID.Bytes(), Agents[m.ID].OPAQUERecord.UserID) {
        return returnMessage, fmt.Errorf("the OPAQUE UserID: %v doesn't match the Merlin UserID: %v", Agents[m.ID].OPAQUERecord.UserID, m.ID.Bytes())
    }

    Log(m.ID, "OPAQUE registration complete")

    if core.Debug {
        message("debug", "Leaving agents.OPAQUERegistrationComplete function without error")
    }

    return returnMessage, nil
}

// OPAQUEAuthenticateInit用于使用OPAQUE密码认证密钥交换（PAKE）协议和预共享密钥对代理进行身份验证
func OPAQUEAuthenticateInit(m messages.Base) (messages.Base, error) {
    if core.Debug {
        message("debug", "Entering into agents.OPAQUEAuthenticateInit function")
    }
    // 创建一个返回消息对象，包含ID、版本、类型和填充数据
    returnMessage := messages.Base{
        ID:      m.ID,
        Version: 1.0,
        Type:    "AuthInit",
        Padding: core.RandStringBytesMaskImprSrc(4096),
    }

    // 检查服务器是否已知该代理
    if !isAgent(m.ID) {
        // 如果代理未知，则将返回消息的类型设置为 "ReRegister"，并返回消息和空错误
        returnMessage.Type = "ReRegister"
        return returnMessage, nil
    }

    // 1 - 接收用户的 UserAuthInit
    // 创建服务器端密钥交换对象和服务器端认证对象
    serverKex := gopaque.NewKeyExchangeSigma(gopaque.CryptoDefault)
    serverAuth := gopaque.NewServerAuth(gopaque.CryptoDefault, serverKex)
    Agents[m.ID].OPAQUEServerAuth = *serverAuth

    // 将接收到的用户初始化消息转换为 UserAuthInit 对象
    var userInit gopaque.UserAuthInit
    errFromBytes := userInit.FromBytes(gopaque.CryptoDefault, m.Payload.([]byte))
    if errFromBytes != nil {
        // 如果转换出错，则记录警告信息
        message("warn", fmt.Sprintf("there was an error unmarshalling the user init message from bytes:\r\n%s", errFromBytes.Error()))
    }

    // 完成服务器端认证
    serverAuthComplete, errServerAuthComplete := serverAuth.Complete(&userInit, &Agents[m.ID].OPAQUERecord)

    // 如果完成服务器端认证出错，则返回消息和错误
    if errServerAuthComplete != nil {
        return returnMessage, fmt.Errorf("there was an error completing the OPAQUE server authentication:\r\n%s", errServerAuthComplete.Error())
    }

    // 如果处于调试模式，则记录用户初始化消息和服务器端认证完成消息
    if core.Debug {
        message("debug", fmt.Sprintf("User Auth Init:\r\n%+v", userInit))
        message("debug", fmt.Sprintf("Server Auth Complete:\r\n%+v", serverAuthComplete))
    }

    // 将服务器端认证完成消息转换为字节流
    serverAuthCompleteBytes, errServerAuthCompleteBytes := serverAuthComplete.ToBytes()
    if errServerAuthCompleteBytes != nil {
        // 如果转换出错，则返回消息和错误
        return returnMessage, fmt.Errorf("there was an error marshalling the OPAQUE server authentication complete message to bytes:\r\n%s", errServerAuthCompleteBytes.Error())
    }

    // 将服务器端认证完成消息设置为返回消息的载荷
    returnMessage.Payload = serverAuthCompleteBytes
    // 将代理的密钥设置为服务器端密钥交换的共享密钥
    Agents[m.ID].secret = []byte(serverKex.SharedSecret.String())

    // 记录接收到新代理 OPAQUE 认证初始化消息的日志
    Log(m.ID, "Received new agent OPAQUE authentication initialization message")
    # 如果调试模式开启
    if core.Debug:
        # 输出调试信息：接收到新的 OPAQUE 认证代理，包括代理 ID 和时间
        message("debug", fmt.Sprintf("Received new agent OPAQUE authentication for %s at %s", m.ID, time.Now().UTC().Format(time.RFC3339)))
        # 输出调试信息：离开 agents.OPAQUEAuthenticateInit 函数，没有错误发生
        message("debug", "Leaving agents.OPAQUEAuthenticateInit function without error")
        # 输出调试信息：服务器 OPAQUE 密钥交换共享密钥
        message("debug", fmt.Sprintf("Server OPAQUE key exchange shared secret: %v", Agents[m.ID].secret))
    # 返回消息和空错误
    return returnMessage, nil
// OPAQUEAuthenticateComplete 用于接收 OPAQUE UserAuthComplete
func OPAQUEAuthenticateComplete(m messages.Base) (messages.Base, error) {
    if core.Debug {
        message("debug", "Entering into agents.OPAQUEAuthenticateComplete function")
    }

    returnMessage := messages.Base{
        ID:      m.ID,
        Version: 1.0,
        Type:    "ServerOk",
        Padding: core.RandStringBytesMaskImprSrc(4096),
    }

    // 检查该代理是否已知于服务器
    if !isAgent(m.ID) {
        return returnMessage, fmt.Errorf("%s is not a known agent", m.ID.String())
    }

    Log(m.ID, "Received agent OPAQUE authentication complete message")

    var userComplete gopaque.UserAuthComplete
    errFromBytes := userComplete.FromBytes(gopaque.CryptoDefault, m.Payload.([]byte))
    if errFromBytes != nil {
        message("warn", fmt.Sprintf("there was an error unmarshalling the user complete message from bytes:\r\n%s", errFromBytes.Error()))
    }

    // 服务器认证完成
    errAuthFinish := Agents[m.ID].OPAQUEServerAuth.Finish(&userComplete)
    if errAuthFinish != nil {
        message("warn", fmt.Sprintf("there was an error finishing authentication:\r\n%s", errAuthFinish.Error()))
    }

    //message("success", fmt.Sprintf("New authenticated agent checkin for %s at %s", m.ID.String(), time.Now().UTC().Format(time.RFC3339)))
    if core.Debug {
        message("debug", "Leaving agents.OPAQUEAuthenticateComplete function without error")
    }
    return returnMessage, nil
}

// OPAQUEReAuthenticate 用于当代理先前完成 OPAQUE 注册但需要重新认证时
func OPAQUEReAuthenticate(agentID uuid.UUID) (messages.Base, error) {
    if core.Debug {
        message("debug", "Entering into agents.OPAQUEReAuthenticate function")
    }
    # 创建一个消息对象，包含代理ID、版本、类型和填充数据
    returnMessage := messages.Base{
        ID:      agentID,
        Version: 1.0,
        Type:    "ReAuthenticate",
        Padding: core.RandStringBytesMaskImprSrc(4096),
    }

    # 检查代理是否已知于服务器
    if !isAgent(agentID) {
        # 如果代理未注册，返回错误消息
        return returnMessage, fmt.Errorf("the %s agent has not OPAQUE registered", agentID.String())
    }

    # 如果调试模式开启，输出调试信息
    if core.Debug {
        message("debug", "Leaving agents.OPAQUEReAuthenticate function without error")
    }
    # 记录日志，指示代理使用OPAQUE协议重新认证
    Log(agentID, "Instructing agent to re-authenticate with OPAQUE protocol")

    # 返回消息对象和空错误
    return returnMessage, nil
// GetEncryptionKey函数用于检索用于解密任何协议消息的每个代理有效载荷加密密钥
func GetEncryptionKey(agentID uuid.UUID) []byte {
    // 如果处于调试模式，则输出进入函数的调试信息
    if core.Debug {
        message("debug", "Entering into agents.GetEncryptionKey function")
    }
    var key []byte

    // 如果代理存在，则获取代理的密钥
    if isAgent(agentID) {
        key = Agents[agentID].secret
    }

    // 如果处于调试模式，则输出离开函数的调试信息
    if core.Debug {
        message("debug", "Leaving agents.GetEncryptionKey function")
    }
    // 返回密钥
    return key
}

// StatusCheckIn是当代理发送消息返回服务器时运行的函数，用于检查是否有额外指令
func StatusCheckIn(m messages.Base) (messages.Base, error) {
    // 检查代理UUID是否在数据集中
    _, ok := Agents[m.ID]
    if !ok {
        // 如果代理UUID不在数据集中，则输出警告信息，并创建一个初始化作业
        message("warn", fmt.Sprintf("Orphaned agent %s has checked in at %s. Instructing agent to re-initialize...",
            time.Now().UTC().Format(time.RFC3339), m.ID.String()))
        logging.Server(fmt.Sprintf("[Orphaned agent %s has checked in", m.ID.String()))
        job := Job{
            ID:      core.RandStringBytesMaskImprSrc(10),
            Type:    "initialize",
            Created: time.Now(),
            Status:  "created",
        }
        // 获取初始化作业的消息
        m, mErr := GetMessageForJob(m.ID, job)
        return m, mErr
    }

    // 记录代理状态检查
    Log(m.ID, "Agent status check in")
    // 如果处于详细模式，则输出成功的消息
    if core.Verbose {
        message("success", fmt.Sprintf("Received agent status checkin from %s", m.ID))
    }
    // 如果处于调试模式，则输出代理状态检查的详细信息
    if core.Debug {
        message("debug", fmt.Sprintf("Received agent status checkin from %s", m.ID))
        message("debug", fmt.Sprintf("Channel length: %d", len(Agents[m.ID].channel))
        message("debug", fmt.Sprintf("Channel content: %v", Agents[m.ID].channel))
    }

    // 更新代理的状态检查时间
    Agents[m.ID].StatusCheckIn = time.Now().UTC()
    // 检查是否有任何作业
}
    # 如果代理的通道长度大于等于1
    if len(Agents[m.ID].channel) >= 1 {
        # 从代理的通道中获取作业
        job := <-Agents[m.ID].channel
        # 如果调试模式开启
        if core.Debug {
            # 输出调试信息，显示通道命令字符串
            message("debug", fmt.Sprintf("Channel command string: %s", job))
            # 输出调试信息，显示代理命令类型
            message("debug", fmt.Sprintf("Agent command type: %s", job[0].Type))
        }

        # 获取作业消息
        m, mErr := GetMessageForJob(m.ID, job[0])
        # 返回消息和错误
        return m, mErr
    }
    # 创建返回消息对象
    returnMessage := messages.Base{
        Version: 1.0,
        ID:      m.ID,
        Type:    "ServerOk",
        Padding: core.RandStringBytesMaskImprSrc(Agents[m.ID].PaddingMax),
    }
    # 返回消息对象和空错误
    return returnMessage, nil
// UpdateInfo函数用于使用传入的消息数据更新代理的信息
func UpdateInfo(m messages.Base) error {
    // 如果调试模式开启，则输出进入agents.UpdateInfo函数的调试信息
    if core.Debug {
        message("debug", "Entering into agents.UpdateInfo function")
    }

    // 将消息载荷转换为AgentInfo类型
    p := m.Payload.(messages.AgentInfo)

    // 如果代理ID不存在，则输出警告信息并返回错误
    if !isAgent(m.ID) {
        message("warn", "The agent was not found while processing an AgentInfo message")
        return fmt.Errorf("%s is not a known agent", m.ID)
    }
    // 如果调试模式开启，则输出处理新代理信息的调试信息
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
    // 记录处理AgentInfo消息的日志信息
    Log(m.ID, "Processing AgentInfo message:")
    Log(m.ID, fmt.Sprintf("\tAgent Version: %s ", p.Version))
    Log(m.ID, fmt.Sprintf("\tAgent Build: %s ", p.Build))
    Log(m.ID, fmt.Sprintf("\tAgent waitTime: %s ", p.WaitTime))
    Log(m.ID, fmt.Sprintf("\tAgent skew: %d ", p.Skew))
    Log(m.ID, fmt.Sprintf("\tAgent paddingMax: %d ", p.PaddingMax))
    Log(m.ID, fmt.Sprintf("\tAgent maxRetry: %d ", p.MaxRetry))
    Log(m.ID, fmt.Sprintf("\tAgent failedCheckin: %d ", p.FailedCheckin))
    Log(m.ID, fmt.Sprintf("\tAgent proto: %s ", p.Proto))
    Log(m.ID, fmt.Sprintf("\tAgent KillDate: %s", time.Unix(p.KillDate, 0).UTC().Format(time.RFC3339)))
    Log(m.ID, fmt.Sprintf("\tAgent JA3 signature: %s", p.JA3))
}
    # 设置 Agents 字典中对应 ID 的 Agent 对象的 Version 属性
    Agents[m.ID].Version = p.Version
    # 设置 Agents 字典中对应 ID 的 Agent 对象的 Build 属性
    Agents[m.ID].Build = p.Build
    # 设置 Agents 字典中对应 ID 的 Agent 对象的 WaitTime 属性
    Agents[m.ID].WaitTime = p.WaitTime
    # 设置 Agents 字典中对应 ID 的 Agent 对象的 Skew 属性
    Agents[m.ID].Skew = p.Skew
    # 设置 Agents 字典中对应 ID 的 Agent 对象的 PaddingMax 属性
    Agents[m.ID].PaddingMax = p.PaddingMax
    # 设置 Agents 字典中对应 ID 的 Agent 对象的 MaxRetry 属性
    Agents[m.ID].MaxRetry = p.MaxRetry
    # 设置 Agents 字典中对应 ID 的 Agent 对象的 FailedCheckin 属性
    Agents[m.ID].FailedCheckin = p.FailedCheckin
    # 设置 Agents 字典中对应 ID 的 Agent 对象的 Proto 属性
    Agents[m.ID].Proto = p.Proto
    # 设置 Agents 字典中对应 ID 的 Agent 对象的 KillDate 属性
    Agents[m.ID].KillDate = p.KillDate
    # 设置 Agents 字典中对应 ID 的 Agent 对象的 JA3 属性
    Agents[m.ID].JA3 = p.JA3

    # 设置 Agents 字典中对应 ID 的 Agent 对象的 Architecture 属性
    Agents[m.ID].Architecture = p.SysInfo.Architecture
    # 设置 Agents 字典中对应 ID 的 Agent 对象的 HostName 属性
    Agents[m.ID].HostName = p.SysInfo.HostName
    # 设置 Agents 字典中对应 ID 的 Agent 对象的 Pid 属性
    Agents[m.ID].Pid = p.SysInfo.Pid
    # 设置 Agents 字典中对应 ID 的 Agent 对象的 Ips 属性
    Agents[m.ID].Ips = p.SysInfo.Ips
    # 设置 Agents 字典中对应 ID 的 Agent 对象的 Platform 属性
    Agents[m.ID].Platform = p.SysInfo.Platform
    # 设置 Agents 字典中对应 ID 的 Agent 对象的 UserName 属性
    Agents[m.ID].UserName = p.SysInfo.UserName
    # 设置 Agents 字典中对应 ID 的 Agent 对象的 UserGUID 属性
    Agents[m.ID].UserGUID = p.SysInfo.UserGUID

    # 如果处于调试模式，输出调试信息
    if core.Debug {
        message("debug", "Leaving agents.UpdateInfo function")
    }
    # 返回空值
    return nil
// Log函数用于向代理的日志文件写入日志消息
func Log(agentID uuid.UUID, logMessage string) {
    // 如果核心调试模式开启，则输出进入agents.Log的调试信息
    if core.Debug {
        message("debug", "Entering into agents.Log")
    }
    // 向代理的日志文件写入时间戳和日志消息
    _, err := Agents[agentID].agentLog.WriteString(fmt.Sprintf("[%s]%s\r\n", time.Now().UTC().Format(time.RFC3339), logMessage))
    // 如果写入出现错误，则输出警告信息
    if err != nil {
        message("warn", fmt.Sprintf("There was an error writing to the agent log agents.Log:\r\n%s", err.Error()))
    }
}

// GetAgentList函数返回存在的代理列表，用于命令行的自动补全
func GetAgentList() func(string) []string {
    return func(line string) []string {
        a := make([]string, 0)
        // 遍历代理列表，将代理ID转换为字符串并添加到列表中
        for k := range Agents {
            a = append(a, k.String())
        }
        return a
    }
}

// ShowInfo函数以表格形式列出指定代理的结构值
func ShowInfo(agentID uuid.UUID) {
    // 如果指定的代理ID不存在，则输出警告信息并返回
    if !isAgent(agentID) {
        message("warn", fmt.Sprintf("%s is not a valid agent!", agentID))
        return
    }
    // 创建一个新的表格写入器，并设置对齐方式为左对齐
    table := tablewriter.NewWriter(os.Stdout)
    table.SetAlignment(tablewriter.ALIGN_LEFT)
}
    // 创建一个二维字符串数组，包含代理状态和各种属性值
    data := [][]string{
        {"Status", GetAgentStatus(agentID)},  // 获取代理状态
        {"ID", Agents[agentID].ID.String()},  // 获取代理ID并转换为字符串
        {"Platform", Agents[agentID].Platform},  // 获取代理平台
        {"Architecture", Agents[agentID].Architecture},  // 获取代理架构
        {"UserName", Agents[agentID].UserName},  // 获取代理用户名
        {"User GUID", Agents[agentID].UserGUID},  // 获取代理用户GUID
        {"Hostname", Agents[agentID].HostName},  // 获取代理主机名
        {"Process ID", strconv.Itoa(Agents[agentID].Pid)},  // 获取代理进程ID并转换为字符串
        {"IP", fmt.Sprintf("%v", Agents[agentID].Ips)},  // 获取代理IP地址
        {"Initial Check In", Agents[agentID].InitialCheckIn.Format(time.RFC3339)},  // 获取代理初始检查时间并格式化为RFC3339格式
        {"Last Check In", Agents[agentID].StatusCheckIn.Format(time.RFC3339)},  // 获取代理最后检查时间并格式化为RFC3339格式
        {"Agent Version", Agents[agentID].Version},  // 获取代理版本
        {"Agent Build", Agents[agentID].Build},  // 获取代理构建版本
        {"Agent Wait Time", Agents[agentID].WaitTime},  // 获取代理等待时间
        {"Agent Wait Time Skew", strconv.FormatInt(Agents[agentID].Skew, 10)},  // 获取代理等待时间偏差并转换为字符串
        {"Agent Message Padding Max", strconv.Itoa(Agents[agentID].PaddingMax)},  // 获取代理消息填充最大值并转换为字符串
        {"Agent Max Retries", strconv.Itoa(Agents[agentID].MaxRetry)},  // 获取代理最大重试次数并转换为字符串
        {"Agent Failed Check In", strconv.Itoa(Agents[agentID].FailedCheckin)},  // 获取代理失败的检查次数并转换为字符串
        {"Agent Kill Date", time.Unix(Agents[agentID].KillDate, 0).UTC().Format(time.RFC3339)},  // 获取代理终止日期并格式化为RFC3339格式
        {"Agent Communication Protocol", Agents[agentID].Proto},  // 获取代理通信协议
        {"Agent JA3 TLS Client Signature", Agents[agentID].JA3},  // 获取代理JA3 TLS客户端签名
    }
    // 将数据批量添加到表格中
    table.AppendBulk(data)
    // 打印空行
    fmt.Println()
    // 渲染表格
    table.Render()
    // 打印空行
    fmt.Println()
// message 用于向所有连接的客户端发送广播消息
func message(level string, message string) {
    // 创建一个 UserMessage 结构体对象
    m := messageAPI.UserMessage{
        Message: message,
        Time:    time.Now().UTC(),
        Error:   false,
    }
    // 根据不同的级别设置消息的级别
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
        // 发送调试级别的消息
        message("debug", fmt.Sprintf("In agents.AddJob function for agent: %s", agentID.String()))
        message("debug", fmt.Sprintf("In agents.AddJob function for type: %s", jobType))
        message("debug", fmt.Sprintf("In agents.AddJob function for command: %s", jobArgs))
    }
}
    # 如果代理ID存在或者代理ID的字符串等于"ffffffff-ffff-ffff-ffff-ffffffffffff"
    if isAgent(agentID) || agentID.String() == "ffffffff-ffff-ffff-ffff-ffffffffffff" {
        # 创建一个作业对象
        job := Job{
            Type:    jobType,
            Status:  "created",
            Args:    jobArgs,
            Created: time.Now().UTC(),
        }

        # 如果代理ID的字符串等于"ffffffff-ffff-ffff-ffff-ffffffffffff"
        if agentID.String() == "ffffffff-ffff-ffff-ffff-ffffffffffff" {
            # 如果可用代理数量小于等于0，则返回错误
            if len(Agents) <= 0 {
                return "", errors.New("there are 0 available agents, no jobs were created")
            }
            # 遍历代理列表，为每个代理创建作业并发送
            for k := range Agents {
                s := Agents[k].channel
                job.ID = core.RandStringBytesMaskImprSrc(10)
                s <- []Job{job}
                Log(k, fmt.Sprintf("Created job Type:%s, ID:%s, Status:%s, Args:%s",
                    job.Type,
                    job.ID,
                    job.Status,
                    job.Args))
            }
            return job.ID, nil
        }
        # 为指定代理创建作业并发送
        job.ID = core.RandStringBytesMaskImprSrc(10)
        s := Agents[agentID].channel
        s <- []Job{job}
        Log(agentID, fmt.Sprintf("Created job Type:%s, ID:%s, Status:%s, Args:%s",
            job.Type,
            job.ID,
            job.Status,
            job.Args))
        return job.ID, nil
    }
    # 如果代理ID无效，则返回错误
    return "", errors.New("invalid agent ID")
// GetMessageForJob 根据作业类型返回消息基础结构
func GetMessageForJob(agentID uuid.UUID, job Job) (messages.Base, error) {
    // 创建消息基础结构
    m := messages.Base{
        Version: 1.0,
        ID:      agentID,
    }
    // 如果代理ID不合法，返回错误
    if !isAgent(agentID) {
        return m, fmt.Errorf("%s is not a valid agent", agentID.String())
    }
    // 为消息添加填充内容
    m.Padding = core.RandStringBytesMaskImprSrc(Agents[agentID].PaddingMax)
    // 根据作业类型进行不同的处理
    switch job.Type {
    case "cmd","cmdScriptFromPath":
        // 根据作业类型设置消息类型
        if job.Type == "cmd"{
            m.Type = "CmdPayload"
        }else {
            m.Type = "CmdPayloadScriptFromPath"
        }
        // 创建命令载荷
        p := messages.CmdPayload{
            Command: job.Args[0],
            Job:     job.ID,
        }
        // 如果作业参数大于1个，将其连接成字符串
        if len(job.Args) > 1 {
            p.Args = strings.Join(job.Args[1:], " ")
        }
        // 设置消息载荷
        m.Payload = p
    case "cmdgo":
        // 设置消息类型
        m.Type = "CmdGoPayload"
        // 创建命令载荷
        p := messages.CmdPayload{
            Command: job.Args[0],
            Job:     job.ID,
        }
        // 设置参数数组
        p.ArgsArray = job.Args[1:]
        // 设置消息载荷
        m.Payload = p
    case "cmdgoprogress":
        // 设置消息类型
        m.Type = "CmdGoProgressPayload"
        // 创建命令载荷
        p := messages.CmdPayload{
            Command: job.Args[0],
            Job:     job.ID,
        }
        // 设置参数数组
        p.ArgsArray = job.Args[1:]
        // 设置消息载荷
        m.Payload = p
    # 如果任务类型是 shellcode
    case "shellcode":
        # 设置消息类型为 Shellcode
        m.Type = "Shellcode"
        # 创建 Shellcode 消息对象
        p := messages.Shellcode{
            Method: job.Args[0],  # 设置方法
            Job:    job.ID,  # 设置任务 ID
        }

        # 如果方法是 self
        if p.Method == "self":
            p.Bytes = job.Args[1]  # 设置字节码
        # 如果方法是 remote、rtlcreateuserthread 或者 userapc
        else if p.Method == "remote" || p.Method == "rtlcreateuserthread" || p.Method == "userapc":
            i, err := strconv.Atoi(job.Args[1])  # 将字符串转换为整数
            if err != nil:
                return m, err  # 返回错误
            p.PID = uint32(i)  # 设置进程 ID
            p.Bytes = job.Args[2]  # 设置字节码
        # 设置消息的负载为 Shellcode 消息对象
        m.Payload = p
    # 如果任务类型是 download
    case "download":
        # 设置消息类型为 FileTransfer
        m.Type = "FileTransfer"
        # 记录日志
        Log(agentID, fmt.Sprintf("Downloading file from agent at %s\n", job.Args[0]))
        # 创建 FileTransfer 消息对象
        p := messages.FileTransfer{
            FileLocation: job.Args[0],  # 设置文件位置
            Job:          job.ID,  # 设置任务 ID
            IsDownload:   false,  # 设置是否下载
        }
        # 设置消息的负载为 FileTransfer 消息对象
        m.Payload = p
    # 如果任务类型是 initialize
    case "initialize":
        # 设置消息类型为 AgentControl
        m.Type = "AgentControl"
        # 创建 AgentControl 消息对象
        p := messages.AgentControl{
            Command: job.Type,  # 设置命令
            Job:     job.ID,  # 设置任务 ID
        }
        # 设置消息的负载为 AgentControl 消息对象
        m.Payload = p
    # 如果任务类型是 kill
    case "kill":
        # 设置消息类型为 AgentControl
        m.Type = "AgentControl"
        # 创建 AgentControl 消息对象
        p := messages.AgentControl{
            Command: job.Args[0],  # 设置命令
            Job:     job.ID,  # 设置任务 ID
        }
        # 设置消息的负载为 AgentControl 消息对象
        m.Payload = p
    # 如果任务类型是 ls
    case "ls":
        # 设置消息类型为 NativeCmd
        m.Type = "NativeCmd"
        # 创建 NativeCmd 消息对象
        p := messages.NativeCmd{
            Job:     job.ID,  # 设置任务 ID
            Command: job.Args[0],  # 设置命令
        }

        # 如果参数数量大于 1
        if len(job.Args) > 1:
            p.Args = job.Args[1]  # 设置参数
        else:
            p.Args = "./"  # 设置默认参数
        # 设置消息的负载为 NativeCmd 消息对象
        m.Payload = p
    # 如果任务类型是 killdate
    case "killdate":
        # 设置消息类型为 AgentControl
        m.Type = "AgentControl"
        # 创建 AgentControl 消息对象
        p := messages.AgentControl{
            Command: job.Args[0],  # 设置命令
            Job:     job.ID,  # 设置任务 ID
        }
        # 如果参数数量为 2
        if len(job.Args) == 2:
            p.Args = job.Args[1]  # 设置参数
        # 设置消息的负载为 AgentControl 消息对象
        m.Payload = p
    # 如果任务类型是 cd
    case "cd":
        # 设置消息类型为 NativeCmd
        m.Type = "NativeCmd"
        # 创建 NativeCmd 消息对象
        p := messages.NativeCmd{
            Job:     job.ID,  # 设置任务 ID
            Command: job.Args[0],  # 设置命令
            Args:    strings.Join(job.Args[1:], " "),  # 设置参数
        }
        # 设置消息的负载为 NativeCmd 消息对象
        m.Payload = p
    # 如果命令是 "pwd"，则设置消息类型为 "NativeCmd"
    case "pwd":
        # 创建本地命令对象
        m.Type = "NativeCmd"
        p := messages.NativeCmd{
            Job:     job.ID,
            Command: job.Args[0],
            Args:    "",
        }
        # 设置消息的载荷为本地命令对象
        m.Payload = p
    # 如果命令是 "maxretry"，则设置消息类型为 "AgentControl"
    case "maxretry":
        # 创建代理控制对象
        m.Type = "AgentControl"
        p := messages.AgentControl{
            Command: job.Args[0],
            Job:     job.ID,
        }
        # 如果命令参数的长度为2，则设置代理控制对象的参数
        if len(job.Args) == 2 {
            p.Args = job.Args[1]
        }
        # 设置消息的载荷为代理控制对象
        m.Payload = p
    # 如果命令是 "padding"，"skew"，"sleep"，"ja3"，则设置消息类型为 "AgentControl"，并设置相应的代理控制对象
    case "padding":
        m.Type = "AgentControl"
        p := messages.AgentControl{
            Command: job.Args[0],
            Job:     job.ID,
        }
        if len(job.Args) == 2 {
            p.Args = job.Args[1]
        }
        m.Payload = p
    case "skew":
        m.Type = "AgentControl"
        p := messages.AgentControl{
            Command: job.Args[0],
            Job:     job.ID,
        }
        if len(job.Args) == 2 {
            p.Args = job.Args[1]
        }
        m.Payload = p
    case "sleep":
        m.Type = "AgentControl"
        p := messages.AgentControl{
            Command: job.Args[0],
            Job:     job.ID,
        }
        if len(job.Args) == 2 {
            p.Args = job.Args[1]
        }
        m.Payload = p
    case "ja3":
        m.Type = "AgentControl"
        p := messages.AgentControl{
            Command: job.Args[0],
            Job:     job.ID,
        }
        if len(job.Args) == 2 {
            p.Args = job.Args[1]
        }
        m.Payload = p
    # 如果命令是 "Minidump"，则设置消息类型为 "Module"，并设置相应的模块对象
    case "Minidump":
        m.Type = "Module"
        p := messages.Module{
            Command: job.Type,
            Job:     job.ID,
            Args:    job.Args,
        }
        m.Payload = p
    # 根据不同的情况进行处理
    case "upload":
        # 设置消息类型为文件传输
        m.Type = "FileTransfer"
        // TODO add error handling; check 2 args (src, dst)
        # 读取要上传的文件内容
        uploadFile, uploadFileErr := ioutil.ReadFile(job.Args[0])
        # 如果读取文件出错，则返回错误消息
        if uploadFileErr != nil {
            // TODO send "ServerOK"
            return m, fmt.Errorf("there was an error reading %s: %v", job.Type, uploadFileErr)
        }
        # 计算文件的 SHA-256 哈希值
        fileHash := sha256.New()
        _, err := io.WriteString(fileHash, string(uploadFile))
        # 如果计算哈希值出错，则记录警告消息
        if err != nil {
            message("warn", fmt.Sprintf("There was an error generating file hash:\r\n%s", err.Error()))
        }
        # 记录上传文件的相关信息
        Log(agentID, fmt.Sprintf("Uploading file from server at %s of size %d bytes and SHA-256: %x to agent at %s",
            job.Args[0],
            len(uploadFile),
            fileHash.Sum(nil),
            job.Args[1]))
        # 设置消息的负载为文件传输相关信息
        p := messages.FileTransfer{
            FileLocation: job.Args[1],
            FileBlob:     base64.StdEncoding.EncodeToString([]byte(uploadFile)),
            IsDownload:   true, // The agent will be downloading the file provided by the server in the FileBlob field
            Job:          job.ID,
        }
        m.Payload = p
    # 如果不是上传操作，则设置消息类型为服务器确认，并返回错误消息
    default:
        m.Type = "ServerOk"
        return m, fmt.Errorf("invalid job type: %s, sending ServerOK", m.Type)
    }
    # 返回消息和空错误
    return m, nil
// GetAgentStatus 评估代理的最后一次检查时间和最大等待时间，以确定其是否活动、延迟或已停止
func GetAgentStatus(agentID uuid.UUID) string {
    var status string
    // 如果代理ID不在Agents映射中，则返回错误消息
    if !isAgent(agentID) {
        return fmt.Sprintf("%s is not a valid agent", agentID.String())
    }
    // 将代理的等待时间解析为持续时间
    dur, errDur := time.ParseDuration(Agents[agentID].WaitTime)
    // 如果解析出错，则记录警告消息
    if errDur != nil {
        message("warn", fmt.Sprintf("Error converting %s to a time duration: %s", Agents[agentID].WaitTime,
            errDur.Error()))
    }
    // 如果代理的状态检查时间加上等待时间在当前时间之后，则状态为"Active"
    if Agents[agentID].StatusCheckIn.Add(dur).After(time.Now()) {
        status = "Active"
    } else if Agents[agentID].StatusCheckIn.Add(dur * time.Duration(Agents[agentID].MaxRetry+1)).After(time.Now()) { // +1 to account for skew
        // 如果代理的状态检查时间加上等待时间乘以最大重试次数加1在当前时间之后，则状态为"Delayed"
        status = "Delayed"
    } else {
        // 否则状态为"Dead"
        status = "Dead"
    }
    return status
}

// RemoveAgent 通过ID从Agents映射中删除代理对象
func RemoveAgent(agentID uuid.UUID) error {
    // 如果代理ID存在，则从Agents映射中删除该代理对象
    if isAgent(agentID) {
        delete(Agents, agentID)
        return nil
    }
    // 否则返回错误消息
    return fmt.Errorf("%s is not a known agent and was not removed", agentID.String())
}

// GetAgentFieldValue 返回指定代理的字段值的字符串值
func GetAgentFieldValue(agentID uuid.UUID, field string) (string, error) {
    // 如果代理ID存在，则根据字段返回相应的值
    if isAgent(agentID) {
        switch strings.ToLower(field) {
        case "platform":
            return Agents[agentID].Platform, nil
        case "architecture":
            return Agents[agentID].Architecture, nil
        case "username":
            return Agents[agentID].UserName, nil
        case "waittime":
            return Agents[agentID].WaitTime, nil
        }
        // 如果字段不存在，则返回错误消息
        return "", fmt.Errorf("the provided agent field could not be found: %s", field)
    }
    // 如果代理ID不存在，则返回错误消息
    return "", fmt.Errorf("%s is not a valid agent", agentID.String())
}

// isAgent 枚举所有实例化代理的映射，并在提供的代理UUID存在时返回true
// 判断给定的代理ID是否存在于全局代理映射中
func isAgent(agentID uuid.UUID) bool {
    // 遍历全局代理映射，检查是否存在给定的代理ID
    for agent := range Agents {
        if Agents[agent].ID == agentID {
            return true
        }
    }
    return false
}

// newAgent函数创建一个新的代理对象并返回该对象，但不将其添加到全局代理映射中
func newAgent(agentID uuid.UUID) (agent, error) {
    // 如果处于调试模式，则输出进入agents.newAgent函数的调试信息
    if core.Debug {
        message("debug", "Entering into agents.newAgent function")
    }
    var agent agent
    // 如果给定的代理ID已存在，则返回错误
    if isAgent(agentID) {
        return agent, fmt.Errorf("the %s agent already exists", agentID)
    }

    // 设置代理文件夹的路径
    agentsDir := filepath.Join(core.CurrentDir, "data", "agents")

    // 创建新代理文件夹
    if _, err := os.Stat(filepath.Join(agentsDir, agentID.String())); os.IsNotExist(err) {
        // 创建代理文件夹，并设置权限为0750
        errM := os.MkdirAll(filepath.Join(agentsDir, agentID.String()), 0750)
        if errM != nil {
            return agent, fmt.Errorf("there was an error creating a directory for agent %s:\r\n%s",
                agentID.String(), err.Error())
        }
        // 创建代理的日志文件
        agentLog, errC := os.Create(filepath.Join(agentsDir, agentID.String(), "agent_log.txt"))
        if errC != nil {
            return agent, fmt.Errorf("there was an error creating the agent_log.txt file for agnet %s:\r\n%s",
                agentID.String(), err.Error())
        }

        // 更改文件权限
        errChmod := os.Chmod(agentLog.Name(), 0600)
        if errChmod != nil {
            return agent, fmt.Errorf("there was an error changing the file permissions for the agent log:\r\n%s", errChmod.Error())
        }

        // 如果处于详细模式，则输出已创建代理日志文件的信息
        if core.Verbose {
            message("note", fmt.Sprintf("Created agent log file at: %s agent_log.txt",
                path.Join(agentsDir, agentID.String())))
        }
    }
    // 以追加写入模式打开代理的日志文件
    f, err := os.OpenFile(filepath.Clean(filepath.Join(agentsDir, agentID.String(), "agent_log.txt")), os.O_APPEND|os.O_WRONLY, 0600)
    # 如果发生错误，则返回错误信息
    if err != nil:
        return agent, fmt.Errorf("there was an error openeing the %s agent's log file:\r\n%s", agentID.String(), err.Error())

    # 设置代理的ID
    agent.ID = agentID
    # 设置代理的日志文件
    agent.agentLog = f
    # 设置代理的初始检查时间
    agent.InitialCheckIn = time.Now().UTC()
    # 设置代理的状态检查时间
    agent.StatusCheckIn = time.Now().UTC()
    # 创建一个通道，用于传输作业
    agent.channel = make(chan []Job, 10)

    # 将实例化代理的信息写入代理日志文件
    _, errAgentLog := agent.agentLog.WriteString(fmt.Sprintf("[%s]%s\r\n", time.Now().UTC().Format(time.RFC3339), "Instantiated agent"))
    # 如果写入日志文件时发生错误，则记录错误信息
    if errAgentLog != nil:
        message("warn", fmt.Sprintf("There was an error writing to the agent log agents.Log:\r\n%s", errAgentLog.Error()))

    # 如果处于调试模式，则记录离开代理创建函数的信息
    if core.Debug:
        message("debug", "Leaving agents.newAgent function without error")
    # 返回代理和空的错误信息
    return agent, nil
// JobResults处理代理发送的响应消息
func JobResults(m messages.Base) error {
    if core.Debug {
        message("debug", "Entering into agents.JobResults")
    }

    // 检查以确保它是已知的代理
    if !isAgent(m.ID) {
        return fmt.Errorf("%s is not a known agent", m.ID)
    }

    // 检查以确保它是该代理的真实作业
    p := m.Payload.(messages.CmdResults)
    Log(m.ID, fmt.Sprintf("Results for job: %s", p.Job))

    message("success", fmt.Sprintf("Results for %s job %s at %s", m.ID, p.Job, time.Now().UTC().Format(time.RFC3339)))

    if len(p.Stdout) > 0 {
        Log(m.ID, fmt.Sprintf("Command Results (stdout):\r\n%s", p.Stdout))
        message("plain", color.GreenString("%s", p.Stdout))
    }
    if len(p.Stderr) > 0 {
        Log(m.ID, fmt.Sprintf("Command Results (stderr):\r\n%s", p.Stderr))
        message("plain", color.RedString("%s", p.Stderr))
    }

    if core.Debug {
        message("debug", "Leaving agents.JobResults")
    }
    fmt.Println()
    return nil
}

// FileTransfer处理文件上传/下载操作
func FileTransfer(m messages.Base) error {
    if core.Debug {
        message("debug", "Entering into agents.FileTransfer")
    }

    // 检查以确保它是已知的代理
    if !isAgent(m.ID) {
        return fmt.Errorf("%s is not a known agent", m.ID)
    }

    p := m.Payload.(messages.FileTransfer)
    # 如果下载标志为真
    if p.IsDownload:
        # 拼接代理目录路径
        agentsDir := filepath.Join(core.CurrentDir, "data", "agents")
        # 获取文件路径中的文件名部分
        _, f := filepath.Split(p.FileLocation) # 我们不需要目录部分
        # 检查代理目录是否存在
        if _, errD := os.Stat(agentsDir); os.IsNotExist(errD):
            # 如果代理目录不存在，则返回错误消息
            errorMessage := fmt.Errorf("there was an error locating the agent's directory:\r\n%s", errD.Error())
            Log(m.ID, errorMessage.Error())
            return errorMessage
        # 输出成功消息
        message("success", fmt.Sprintf("Results for %s job %s at %s", m.ID, p.Job, time.Now().UTC().Format(time.RFC3339)))
        # 解码文件数据
        downloadBlob, downloadBlobErr := base64.StdEncoding.DecodeString(p.FileBlob)
        # 如果解码出错，则返回错误消息
        if downloadBlobErr != nil:
            errorMessage := fmt.Errorf("there was an error decoding the fileBlob:\r\n%s", downloadBlobErr.Error())
            Log(m.ID, errorMessage.Error())
            return errorMessage
        # 拼接下载文件的完整路径
        downloadFile := filepath.Join(agentsDir, m.ID.String(), f)
        # 将解码后的文件数据写入文件
        writingErr := ioutil.WriteFile(downloadFile, downloadBlob, 0600)
        # 如果写入出错，则返回错误消息
        if writingErr != nil:
            errorMessage := fmt.Errorf("there was an error writing to -> %s:\r\n%s", p.FileLocation, writingErr.Error())
            Log(m.ID, errorMessage.Error())
            return errorMessage
        # 输出成功消息
        successMessage := fmt.Sprintf("Successfully downloaded file %s with a size of %d bytes from agent %s to %s",
            p.FileLocation,
            len(downloadBlob),
            m.ID.String(),
            downloadFile)
        message("success", successMessage)
        Log(m.ID, successMessage)
    # 如果调试标志为真
    if core.Debug:
        # 输出调试消息
        message("debug", "Leaving agents.FileTransfer")
    # 返回空值
    return nil
// GetLifetime 返回一个代理在成功与服务器通信之前可以存活的时间
func GetLifetime(agentID uuid.UUID) (time.Duration, error) {
    if core.Debug {
        message("debug", "Entering into agents.GetLifeTime")
    }
    // 检查代理是否已知
    if !isAgent(agentID) {
        return 0, fmt.Errorf("%s is not a known agent", agentID)
    }

    // 检查 PID 是否设置，以确定是否已发送第一个 AgentInfo 消息
    if Agents[agentID].Pid == 0 {
        return 0, nil
    }

    // 解析代理的等待时间
    sleep, errSleep := time.ParseDuration(Agents[agentID].WaitTime)
    if errSleep != nil {
        return 0, fmt.Errorf("there was an error parsing the agent WaitTime to a duration:\r\n%s", errSleep.Error())
    }
    if sleep == 0 {
        return 0, fmt.Errorf("agent WaitTime is equal to zero")
    }

    // 检查最大重试次数
    retry := Agents[agentID].MaxRetry
    if retry == 0 {
        return 0, fmt.Errorf("agent MaxRetry is equal to zero")
    }

    // 获取时间偏差和最大重试次数
    skew := time.Duration(Agents[agentID].Skew) * time.Millisecond
    maxRetry := Agents[agentID].MaxRetry

    // 计算代理在死亡之前可能存活的最长时间
    lifetime := sleep + skew
    for maxRetry > 1 {
        lifetime = lifetime + (sleep + skew)
        maxRetry--
    }

    // 检查代理的生命周期是否超过了 killdate
    if Agents[agentID].KillDate > 0 {
        if time.Now().Add(lifetime).After(time.Unix(Agents[agentID].KillDate, 0)) {
            return 0, fmt.Errorf("the agent lifetime will exceed the killdate")
        }
    }

    if core.Debug {
        message("debug", "Leaving agents.GetLifeTime without error")
    }

    return lifetime, nil
}

// Job 是一个结构，用于保存分配给单个代理的单个任务的数据
type Job struct {
    ID      string
    Type    string
    Status  string // 有效的状态为 created, sent, returned //TODO this might not be needed
    Args    []string
    Created time.Time
}
```