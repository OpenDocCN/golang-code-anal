# `kubesploit\pkg\api\modules\modules.go`

```
// Kubesploit 是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd.

// Kubesploit 是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 其中包括许可证的第3版或任何以后的版本。

// Kubesploit被分发是希望它对增强组织的安全有用。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何担保；包括适销性或特定用途的隐含担保。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

// 模块包
package modules
// 导入标准库中的模块
import (
	"fmt" // 格式化输出
	"io/ioutil" // 读取文件内容
	"strings" // 字符串操作
	"time" // 时间操作

	// 导入自定义模块
	"kubesploit/pkg/agents" // 代理模块
	agentAPI "kubesploit/pkg/api/agents" // 代理 API
	"kubesploit/pkg/api/messages" // API 消息
	"kubesploit/pkg/modules" // 模块
)

// GetModuleListCompleter 返回一个用于 CLI 交互的可用模块的 tab 补全器
func GetModuleListCompleter() func(string) []string {
	return modules.GetModuleList() // 返回可用模块列表
}

// GetModule 根据 JSON 文件位置返回一个模块对象
// GetModule函数根据模块路径获取模块信息
func GetModule(modulePath string) (messages.UserMessage, modules.Module) {
	// 创建模块对象
	module, err := modules.Create(modulePath)
	// 如果创建过程中出现错误，返回错误消息和空模块
	if err != nil {
		return messages.ErrorMessage(err.Error()), modules.Module{}
	}
	// 返回成功消息和模块对象
	return messages.UserMessage{Error: false}, module
}

// RunModule函数执行提供的模块
func RunModule(module modules.Module) []messages.UserMessage {
	// 存储返回的消息
	var returnMessages []messages.UserMessage
	// 如果模块需要使用Go解释器或者加载脚本，则处理模块的命令
	if module.GoInterpreter == true || module.GoInterpreterProgress == true || module.LoadScriptFromPath == true {
		// 遍历模块的命令
		for index, command := range module.Commands {
			// 读取命令对应的文件内容，并将内容转换为字符串
			if content, err := ioutil.ReadFile(command); err == nil {
				module.Commands[index] = string(content)
			}
		}
	}

	// 运行模块
	r, err := modules.Run(module)
```
// 如果发生错误，将错误信息添加到返回消息中，并返回
if err != nil {
    returnMessages = append(returnMessages, messages.ErrorMessage(err.Error()))
    return returnMessages
}

// 如果返回的命令长度小于等于0，返回错误消息
if len(r) <= 0 {
    err := fmt.Errorf("the %s module did not return a command to task an agent with", module.Name)
    returnMessages = append(returnMessages, messages.ErrorMessage(err.Error()))
    return returnMessages
}

// TODO 将所有这些逻辑移动到 modules.Run() 函数中

// 如果模块的代理为 "ffffffff-ffff-ffff-ffff-ffffffffffff"
if strings.ToLower(module.Agent.String()) == "ffffffff-ffff-ffff-ffff-ffffffffffff" {
    // 如果没有可用的代理，返回错误消息
    if len(agents.Agents) <= 0 {
        err := fmt.Errorf("there are 0 available agents, no jobs were created")
        returnMessages = append(returnMessages, messages.ErrorMessage(err.Error()))
        return returnMessages
    }
    // 遍历代理列表
    for id := range agents.Agents {
        // 确保操作系统平台匹配
			// 如果模块平台与代理平台不匹配，则跳过当前任务并返回消息
			if !strings.EqualFold(agents.Agents[id].Platform, module.Platform) {
				m := fmt.Sprintf("Module platform %s does not match agent %s platform %s. Skipping job...",
					module.Platform, id, agents.Agents[id].Platform)
				um := messages.UserMessage{
					Error:   false,
					Level:   messages.Note,
					Message: m,
					Time:    time.Now().UTC(),
				}
				returnMessages = append(returnMessages, um)
				continue
			}
			// 根据模块类型进行不同的处理
			switch strings.ToLower(module.Type) {
			case "standard":
				// 如果模块需要使用 Go 解释器，则调用 agentAPI.CMD 方法发送命令
				if module.GoInterpreter{
					//returnMessages = append(returnMessages, agentAPI.CMDGO(id, r))
					returnMessages = append(returnMessages, agentAPI.CMD(id, r,"cmdgo"))
				} else if module.GoInterpreterProgress{
					// 如果模块需要使用带进度的 Go 解释器，则调用 agentAPI.CMD 方法发送命令
					//returnMessages = append(returnMessages, agentAPI.CMDGOPROG(id, r))
					returnMessages = append(returnMessages, agentAPI.CMD(id, r,"cmdgoprogress"))
				} else if module.LoadScriptFromPath{
					// 如果模块从路径加载脚本，则使用`cmd`消息类型，该类型必须在位置0
					// 将命令消息添加到返回消息列表中
					returnMessages = append(returnMessages, agentAPI.CMD(id, r,"cmdScriptFromPath"))
				} else {
					// 如果模块不是从路径加载脚本，则使用`cmd`消息类型，该类型必须在位置0
					// 将命令消息添加到返回消息列表中
					returnMessages = append(returnMessages, agentAPI.CMD(id, r,"cmd"))
				}
			case "extended":
				// 使用r[0]作为方法，将作业添加到代理中
				job, err := agents.AddJob(id, r[0], r[1:])
				// 如果出现错误，将错误消息添加到返回消息列表中
				if err != nil {
					returnMessages = append(returnMessages, messages.ErrorMessage(err.Error()))
				} else {
					// 将作业消息添加到返回消息列表中
					returnMessages = append(returnMessages, messages.JobMessage(id, job))
				}
			default:
				// 如果模块类型无效，创建相应的错误消息
				err := fmt.Errorf("invalid module type: %s", module.Type)
				returnMessages = append(returnMessages, messages.ErrorMessage(err.Error()))
// 返回消息列表
return returnMessages
// 单个代理
switch strings.ToLower(module.Type) {
    case "standard":
        // 如果是标准模块并且使用 Go 解释器
        if module.GoInterpreter {
            // 将命令添加到返回消息列表
            returnMessages = append(returnMessages, agentAPI.CMD(module.Agent, r,"cmdgo"))
        // 如果是标准模块并且使用 Go 解释器进度
        } else if module.GoInterpreterProgress {
            // 将命令添加到返回消息列表
            returnMessages = append(returnMessages, agentAPI.CMD(module.Agent, r,"cmdgoprogress"))
        // 如果是标准模块并且从路径加载脚本
        } else if module.LoadScriptFromPath {
            // 将命令添加到返回消息列表
            returnMessages = append(returnMessages, agentAPI.CMD(module.Agent,  r,"cmdScriptFromPath"))
        } else {
            // 标准模块使用 `cmd` 消息类型，必须在位置 0
// 如果模块类型为 "cmd"，则调用 agentAPI.CMD 函数执行命令，并将返回的消息添加到 returnMessages 中
// 如果模块类型为 "extended"，则调用 agents.AddJob 函数添加作业，并将返回的消息添加到 returnMessages 中
// 如果模块类型不是 "cmd" 也不是 "extended"，则返回错误消息
// 返回最终的 returnMessages
```