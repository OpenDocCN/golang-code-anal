# `kubesploit\pkg\api\agents\agents.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 由自由软件基金会发布的版本3或任何以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

package agents

import (
    // 标准库
    "fmt"
    "os"
    "strconv"
    "strings"

    // 第三方库
    "github.com/mattn/go-shellwords"
    uuid "github.com/satori/go.uuid"

    // Merlin
    "kubesploit/pkg/agents"
    "kubesploit/pkg/api/messages"
    "kubesploit/pkg/modules/shellcode"
)

// CD用于更改代理的当前工作目录
func CD(agentID uuid.UUID, Args []string) messages.UserMessage {
    if len(Args) > 1 {
        arg := strings.Join(Args[0:], " ")
        argS, errS := shellwords.Parse(arg)
        if errS != nil {
            m := fmt.Sprintf("解析命令行参数时出错：%s\r\n%s", Args, errS.Error())
            return messages.ErrorMessage(m)
        }
        job, err := agents.AddJob(agentID, "cd", argS)
        if err != nil {
            return messages.ErrorMessage(err.Error())
        }
        return messages.JobMessage(agentID, job)
    }
    job, err := agents.AddJob(agentID, "cd", Args)
    if err != nil {
        return messages.ErrorMessage(err.Error())
    # 返回一个包含代理ID和作业的JobMessage对象
    return messages.JobMessage(agentID, job)
// 用于向代理发送命令以运行命令或执行程序
// Args[0] = "cmd"
// Args[1:] = 在运行代理的主机操作系统上要执行的程序和参数
// 与 `cmd` 和 `shell` 命令一起使用，以及通过 "standard" 模块
func CMD(agentID uuid.UUID, Args []string, jobType string) messages.UserMessage {
    if len(Args) > 0 {
        job, err := agents.AddJob(agentID, jobType, Args[1:])
        if err != nil {
            return messages.ErrorMessage(err.Error())
        }
        return messages.JobMessage(agentID, job)
    }
    return messages.ErrorMessage("not enough arguments provided for the Agent Cmd call")
}

// CMDGO 用于向代理发送命令以运行命令或执行程序
// Args[0] = "cmdgo"
// Args[1:] = 在运行代理的主机操作系统上要执行的程序和参数
func CMDGO(agentID uuid.UUID, Args []string) messages.UserMessage {
    if len(Args) > 0 {
        job, err := agents.AddJob(agentID, "cmdgo", Args)
        if err != nil {
            return messages.ErrorMessage(err.Error())
        }
        return messages.JobMessage(agentID, job)
    }
    return messages.ErrorMessage("not enough arguments provided for the Agent Cmd call")
}

// CMDGOPROG 用于向代理发送命令以运行命令或执行程序
// Args[0] = "cmd"
// Args[1:] = 在运行代理的主机操作系统上要执行的程序和参数
// Download函数用于通过相应的代理从提供的输入文件路径下载文件
// Args[0] = download
// Args[1] = 要下载的文件路径
func Download(agentID uuid.UUID, Args []string) messages.UserMessage {
    if len(Args) >= 2 {
        // 将参数连接成一个字符串
        arg := strings.Join(Args[1:], " ")
        // 解析参数字符串
        argS, errS := shellwords.Parse(arg)
        if errS != nil {
            // 如果解析参数出错，返回错误消息
            m := fmt.Sprintf("解析命令行参数时出错: %s\r\n%s",
                Args[1:], errS.Error())
            return messages.ErrorMessage(m)
        }
        if len(argS) >= 1 {
            // 向代理添加下载任务
            job, err := agents.AddJob(agentID, "download", argS[0:1])
            if err != nil {
                return messages.ErrorMessage(err.Error())
            }
            return messages.JobMessage(agentID, job)
        }
    }
    return messages.ErrorMessage(fmt.Sprintf("代理下载调用时未提供足够的参数: %s", Args))
}

// ExecuteShellcode调用相应的shellcode模块来创建一个执行提供的shellcode的作业
// Args[0] = "execute-shellcode
// Args[1] = Shellcode执行方法 [self, remote, retlcreateuserthread, userapc]
func ExecuteShellcode(agentID uuid.UUID, Args []string) messages.UserMessage {
    // 如果参数数量不足，返回错误消息
    return messages.ErrorMessage(fmt.Sprintf("代理ExecuteShellcode调用时未提供足够的参数: %s", Args))
}

// Kill指示代理停止运行
func Kill(agentID uuid.UUID, Args []string) messages.UserMessage {
    # 如果参数列表长度大于0
    if len(Args) > 0 {
        # 向代理程序添加一个“kill”类型的作业，并传入参数列表
        job, err := agents.AddJob(agentID, "kill", Args[0:])
        # 如果出现错误，返回错误消息
        if err != nil {
            return messages.ErrorMessage(err.Error())
        }
        # 返回代理程序的作业消息
        return messages.JobMessage(agentID, job)
    }
    # 如果参数不足，返回错误消息
    return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent Kill call: %s", Args))
// LS 使用原生 Go 语言列出目录
func LS(agentID uuid.UUID, Args []string) messages.UserMessage {
    // 如果参数数量大于1
    if len(Args) > 1 {
        // 将参数连接成一个字符串
        arg := strings.Join(Args[0:], " ")
        // 解析参数字符串
        argS, errS := shellwords.Parse(arg)
        // 如果解析出错
        if errS != nil {
            // 返回错误消息
            m := fmt.Sprintf("there was an error parsing command line argments: %s\r\n%s", Args, errS.Error())
            return messages.ErrorMessage(m)
        }
        // 添加一个 ls 任务到代理
        job, err := agents.AddJob(agentID, "ls", argS)
        // 如果添加任务出错
        if err != nil {
            // 返回错误消息
            return messages.ErrorMessage(err.Error())
        }
        // 返回任务消息
        return messages.JobMessage(agentID, job)
    }
    // 添加一个 ls 任务到代理
    job, err := agents.AddJob(agentID, "ls", Args)
    // 如果添加任务出错
    if err != nil {
        // 返回错误消息
        return messages.ErrorMessage(err.Error())
    }
    // 返回任务消息
    return messages.JobMessage(agentID, job)
}

// PWD 用于打印代理的当前工作目录
func PWD(agentID uuid.UUID, Args []string) messages.UserMessage {
    // 添加一个 pwd 任务到代理
    job, err := agents.AddJob(agentID, "pwd", Args)
    // 如果添加任务出错
    if err != nil {
        // 返回错误消息
        return messages.ErrorMessage(err.Error())
    }
    // 返回任务消息
    return messages.JobMessage(agentID, job)
}

// SetJA3 用于更改代理的 JA3 签名
func SetJA3(agentID uuid.UUID, Args []string) messages.UserMessage {
    // 如果参数数量大于2
    if len(Args) > 2 {
        // 添加一个 ja3 任务到代理
        job, err := agents.AddJob(agentID, "ja3", Args[1:])
        // 如果添加任务出错
        if err != nil {
            // 返回错误消息
            return messages.ErrorMessage(err.Error())
        }
        // 返回任务消息
        return messages.JobMessage(agentID, job)
    }
    // 返回错误消息，表示参数不足
    return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent SetJA3 call: %s", Args))
}

// SetKillDate 配置代理将停止运行的日期和时间
func SetKillDate(agentID uuid.UUID, Args []string) messages.UserMessage {
    # 检查参数列表的长度是否大于2
    if len(Args) > 2 {
        # 将参数转换为int64类型
        _, errU := strconv.ParseInt(Args[2], 10, 64)
        # 如果转换出错，返回错误消息
        if errU != nil {
            m := fmt.Sprintf("There was an error converting %s to an int64", Args[2])
            m = m + "\r\nKill date takes in a UNIX epoch timestamp such as 811123200 for September 15, 1995"
            return messages.ErrorMessage(m)
        }
        # 向代理添加一个名为"killdate"的作业
        job, err := agents.AddJob(agentID, "killdate", Args[1:])
        # 如果添加作业出错，返回错误消息
        if err != nil {
            return messages.ErrorMessage(err.Error())
        }
        # 返回代理和作业的消息
        return messages.JobMessage(agentID, job)
    }
    # 如果参数不足，返回错误消息
    return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent SetKillDate call: %s", Args))
// 设置最大重试次数，配置代理在放弃之前尝试检查的次数
func SetMaxRetry(agentID uuid.UUID, Args []string) messages.UserMessage {
    // 如果参数数量大于2
    if len(Args) > 2 {
        // 添加作业到代理，并返回作业信息
        job, err := agents.AddJob(agentID, "maxretry", Args[1:])
        // 如果出现错误，返回错误信息
        if err != nil {
            return messages.ErrorMessage(err.Error())
        }
        // 返回作业信息
        return messages.JobMessage(agentID, job)
    }
    // 返回错误信息，指示代理 SetMaxRetry 调用提供的参数不足
    return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent SetMaxRetry call: %s", Args))
}

// 设置填充大小，配置每条消息添加的随机填充的最大大小
func SetPadding(agentID uuid.UUID, Args []string) messages.UserMessage {
    // 如果参数数量大于2
    if len(Args) > 2 {
        // 添加作业到代理，并返回作业信息
        job, err := agents.AddJob(agentID, "padding", Args[1:])
        // 如果出现错误，返回错误信息
        if err != nil {
            return messages.ErrorMessage(err.Error())
        }
        // 返回作业信息
        return messages.JobMessage(agentID, job)
    }
    // 返回错误信息，指示代理 SetPadding 调用提供的参数不足
    return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent SetPadding call: %s", Args))
}

// 设置休眠时间，配置代理在检查之间的休眠时间
func SetSleep(agentID uuid.UUID, Args []string) messages.UserMessage {
    // 如果参数数量大于2
    if len(Args) > 2 {
        // 添加作业到代理，并返回作业信息
        job, err := agents.AddJob(agentID, "sleep", Args[1:])
        // 如果出现错误，返回错误信息
        if err != nil {
            return messages.ErrorMessage(err.Error())
        }
        // 返回作业信息
        return messages.JobMessage(agentID, job)
    }
    // 返回错误信息，指示代理 SetSleep 调用提供的参数不足
    return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent SetSleep call: %s", Args))
}

// 设置偏移量，配置代理用于随机化检查时间的偏移量
func SetSkew(agentID uuid.UUID, Args []string) messages.UserMessage {
    // 如果参数数量大于2
    if len(Args) > 2 {
        // 添加作业到代理，并返回作业信息
        job, err := agents.AddJob(agentID, "skew", Args[1:])
        // 如果出现错误，返回错误信息
        if err != nil {
            return messages.ErrorMessage(err.Error())
        }
        // 返回作业信息
        return messages.JobMessage(agentID, job)
    }
    # 返回一个错误消息，格式化字符串中包含参数 Args
    return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent SetSkew call: %s", Args))
// Upload函数用于将文件从Merlin服务器传输到Agent
func Upload(agentID uuid.UUID, Args []string) messages.UserMessage {
    // 检查参数数量是否大于等于3
    if len(Args) >= 3 {
        // 将参数连接成一个字符串
        arg := strings.Join(Args[1:], " ")
        // 解析命令行参数
        argS, errS := shellwords.Parse(arg)
        // 如果解析出错，返回错误消息
        if errS != nil {
            m := fmt.Sprintf("there was an error parsing command line argments: %s\r\n%s", Args, errS.Error())
            return messages.ErrorMessage(m)
        }
        // 如果参数数量大于等于2
        if len(argS) >= 2 {
            // 检查源文件是否存在
            _, errF := os.Stat(argS[0])
            // 如果访问源文件出错，返回错误消息
            if errF != nil {
                m := fmt.Sprintf("there was an error accessing the source upload file:\r\n%s", errF.Error())
                return messages.ErrorMessage(m)
            }
            // 添加上传任务到Agent
            job, err := agents.AddJob(agentID, "upload", argS[0:2])
            // 如果添加任务出错，返回错误消息
            if err != nil {
                return messages.ErrorMessage(err.Error())
            }
            // 返回任务消息
            return messages.JobMessage(agentID, job)
        }
    }
    // 如果参数不足，返回错误消息
    return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent Upload call: %s", Args))
}
```