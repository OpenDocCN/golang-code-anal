# `kubesploit\pkg\cli\cli.go`

```go
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 由自由软件基金会发布，无论是许可证的第3版还是任何以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

package cli

import (
    "bufio"
    "fmt"
    "gopkg.in/yaml.v2"
    "io"
    "io/ioutil"
    "log"
    "os"
    "os/exec"
    "path"
    "strings"
    "time"

    // 第三方库
    "github.com/chzyer/readline"
    "github.com/fatih/color"
    "github.com/olekukonko/tablewriter"
    "github.com/satori/go.uuid"

    // Merlin
    "kubesploit/pkg"
    "kubesploit/pkg/agents"
    agentAPI "kubesploit/pkg/api/agents"
    listenerAPI "kubesploit/pkg/api/listeners"
    "kubesploit/pkg/api/messages"
    moduleAPI "kubesploit/pkg/api/modules"
    "kubesploit/pkg/banner"
    "kubesploit/pkg/core"
    "kubesploit/pkg/logging"
    "kubesploit/pkg/modules"
    "kubesploit/pkg/servers"
)

// 全局变量
var shellModule modules.Module
var shellAgent uuid.UUID
var shellListener listener
var shellListenerOptions map[string]string
var prompt *readline.Instance
var shellCompleter *readline.PrefixCompleter
var shellMenuContext = "main"
// MessageChannel用于输入用户消息，最终写入CLI应用程序的STDOUT
var MessageChannel = make(chan messages.UserMessage)  // 创建一个用于传递用户消息的通道
var clientID = uuid.NewV4()  // 生成一个新的UUID作为客户端ID

type YamListener struct {
    Name string `yaml:"Name"`  // 定义YamListener结构体，包含Name字段
    Protocol string `yaml:"Protocol"`  // 包含Protocol字段
    Interface string `yaml:"Interface"`  // 包含Interface字段
    Port string `yaml:"Port"`  // 包含Port字段
}

type YamlConfig struct {
    AutoStart string `yaml:"AutoStart"`  // 定义YamlConfig结构体，包含AutoStart字段
    AutoSetAllAgents string `yaml:"AutoSetAllAgents"`  // 包含AutoSetAllAgents字段
    Listeners []YamListener `yaml:"Listeners"`  // 包含Listeners字段，类型为YamListener的切片
}
var GlobalYamlConfig YamlConfig  // 定义全局变量GlobalYamlConfig，类型为YamlConfig

const DEFAULT_YAML_FILE = "./config.yaml"  // 定义常量DEFAULT_YAML_FILE，值为"./config.yaml"

func initializeYamlFile() error {
    yamlFile := DEFAULT_YAML_FILE  // 将DEFAULT_YAML_FILE赋值给yamlFile变量

    yamlFileBytes, err := ioutil.ReadFile(yamlFile)  // 读取yamlFile文件的内容到yamlFileBytes变量中
    //fmt.Printf("[*] Read YAML file\n")  // 打印日志信息
    m := fmt.Sprintf("Reading config YAML file")  // 格式化日志信息
    um := messages.UserMessage{  // 创建UserMessage结构体实例
        Level:   messages.Success,  // 设置Level字段为Success
        Time:    time.Now().UTC(),  // 设置Time字段为当前时间
        Message: m,  // 设置Message字段为m
        Error:   false,  // 设置Error字段为false
    }

    MessageChannel <- um  // 将um发送到MessageChannel通道中

    if err != nil {  // 如果读取文件出错
        //fmt.Printf("[*] Failed to read YAML file: %s\n", err)  // 打印错误日志信息
        m := fmt.Sprintf("Failed to read YAML file: %s\n", err)  // 格式化错误日志信息
        messages.SendBroadcastMessage(messages.UserMessage{  // 发送广播消息
            Level:   messages.Note,  // 设置Level字段为Note
            Message: m,  // 设置Message字段为m
            Time:    time.Now().UTC(),  // 设置Time字段为当前时间
            Error:   false,  // 设置Error字段为false
        })
    }
}
    } else {
        // 打印解析 YAML 文件的提示信息
        // m 变量存储解析配置 YAML 文件的消息
        m := fmt.Sprintf("Parsing config YAML file")
        // 创建用户消息对象 um
        um := messages.UserMessage{
            Level:   messages.Success,
            Time:    time.Now().UTC(),
            Message: m,
            Error:   false,
        }
        // 将用户消息对象发送到消息通道
        MessageChannel <- um
        // 使用 yaml.Unmarshal 解析 YAML 文件的内容到 GlobalYamlConfig 变量
        err = yaml.Unmarshal(yamlFileBytes, &GlobalYamlConfig)
        // 如果解析出错，发送解析失败的消息到消息通道
        if err != nil {
            // 打印解析失败的提示信息
            m := fmt.Sprintf("Failed to parse YAML file: %s\n", err)
            // 发送解析失败的消息到消息通道
            messages.SendBroadcastMessage(messages.UserMessage{
                Level:   messages.Note,
                Message: m,
                Time:    time.Now().UTC(),
                Error:   false,
            })
        }
    }
    // 返回错误信息
    return err
# 加载 YAML 配置文件的函数
func loadYamlConfigFile() {
    # 初始化 YAML 文件，检查是否有错误
    err := initializeYamlFile()
    # 如果没有错误
    if err == nil {
        # 如果全局 YAML 配置中的 AutoStart 字段为 true
        if strings.ToLower(GlobalYamlConfig.AutoStart) == "true" {
            # 遍历全局 YAML 配置中的监听器列表
            for _, yamlListener := range GlobalYamlConfig.Listeners{
                # 获取监听器选项
                shellListenerOptions = listenerAPI.GetListenerOptions(yamlListener.Protocol)
                shellListenerOptions["Protocol"] = yamlListener.Protocol
                shellListenerOptions["Interface"] = yamlListener.Interface
                shellListenerOptions["Port"] = yamlListener.Port
                shellListenerOptions["Name"] = yamlListener.Name

                # 创建新的监听器
                um, id := listenerAPI.NewListener(shellListenerOptions)
                # 将消息发送到消息通道
                MessageChannel <- um
                # 如果有错误，返回
                if um.Error {
                    return
                }

                # 如果返回的 ID 为 nil
                if id == uuid.Nil {
                    # 发送警告消息到消息通道
                    MessageChannel <- messages.UserMessage{
                        Level:   messages.Warn,
                        Message: "a nil Listener UUID was returned",
                        Time:    time.Time{},
                        Error:   true,
                    }
                    return
                }

                # 设置 shellListener 变量
                shellListener = listener{id: id, name: shellListenerOptions["Name"]}
                # 启动监听器
                startMessage := listenerAPI.Start(shellListener.name)
                # 将启动消息发送到消息通道
                MessageChannel <- startMessage
                # 获取监听器配置选项
                um, _ = listenerAPI.GetListenerConfiguredOptions(shellListener.id)
                # 如果有错误，将错误消息发送到消息通道
                if um.Error {
                    MessageChannel <- um
                }
            }
        }
    }
}

# Shell 是启动命令行界面的导出函数
func Shell() {
    # 获取命令补全器
    shellCompleter = getCompleter("main")

    # 打印用户消息
    printUserMessage()
    # 注册消息通道
    registerMessageChannel()
    # 获取用户消息
    getUserMessages()
}
    // 创建一个新的 readline 实例，配置提示符、历史文件、自动补全、中断提示、结束提示、历史搜索折叠和输入过滤函数
    p, err := readline.NewEx(&readline.Config{
        Prompt:              "\033[31mkubesploit»\033[0m ",
        HistoryFile:         "/tmp/readline.tmp",
        AutoComplete:        shellCompleter,
        InterruptPrompt:     "^C",
        EOFPrompt:           "exit",
        HistorySearchFold:   true,
        FuncFilterInputRune: filterInput,
    })

    // 如果创建 readline 实例时发生错误，向消息通道发送警告消息
    if err != nil {
        MessageChannel <- messages.UserMessage{
            Level:   messages.Warn,
            Message: fmt.Sprintf("There was an error with the provided input: %s", err.Error()),
            Time:    time.Now().UTC(),
            Error:   true,
        }
    }
    // 将创建的 readline 实例赋值给 prompt
    prompt = p

    // 延迟执行的函数，用于关闭 prompt
    defer func() {
        err := prompt.Close()
        if err != nil {
            log.Fatal(err)
        }
    }()

    // 加载 YAML 配置文件
    loadYamlConfigFile()
    // 设置日志输出到 prompt 的标准错误流
    log.SetOutput(prompt.Stderr())
# 定义一个名为menuUse的函数，接受一个字符串切片作为参数
func menuUse(cmd []string) {
    # 检查cmd切片的长度是否大于0
    if len(cmd) > 0 {
        # 如果长度大于0，根据cmd[0]的值进行不同的操作
        switch cmd[0] {
        # 如果cmd[0]的值为"module"
        case "module":
            # 如果cmd切片的长度大于1，调用menuSetModule函数并传入cmd[1]作为参数
            if len(cmd) > 1 {
                menuSetModule(cmd[1])
            } else {
                # 如果长度不大于1，向消息通道发送警告消息
                MessageChannel <- messages.UserMessage{
                    Level:   messages.Warn,
                    Message: "Invalid module",
                    Time:    time.Now().UTC(),
                    Error:   false,
                }
            }
        # 如果cmd[0]的值为空
        case "":
        # 如果cmd[0]的值不匹配任何case
        default:
            # 向消息通道发送提示消息
            MessageChannel <- messages.UserMessage{
                Level:   messages.Note,
                Message: "Invalid 'use' command",
                Time:    time.Now().UTC(),
                Error:   false,
            }
        }
    } else {
        # 如果cmd切片的长度为0，向消息通道发送提示消息
        MessageChannel <- messages.UserMessage{
            Level:   messages.Note,
            Message: "Invalid 'use' command",
            Time:    time.Now().UTC(),
            Error:   false,
        }
    }
}

# 定义一个名为menuAgent的函数，接受一个字符串切片作为参数
func menuAgent(cmd []string) {
    # 根据cmd[0]的值进行不同的操作
    switch cmd[0] {
    # 如果命令是 "list"，则创建一个表格写入器，并设置表头
    table := tablewriter.NewWriter(os.Stdout)
    table.SetHeader([]string{"Agent GUID", "Platform", "User", "Host", "Transport", "Status"})
    table.SetAlignment(tablewriter.ALIGN_CENTER)
    # 遍历代理对象，将代理信息添加到表格中
    for k, v := range agents.Agents {
        # 将协议类型转换为用户友好的字符串
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
        # 将代理信息添加到表格中
        table.Append([]string{k.String(), v.Platform + "/" + v.Architecture, v.UserName,
            v.HostName, proto, agents.GetAgentStatus(k)})
    }
    # 输出空行，渲染表格，再输出空行
    fmt.Println()
    table.Render()
    fmt.Println()
    # 如果命令是 "interact"，并且命令参数长度大于1，则尝试将参数解析为 UUID
    if len(cmd) > 1 {
        i, errUUID := uuid.FromString(cmd[1])
        # 如果解析出错，则发送警告消息
        if errUUID != nil {
            MessageChannel <- messages.UserMessage{
                Level:   messages.Warn,
                Message: fmt.Sprintf("There was an error interacting with agent %s", cmd[1]),
                Time:    time.Now().UTC(),
                Error:   true,
            }
        } else {
            # 否则，设置交互菜单的代理对象
            menuSetAgent(i)
        }
    }
    # 根据命令类型进行处理，如果是"remove"则执行以下操作
    case "remove":
        # 检查命令参数的长度是否大于1
        if len(cmd) > 1 {
            # 如果命令参数为"all"，则移除所有代理
            if cmd[1] == "all" {
                # 遍历代理列表，通过代理的 UUID 移除代理
                for _, v := range agents.Agents {
                    removeAgentByUUID(v.ID.String(), v.ID)
                }
            } else {
                # 将命令参数解析为 UUID
                i, errUUID := uuid.FromString(cmd[1])
                # 如果解析出错，则向消息通道发送警告消息
                if errUUID != nil {
                    MessageChannel <- messages.UserMessage{
                        Level:   messages.Warn,
                        Message: fmt.Sprintf("There was an error interacting with agent %s", cmd[1]),
                        Time:    time.Now().UTC(),
                        Error:   true,
                    }
                } else {
                    # 否则，通过命令参数中的 UUID 移除代理
                    removeAgentByUUID(cmd[1], i)
                }
            }
        }
    }
// 根据代理ID字符串和代理ID删除代理
func removeAgentByUUID(agentIDStr string, agentID uuid.UUID){
    // 通过代理ID从代理列表中移除代理
    errRemove := agents.RemoveAgent(agentID)
    // 如果移除代理时出现错误，则发送警告消息到消息通道
    if errRemove != nil {
        MessageChannel <- messages.UserMessage{
            Level:   messages.Warn,
            Message: errRemove.Error(),
            Time:    time.Now().UTC(),
            Error:   true,
        }
    } else {
        // 如果成功移除代理，则发送信息消息到消息通道
        m := fmt.Sprintf("Agent %s was removed from the server at %s",
            agentIDStr, time.Now().UTC().Format(time.RFC3339))
        MessageChannel <- messages.UserMessage{
            Level:   messages.Info,
            Message: m,
            Time:    time.Now().UTC(),
            Error:   false,
        }
    }
}

// menuSetAgent 根据代理ID设置shellAgent和提示符
func menuSetAgent(agentID uuid.UUID) {
    // 遍历代理列表，根据代理ID设置shellAgent和提示符
    for k := range agents.Agents {
        if agentID == agents.Agents[k].ID {
            shellAgent = agentID
            prompt.Config.AutoComplete = getCompleter("agent")
            prompt.SetPrompt("\033[31mkubesploit[\033[32magent\033[31m][\033[33m" + shellAgent.String() + "\033[31m]»\033[0m ")
            shellMenuContext = "agent"
        }
    }
}

// menuListener 处理与实例化监听器交互的所有逻辑
func menuListener(cmd []string) {
    // 根据命令进行不同的逻辑处理
    switch strings.ToLower(cmd[0]) {
    case "back":
        // 如果命令是"back"，则将shellMenuContext设置为"listenersmain"，并设置提示符
        shellMenuContext = "listenersmain"
        prompt.Config.AutoComplete = getCompleter("listenersmain")
        prompt.SetPrompt("\033[31mkubesploit[\033[32mlisteners\033[31m]»\033[0m ")
    # 如果命令是 "delete"，则执行以下操作
    case "delete":
        # 如果确认要删除监听器，则执行以下操作
        if confirm(fmt.Sprintf("Are you sure you want to delete the %s listener?", shellListener.name)) {
            # 调用 listenerAPI 的 Remove 方法删除 shellListener.name 对应的监听器
            um := listenerAPI.Remove(shellListener.name)
            # 如果删除操作没有错误，则执行以下操作
            if !um.Error {
                # 重置 shellListener 和相关选项
                shellListener = listener{}
                shellListenerOptions = nil
                shellMenuContext = "listenersmain"
                # 设置提示符为默认值
                prompt.Config.AutoComplete = getCompleter("listenersmain")
                prompt.SetPrompt("\033[31mkubesploit[\033[32mlisteners\033[31m]»\033[0m ")
            } else {
                # 如果删除操作有错误，则将错误信息发送到 MessageChannel
                MessageChannel <- um
            }
        }
    # 如果命令是 "exit" 或 "quit"，则执行以下操作
    case "exit", "quit":
        # 如果命令长度大于 1，则执行以下操作
        if len(cmd) > 1 {
            # 如果第二个参数是 "-y"，则退出程序
            if strings.ToLower(cmd[1]) == "-y" {
                exit()
            }
        }
        # 如果确认要退出程序，则执行以下操作
        if confirm("Are you sure you want to exit?") {
            exit()
        }
    # 如果命令是 "help"，则执行以下操作
    case "help":
        # 调用 menuHelpListener 函数显示监听器的帮助信息
        menuHelpListener()
    # 如果命令是 "info" 或 "show"，则执行以下操作
    case "info", "show":
        # 获取 shellListener.id 对应监听器的配置选项
        um, options := listenerAPI.GetListenerConfiguredOptions(shellListener.id)
        # 如果获取配置选项时出现错误，则将错误信息发送到 MessageChannel，并跳出当前循环
        if um.Error {
            MessageChannel <- um
            break
        }
        # 获取 shellListener.id 对应监听器的状态信息
        statusMessage := listenerAPI.GetListenerStatus(shellListener.id)
        # 如果获取状态信息时出现错误，则将错误信息发送到 MessageChannel，并跳出当前循环
        if statusMessage.Error {
            MessageChannel <- statusMessage
            break
        }
        # 如果配置选项不为空，则创建表格并显示配置选项和状态信息
        if options != nil {
            table := tablewriter.NewWriter(os.Stdout)
            table.SetHeader([]string{"Name", "Value"})
            table.SetAlignment(tablewriter.ALIGN_LEFT)
            table.SetRowLine(true)
            table.SetBorder(true)

            for k, v := range options {
                table.Append([]string{k, v})
            }
            table.Append([]string{"Status", shellListener.status})
            table.Render()
        }
    # 如果命令是 "main"，则执行以下操作
    case "main":
        # 调用 menuSetMain 函数设置主菜单
        menuSetMain()
    # 如果命令是 "restart"，则向消息通道发送重启监听器的消息，并获取监听器配置选项
    case "restart":
        MessageChannel <- listenerAPI.Restart(shellListener.id)
        um, options := listenerAPI.GetListenerConfiguredOptions(shellListener.id)
        # 如果获取配置选项时出现错误，则向消息通道发送错误消息并跳出循环
        if um.Error:
            MessageChannel <- um
            break
        # 设置命令行提示符为指定格式
        prompt.SetPrompt("\033[31mkubesploit[\033[32mlisteners\033[31m][\033[33m" + options["Name"] + "\033[31m]»\033[0m ")
    # 如果命令是 "set"，则向消息通道发送设置选项的消息
    case "set":
        MessageChannel <- listenerAPI.SetOption(shellListener.id, cmd)
    # 如果命令是 "start"，则向消息通道发送启动监听器的消息
    case "start":
        MessageChannel <- listenerAPI.Start(shellListener.name)
    # 如果命令是 "status"，则向消息通道发送获取监听器状态的消息
    case "status":
        MessageChannel <- listenerAPI.GetListenerStatus(shellListener.id)
    # 如果命令是 "stop"，则向消息通道发送停止监听器的消息
    case "stop":
        MessageChannel <- listenerAPI.Stop(shellListener.name)
    # 如果命令不是以上任何一种情况
    default:
        # 如果命令长度大于1，则执行指定命令和参数
        if len(cmd) > 1:
            executeCommand(cmd[0], cmd[1:])
        # 否则执行指定命令和空参数
        else:
            var x []string
            executeCommand(cmd[0], x)
    }
// menuListeners 处理根监听器菜单的所有逻辑
func menuListeners(cmd []string) {
    // 根据命令的第一个单词进行不区分大小写的匹配
    switch strings.ToLower(cmd[0]) {
    // 如果命令是 exit 或 quit
    case "exit", "quit":
        // 如果命令长度大于1
        if len(cmd) > 1 {
            // 如果第二个单词是 -y
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
    // 如果命令是 delete
    case "delete":
        // 如果命令长度大于等于2
        if len(cmd) >= 2 {
            // 将命令中除第一个单词外的部分拼接成字符串
            name := strings.Join(cmd[1:], " ")
            // 检查监听器是否存在
            um := listenerAPI.Exists(name)
            // 如果存在错误，发送错误消息并返回
            if um.Error {
                MessageChannel <- um
                return
            }
            // 如果确认要删除监听器
            if confirm(fmt.Sprintf("Are you sure you want to delete the %s listener?", name)) {
                // 删除监听器
                removeMessage := listenerAPI.Remove(name)
                MessageChannel <- removeMessage
                // 如果删除出现错误，返回
                if removeMessage.Error {
                    return
                }
                // 重置shellListener、shellListenerOptions和shellMenuContext
                shellListener = listener{}
                shellListenerOptions = nil
                shellMenuContext = "listenersmain"
                // 设置提示符和自动补全
                prompt.Config.AutoComplete = getCompleter("listenersmain")
                prompt.SetPrompt("\033[31mkubesploit[\033[32mlisteners\033[31m]»\033[0m ")
            }
        }
    // 如果命令是 help
    case "help":
        // 显示监听器主菜单的帮助信息
        menuHelpListenersMain()
    # 如果命令长度大于等于2
    case "info":
        if len(cmd) >= 2 {
            # 将命令参数连接成字符串
            name := strings.Join(cmd[1:], " ")
            # 检查监听器是否存在
            um := listenerAPI.Exists(name)
            if um.Error {
                # 将错误消息发送到消息通道
                MessageChannel <- um
                return
            }
            # 通过名称获取监听器
            r, id := listenerAPI.GetListenerByName(name)
            if r.Error {
                # 将错误消息发送到消息通道
                MessageChannel <- r
                return
            }
            # 如果返回的 ID 为空
            if id == uuid.Nil {
                # 发送警告消息到消息通道
                MessageChannel <- messages.UserMessage{
                    Level:   messages.Warn,
                    Message: "a nil Listener UUID was returned",
                    Time:    time.Time{},
                    Error:   true,
                }
            }
            # 获取监听器配置选项
            oMessage, options := listenerAPI.GetListenerConfiguredOptions(id)
            if oMessage.Error {
                # 将错误消息发送到消息通道
                MessageChannel <- oMessage
                return
            }
            # 如果选项不为空
            if options != nil {
                # 创建表格写入器
                table := tablewriter.NewWriter(os.Stdout)
                # 设置表头
                table.SetHeader([]string{"Name", "Value"})
                # 设置对齐方式
                table.SetAlignment(tablewriter.ALIGN_LEFT)
                # 设置行线
                table.SetRowLine(true)
                # 设置边框
                table.SetBorder(true)

                # 遍历选项并添加到表格中
                for k, v := range options {
                    table.Append([]string{k, v})
                }
                # 渲染表格
                table.Render()
            }
        }
    # 如果命令是"interact"
    case "interact":
        # 如果命令参数数量大于等于2
        if len(cmd) >= 2 {
            # 将参数拼接成名称
            name := strings.Join(cmd[1:], " ")
            # 通过名称获取监听器和其ID
            r, id := listenerAPI.GetListenerByName(name)
            # 如果获取监听器出现错误，将错误消息发送到消息通道并返回
            if r.Error {
                MessageChannel <- r
                return
            }
            # 如果ID为空，直接返回
            if id == uuid.Nil {
                return
            }

            # 获取监听器状态消息
            status := listenerAPI.GetListenerStatus(id).Message
            # 设置shellListener结构体
            shellListener = listener{
                id:     id,
                name:   name,
                status: status,
            }
            # 设置shellMenuContext为"listener"
            shellMenuContext = "listener"
            # 配置自动补全
            prompt.Config.AutoComplete = getCompleter("listener")
            # 设置提示符
            prompt.SetPrompt("\033[31mkubesploit[\033[32mlisteners\033[31m][\033[33m" + name + "\033[31m]»\033[0m ")
        } else {
            # 如果参数数量不足2，发送提示消息到消息通道
            MessageChannel <- messages.UserMessage{
                Level:   messages.Note,
                Message: "you must select a listener to interact with",
                Time:    time.Now().UTC(),
                Error:   false,
            }
        }
    # 如果命令是"list"
    case "list":
        # 创建表格写入器
        table := tablewriter.NewWriter(os.Stdout)
        # 设置表头
        table.SetHeader([]string{"Name", "Interface", "Port", "Protocol", "Status", "Description"})
        # 设置对齐方式
        table.SetAlignment(tablewriter.ALIGN_CENTER)
        # 获取所有监听器
        listeners := listenerAPI.GetListeners()
        # 遍历监听器并添加到表格中
        for _, v := range listeners {
            table.Append([]string{
                v.Name,
                v.Server.GetInterface(),
                fmt.Sprintf("%d", v.Server.GetPort()),
                servers.GetProtocol(v.Server.GetProtocol()),
                servers.GetStateString(v.Server.Status()),
                v.Description})
        }
        # 打印空行
        fmt.Println()
        # 渲染表格
        table.Render()
        # 打印空行
        fmt.Println()
    # 如果命令是"main"或"back"
    case "main", "back":
        # 设置主菜单
        menuSetMain()
    # 如果命令是"start"
    case "start":
        # 如果参数数量大于等于2
        if len(cmd) >= 2 {
            # 将参数拼接成名称，并发送启动监听器的消息到消息通道
            name := strings.Join(cmd[1:], " ")
            MessageChannel <- listenerAPI.Start(name)
        }
    # 如果命令是 "stop"，并且参数数量大于等于2
    case "stop":
        if len(cmd) >= 2 {
            # 将参数列表中的所有元素连接成一个字符串
            name := strings.Join(cmd[1:], " ")
            # 向消息通道发送停止监听器的消息
            MessageChannel <- listenerAPI.Stop(name)
        }
    # 如果命令是 "use"，并且参数数量大于等于2
    case "use":
        if len(cmd) >= 2 {
            # 获取所有监听器类型
            types := listenerAPI.GetListenerTypes()
            # 遍历所有监听器类型
            for _, v := range types {
                # 如果参数指定的类型与当前类型匹配
                if strings.ToLower(cmd[1]) == v {
                    # 获取指定类型的监听器选项
                    shellListenerOptions = listenerAPI.GetListenerOptions(cmd[1])
                    # 设置监听器选项中的协议
                    shellListenerOptions["Protocol"] = strings.ToLower(cmd[1])
                    # 设置 shellMenuContext 为 "listenersetup"
                    shellMenuContext = "listenersetup"
                    # 设置提示符的自动补全
                    prompt.Config.AutoComplete = getCompleter("listenersetup")
                    # 设置提示符的文本
                    prompt.SetPrompt("\033[31mkubesploit[\033[32mlisteners\033[31m][\033[33m" + strings.ToLower(cmd[1]) + "\033[31m]»\033[0m ")
                }
            }
        }
    # 如果命令不是 "stop" 也不是 "use"
    default:
        # 如果参数数量大于1
        if len(cmd) > 1 {
            # 执行指定命令和参数
            executeCommand(cmd[0], cmd[1:])
        } else {
            # 如果参数数量为1
            var x []string
            # 执行指定命令和空参数列表
            executeCommand(cmd[0], x)
        }
    }
// menuListenerSetup处理设置监听器的所有逻辑
func menuListenerSetup(cmd []string) {
    // 根据命令的第一个单词进行判断
    switch strings.ToLower(cmd[0]) {
    case "back":
        // 设置shellMenuContext为"listenersmain"
        shellMenuContext = "listenersmain"
        // 设置提示符
        prompt.Config.AutoComplete = getCompleter("listenersmain")
        prompt.SetPrompt("\033[31mkubesploit[\033[32mlisteners\033[31m]»\033[0m ")
    case "exit", "quit":
        // 如果命令长度大于1，并且第二个单词为"-y"，则退出程序
        if len(cmd) > 1 {
            if strings.ToLower(cmd[1]) == "-y" {
                exit()
            }
        }
        // 如果确认要退出，则退出程序
        if confirm("Are you sure you want to exit?") {
            exit()
        }
    case "help":
        // 调用menuHelpListenerSetup函数
        menuHelpListenerSetup()
    case "info", "show":
        // 如果shellListenerOptions不为空，则创建表格并输出
        if shellListenerOptions != nil {
            table := tablewriter.NewWriter(os.Stdout)
            table.SetHeader([]string{"Name", "Value"})
            table.SetAlignment(tablewriter.ALIGN_LEFT)
            table.SetRowLine(true)
            table.SetBorder(true)

            for k, v := range shellListenerOptions {
                table.Append([]string{k, v})
            }
            table.Render()
        }
    case "main":
        // 调用menuSetMain函数
        menuSetMain()
    case "set":
        // 如果命令长度大于等于2，则遍历shellListenerOptions并设置对应的值
        if len(cmd) >= 2 {
            for k := range shellListenerOptions {
                if cmd[1] == k {
                    shellListenerOptions[k] = strings.Join(cmd[2:], " ")
                    m := fmt.Sprintf("set %s to: %s", k, strings.Join(cmd[2:], " "))
                    // 发送成功消息到MessageChannel
                    MessageChannel <- messages.UserMessage{
                        Level:   messages.Success,
                        Message: m,
                        Time:    time.Now().UTC(),
                        Error:   false,
                    }
                }
            }
        }
    # 如果命令是 "start", "run", "execute"，则执行以下操作
    case "start", "run", "execute":
        # 使用监听器API创建新的监听器，并返回用户消息和监听器ID
        um, id := listenerAPI.NewListener(shellListenerOptions)
        # 将用户消息发送到消息通道
        MessageChannel <- um
        # 如果返回的用户消息包含错误，则返回
        if um.Error {
            return
        }
        # 如果返回的监听器ID是空，则发送警告消息到消息通道并返回
        if id == uuid.Nil {
            MessageChannel <- messages.UserMessage{
                Level:   messages.Warn,
                Message: "a nil Listener UUID was returned",
                Time:    time.Time{},
                Error:   true,
            }
            return
        }

        # 初始化shellListener对象
        shellListener = listener{id: id, name: shellListenerOptions["Name"]}
        # 启动监听器，并将启动消息发送到消息通道
        startMessage := listenerAPI.Start(shellListener.name)
        MessageChannel <- startMessage
        # 获取监听器配置选项，并返回用户消息和选项
        um, options := listenerAPI.GetListenerConfiguredOptions(shellListener.id)
        # 如果返回的用户消息包含错误，则将用户消息发送到消息通道并跳出循环
        if um.Error {
            MessageChannel <- um
            break
        }
        # 设置shellMenuContext为"listener"
        shellMenuContext = "listener"
        # 配置自动完成选项为"listener"的自动完成器
        prompt.Config.AutoComplete = getCompleter("listener")
        # 设置提示符为特定格式的字符串
        prompt.SetPrompt("\033[31mkubesploit[\033[32mlisteners\033[31m][\033[33m" + options["Name"] + "\033[31m]»\033[0m ")
    # 如果命令不是 "start", "run", "execute"，则执行以下操作
    default:
        # 如果命令长度大于1，则执行命令和参数
        if len(cmd) > 1 {
            executeCommand(cmd[0], cmd[1:])
        } else {
            # 如果命令长度为1，则执行命令和空参数
            var x []string
            executeCommand(cmd[0], x)
        }
    }
func menuSetModule(cmd string) {
    // 如果命令长度大于0
    if len(cmd) > 0 {
        // 拼接模块路径
        mPath := path.Join(core.CurrentDir, "data", "modules", cmd+".json")
        // 获取模块信息
        um, m := moduleAPI.GetModule(mPath)
        // 如果获取模块信息出错，发送错误消息到消息通道并返回
        if um.Error {
            MessageChannel <- um
            return
        }
        // 如果模块名称不为空
        if m.Name != "" {
            // 设置当前shell模块
            shellModule = m
            // 配置自动补全
            prompt.Config.AutoComplete = getCompleter("module")
            // 设置提示符
            prompt.SetPrompt("\033[31mkubesploit[\033[32mmodule\033[31m][\033[33m" + shellModule.Name + "\033[31m]»\033[0m ")
            // 设置shell菜单上下文为模块
            shellMenuContext = "module"
        }
        // 如果全局YAML配置中自动设置所有代理为true
        if GlobalYamlConfig.AutoSetAllAgents == "true" {
            // 设置模块代理为所有
            shellModule.SetAgent("all")
        }
    }
}

func menuSetMain() {
    // 设置自动补全
    prompt.Config.AutoComplete = getCompleter("main")
    // 设置提示符
    prompt.SetPrompt("\033[31mkubesploit»\033[0m ")
    // 设置shell菜单上下文为主菜单
    shellMenuContext = "main"
}

func getCompleter(completer string) *readline.PrefixCompleter {

    // 主菜单自动补全
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
        ),
        readline.PcItem("listeners"),
        readline.PcItem("remove",
            readline.PcItem("all"),
            readline.PcItemDynamic(agents.GetAgentList()),
        ),
        readline.PcItem("sessions"),
        readline.PcItem("use",
            readline.PcItem("module",
                readline.PcItemDynamic(moduleAPI.GetModuleListCompleter()),
            ),
        ),
        readline.PcItem("version"),
    )

    // 模块菜单
    // 创建一个模块的自动补全器，包括back、help、info、main、reload、run、show等选项
    var module = readline.NewPrefixCompleter(
        readline.PcItem("back"),
        readline.PcItem("help"),
        readline.PcItem("info"),
        readline.PcItem("main"),
        readline.PcItem("reload"),
        readline.PcItem("run"),
        readline.PcItem("show",
            readline.PcItem("options"),
            readline.PcItem("info"),
        ),
        readline.PcItem("set",
            readline.PcItem("Agent",
                readline.PcItem("all"),
                readline.PcItemDynamic(agents.GetAgentList()),
            ),
            readline.PcItemDynamic(shellModule.GetOptionsList()),
        ),
        readline.PcItem("unset",
            readline.PcItemDynamic(shellModule.GetOptionsList()),
        ),
    )

    // 创建一个代理的自动补全器，包括cmd、back、download、interactive等选项
    // 以及set下的ja3、killdate、maxretry等选项
    var agent = readline.NewPrefixCompleter(
        readline.PcItem("cmd"),
        readline.PcItem("back"),
        readline.PcItem("download"),
        readline.PcItem("interactive"),
        readline.PcItem("execute-shellcode",
            readline.PcItem("self"),
            readline.PcItem("remote"),
            readline.PcItem("RtlCreateUserThread"),
        ),
        readline.PcItem("help"),
        readline.PcItem("info"),
        readline.PcItem("kill"),
        readline.PcItem("ls"),
        readline.PcItem("cd"),
        readline.PcItem("pwd"),
        readline.PcItem("main"),
        readline.PcItem("shell"),
        readline.PcItem("set",
            readline.PcItem("ja3"),
            readline.PcItem("killdate"),
            readline.PcItem("maxretry"),
            readline.PcItem("padding"),
            readline.PcItem("skew"),
            readline.PcItem("sleep"),
        ),
        readline.PcItem("status"),
        readline.PcItem("upload"),
    )

    // 监听器菜单（特定监听器）
    // 创建一个具有自动补全功能的命令行监听器
    var listener = readline.NewPrefixCompleter(
        // 添加命令选项
        readline.PcItem("back"),
        readline.PcItem("delete"),
        readline.PcItem("help"),
        readline.PcItem("info"),
        readline.PcItem("main"),
        readline.PcItem("remove"),
        readline.PcItem("restart"),
        // 设置子命令选项，并使用动态自动补全
        readline.PcItem("set",
            readline.PcItemDynamic(listenerAPI.GetListenerOptionsCompleter(shellListenerOptions["Protocol"])),
        ),
        readline.PcItem("show"),
        readline.PcItem("start"),
        readline.PcItem("status"),
        readline.PcItem("stop"),
    )

    // 创建主菜单的命令行监听器
    var listenersmain = readline.NewPrefixCompleter(
        // 添加命令选项
        readline.PcItem("back"),
        // 设置子命令选项，并使用动态自动补全
        readline.PcItem("delete",
            readline.PcItemDynamic(listenerAPI.GetListenerNamesCompleter()),
        ),
        readline.PcItem("help"),
        // 设置子命令选项，并使用动态自动补全
        readline.PcItem("info",
            readline.PcItemDynamic(listenerAPI.GetListenerNamesCompleter()),
        ),
        // 设置子命令选项，并使用动态自动补全
        readline.PcItem("interact",
            readline.PcItemDynamic(listenerAPI.GetListenerNamesCompleter()),
        ),
        readline.PcItem("list"),
        readline.PcItem("main"),
        // 设置子命令选项，并使用动态自动补全
        readline.PcItem("start",
            readline.PcItemDynamic(listenerAPI.GetListenerNamesCompleter()),
        ),
        // 设置子命令选项，并使用动态自动补全
        readline.PcItem("stop",
            readline.PcItemDynamic(listenerAPI.GetListenerNamesCompleter()),
        ),
        // 设置子命令选项，并使用动态自动补全
        readline.PcItem("use",
            readline.PcItemDynamic(listenerAPI.GetListenerTypesCompleter()),
        ),
    )

    // Listener Setup Menu
    # 创建一个自动补全器，用于读取用户输入并提供自动补全选项
    var listenersetup = readline.NewPrefixCompleter(
        # 设置自动补全选项：back
        readline.PcItem("back"),
        # 设置自动补全选项：execute
        readline.PcItem("execute"),
        # 设置自动补全选项：help
        readline.PcItem("help"),
        # 设置自动补全选项：info
        readline.PcItem("info"),
        # 设置自动补全选项：main
        readline.PcItem("main"),
        # 设置自动补全选项：run
        readline.PcItem("run"),
        # 设置自动补全选项：set，并包含动态补全选项
        readline.PcItem("set",
            readline.PcItemDynamic(listenerAPI.GetListenerOptionsCompleter(shellListenerOptions["Protocol"])),
        ),
        # 设置自动补全选项：show
        readline.PcItem("show"),
        # 设置自动补全选项：start
        readline.PcItem("start"),
        # 设置自动补全选项：stop
        readline.PcItem("stop"),
    )
    
    # 根据不同的自动补全选项返回相应的补全器
    switch completer {
    case "agent":
        return agent
    case "listener":
        return listener
    case "listenersmain":
        return listenersmain
    case "listenersetup":
        return listenersetup
    case "main":
        return main
    case "module":
        return module
    default:
        return main
    }
# 主菜单帮助函数，向消息通道发送用户消息，显示服务器版本信息
func menuHelpMain() {
    MessageChannel <- messages.UserMessage{
        Level:   messages.Plain,
        Message: color.YellowString("Merlin C2 Server (version %s)\n", kubesploitVersion.Version),
        Time:    time.Now().UTC(),
        Error:   false,
    }
    # 创建表格对象，设置对齐方式、边框和标题
    table := tablewriter.NewWriter(os.Stdout)
    table.SetAlignment(tablewriter.ALIGN_LEFT)
    table.SetBorder(false)
    table.SetCaption(true, "Main Menu Help")
    table.SetHeader([]string{"Command", "Description", "Options"})

    # 定义菜单帮助信息的数据
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

    # 向表格添加数据
    table.AppendBulk(data)
    # 渲染表格并打印到控制台
    fmt.Println()
    table.Render()
    fmt.Println()
    # 向消息通道发送用户消息，提供额外信息的链接
    MessageChannel <- messages.UserMessage{
        Level:   messages.Info,
        Message: "Visit the wiki for additional information https://merlin-c2.readthedocs.io/en/latest/server/menu/main.html",
        Time:    time.Now().UTC(),
        Error:   false,
    }
}

# 在模块菜单中的帮助菜单函数
func menuHelpModule() {
    # 创建表格对象，设置对齐方式、边框和标题
    table := tablewriter.NewWriter(os.Stdout)
    table.SetAlignment(tablewriter.ALIGN_LEFT)
    table.SetBorder(false)
    table.SetCaption(true, "Module Menu Help")
    table.SetHeader([]string{"Command", "Description", "Options"})
    # 创建一个二维字符串数组，包含每个选项的命令、描述和参数
    data := [][]string{
        {"back", "Return to the main menu", ""},
        {"info", "Show information about a module"},
        {"main", "Return to the main menu", ""},
        {"reload", "Reloads the module to a fresh clean state"},
        {"run", "Run or execute the module", ""},
        {"set", "Set the value for one of the module's options", "<option name> <option value>"},
        {"show", "Show information about a module or its options", "info, options"},
        {"unset", "Clear a module option to empty", "<option name>"},
        {"*", "Anything else will be execute on the host operating system", ""},
    }

    # 将二维数组的内容批量添加到表格中
    table.AppendBulk(data)
    # 打印空行
    fmt.Println()
    # 渲染表格
    table.Render()
    # 打印空行
    fmt.Println()
    # 将消息发送到消息通道
    MessageChannel <- messages.UserMessage{
        Level:   messages.Info,
        Message: "Visit the wiki for additional information https://merlin-c2.readthedocs.io/en/latest/server/menu/modules.html",
        Time:    time.Now().UTC(),
        Error:   false,
    }
// 在代理菜单中的帮助菜单
func menuHelpAgent() {
    // 创建一个表格写入器，并设置对齐方式为左对齐
    table := tablewriter.NewWriter(os.Stdout)
    table.SetAlignment(tablewriter.ALIGN_LEFT)
    // 设置表格边框不可见
    table.SetBorder(false)
    // 设置表格标题为"Agent Help Menu"
    table.SetCaption(true, "Agent Help Menu")
    // 设置表格头部内容为Command, Description, Options
    table.SetHeader([]string{"Command", "Description", "Options"})

    // 定义表格数据
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
        {"shell", "Execute a command on the agent", "shell ping -c 3 8.8.8.8"},
        {"status", "Print the current status of the agent", ""},
        {"upload", "Upload a file to the agent", "upload <local_file> <remote_file>"},
        {"*", "Anything else will be execute on the host operating system", ""},
    }

    // 向表格中批量添加数据
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
        Message: "Visit the wiki for additional information https://merlin-c2.readthedocs.io/en/latest/server/menu/agents.html",
        Time:    time.Now().UTC(),
        Error:   false,
    }
}

// 主或根监听器菜单的帮助菜单
func menuHelpListenersMain() {
    // 创建一个表格写入器，并设置输出到标准输出
    table := tablewriter.NewWriter(os.Stdout)
    # 设置表格对齐方式为左对齐
    table.SetAlignment(tablewriter.ALIGN_LEFT)
    # 设置表格无边框
    table.SetBorder(false)
    # 设置表格标题为"Listeners Help Menu"
    table.SetCaption(true, "Listeners Help Menu")
    # 设置表格头部为["Command", "Description", "Options"]
    table.SetHeader([]string{"Command", "Description", "Options"})
    
    # 定义表格数据
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
    # 打印空行
    fmt.Println()
    # 渲染表格
    table.Render()
    # 打印空行
    fmt.Println()
    # 将消息发送到消息通道
    MessageChannel <- messages.UserMessage{
        Level:   messages.Info,
        Message: "Visit the wiki for additional information https://merlin-c2.readthedocs.io/en/latest/server/menu/listeners.html",
        Time:    time.Now().UTC(),
        Error:   false,
    }
// Listeners模板或设置菜单的帮助菜单
func menuHelpListenerSetup() {
    // 创建一个新的表格写入器，并设置对齐方式为左对齐
    table := tablewriter.NewWriter(os.Stdout)
    table.SetAlignment(tablewriter.ALIGN_LEFT)
    // 设置表格边框为false
    table.SetBorder(false)
    // 设置表格标题为"Listener Setup Help Menu"
    table.SetCaption(true, "Listener Setup Help Menu")
    // 设置表格头部内容为Command, Description, Options
    table.SetHeader([]string{"Command", "Description", "Options"})

    // 定义表格数据
    data := [][]string{
        {"back", "Return to the listeners menu", ""},
        {"execute", "Create and start the listener (alias)", ""},
        {"info", "Display all configurable information about a listener", ""},
        {"main", "Return to the main menu", ""},
        {"run", "Create and start the listener (alias)", ""},
        {"set", "Set a configurable option", "set <option_name>"},
        {"show", "Display all configurable information about a listener", ""},
        {"start", "Create and start the listener", ""},
        {"*", "Anything else will be execute on the host operating system", ""},
    }

    // 向表格中批量添加数据
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

// 特定实例化监听器的帮助菜单
func menuHelpListener() {
    // 创建一个新的表格写入器，并设置对齐方式为左对齐
    table := tablewriter.NewWriter(os.Stdout)
    table.SetAlignment(tablewriter.ALIGN_LEFT)
    // 设置表格边框为false
    table.SetBorder(false)
    // 设置表格标题为"Listener Help Menu"
    table.SetCaption(true, "Listener Help Menu")
    // 设置表格头部内容为Command, Description, Options
    table.SetHeader([]string{"Command", "Description", "Options"})
    // 定义一个二维字符串数组，包含了监听器菜单的各种操作和说明
    data := [][]string{
        {"back", "Return to the listeners menu", ""},
        {"delete", "Delete this listener", "delete <listener_name>"},
        {"info", "Display all configurable information the current listener", ""},
        {"main", "Return to the main menu", ""},
        {"restart", "Restart this listener", ""},
        {"set", "Set a configurable option", "set <option_name>"},
        {"show", "Display all configurable information about a listener", ""},
        {"start", "Start this listener", ""},
        {"status", "Get the server's current status", ""},
        {"stop", "Stop the listener", ""},
        {"*", "Anything else will be execute on the host operating system", ""},
    }

    // 将二维数组的内容批量添加到表格中
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
        Message: "Visit the wiki for additional information https://merlin-c2.readthedocs.io/en/latest/server/menu/listeners.html",
        Time:    time.Now().UTC(),
        Error:   false,
    }
// 定义一个函数，用于过滤输入的字符，如果是CtrlZ则返回false
func filterInput(r rune) (rune, bool) {
    switch r {
    // 阻止CtrlZ功能
    case readline.CharCtrlZ:
        return r, false
    }
    return r, true
}

// 确认函数读取一个字符串，如果字符串是y或yes则返回true，但不提供提示问题
func confirm(question string) bool {
    reader := bufio.NewReader(os.Stdin)
    // 输出带有颜色的提示信息
    MessageChannel <- messages.UserMessage{
        Level:   messages.Plain,
        Message: color.RedString(fmt.Sprintf("%s [yes/NO]: ", question)),
        Time:    time.Now().UTC(),
        Error:   false,
    }
    response, err := reader.ReadString('\n')
    if err != nil {
        // 输出警告信息
        MessageChannel <- messages.UserMessage{
            Level:   messages.Warn,
            Message: fmt.Sprintf("There was an error reading the input:\r\n%s", err.Error()),
            Time:    time.Now().UTC(),
            Error:   true,
        }
    }
    response = strings.ToLower(response)
    response = strings.Trim(response, "\r\n")
    yes := []string{"y", "yes", "-y", "-Y"}

    for _, match := range yes {
        if response == match {
            return true
        }
    }
    return false
}

// 退出函数，提示用户确认是否要退出
func exit() {
    color.Red("[!]Quitting...")
    logging.Server("Shutting down Merlin due to user input")
    os.Exit(0)
}

// 执行命令函数，接收命令名和参数
func executeCommand(name string, arg []string) {
    // 创建一个命令对象
    cmd := exec.Command(name, arg...) // #nosec G204 Users can execute any arbitrary command by design

    // 执行命令并获取输出
    out, err := cmd.CombinedOutput()

    // 输出信息到消息通道
    MessageChannel <- messages.UserMessage{
        Level:   messages.Info,
        Message: "Executing system command...",
        Time:    time.Time{},
        Error:   false,
    }
    if err != nil {
        // 输出警告信息
        MessageChannel <- messages.UserMessage{
            Level:   messages.Warn,
            Message: err.Error(),
            Time:    time.Time{},
            Error:   true,
        }
    }
}
    } else {
        # 如果条件不满足，向消息通道发送用户消息
        MessageChannel <- messages.UserMessage{
            # 设置消息级别为成功
            Level:   messages.Success,
            # 设置消息内容为输出的格式化字符串
            Message: fmt.Sprintf("%s", out),
            # 设置消息时间为空
            Time:    time.Time{},
            # 设置错误标志为假
            Error:   false,
        }
    }
// 注册消息通道，将客户端ID注册到消息通道中
func registerMessageChannel() {
    // 注册消息通道并返回消息
    um := messages.Register(clientID)
    // 如果注册消息时出现错误，将错误消息发送到消息通道并返回
    if um.Error {
        MessageChannel <- um
        return
    }
    // 如果处于调试模式，将消息发送到消息通道
    if core.Debug {
        MessageChannel <- um
    }
}

// 获取用户消息，启动一个 goroutine 不断从消息通道中获取消息并发送给客户端
func getUserMessages() {
    go func() {
        for {
            MessageChannel <- messages.GetMessageForClient(clientID)
        }
    }()
}

// printUserMessage 用于将所有消息打印到命令行客户端的标准输出
func printUserMessage() {
    go func() {
        for {
            m := <-MessageChannel
            // 根据消息级别使用不同颜色打印消息
            switch m.Level {
            case messages.Info:
                fmt.Println(color.CyanString("\n[i] %s", m.Message))
            case messages.Note:
                fmt.Println(color.YellowString("\n[-] %s", m.Message))
            case messages.Warn:
                fmt.Println(color.RedString("\n[!] %s", m.Message))
            case messages.Debug:
                // 如果处于调试模式，打印调试消息
                if core.Debug {
                    fmt.Println(color.RedString("\n[DEBUG] %s", m.Message))
                }
            case messages.Success:
                fmt.Println(color.GreenString("\n[+] %s", m.Message))
            case messages.Plain:
                fmt.Println("\n" + m.Message)
            default:
                fmt.Println(color.RedString("\n[_-_] Invalid message level: %d\r\n%s", m.Level, m.Message))
            }
        }
    }()
}

// 监听器结构体，包含唯一标识符、名称和状态
type listener struct {
    id     uuid.UUID // 监听器唯一标识符
    name   string    // 监听器唯一名称
    status string    // 监听器服务器状态
}
```