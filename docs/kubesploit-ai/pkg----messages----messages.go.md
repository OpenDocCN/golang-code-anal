# `kubesploit\pkg\messages\messages.go`

```go
// Kubesploit 是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 © 2021 CyberArk Software Ltd.

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 由自由软件基金会发布的版本3或任何以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何担保；包括适销性或特定用途适用性的隐含担保。有关更多详细信息，请参见GNU通用公共许可证。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

package messages

import (
    "crypto/rsa"
    "encoding/gob"
    "github.com/satori/go.uuid"
)

// init函数用于注册基于gob的消息类型，这些消息类型是Base.Payload的接口
func init() {
    gob.Register(AgentControl{})
    gob.Register(AgentInfo{})
    gob.Register(CmdPayload{})
    gob.Register(CmdResults{})
    gob.Register(FileTransfer{})
    gob.Register(KeyExchange{})
    gob.Register(Module{})
    gob.Register(NativeCmd{})
    gob.Register(Shellcode{})
    gob.Register(SysInfo{})
}

// Base是HTTP POST负载的基本JSON对象
type Base struct {
    Version float32     `json:"version"`
    ID      uuid.UUID   `json:"id"`
    Type    string      `json:"type"`
    Payload interface{} `json:"payload,omitempty"`
    Padding string      `json:"padding"`
    Token   string      `json:"token,omitempty"`
}

// FileTransfer是用于在服务器和代理之间传输文件的JSON负载
type FileTransfer struct {
    FileLocation string `json:"dest"`
    FileBlob     string `json:"blob"`
    # 定义一个布尔类型的字段 IsDownload，用于表示是否下载
    IsDownload   bool   `json:"download"`
    # 定义一个字符串类型的字段 Job，用于表示工作
    Job          string `json:"job"`
// CmdPayload 是用于在代理上执行命令的 JSON 负载
type CmdPayload struct {
    Command string `json:"executable"`  // 命令
    Args    string `json:"args"`        // 参数
    ArgsArray    []string `json:"argsarray"`  // 参数数组
    Job     string `json:"job"`          // 作业
}

// SysInfo 是一个包含代理运行系统信息的 JSON 负载
type SysInfo struct {
    Platform     string   `json:"platform,omitempty"`  // 平台
    Architecture string   `json:"architecture,omitempty"`  // 架构
    UserName     string   `json:"username,omitempty"`  // 用户名
    UserGUID     string   `json:"userguid,omitempty"`  // 用户 GUID
    HostName     string   `json:"hostname,omitempty"`  // 主机名
    Pid          int      `json:"pid,omitempty"`  // 进程 ID
    Ips          []string `json:"ips,omitempty"`  // IP 地址
}

// CmdResults 是一个包含代理执行命令结果的 JSON 负载
type CmdResults struct {
    Job     string `json:"job"`      // 作业
    Stdout  string `json:"stdout"`   // 标准输出
    Stderr  string `json:"stderr"`   // 标准错误输出
    Padding string `json:"padding"`  // 用于帮助规避检测的填充
}

// AgentControl 是一个用于向代理发送控制消息（如终止）的 JSON 负载
type AgentControl struct {
    Job     string `json:"job"`      // 作业
    Command string `json:"command"`  // 命令
    Args    string `json:"args,omitempty"`  // 参数
    Result  string `json:"result"`   // 结果
}

// AgentInfo 是一个包含有关代理及其配置信息的 JSON 负载
type AgentInfo struct {
    Version       string  `json:"version,omitempty"`  // 版本
    Build         string  `json:"build,omitempty"`    // 构建
    WaitTime      string  `json:"waittime,omitempty"`  // 等待时间
    PaddingMax    int     `json:"paddingmax,omitempty"`  // 填充最大值
    MaxRetry      int     `json:"maxretry,omitempty"`  // 最大重试次数
    FailedCheckin int     `json:"failedcheckin,omitempty"`  // 失败签入次数
    Skew          int64   `json:"skew,omitempty"`  // 偏差
    Proto         string  `json:"proto,omitempty"`  // 协议
    SysInfo       SysInfo `json:"sysinfo,omitempty"`  // 系统信息
    KillDate      int64   `json:"killdate,omitempty"`  // 终止日期
    # 定义一个名为JA3的字符串变量，并使用json标签指定其在序列化为json时的字段名为ja3，如果字段值为空则忽略该字段
    JA3           string  `json:"ja3,omitempty"`
// Shellcode 是一个包含 shellcode 和执行方法的 JSON 负载
type Shellcode struct {
    Method string `json:"method"` // 执行方法
    Bytes  string `json:"bytes"` // shellcode 字节的 Base64 字符串
    Job    string `json:"job"` // 作业标识
    PID    uint32 `json:"pid,omitempty"` // 远程注入的进程 ID
}

// Module 是用于发送模块指令的 JSON 负载
type Module struct {
    Job     string   `json:"job"` // 作业标识
    Command string   `json:"command"` // 指令
    Args    []string `json:"args,omitempty"` // 参数列表
    Result  string   `json:"result"` // 结果
}

// NativeCmd 是用于在 Merlin 中使用 Go 执行本地命令的 JSON 负载，而不是在主机上执行二进制程序（例如 ls）
type NativeCmd struct {
    Job     string `json:"job"` // 作业标识
    Command string `json:"command"` // 命令
    Args    string `json:"args,omitempty"` // 参数
}

// KeyExchange 是用于交换加密公钥的 JSON 负载
type KeyExchange struct {
    PublicKey rsa.PublicKey `json:"publickey"` // 公钥
}
```