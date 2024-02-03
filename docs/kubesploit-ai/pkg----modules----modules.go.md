# `kubesploit\pkg\modules\modules.go`

```go
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 由自由软件基金会发布，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

package modules

import (
    // 标准库
    "encoding/json"
    "errors"
    "fmt"
    "io/ioutil"
    "os"
    "path"
    "path/filepath"
    "regexp"
    "strconv"
    "strings"

    // 第三方库
    "github.com/fatih/color"
    "github.com/olekukonko/tablewriter"
    "github.com/satori/go.uuid"

    // Merlin
    "kubesploit/pkg/agents"
    "kubesploit/pkg/core"
    "kubesploit/pkg/modules/minidump"
    "kubesploit/pkg/modules/shellcode"
    "kubesploit/pkg/modules/srdi"
)

// Module是一个包含模块基本信息或模板的结构
type Module struct {
    Agent        uuid.UUID      // 将在执行前与此模块关联的代理
    Name         string         `json:"name"`                 // 模块的名称
    Type         string         `json:"type"`                 // 模块的类型（标准或扩展）
    Author       []string       `json:"author"`               // 模块作者的列表
    // 用于记录为基础工具或技术贡献的人员列表
    Credits      []string       `json:"credits"`
    // 模块的路径（例如 data/modules/powershell/powerview）
    Path         []string       `json:"path"`
    // 模块可以运行的平台（例如 Windows、Linux、Darwin 或 ALL）
    Platform     string         `json:"platform"`
    // 模块可以运行的架构（例如 x86、x64、MIPS、ARM 或 ALL）
    Arch         string         `json:"arch"`
    // 模块执行的语言（例如 PowerShell、Python 或 Perl）
    Lang         string         `json:"lang"`
    // 模块是否需要特权级别账户（如 root 或 SYSTEM）
    Priv         bool           `json:"privilege"`
    // 模块的描述
    Description  string         `json:"description"`
    // 模块的附加信息或注释
    Notes        string         `json:"notes"`
    // 在代理上运行的命令列表
    Commands     []string       `json:"commands"`
    // 模块的在线或远程源代码
    SourceRemote string         `json:"remote"`
    // 脚本或有效载荷的本地文件路径
    SourceLocal  []string       `json:"local"`
    // 模块的可配置选项/参数列表
    Options      []Option       `json:"options"`
    // 包含特定于 PowerShell 的命令和配置项的选项 JSON 对象
    Powershell   interface{}    `json:"powershell,omitempty"`
    // 通知代理使用 yaegi（Go 解释器）的 Go 代码
    GoInterpreter bool            `json:"GoInterpreter"`
    // 通知代理在进行中打印的 Go 代码，以便使用 yaegi（Go 解释器）
    GoInterpreterProgress bool  `json:"GoInterpreterProgress"`
    // LoadScriptFromPath 是一个布尔类型的字段，用于指示是否从路径加载脚本到命令中
    // `json:"LoadScriptFromPath"` 是该字段在 JSON 中的键名
    // Notification for loading script from path into commands with the 是对该字段的描述
// 结构体 Option 包含对象的键
type Option struct {
    Name        string `json:"name"`        // 选项的名称
    Value       string `json:"value"`       // 选项的值
    Required    bool   `json:"required"`    // 是否是必需选项？
    Flag        string `json:"flag"`        // 选项的命令行标志
    Description string `json:"description"` // 选项的描述
}

// 结构体 PowerShell 用于描述利用 PowerShell 的模块的附加功能
type PowerShell struct {
    DisableAV   bool // 禁用 Windows 实时监控 "Set-MpPreference -DisableRealtimeMonitoring $true"
    Obfuscation bool // 未实现的命令，用于混淆 PowerShell
    Base64      bool // 对 PowerShell 命令进行 Base64 编码？
}

// Run 函数返回一个要在代理上执行模块的命令数组
func Run(m Module) ([]string, error) {
    if m.Agent == uuid.FromStringOrNil("00000000-0000-0000-0000-000000000000") {
        return nil, errors.New("未设置模块的代理")
    }

    if strings.ToLower(m.Agent.String()) != "ffffffff-ffff-ffff-ffff-ffffffffffff" {
        platform, platformError := agents.GetAgentFieldValue(m.Agent, "platform")
        if platformError != nil {
            return nil, platformError
        }

        if !strings.EqualFold(m.Platform, platform) {
            return nil, fmt.Errorf("%s 模块仅兼容 %s 平台。代理的平台是 %s", m.Name, m.Platform, platform)
        }
    }

    // 检查每个“required”选项，确保它不为空
    for _, v := range m.Options {
        if v.Required {
            if v.Value == "" {
                return nil, errors.New(v.Name + " 是必需的")
            }
        }
    }
}
    // 如果消息类型为"extended"，则执行以下操作
    if strings.ToLower(m.Type) == "extended" {
        // 获取扩展命令，并检查是否有错误
        extendedCommand, err := getExtendedCommand(&m)
        if err != nil {
            return nil, err
        }
        // 返回扩展命令
        return extendedCommand, nil
    }

    // 填充或移除选项值
    command := make([]string, len(m.Commands))
    copy(command, m.Commands)

    // 遍历选项列表
    for _, o := range m.Options {
        // 从后向前遍历命令列表
        for k := len(command) - 1; k >= 0; k-- {
            // 创建用于匹配选项名称的正则表达式
            reName := regexp.MustCompile(`(?iU)({{2}` + o.Name + `}{2})`)
            // 创建用于匹配选项标志的正则表达式
            reFlag := regexp.MustCompile(`(?iU)({{2}` + o.Name + `.Flag}{2})`)
            // 创建用于匹配选项值的正则表达式
            reValue := regexp.MustCompile(`(?iU)({{2}` + o.Name + `.Value}{2})`)
            // 检查是否设置了选项但没有标志或值修饰符
            if reName.MatchString(command[k]) {
                // 如果选项有值，则替换命令中的选项名称为标志和值
                if o.Value != "" {
                    flagSpace := ""
                    if o.Flag != "" {
                        flagSpace = " "
                    }
                    command[k] = reName.ReplaceAllString(command[k], o.Flag+flagSpace+o.Value)
                } else {
                    // 如果选项没有值，则从命令列表中移除该选项
                    command = append(command[:k], command[k+1:]...)
                }
                // 检查是否设置了选项并且只有标志修饰符
            } else if reFlag.MatchString(command[k]) {
                // 如果选项值为"true"，则替换命令中的选项标志
                if strings.ToLower(o.Value) == "true" {
                    command[k] = reFlag.ReplaceAllString(command[k], o.Flag)
                } else {
                    // 如果选项值不为"true"，则从命令列表中移除该选项
                    command = append(command[:k], command[k+1:]...)
                }
                // 检查是否设置了选项并且只有值修饰符
            } else if reValue.MatchString(command[k]) {
                // 如果选项有值，则替换命令中的选项值
                if o.Value != "" {
                    command[k] = reValue.ReplaceAllString(command[k], o.Value)
                } else {
                    // 如果选项没有值，则从命令列表中移除该选项
                    command = append(command[:k], command[k+1:]...)
                }
            }
        }
    }
    // 返回处理后的命令列表
    return command, nil
// ShowOptions函数用于显示模块的可配置选项
func (m *Module) ShowOptions() {
    // 使用颜色库输出Agent信息
    color.Cyan(fmt.Sprintf("\r\nAgent: %s\r\n", m.Agent.String()))
    // 使用颜色库输出模块选项信息
    color.Yellow("\r\nModule options(" + m.Name + ")\r\n\r\n")
    // 创建一个表格写入器
    table := tablewriter.NewWriter(os.Stdout)
    // 设置表头
    table.SetHeader([]string{"Name", "Value", "Required", "Description"})
    // 设置表格边框
    table.SetBorder(false)
    // 添加Agent别名选项
    // TODO 在这里更新tablewriter到最新版本，并使用SetColMinWidth设置Description列的最小宽度
    // TODO 在这里添加代理别名选项
    table.Append([]string{"Agent", m.Agent.String(), "true", "Agent on which to run module " + m.Name})
    // 遍历模块的选项，添加到表格中
    for _, v := range m.Options {
        table.Append([]string{v.Name, v.Value, strconv.FormatBool(v.Required), v.Description})
    }
    // 渲染表格
    table.Render()
}

// GetOptionsList生成并返回模块可配置选项的列表。与tab键补全一起使用
func (m *Module) GetOptionsList() func(string) []string {
    return func(line string) []string {
        o := make([]string, 0)
        // 遍历模块的选项，生成选项名称列表
        for _, v := range m.Options {
            o = append(o, v.Name)
        }
        return o
    }
}

// GetModuleList生成并返回Merlin“module”目录文件夹中所有模块的列表。与tab键补全一起使用
func GetModuleList() func(string) []string {
    # 定义一个匿名函数，接受一个字符串参数并返回一个字符串切片
    return func(line string) []string {
        # 拼接出模块目录的路径
        ModuleDir := path.Join(filepath.ToSlash(core.CurrentDir), "data", "modules")
        # 创建一个空的字符串切片
        o := make([]string, 0)

        # 遍历模块目录下的所有文件和子目录
        err := filepath.Walk(ModuleDir, func(path string, f os.FileInfo, err error) error {
            # 处理遍历过程中可能出现的错误
            if err != nil {
                fmt.Printf("prevent panic by handling failure accessing a path %q: %v\n", ModuleDir, err)
                return err
            }
            # 判断文件是否以".json"结尾
            if strings.HasSuffix(f.Name(), ".json") {
                # 将文件路径转换为斜杠分隔的路径，并从模块目录中提取出相对路径
                d := strings.SplitAfter(filepath.ToSlash(path), ModuleDir)
                if len(d) > 0 {
                    m := d[1]
                    m = strings.TrimLeft(m, "/")
                    m = strings.TrimSuffix(m, ".json")
                    # 如果路径中不包含"templates"，则将其添加到结果切片中
                    if !strings.Contains(m, "templates") {
                        o = append(o, m)
                    }
                }
            }
            return nil
        })
        # 处理遍历过程中可能出现的错误
        if err != nil {
            fmt.Printf("error walking the path %q: %v\n", ModuleDir, err)
        }
        # 返回结果切片
        return o
    }
// SetOption 用于更改传入模块选项的值。当用户配置模块时使用
func (m *Module) SetOption(option string, value []string) (string, error) {
    // 验证该选项是否存在
    for k, v := range m.Options {
        if option == v.Name {
            // 为包含空格的参数接受一个字符串切片；https://github.com/Ne0nd0g/merlin/issues/88
            m.Options[k].Value = strings.Join(value, " ")
            return fmt.Sprintf("%s set to %s", v.Name, m.Options[k].Value), nil
        }
    }
    return "", fmt.Errorf("invalid module option: %s", option)
}

// SetAgent 用于设置与模块关联的代理
func (m *Module) SetAgent(agentUUID string) (string, error) {
    if strings.ToLower(agentUUID) == "all" {
        agentUUID = "ffffffff-ffff-ffff-ffff-ffffffffffff"
    }
    i, err := uuid.FromString(agentUUID)
    if err != nil {
        return "", fmt.Errorf("invalid UUID")
    }
    m.Agent = i
    return fmt.Sprintf("agent set to %s", m.Agent.String()), nil
}

// ShowInfo 函数显示有关模块的所有信息，包括作者和选项等内容
func (m *Module) ShowInfo() {
    color.Yellow("Module:\r\n\t%s\r\n", m.Name)
    color.Yellow("Platform:\r\n\t%s\\%s\\%s\r\n", m.Platform, m.Arch, m.Lang)
    color.Yellow("Module Authors:")
    for a := range m.Author {
        color.Yellow("\t%s", m.Author[a])
    }
    color.Yellow("Credits:")
    for c := range m.Credits {
        color.Yellow("\t%s", m.Credits[c])
    }
    color.Yellow("Description:\r\n\t%s", m.Description)
    m.ShowOptions()
    fmt.Println()
    color.Yellow("Notes: %s", m.Notes)
}

// Create 是用于使用提供的模块 JSON 文件路径实例化模块对象的模块函数
func Create(modulePath string) (Module, error) {
    var m Module

    // 读取模块的 JSON 配置文件
    // 读取模块路径下的文件内容，#nosec G304 - 用户应该能够读取任何文件
    f, err := ioutil.ReadFile(modulePath)
    if err != nil {
        return m, err
    }

    // 反序列化模块的 JSON 消息
    var moduleJSON map[string]*json.RawMessage
    errModule := json.Unmarshal(f, &moduleJSON)
    if errModule != nil {
        return m, errModule
    }

    // 确定所有消息类型
    var keys []string
    for k := range moduleJSON {
        keys = append(keys, k)
    }

    // 验证模块的 JSON 至少包含基本消息
    var containsBase bool
    for i := range keys {
        if keys[i] == "base" {
            containsBase = true
        }
    }

    // 序列化基本消息类型
    if !containsBase {
        return m, errors.New("模块的定义不包含'BASE'消息类型")
    }
    errJSON := json.Unmarshal(*moduleJSON["base"], &m)
    if errJSON != nil {
        return m, errJSON
    }

    // 检查 PowerShell 配置选项
    for k := range keys {
        switch keys[k] {
        case "base":
        case "powershell":
            k := marshalMessage(*moduleJSON["powershell"])
            m.Powershell = (*json.RawMessage)(&k)
            var p PowerShell
            err := json.Unmarshal(k, &p)
            if err != nil {
                return m, errors.New("解组 powershell JSON 对象时出错")
            }
        }
    }

    // 验证模块
    _, errValidate := validateModule(m)
    if errValidate != nil {
        return m, errValidate
    }
    return m, nil
// validateModule function is used to check a module's configuration for errors
func validateModule(m Module) (bool, error) {

    // Validate Platform
    switch strings.ToUpper(m.Platform) {
    case "WINDOWS":
    case "LINUX":
    case "DARWIN":
    default:
        return false, errors.New("invalid or missing 'platform' value in the module's JSON file")
    }

    // Validate Architecture
    switch strings.ToUpper(m.Arch) {
    case "X64":
    case "X32":
    default:
        return false, errors.New("invalid or missing 'arch' value in the module's JSON file")
    }

    // Validate Type
    switch strings.ToUpper(m.Type) {
    case "STANDARD":
    case "EXTENDED":
    default:
        return false, errors.New("invalid or missing `type` value in the module's JSON file")
    }
    return true, nil
}

// marshalMessage is a generic function used to marshal JSON messages
func marshalMessage(m interface{}) []byte {
    k, err := json.Marshal(m)
    if err != nil {
        color.Red("There was an error marshaling the JSON object")
        color.Red(err.Error())
    }
    return k
}

// getExtendedCommand processes "extended" modules and returns the associated command by matching the extended module's
// name to a the Parse function of its associated module package
func getExtendedCommand(m *Module) ([]string, error) {
    // TODO document that every extended module must have a parse function as its entry point
    var extendedCommand []string
    var err error
    switch strings.ToLower(m.Name) {
    case "minidump":
        extendedCommand, err = minidump.Parse(m.getMapFromOptions())
    case "shellcodeinjection":
        extendedCommand, err = shellcode.Parse(m.getMapFromOptions())
    case "srdi":
        extendedCommand, err = srdi.Parse(m.getMapFromOptions())
    default:
        return nil, fmt.Errorf("the %s module's extended command function was not found", m.Name)
    }
    return extendedCommand, err
}
// getMapFromOptions函数用于生成一个包含模块选项名称和值的映射，以便在其他函数中使用
func (m *Module) getMapFromOptions() map[string]string {
    // 创建一个空的字符串映射
    optionsMap := make(map[string]string)

    // 遍历模块的选项列表，将选项名称和值存入映射中
    for _, v := range m.Options {
        optionsMap[v.Name] = v.Value
    }
    // 返回包含选项名称和值的映射
    return optionsMap
}
```