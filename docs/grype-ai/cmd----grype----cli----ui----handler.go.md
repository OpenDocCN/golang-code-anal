# `grype\cmd\grype\cli\ui\handler.go`

```
// 导入 ui 包
package ui

// 导入 sync 包
import (
	"sync"

	// 导入 bubbletea 包
	tea "github.com/charmbracelet/bubbletea"
	// 导入 go-partybus 包
	"github.com/wagoodman/go-partybus"

	// 导入 bubbly 包
	"github.com/anchore/bubbly"
	// 导入 taskprogress 包
	"github.com/anchore/bubbly/bubbles/taskprogress"
	// 导入 event 包
	"github.com/anchore/grype/grype/event"
)

// 定义 Handler 结构体
var _ interface {
	bubbly.EventHandler
	bubbly.MessageListener
	bubbly.HandleWaiter
} = (*Handler)(nil)

// 定义 HandlerConfig 结构体
type HandlerConfig struct {
# 定义一个名为 TitleWidth 的整数变量
TitleWidth int
# 定义一个名为 AdjustDefaultTask 的函数变量，该函数接受一个 taskprogress.Model 类型的参数并返回一个 taskprogress.Model 类型的结果
AdjustDefaultTask func(taskprogress.Model) taskprogress.Model
# 定义一个名为 Handler 的结构体类型，包含 WindowSize、Running、Config 和 bubbly.EventHandler 四个字段
type Handler struct {
    WindowSize tea.WindowSizeMsg
    Running    *sync.WaitGroup
    Config     HandlerConfig
    bubbly.EventHandler
}
# 定义一个名为 DefaultHandlerConfig 的函数，返回一个默认的 HandlerConfig 结构体
func DefaultHandlerConfig() HandlerConfig {
    return HandlerConfig{
        TitleWidth: 30,
    }
}
# 定义一个名为 New 的函数，接受一个 HandlerConfig 类型的参数，并返回一个指向 Handler 结构体的指针
func New(cfg HandlerConfig) *Handler {
    # 创建一个新的事件分发器
    d := bubbly.NewEventDispatcher()
// 创建一个新的处理程序实例，设置事件处理器、运行状态、和配置信息
h := &Handler{
    EventHandler: d,
    Running:      &sync.WaitGroup{},
    Config:       cfg,
}

// 使用事件处理器注册所有支持的事件类型及其对应的处理函数
d.AddHandlers(map[partybus.EventType]bubbly.EventHandlerFn{
    event.UpdateVulnerabilityDatabase:  h.handleUpdateVulnerabilityDatabase,
    event.VulnerabilityScanningStarted: h.handleVulnerabilityScanningStarted,
    event.DatabaseDiffingStarted:       h.handleDatabaseDiffStarted,
})

// 返回处理程序实例
return h
}

// 处理接收到的消息
func (m *Handler) OnMessage(msg tea.Msg) {
    // 如果消息是窗口大小消息，则更新处理程序的窗口大小
    if msg, ok := msg.(tea.WindowSizeMsg); ok {
        m.WindowSize = msg
# 结束方法的定义
}

# 定义一个名为Wait的方法，参数为m，类型为Handler指针
func (m *Handler) Wait() {
    # 调用Running的Wait方法
    m.Running.Wait()
}
```