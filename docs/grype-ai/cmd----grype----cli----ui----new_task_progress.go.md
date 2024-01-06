# `grype\cmd\grype\cli\ui\new_task_progress.go`

```
package ui
# 导入 taskprogress 包

import "github.com/anchore/bubbly/bubbles/taskprogress"
# 导入 taskprogress 包中的模块

func (m Handler) newTaskProgress(title taskprogress.Title, opts ...taskprogress.Option) taskprogress.Model {
# 定义一个新的任务进度模型，接受标题和选项作为参数

	tsk := taskprogress.New(m.Running, opts...)
# 使用 taskprogress 包中的 New 函数创建一个任务进度模型，并传入运行状态和选项参数

	tsk.HideProgressOnSuccess = true
# 当任务成功时隐藏进度条

	tsk.HideStageOnSuccess = true
# 当任务成功时隐藏阶段信息

	tsk.WindowSize = m.WindowSize
# 设置任务进度模型的窗口大小为当前窗口大小

	tsk.TitleWidth = m.Config.TitleWidth
# 设置任务进度模型的标题宽度为配置文件中的标题宽度

	tsk.TitleOptions = title
# 设置任务进度模型的标题选项为传入的标题选项

	if m.Config.AdjustDefaultTask != nil {
		tsk = m.Config.AdjustDefaultTask(tsk)
	}
# 如果配置文件中有自定义的默认任务调整函数，则调用该函数对任务进度模型进行调整

	return tsk
# 返回创建的任务进度模型
}
```