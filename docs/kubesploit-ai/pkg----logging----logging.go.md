# `kubesploit\pkg\logging\logging.go`

```
// Kubesploit 是建立在 Russel Van Tuyl 的 Merlin 之上的后渗透命令和控制框架。
// 本文件是 Kubesploit 的一部分。
// 版权所有 2021 CyberArk Software Ltd. 保留所有权利。

// Kubesploit 是自由软件：您可以根据 GNU 通用公共许可证的条款重新分发和/或修改它，
// 由自由软件基金会发布，无论是许可证的第3版还是以后的版本。

// Kubesploit 分发的目的是为了增强组织的安全性。
// Kubesploit 不得以任何恶意方式使用。
// Kubesploit 按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。请参阅
// GNU 通用公共许可证以获取更多详细信息。

// 您应该已经收到了 GNU 通用公共许可证的副本
// 与 Kubesploit 一起。如果没有，请参阅 <http://www.gnu.org/licenses/>。

package logging

import (
    // 标准库
    "fmt"
    "os"
    "path/filepath"
    "time"

    // 第三方库
    "github.com/fatih/color"

    // Merlin
    "kubesploit/pkg/core"
)

var serverLog *os.File

func init() {

    // 服务器日志记录
    # 检查指定路径下的文件是否存在
    if _, err := os.Stat(filepath.Join(core.CurrentDir, "data", "log", "merlinServerLog.txt")); os.IsNotExist(err) {
        # 如果文件不存在，则创建目录
        errM := os.MkdirAll(filepath.Join(core.CurrentDir, "data", "log"), 0750)
        if errM != nil:
            # 如果创建目录失败，则输出警告信息
            message("warn", "there was an error creating the log directory")
        # 创建日志文件
        serverLog, errC := os.Create(filepath.Join(core.CurrentDir, "data", "log", "merlinServerLog.txt"))
        if errC != nil:
            # 如果创建文件失败，则输出警告信息并返回
            message("warn", "there was an error creating the merlinServerLog.txt file")
            return
        # 修改文件权限为 0600
        errChmod := os.Chmod(serverLog.Name(), 0600)
        if errChmod != nil:
            # 如果修改文件权限失败，则输出警告信息
            message("warn", fmt.Sprintf("there was an error changing the file permissions for the agent log:\r\n%s", errChmod.Error()))
        # 如果处于调试模式，则输出调试信息
        if core.Debug:
            message("debug", fmt.Sprintf("Created server log file at: %s\\data\\log\\merlinServerLog.txt", core.CurrentDir))
    }

    # 打开日志文件，以追加写入模式
    var errLog error
    serverLog, errLog = os.OpenFile(filepath.Join(core.CurrentDir, "data", "log", "merlinServerLog.txt"), os.O_APPEND|os.O_WRONLY, 0600)
    if errLog != nil:
        # 如果打开文件失败，则输出警告信息
        message("warn", "there was an error with the Merlin Server log file")
        message("warn", errLog.Error())
    }
// Server函数用于将日志条目写入服务器的日志文件
func Server(logMessage string) {
    // 将日志消息格式化并写入服务器日志文件
    _, err := serverLog.WriteString(fmt.Sprintf("[%s]%s\r\n", time.Now().UTC().Format(time.RFC3339), logMessage))
    // 如果发生错误，打印警告消息并将错误信息写入日志
    if err != nil {
        message("warn", "there was an error writing to the Merlin Server log file")
        message("warn", err.Error())
    }
}

// Message函数用于在命令行打印消息
func message(level string, message string) {
    // 根据消息级别使用不同颜色打印消息
    switch level {
    case "info":
        color.Cyan("[i]" + message)
    case "note":
        color.Yellow("[-]" + message)
    case "warn":
        color.Red("[!]" + message)
    case "debug":
        color.Red("[DEBUG]" + message)
    case "success":
        color.Green("[+]" + message)
    default:
        color.Red("[_-_]Invalid message level: " + message)
    }
}

// TODO 配置所有要显示在命令行的消息以作为错误返回，而不是在此处写入命令行
```