# `grype\cmd\grype\internal\ui\post_ui_event_writer.go`

```
// 声明了一个名为 ui 的包
package ui

// 导入了 fmt、io、strings 等标准库，以及一些第三方库
import (
	"fmt"
	"io"
	"strings"

	"github.com/charmbracelet/lipgloss"  // 导入了 charmbracelet/lipgloss 库
	"github.com/hashicorp/go-multierror"  // 导入了 hashicorp/go-multierror 库
	"github.com/wagoodman/go-partybus"   // 导入了 wagoodman/go-partybus 库

	"github.com/anchore/grype/grype/event"  // 导入了 anchore/grype/grype/event 库
	"github.com/anchore/grype/grype/event/parsers"  // 导入了 anchore/grype/grype/event/parsers 库
	"github.com/anchore/grype/internal/log"  // 导入了 anchore/grype/internal/log 库
)

// 声明了一个名为 postUIEventWriter 的结构体类型
type postUIEventWriter struct {
	handles []postUIHandle  // 结构体包含了一个名为 handles 的切片
}
# 定义一个结构体，用于处理UI事件
type postUIHandle struct {
    respectQuiet bool          # 是否忽略静默模式
    event        partybus.EventType  # 事件类型
    writer       io.Writer      # 写入数据的接口
    dispatch     eventWriter    # 事件写入函数
}

# 定义一个函数类型，用于处理事件写入
type eventWriter func(io.Writer, ...partybus.Event) error

# 创建一个新的UI事件写入对象
func newPostUIEventWriter(stdout, stderr io.Writer) *postUIEventWriter {
    return &postUIEventWriter{
        handles: []postUIHandle{  # 初始化UI事件处理器列表
            {
                event:        event.CLIReport,  # 设置事件类型为CLI报告
                respectQuiet: false,            # 设置不忽略静默模式
                writer:       stdout,            # 设置写入数据的接口为标准输出
                dispatch:     writeReports,      # 设置事件写入函数为writeReports
            },
            {
                event:        event.CLINotification,  # 设置事件类型为CLI通知
// 创建一个postUIEventWriter结构体的方法，用于写入UI事件
func newPostUIEventWriter(stderr io.Writer) postUIEventWriter {
	// 返回一个postUIEventWriter结构体，包含了写入UI事件所需的参数
	return postUIEventWriter{
		handles: []eventHandle{
			{
				event:        event.CLIAppError,
				respectQuiet: true, // 是否尊重安静模式
				writer:       stderr, // 写入的目标
				dispatch:     writeErrors, // 调度写入错误
			},
			{
				event:        event.CLIAppUpdateAvailable,
				respectQuiet: true, // 是否尊重安静模式
				writer:       stderr, // 写入的目标
				dispatch:     writeAppUpdate, // 调度写入应用更新
			},
		},
	}
}

// 写入UI事件的方法
func (w postUIEventWriter) write(quiet bool, events ...partybus.Event) error {
	var errs error
	// 遍历事件处理器
	for _, h := range w.handles {
		// 如果是安静模式并且需要尊重安静模式，则跳过
		if quiet && h.respectQuiet {
			continue
		}
# 遍历事件列表
for _, e := range events {
    # 如果事件类型不符合要求，则跳过当前循环
    if e.Type != h.event {
        continue
    }
    # 调用 dispatch 方法处理事件，将结果保存到 errs 中
    if err := h.dispatch(h.writer, e); err != nil {
        errs = multierror.Append(errs, err)
    }
}
# 返回处理过程中出现的错误
return errs
}

# 写入报告的方法
func writeReports(writer io.Writer, events ...partybus.Event) error {
    # 定义报告列表
    var reports []string
    # 遍历事件列表
    for _, e := range events {
        # 调用 ParseCLIReport 方法解析事件，获取报告和错误信息
        _, report, err := parsers.ParseCLIReport(e)
        # 如果解析出错，则记录警告日志
        if err != nil {
            log.WithFields("error", err).Warn("failed to gather final report")
		// 如果报告为空，则跳过
		if report == "" {
			continue
		}

		// 去除报告末尾的所有空白字符
		reports = append(reports, strings.TrimRight(report, "\n ")+"\n")
	}

	// 防止报告末尾出现双换行
	report := strings.Join(reports, "\n")

	// 将最终报告写入到输出流中
	if _, err := fmt.Fprint(writer, report); err != nil {
		return fmt.Errorf("failed to write final report to stdout: %w", err)
	}
	// 返回空值
	return nil
}

func writeNotifications(writer io.Writer, events ...partybus.Event) error {
	// 13 = 高强度品红色（ANSI 16位代码）
	style := lipgloss.NewStyle().Foreground(lipgloss.Color("13"))
		// 遍历事件列表
		_, notification, err := parsers.ParseCLINotification(e)
		// 解析事件的通知内容
		if err != nil {
			// 如果解析出错，记录错误并继续下一个事件
			log.WithFields("error", err).Warn("failed to parse notification")
			continue
		}

		if _, err := fmt.Fprintln(writer, style.Render(notification)); err != nil {
			// 如果写入通知内容出错，记录错误但不中断程序
			log.WithFields("error", err).Warn("failed to write final notifications")
		}
	}
	// 返回空错误
	return nil
}

func writeAppUpdate(writer io.Writer, events ...partybus.Event) error {
	// 13 = high intensity magenta (ANSI 16 bit code) + italics
	// 创建样式对象，设置颜色和斜体
	style := lipgloss.NewStyle().Foreground(lipgloss.Color("13")).Italic(true)

	for _, e := range events {
		// 遍历事件列表
// 解析 CLI 应用程序更新的可用版本
version, err := parsers.ParseCLIAppUpdateAvailable(e)
// 如果解析出错，记录错误并继续循环
if err != nil {
    log.WithFields("error", err).Warn("failed to parse app update notification")
    continue
}

// 如果新版本为空，继续循环
if version.New == "" {
    continue
}

// 格式化通知消息，包含新版本和当前版本信息
notice := fmt.Sprintf("A newer version of grype is available for download: %s (installed version is %s)", version.New, version.Current)

// 将通知消息写入输出流，如果出错，记录错误但不中断程序
if _, err := fmt.Fprintln(writer, style.Render(notice)); err != nil {
    log.WithFields("error", err).Warn("failed to write app update notification")
}
// 返回空值
return nil
```