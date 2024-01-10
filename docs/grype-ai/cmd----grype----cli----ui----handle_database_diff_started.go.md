# `grype\cmd\grype\cli\ui\handle_database_diff_started.go`

```
package ui

import (
    "fmt"

    tea "github.com/charmbracelet/bubbletea"
    "github.com/wagoodman/go-partybus"
    "github.com/wagoodman/go-progress"

    "github.com/anchore/bubbly/bubbles/taskprogress"
    "github.com/anchore/grype/grype/event/monitor"
    "github.com/anchore/grype/grype/event/parsers"
    "github.com/anchore/grype/internal/log"
)

// 定义数据库差异进度展示器结构体
type dbDiffProgressStager struct {
    monitor *monitor.DBDiff
}

// 实现 Stage 方法，返回当前阶段的描述
func (p dbDiffProgressStager) Stage() string {
    if progress.IsErrCompleted(p.monitor.StageProgress.Error()) {
        return fmt.Sprintf("%d differences found", p.monitor.DifferencesDiscovered.Current())
    }
    return p.monitor.Stager.Stage()
}

// 实现 Current 方法，返回当前进度
func (p dbDiffProgressStager) Current() int64 {
    return p.monitor.StageProgress.Current()
}

// 实现 Error 方法，返回错误信息
func (p dbDiffProgressStager) Error() error {
    return p.monitor.StageProgress.Error()
}

// 实现 Size 方法，返回总大小
func (p dbDiffProgressStager) Size() int64 {
    return p.monitor.StageProgress.Size()
}

// 处理数据库差异开始事件
func (m *Handler) handleDatabaseDiffStarted(e partybus.Event) ([]tea.Model, tea.Cmd) {
    // 解析数据库差异开始事件
    mon, err := parsers.ParseDatabaseDiffingStarted(e)
    if err != nil {
        log.WithFields("error", err).Warn("unable to parse event")
        return nil, nil
    }

    // 创建新的任务进度
    tsk := m.newTaskProgress(
        taskprogress.Title{
            Default: "Compare Vulnerability DBs",
            Running: "Comparing Vulnerability DBs",
            Success: "Compared Vulnerability DBs",
        },
        taskprogress.WithStagedProgressable(dbDiffProgressStager{monitor: mon}),
    )

    tsk.HideStageOnSuccess = false

    return []tea.Model{tsk}, nil
}
```