# `kubesploit\pkg\api\modules\modules.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。请参阅GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本。
// 如果没有，请参见<http://www.gnu.org/licenses/>。

package modules

import (
    // 标准库
    "fmt"
    "io/ioutil"
    "strings"
    "time"

    // Merlin
    "kubesploit/pkg/agents"
    agentAPI "kubesploit/pkg/api/agents"
    "kubesploit/pkg/api/messages"
    "kubesploit/pkg/modules"
)

// GetModuleListCompleter返回一个用于CLI交互的可用模块的Tab补全器
func GetModuleListCompleter() func(string) []string {
    return modules.GetModuleList()
}

// 根据其JSON文件位置返回一个模块对象
func GetModule(modulePath string) (messages.UserMessage, modules.Module) {
    module, err := modules.Create(modulePath)
    if err != nil {
        return messages.ErrorMessage(err.Error()), modules.Module{}
    }
    return messages.UserMessage{Error: false}, module
}

// RunModule执行提供的模块
func RunModule(module modules.Module) []messages.UserMessage {
    var returnMessages []messages.UserMessage
    # 如果模块的GoInterpreter、GoInterpreterProgress或LoadScriptFromPath为真
    if module.GoInterpreter == true || module.GoInterpreterProgress == true || module.LoadScriptFromPath == true {
        # 遍历模块的命令列表
        for index, command := range module.Commands {
            # 读取命令文件的内容
            if content, err := ioutil.ReadFile(command); err == nil {
                # 将文件内容转换为字符串并替换原来的命令
                module.Commands[index] = string(content)
            }
        }
    }

    # 运行模块
    r, err := modules.Run(module)
    # 如果出现错误，返回错误消息
    if err != nil {
        returnMessages = append(returnMessages, messages.ErrorMessage(err.Error()))
        return returnMessages
    }
    # 如果返回结果为空，返回错误消息
    if len(r) <= 0 {
        err := fmt.Errorf("the %s module did not return a command to task an agent with", module.Name)
        returnMessages = append(returnMessages, messages.ErrorMessage(err.Error()))
        return returnMessages
    }

    # TODO 将所有这些逻辑移动到modules.Run()函数中
    # 所有代理
    }
    # 单个代理
    switch strings.ToLower(module.Type) {
    # 根据不同的模块类型执行不同的操作
    case "standard":
        # 如果模块支持 Go 解释器，则执行相应的命令
        if module.GoInterpreter{
            returnMessages = append(returnMessages, agentAPI.CMD(module.Agent, r,"cmdgo"))
        } else if module.GoInterpreterProgress{
            # 如果模块支持带进度的 Go 解释器，则执行相应的命令
            returnMessages = append(returnMessages, agentAPI.CMD(module.Agent, r,"cmdgoprogress"))
        } else if module.LoadScriptFromPath:
            # 如果模块支持从路径加载脚本，则执行相应的命令
            returnMessages = append(returnMessages, agentAPI.CMD(module.Agent,  r,"cmdScriptFromPath"))
        else:
            # 否则执行默认的命令
            returnMessages = append(returnMessages, agentAPI.CMD(module.Agent,  r,"cmd"))
    case "extended":
        # 对于扩展模块，添加作业并返回相应消息
        job, err := agents.AddJob(module.Agent, r[0], r[1:])
        if err != nil:
            # 如果添加作业出错，则返回错误消息
            returnMessages = append(returnMessages, messages.ErrorMessage(err.Error()))
        else:
            # 否则返回作业消息
            returnMessages = append(returnMessages, messages.JobMessage(module.Agent, job))
        return returnMessages
    default:
        # 对于无效的模块类型，返回错误消息
        err := fmt.Errorf("invalid module type: %s", module.Type)
        returnMessages = append(returnMessages, messages.ErrorMessage(err.Error()))
        return returnMessages
    }
    return returnMessages
# 闭合前面的函数定义
```