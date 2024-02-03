# `grype\cmd\grype\internal\ui\no_ui.go`

```go
package ui

import (
    "os"  // 导入操作系统相关的包

    "github.com/wagoodman/go-partybus"  // 导入第三方库

    "github.com/anchore/clio"  // 导入 anchore/clio 包
    "github.com/anchore/grype/grype/event"  // 导入 anchore/grype/grype/event 包
)

var _ clio.UI = (*NoUI)(nil)  // 定义一个变量，用于实现 clio.UI 接口

type NoUI struct {
    finalizeEvents []partybus.Event  // 定义一个存储 partybus.Event 的切片
    subscription   partybus.Unsubscribable  // 定义一个 partybus.Unsubscribable 类型的变量
    quiet          bool  // 定义一个布尔类型的变量
}

func None(quiet bool) *NoUI {
    return &NoUI{
        quiet: quiet,  // 返回一个 NoUI 类型的指针，初始化 quiet 字段
    }
}

func (n *NoUI) Setup(subscription partybus.Unsubscribable) error {
    n.subscription = subscription  // 设置 subscription 字段为传入的参数
    return nil  // 返回空值
}

func (n *NoUI) Handle(e partybus.Event) error {
    switch e.Type {  // 根据事件类型进行判断
    case event.CLIReport, event.CLINotification:  // 如果事件类型是 CLIReport 或 CLINotification
        // keep these for when the UI is terminated to show to the screen (or perform other events)
        n.finalizeEvents = append(n.finalizeEvents, e)  // 将事件添加到 finalizeEvents 切片中
    }
    return nil  // 返回空值
}

func (n NoUI) Teardown(_ bool) error {
    return newPostUIEventWriter(os.Stdout, os.Stderr).write(n.quiet, n.finalizeEvents...)  // 调用 newPostUIEventWriter 方法，并传入参数进行写入操作
}
```