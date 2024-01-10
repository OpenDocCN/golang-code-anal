# `grype\cmd\grype\internal\ui\post_ui_event_writer_test.go`

```
package ui

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/gkampitakis/go-snaps/snaps"  // 导入外部包 snaps
    "github.com/stretchr/testify/require"  // 导入外部包 require，用于断言测试结果
    "github.com/wagoodman/go-partybus"  // 导入外部包 go-partybus

    "github.com/anchore/grype/grype/event"  // 导入外部包 event
    "github.com/anchore/grype/grype/event/parsers"  // 导入外部包 parsers
)

func Test_postUIEventWriter_write(t *testing.T) {  // 定义测试函数 Test_postUIEventWriter_write

    tests := []struct {  // 定义测试用例切片
        name    string  // 测试用例名称
        quiet   bool  // 是否安静模式
        events  []partybus.Event  // 事件切片
        wantErr require.ErrorAssertionFunc  // 期望的错误断言函数
    # 第一个对象，没有事件
    {
        name: "no events",
    },
    # 第二个对象，包含多个事件
    {
        name: "all events",
        # 事件列表
        events: []partybus.Event{
            # 第一个事件
            {
                Type:  event.CLINotification,
                Value: "\n\n<my notification 1!!\n...still notifying>\n\n",
            },
            # 第二个事件
            {
                Type:  event.CLINotification,
                Value: "<notification 2>",
            },
            # 第三个事件
            {
                Type: event.CLIAppUpdateAvailable,
                Value: parsers.UpdateCheck{
                    New:     "v0.33.0",
                    Current: "[not provided]",
                },
            },
            # 第四个事件
            {
                Type:  event.CLINotification,
                Value: "<notification 3>",
            },
            # 第五个事件
            {
                Type:  event.CLIReport,
                Value: "\n\n<my --\n-\n-\nreport 1!!>\n\n",
            },
            # 第六个事件
            {
                Type:  event.CLIReport,
                Value: "<report 2>",
            },
        },
    },
    # 第三个对象，只显示报告，且为安静模式
    {
        name:  "quiet only shows report",
        quiet: true,
        # 事件列表
        events: []partybus.Event{
            # 第一个事件
            {
                Type:  event.CLINotification,
                Value: "<notification 1>",
            },
            # 第二个事件
            {
                Type: event.CLIAppUpdateAvailable,
                Value: parsers.UpdateCheck{
                    New:     "<new version>",
                    Current: "<current version>",
                },
            },
            # 第三个事件
            {
                Type:  event.CLIReport,
                Value: "<report 1>",
            },
        },
    }
    # 遍历测试用例集合
    for _, tt := range tests {
        # 使用测试用例的名称创建子测试，并运行
        t.Run(tt.name, func(t *testing.T) {
            # 如果期望的错误为空，则将其设置为 require.NoError
            if tt.wantErr == nil {
                tt.wantErr = require.NoError
            }

            # 创建一个字节缓冲区用于存储标准输出
            stdout := &bytes.Buffer{}
            # 创建一个字节缓冲区用于存储标准错误输出
            stderr := &bytes.Buffer{}
            # 创建一个新的事件写入器
            w := newPostUIEventWriter(stdout, stderr)

            # 使用测试用例的期望错误和事件调用 write 方法，并进行断言
            tt.wantErr(t, w.write(tt.quiet, tt.events...))

            # 创建子测试用例，检查标准输出是否符合预期
            t.Run("stdout", func(t *testing.T) {
                snaps.MatchSnapshot(t, stdout.String())
            })

            # 创建子测试用例，检查标准错误输出是否符合预期
            t.Run("stderr", func(t *testing.T) {
                snaps.MatchSnapshot(t, stderr.String())
            })
        })
    }
# 闭合前面的函数定义
```