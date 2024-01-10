# `kubesploit\pkg\modules\minidump\minidump.go`

```
/*
Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
此文件是Kubesploit的一部分。
版权所有©2021 CyberArk Software Ltd。

Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

Kubesploit的分发希望能够有助于增强组织的安全性。
Kubesploit不得以任何恶意方式使用。
Kubesploit按原样分发，没有任何担保；包括适销性或特定用途的隐含担保。有关更多详细信息，请参见GNU通用公共许可证。

您应该已经收到了GNU通用公共许可证的副本。
如果没有，请参见<http://www.gnu.org/licenses/>。

*/

package minidump

import (
    "fmt"
    "strconv"
)

// Parse是所有扩展模块的初始入口点。所有验证检查和处理将在此处执行
// 函数输入类型仅限于字符串，因此需要额外的处理
func Parse(options map[string]string) ([]string, error) {
    // 将PID转换为整数
    if options["pid"] != "" && options["pid"] != "0" {
        _, errPid := strconv.Atoi(options["pid"])
        if errPid != nil {
            return nil, fmt.Errorf("将PID转换为整数时出错：\r\n%s", errPid.Error())
        }
    }

    command, errCommand := GetJob(options["process"], options["pid"], options["tempLocation"])
    if errCommand != nil {
        return nil, fmt.Errorf("获取minidump作业时出错：\r\n%s", errCommand.Error())
    }

    return command, nil
}

// GetJob返回一个字符串数组，其中包含要与agents.AddJob一起使用的命令，以正确的顺序
func GetJob(process string, pid string, tempLocation string) ([]string, error) {
    # 返回一个包含字符串元素的列表，包括"Minidump"、process、pid、tempLocation
    return []string{"Minidump", process, pid, tempLocation}, nil
# 闭合前面的函数定义
```