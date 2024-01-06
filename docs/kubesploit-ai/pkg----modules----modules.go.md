# `kubesploit\pkg\modules\modules.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd.

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 其中包括许可证的第3版或任何以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何担保；包括适销性或特定用途的隐含担保。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参阅<http://www.gnu.org/licenses/>。

// 模块包
package modules
# 导入标准库
import (
	"encoding/json"  # 导入用于 JSON 编解码的包
	"errors"  # 导入用于定义错误的包
	"fmt"  # 导入用于格式化输出的包
	"io/ioutil"  # 导入用于读取文件内容的包
	"os"  # 导入操作系统相关的包
	"path"  # 导入处理文件路径的包
	"path/filepath"  # 导入处理文件路径的包
	"regexp"  # 导入正则表达式的包
	"strconv"  # 导入字符串和数字之间转换的包
	"strings"  # 导入处理字符串的包

	# 导入第三方库
	"github.com/fatih/color"  # 导入用于在终端输出彩色文本的包
	"github.com/olekukonko/tablewriter"  # 导入用于在终端输出表格的包
	"github.com/satori/go.uuid"  # 导入用于生成 UUID 的包

	# 导入自定义的 Merlin 库
	"kubesploit/pkg/agents"  # 导入自定义的代理程序包
// 导入所需的包
"kubesploit/pkg/core"
"kubesploit/pkg/modules/minidump"
"kubesploit/pkg/modules/shellcode"
"kubesploit/pkg/modules/srdi"
)

// Module 结构包含模块的基本信息或模板
type Module struct {
	Agent        uuid.UUID      // 将在执行前与此模块关联的代理
	Name         string         `json:"name"`                 // 模块的名称
	Type         string         `json:"type"`                 // 模块的类型（标准或扩展）
	Author       []string       `json:"author"`               // 模块作者的列表
	Credits      []string       `json:"credits"`              // 用于底层工具或技术的人员名单
	Path         []string       `json:"path"`                 // 模块的路径（例如，data/modules/powershell/powerview）
	Platform     string         `json:"platform"`             // 模块可以运行的平台（例如，Windows、Linux、Darwin或ALL）
	Arch         string         `json:"arch"`                 // 模块可以运行的架构（例如，x86、x64、MIPS、ARM或ALL）
	Lang         string         `json:"lang"`                 // 模块执行的语言（例如，PowerShell、Python或Perl）
	Priv         bool           `json:"privilege"`            // 此模块是否需要特权级别帐户，如root或SYSTEM？
	Description  string         `json:"description"`          // 模块的描述
	Notes        string         `json:"notes"`                // 关于模块的其他信息或注释
// Commands is a list of commands to be run on the agent
// SourceRemote is the online or remote source code for a module
// SourceLocal is the local file path to the script or payload
// Options is a list of configurable options/arguments for the module
// Powershell is an option json object containing commands and configuration items specific to PowerShell
// GoInterpreter is a notification for Go code for the agent to use yaegi (Go interpreter)
// GoInterpreterProgress is a notification for Go code that prints while still in progress for the agent to use yaegi (Go interpreter)
// LoadScriptFromPath is a notification for loading script from path into commands with the
// TODO add notification part

// Option is a structure containing the keys for the object
type Option struct {
    Name        string `json:"name"`        // Name of the option
    Value       string `json:"value"`       // Value of the option
    Required    bool   `json:"required"`    // Is this a required option?
    Flag        string `json:"flag"`        // The command line flag used for the option
    Description string `json:"description"` // A description of the option
}
// PowerShell 结构用于描述利用 PowerShell 的模块的附加功能
type PowerShell struct {
	DisableAV   bool // 是否禁用 Windows 实时监控 "Set-MpPreference -DisableRealtimeMonitoring $true"
	Obfuscation bool // 未实现的命令，用于混淆 PowerShell
	Base64      bool // 是否对 PowerShell 命令进行 Base64 编码
}

// Run 函数返回一个要在代理上执行模块的命令数组
func Run(m Module) ([]string, error) {
	// 如果模块的代理未设置，则返回错误
	if m.Agent == uuid.FromStringOrNil("00000000-0000-0000-0000-000000000000") {
		return nil, errors.New("agent not set for module")
	}

	// 如果模块的代理不是默认值，则获取代理的平台信息
	if strings.ToLower(m.Agent.String()) != "ffffffff-ffff-ffff-ffff-ffffffffffff" {
		platform, platformError := agents.GetAgentFieldValue(m.Agent, "platform")
		if platformError != nil {
			return nil, platformError
		}

		// 如果模块的平台与代理的平台不匹配，则...
		// 如果模块的平台与代理的平台不兼容，则返回错误
		return nil, fmt.Errorf("the %s module is only compatible with %s platform. The agent's platform is %s", m.Name, m.Platform, platform)
	}

	// 检查每个“required”选项，确保它不是空值
	for _, v := range m.Options {
		if v.Required {
			if v.Value == "" {
				// 如果必填选项的值为空，则返回错误
				return nil, errors.New(v.Name + " is required")
			}
		}
	}

	// 如果模块类型为“extended”，则获取扩展命令并返回
	if strings.ToLower(m.Type) == "extended" {
		extendedCommand, err := getExtendedCommand(&m)
		if err != nil {
			// 如果获取扩展命令时出现错误，则返回该错误
			return nil, err
		}
		// 返回获取的扩展命令
		return extendedCommand, nil
	}
// 填充或移除选项值
command := make([]string, len(m.Commands))  // 创建一个与 m.Commands 长度相同的切片
copy(command, m.Commands)  // 复制 m.Commands 的值到 command 切片中

for _, o := range m.Options {  // 遍历 m.Options
    for k := len(command) - 1; k >= 0; k-- {  // 从后往前遍历 command 切片
        reName := regexp.MustCompile(`(?iU)({{2}` + o.Name + `}{2})`)  // 创建一个正则表达式，用于匹配 o.Name 的名称
        reFlag := regexp.MustCompile(`(?iU)({{2}` + o.Name + `.Flag}{2})`)  // 创建一个正则表达式，用于匹配 o.Name 的标志
        reValue := regexp.MustCompile(`(?iU)({{2}` + o.Name + `.Value}{2})`)  // 创建一个正则表达式，用于匹配 o.Name 的值
        // 检查是否设置了选项，但没有标志或值的限定符
        if reName.MatchString(command[k]) {  // 检查 command[k] 是否匹配 reName 正则表达式
            if o.Value != "" {  // 如果选项的值不为空
                flagSpace := ""  // 初始化标志空间为空字符串
                if o.Flag != "" {  // 如果选项的标志不为空
                    flagSpace = " "  // 设置标志空间为一个空格
                }
                command[k] = reName.ReplaceAllString(command[k], o.Flag+flagSpace+o.Value)  // 用选项的标志和值替换 command[k] 中匹配 reName 的部分
            } else {
                command = append(command[:k], command[k+1:]...)  // 如果选项的值为空，则从 command 中移除该选项
				}
				// 检查是否设置了选项，并且只有标志限定符
			} else if reFlag.MatchString(command[k]) {
				// 如果值为true，则用选项的标志替换命令中的限定符
				if strings.ToLower(o.Value) == "true" {
					command[k] = reFlag.ReplaceAllString(command[k], o.Flag)
				} else {
					// 否则，从命令中移除该选项
					command = append(command[:k], command[k+1:]...)
				}
				// 检查是否设置了选项，并且只有值限定符
			} else if reValue.MatchString(command[k]) {
				// 如果选项的值不为空，则用选项的值替换命令中的限定符
				if o.Value != "" {
					command[k] = reValue.ReplaceAllString(command[k], o.Value)
				} else {
					// 否则，从命令中移除该选项
					command = append(command[:k], command[k+1:]...)
				}
			}
		}
	}
	// 返回处理后的命令和空错误
	return command, nil
}
// ShowOptions 函数用于显示模块的可配置选项
func (m *Module) ShowOptions() {
    // 显示代理信息
    color.Cyan(fmt.Sprintf("\r\nAgent: %s\r\n", m.Agent.String()))
    // 显示模块选项
    color.Yellow("\r\nModule options(" + m.Name + ")\r\n\r\n")
    // 创建表格
    table := tablewriter.NewWriter(os.Stdout)
    // 设置表头
    table.SetHeader([]string{"Name", "Value", "Required", "Description"})
    // TODO 更新 tablewriter 到最新版本，并为 Description 列使用 SetColMinWidth
    table.SetBorder(false)
    // TODO 在这里为代理别名添加选项
    table.Append([]string{"Agent", m.Agent.String(), "true", "Agent on which to run module " + m.Name})
    // 遍历模块的选项，添加到表格中
    for _, v := range m.Options {
        table.Append([]string{v.Name, v.Value, strconv.FormatBool(v.Required), v.Description})
    }
    // 渲染表格
    table.Render()
}

// GetOptionsList 生成并返回模块的可配置选项列表，用于制表完成
func (m *Module) GetOptionsList() func(string) []string {
    return func(line string) []string {
		// 创建一个空字符串切片
		o := make([]string, 0)
		// 遍历选项列表，将选项的名称添加到切片中
		for _, v := range m.Options {
			o = append(o, v.Name)
		}
		// 返回选项名称的切片
		return o
	}
}

// GetModuleList 生成并返回 Merlin 的 "module" 目录文件夹中所有模块的列表。用于选项补全
func GetModuleList() func(string) []string {
	return func(line string) []string {
		// 拼接模块目录的路径
		ModuleDir := path.Join(filepath.ToSlash(core.CurrentDir), "data", "modules")
		// 创建一个空字符串切片
		o := make([]string, 0)

		// 遍历模块目录，处理每个文件
		err := filepath.Walk(ModuleDir, func(path string, f os.FileInfo, err error) error {
			// 处理访问路径失败的情况
			if err != nil {
				fmt.Printf("prevent panic by handling failure accessing a path %q: %v\n", ModuleDir, err)
				return err
			}
			// 如果文件名以 ".json" 结尾，将文件名添加到切片中
			if strings.HasSuffix(f.Name(), ".json") {
// 将路径转换为斜杠分隔的字符串，并根据模块目录进行分割
d := strings.SplitAfter(filepath.ToSlash(path), ModuleDir)

// 如果分割后的结果长度大于0
if len(d) > 0 {
    // 取出第一个分割后的字符串作为模块名
    m := d[1]
    // 去掉字符串开头的斜杠
    m = strings.TrimLeft(m, "/")
    // 去掉字符串结尾的 .json
    m = strings.TrimSuffix(m, ".json")
    // 如果字符串中不包含 "templates"
    if !strings.Contains(m, "templates") {
        // 将模块名添加到结果列表中
        o = append(o, m)
    }
}

// 如果有错误发生
if err != nil {
    // 打印错误信息
    fmt.Printf("error walking the path %q: %v\n", ModuleDir, err)
}

// 返回结果列表
return o
// SetOption函数用于设置模块的选项，接受一个选项名和一个字符串数组作为参数，返回一个字符串和一个错误
func (m *Module) SetOption(option string, value []string) (string, error) {
	// 遍历模块的选项，检查是否存在传入的选项名
	for k, v := range m.Options {
		if option == v.Name {
			// 将传入的字符串数组连接成一个字符串，用空格分隔，赋值给选项的值
			m.Options[k].Value = strings.Join(value, " ")
			return fmt.Sprintf("%s set to %s", v.Name, m.Options[k].Value), nil
		}
	}
	// 如果不存在传入的选项名，则返回错误
	return "", fmt.Errorf("invalid module option: %s", option)
}

// SetAgent函数用于设置与模块关联的代理
func (m *Module) SetAgent(agentUUID string) (string, error) {
	// 如果传入的代理UUID为"all"，则将其设置为默认值
	if strings.ToLower(agentUUID) == "all" {
		agentUUID = "ffffffff-ffff-ffff-ffff-ffffffffffff"
	}
	// 将传入的代理UUID转换为UUID对象
	i, err := uuid.FromString(agentUUID)
	if err != nil {
		// 如果转换失败，则返回错误
		return "", fmt.Errorf("invalid UUID")
	}
	// 其他操作...
}
// ShowInfo函数用于显示模块的所有信息，包括作者和选项等内容
func (m *Module) ShowInfo() {
    // 显示模块名称
    color.Yellow("Module:\r\n\t%s\r\n", m.Name)
    // 显示平台、架构和语言信息
    color.Yellow("Platform:\r\n\t%s\\%s\\%s\r\n", m.Platform, m.Arch, m.Lang)
    // 显示模块作者
    color.Yellow("Module Authors:")
    for a := range m.Author {
        color.Yellow("\t%s", m.Author[a])
    }
    // 显示模块贡献者
    color.Yellow("Credits:")
    for c := range m.Credits {
        color.Yellow("\t%s", m.Credits[c])
    }
    // 显示模块描述
    color.Yellow("Description:\r\n\t%s", m.Description)
    // 调用ShowOptions函数显示模块选项
    m.ShowOptions()
    fmt.Println()
}
// 使用黄色打印模块的注释信息
color.Yellow("Notes: %s", m.Notes)
}

// Create 是一个模块函数，用于使用提供的模块 JSON 文件路径实例化一个模块对象
func Create(modulePath string) (Module, error) {
	var m Module

	// 读取模块的 JSON 配置文件
	f, err := ioutil.ReadFile(modulePath) // #nosec G304 - 用户应该能够读取任何文件
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

// 编组基本消息类型
if !containsBase {
    return m, errors.New("the module's definition does not contain the 'BASE' message type")
}
errJSON := json.Unmarshal(*moduleJSON["base"], &m)
if errJSON != nil {
		return m, errJSON
	}

	// 检查 PowerShell 配置选项
	for k := range keys {
		switch keys[k] {
		case "base":
		// 如果是 "base" 选项，不做任何操作
		case "powershell":
			// 如果是 "powershell" 选项，处理 PowerShell 配置
			k := marshalMessage(*moduleJSON["powershell"])
			// 将 PowerShell 配置信息转换为 JSON 格式
			m.Powershell = (*json.RawMessage)(&k)
			var p PowerShell
			// 将 JSON 格式的 PowerShell 配置信息解析为结构体
			err := json.Unmarshal(k, &p)
			if err != nil {
				// 如果解析出错，返回错误信息
				return m, errors.New("there was an error unmarshaling the powershell JSON object")
			}
		}
	}

	// 验证模块是否有效
	_, errValidate := validateModule(m)
	if errValidate != nil {
		return m, errValidate
	}
	return m, nil
}

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
	// 如果架构为 X64，则继续执行下面的代码
	case "X86":
	// 如果架构为 X86，则继续执行下面的代码
	default:
	// 如果架构不是 X64 或 X86，则返回错误信息
		return false, errors.New("invalid or missing 'arch' value in the module's JSON file")
	}
	// 如果通过所有验证，则返回 true 和 nil
	return true, nil
}
// 根据不同的架构类型进行判断和处理
switch arch {
case "X32":
    // 如果架构类型为X32，则返回false和错误信息
    return false, errors.New("invalid or missing 'arch' value in the module's JSON file")
default:
    // 如果架构类型不为X32，则返回false和错误信息
    return false, errors.New("invalid or missing 'arch' value in the module's JSON file")
}

// 验证模块类型
switch strings.ToUpper(m.Type) {
case "STANDARD":
    // 如果模块类型为STANDARD，则继续执行
case "EXTENDED":
    // 如果模块类型为EXTENDED，则继续执行
default:
    // 如果模块类型不为STANDARD或EXTENDED，则返回false和错误信息
    return false, errors.New("invalid or missing `type` value in the module's JSON file")
}
// 返回true和nil
return true, nil
}

// marshalMessage是一个用于编组JSON消息的通用函数
func marshalMessage(m interface{}) []byte {
    // 将接口类型的m编组成JSON格式的字节流
    k, err := json.Marshal(m)
    // 如果编组过程中出现错误，则打印错误信息
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
	// 声明变量
	var extendedCommand []string
	var err error
	// 根据模块名匹配对应模块包的解析函数，返回扩展命令
	switch strings.ToLower(m.Name) {
	case "minidump":
		// 调用 minidump 模块的解析函数
		extendedCommand, err = minidump.Parse(m.getMapFromOptions())
	case "shellcodeinjection":
		// 调用 shellcodeinjection 模块的解析函数
		extendedCommand, err = shellcode.Parse(m.getMapFromOptions())
	case "srdi":
		// 调用 srdi 模块的解析函数
		extendedCommand, err = srdi.Parse(m.getMapFromOptions())
	default:
		// 如果模块名不匹配，则返回错误
		return nil, fmt.Errorf("the %s module's extended command function was not found", m.Name)
```

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