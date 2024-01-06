# `kubesploit\pkg\api\agents\agents.go`

```
// Kubesploit 是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，不附带任何保证；包括对适销性或特定用途的隐含保证。请参阅GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本。
// 如果没有，请访问<http://www.gnu.org/licenses/>。

// 代理包
package agents
// 导入标准库中的 fmt、os、strconv、strings 模块
import (
	"fmt"
	"os"
	"strconv"
	"strings"
	
	// 导入第三方库中的 go-shellwords 和 go.uuid 模块
	"github.com/mattn/go-shellwords"
	uuid "github.com/satori/go.uuid"
	
	// 导入自定义模块中的 agents、messages 和 shellcode 模块
	"kubesploit/pkg/agents"
	"kubesploit/pkg/api/messages"
	"kubesploit/pkg/modules/shellcode"
)

// CD 函数用于改变代理的当前工作目录
func CD(agentID uuid.UUID, Args []string) messages.UserMessage {
	// 如果参数数量大于 1
	if len(Args) > 1 {
// 将命令行参数连接成一个字符串
arg := strings.Join(Args[0:], " ")
// 解析命令行参数，返回解析后的字符串数组
argS, errS := shellwords.Parse(arg)
// 如果解析出错，返回错误信息
if errS != nil {
    m := fmt.Sprintf("There was an error parsing command line argments: %s\r\n%s", Args, errS.Error())
    return messages.ErrorMessage(m)
}
// 向代理添加一个作业，指定代理ID、命令和参数，返回作业和可能的错误
job, err := agents.AddJob(agentID, "cd", argS)
// 如果添加作业出错，返回错误信息
if err != nil {
    return messages.ErrorMessage(err.Error())
}
// 返回代理ID和作业信息
return messages.JobMessage(agentID, job)
```

```
// 向代理添加一个作业，指定代理ID、命令和参数，返回作业和可能的错误
job, err := agents.AddJob(agentID, "cd", Args)
// 如果添加作业出错，返回错误信息
if err != nil {
    return messages.ErrorMessage(err.Error())
}
// 返回代理ID和作业信息
return messages.JobMessage(agentID, job)
}

// CMD is used to send a command to the agent to run a command or execute a program
// 定义一个名为CMD的函数，用于执行在运行代理主机操作系统上的程序和参数
func CMD(agentID uuid.UUID, Args []string, jobType string) messages.UserMessage {
    // 如果参数数量大于0，则将任务添加到代理中执行
    if len(Args) > 0 {
        // 调用agents.AddJob函数将任务添加到代理中执行
        job, err := agents.AddJob(agentID, jobType, Args)
        // 如果出现错误，则返回错误消息
        if err != nil {
// 返回一个包含错误信息的 ErrorMessage 对象
		return messages.ErrorMessage(err.Error())
	}
	// 返回一个包含代理ID和作业信息的 JobMessage 对象
	return messages.JobMessage(agentID, job)
}

// CMDGO 用于向代理发送命令以运行命令或执行程序
// Args[0] = "cmdgo"
// Args[1:] = 在运行代理的主机操作系统上要执行的程序和参数
//func CMDGO(agentID uuid.UUID, Args []string) messages.UserMessage {
//	if len(Args) > 0 {
//		// 向代理添加作业，类型为"cmdgo"，参数为Args
//		job, err := agents.AddJob(agentID, "cmdgo", Args)
//		if err != nil {
//			// 返回一个包含错误信息的 ErrorMessage 对象
//			return messages.ErrorMessage(err.Error())
//		}
//		// 返回一个包含代理ID和作业信息的 JobMessage 对象
//		return messages.JobMessage(agentID, job)
//	}
	// 返回一个包含错误信息的 ErrorMessage 对象，表示未提供足够的参数进行代理命令调用
//	return messages.ErrorMessage("not enough arguments provided for the Agent Cmd call")
//}
// CMDGOPROG 用于向代理发送命令以运行命令或执行程序
// Args[0] = "cmd"
// Args[1:] = 在运行代理的主机操作系统上要执行的程序和参数
//func CMDGOPROG(agentID uuid.UUID, Args []string) messages.UserMessage {
//	if len(Args) > 0 {
//		job, err := agents.AddJob(agentID, "cmdgoprogress", Args)
//		if err != nil {
//			return messages.ErrorMessage(err.Error())
//		}
//		return messages.JobMessage(agentID, job)
//	}
//	return messages.ErrorMessage("not enough arguments provided for the Agent Cmd call")
//}

// Download 用于通过相应的代理从提供的输入文件路径下载文件
// Args[0] = download
// Args[1] = 要下载的文件路径
# 定义一个名为 Download 的函数，接受代理ID和参数列表作为输入，并返回用户消息
func Download(agentID uuid.UUID, Args []string) messages.UserMessage {
    # 检查参数列表长度是否大于等于2
    if len(Args) >= 2 {
        # 将参数列表中除第一个元素外的所有元素连接成一个字符串
        arg := strings.Join(Args[1:], " ")
        # 解析命令行参数
        argS, errS := shellwords.Parse(arg)
        # 如果解析出错，返回错误消息
        if errS != nil {
            m := fmt.Sprintf("there was an error parsing command line argments: %s\r\n%s",
                Args[1:], errS.Error())
            return messages.ErrorMessage(m)
        }
        # 如果解析成功，继续处理
        if len(argS) >= 1 {
            # 向代理添加一个下载任务
            job, err := agents.AddJob(agentID, "download", argS[0:1])
            # 如果添加任务出错，返回错误消息
            if err != nil {
                return messages.ErrorMessage(err.Error())
            }
            # 返回代理和任务的消息
            return messages.JobMessage(agentID, job)
        }
    }
    # 如果参数不足，返回错误消息
    return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent Download call: %s", Args))
}
// ExecuteShellcode调用相应的shellcode模块来创建一个执行提供的shellcode的作业
// Args[0] = "execute-shellcode
// Args[1] = Shellcode执行方法 [self, remote, retlcreateuserthread, userapc]
func ExecuteShellcode(agentID uuid.UUID, Args []string) messages.UserMessage {
    // 检查参数数量是否大于2
    if len(Args) > 2 {
        // 创建一个选项映射
        options := make(map[string]string)
        // 根据参数[1]的值进行不同的操作
        switch strings.ToLower(Args[1]) {
        // 如果是"self"，设置执行方法为self，pid为空，shellcode为参数[2]之后的所有内容
        case "self":
            options["method"] = "self"
            options["pid"] = ""
            options["shellcode"] = strings.Join(Args[2:], " ")
        // 如果是"remote"，并且参数数量大于3，设置执行方法为remote，pid为参数[2]，shellcode为参数[3]之后的所有内容
        case "remote":
            if len(Args) > 3 {
                options["method"] = "remote"
                options["pid"] = Args[2]
                options["shellcode"] = strings.Join(Args[3:], " ")
            } else {
                // 如果参数数量不够，返回错误消息
                return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent ExecuteShellcode (remote) call: %s", Args))
            }
        // 如果是"rtlcreateuserthread"，进行相应的操作
        case "rtlcreateuserthread":
# 如果参数数量大于3
if len(Args) > 3:
    # 设置执行方法为rtlcreateuserthread
    options["method"] = "rtlcreateuserthread"
    # 设置进程ID
    options["pid"] = Args[2]
    # 将参数列表中的第三个参数及其后的参数连接成字符串，作为shellcode
    options["shellcode"] = strings.Join(Args[3:], " ")
else:
    # 返回错误消息，指出Agent ExecuteShellcode (rtlcreateuserthread)调用提供的参数不足
    return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent ExecuteShellcode (rtlcreateuserthread) call: %s", Args))

# 如果执行方法为userapc
case "userapc":
    if len(Args) > 3:
        # 设置执行方法为userapc
        options["method"] = "userapc"
        # 设置进程ID
        options["pid"] = Args[2]
        # 将参数列表中的第三个参数及其后的参数连接成字符串，作为shellcode
        options["shellcode"] = strings.Join(Args[3:], " ")
    else:
        # 返回错误消息，指出Agent ExecuteShellcode (userapc)调用提供的参数不足
        return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent ExecuteShellcode (userapc) call: %s", Args))

# 如果执行方法不是rtlcreateuserthread或userapc
default:
    # 返回错误消息，指出ExecuteShellcode方法无效
    return messages.ErrorMessage(fmt.Sprintf("invalide ExecuteShellcode method: %s", Args[1]))

# 如果options字典中有内容
if len(options) > 0:
    # 解析shellcode
    sh, errSh := shellcode.Parse(options)
// 如果解析 shellcode 出现错误，返回错误消息
if errSh != nil {
    m := fmt.Sprintf("there was an error parsing the shellcode:\r\n%s", errSh.Error())
    return messages.ErrorMessage(m)
}

// 向代理程序添加一个作业，并返回作业信息
job, err := agents.AddJob(agentID, sh[0], sh[1:])
if err != nil {
    return messages.ErrorMessage(err.Error())
}

// 返回代理程序的作业信息
return messages.JobMessage(agentID, job)
```

```
// 如果没有提供足够的参数来执行 Agent ExecuteShellcode 调用，返回错误消息
return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent ExecuteShellcode call: %s", Args))
```

```
// Kill 指令代理程序退出运行
func Kill(agentID uuid.UUID, Args []string) messages.UserMessage {
    // 如果提供了参数，向代理程序添加一个作业，并返回作业信息
    if len(Args) > 0 {
        job, err := agents.AddJob(agentID, "kill", Args[0:])
        if err != nil {
            return messages.ErrorMessage(err.Error())
    }
}
// 返回一个代理任务消息，其中包含代理ID和作业
func AgentKill(agentID uuid.UUID, Args []string) messages.UserMessage {
	// 如果参数数量大于1，则创建代理杀死作业
	if len(Args) > 1 {
		// 根据参数创建消息
		return messages.JobMessage(agentID, job)
	}
	// 如果参数不足，则返回错误消息
	return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent Kill call: %s", Args))
}

// LS 使用原生 Go 语言列出目录
func LS(agentID uuid.UUID, Args []string) messages.UserMessage {
	// 如果参数数量大于1
	if len(Args) > 1 {
		// 将参数连接成一个字符串
		arg := strings.Join(Args[0:], " ")
		// 解析参数
		argS, errS := shellwords.Parse(arg)
		// 如果解析出错，则返回错误消息
		if errS != nil {
			m := fmt.Sprintf("there was an error parsing command line argments: %s\r\n%s", Args, errS.Error())
			return messages.ErrorMessage(m)
		}
		// 向代理添加作业
		job, err := agents.AddJob(agentID, "ls", argS)
		// 如果出错，则返回错误消息
		if err != nil {
			return messages.ErrorMessage(err.Error())
		}
		// 返回代理任务消息
		return messages.JobMessage(agentID, job)
	}
}
	}
	// 向代理添加一个名为 "ls" 的作业，并返回作业信息
	job, err := agents.AddJob(agentID, "ls", Args)
	// 如果出现错误，返回错误信息
	if err != nil {
		return messages.ErrorMessage(err.Error())
	}
	// 返回代理的作业信息
	return messages.JobMessage(agentID, job)
}

// PWD 用于打印代理的当前工作目录
func PWD(agentID uuid.UUID, Args []string) messages.UserMessage {
	// 向代理添加一个名为 "pwd" 的作业，并返回作业信息
	job, err := agents.AddJob(agentID, "pwd", Args)
	// 如果出现错误，返回错误信息
	if err != nil {
		return messages.ErrorMessage(err.Error())
	}
	// 返回代理的作业信息
	return messages.JobMessage(agentID, job)
}

// SetJA3 用于更改代理的 JA3 签名
func SetJA3(agentID uuid.UUID, Args []string) messages.UserMessage {
	// 如果参数数量大于2
	if len(Args) > 2 {
	// 使用代理ID和参数添加一个作业
	job, err := agents.AddJob(agentID, "ja3", Args[1:])
	// 如果出现错误，返回错误消息
	if err != nil {
		return messages.ErrorMessage(err.Error())
	}
	// 返回代理ID和作业信息
	return messages.JobMessage(agentID, job)
}
// 如果参数不足两个，返回错误消息
return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent SetJA3 call: %s", Args))
}

// SetKillDate配置代理将停止运行的日期和时间
func SetKillDate(agentID uuid.UUID, Args []string) messages.UserMessage {
	// 如果参数数量大于2
	if len(Args) > 2 {
		// 尝试将参数转换为int64类型
		_, errU := strconv.ParseInt(Args[2], 10, 64)
		// 如果转换出错，返回错误消息
		if errU != nil {
			m := fmt.Sprintf("There was an error converting %s to an int64", Args[2])
			m = m + "\r\nKill date takes in a UNIX epoch timestamp such as 811123200 for September 15, 1995"
			return messages.ErrorMessage(m)
		}
		// 使用代理ID和参数添加一个作业
		job, err := agents.AddJob(agentID, "killdate", Args[1:])
		// 如果出现错误，返回错误消息
		if err != nil {
// 返回一个包含错误信息的 ErrorMessage 对象
return messages.ErrorMessage(err.Error())
// 返回一个包含代理 ID 和作业信息的 JobMessage 对象
return messages.JobMessage(agentID, job)
// 返回一个包含错误信息的 ErrorMessage 对象，如果设置最大重试次数时出现错误
return messages.ErrorMessage(err.Error())
// 返回一个包含代理 ID 和作业信息的 JobMessage 对象，如果成功设置最大重试次数
return messages.JobMessage(agentID, job)
// 返回一个包含错误信息的 ErrorMessage 对象，如果没有提供足够的参数来设置最大重试次数
return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent SetMaxRetry call: %s", Args))
// 设置最大重试次数，配置代理将尝试检查的次数，如果超过则退出
func SetMaxRetry(agentID uuid.UUID, Args []string) messages.UserMessage {
// 如果提供的参数数量大于 2
if len(Args) > 2 {
    // 添加一个作业，类型为 "maxretry"，参数为 Args[1:]
    job, err := agents.AddJob(agentID, "maxretry", Args[1:])
    // 如果添加作业时出现错误
    if err != nil {
        // 返回一个包含错误信息的 ErrorMessage 对象
        return messages.ErrorMessage(err.Error())
    }
    // 返回一个包含代理 ID 和作业信息的 JobMessage 对象
    return messages.JobMessage(agentID, job)
}
// 返回一个包含错误信息的 ErrorMessage 对象，如果没有提供足够的参数来设置最大重试次数
return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent SetMaxRetry call: %s", Args))
}
// 配置每条消息添加的随机填充的最大大小
// SetPadding函数用于设置Agent的填充操作，接受代理ID和参数列表作为输入，返回用户消息
func SetPadding(agentID uuid.UUID, Args []string) messages.UserMessage {
	// 如果参数数量大于2
	if len(Args) > 2 {
		// 调用agents.AddJob函数添加填充任务，并返回任务和错误信息
		job, err := agents.AddJob(agentID, "padding", Args[1:])
		// 如果添加任务出现错误，返回错误消息
		if err != nil {
			return messages.ErrorMessage(err.Error())
		}
		// 返回包含代理ID和任务的消息
		return messages.JobMessage(agentID, job)
	}
	// 如果参数数量不足2，返回错误消息
	return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent SetPadding call: %s", Args))
}

// SetSleep函数用于配置Agent在检查间隔之间的休眠时间
func SetSleep(agentID uuid.UUID, Args []string) messages.UserMessage {
	// 如果参数数量大于2
	if len(Args) > 2 {
		// 调用agents.AddJob函数添加休眠任务，并返回任务和错误信息
		job, err := agents.AddJob(agentID, "sleep", Args[1:])
		// 如果添加任务出现错误，返回错误消息
		if err != nil {
			return messages.ErrorMessage(err.Error())
		}
		// 返回包含代理ID和任务的消息
		return messages.JobMessage(agentID, job)
	}
}
// 返回一个错误消息，指示代理设置休眠调用时提供的参数不足
return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent SetSleep call: %s", Args))
}

// SetSkew配置代理使用的偏移量来随机化签入时间
func SetSkew(agentID uuid.UUID, Args []string) messages.UserMessage {
    // 如果参数数量大于2
	if len(Args) > 2 {
	    // 向代理添加一个名为"skew"的作业
		job, err := agents.AddJob(agentID, "skew", Args[1:])
		// 如果出现错误，返回错误消息
		if err != nil {
			return messages.ErrorMessage(err.Error())
		}
		// 返回作业消息
		return messages.JobMessage(agentID, job)
	}
	// 返回一个错误消息，指示代理设置偏移量调用时提供的参数不足
	return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent SetSkew call: %s", Args))
}

// Upload将文件从Merlin服务器传输到代理
func Upload(agentID uuid.UUID, Args []string) messages.UserMessage {
    // 如果参数数量大于等于3
	if len(Args) >= 3 {
	    // 将参数连接成一个字符串
		arg := strings.Join(Args[1:], " ")
		// 解析参数
		argS, errS := shellwords.Parse(arg)
		// 如果解析命令行参数时出现错误，返回错误消息
		if errS != nil {
			m := fmt.Sprintf("there was an error parsing command line argments: %s\r\n%s", Args, errS.Error())
			return messages.ErrorMessage(m)
		}
		// 如果参数的长度大于等于2
		if len(argS) >= 2 {
			// 检查第一个参数所指定的文件是否存在
			_, errF := os.Stat(argS[0])
			// 如果文件不存在，返回错误消息
			if errF != nil {
				m := fmt.Sprintf("there was an error accessing the source upload file:\r\n%s", errF.Error())
				return messages.ErrorMessage(m)
			}
			// 向代理服务器添加一个上传任务
			job, err := agents.AddJob(agentID, "upload", argS[0:2])
			// 如果添加任务时出现错误，返回错误消息
			if err != nil {
				return messages.ErrorMessage(err.Error())
			}
			// 返回包含代理ID和任务的消息
			return messages.JobMessage(agentID, job)
		}
	}
	// 如果参数不足，返回错误消息
	return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Agent Upload call: %s", Args))
}
```