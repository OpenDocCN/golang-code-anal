# `kubesploit\pkg\logging\logging.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd.

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 由自由软件基金会发布，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

// 日志记录包
package logging
# 导入标准库
import (
	"fmt"  # 格式化输出
	"os"  # 操作系统功能
	"path/filepath"  # 文件路径操作
	"time"  # 时间操作

	# 导入第三方库
	"github.com/fatih/color"  # 用于在终端输出彩色文字

	# 导入自定义库
	"kubesploit/pkg/core"  # 导入自定义的核心功能包
)

# 定义全局变量 serverLog，用于存储服务器日志文件
var serverLog *os.File

# 初始化函数
func init() {

	# 服务器日志记录
	# 检查服务器日志文件是否存在
	if _, err := os.Stat(filepath.Join(core.CurrentDir, "data", "log", "merlinServerLog.txt")); os.IsNotExist(err) {
		// 创建一个名为 "log" 的目录，如果不存在的话，权限为 0750
		errM := os.MkdirAll(filepath.Join(core.CurrentDir, "data", "log"), 0750)
		// 如果创建目录时出现错误，输出警告信息
		if errM != nil {
			message("warn", "there was an error creating the log directory")
		}
		// 创建一个名为 "merlinServerLog.txt" 的文件，如果不存在的话
		serverLog, errC := os.Create(filepath.Join(core.CurrentDir, "data", "log", "merlinServerLog.txt"))
		// 如果创建文件时出现错误，输出警告信息并返回
		if errC != nil {
			message("warn", "there was an error creating the merlinServerLog.txt file")
			return
		}
		// 修改文件的权限为 0600
		errChmod := os.Chmod(serverLog.Name(), 0600)
		// 如果修改权限时出现错误，输出警告信息
		if errChmod != nil {
			message("warn", fmt.Sprintf("there was an error changing the file permissions for the agent log:\r\n%s", errChmod.Error()))
		}
		// 如果处于调试模式，输出调试信息
		if core.Debug {
			message("debug", fmt.Sprintf("Created server log file at: %s\\data\\log\\merlinServerLog.txt", core.CurrentDir))
		}
	}

	var errLog error
# 打开或创建一个名为merlinServerLog.txt的日志文件，并返回一个文件对象和可能的错误
serverLog, errLog = os.OpenFile(filepath.Join(core.CurrentDir, "data", "log", "merlinServerLog.txt"), os.O_APPEND|os.O_WRONLY, 0600)
# 如果打开或创建日志文件时出现错误，输出警告信息
if errLog != nil:
    message("warn", "there was an error with the Merlin Server log file")
    message("warn", errLog.Error())
}

// Server writes a log entry into the server's log file
// Server函数用于将日志条目写入服务器的日志文件
func Server(logMessage string) {
    // 将日志消息和时间戳格式化后写入服务器日志文件
    _, err := serverLog.WriteString(fmt.Sprintf("[%s]%s\r\n", time.Now().UTC().Format(time.RFC3339), logMessage))
    // 如果写入日志文件时出现错误，输出警告信息
    if err != nil {
        message("warn", "there was an error writing to the Merlin Server log file")
        message("warn", err.Error())
    }
}

// Message is used to print a message to the command line
// Message函数用于在命令行打印消息
func message(level string, message string) {
    // 根据消息级别输出相应的信息
    switch level {
    case "info":
// 根据消息级别使用不同颜色在命令行界面打印消息
switch level {
    // 如果消息级别为 info，则使用青色打印消息
    case "info":
        color.Cyan("[i]" + message)
    // 如果消息级别为 note，则使用黄色打印消息
    case "note":
        color.Yellow("[-]" + message)
    // 如果消息级别为 warn，则使用红色打印消息
    case "warn":
        color.Red("[!]" + message)
    // 如果消息级别为 debug，则使用红色打印消息
    case "debug":
        color.Red("[DEBUG]" + message)
    // 如果消息级别为 success，则使用绿色打印消息
    case "success":
        color.Green("[+]" + message)
    // 如果消息级别为其他未知级别，则使用红色打印错误消息
    default:
        color.Red("[_-_]Invalid message level: " + message)
    }
}

// TODO configure all message to be displayed on the CLI to be returned as errors and not written to the CLI here
// 待办事项：配置所有在命令行界面显示的消息都作为错误返回，而不在此处写入到命令行界面
```