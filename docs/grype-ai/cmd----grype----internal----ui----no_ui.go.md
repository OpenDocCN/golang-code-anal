# `grype\cmd\grype\internal\ui\no_ui.go`

```
package ui
# 声明了一个名为 ui 的包

import (
	"os"
	# 导入了 os 包，用于操作系统功能

	"github.com/wagoodman/go-partybus"
	# 导入了 wagoodman/go-partybus 包

	"github.com/anchore/clio"
	# 导入了 anchore/clio 包，用于命令行输入输出

	"github.com/anchore/grype/grype/event"
	# 导入了 anchore/grype/grype/event 包，用于事件处理
)

var _ clio.UI = (*NoUI)(nil)
# 声明了一个变量，用于实现 clio.UI 接口

type NoUI struct {
	finalizeEvents []partybus.Event
	subscription   partybus.Unsubscribable
	quiet          bool
}
# 声明了一个名为 NoUI 的结构体，包含了 finalizeEvents、subscription 和 quiet 三个字段

func None(quiet bool) *NoUI {
# 声明了一个名为 None 的函数，接受一个 bool 类型的参数 quiet，并返回一个 NoUI 结构体指针
# 创建一个 NoUI 结构体的实例，并设置 quiet 属性
return &NoUI{
    quiet: quiet,
}

# 设置 NoUI 实例的订阅对象
func (n *NoUI) Setup(subscription partybus.Unsubscribable) error {
    n.subscription = subscription
    return nil
}

# 处理事件的方法，根据事件类型进行不同的处理
func (n *NoUI) Handle(e partybus.Event) error {
    switch e.Type {
    case event.CLIReport, event.CLINotification:
        # 将事件添加到 finalizeEvents 列表中，以便在 UI 终止时显示到屏幕上（或执行其他事件）
        n.finalizeEvents = append(n.finalizeEvents, e)
    }
    return nil
}

# 清理方法，根据参数进行不同的清理操作
func (n NoUI) Teardown(_ bool) error {
# 返回一个新的PostUIEventWriter对象，该对象将事件写入标准输出和标准错误流
return newPostUIEventWriter(os.Stdout, os.Stderr).write(n.quiet, n.finalizeEvents...)
```