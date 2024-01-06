# `grype\cmd\grype\cli\ui\handle_database_diff_started.go`

```
package ui

import (
	"fmt"

	tea "github.com/charmbracelet/bubbletea"  // 导入 bubbletea 包，用于构建终端用户界面
	"github.com/wagoodman/go-partybus"  // 导入 go-partybus 包，用于事件总线
	"github.com/wagoodman/go-progress"  // 导入 go-progress 包，用于显示进度条

	"github.com/anchore/bubbly/bubbles/taskprogress"  // 导入 taskprogress 包，用于显示任务进度
	"github.com/anchore/grype/grype/event/monitor"  // 导入 monitor 包，用于监控事件
	"github.com/anchore/grype/grype/event/parsers"  // 导入 parsers 包，用于解析事件
	"github.com/anchore/grype/internal/log"  // 导入 log 包，用于日志记录
)

type dbDiffProgressStager struct {
	monitor *monitor.DBDiff  // 定义一个监控数据库差异的对象
}

func (p dbDiffProgressStager) Stage() string {
// 如果进度监视器的阶段进度出现错误，则返回发现的差异数量
if progress.IsErrCompleted(p.monitor.StageProgress.Error()) {
    return fmt.Sprintf("%d differences found", p.monitor.DifferencesDiscovered.Current())
}
// 返回当前阶段的进度
return p.monitor.Stager.Stage()
}

// 返回当前进度
func (p dbDiffProgressStager) Current() int64 {
    return p.monitor.StageProgress.Current()
}

// 返回错误信息
func (p dbDiffProgressStager) Error() error {
    return p.monitor.StageProgress.Error()
}

// 返回进度大小
func (p dbDiffProgressStager) Size() int64 {
    return p.monitor.StageProgress.Size()
}

// 处理数据库差异开始事件
func (m *Handler) handleDatabaseDiffStarted(e partybus.Event) ([]tea.Model, tea.Cmd) {
    // 解析数据库差异开始事件
    mon, err := parsers.ParseDatabaseDiffingStarted(e)
// 如果发生错误，记录错误信息并返回空值
if err != nil {
    log.WithFields("error", err).Warn("unable to parse event")
    return nil, nil
}

// 创建一个新的任务进度对象，设置标题和运行时的提示信息
tsk := m.newTaskProgress(
    taskprogress.Title{
        Default: "Compare Vulnerability DBs",
        Running: "Comparing Vulnerability DBs",
        Success: "Compared Vulnerability DBs",
    },
    // 设置任务进度对象的阶段性进度
    taskprogress.WithStagedProgressable(dbDiffProgressStager{monitor: mon}),
)

// 设置任务成功时是否隐藏阶段信息
tsk.HideStageOnSuccess = false

// 返回包含任务进度对象的模型切片和空值
return []tea.Model{tsk}, nil
}
```