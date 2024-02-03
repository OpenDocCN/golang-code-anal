# `grype\cmd\grype\cli\ui\handler.go`

```go
package ui

import (
    "sync"  // 导入 sync 包，用于实现同步功能

    tea "github.com/charmbracelet/bubbletea"  // 导入 bubbletea 包，并将其命名为 tea
    "github.com/wagoodman/go-partybus"  // 导入 go-partybus 包

    "github.com/anchore/bubbly"  // 导入 bubbly 包
    "github.com/anchore/bubbly/bubbles/taskprogress"  // 导入 taskprogress 模块
    "github.com/anchore/grype/grype/event"  // 导入 event 模块
)

var _ interface {
    bubbly.EventHandler  // 定义 bubbly.EventHandler 接口
    bubbly.MessageListener  // 定义 bubbly.MessageListener 接口
    bubbly.HandleWaiter  // 定义 bubbly.HandleWaiter 接口
} = (*Handler)(nil)  // 确保 Handler 类型实现了指定的接口

type HandlerConfig struct {
    TitleWidth        int  // 定义 HandlerConfig 结构体，包含 TitleWidth 字段
    AdjustDefaultTask func(taskprogress.Model) taskprogress.Model  // 定义 HandlerConfig 结构体，包含 AdjustDefaultTask 函数类型字段
}

type Handler struct {
    WindowSize tea.WindowSizeMsg  // 定义 Handler 结构体，包含 WindowSize 字段
    Running    *sync.WaitGroup  // 定义 Handler 结构体，包含 Running 字段，用于同步等待
    Config     HandlerConfig  // 定义 Handler 结构体，包含 Config 字段，类型为 HandlerConfig

    bubbly.EventHandler  // Handler 结构体实现 bubbly.EventHandler 接口
}

func DefaultHandlerConfig() HandlerConfig {
    return HandlerConfig{
        TitleWidth: 30,  // 返回默认的 HandlerConfig 结构体，设置 TitleWidth 为 30
    }
}

func New(cfg HandlerConfig) *Handler {
    d := bubbly.NewEventDispatcher()  // 创建事件分发器

    h := &Handler{
        EventHandler: d,  // 初始化 Handler 结构体，设置 EventHandler 为 d
        Running:      &sync.WaitGroup{},  // 初始化 Running 字段为一个新的 WaitGroup
        Config:       cfg,  // 初始化 Config 字段为传入的参数 cfg
    }

    // 注册所有支持的事件类型及其对应的处理函数
    d.AddHandlers(map[partybus.EventType]bubbly.EventHandlerFn{
        event.UpdateVulnerabilityDatabase:  h.handleUpdateVulnerabilityDatabase,  // 注册更新漏洞数据库事件的处理函数
        event.VulnerabilityScanningStarted: h.handleVulnerabilityScanningStarted,  // 注册漏洞扫描开始事件的处理函数
        event.DatabaseDiffingStarted:       h.handleDatabaseDiffStarted,  // 注册数据库差异开始事件的处理函数
    })

    return h  // 返回初始化后的 Handler 结构体
}

func (m *Handler) OnMessage(msg tea.Msg) {
    if msg, ok := msg.(tea.WindowSizeMsg); ok {  // 处理消息，如果是窗口大小消息
        m.WindowSize = msg  // 更新 Handler 结构体的 WindowSize 字段
    }
}

func (m *Handler) Wait() {
    m.Running.Wait()  // 等待 Running 字段的 WaitGroup 完成
}
```