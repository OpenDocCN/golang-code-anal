# `kubesploit\cmd\merlinagent\main.go`

```
// Kubesploit 是一个基于 Russel Van Tuyl 的 Merlin 框架构建的后渗透命令和控制框架。
// 本文件是 Kubesploit 的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit 是自由软件：您可以根据 GNU 通用公共许可证的条款重新分发和/或修改它，
// 无论是许可证的第3版还是以后的版本。

// Kubesploit 的分发希望能够有助于增强组织的安全性。
// Kubesploit 不得以任何恶意方式使用。
// Kubesploit 按原样分发，没有任何担保；包括对适销性或特定用途的隐含担保。请参阅
// GNU 通用公共许可证以获取更多详细信息。

// 您应该已经收到了 GNU 通用公共许可证的副本
// 与 Kubesploit 一起。如果没有，请参阅 <http://www.gnu.org/licenses/>。

// 主包声明
package main
// 导入标准库中的 flag、fmt、os、time 包
import (
	"flag"
	"fmt"
	"os"
	"time"
	
	// 导入第三方库中的 color 包
	"github.com/fatih/color"
	
	// 导入自定义包 merlin 和 agent
	merlin "kubesploit/pkg"
	"kubesploit/pkg/agent"
)

// 定义全局变量 url，并赋值为 "https://127.0.0.1:443"
var url = "https://127.0.0.1:443"
// 定义变量 protocol 为字符串 "h2"
var protocol = "h2"
// 定义变量 build 为字符串 "nonRelease"
var build = "nonRelease"
// 定义变量 psk 为字符串 "kubesploit"
var psk = "kubesploit"
// 定义变量 proxy 为空字符串
var proxy = ""
// 定义变量 host 为空字符串
var host = ""
// 定义变量 ja3 为空字符串
var ja3 = ""

// 主函数
func main() {
    // 定义并解析命令行参数
	verbose := flag.Bool("v", false, "Enable verbose output")
	version := flag.Bool("version", false, "Print the agent version and exit")
	debug := flag.Bool("debug", false, "Enable debug output")
	flag.StringVar(&url, "url", url, "Full URL for agent to connect to")
	flag.StringVar(&psk, "psk", psk, "Pre-Shared Key used to encrypt initial communications")
	flag.StringVar(&protocol, "proto", protocol, "Protocol for the agent to connect with [https (HTTP/1.1), http (HTTP/1.1 Clear-Text), h2 (HTTP/2), h2c (HTTP/2 Clear-Text), http3 (QUIC or HTTP/3.0)]")
	flag.StringVar(&proxy, "proxy", proxy, "Hardcoded proxy to use for http/1.1 traffic only that will override host configuration")
	flag.StringVar(&host, "host", host, "HTTP Host header")
	flag.StringVar(&ja3, "ja3", ja3, "JA3 signature string (not the MD5 hash). Overrides -proto flag")
	sleep := flag.Duration("sleep", 10000*time.Millisecond, "Time for agent to sleep")
	flag.Usage = usage
}
# 解析命令行参数
flag.Parse()

# 如果指定了-version参数，则打印Agent和Merlin的版本信息，并退出程序
if *version:
    color.Blue(fmt.Sprintf("Kubesploit Agent Version: %s", merlin.Version))
    color.Blue(fmt.Sprintf("Merlin Agent Version: %s", merlin.MerlinVersion))
    color.Blue(fmt.Sprintf("Kubesploit Agent Build: %s", build))
    os.Exit(0)

# 设置并运行Agent
a, err := agent.New(protocol, url, host, psk, proxy, ja3, *verbose, *debug)
if err != nil:
    # 如果设置了verbose参数，则打印错误信息
    if *verbose:
        color.Red(err.Error())
    os.Exit(1)

# 设置Agent的等待时间
a.WaitTime = *sleep
# 运行Agent
errRun := a.Run()
if errRun != nil:
// 如果 verbose 标志被设置为 true，则打印错误信息
if *verbose {
    color.Red(errRun.Error())
}
// 无论是否打印了错误信息，都退出程序并返回状态码 1
os.Exit(1)
// 结束 main 函数

// usage 函数用于打印命令行选项的使用方法
func usage() {
    // 打印程序名称
    fmt.Printf("Kubesploit Agent\r\n")
    // 打印命令行选项的默认值
    flag.PrintDefaults()
    // 退出程序并返回状态码 0
    os.Exit(0)
}
```