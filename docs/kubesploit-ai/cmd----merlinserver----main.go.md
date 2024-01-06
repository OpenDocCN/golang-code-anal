# `kubesploit\cmd\merlinserver\main.go`

```
// Kubesploit 是一个基于 Russel Van Tuyl 的 Merlin 框架构建的后渗透命令和控制框架。
// 本文件是 Kubesploit 的一部分。
// 版权所有 2021 CyberArk Software Ltd. 保留所有权利。

// Kubesploit 是自由软件：您可以根据 GNU 通用公共许可证的条款重新分发和/或修改它，
// 由自由软件基金会发布，无论是许可证的第3版还是以后的版本。

// Kubesploit 的分发是希望它对增强组织的安全性有所帮助。
// Kubesploit 不得以任何恶意方式使用。
// Kubesploit 按原样分发，没有任何担保；包括适销性或特定用途的隐含担保。请参阅
// GNU 通用公共许可证以获取更多详细信息。

// 您应该已经收到了 GNU 通用公共许可证的副本
// 与 Kubesploit 一起。如果没有，请参见 <http://www.gnu.org/licenses/>。

// 主包声明
package main
// 导入标准库中的 flag 和 os 包
import (
	"flag"
	"os"
	
	// 导入第三方库中的 github.com/fatih/color 包
	"github.com/fatih/color"
	
	// 导入自定义包中的 kubesploit/pkg、kubesploit/pkg/banner、kubesploit/pkg/cli、kubesploit/pkg/logging 包
	"kubesploit/pkg"
	"kubesploit/pkg/banner"
	"kubesploit/pkg/cli"
	"kubesploit/pkg/logging"
)

// 全局变量 build，用于存储构建版本信息
var build = "nonRelease"

// 主函数
func main() {
	// 记录服务器启动信息，包括 Kubesploit 服务器版本、Merlin 版本和构建版本
	logging.Server("Starting Kubesploit Server version " + kubesploitVersion.Version + ", Merlin version " + kubesploitVersion.Version + " build " + kubesploitVersion.Build)
# 设置 Usage 函数，用于打印 KubeSploit 服务器的信息
flag.Usage = func() {
    # 打印 KubeSploit 服务器的信息
    color.Blue("#################################################")
    color.Blue("#\t\tKubeSploit SERVER\t\t\t#")
    color.Blue("#################################################")
    color.Blue("Version: " + kubesploitVersion.Version)
    color.Blue("Merlin Version: " + kubesploitVersion.MerlinVersion)
    color.Blue("Build: " + build)
    color.Yellow("KubeSploit Server does not take any command line arguments")
    color.Yellow("Visit the Merlin wiki for additional information: https://merlin-c2.readthedocs.io/en/latest/")
    # 打印默认的 flag 信息
    flag.PrintDefaults()
    # 退出程序
    os.Exit(0)
}
# 解析命令行参数
flag.Parse()

# 打印 KubeSploit 服务器的横幅
color.Blue(banner.KubesploitBanner)
color.Blue("\t\tVersion: %s", kubesploitVersion.Version)
color.Blue("\t\tMerlin version: %s", kubesploitVersion.MerlinVersion)
color.Blue("\t\tBuild: %s", build)
color.Blue("\t\tGitHub: %s", "https://github.com/cyberark/kubesploit")
// 启动梅林命令行界面
cli.Shell()
```