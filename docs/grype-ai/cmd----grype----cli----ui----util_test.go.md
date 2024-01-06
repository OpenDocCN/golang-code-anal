# `grype\cmd\grype\cli\ui\util_test.go`

```
package ui

import (
	"reflect" // 导入 reflect 包，用于在运行时操作对象
	"sync" // 导入 sync 包，提供同步原语
	"testing" // 导入 testing 包，用于编写测试函数
	"unsafe" // 导入 unsafe 包，提供对底层数据结构的访问

	tea "github.com/charmbracelet/bubbletea" // 导入第三方包，用于构建终端用户界面
)

func runModel(t testing.TB, m tea.Model, iterations int, message tea.Msg, wgs ...*sync.WaitGroup) string {
	t.Helper() // 标记当前测试函数为辅助函数
	if iterations == 0 { // 如果迭代次数为0
		iterations = 1 // 将迭代次数设置为1
	}
	m.Init() // 初始化模型
	var cmd tea.Cmd = func() tea.Msg { // 定义一个命令，返回消息
		return message // 返回指定的消息
	}
# 遍历 wgs 切片中的元素，wg 为每个元素的值
for _, wg := range wgs:
    # 如果 wg 不为空，则等待其完成
    if wg != nil:
        wg.Wait()

# 使用循环执行 cmd 函数，最多执行 iterations 次
for i := 0; cmd != nil && i < iterations; i++:
    # 调用 cmd 函数并将返回的消息扁平化
    msgs := flatten(cmd())
    var nextCmds []tea.Cmd
    var next tea.Cmd
    # 遍历消息切片
    for _, msg := range msgs:
        # 打印消息的类型和内容
        t.Logf("Message: %+v %+v\n", reflect.TypeOf(msg), msg)
        # 调用 m.Update 方法处理消息，并获取下一个命令
        m, next = m.Update(msg)
        # 将下一个命令添加到 nextCmds 切片中
        nextCmds = append(nextCmds, next)
    # 将所有下一个命令合并成一个批处理命令
    cmd = tea.Batch(nextCmds...)
# 返回 m 的视图
return m.View()
# 将嵌套的消息扁平化，返回一个消息数组
func flatten(p tea.Msg) (msgs []tea.Msg) {
    # 如果消息类型是 batchMsg，则提取其中的部分消息并递归调用 flatten 函数
    if reflect.TypeOf(p).Name() == "batchMsg" {
        partials := extractBatchMessages(p)
        for _, m := range partials {
            msgs = append(msgs, flatten(m)...)
        }
    } else {
        # 如果消息类型不是 batchMsg，则将其作为单独的消息添加到消息数组中
        msgs = []tea.Msg{p}
    }
    return msgs
}

# 从消息中提取批量消息
func extractBatchMessages(m tea.Msg) (ret []tea.Msg) {
    # 获取消息的类型
    sliceMsgType := reflect.SliceOf(reflect.TypeOf(tea.Cmd(nil)))
    # 获取消息的值
    value := reflect.ValueOf(m) # 注意：这在技术上是不可寻址的

    # 创建一个可寻址的消息值的副本
    valueCopy := reflect.New(value.Type()).Elem()
    valueCopy.Set(value)
# 使用反射创建一个新的切片对象，指向valueCopy的地址，并获取其元素
cmds := reflect.NewAt(sliceMsgType, unsafe.Pointer(valueCopy.UnsafeAddr())).Elem()

# 遍历切片对象的每个元素
for i := 0; i < cmds.Len(); i++ {
    # 获取切片中的元素
    item := cmds.Index(i)
    # 调用元素的方法，返回结果存储在r中
    r := item.Call(nil)
    # 将结果转换为tea.Msg类型，并添加到ret切片中
    ret = append(ret, r[0].Interface().(tea.Msg))
}
# 返回结果切片
return ret
```