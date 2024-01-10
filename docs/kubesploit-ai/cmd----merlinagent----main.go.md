# `kubesploit\cmd\merlinagent\main.go`

```
// Kubesploit 是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。有关更多详细信息，请参见GNU通用公共许可证。

// 您应该已经收到GNU通用公共许可证的副本。
// 如果没有，请参见<http://www.gnu.org/licenses/>。

package main

import (
    // 标准库
    "flag"  // 导入flag包，用于命令行参数解析
    "fmt"   // 导入fmt包，用于格式化输入输出
    merlin "kubesploit/pkg"  // 导入kubesploit/pkg包，并命名为merlin
    "kubesploit/pkg/agent"   // 导入kubesploit/pkg/agent包

    "os"    // 导入os包，提供了与操作系统交互的函数
    "time"  // 导入time包，提供了时间的函数

    // 第三方库
    "github.com/fatih/color"  // 导入github.com/fatih/color包，用于控制台输出颜色

    // Merlin
    //"kubesploit/pkg"
)

// 全局变量
var url = "https://127.0.0.1:443"  // 定义url变量并初始化为"https://127.0.0.1:443"
var protocol = "h2"  // 定义protocol变量并初始化为"h2"
var build = "nonRelease"  // 定义build变量并初始化为"nonRelease"
var psk = "kubesploit"  // 定义psk变量并初始化为"kubesploit"
var proxy = ""  // 定义proxy变量并初始化为空字符串
var host = ""  // 定义host变量并初始化为空字符串
var ja3 = ""  // 定义ja3变量并初始化为空字符串

func main() {
    verbose := flag.Bool("v", false, "Enable verbose output")  // 定义verbose变量并使用flag.Bool()函数解析命令行参数
    version := flag.Bool("version", false, "Print the agent version and exit")  // 定义version变量并使用flag.Bool()函数解析命令行参数
    debug := flag.Bool("debug", false, "Enable debug output")  // 定义debug变量并使用flag.Bool()函数解析命令行参数
    flag.StringVar(&url, "url", url, "Full URL for agent to connect to")  // 解析命令行参数并将结果存储到url变量中
    flag.StringVar(&psk, "psk", psk, "Pre-Shared Key used to encrypt initial communications")  // 解析命令行参数并将结果存储到psk变量中
    flag.StringVar(&protocol, "proto", protocol, "Protocol for the agent to connect with [https (HTTP/1.1), http (HTTP/1.1 Clear-Text), h2 (HTTP/2), h2c (HTTP/2 Clear-Text), http3 (QUIC or HTTP/3.0)]")  // 解析命令行参数并将结果存储到protocol变量中
    # 定义命令行参数 proxy，并指定默认值和说明
    flag.StringVar(&proxy, "proxy", proxy, "Hardcoded proxy to use for http/1.1 traffic only that will override host configuration")
    # 定义命令行参数 host，并指定默认值和说明
    flag.StringVar(&host, "host", host, "HTTP Host header")
    # 定义命令行参数 ja3，并指定默认值和说明
    flag.StringVar(&ja3, "ja3", ja3, "JA3 signature string (not the MD5 hash). Overrides -proto flag")
    # 定义命令行参数 sleep，并指定默认值和说明
    sleep := flag.Duration("sleep", 10000*time.Millisecond, "Time for agent to sleep")
    # 设置命令行参数的使用说明
    flag.Usage = usage
    # 解析命令行参数
    flag.Parse()

    # 如果命令行参数 version 为 true，则打印版本信息并退出程序
    if *version {
        color.Blue(fmt.Sprintf("Kubesploit Agent Version: %s", merlin.Version))
        color.Blue(fmt.Sprintf("Merlin Agent Version: %s", merlin.MerlinVersion))
        color.Blue(fmt.Sprintf("Kubesploit Agent Build: %s", build))
        os.Exit(0)
    }

    # 设置并运行 agent
    a, err := agent.New(protocol, url, host, psk, proxy, ja3, *verbose, *debug)
    # 如果创建 agent 出现错误，根据 verbose 参数打印错误信息并退出程序
    if err != nil {
        if *verbose {
            color.Red(err.Error())
        }
        os.Exit(1)
    }
    # 设置 agent 的等待时间为 sleep 参数的值
    a.WaitTime = *sleep
    # 运行 agent，如果出现错误，根据 verbose 参数打印错误信息并退出程序
    errRun := a.Run()
    if errRun != nil {
        if *verbose {
            color.Red(errRun.Error())
        }
        os.Exit(1)
    }
// usage 函数用于打印命令行选项的用法说明
func usage() {
    // 打印程序名称
    fmt.Printf("Kubesploit Agent\r\n")
    // 打印命令行选项的默认值
    flag.PrintDefaults()
    // 退出程序
    os.Exit(0)
}
```