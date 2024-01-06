# `grype\internal\bus\helpers.go`

```
package bus

import (
	"github.com/wagoodman/go-partybus" // 导入第三方库 go-partybus

	"github.com/anchore/clio" // 导入 anchore 公司的 clio 库
	"github.com/anchore/grype/grype/event" // 导入 anchore 公司的 grype 事件库
	"github.com/anchore/grype/internal/redact" // 导入 anchore 公司的 grype 内部 redact 库
)

func Exit() {
	Publish(clio.ExitEvent(false)) // 调用 Publish 函数，发布 clio 库的 ExitEvent 事件，参数为 false
}

func ExitWithInterrupt() {
	Publish(clio.ExitEvent(true)) // 调用 Publish 函数，发布 clio 库的 ExitEvent 事件，参数为 true
}

func Report(report string) {
	Publish(partybus.Event{ // 调用 Publish 函数，发布 partybus 库的 Event 事件，参数为 report 字符串
# 定义一个事件类型为CLIReport的事件，值为对report进行redact.Apply()处理后的结果
Type:  event.CLIReport,
Value: redact.Apply(report),
})

# 定义一个通知函数，发布一个事件类型为CLINotification的事件，值为message
func Notify(message string) {
    Publish(partybus.Event{
        Type:  event.CLINotification,
        Value: message,
    })
}
```