# `kubesploit\cmd\merlinserver\main.go`

```go
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
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
    "flag"
    "os"

    // 第三方库
    "github.com/fatih/color"

    // Merlin
    "kubesploit/pkg"
    "kubesploit/pkg/banner"
    "kubesploit/pkg/cli"
    "kubesploit/pkg/logging"
)

// 全局变量
var build = "nonRelease"

func main() {
    // 记录Kubesploit服务器的启动信息，包括Kubesploit版本、Merlin版本和构建信息
    logging.Server("Starting Kubesploit Server version " + kubesploitVersion.Version + ", Merlin version " + kubesploitVersion.Version + " build " + kubesploitVersion.Build)
    # 设置 Usage 函数，用于显示 KubeSploit 服务器的信息和版本
    flag.Usage = func() {
        # 打印 KubeSploit 服务器的标题
        color.Blue("#################################################")
        color.Blue("#\t\tKubeSploit SERVER\t\t\t#")
        color.Blue("#################################################")
        # 打印 KubeSploit 服务器的版本信息
        color.Blue("Version: " + kubesploitVersion.Version)
        color.Blue("Merlin Version: " + kubesploitVersion.MerlinVersion)
        color.Blue("Build: " + build)
        # 打印 KubeSploit 服务器不接受命令行参数的提示
        color.Yellow("KubeSploit Server does not take any command line arguments")
        # 提示用户访问 Merlin wiki 获取更多信息
        color.Yellow("Visit the Merlin wiki for additional information: https://merlin-c2.readthedocs.io/en/latest/")
        # 打印默认的 flag 值
        flag.PrintDefaults()
        # 退出程序
        os.Exit(0)
    }
    # 解析命令行参数
    flag.Parse()

    # 打印 KubeSploit 的横幅
    color.Blue(banner.KubesploitBanner)
    # 打印 KubeSploit 的版本信息
    color.Blue("\t\tVersion: %s", kubesploitVersion.Version)
    color.Blue("\t\tMerlin version: %s", kubesploitVersion.MerlinVersion)
    color.Blue("\t\tBuild: %s", build)
    color.Blue("\t\tGitHub: %s", "https://github.com/cyberark/kubesploit")

    # 启动 Merlin 命令行界面
    // Start Merlin Command Line Interface
    cli.Shell()
# 闭合前面的函数定义
```