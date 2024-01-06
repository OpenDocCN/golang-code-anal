# `grype\cmd\grype\cli\ui\handle_database_diff_started_test.go`

```
package ui

import (
	"testing" // 导入测试包
	"time" // 导入时间包

	tea "github.com/charmbracelet/bubbletea" // 导入 bubbletea 包
	"github.com/gkampitakis/go-snaps/snaps" // 导入 go-snaps 包
	"github.com/stretchr/testify/require" // 导入 testify 包
	"github.com/wagoodman/go-partybus" // 导入 go-partybus 包
	"github.com/wagoodman/go-progress" // 导入 go-progress 包

	"github.com/anchore/bubbly/bubbles/taskprogress" // 导入 taskprogress 包
	"github.com/anchore/grype/grype/event" // 导入 event 包
	"github.com/anchore/grype/grype/event/monitor" // 导入 monitor 包
)

func TestHandler_handleDatabaseDiffStarted(t *testing.T) {
	// 定义测试函数
	tests := []struct {
```
		name       string  // 定义一个字符串类型的变量name
		eventFn    func(*testing.T) partybus.Event  // 定义一个函数类型的变量eventFn，该函数接受*testing.T类型的参数并返回partybus.Event类型
		iterations int  // 定义一个整型变量iterations
	}{
		{
			name: "DB diff started",  // 设置name变量的值为"DB diff started"
			eventFn: func(t *testing.T) partybus.Event {  // 定义一个匿名函数，该函数接受*testing.T类型的参数并返回partybus.Event类型
				prog := &progress.Manual{}  // 创建一个progress.Manual类型的指针变量prog
				prog.SetTotal(100)  // 设置prog的总数为100
				prog.Set(50)  // 设置prog的当前进度为50

				diffs := &progress.Manual{}  // 创建一个progress.Manual类型的指针变量diffs
				diffs.Set(20)  // 设置diffs的进度为20

				mon := monitor.DBDiff{  // 创建一个monitor.DBDiff类型的变量mon
					Stager:                &progress.Stage{Current: "current"},  // 设置mon的Stager字段为一个progress.Stage类型的指针变量，该指针变量的Current字段值为"current"
					StageProgress:         prog,  // 设置mon的StageProgress字段为prog
					DifferencesDiscovered: diffs,  // 设置mon的DifferencesDiscovered字段为diffs
				}
# 返回一个事件对象，表示数据库差异比较开始
return partybus.Event{
    Type:  event.DatabaseDiffingStarted,  # 事件类型为数据库差异比较开始
    Value: mon,  # 事件值为mon
}

# 返回一个事件对象，表示数据库差异比较完成
eventFn: func(t *testing.T) partybus.Event {
    # 创建一个手动进度对象，设置总数为100，当前进度为100，标记为已完成
    prog := &progress.Manual{}
    prog.SetTotal(100)
    prog.Set(100)
    prog.SetCompleted()

    # 创建一个手动进度对象，设置当前进度为20
    diffs := &progress.Manual{}
    diffs.Set(20)

    # 创建一个数据库差异监控对象，设置当前阶段为"current"，阶段进度为prog
    mon := monitor.DBDiff{
        Stager:                &progress.Stage{Current: "current"},
        StageProgress:         prog,
# 创建一个包含差异信息的事件对象
DifferencesDiscovered: diffs,

# 返回一个包含数据库差异开始事件类型和差异信息的事件对象
return partybus.Event{
    Type:  event.DatabaseDiffingStarted,
    Value: mon,
}

# 遍历测试用例，对每个测试用例运行测试函数
for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        # 调用测试用例的事件函数，获取事件对象
        e := tt.eventFn(t)
        # 创建一个新的事件处理器
        handler := New(DefaultHandlerConfig())
        # 设置处理器的窗口大小
        handler.WindowSize = tea.WindowSizeMsg{
            Width:  100,
            Height: 80,
        }
        
        # 处理事件并获取处理结果
        models, _ := handler.Handle(e)
# 确保 models 数组的长度为 1
require.Len(t, models, 1)
# 获取 models 数组的第一个元素
model := models[0]

# 尝试将 model 转换为 taskprogress.Model 类型，返回转换结果和是否成功的标志
tsk, ok := model.(taskprogress.Model)
# 确保转换成功
require.True(t, ok)

# 调用 runModel 函数执行任务模型，获取结果
got := runModel(t, tsk, tt.iterations, taskprogress.TickMsg{
    Time:     time.Now(),
    Sequence: tsk.Sequence(),
    ID:       tsk.ID(),
})
# 记录结果日志
t.Log(got)
# 使用 snaps 包的 MatchSnapshot 函数对结果进行快照测试
snaps.MatchSnapshot(t, got)
```