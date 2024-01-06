# `kubesploit\pkg\modules\shellcode\shellcode.go`

```
/*
Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
此文件是Kubesploit的一部分。
版权所有© 2021 CyberArk Software Ltd。

Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

Kubesploit的分发希望能够有助于增强组织的安全性。
Kubesploit不得以任何恶意方式使用。
Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。请参阅GNU通用公共许可证以获取更多详细信息。

您应该已经收到了GNU通用公共许可证的副本。
如果没有，请参阅<http://www.gnu.org/licenses/>。
*/
// 声明 shellcode 包
package shellcode

// 导入标准库
import (
	"encoding/base64" // 导入 base64 编码库
	"encoding/hex" // 导入十六进制编码库
	"errors" // 导入错误处理库
	"fmt" // 导入格式化输出库
	"io/ioutil" // 导入文件 I/O 库
	"os" // 导入操作系统库
	"strconv" // 导入字符串转换库
	"strings" // 导入字符串处理库
)

// Parse 是所有扩展模块的初始入口点。所有验证检查和处理都将在此处执行
// 函数输入类型限制为字符串，因此需要额外的处理
func Parse(options map[string]string) ([]string, error) {
	// 检查参数数量是否为3个
	if len(options) != 3 {
		// 如果参数数量不符合预期，返回错误信息
		return nil, fmt.Errorf("3 arguments were expected, %d were provided", len(options))
	}
// 声明一个变量 b64，用于存储 base64 编码后的 shellcode
var b64 string

// 获取文件信息
f, errF := os.Stat(options["shellcode"])
// 如果获取文件信息出错
if errF != nil {
    // 尝试解析十六进制字符串
    h, errH := parseHex([]string{options["shellcode"]})
    // 如果解析出错，返回错误
    if errH != nil {
        return nil, errH
    }
    // 将解析后的十六进制字符串进行 base64 编码
    b64 = base64.StdEncoding.EncodeToString(h)
} else {
    // 如果获取文件信息成功，检查是否是目录
    if f.IsDir() {
        // 如果是目录，返回错误
        return nil, fmt.Errorf("a directory was provided instead of a file: %s", options["shellcode"])
    }
    // 解析 shellcode 文件
    b, errB := parseShellcodeFile(options["shellcode"])
    // 如果解析出错，返回错误
    if errB != nil {
        return nil, fmt.Errorf("there was an error parsing the shellcode file:\r\n%s", errB.Error())
    }
    // 将解析后的 shellcode 进行 base64 编码
    b64 = base64.StdEncoding.EncodeToString(b)
}
// 将 PID 转换为整数
if options["pid"] != "" {
    // 尝试将 PID 转换为整数
    _, errPid := strconv.Atoi(options["pid"])
    // 如果转换出错，返回错误信息
    if errPid != nil {
        return nil, fmt.Errorf("there was an error converting the PID to an integer:\r\n%s", errPid.Error())
    }
}

// 如果方法不是 "self" 并且 PID 为空，则返回错误
if strings.ToLower(options["method"]) != "self" && options["pid"] == "" {
    return nil, fmt.Errorf("a valid PID must be provided for any method except self")
}

// 验证方法是否是有效的类型
switch strings.ToLower(options["method"]) {
case "self":
case "remote":
case "rtlcreateuserthread":
case "userapc":
default:
    return nil, fmt.Errorf("invalid shellcode execution method: %s", options["method"])
}
// 结束当前函数
	}
	// 调用GetJob函数，获取命令和错误信息
	command, errCommand := GetJob(options["method"], b64, options["pid"])
	// 如果获取命令时出现错误，返回错误信息
	if errCommand != nil {
		return nil, fmt.Errorf("there was an error getting the shellcode job:\r\n%s", errCommand.Error())
	}

	// 返回获取的命令
	return command, nil
}

// GetJob函数返回一个字符串数组，包含要与agents.AddJob一起使用的命令，以正确的顺序
func GetJob(method string, shellcode string, pid string) ([]string, error) {
	// TODO shellcode输入需要进行Base64编码
	// 根据method的值进行不同的处理
	switch strings.ToLower(method) {
	case "self":
		// 如果method为self，返回包含shellcode和self的字符串数组
		return []string{"shellcode", "self", shellcode}, nil
	case "remote":
		// 如果method为remote，返回包含shellcode、remote和pid的字符串数组
		return []string{"shellcode", "remote", pid, shellcode}, nil
	case "rtlcreateuserthread":
		// 如果method为rtlcreateuserthread，返回包含shellcode、rtlcreateuserthread、pid的字符串数组
		return []string{"shellcode", "rtlcreateuserthread", pid, shellcode}, nil
// 根据输入的字符串判断其格式并返回十六进制的字节数组
func parseHex(str []string) ([]byte, error) {
    // 将字符串数组连接成一个字符串
    hexString := strings.Join(str, "")

    // 尝试将字符串解码为 base64 编码的字节数组
    data, err := base64.StdEncoding.DecodeString(hexString)
    if err == nil {
        // 如果解码成功，将字节数组转换为字符串
        s := string(data)
        hexString = s
    }

    // 检查字符串是否以 "0x" 开头
    if hexString[0:2] == "0x" {
        // 如果是，去掉前缀 "0x"
        hexString = strings.Replace(hexString, "0x", "", -1)
        // 如果字符串中包含逗号
        if strings.Contains(hexString, ",") {
		// 如果字符串中包含逗号，则将逗号替换为空字符串
		hexString = strings.Replace(hexString, ",", "", -1)
		// 如果字符串中包含空格，则将空格替换为空字符串
		if strings.Contains(hexString, " ") {
			hexString = strings.Replace(hexString, " ", "", -1)
		}
	}

	// 检查字符串是否以\x开头
	if hexString[0:2] == "\\x" {
		// 如果字符串中包含\x，则将\x替换为空字符串
		hexString = strings.Replace(hexString, "\\x", "", -1)
		// 如果字符串中包含逗号，则将逗号替换为空字符串
		if strings.Contains(hexString, ",") {
			hexString = strings.Replace(hexString, ",", "", -1)
		}
		// 如果字符串中包含空格，则将空格替换为空字符串
		if strings.Contains(hexString, " ") {
			hexString = strings.Replace(hexString, " ", "", -1)
		}
	}

	// 将十六进制字符串解码为字节数组
	h, errH := hex.DecodeString(hexString)
// parseShellcodeFile函数解析一个文件路径，评估文件的内容，并返回shellcode的字节数组
func parseShellcodeFile(filePath string) ([]byte, error) {
    // 读取文件的内容
    fileContents, err := ioutil.ReadFile(filePath) // #nosec G304 Users can include any file from anywhere
    if err != nil {
        return nil, err
    }

    // 将文件内容解析为十六进制字节数组
    hexBytes, errHex := parseHex([]string{string(fileContents)})

    // 如果解析字节时出现错误，则可能不是ASCII十六进制，因此继续处理
    if errHex == nil {
        return hexBytes, nil
    }

    // 查看是否是Base64编码的二进制数据
# 使用标准 base64 编码对文件内容进行解码，将解码后的数据存储在 base64Data 中，错误信息存储在 errB64 中
base64Data, errB64 := base64.StdEncoding.DecodeString(string(fileContents))
# 如果解码过程中没有出现错误，则返回解码后的数据和空的错误信息
if errB64 == nil:
    return base64Data, nil

# 如果解码过程中出现错误，则返回原始文件内容和空的错误信息
return fileContents, nil
```