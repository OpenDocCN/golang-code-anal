# `kubesploit\pkg\cli\cli.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 无论是许可证的第3版还是任何以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。请参阅GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本。
// 如果没有，请访问<http://www.gnu.org/licenses/>。

// 包声明
package cli
# 导入标准库中的模块
import (
	"bufio"  # 用于缓冲 I/O
	"fmt"  # 用于格式化输出
	"gopkg.in/yaml.v2"  # 用于处理 YAML 格式的数据
	"io"  # 提供了基本的 I/O 接口
	"io/ioutil"  # 用于读取文件内容
	"log"  # 用于记录日志
	"os"  # 提供了操作系统功能
	"os/exec"  # 用于执行外部命令
	"path"  # 用于处理文件路径
	"strings"  # 用于处理字符串
	"time"  # 提供了时间相关的功能

	# 第三方库
	"github.com/chzyer/readline"  # 用于实现命令行交互
	"github.com/fatih/color"  # 用于在终端中输出带颜色的文本
	"github.com/olekukonko/tablewriter"  # 用于在终端中输出表格
	"github.com/satori/go.uuid"  # 用于生成 UUID

	# Merlin
	# 这里是一个空注释，可能是为了将 Merlin 模块与第三方库模块分隔开
// 导入所需的包
"kubesploit/pkg" // 导入pkg包
"kubesploit/pkg/agents" // 导入agents包
agentAPI "kubesploit/pkg/api/agents" // 导入api/agents包并命名为agentAPI
listenerAPI "kubesploit/pkg/api/listeners" // 导入api/listeners包并命名为listenerAPI
"kubesploit/pkg/api/messages" // 导入api/messages包
moduleAPI "kubesploit/pkg/api/modules" // 导入api/modules包
"kubesploit/pkg/banner" // 导入banner包
"kubesploit/pkg/core" // 导入core包
"kubesploit/pkg/logging" // 导入logging包
"kubesploit/pkg/modules" // 导入modules包
"kubesploit/pkg/servers" // 导入servers包

// 全局变量
var shellModule modules.Module // 定义名为shellModule的全局变量，类型为modules.Module
var shellAgent uuid.UUID // 定义名为shellAgent的全局变量，类型为uuid.UUID
var shellListener listener // 定义名为shellListener的全局变量，类型为listener
var shellListenerOptions map[string]string // 定义名为shellListenerOptions的全局变量，类型为map，键为string类型，值为string类型
var prompt *readline.Instance // 定义名为prompt的全局变量，类型为*readline.Instance
var shellCompleter *readline.PrefixCompleter // 定义名为shellCompleter的全局变量，类型为*readline.PrefixCompleter
// 定义变量 shellMenuContext，用于存储当前的菜单上下文

// MessageChannel 用于输入用户消息，最终会被写入到 CLI 应用程序的 STDOUT
var MessageChannel = make(chan messages.UserMessage) // 创建一个用于存储用户消息的通道
var clientID = uuid.NewV4() // 生成一个新的 UUID 作为客户端的 ID

// 定义结构体 YamListener，用于存储 YAML 配置文件中的监听器信息
type YamListener struct {
    Name string `yaml:"Name"` // 监听器的名称
    Protocol string `yaml:"Protocol"` // 监听器的协议
    Interface string `yaml:"Interface"` // 监听器的接口
    Port string `yaml:"Port"` // 监听器的端口
}

// 定义结构体 YamlConfig，用于存储整个 YAML 配置文件的信息
type YamlConfig struct {
    AutoStart string `yaml:"AutoStart"` // 自动启动配置
    AutoSetAllAgents string `yaml:"AutoSetAllAgents"` // 自动设置所有代理配置
    Listeners []YamListener `yaml:"Listeners"` // 监听器列表
}
var GlobalYamlConfig YamlConfig // 定义全局变量 GlobalYamlConfig，用于存储 YAML 配置信息
// 默认的 YAML 文件路径
const DEFAULT_YAML_FILE = "./config.yaml"

// 初始化 YAML 文件
func initializeYamlFile() error {
	// 设置 YAML 文件路径
	yamlFile := DEFAULT_YAML_FILE

	// 读取 YAML 文件的内容
	yamlFileBytes, err := ioutil.ReadFile(yamlFile)
	// 打印读取 YAML 文件的消息
	//fmt.Printf("[*] Read YAML file\n")
	// 创建用户消息
	m := fmt.Sprintf("Reading config YAML file")
	um := messages.UserMessage{
		Level:   messages.Success,
		Time:    time.Now().UTC(),
		Message: m,
		Error:   false,
	}

	// 将用户消息发送到消息通道
	MessageChannel <- um

	// 如果读取出错，打印错误消息
	if err != nil {
		//fmt.Printf("[*] Failed to read YAML file: %s\n", err)
		m := fmt.Sprintf("Failed to read YAML file: %s\n", err)
		// 发送广播消息，包括消息级别、消息内容、时间和错误信息
		messages.SendBroadcastMessage(messages.UserMessage{
			Level:   messages.Note,
			Message: m,
			Time:    time.Now().UTC(),
			Error:   false,
		})

	} else {
		// 打印解析 YAML 文件的提示信息
		// m 变量存储解析配置 YAML 文件的消息
		m := fmt.Sprintf("Parsing config YAML file")
		// 创建用户消息对象，包括消息级别、时间、消息内容和错误信息
		um := messages.UserMessage{
			Level:   messages.Success,
			Time:    time.Now().UTC(),
			Message: m,
			Error:   false,
		}
		// 将用户消息发送到消息通道
		MessageChannel <- um
		// 使用 yaml.Unmarshal 解析 YAML 文件的字节流，并将结果存储到 GlobalYamlConfig 变量中
		err = yaml.Unmarshal(yamlFileBytes, &GlobalYamlConfig)
		// 如果解析出错，处理错误
		if err != nil {
			// 打印错误信息到标准输出
			// m := fmt.Sprintf("Failed to parse YAML file: %s\n", err)
			// 将错误信息封装成消息结构体并发送广播消息
			// messages.SendBroadcastMessage(messages.UserMessage{
			// 	Level:   messages.Note,
			// 	Message: m,
			// 	Time:    time.Now().UTC(),
			// 	Error:   false,
			// })
		}
	}

	return err
}

func loadYamlConfigFile() {
	// 初始化 YAML 文件
	err := initializeYamlFile()
	// 如果初始化成功
	if err == nil {
		// 如果全局 YAML 配置中的 AutoStart 字段为 true
		if strings.ToLower(GlobalYamlConfig.AutoStart) == "true" {
			// 遍历全局 YAML 配置中的监听器列表
			for _, yamlListener := range GlobalYamlConfig.Listeners{
				// 获取监听器选项
				shellListenerOptions = listenerAPI.GetListenerOptions(yamlListener.Protocol)
# 将yamlListener中的Protocol赋值给shellListenerOptions中的Protocol
shellListenerOptions["Protocol"] = yamlListener.Protocol
# 将yamlListener中的Interface赋值给shellListenerOptions中的Interface
shellListenerOptions["Interface"] = yamlListener.Interface
# 将yamlListener中的Port赋值给shellListenerOptions中的Port
shellListenerOptions["Port"] = yamlListener.Port
# 将yamlListener中的Name赋值给shellListenerOptions中的Name
shellListenerOptions["Name"] = yamlListener.Name

# 使用shellListenerOptions创建新的Listener，并将返回的um和id分别赋值给um和id
um, id := listenerAPI.NewListener(shellListenerOptions)
# 将um发送到MessageChannel中
MessageChannel <- um
# 如果um中包含错误信息，则直接返回
if um.Error {
    return
}

# 如果id为uuid.Nil，则向MessageChannel发送包含警告信息的UserMessage，并直接返回
if id == uuid.Nil {
    MessageChannel <- messages.UserMessage{
        Level:   messages.Warn,
        Message: "a nil Listener UUID was returned",
        Time:    time.Time{},
        Error:   true,
    }
    return
}
// 创建一个监听器对象，包括id和name属性
shellListener = listener{id: id, name: shellListenerOptions["Name"]}
// 启动监听器，并将启动消息发送到消息通道
startMessage := listenerAPI.Start(shellListener.name)
MessageChannel <- startMessage
// 获取监听器配置选项
um, _ = listenerAPI.GetListenerConfiguredOptions(shellListener.id)
// 如果出现错误，将错误消息发送到消息通道
if um.Error {
    MessageChannel <- um
}

// Shell是启动命令行界面的导出函数
func Shell() {
    // 获取命令行自动补全器
    shellCompleter = getCompleter("main")
    // 打印用户消息
    printUserMessage()
    // 注册消息通道
    registerMessageChannel()
}
# 调用函数getUserMessages()，获取用户消息

# 使用readline.NewEx()创建一个新的readline实例，并配置其属性
# Prompt: 设置命令行提示符
# HistoryFile: 设置历史记录文件路径
# AutoComplete: 设置自动补全函数
# InterruptPrompt: 设置中断提示符
# EOFPrompt: 设置退出提示符
# HistorySearchFold: 设置历史搜索折叠
# FuncFilterInputRune: 设置输入过滤函数

# 检查是否有错误发生，如果有则向消息通道发送警告消息
# Level: 消息级别为警告
# Message: 消息内容为提供的输入出现错误
# Time: 消息发送时间
# Error: 标记为错误消息

# 将变量 p 赋值给 prompt
prompt = p

# 延迟执行的匿名函数，用于关闭 prompt
defer func() {
    err := prompt.Close()
    if err != nil {
        log.Fatal(err)
    }
}()

# 加载 YAML 配置文件
loadYamlConfigFile()

# 设置日志输出到 prompt 的标准错误流
log.SetOutput(prompt.Stderr())

# 进入循环，等待用户输入
for {
    // 读取用户输入的一行
    line, err := prompt.Readline()
    // 如果用户输入中断信号
    if err == readline.ErrInterrupt {
        // 如果输入为空，则退出循环
        if len(line) == 0 {
            break
        } else {
            // 如果输入不为空，则继续循环
            continue
        }
		} else if err == io.EOF {
			// 如果发生文件结束错误，退出程序
			exit()
		}

		// 去除字符串两端的空白字符
		line = strings.TrimSpace(line)
		// 将字符串按空格分割成切片
		cmd := strings.Fields(line)

		// 如果切片长度大于0
		if len(cmd) > 0 {
			// 根据不同的上下文执行不同的命令
			switch shellMenuContext {
			case "listener":
				// 执行监听器菜单命令
				menuListener(cmd)
			case "listenersmain":
				// 执行主监听器菜单命令
				menuListeners(cmd)
			case "listenersetup":
				// 执行监听器设置菜单命令
				menuListenerSetup(cmd)
			case "main":
				// 根据第一个命令执行不同的操作
				switch cmd[0] {
				case "agent":
					// 如果命令长度大于1，执行代理菜单命令
					if len(cmd) > 1 {
						menuAgent(cmd[1:])
				// 如果用户输入的命令是 "banner"，则显示 Kubesploit 的横幅信息
				case "banner":
					// 初始化一个包含换行符的字符串
					m := "\n"
					// 将 Kubesploit 的横幅信息添加到字符串中
					m += color.BlueString(banner.KubesploitBanner)
					// 将 Kubesploit 的版本信息添加到字符串中
					m += color.BlueString("\r\n\t\t   Version: %s", kubesploitVersion.Version)
					// 将 Kubesploit 的构建信息添加到字符串中
					m += color.BlueString("\r\n\t\t   Build: %s\n", kubesploitVersion.Build)
					// 将消息发送到消息通道
					MessageChannel <- messages.UserMessage{
						Level:   messages.Plain,
						Message: m,
						Time:    time.Now().UTC(),
						Error:   false,
					}
				// 如果用户输入的命令是 "help" 或者 "?"，则显示帮助菜单
				case "help":
				case "?":
					// 调用显示主帮助菜单的函数
					menuHelpMain()
				// 如果用户输入的命令是 "exit" 或者 "quit"，则退出程序
				case "exit", "quit":
					// 如果用户输入了额外的参数 "-y"，则立即退出程序
					if len(cmd) > 1 {
						if strings.ToLower(cmd[1]) == "-y" {
							exit()
# 如果用户输入的命令是 "exit"，则询问用户是否确认退出，如果确认则退出程序
if cmd[0] == "exit":
    if confirm("Are you sure you want to exit?"):
        exit()

# 如果用户输入的命令是 "interact"，并且命令长度大于1，则将命令和参数组成一个列表，然后调用menuAgent函数
elif cmd[0] == "interact":
    if len(cmd) > 1:
        i := []string{"interact"}
        i = append(i, cmd[1])
        menuAgent(i)

# 如果用户输入的命令是 "listeners"，则设置shellMenuContext为"listenersmain"，配置自动补全，设置提示符
elif cmd[0] == "listeners":
    shellMenuContext = "listenersmain"
    prompt.Config.AutoComplete = getCompleter("listenersmain")
    prompt.SetPrompt("\033[31mkubesploit[\033[32mlisteners\033[31m]»\033[0m ")

# 如果用户输入的命令是 "remove"，并且命令长度大于1，则将命令和参数组成一个列表，然后调用menuAgent函数
elif cmd[0] == "remove":
    if len(cmd) > 1:
        i := []string{"remove"}
        i = append(i, cmd[1])
        menuAgent(i)
					}
				// 如果命令是 "sessions"，则调用 menuAgent 函数并传入参数列表 ["list"]
				case "sessions":
					menuAgent([]string{"list"})
				// 如果命令是 "set" 且参数数量大于2
				case "set":
					if len(cmd) > 2 {
						// 根据第二个参数进行不同的操作
						switch cmd[1] {
						// 如果第二个参数是 "verbose"
						case "verbose":
							// 如果第三个参数是 "true"，则设置核心的 Verbose 属性为 true，并发送成功消息到消息通道
							if strings.ToLower(cmd[2]) == "true" {
								core.Verbose = true
								MessageChannel <- messages.UserMessage{
									Level:   messages.Success,
									Message: "Verbose output enabled",
									Time:    time.Now(),
									Error:   false,
								}
							// 如果第三个参数是 "false"，则设置核心的 Verbose 属性为 false，并发送成功消息到消息通道
							} else if strings.ToLower(cmd[2]) == "false" {
								core.Verbose = false
								MessageChannel <- messages.UserMessage{
									Level:   messages.Success,
									Message: "Verbose output disabled",
# 根据命令行参数设置调试模式的开启或关闭
case "debug":
    # 如果命令行参数为 true，则开启调试模式
    if strings.ToLower(cmd[2]) == "true":
        core.Debug = true
        # 发送成功消息到消息通道，表示调试模式已开启
        MessageChannel <- messages.UserMessage{
            Level:   messages.Success,
            Message: "Debug output enabled",
            Time:    time.Now().UTC(),
            Error:   false,
        }
    # 如果命令行参数为 false，则关闭调试模式
    elif strings.ToLower(cmd[2]) == "false":
        core.Debug = false
        # 发送成功消息到消息通道，表示调试模式已关闭
        MessageChannel <- messages.UserMessage{
            Level:   messages.Success,
            Message: "Debug output disabled",
            Time:    time.Now().UTC(),
            Error:   false,
        }
				}
			}
		}
	}
case "use":
	menuUse(cmd[1:])  # 如果命令是"use"，则调用menuUse函数并传入参数cmd[1:]
case "version":
	MessageChannel <- messages.UserMessage{  # 发送用户消息到MessageChannel
		Level:   messages.Plain,  # 消息级别为普通
		Message: color.BlueString("Merlin version: %s\n", kubesploitVersion.Version),  # 消息内容为带有蓝色的版本信息
		Time:    time.Now().UTC(),  # 消息发送时间
		Error:   false,  # 没有错误发生
	}
case "":  # 如果命令为空
default:  # 如果命令不是以上任何一种情况
	if len(cmd) > 1:  # 如果命令长度大于1
		executeCommand(cmd[0], cmd[1:])  # 执行executeCommand函数并传入参数cmd[0]和cmd[1:]
	else:  # 如果命令长度不大于1
		var x []string  # 创建一个空字符串数组x
		executeCommand(cmd[0], x)  # 执行executeCommand函数并传入参数cmd[0]和x
					}
				}
			// 处理命令为"module"的情况
			case "module":
				// 根据命令的第一个参数进行不同的处理
				switch cmd[0] {
				// 处理命令为"show"的情况
				case "show":
					// 如果命令长度大于1
					if len(cmd) > 1 {
						// 根据命令的第二个参数进行不同的处理
						switch cmd[1] {
						// 处理命令为"info"的情况
						case "info":
							// 调用shellModule的ShowInfo方法
							shellModule.ShowInfo()
						// 处理命令为"options"的情况
						case "options":
							// 调用shellModule的ShowOptions方法
							shellModule.ShowOptions()
						}
					}
				// 处理命令为"info"的情况
				case "info":
					// 调用shellModule的ShowInfo方法
					shellModule.ShowInfo()
				// 处理命令为"set"的情况
				case "set":
					// 如果命令长度大于2
					if len(cmd) > 2 {
						// 如果命令的第一个参数为"Agent"
						if cmd[1] == "Agent" {
							// 调用shellModule的SetAgent方法设置Agent，并获取返回的状态和错误信息
							s, err := shellModule.SetAgent(cmd[2])
							// 如果有错误发生
							if err != nil {
# 如果发生错误，向消息通道发送警告级别的用户消息，包括错误信息和时间
MessageChannel <- messages.UserMessage{
    Level:   messages.Warn,
    Message: err.Error(),
    Time:    time.Now().UTC(),
    Error:   true,
}
# 如果没有发生错误，向消息通道发送成功级别的用户消息，包括消息内容和时间
MessageChannel <- messages.UserMessage{
    Level:   messages.Success,
    Message: s,
    Time:    time.Now().UTC(),
    Error:   false,
}
# 如果命令不是"set"，则执行命令并获取结果
s, err := shellModule.SetOption(cmd[1], cmd[2:])
# 如果发生错误，向消息通道发送警告级别的用户消息，包括错误信息
MessageChannel <- messages.UserMessage{
    Level:   messages.Warn,
    Message: err.Error(),
								// 设置时间为当前的协调世界时
								Time:    time.Now().UTC(),
								// 设置错误标志为true
								Error:   true,
							}
						} else {
							// 发送成功级别的用户消息到消息通道
							MessageChannel <- messages.UserMessage{
								Level:   messages.Success,
								// 设置消息内容为s
								Message: s,
								// 设置时间为当前的协调世界时
								Time:    time.Now().UTC(),
								// 设置错误标志为false
								Error:   false,
							}
						}
					}
				}
			case "reload":
				// 调用menuSetModule函数，参数为去除后缀的shellModule.Path连接成的字符串
				menuSetModule(strings.TrimSuffix(strings.Join(shellModule.Path, "/"), ".json"))
			case "run":
				// 调用moduleAPI.RunModule函数，参数为shellModule，并将返回的消息逐个发送到消息通道
				modMessages := moduleAPI.RunModule(shellModule)
				for _, message := range modMessages {
					MessageChannel <- message
				}
# 根据命令执行不同的操作
case "back", "main":
    # 调用menuSetMain函数
    menuSetMain()
case "exit", "quit":
    # 如果命令长度大于1且第二个参数为"-y"，则直接退出程序
    if len(cmd) > 1 {
        if strings.ToLower(cmd[1]) == "-y" {
            exit()
        }
    }
    # 如果确认要退出程序，则退出
    if confirm("Are you sure you want to exit?") {
        exit()
    }
case "unset":
    # 如果命令长度大于等于2，调用shellModule.SetOption函数将指定选项设置为nil
    if len(cmd) >= 2 {
        s, err := shellModule.SetOption(cmd[1], nil)
        # 如果出现错误，向消息通道发送警告消息
        if err != nil {
            MessageChannel <- messages.UserMessage{
                Level:   messages.Warn,
                Message: err.Error(),
                Time:    time.Now().UTC(),
                Error:   true,
						}
					} else {
						// 如果不是特殊命令，则发送用户消息到消息通道
						MessageChannel <- messages.UserMessage{
							Level:   messages.Success,
							Message: s,
							Time:    time.Now().UTC(),
							Error:   false,
						}
					}
				}
				// 处理特殊命令 "?", "help"
				case "?", "help":
					// 调用菜单帮助模块
					menuHelpModule()
				default:
					// 如果命令长度大于1，则执行命令
					if len(cmd) > 1 {
						executeCommand(cmd[0], cmd[1:])
					} else {
						// 否则执行命令，传入空字符串数组
						var x []string
						executeCommand(cmd[0], x)
					}
				}
# 根据命令类型进行不同的操作
case "agent":
    # 根据命令进行不同的操作
    switch cmd[0]:
        # 返回主菜单
        case "back":
            menuSetMain()
        # 切换目录
        case "cd":
            MessageChannel <- agentAPI.CD(shellAgent, cmd)
        # 执行命令
        case "cmd", "shell":
            MessageChannel <- agentAPI.CMD(shellAgent, cmd[1:],"cmd")
        # 下载文件
        case "download":
            MessageChannel <- agentAPI.Download(shellAgent, cmd)
        # 执行 shellcode
        case "execute-shellcode":
            MessageChannel <- agentAPI.ExecuteShellcode(shellAgent, cmd)
        # 退出程序
        case "exit", "quit":
            # 如果命令中包含参数 "-y"，则直接退出
            if len(cmd) > 1:
                if strings.ToLower(cmd[1]) == "-y":
                    exit()
            # 否则确认用户是否要退出
            if confirm("Are you sure you want to exit?"):
                exit()
				// 如果用户输入了 "?" 或 "help"，显示菜单帮助
				case "?", "help":
					menuHelpAgent()
				// 如果用户输入了 "info"，显示代理信息
				case "info":
					agents.ShowInfo(shellAgent)
				// 如果用户输入了 "kill"，设置主菜单并发送杀死代理的命令
					menuSetMain()
					MessageChannel <- agentAPI.Kill(shellAgent, cmd)

				// 如果用户输入了 "ls"，发送列出文件列表的命令
				case "ls":
					MessageChannel <- agentAPI.LS(shellAgent, cmd)
				// 如果用户输入了 "main"，设置主菜单
				case "main":
					menuSetMain()
				// 如果用户输入了 "pwd"，发送获取当前路径的命令
				case "pwd":
					MessageChannel <- agentAPI.PWD(shellAgent, cmd)
				// 如果用户输入了 "set"，并且输入了参数
				case "set":
					if len(cmd) > 1 {
						// 根据第二个参数的值发送相应的设置命令
						switch cmd[1] {
						case "ja3":
							MessageChannel <- agentAPI.SetJA3(shellAgent, cmd)
# 根据命令类型执行相应的操作，并将结果发送到消息通道
switch cmd[0]:
    # 如果命令类型是设置killdate，则调用agentAPI.SetKillDate方法，并将结果发送到消息通道
    case "killdate":
        MessageChannel <- agentAPI.SetKillDate(shellAgent, cmd)
    # 如果命令类型是设置maxretry，则调用agentAPI.SetMaxRetry方法，并将结果发送到消息通道
    case "maxretry":
        MessageChannel <- agentAPI.SetMaxRetry(shellAgent, cmd)
    # 如果命令类型是设置padding，则调用agentAPI.SetPadding方法，并将结果发送到消息通道
    case "padding":
        MessageChannel <- agentAPI.SetPadding(shellAgent, cmd)
    # 如果命令类型是设置sleep，则调用agentAPI.SetSleep方法，并将结果发送到消息通道
    case "sleep":
        MessageChannel <- agentAPI.SetSleep(shellAgent, cmd)
    # 如果命令类型是设置skew，则调用agentAPI.SetSkew方法，并将结果发送到消息通道
    case "skew":
        MessageChannel <- agentAPI.SetSkew(shellAgent, cmd)
    # 如果命令类型不在上述类型中，则发送警告消息到消息通道
    default:
        MessageChannel <- messages.UserMessage{
            Level:   messages.Warn,
            Message: fmt.Sprintf("invalid option to set: %s", cmd[1]),
            Time:    time.Time{},
            Error:   true,
        }
    # 如果命令类型是status，则执行相应的操作
    case "status":
# 获取指定代理的状态
status := agents.GetAgentStatus(shellAgent)
# 如果代理状态为"Active"，向消息通道发送活跃代理的消息
if status == "Active" {
    MessageChannel <- messages.UserMessage{
        Level:   messages.Plain,
        Message: color.GreenString("%s agent is active\n", shellAgent),
        Time:    time.Now().UTC(),
        Error:   false,
    }
# 如果代理状态为"Delayed"，向消息通道发送延迟代理的消息
} else if status == "Delayed" {
    MessageChannel <- messages.UserMessage{
        Level:   messages.Plain,
        Message: color.YellowString("%s agent is delayed\n", shellAgent),
        Time:    time.Now().UTC(),
        Error:   false,
    }
# 如果代理状态为"Dead"，向消息通道发送失效代理的消息
} else if status == "Dead" {
    MessageChannel <- messages.UserMessage{
        Level:   messages.Plain,
        Message: color.RedString("%s agent is dead\n", shellAgent),
        Time:    time.Now().UTC(),
// 如果命令是 "status"，则向消息通道发送代理状态消息
if cmd[0] == "status" {
    // 如果代理状态是 "active"，则发送普通级别的消息
    if status == "active" {
        MessageChannel <- messages.UserMessage{
            Level:   messages.Plain,
            Message: color.BlueString("%s agent is %s\n", shellAgent, status),
            Time:    time.Now().UTC(),
            Error:   false,
        }
    } else {
        // 如果代理状态不是 "active"，则发送普通级别的消息
        MessageChannel <- messages.UserMessage{
            Level:   messages.Plain,
            Message: color.BlueString("%s agent is %s\n", shellAgent, status),
            Time:    time.Now().UTC(),
            Error:   false,
        }
    }
} 
// 如果命令是 "upload"，则向消息通道发送上传命令的结果
else if cmd[0] == "upload" {
    MessageChannel <- agentAPI.Upload(shellAgent, cmd)
} 
// 如果命令不是 "status" 或 "upload"，则执行相应的命令
else {
    if len(cmd) > 1 {
        executeCommand(cmd[0], cmd[1:])
    } else {
        executeCommand(cmd[0], []string{})
    }
}
		}

	}
}

func menuUse(cmd []string) {
	// 检查命令参数是否大于0
	if len(cmd) > 0 {
		// 根据命令参数进行不同的操作
		switch cmd[0] {
		case "module":
			// 如果命令参数大于1，调用menuSetModule函数
			if len(cmd) > 1 {
				menuSetModule(cmd[1])
			} else {
				// 如果命令参数不合法，发送警告消息到消息通道
				MessageChannel <- messages.UserMessage{
					Level:   messages.Warn,
					Message: "Invalid module",
					Time:    time.Now().UTC(),
					Error:   false,
				}
			}
		case "":
// 如果命令不匹配任何情况，向消息通道发送用户消息，表示无效的'use'命令
default:
    MessageChannel <- messages.UserMessage{
        Level:   messages.Note,
        Message: "Invalid 'use' command",
        Time:    time.Now().UTC(),
        Error:   false,
    }
}
// 如果命令不为空，向消息通道发送用户消息，表示无效的'use'命令
} else {
    MessageChannel <- messages.UserMessage{
        Level:   messages.Note,
        Message: "Invalid 'use' command",
        Time:    time.Now().UTC(),
        Error:   false,
    }
}

// 根据命令的第一个参数进行不同的操作
func menuAgent(cmd []string) {
switch cmd[0] {
# 根据命令行参数 "list" 执行相应的操作
case "list":
    # 创建一个新的表格写入器，并将表头设置为指定的字段
    table := tablewriter.NewWriter(os.Stdout)
    table.SetHeader([]string{"Agent GUID", "Platform", "User", "Host", "Transport", "Status"})
    table.SetAlignment(tablewriter.ALIGN_CENTER)
    # 遍历代理对象的 Agents 字段
    for k, v := range agents.Agents {
        # 将代理的传输协议转换为用户友好的字符串
        var proto string
        switch v.Proto {
            case "http":
                proto = "HTTP/1.1 clear-text"
            case "https":
                proto = "HTTP/1.1 over TLS"
            case "h2c":
                proto = "HTTP/2 clear-text"
            case "h2":
                proto = "HTTP/2 over TLS"
            case "http3":
                proto = "HTTP/3 (HTTP/2 over QUIC)"
            default:
                proto = fmt.Sprintf("Unknown: %s", v.Proto)
		}

		// 将数据添加到表格中，包括键、平台/架构、用户名、主机名、协议和代理状态
		table.Append([]string{k.String(), v.Platform + "/" + v.Architecture, v.UserName,
			v.HostName, proto, agents.GetAgentStatus(k)})
	}
	// 打印空行
	fmt.Println()
	// 渲染表格
	table.Render()
	// 打印空行
	fmt.Println()
case "interact":
	// 如果命令长度大于1
	if len(cmd) > 1 {
		// 将字符串转换为 UUID
		i, errUUID := uuid.FromString(cmd[1])
		// 如果转换出错
		if errUUID != nil {
			// 向消息通道发送警告消息
			MessageChannel <- messages.UserMessage{
				Level:   messages.Warn,
				Message: fmt.Sprintf("There was an error interacting with agent %s", cmd[1]),
				Time:    time.Now().UTC(),
				Error:   true,
			}
		} else {
			// 设置菜单中的代理
			menuSetAgent(i)
		}
	}
	// 根据命令类型进行处理
	case "remove":
		// 如果命令参数大于1
		if len(cmd) > 1 {
			// 如果命令参数为"all"
			if cmd[1] == "all" {
				// 遍历所有代理并根据UUID删除
				for _, v := range agents.Agents {
					removeAgentByUUID(v.ID.String(), v.ID)
				}
			} else {
				// 将命令参数转换为UUID
				i, errUUID := uuid.FromString(cmd[1])
				// 如果转换出错
				if errUUID != nil {
					// 发送警告消息
					MessageChannel <- messages.UserMessage{
						Level:   messages.Warn,
						Message: fmt.Sprintf("There was an error interacting with agent %s", cmd[1]),
						Time:    time.Now().UTC(),
						Error:   true,
					}
				} else {
					// 根据UUID删除代理
					removeAgentByUUID(cmd[1], i)
				}
// 根据 agentIDStr 和 agentID 删除代理
func removeAgentByUUID(agentIDStr string, agentID uuid.UUID){
	// 调用 agents 包中的 RemoveAgent 方法删除代理
	errRemove := agents.RemoveAgent(agentID)
	// 如果删除出错，向消息通道发送警告消息
	if errRemove != nil {
		MessageChannel <- messages.UserMessage{
			Level:   messages.Warn,
			Message: errRemove.Error(),
			Time:    time.Now().UTC(),
			Error:   true,
		}
	} else {
		// 如果删除成功，向消息通道发送代理已被移除的信息
		m := fmt.Sprintf("Agent %s was removed from the server at %s",
			agentIDStr, time.Now().UTC().Format(time.RFC3339))
		MessageChannel <- messages.UserMessage{
			Level:   messages.Info,
			Message: m,
		// 获取当前时间并转换为UTC时间
		Time:    time.Now().UTC(),
		// 设置错误标志为false
		Error:   false,
	}
}

// 设置代理人
func menuSetAgent(agentID uuid.UUID) {
	// 遍历代理人列表
	for k := range agents.Agents {
		// 如果代理人ID匹配，则设置shellAgent为该代理人ID
		if agentID == agents.Agents[k].ID {
			shellAgent = agentID
			// 设置命令提示符
			prompt.Config.AutoComplete = getCompleter("agent")
			prompt.SetPrompt("\033[31mkubesploit[\033[32magent\033[31m][\033[33m" + shellAgent.String() + "\033[31m]»\033[0m ")
			// 设置shellMenuContext为"agent"
			shellMenuContext = "agent"
		}
	}
}

// menuListener处理与实例化监听器的交互的所有逻辑
func menuListener(cmd []string) {
	// 切换命令的第一个单词为小写
	switch strings.ToLower(cmd[0]) {
# 如果用户输入的命令是 "back"，则将 shellMenuContext 设置为 "listenersmain"
# 并根据 "listenersmain" 设置自动补全选项
# 设置提示符为 "\033[31mkubesploit[\033[32mlisteners\033[31m]»\033[0m "
case "back":
    shellMenuContext = "listenersmain"
    prompt.Config.AutoComplete = getCompleter("listenersmain")
    prompt.SetPrompt("\033[31mkubesploit[\033[32mlisteners\033[31m]»\033[0m ")

# 如果用户输入的命令是 "delete"，则确认是否要删除 shellListener.name 对应的监听器
# 如果确认删除，则调用 listenerAPI.Remove() 方法删除监听器
# 如果删除成功，则重置 shellListener 和 shellListenerOptions，设置 shellMenuContext 为 "listenersmain"
# 并根据 "listenersmain" 设置自动补全选项
# 设置提示符为 "\033[31mkubesploit[\033[32mlisteners\033[31m]»\033[0m "
# 如果删除失败，则将错误信息发送到 MessageChannel
case "delete":
    if confirm(fmt.Sprintf("Are you sure you want to delete the %s listener?", shellListener.name)) {
        um := listenerAPI.Remove(shellListener.name)
        if !um.Error {
            shellListener = listener{}
            shellListenerOptions = nil
            shellMenuContext = "listenersmain"
            prompt.Config.AutoComplete = getCompleter("listenersmain")
            prompt.SetPrompt("\033[31mkubesploit[\033[32mlisteners\033[31m]»\033[0m ")
        } else {
            MessageChannel <- um
        }
    }

# 如果用户输入的命令是 "exit" 或 "quit"，并且命令长度大于 1 且第二个参数是 "-y"
# 则执行退出操作
case "exit", "quit":
    if len(cmd) > 1 {
        if strings.ToLower(cmd[1]) == "-y" {
		# 退出程序
		exit()
	}
}
# 如果确认要退出，则退出程序
if confirm("Are you sure you want to exit?") {
	exit()
}
# 如果命令是 "help"，则调用菜单帮助监听器
case "help":
	menuHelpListener()
# 如果命令是 "info" 或 "show"
case "info", "show":
	# 获取监听器配置选项
	um, options := listenerAPI.GetListenerConfiguredOptions(shellListener.id)
	# 如果出现错误，向消息通道发送错误信息并跳出循环
	if um.Error {
		MessageChannel <- um
		break
	}
	# 获取监听器状态信息
	statusMessage := listenerAPI.GetListenerStatus(shellListener.id)
	# 如果出现错误，向消息通道发送错误信息并跳出循环
	if statusMessage.Error {
		MessageChannel <- statusMessage
		break
	}
	# 如果选项不为空
	if options != nil {
# 创建一个新的表格写入器，输出到标准输出
table := tablewriter.NewWriter(os.Stdout)
# 设置表头
table.SetHeader([]string{"Name", "Value"})
# 设置对齐方式
table.SetAlignment(tablewriter.ALIGN_LEFT)
# 设置行线
table.SetRowLine(true)
# 设置边框
table.SetBorder(true)

# 遍历选项，将键值对添加到表格中
for k, v := range options {
    table.Append([]string{k, v})
}
# 添加状态信息到表格中
table.Append([]string{"Status", shellListener.status})
# 渲染表格
table.Render()
```

```
case "main":
    # 设置主菜单
    menuSetMain()
case "restart":
    # 发送重启消息到消息通道
    MessageChannel <- listenerAPI.Restart(shellListener.id)
    # 获取监听器配置选项
    um, options := listenerAPI.GetListenerConfiguredOptions(shellListener.id)
    # 如果出现错误，发送错误消息到消息通道并跳出循环
    if um.Error {
        MessageChannel <- um
        break
		}
		// 设置提示符，显示当前监听器的名称
		prompt.SetPrompt("\033[31mkubesploit[\033[32mlisteners\033[31m][\033[33m" + options["Name"] + "\033[31m]»\033[0m ")
	case "set":
		// 向消息通道发送设置选项的命令
		MessageChannel <- listenerAPI.SetOption(shellListener.id, cmd)
	case "start":
		// 向消息通道发送启动监听器的命令
		MessageChannel <- listenerAPI.Start(shellListener.name)
	case "status":
		// 向消息通道发送获取监听器状态的命令
		MessageChannel <- listenerAPI.GetListenerStatus(shellListener.id)
	case "stop":
		// 向消息通道发送停止监听器的命令
		MessageChannel <- listenerAPI.Stop(shellListener.name)
	default:
		// 如果命令长度大于1，则执行命令
		if len(cmd) > 1 {
			executeCommand(cmd[0], cmd[1:])
		} else {
			// 否则执行空命令
			var x []string
			executeCommand(cmd[0], x)
		}
	}
}
// menuListeners 处理根监听器菜单的所有逻辑
func menuListeners(cmd []string) {
    // 根据命令的第一个单词进行不区分大小写的匹配
    switch strings.ToLower(cmd[0]) {
    // 如果命令是 "exit" 或 "quit"
    case "exit", "quit":
        // 如果命令长度大于1
        if len(cmd) > 1 {
            // 如果第二个单词是 "-y"
            if strings.ToLower(cmd[1]) == "-y" {
                // 退出程序
                exit()
            }
        }
        // 如果确认要退出
        if confirm("Are you sure you want to exit?") {
            // 退出程序
            exit()
        }
    // 如果命令是 "delete"
    case "delete":
        // 如果命令长度大于等于2
        if len(cmd) >= 2 {
            // 将除第一个单词外的所有单词连接成一个字符串
            name := strings.Join(cmd[1:], " ")
            // 检查监听器是否存在
            um := listenerAPI.Exists(name)
            // 如果出现错误，发送消息并返回
            if um.Error {
                MessageChannel <- um
                return
            }
# 如果用户确认要删除特定名称的监听器，则执行以下操作
if confirm(fmt.Sprintf("Are you sure you want to delete the %s listener?", name)) {
    # 调用监听器API的Remove方法删除监听器，并将删除消息发送到消息通道
    removeMessage := listenerAPI.Remove(name)
    MessageChannel <- removeMessage
    # 如果删除操作出现错误，则立即返回
    if removeMessage.Error {
        return
    }
    # 重置shellListener、shellListenerOptions和shellMenuContext
    shellListener = listener{}
    shellListenerOptions = nil
    shellMenuContext = "listenersmain"
    # 配置提示符的自动补全和设置提示符
    prompt.Config.AutoComplete = getCompleter("listenersmain")
    prompt.SetPrompt("\033[31mkubesploit[\033[32mlisteners\033[31m]»\033[0m ")
}
# 如果用户输入的命令是"help"，则调用menuHelpListenersMain函数
case "help":
    menuHelpListenersMain()
# 如果用户输入的命令是"info"，并且命令的长度大于等于2
case "info":
    # 获取监听器的名称
    if len(cmd) >= 2 {
        name := strings.Join(cmd[1:], " ")
        # 检查监听器是否存在
        um := listenerAPI.Exists(name)
        # 如果出现错误
        if um.Error {
# 从消息通道中接收消息
MessageChannel <- um
# 返回
return
# 如果获取监听器名称失败，则向消息通道发送错误消息并返回
}
r, id := listenerAPI.GetListenerByName(name)
if r.Error {
    MessageChannel <- r
    return
}
# 如果返回的 ID 为空，则向消息通道发送警告消息
if id == uuid.Nil {
    MessageChannel <- messages.UserMessage{
        Level:   messages.Warn,
        Message: "a nil Listener UUID was returned",
        Time:    time.Time{},
        Error:   true,
    }
}
# 获取监听器配置选项和消息
oMessage, options := listenerAPI.GetListenerConfiguredOptions(id)
# 如果获取监听器配置选项失败，则向消息通道发送错误消息并返回
if oMessage.Error {
    MessageChannel <- oMessage
    return
		}
		// 如果选项不为空
		if options != nil {
			// 创建一个新的表格写入器，输出到标准输出
			table := tablewriter.NewWriter(os.Stdout)
			// 设置表头
			table.SetHeader([]string{"Name", "Value"})
			// 设置对齐方式
			table.SetAlignment(tablewriter.ALIGN_LEFT)
			// 设置行线
			table.SetRowLine(true)
			// 设置边框
			table.SetBorder(true)

			// 遍历选项，将键值对添加到表格中
			for k, v := range options {
				table.Append([]string{k, v})
			}
			// 渲染表格
			table.Render()
		}
	}
// 如果命令是"interact"
case "interact":
	// 如果命令长度大于等于2
	if len(cmd) >= 2 {
		// 将命令参数连接成字符串
		name := strings.Join(cmd[1:], " ")
		// 通过名称获取监听器和ID
		r, id := listenerAPI.GetListenerByName(name)
		// 如果返回结果有错误
		if r.Error {
			// 将结果发送到消息通道
			MessageChannel <- r
				return
			}
			// 如果 id 为空，则返回
			if id == uuid.Nil {
				// 如果 id 为默认值，则返回
				return
			}

			// 获取指定 id 的监听器状态消息
			status := listenerAPI.GetListenerStatus(id).Message
			// 创建监听器对象
			shellListener = listener{
				id:     id,
				name:   name,
				status: status,
			}
			// 设置 shellMenuContext 为 "listener"
			shellMenuContext = "listener"
			// 配置自动完成选项为 "listener" 的补全器
			prompt.Config.AutoComplete = getCompleter("listener")
			// 设置提示符为特定格式的字符串
			prompt.SetPrompt("\033[31mkubesploit[\033[32mlisteners\033[31m][\033[33m" + name + "\033[31m]»\033[0m ")
		} else {
			// 向消息通道发送用户消息
			MessageChannel <- messages.UserMessage{
				Level:   messages.Note,
				Message: "you must select a listener to interact with",
				Time:    time.Now().UTC(),
// 设置错误为假
Error: false,
}

// 如果命令是“list”，则执行以下代码
case "list":
    // 创建一个新的表格写入器，并将表头设置为指定的字段
    table := tablewriter.NewWriter(os.Stdout)
    table.SetHeader([]string{"Name", "Interface", "Port", "Protocol", "Status", "Description"})
    table.SetAlignment(tablewriter.ALIGN_CENTER)
    // 获取所有监听器的信息
    listeners := listenerAPI.GetListeners()
    // 遍历所有监听器，并将它们的信息添加到表格中
    for _, v := range listeners {
        table.Append([]string{
            v.Name,
            v.Server.GetInterface(),
            fmt.Sprintf("%d", v.Server.GetPort()),
            servers.GetProtocol(v.Server.GetProtocol()),
            servers.GetStateString(v.Server.Status()),
            v.Description})
    }
    // 打印空行
    fmt.Println()
    // 渲染表格
    table.Render()
    // 打印空行
    fmt.Println()
# 根据命令选择不同的操作
case "main", "back":
	# 设置主菜单
	menuSetMain()
case "start":
	# 如果命令长度大于等于2，将命令参数拼接成字符串
	if len(cmd) >= 2:
		name := strings.Join(cmd[1:], " ")
		# 向消息通道发送启动监听器的消息
		MessageChannel <- listenerAPI.Start(name)
case "stop":
	# 如果命令长度大于等于2，将命令参数拼接成字符串
	if len(cmd) >= 2:
		name := strings.Join(cmd[1:], " ")
		# 向消息通道发送停止监听器的消息
		MessageChannel <- listenerAPI.Stop(name)
case "use":
	# 如果命令长度大于等于2，获取监听器类型列表
	if len(cmd) >= 2:
		types := listenerAPI.GetListenerTypes()
		# 遍历类型列表
		for _, v := range types:
			# 如果命令参数与类型匹配，设置监听器选项并进入监听器设置菜单
			if strings.ToLower(cmd[1]) == v:
				shellListenerOptions = listenerAPI.GetListenerOptions(cmd[1])
				shellListenerOptions["Protocol"] = strings.ToLower(cmd[1])
				shellMenuContext = "listenersetup"
// 设置自动补全功能为 "listenersetup"
prompt.Config.AutoComplete = getCompleter("listenersetup")
// 设置提示符为特定格式的字符串，包含颜色和命令参数
prompt.SetPrompt("\033[31mkubesploit[\033[32mlisteners\033[31m][\033[33m" + strings.ToLower(cmd[1]) + "\033[31m]»\033[0m ")
// 处理 "back" 命令的逻辑，将 shellMenuContext 设置为 "listenersmain"
case "back":
	shellMenuContext = "listenersmain"
		# 设置自动补全功能的配置为 "listenersmain" 的补全器
		prompt.Config.AutoComplete = getCompleter("listenersmain")
		# 设置提示符为红色的 "kubesploit[listeners]»" 
		prompt.SetPrompt("\033[31mkubesploit[\033[32mlisteners\033[31m]»\033[0m ")
	case "exit", "quit":
		# 如果命令长度大于1
		if len(cmd) > 1 {
			# 如果第二个参数为 "-y"，则退出程序
			if strings.ToLower(cmd[1]) == "-y" {
				exit()
			}
		}
		# 如果确认要退出程序，则退出
		if confirm("Are you sure you want to exit?") {
			exit()
		}
	case "help":
		# 显示帮助菜单
		menuHelpListenerSetup()
	case "info", "show":
		# 如果 shellListenerOptions 不为空
		if shellListenerOptions != nil {
			# 创建一个新的表格写入器
			table := tablewriter.NewWriter(os.Stdout)
			# 设置表头
			table.SetHeader([]string{"Name", "Value"})
			# 设置对齐方式
			table.SetAlignment(tablewriter.ALIGN_LEFT)
			# 设置行线
			table.SetRowLine(true)
			# 设置边框
			table.SetBorder(true)
# 遍历 shellListenerOptions 字典，将键值对添加到表格中
for k, v := range shellListenerOptions:
    table.Append([]string{k, v})
# 渲染表格
table.Render()

# 如果命令是 "main"，则调用 menuSetMain() 函数
case "main":
    menuSetMain()

# 如果命令是 "set"，并且命令长度大于等于2
if len(cmd) >= 2:
    # 遍历 shellListenerOptions 字典
    for k := range shellListenerOptions:
        # 如果命令的第二个参数等于字典中的键
        if cmd[1] == k:
            # 将字典中对应键的值设置为命令后面的参数拼接而成的字符串
            shellListenerOptions[k] = strings.Join(cmd[2:], " ")
            # 发送成功消息到消息通道
            m := fmt.Sprintf("set %s to: %s", k, strings.Join(cmd[2:], " "))
            MessageChannel <- messages.UserMessage{
                Level:   messages.Success,
                Message: m,
                Time:    time.Now().UTC(),
                Error:   false,
            }
		}
	}
}
# 当命令为"start", "run", "execute"时执行以下操作
case "start", "run", "execute":
	# 使用shellListenerOptions创建新的监听器
	um, id := listenerAPI.NewListener(shellListenerOptions)
	# 将um发送到MessageChannel
	MessageChannel <- um
	# 如果um中包含错误信息，则返回
	if um.Error {
		return
	}
	# 如果id为uuid.Nil，则发送警告消息到MessageChannel并返回
	if id == uuid.Nil {
		MessageChannel <- messages.UserMessage{
			Level:   messages.Warn,
			Message: "a nil Listener UUID was returned",
			Time:    time.Time{},
			Error:   true,
		}
		return
	}

	# 将id和shellListenerOptions中的名称创建监听器对象
	shellListener = listener{id: id, name: shellListenerOptions["Name"]}
		// 调用 listenerAPI.Start 方法启动监听器，并将返回的消息发送到 MessageChannel
		startMessage := listenerAPI.Start(shellListener.name)
		MessageChannel <- startMessage
		// 调用 listenerAPI.GetListenerConfiguredOptions 方法获取监听器配置选项
		um, options := listenerAPI.GetListenerConfiguredOptions(shellListener.id)
		// 如果出现错误，将错误消息发送到 MessageChannel 并跳出循环
		if um.Error {
			MessageChannel <- um
			break
		}
		// 设置 shellMenuContext 为 "listener"
		shellMenuContext = "listener"
		// 配置提示符的自动补全
		prompt.Config.AutoComplete = getCompleter("listener")
		// 设置提示符的文本
		prompt.SetPrompt("\033[31mkubesploit[\033[32mlisteners\033[31m][\033[33m" + options["Name"] + "\033[31m]»\033[0m ")
	default:
		// 如果命令长度大于1，执行命令
		if len(cmd) > 1 {
			executeCommand(cmd[0], cmd[1:])
		} else {
			// 否则执行空命令
			var x []string
			executeCommand(cmd[0], x)
		}
	}
}
# 定义一个函数，用于设置菜单模块
func menuSetModule(cmd string) {
    # 如果命令长度大于0
	if len(cmd) > 0 {
        # 拼接模块路径
		mPath := path.Join(core.CurrentDir, "data", "modules", cmd+".json")
        # 获取模块信息
		um, m := moduleAPI.GetModule(mPath)
        # 如果获取模块信息出错，发送错误消息并返回
		if um.Error {
			MessageChannel <- um
			return
		}
        # 如果模块名称不为空
		if m.Name != "" {
            # 设置当前模块为shellModule
			shellModule = m
            # 配置自动完成
			prompt.Config.AutoComplete = getCompleter("module")
            # 设置提示符
			prompt.SetPrompt("\033[31mkubesploit[\033[32mmodule\033[31m][\033[33m" + shellModule.Name + "\033[31m]»\033[0m ")
            # 设置shellMenuContext为"module"
			shellMenuContext = "module"
		}
        # 如果全局Yaml配置中AutoSetAllAgents为"true"
		if GlobalYamlConfig.AutoSetAllAgents == "true" {
            # 设置shellModule的代理为"all"
			shellModule.SetAgent("all")
		}
	}
}
func menuSetMain() {
    // 设置自动补全功能的配置为主菜单的补全器
    prompt.Config.AutoComplete = getCompleter("main")
    // 设置提示符为红色的"kubesploit»"
    prompt.SetPrompt("\033[31mkubesploit»\033[0m ")
    // 设置当前的shell菜单上下文为"main"
    shellMenuContext = "main"
}

func getCompleter(completer string) *readline.PrefixCompleter {

    // 获取补全器的函数，根据传入的参数选择不同的补全器
    // 主菜单的补全器
    var main = readline.NewPrefixCompleter(
        readline.PcItem("agent",
            readline.PcItem("list"),
            readline.PcItem("interact",
                readline.PcItemDynamic(agents.GetAgentList()),
            ),
        ),
        readline.PcItem("banner"),
        readline.PcItem("help"),
        readline.PcItem("interact",
            readline.PcItemDynamic(agents.GetAgentList()),
```

// 创建一个包含命令提示的根菜单
var root = readline.NewPrefixCompleter(
    // 添加命令提示项
    readline.PcItem("agents"),
    readline.PcItem("listeners"),
    // 添加带有子菜单的命令提示项
    readline.PcItem("remove",
        readline.PcItem("all"),
        readline.PcItemDynamic(agents.GetAgentList()),
    ),
    readline.PcItem("sessions"),
    // 添加带有子菜单的命令提示项
    readline.PcItem("use",
        readline.PcItem("module",
            readline.PcItemDynamic(moduleAPI.GetModuleListCompleter()),
        ),
    ),
    readline.PcItem("version"),
)

// 创建一个包含模块相关命令提示的菜单
var module = readline.NewPrefixCompleter(
    readline.PcItem("back"),
    readline.PcItem("help"),
    readline.PcItem("info"),
    // 其他模块相关的命令提示项可以继续添加在这里
)
# 创建一个名为 readline 的模块，并调用 PcItem 方法创建不同的选项
readline.PcItem("main"),  # 创建一个名为 "main" 的选项
readline.PcItem("reload"),  # 创建一个名为 "reload" 的选项
readline.PcItem("run"),  # 创建一个名为 "run" 的选项
readline.PcItem("show",  # 创建一个名为 "show" 的选项，并包含子选项
    readline.PcItem("options"),  # "show" 选项的子选项 "options"
    readline.PcItem("info"),  # "show" 选项的子选项 "info"
),
readline.PcItem("set",  # 创建一个名为 "set" 的选项，并包含子选项
    readline.PcItem("Agent",  # "set" 选项的子选项 "Agent"，并包含子选项
        readline.PcItem("all"),  # "Agent" 选项的子选项 "all"
        readline.PcItemDynamic(agents.GetAgentList()),  # "Agent" 选项的动态子选项，根据 agents.GetAgentList() 动态生成
    ),
    readline.PcItemDynamic(shellModule.GetOptionsList()),  # "set" 选项的动态子选项，根据 shellModule.GetOptionsList() 动态生成
),
readline.PcItem("unset",  # 创建一个名为 "unset" 的选项，并包含子选项
    readline.PcItemDynamic(shellModule.GetOptionsList()),  # "unset" 选项的动态子选项，根据 shellModule.GetOptionsList() 动态生成
)

# Agent Menu
# 创建一个新的命令行自动补全器
var agent = readline.NewPrefixCompleter(
    # 添加命令提示：cmd
    readline.PcItem("cmd"),
    # 添加命令提示：back
    readline.PcItem("back"),
    # 添加命令提示：download
    readline.PcItem("download"),
    # 添加命令提示：interactive
    readline.PcItem("interactive"),
    # 添加命令提示：execute-shellcode，并为其添加子命令提示
    readline.PcItem("execute-shellcode",
        readline.PcItem("self"),
        readline.PcItem("remote"),
        readline.PcItem("RtlCreateUserThread"),
    ),
    # 添加命令提示：help
    readline.PcItem("help"),
    # 添加命令提示：info
    readline.PcItem("info"),
    # 添加命令提示：kill
    readline.PcItem("kill"),
    # 添加命令提示：ls
    readline.PcItem("ls"),
    # 添加命令提示：cd
    readline.PcItem("cd"),
    # 添加命令提示：pwd
    readline.PcItem("pwd"),
    # 添加命令提示：main
    readline.PcItem("main"),
    # 添加命令提示：shell
    readline.PcItem("shell"),
    # 添加命令提示：set，并为其添加子命令提示
    readline.PcItem("set",
        readline.PcItem("ja3"),
    ),
)
// 创建一个包含"killdate"、"maxretry"、"padding"、"skew"、"sleep"的选项的自动补全菜单
readline.PcItem("killdate"),
readline.PcItem("maxretry"),
readline.PcItem("padding"),
readline.PcItem("skew"),
readline.PcItem("sleep"),

// 创建一个包含"status"和"upload"选项的自动补全菜单
readline.PcItem("status"),
readline.PcItem("upload"),

// 创建一个包含"back"、"delete"、"help"、"info"、"main"、"remove"、"restart"、"set"选项的自动补全菜单
var listener = readline.NewPrefixCompleter(
    readline.PcItem("back"),
    readline.PcItem("delete"),
    readline.PcItem("help"),
    readline.PcItem("info"),
    readline.PcItem("main"),
    readline.PcItem("remove"),
    readline.PcItem("restart"),
    readline.PcItem("set"),
// 使用listenerAPI.GetListenerOptionsCompleter(shellListenerOptions["Protocol"])获取动态选项并添加到PcItemDynamic中
readline.PcItemDynamic(listenerAPI.GetListenerOptionsCompleter(shellListenerOptions["Protocol"])),
// 创建一个名为"show"的PcItem
readline.PcItem("show"),
// 创建一个名为"start"的PcItem
readline.PcItem("start"),
// 创建一个名为"status"的PcItem
readline.PcItem("status"),
// 创建一个名为"stop"的PcItem
readline.PcItem("stop"),

// 创建一个名为"back"的PcItem
readline.PcItem("back"),
// 创建一个名为"delete"的PcItem，并使用listenerAPI.GetListenerNamesCompleter()获取动态选项
readline.PcItem("delete",
    readline.PcItemDynamic(listenerAPI.GetListenerNamesCompleter()),
),
// 创建一个名为"help"的PcItem
readline.PcItem("help"),
// 创建一个名为"info"的PcItem，并使用listenerAPI.GetListenerNamesCompleter()获取动态选项
readline.PcItem("info",
    readline.PcItemDynamic(listenerAPI.GetListenerNamesCompleter()),
),
// 创建一个名为"interact"的PcItem，并使用listenerAPI.GetListenerNamesCompleter()获取动态选项
readline.PcItem("interact",
    readline.PcItemDynamic(listenerAPI.GetListenerNamesCompleter()),
```
这段代码是使用readline库创建了一系列的PcItem（Partial Completer Item），用于在命令行中提供自动补全的功能。其中使用了listenerAPI中的动态选项来创建PcItemDynamic，并且使用了不同的选项来创建不同的PcItem。
// 创建一个 PrefixCompleter 对象，用于设置命令行自动补全
// 添加了一系列的 PcItem 对象，用于定义命令行自动补全的选项
// 每个 PcItem 对象都可以包含子选项，用于更详细的命令行自动补全
// 最后将这些 PcItem 对象组合成一个 PrefixCompleter 对象
# 创建一个包含字符串 "main" 的 readline.PcItem 对象
readline.PcItem("main"),
# 创建一个包含字符串 "run" 的 readline.PcItem 对象
readline.PcItem("run"),
# 创建一个包含字符串 "set" 的 readline.PcItem 对象，并添加动态选项
readline.PcItem("set",
    readline.PcItemDynamic(listenerAPI.GetListenerOptionsCompleter(shellListenerOptions["Protocol"])),
),
# 创建一个包含字符串 "show" 的 readline.PcItem 对象
readline.PcItem("show"),
# 创建一个包含字符串 "start" 的 readline.PcItem 对象
readline.PcItem("start"),
# 创建一个包含字符串 "stop" 的 readline.PcItem 对象
readline.PcItem("stop"),
# 创建一个包含字符串 "agent" 的 case 分支
switch completer {
    case "agent":
        return agent
    # 创建一个包含字符串 "listener" 的 case 分支
    case "listener":
        return listener
    # 创建一个包含字符串 "listenersmain" 的 case 分支
    case "listenersmain":
        return listenersmain
    # 创建一个包含字符串 "listenersetup" 的 case 分支
    case "listenersetup":
        return listenersetup
    # 创建一个包含字符串 "main" 的 case 分支
    case "main":
		// 返回 main 变量的值
		return main
	// 如果参数为 "module"，返回 module 变量的值
	case "module":
		return module
	// 默认情况下返回 main 变量的值
	default:
		return main
	}
}

func menuHelpMain() {
	// 向消息通道发送用户消息
	MessageChannel <- messages.UserMessage{
		Level:   messages.Plain,
		Message: color.YellowString("Merlin C2 Server (version %s)\n", kubesploitVersion.Version),
		Time:    time.Now().UTC(),
		Error:   false,
	}
	// 创建一个新的表格写入器
	table := tablewriter.NewWriter(os.Stdout)
	// 设置表格对齐方式
	table.SetAlignment(tablewriter.ALIGN_LEFT)
	// 设置表格边框
	table.SetBorder(false)
	// 设置表格标题
	table.SetCaption(true, "Main Menu Help")
	// 设置表格头部
	table.SetHeader([]string{"Command", "Description", "Options"})
	// 创建一个包含命令、描述和可能的参数的二维字符串数组
	data := [][]string{
		{"agent", "Interact with agents or list agents", "interact, list"},
		{"banner", "Print the Merlin banner", ""},
		{"exit", "Exit and close the Merlin server", ""},
		{"listeners", "Move to the listeners menu", ""},
		{"interact", "Interact with an agent. Alias for Empire users", ""},
		{"quit", "Exit and close the Merlin server", ""},
		{"remove", "Remove or delete a DEAD agent from the server"},
		{"sessions", "List all agents session information. Alias for MSF users", ""},
		{"use", "Use a function of Merlin", "module"},
		{"version", "Print the Merlin server version", ""},
		{"*", "Anything else will be execute on the host operating system", ""},
	}

	// 将数据批量添加到表格中
	table.AppendBulk(data)
	// 打印空行
	fmt.Println()
	// 渲染表格
	table.Render()
	// 打印空行
	fmt.Println()
	// 将消息发送到消息通道
	MessageChannel <- messages.UserMessage{
// 设置日志级别为信息
Level:   messages.Info,
// 显示访问 wiki 获取额外信息的消息
Message: "Visit the wiki for additional information https://merlin-c2.readthedocs.io/en/latest/server/menu/main.html",
// 获取当前时间
Time:    time.Now().UTC(),
// 错误标志设为假
Error:   false,
}

// 在模块菜单中的帮助菜单
func menuHelpModule() {
    // 创建一个表格写入器
    table := tablewriter.NewWriter(os.Stdout)
    // 设置表格对齐方式
    table.SetAlignment(tablewriter.ALIGN_LEFT)
    // 设置表格边框
    table.SetBorder(false)
    // 设置表格标题
    table.SetCaption(true, "Module Menu Help")
    // 设置表头
    table.SetHeader([]string{"Command", "Description", "Options"})

    // 表格数据
    data := [][]string{
        {"back", "Return to the main menu", ""},
        {"info", "Show information about a module"},
        {"main", "Return to the main menu", ""},
        {"reload", "Reloads the module to a fresh clean state"},
// 定义一个包含命令、描述和参数的二维数组
{
    {"run", "Run or execute the module", ""},
    {"set", "Set the value for one of the module's options", "<option name> <option value>"},
    {"show", "Show information about a module or its options", "info, options"},
    {"unset", "Clear a module option to empty", "<option name>"},
    {"*", "Anything else will be execute on the host operating system", ""},
}

// 将数据批量添加到表格中
table.AppendBulk(data)
// 打印空行
fmt.Println()
// 渲染表格
table.Render()
// 打印空行
fmt.Println()
// 将消息发送到消息通道
MessageChannel <- messages.UserMessage{
    Level:   messages.Info,
    Message: "Visit the wiki for additional information https://merlin-c2.readthedocs.io/en/latest/server/menu/modules.html",
    Time:    time.Now().UTC(),
    Error:   false,
}

// 在代理菜单中的帮助菜单
// The help menu while in the agent menu
# 定义一个名为menuHelpAgent的函数
func menuHelpAgent() {
    # 创建一个新的表格写入器，并将其输出定向到标准输出流
    table := tablewriter.NewWriter(os.Stdout)
    # 设置表格对齐方式为左对齐
    table.SetAlignment(tablewriter.ALIGN_LEFT)
    # 设置表格边框不可见
    table.SetBorder(false)
    # 设置表格标题为"Agent Help Menu"
    table.SetCaption(true, "Agent Help Menu")
    # 设置表格头部内容
    table.SetHeader([]string{"Command", "Description", "Options"})

    # 定义表格数据
    data := [][]string{
        {"cd", "Change directories", "cd ../../ OR cd c:\\\\Users"},
        {"cmd", "Execute a command on the agent (DEPRECIATED)", "cmd ping -c 3 8.8.8.8"},
        {"back", "Return to the main menu", ""},
        {"download", "Download a file from the agent", "download <remote_file>"},
        {"execute-shellcode", "Execute shellcode", "self, remote <pid>, RtlCreateUserThread <pid>"},
        {"interactive", "Interactive shell", ""},
        {"info", "Display all information about the agent", ""},
        {"kill", "Instruct the agent to die or quit", ""},
        {"ls", "List directory contents", "ls /etc OR ls C:\\\\Users"},
        {"main", "Return to the main menu", ""},
        {"pwd", "Display the current working directory", "pwd"},
        {"set", "Set the value for one of the agent's options", "ja3, killdate, maxretry, padding, skew, sleep"},
    }
// 定义一个包含命令、描述和示例的数据表
data := []struct {
	Command string
	Description string
	Example string
}{
	{"shell", "在代理上执行命令", "shell ping -c 3 8.8.8.8"},
	{"status", "打印代理的当前状态", ""},
	{"upload", "将文件上传到代理", "upload <local_file> <remote_file>"},
	{"*", "其他任何命令将在主机操作系统上执行", ""},
}

// 将数据批量添加到表格中
table.AppendBulk(data)
// 打印空行
fmt.Println()
// 渲染表格
table.Render()
// 打印空行
fmt.Println()
// 将消息发送到消息通道
MessageChannel <- messages.UserMessage{
	Level:   messages.Info,
	Message: "访问维基获取更多信息 https://merlin-c2.readthedocs.io/en/latest/server/menu/agents.html",
	Time:    time.Now().UTC(),
	Error:   false,
}
}

// 主监听器菜单的帮助菜单
func menuHelpListenersMain() {
# 创建一个新的表格写入器，并将其输出到标准输出
table := tablewriter.NewWriter(os.Stdout)
# 设置表格对齐方式为左对齐
table.SetAlignment(tablewriter.ALIGN_LEFT)
# 设置表格边框为无
table.SetBorder(false)
# 设置表格标题为"Listeners Help Menu"
table.SetCaption(true, "Listeners Help Menu")
# 设置表格头部内容为指定的字符串数组
table.SetHeader([]string{"Command", "Description", "Options"})

# 创建包含命令、描述和选项的二维字符串数组
data := [][]string{
    {"back", "Return to the main menu", ""},
    {"delete", "Delete a named listener", "delete <listener_name>"},
    {"info", "Display all information about a listener", "info <listener_name>"},
    {"interact", "Interact with a named agent to modify it", "interact <listener_name>"},
    {"list", "List all created listeners", ""},
    {"main", "Return to the main menu", ""},
    {"start", "Start a named listener", "start <listener_name>"},
    {"stop", "Stop a named listener", "stop <listener_name>"},
    {"use", "Create a new listener by protocol type", "use [http,https,http2,http3,h2c]"},
    {"*", "Anything else will be execute on the host operating system", ""},
}

# 将数据批量添加到表格中
table.AppendBulk(data)
// 打印空行
fmt.Println()
// 渲染表格
table.Render()
// 打印空行
fmt.Println()
// 向消息通道发送用户消息
MessageChannel <- messages.UserMessage{
    Level:   messages.Info,
    Message: "Visit the wiki for additional information https://merlin-c2.readthedocs.io/en/latest/server/menu/listeners.html",
    Time:    time.Now().UTC(),
    Error:   false,
}
}

// Listeners 模板或设置菜单的帮助菜单
func menuHelpListenerSetup() {
    // 创建表格写入器
    table := tablewriter.NewWriter(os.Stdout)
    // 设置对齐方式
    table.SetAlignment(tablewriter.ALIGN_LEFT)
    // 设置边框
    table.SetBorder(false)
    // 设置标题
    table.SetCaption(true, "Listener Setup Help Menu")
    // 设置表头
    table.SetHeader([]string{"Command", "Description", "Options"})

    // 数据
    data := [][]string{
# 定义一个包含监听器菜单选项的数组，每个选项包括名称、描述和可选参数
{"back", "Return to the listeners menu", ""},
{"execute", "Create and start the listener (alias)", ""},
{"info", "Display all configurable information about a listener", ""},
{"main", "Return to the main menu", ""},
{"run", "Create and start the listener (alias)", ""},
{"set", "Set a configurable option", "set <option_name>"},
{"show", "Display all configurable information about a listener", ""},
{"start", "Create and start the listener", ""},
{"*", "Anything else will be execute on the host operating system", ""}

# 将数据批量添加到表格中
table.AppendBulk(data)
# 打印空行
fmt.Println()
# 渲染表格
table.Render()
# 打印空行
fmt.Println()
# 将消息发送到消息通道，包括消息级别、内容、时间和错误信息
MessageChannel <- messages.UserMessage{
    Level:   messages.Info,
    Message: "Visit the wiki for additional information https://merlin-c2.readthedocs.io/en/latest/server/menu/listeners.html",
    Time:    time.Now().UTC(),
    Error:   false,
}
// 为特定的、实例化的监听器创建帮助菜单
func menuHelpListener() {
    // 创建一个表格写入器，并设置对齐方式、边框和标题
    table := tablewriter.NewWriter(os.Stdout)
    table.SetAlignment(tablewriter.ALIGN_LEFT)
    table.SetBorder(false)
    table.SetCaption(true, "Listener Help Menu")
    table.SetHeader([]string{"Command", "Description", "Options"})

    // 定义菜单中的数据
    data := [][]string{
        {"back", "Return to the listeners menu", ""},
        {"delete", "Delete this listener", "delete <listener_name>"},
        {"info", "Display all configurable information the current listener", ""},
        {"main", "Return to the main menu", ""},
        {"restart", "Restart this listener", ""},
        {"set", "Set a configurable option", "set <option_name>"},
        {"show", "Display all configurable information about a listener", ""},
        {"start", "Start this listener", ""},
    }
// 定义一个包含命令、描述和备注的数据结构
data := []struct{
    command string
    description string
    note string
}{
    {"status", "Get the server's current status", ""},
    {"stop", "Stop the listener", ""},
    {"*", "Anything else will be execute on the host operating system", ""},
}

// 将数据批量添加到表格中
table.AppendBulk(data)
// 打印空行
fmt.Println()
// 渲染表格
table.Render()
// 打印空行
fmt.Println()
// 向消息通道发送用户消息
MessageChannel <- messages.UserMessage{
    Level:   messages.Info,
    Message: "Visit the wiki for additional information https://merlin-c2.readthedocs.io/en/latest/server/menu/listeners.html",
    Time:    time.Now().UTC(),
    Error:   false,
}

// 定义一个过滤输入的函数
func filterInput(r rune) (rune, bool) {
    switch r {
        // 阻止 CtrlZ 功能
// 根据读取到的字符进行判断，如果是Ctrl+Z则返回r和false
case readline.CharCtrlZ:
	return r, false
}
// 如果不是Ctrl+Z则返回r和true
return r, true
}

// confirm函数读取一个字符串，如果字符串是y或者yes则返回true，但不提供提示问题
func confirm(question string) bool {
	reader := bufio.NewReader(os.Stdin)
// 打印提示信息
MessageChannel <- messages.UserMessage{
	Level:   messages.Plain,
	Message: color.RedString(fmt.Sprintf("%s [yes/NO]: ", question)),
	Time:    time.Now().UTC(),
	Error:   false,
}
// 读取用户输入的字符串
response, err := reader.ReadString('\n')
if err != nil {
	// 如果发生错误则发送警告信息
	MessageChannel <- messages.UserMessage{
		Level:   messages.Warn,
// 格式化错误消息并记录时间和错误标志，返回一个包含错误信息的结构体
Message: fmt.Sprintf("There was an error reading the input:\r\n%s", err.Error()),
Time:    time.Now().UTC(),
Error:   true,
}

// 将响应字符串转换为小写
response = strings.ToLower(response)
// 去除响应字符串中的换行符和空格
response = strings.Trim(response, "\r\n")
// 定义可能的肯定响应列表
yes := []string{"y", "yes", "-y", "-Y"}

// 遍历可能的肯定响应列表，如果响应匹配其中之一，则返回 true
for _, match := range yes {
    if response == match {
        return true
    }
}
// 如果响应不匹配任何肯定响应，则返回 false
return false
}

// exit 将提示用户确认是否要退出
func exit() {
    // 输出退出提示信息
    color.Red("[!]Quitting...")
# 记录日志，提示用户因输入而关闭 Merlin 服务器
logging.Server("Shutting down Merlin due to user input")
# 退出程序，返回状态码 0
os.Exit(0)
}

# 执行系统命令
func executeCommand(name string, arg []string) {

    # 创建一个命令对象，用于执行指定的命令和参数
    cmd := exec.Command(name, arg...) // #nosec G204 Users can execute any arbitrary command by design

    # 执行命令并获取输出
    out, err := cmd.CombinedOutput()

    # 发送用户消息到消息通道，提示正在执行系统命令
    MessageChannel <- messages.UserMessage{
        Level:   messages.Info,
        Message: "Executing system command...",
        Time:    time.Time{},
        Error:   false,
    }
    # 如果执行命令出现错误，发送警告消息到消息通道
    if err != nil {
        MessageChannel <- messages.UserMessage{
            Level:   messages.Warn,
            Message: err.Error(),
		// 初始化一个空的时间对象
		Time:    time.Time{},
		// 设置错误标志为true
		Error:   true,
	}
} else {
	// 将成功消息发送到消息通道
	MessageChannel <- messages.UserMessage{
		// 设置消息级别为成功
		Level:   messages.Success,
		// 格式化输出消息内容
		Message: fmt.Sprintf("%s", out),
		// 初始化一个空的时间对象
		Time:    time.Time{},
		// 设置错误标志为false
		Error:   false,
	}
}

func registerMessageChannel() {
	// 注册消息通道
	um := messages.Register(clientID)
	// 如果注册出现错误，则将错误消息发送到消息通道并返回
	if um.Error {
		MessageChannel <- um
		return
	}
	// 如果调试模式开启
	if core.Debug {
		MessageChannel <- um
	}
}

func getUserMessages() {
	// 创建一个 goroutine，用于不断从消息队列中获取消息并发送到 MessageChannel
	go func() {
		for {
			MessageChannel <- messages.GetMessageForClient(clientID)
		}
	}()
}

// printUserMessage is used to print all messages to STDOUT for command line clients
func printUserMessage() {
	// 创建一个 goroutine，用于不断从 MessageChannel 中获取消息并打印到标准输出
	go func() {
		for {
			m := <-MessageChannel
			// 根据消息的级别使用不同的颜色打印到标准输出
			switch m.Level {
			case messages.Info:
				fmt.Println(color.CyanString("\n[i] %s", m.Message))
```
// 根据消息类型打印不同颜色的消息
switch m.Level {
    // 如果是Note类型的消息，以黄色打印
    case messages.Note:
        fmt.Println(color.YellowString("\n[-] %s", m.Message))
    // 如果是Warn类型的消息，以红色打印
    case messages.Warn:
        fmt.Println(color.RedString("\n[!] %s", m.Message))
    // 如果是Debug类型的消息，并且Debug模式开启，以红色打印
    case messages.Debug:
        if core.Debug {
            fmt.Println(color.RedString("\n[DEBUG] %s", m.Message))
        }
    // 如果是Success类型的消息，以绿色打印
    case messages.Success:
        fmt.Println(color.GreenString("\n[+] %s", m.Message))
    // 如果是Plain类型的消息，直接打印
    case messages.Plain:
        fmt.Println("\n" + m.Message)
    // 如果是其他类型的消息，以红色打印提示无效消息级别
    default:
        fmt.Println(color.RedString("\n[_-_] Invalid message level: %d\r\n%s", m.Level, m.Message))
    }
}
// 结束switch语句
}()
// 结束匿名函数
}

// 定义listener结构体
type listener struct {
// id: 监听器的唯一标识符，类型为uuid.UUID
// name: 监听器的唯一名称，类型为string
// status: 监听器服务器的状态，类型为string
```