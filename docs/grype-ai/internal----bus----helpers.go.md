# `grype\internal\bus\helpers.go`

```go
// 导入 bus 包
package bus

// 导入 go-partybus 包
import (
    "github.com/wagoodman/go-partybus"

    // 导入 anchore 的 clio 包
    "github.com/anchore/clio"
    // 导入 anchore 的 grype 包下的 event 包
    "github.com/anchore/grype/grype/event"
    // 导入 anchore 的 grype 包下的 internal/redact 包
    "github.com/anchore/grype/internal/redact"
)

// 定义 Exit 函数
func Exit() {
    // 发布一个 clio.ExitEvent 事件，参数为 false
    Publish(clio.ExitEvent(false))
}

// 定义 ExitWithInterrupt 函数
func ExitWithInterrupt() {
    // 发布一个 clio.ExitEvent 事件，参数为 true
    Publish(clio.ExitEvent(true))
}

// 定义 Report 函数
func Report(report string) {
    // 发布一个 partybus.Event 事件，类型为 event.CLIReport，值为经过 redact.Apply 处理的 report
    Publish(partybus.Event{
        Type:  event.CLIReport,
        Value: redact.Apply(report),
    })
}

// 定义 Notify 函数
func Notify(message string) {
    // 发布一个 partybus.Event 事件，类型为 event.CLINotification，值为 message
    Publish(partybus.Event{
        Type:  event.CLINotification,
        Value: message,
    })
}
```