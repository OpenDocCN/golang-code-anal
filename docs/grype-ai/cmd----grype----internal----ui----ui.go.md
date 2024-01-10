# `grype\cmd\grype\internal\ui\ui.go`

```
package ui

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "os" // 导入 os 包，提供对操作系统功能的访问
    "sync" // 导入 sync 包，提供同步原语的支持
    "time" // 导入 time 包，提供时间的功能

    tea "github.com/charmbracelet/bubbletea" // 导入 bubbletea 包，并将其命名为 tea
    "github.com/wagoodman/go-partybus" // 导入 go-partybus 包

    "github.com/anchore/bubbly" // 导入 bubbly 包
    "github.com/anchore/bubbly/bubbles/frame" // 导入 frame 包
    "github.com/anchore/clio" // 导入 clio 包
    "github.com/anchore/go-logger" // 导入 go-logger 包
    "github.com/anchore/grype/grype/event" // 导入 event 包
    "github.com/anchore/grype/internal/bus" // 导入 bus 包
    "github.com/anchore/grype/internal/log" // 导入 log 包
)

var _ interface {
    tea.Model
    partybus.Responder
    clio.UI
} = (*UI)(nil) // 确保 UI 结构体实现了指定的接口

type UI struct {
    program        *tea.Program // 定义指向 tea.Program 的指针
    running        *sync.WaitGroup // 定义指向 sync.WaitGroup 的指针
    quiet          bool // 定义布尔类型的 quiet 变量
    subscription   partybus.Unsubscribable // 定义 partybus.Unsubscribable 类型的 subscription 变量
    finalizeEvents []partybus.Event // 定义 partybus.Event 类型的切片 finalizeEvents

    handler *bubbly.HandlerCollection // 定义 bubbly.HandlerCollection 类型的 handler
    frame   tea.Model // 定义 tea.Model 类型的 frame
}

func New(quiet bool, handlers ...bubbly.EventHandler) *UI {
    return &UI{
        handler: bubbly.NewHandlerCollection(handlers...), // 使用提供的 handlers 创建一个新的 bubbly.HandlerCollection
        frame:   frame.New(), // 创建一个新的 frame
        running: &sync.WaitGroup{}, // 初始化一个新的 sync.WaitGroup
        quiet:   quiet, // 设置 quiet 变量的值
    }
}

func (m *UI) Setup(subscription partybus.Unsubscribable) error {
    // 我们仍然希望收集日志消息，但是日志记录器不应直接写入屏幕
    if logWrapper, ok := log.Get().(logger.Controller); ok { // 检查是否可以获取日志记录器
        logWrapper.SetOutput(m.frame.(*frame.Frame).Footer()) // 设置日志输出到 frame 的底部
    }

    m.subscription = subscription // 设置 subscription 变量的值
    m.program = tea.NewProgram(m, tea.WithOutput(os.Stderr), tea.WithInput(os.Stdin)) // 创建一个新的 tea.Program
    m.running.Add(1) // 向 running WaitGroup 添加一个计数

    go func() {
        defer m.running.Done() // 在函数退出时减少 running WaitGroup 的计数
        if _, err := m.program.Run(); err != nil { // 运行程序并检查是否有错误
            log.Errorf("unable to start UI: %+v", err) // 输出错误日志
            bus.ExitWithInterrupt() // 通过 bus 退出程序
        }
    }()

    return nil
}

func (m *UI) Handle(e partybus.Event) error {
    if m.program != nil { // 检查程序是否存在
        m.program.Send(e) // 发送事件到程序
    }
    return nil
}

func (m *UI) Teardown(force bool) error {
    // 如果不是强制退出，则等待处理程序完成
    if !force {
        m.handler.Wait()
        m.program.Quit()
        // 通常情况下，我们希望等待 UI 完成。然而，仍然存在未考虑的错误情况，导致程序挂起。目前，我们只会在正常情况下等待 UI 完成。在拆除后，将始终通过报告工作程序的错误字符串向用户指示问题。
        m.running.Wait()
    } else {
        _ = runWithTimeout(250*time.Millisecond, func() error {
            m.handler.Wait()
            return nil
        })

        // 可能会诱人使用 Kill()，但已发现这可能导致终端处于糟糕状态（在该终端中将不再能够使用 Ctrl+C 和其他控制字符来控制未来的进程）。
        m.program.Quit()

        _ = runWithTimeout(250*time.Millisecond, func() error {
            m.running.Wait()
            return nil
        })
    }

    // TODO: 允许将完整日志输出到屏幕（目前只显示部分日志）
    // 这需要协调以知道最后一个帧事件是什么，以相应地更改状态（目前不可能实现）

    // 返回一个新的 PostUIEventWriter 对象，用于将输出写入到标准输出和标准错误流
    return newPostUIEventWriter(os.Stdout, os.Stderr).write(m.quiet, m.finalizeEvents...)
// bubbletea.Model functions

// 初始化 UI 模型，返回一个命令
func (m UI) Init() tea.Cmd {
    return m.frame.Init()
}

// 返回 UI 模型响应的事件类型列表
func (m UI) RespondsTo() []partybus.EventType {
    return append([]partybus.EventType{
        event.CLIReport,
        event.CLINotification,
        event.CLIAppUpdateAvailable,
    }, m.handler.RespondsTo()...)
}

// 更新 UI 模型
func (m *UI) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    // 注意：我们需要一个指针接收器，以便在 Teardown 中引用相同的 UI 实例（以保持最终事件）

    var cmds []tea.Cmd

    // 允许进行非 partybus UI 更新（例如窗口大小事件）。注意：这些更新不能影响现有模型，这是 UI 对象上的框架对象的责任。
    // 处理程序是模型的工厂，框架负责模型的生命周期。此更新允许在创建这些模型时注入世界的初始状态。
    m.handler.OnMessage(msg)

    switch msg := msg.(type) {
    case tea.KeyMsg:
        switch msg.String() {
        // 今天我们将 esc 和 ctrl+c 视为相同，但在将来，当 worker 通过上下文具有优雅的方式来取消正在进行的工作时，我们可以将 esc 与 bus.Exit() 绑定到此路径
        case "esc", "ctrl+c":
            bus.ExitWithInterrupt()
            return m, tea.Quit
        }
    }
}
    # 根据消息类型进行不同的处理
    case partybus.Event:
        # 记录日志，跟踪事件类型
        log.WithFields("component", "ui").Tracef("event: %q", msg.Type)

        # 根据消息类型进行不同的处理
        switch msg.Type:
            # 处理 CLIReport, CLINotification, CLIAppUpdateAvailable 事件
            case event.CLIReport, event.CLINotification, event.CLIAppUpdateAvailable:
                # 将这些事件保存起来，在 UI 终止时显示到屏幕上（或执行其他事件）
                m.finalizeEvents = append(m.finalizeEvents, msg)

                # 为什么在这里返回 tea.Quit 退出事件？因为可能还有 UI 组件需要更新-渲染循环。
                # 因此我们将让事件循环调用 Teardown()，它将显式等待这些组件
                return m, nil

        # 处理消息并获取新的模型和命令
        newModels, _ := m.handler.Handle(msg)
        for _, newModel := range newModels:
            if newModel == nil:
                continue
            # 将新模型的初始化命令添加到命令列表中
            cmds = append(cmds, newModel.Init())
            # 将新模型添加到帧模型中
            m.frame.(*frame.Frame).AppendModel(newModel)
        # 故意继续更新帧模型
    }

    # 更新帧模型并获取命令
    frameModel, cmd := m.frame.Update(msg)
    # 更新帧模型
    m.frame = frameModel
    # 将命令添加到命令列表中
    cmds = append(cmds, cmd)

    # 返回模型和命令的批处理
    return m, tea.Batch(cmds...)
# 定义 UI 结构体的 View 方法，返回字符串类型的视图
func (m UI) View() string {
    # 调用 frame 结构体的 View 方法，并返回结果
    return m.frame.View()
}

# 定义带有超时功能的运行函数，接受超时时长和函数作为参数，返回错误
func runWithTimeout(timeout time.Duration, fn func() error) (err error) {
    # 创建一个带有缓冲区大小为1的通道
    c := make(chan struct{}, 1)
    # 启动一个 goroutine，执行传入的函数，并将结果赋值给 err，然后向通道发送信号
    go func() {
        err = fn()
        c <- struct{}{}
    }()
    # 使用 select 语句监听通道的信号
    select {
    # 如果收到通道的信号，则继续执行
    case <-c:
    # 如果超时，则返回超时错误
    case <-time.After(timeout):
        return fmt.Errorf("timed out after %v", timeout)
    }
    # 返回执行函数的结果
    return err
}
```