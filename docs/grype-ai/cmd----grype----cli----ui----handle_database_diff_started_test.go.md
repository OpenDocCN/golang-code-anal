# `grype\cmd\grype\cli\ui\handle_database_diff_started_test.go`

```go
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
    // 定义测试用例
    tests := []struct {
        name       string // 测试用例名称
        eventFn    func(*testing.T) partybus.Event // 事件处理函数
        iterations int // 迭代次数
    }{
        {
            name: "DB diff started",
            eventFn: func(t *testing.T) partybus.Event {
                // 创建一个手动进度对象，并设置总进度为100，当前进度为50
                prog := &progress.Manual{}
                prog.SetTotal(100)
                prog.Set(50)

                // 创建一个手动进度对象，并设置当前进度为20
                diffs := &progress.Manual{}
                diffs.Set(20)

                // 创建一个数据库差异监视对象，包括当前阶段、阶段进度和发现的差异
                mon := monitor.DBDiff{
                    Stager:                &progress.Stage{Current: "current"},
                    StageProgress:         prog,
                    DifferencesDiscovered: diffs,
                }

                // 返回一个事件对象，类型为数据库差异开始，值为数据库差异监视对象
                return partybus.Event{
                    Type:  event.DatabaseDiffingStarted,
                    Value: mon,
                }
            },
        },
        {
            name: "DB diff complete",
            eventFn: func(t *testing.T) partybus.Event {
                // 创建一个手动进度对象，并设置总进度为100，当前进度为100，标记为已完成
                prog := &progress.Manual{}
                prog.SetTotal(100)
                prog.Set(100)
                prog.SetCompleted()

                // 创建一个手动进度对象，并设置当前进度为20
                diffs := &progress.Manual{}
                diffs.Set(20)

                // 创建一个数据库差异监视对象，包括当前阶段、阶段进度和发现的差异
                mon := monitor.DBDiff{
                    Stager:                &progress.Stage{Current: "current"},
                    StageProgress:         prog,
                    DifferencesDiscovered: diffs,
                }

                // 返回一个事件对象，类型为数据库差异开始，值为数据库差异监视对象
                return partybus.Event{
                    Type:  event.DatabaseDiffingStarted,
                    Value: mon,
                }
            },
        },
    }
    for _, tt := range tests {
        // 遍历测试用例
        t.Run(tt.name, func(t *testing.T) {
            // 运行测试用例
            e := tt.eventFn(t)
            // 获取事件
            handler := New(DefaultHandlerConfig())
            // 创建处理程序
            handler.WindowSize = tea.WindowSizeMsg{
                Width:  100,
                Height: 80,
            }
            // 设置处理程序窗口大小

            models, _ := handler.Handle(e)
            // 处理事件，获取模型
            require.Len(t, models, 1)
            // 断言模型数量为1
            model := models[0]
            // 获取第一个模型

            tsk, ok := model.(taskprogress.Model)
            // 断言模型类型为taskprogress.Model
            require.True(t, ok)
            // 断言结果为true

            got := runModel(t, tsk, tt.iterations, taskprogress.TickMsg{
                Time:     time.Now(),
                Sequence: tsk.Sequence(),
                ID:       tsk.ID(),
            })
            // 运行模型
            t.Log(got)
            // 记录日志
            snaps.MatchSnapshot(t, got)
            // 匹配快照
        })
    }
# 闭合前面的函数定义
```