# `grype\cmd\grype\cli\ui\util_test.go`

```go
package ui

import (
    "reflect"  // 导入 reflect 包，用于获取变量的类型信息
    "sync"     // 导入 sync 包，用于实现并发控制
    "testing"  // 导入 testing 包，用于编写测试函数
    "unsafe"   // 导入 unsafe 包，用于进行底层操作

    tea "github.com/charmbracelet/bubbletea"  // 导入第三方包，用于构建终端用户界面
)

func runModel(t testing.TB, m tea.Model, iterations int, message tea.Msg, wgs ...*sync.WaitGroup) string {
    t.Helper()  // 标记当前测试函数是辅助函数
    if iterations == 0 {  // 如果迭代次数为0
        iterations = 1  // 将迭代次数设置为1
    }
    m.Init()  // 初始化模型
    var cmd tea.Cmd = func() tea.Msg {  // 定义一个命令函数
        return message  // 返回指定的消息
    }

    for _, wg := range wgs {  // 遍历传入的 WaitGroup 切片
        if wg != nil {  // 如果 WaitGroup 不为nil
            wg.Wait()  // 等待 WaitGroup 完成
        }
    }

    for i := 0; cmd != nil && i < iterations; i++ {  // 循环执行命令，直到命令为空或达到指定迭代次数
        msgs := flatten(cmd())  // 扁平化处理命令返回的消息
        var nextCmds []tea.Cmd  // 定义下一个命令的切片
        var next tea.Cmd  // 定义下一个命令
        for _, msg := range msgs {  // 遍历消息切片
            t.Logf("Message: %+v %+v\n", reflect.TypeOf(msg), msg)  // 记录消息的类型和内容
            m, next = m.Update(msg)  // 更新模型并获取下一个命令
            nextCmds = append(nextCmds, next)  // 将下一个命令添加到切片中
        }
        cmd = tea.Batch(nextCmds...)  // 将下一个命令切片合并成一个命令
    }
    return m.View()  // 返回模型的视图
}

func flatten(p tea.Msg) (msgs []tea.Msg) {
    if reflect.TypeOf(p).Name() == "batchMsg" {  // 如果消息类型为 batchMsg
        partials := extractBatchMessages(p)  // 提取批量消息
        for _, m := range partials {  // 遍历批量消息
            msgs = append(msgs, flatten(m)...)  // 扁平化处理批量消息并添加到消息切片中
        }
    } else {
        msgs = []tea.Msg{p}  // 否则将消息添加到消息切片中
    }
    return msgs  // 返回消息切片
}

func extractBatchMessages(m tea.Msg) (ret []tea.Msg) {
    sliceMsgType := reflect.SliceOf(reflect.TypeOf(tea.Cmd(nil)))  // 获取 Cmd 类型的切片类型
    value := reflect.ValueOf(m)  // 获取消息的反射值，注意这是不可寻址的

    // 创建一个可寻址的副本
    valueCopy := reflect.New(value.Type()).Elem()
    valueCopy.Set(value)

    cmds := reflect.NewAt(sliceMsgType, unsafe.Pointer(valueCopy.UnsafeAddr())).Elem()  // 获取消息中的命令切片
    for i := 0; i < cmds.Len(); i++ {  // 遍历命令切片
        item := cmds.Index(i)  // 获取切片中的元素
        r := item.Call(nil)  // 调用命令函数
        ret = append(ret, r[0].Interface().(tea.Msg))  // 将返回的消息添加到结果切片中
    }
    return ret  // 返回结果切片
}
```