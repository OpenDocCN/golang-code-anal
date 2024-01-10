# `grype\cmd\grype\cli\ui\new_task_progress.go`

```
package ui

import "github.com/anchore/bubbly/bubbles/taskprogress"

// 创建一个新的任务进度条模型
func (m Handler) newTaskProgress(title taskprogress.Title, opts ...taskprogress.Option) taskprogress.Model {
    // 使用传入的参数创建一个新的任务进度条
    tsk := taskprogress.New(m.Running, opts...)

    // 设置任务成功时隐藏进度条和阶段信息
    tsk.HideProgressOnSuccess = true
    tsk.HideStageOnSuccess = true
    // 设置任务进度条的窗口大小和标题宽度
    tsk.WindowSize = m.WindowSize
    tsk.TitleWidth = m.Config.TitleWidth
    tsk.TitleOptions = title

    // 如果配置中有自定义的任务调整函数，则使用该函数对任务进行调整
    if m.Config.AdjustDefaultTask != nil {
        tsk = m.Config.AdjustDefaultTask(tsk)
    }

    // 返回创建的任务进度条模型
    return tsk
}
```