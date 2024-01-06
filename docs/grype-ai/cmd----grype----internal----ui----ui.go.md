# `grype\cmd\grype\internal\ui\ui.go`

```
package ui
// 导入所需的包

import (
	"fmt" // 导入格式化输出包
	"os" // 导入操作系统包
	"sync" // 导入同步包
	"time" // 导入时间包

	tea "github.com/charmbracelet/bubbletea" // 导入 bubbletea 包
	"github.com/wagoodman/go-partybus" // 导入 go-partybus 包

	"github.com/anchore/bubbly" // 导入 bubbly 包
	"github.com/anchore/bubbly/bubbles/frame" // 导入 frame 包
	"github.com/anchore/clio" // 导入 clio 包
	"github.com/anchore/go-logger" // 导入 go-logger 包
	"github.com/anchore/grype/grype/event" // 导入 event 包
	"github.com/anchore/grype/internal/bus" // 导入 bus 包
	"github.com/anchore/grype/internal/log" // 导入 log 包
)
# 定义一个接口变量，包含了tea.Model、partybus.Responder和clio.UI接口的方法
var _ interface {
	tea.Model
	partybus.Responder
	clio.UI
} = (*UI)(nil)

# 定义UI结构体，包含了程序、运行状态、是否安静、订阅、最终事件等属性
type UI struct {
	program        *tea.Program
	running        *sync.WaitGroup
	quiet          bool
	subscription   partybus.Unsubscribable
	finalizeEvents []partybus.Event

	handler *bubbly.HandlerCollection
	frame   tea.Model
}

# 创建一个新的UI对象，传入是否安静和事件处理器
func New(quiet bool, handlers ...bubbly.EventHandler) *UI {
	# 返回一个UI对象，包含了传入的事件处理器
	return &UI{
		handler: bubbly.NewHandlerCollection(handlers...),
		frame:   frame.New(),  // 创建一个新的 frame 对象并赋值给 UI 结构体的 frame 字段
		running: &sync.WaitGroup{},  // 创建一个新的 WaitGroup 对象并赋值给 UI 结构体的 running 字段
		quiet:   quiet,  // 将 quiet 参数赋值给 UI 结构体的 quiet 字段
	}
}

func (m *UI) Setup(subscription partybus.Unsubscribable) error {
	// 我们仍然希望收集日志消息，但是日志记录器不应直接写入屏幕
	if logWrapper, ok := log.Get().(logger.Controller); ok {  // 检查是否日志记录器实现了 Controller 接口
		logWrapper.SetOutput(m.frame.(*frame.Frame).Footer())  // 将日志输出设置为 frame 对象的底部
	}

	m.subscription = subscription  // 将传入的 subscription 参数赋值给 UI 结构体的 subscription 字段
	m.program = tea.NewProgram(m, tea.WithOutput(os.Stderr), tea.WithInput(os.Stdin))  // 创建一个新的程序并赋值给 UI 结构体的 program 字段
	m.running.Add(1)  // 向 WaitGroup 中添加一个计数

	go func() {
		defer m.running.Done()  // 在函数结束时减少 WaitGroup 的计数
		if _, err := m.program.Run(); err != nil {  // 运行程序并检查是否有错误发生
			log.Errorf("unable to start UI: %+v", err)  // 如果有错误发生，则记录错误消息
// 退出程序并抛出中断异常
bus.ExitWithInterrupt()
// 返回空值
return nil
}

// 处理事件
func (m *UI) Handle(e partybus.Event) error {
	// 如果程序不为空，发送事件
	if m.program != nil {
		m.program.Send(e)
	}
	// 返回空值
	return nil
}

// 关闭UI
func (m *UI) Teardown(force bool) error {
	// 如果不是强制关闭，等待处理程序完成
	if !force {
		m.handler.Wait()
		m.program.Quit()
		// 通常情况下我们希望等待UI完成。然而还有一些未考虑的错误情况，可能导致程序挂起。暂时我们只等待UI完成
// 只处理正常情况。将通过工作人员报告的错误字符串向用户指示问题
// 等待运行完成
m.running.Wait()
} else {
	_ = runWithTimeout(250*time.Millisecond, func() error {
		m.handler.Wait()
		return nil
	})

	// 可能会诱人地使用Kill()，但已经发现这可能导致终端处于
	// 不良状态（在该终端中未来进程无法使用Ctrl+C和其他控制字符）。
	m.program.Quit()

	_ = runWithTimeout(250*time.Millisecond, func() error {
		m.running.Wait()
		return nil
	})
}

// TODO: 允许将完整的日志输出写入屏幕（目前只显示部分日志）
// 返回一个新的PostUIEventWriter，将事件写入os.Stdout和os.Stderr，并根据最终事件改变状态
return newPostUIEventWriter(os.Stdout, os.Stderr).write(m.quiet, m.finalizeEvents...)

// 初始化UI模型，返回一个命令
func (m UI) Init() tea.Cmd {
    return m.frame.Init()
}

// 返回UI模型响应的事件类型列表
func (m UI) RespondsTo() []partybus.EventType {
    return append([]partybus.EventType{
        event.CLIReport,
        event.CLINotification,
        event.CLIAppUpdateAvailable,
    }, m.handler.RespondsTo()...)
}

// 更新UI模型，返回更新后的模型和命令
func (m *UI) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
	// 使用指针接收器以便在 Teardown 中引用相同的 UI 实例（以保持 finalize 事件）
	var cmds []tea.Cmd

	// 允许非 partybus UI 更新（例如窗口大小事件）。注意：这些更新不能影响现有的模型，这是 UI 对象上的框架对象的责任。
	// 处理程序是模型的工厂，框架负责模型的生命周期。此更新允许在创建这些模型时注入世界的初始状态。
	m.handler.OnMessage(msg)

	switch msg := msg.(type) {
	case tea.KeyMsg:
		switch msg.String() {
		// 今天我们将 esc 和 ctrl+c 视为相同，但在将来，当 worker 通过上下文具有优雅的方式来取消正在进行的工作时，
		// 我们可以将 esc 与 bus.Exit() 绑定到这个路径
		case "esc", "ctrl+c":
			bus.ExitWithInterrupt()
			return m, tea.Quit
		}
# 根据消息类型处理不同的事件
case partybus.Event:
    # 记录日志，跟踪事件类型
    log.WithFields("component", "ui").Tracef("event: %q", msg.Type)

    # 根据消息类型进行不同的处理
    switch msg.Type:
        # 对于 CLIReport、CLINotification、CLIAppUpdateAvailable 事件，将其保存在 finalizeEvents 中以便在 UI 终止时显示在屏幕上（或执行其他事件）
        case event.CLIReport, event.CLINotification, event.CLIAppUpdateAvailable:
            m.finalizeEvents = append(m.finalizeEvents, msg)
            # 为什么在这里不返回 tea.Quit 以退出事件？因为可能还有 UI 组件需要更新-渲染循环。因此，我们将让事件循环调用 Teardown()，该函数将显式等待这些组件。
            return m, nil

    # 处理消息并获取新的模型
    newModels, _ := m.handler.Handle(msg)
    # 遍历新的模型
    for _, newModel := range newModels:
        # 如果新模型为空，则继续下一个循环
        if newModel == nil:
            continue
        # 将新模型的初始化命令添加到命令列表中
        cmds = append(cmds, newModel.Init())
        # 将新模型添加到帧中
        m.frame.(*frame.Frame).AppendModel(newModel)
		}
		// 故意继续执行以更新帧模型
	}

	// 使用消息更新帧模型，并返回更新后的帧模型和命令
	frameModel, cmd := m.frame.Update(msg)
	m.frame = frameModel
	cmds = append(cmds, cmd)

	// 返回更新后的 UI 对象和命令
	return m, tea.Batch(cmds...)
}

// 返回 UI 对象的视图
func (m UI) View() string {
	return m.frame.View()
}

// 使用超时时间运行函数，返回错误
func runWithTimeout(timeout time.Duration, fn func() error) (err error) {
	// 创建一个带缓冲的通道
	c := make(chan struct{}, 1)
	// 启动一个 goroutine 执行函数，并将结果发送到通道
	go func() {
		err = fn()
		c <- struct{}{}
	}()
	// 匿名函数调用，可能是用于清理资源或执行其他操作
	select {
	case <-c:
	// 从通道 c 中接收数据
	case <-time.After(timeout):
	// 在超时时间之后向通道发送数据
		return fmt.Errorf("timed out after %v", timeout)
	// 返回超时错误
	}
	// 返回错误
	return err
}
```