# `grype\cmd\grype\internal\ui\post_ui_event_writer.go`

```go
package ui

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "io" // 导入 io 包，用于进行输入输出操作
    "strings" // 导入 strings 包，用于处理字符串

    "github.com/charmbracelet/lipgloss" // 导入 lipgloss 包，用于创建漂亮的终端输出
    "github.com/hashicorp/go-multierror" // 导入 go-multierror 包，用于处理多个错误
    "github.com/wagoodman/go-partybus" // 导入 go-partybus 包，用于处理事件总线

    "github.com/anchore/grype/grype/event" // 导入 grype 事件包
    "github.com/anchore/grype/grype/event/parsers" // 导入 grype 事件解析包
    "github.com/anchore/grype/internal/log" // 导入 grype 内部日志包
)

type postUIEventWriter struct {
    handles []postUIHandle // 定义 postUIHandle 切片
}

type postUIHandle struct {
    respectQuiet bool // 是否尊重安静模式的标志
    event        partybus.EventType // 事件类型
    writer       io.Writer // 写入器
    dispatch     eventWriter // 事件写入器
}

type eventWriter func(io.Writer, ...partybus.Event) error // 定义事件写入器函数类型

func newPostUIEventWriter(stdout, stderr io.Writer) *postUIEventWriter {
    return &postUIEventWriter{
        handles: []postUIHandle{ // 初始化 handles 切片
            {
                event:        event.CLIReport, // 设置事件类型为 CLIReport
                respectQuiet: false, // 不尊重安静模式
                writer:       stdout, // 设置写入器为标准输出
                dispatch:     writeReports, // 设置事件写入器为 writeReports 函数
            },
            {
                event:        event.CLINotification, // 设置事件类型为 CLINotification
                respectQuiet: true, // 尊重安静模式
                writer:       stderr, // 设置写入器为标准错误输出
                dispatch:     writeNotifications, // 设置事件写入器为 writeNotifications 函数
            },
            {
                event:        event.CLIAppUpdateAvailable, // 设置事件类型为 CLIAppUpdateAvailable
                respectQuiet: true, // 尊重安静模式
                writer:       stderr, // 设置写入器为标准错误输出
                dispatch:     writeAppUpdate, // 设置事件写入器为 writeAppUpdate 函数
            },
        },
    }
}

func (w postUIEventWriter) write(quiet bool, events ...partybus.Event) error {
    var errs error // 定义错误变量
    for _, h := range w.handles { // 遍历 handles 切片
        if quiet && h.respectQuiet { // 如果安静模式并且需要尊重安静模式
            continue // 跳过当前循环
        }

        for _, e := range events { // 遍历事件切片
            if e.Type != h.event { // 如果事件类型不匹配当前处理器的事件类型
                continue // 跳过当前循环
            }

            if err := h.dispatch(h.writer, e); err != nil { // 调用事件写入器函数
                errs = multierror.Append(errs, err) // 将错误追加到 errs 变量
            }
        }
    }
    return errs // 返回错误
}

func writeReports(writer io.Writer, events ...partybus.Event) error {
    var reports []string // 定义报告字符串切片
    // 遍历 events 列表中的每个元素
    for _, e := range events {
        // 使用 parsers 包中的 ParseCLIReport 函数解析事件 e，返回解析结果和错误
        _, report, err := parsers.ParseCLIReport(e)
        // 如果有错误发生
        if err != nil {
            // 记录错误信息并继续下一个事件的处理
            log.WithFields("error", err).Warn("failed to gather final report")
            continue
        }

        // 去除报告末尾的所有空白字符
        reports = append(reports, strings.TrimRight(report, "\n ")+"\n")
    }

    // 防止报告末尾出现双换行
    report := strings.Join(reports, "\n")

    // 将最终报告写入到 writer 中
    if _, err := fmt.Fprint(writer, report); err != nil {
        // 如果写入过程中出现错误，返回错误信息
        return fmt.Errorf("failed to write final report to stdout: %w", err)
    }
    // 写入成功，返回 nil
    return nil
func writeNotifications(writer io.Writer, events ...partybus.Event) error {
    // 创建高强度品红色（ANSI 16位代码）的样式
    style := lipgloss.NewStyle().Foreground(lipgloss.Color("13"))

    // 遍历事件列表
    for _, e := range events {
        // 解析 CLI 通知
        _, notification, err := parsers.ParseCLINotification(e)
        // 如果解析出错，则记录警告并继续下一个事件
        if err != nil {
            log.WithFields("error", err).Warn("failed to parse notification")
            continue
        }

        // 将通知内容渲染成样式并写入到输出流中
        if _, err := fmt.Fprintln(writer, style.Render(notification)); err != nil {
            // 如果写入出错，则记录警告
            log.WithFields("error", err).Warn("failed to write final notifications")
        }
    }
    return nil
}

func writeAppUpdate(writer io.Writer, events ...partybus.Event) error {
    // 创建高强度品红色（ANSI 16位代码）并斜体的样式
    style := lipgloss.NewStyle().Foreground(lipgloss.Color("13")).Italic(true)

    // 遍历事件列表
    for _, e := range events {
        // 解析 CLI 应用更新通知
        version, err := parsers.ParseCLIAppUpdateAvailable(e)
        // 如果解析出错，则记录警告并继续下一个事件
        if err != nil {
            log.WithFields("error", err).Warn("failed to parse app update notification")
            continue
        }

        // 如果新版本为空，则继续下一个事件
        if version.New == "" {
            continue
        }

        // 格式化应用更新通知内容
        notice := fmt.Sprintf("A newer version of grype is available for download: %s (installed version is %s)", version.New, version.Current)

        // 将通知内容渲染成样式并写入到输出流中
        if _, err := fmt.Fprintln(writer, style.Render(notice)); err != nil {
            // 如果写入出错，则记录警告
            log.WithFields("error", err).Warn("failed to write app update notification")
        }
    }
    return nil
}
```