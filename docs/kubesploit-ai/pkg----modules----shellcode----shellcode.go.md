# `kubesploit\pkg\modules\shellcode\shellcode.go`

```
/*
Kubesploit is a post-exploitation command and control framework built on top of Merlin by Russel Van Tuyl.
This file is part of Kubesploit.
Copyright (c) 2021 CyberArk Software Ltd. All rights reserved.

Kubesploit is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
any later version.

Kubesploit is distributed in the hope that it will be useful for enhancing organizations' security.
Kubesploit shall not be used in any malicious manner.
Kubesploit is distributed AS-IS, WITHOUT ANY WARRANTY; including the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with Kubesploit.  If not, see <http://www.gnu.org/licenses/>.
*/
*/

package shellcode

import (
    // Standard
    "encoding/base64"  // 导入 base64 编码包
    "encoding/hex"     // 导入十六进制编码包
    "errors"           // 导入错误处理包
    "fmt"              // 导入格式化包
    "io/ioutil"         // 导入输入输出工具包
    "os"               // 导入操作系统功能包
    "strconv"          // 导入字符串转换包
    "strings"          // 导入字符串处理包
)

// Parse is the initial entry point for all extended modules. All validation checks and processing will be performed here
// The function input types are limited to strings and therefore require additional processing
// Parse 函数是所有扩展模块的初始入口点。所有验证检查和处理将在此处执行
// 函数输入类型仅限于字符串，因此需要额外的处理
func Parse(options map[string]string) ([]string, error) {
    // 检查参数数量是否为3
    if len(options) != 3 {
        return nil, fmt.Errorf("3 arguments were expected, %d were provided", len(options))
    }
    var b64 string

    // 检查 shellcode 文件是否存在
    f, errF := os.Stat(options["shellcode"])
    if errF != nil {
        // 如果文件不存在，尝试解析为十六进制
        h, errH := parseHex([]string{options["shellcode"]})
        if errH != nil {
            return nil, errH
        }
        // 将十六进制编码转换为 base64 编码
        b64 = base64.StdEncoding.EncodeToString(h)
    } else {
        // 如果不是目录，则解析 shellcode 文件并进行 base64 编码
        if f.IsDir() {
            return nil, fmt.Errorf("a directory was provided instead of a file: %s", options["shellcode"])
        }
        b, errB := parseShellcodeFile(options["shellcode"])
        // 如果解析出错，则返回错误信息
        if errB != nil {
            return nil, fmt.Errorf("there was an error parsing the shellcode file:\r\n%s", errB.Error())
        }
        b64 = base64.StdEncoding.EncodeToString(b)
    }

    // 将 PID 转换为整数
    if options["pid"] != "" {
        _, errPid := strconv.Atoi(options["pid"])
        // 如果转换出错，则返回错误信息
        if errPid != nil {
            return nil, fmt.Errorf("there was an error converting the PID to an integer:\r\n%s", errPid.Error())
        }
    }

    // 如果执行方法不是 "self" 并且 PID 为空，则返回错误信息
    if strings.ToLower(options["method"]) != "self" && options["pid"] == "" {
        return nil, fmt.Errorf("a valid PID must be provided for any method except self")
    }

    // 验证执行方法是否是有效类型
    switch strings.ToLower(options["method"]) {
    case "self":
    case "remote":
    case "rtlcreateuserthread":
    case "userapc":
    default:
        return nil, fmt.Errorf("invalid shellcode execution method: %s", options["method"])
    }

    // 获取 shellcode 任务的命令
    command, errCommand := GetJob(options["method"], b64, options["pid"])
    // 如果获取命令出错，则返回错误信息
    if errCommand != nil {
        return nil, fmt.Errorf("there was an error getting the shellcode job:\r\n%s", errCommand.Error())
    }

    // 返回命令和空错误信息
    return command, nil
// GetJob函数返回一个字符串数组，其中包含要与agents.AddJob一起使用的命令，以正确的顺序
func GetJob(method string, shellcode string, pid string) ([]string, error) {
    // TODO shellcode输入需要进行Base64编码
    switch strings.ToLower(method) {
    case "self":
        return []string{"shellcode", "self", shellcode}, nil
    case "remote":
        return []string{"shellcode", "remote", pid, shellcode}, nil
    case "rtlcreateuserthread":
        return []string{"shellcode", "rtlcreateuserthread", pid, shellcode}, nil
    case "userapc":
        return []string{"shellcode", "userapc", pid, shellcode}, nil
    }
    return nil, errors.New("未提供有效的shellcode方法")
}

// parseHex函数评估一个字符串数组以确定其格式，并返回十六进制的字节数组
func parseHex(str []string) ([]byte, error) {
    hexString := strings.Join(str, "")

    data, err := base64.StdEncoding.DecodeString(hexString)
    if err == nil {
        s := string(data)
        hexString = s
    }

    // 查看字符串是否以0x为前缀
    if hexString[0:2] == "0x" {
        hexString = strings.Replace(hexString, "0x", "", -1)
        if strings.Contains(hexString, ",") {
            hexString = strings.Replace(hexString, ",", "", -1)
        }
        if strings.Contains(hexString, " ") {
            hexString = strings.Replace(hexString, " ", "", -1)
        }
    }

    // 查看字符串是否以\x为前缀
    if hexString[0:2] == "\\x" {
        hexString = strings.Replace(hexString, "\\x", "", -1)
        if strings.Contains(hexString, ",") {
            hexString = strings.Replace(hexString, ",", "", -1)
        }
        if strings.Contains(hexString, " ") {
            hexString = strings.Replace(hexString, " ", "", -1)
        }
    }

    h, errH := hex.DecodeString(hexString)

    return h, errH

}

// parseShellcodeFile函数解析路径，评估文件的内容，并返回shellcode的字节数组
func parseShellcodeFile(filePath string) ([]byte, error) {
    // 读取文件内容
    fileContents, err := ioutil.ReadFile(filePath) // #nosec G304 Users can include any file from anywhere
    if err != nil {
        return nil, err
    }

    // 尝试解析十六进制编码的字节
    hexBytes, errHex := parseHex([]string{string(fileContents)})

    // 如果解析字节时出现错误，则可能不是 ASCII 十六进制，因此继续处理
    if errHex == nil {
        return hexBytes, nil
    }

    // 检查是否是 Base64 编码的二进制数据
    base64Data, errB64 := base64.StdEncoding.DecodeString(string(fileContents))
    if errB64 == nil {
        return base64Data, nil
    }

    // 如果既不是十六进制编码也不是 Base64 编码，则直接返回文件内容
    return fileContents, nil
}
```