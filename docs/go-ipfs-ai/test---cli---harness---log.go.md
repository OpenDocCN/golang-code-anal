# `kubo\test\cli\harness\log.go`

```
package harness

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "path/filepath"  // 导入 filepath 包，用于处理文件路径
    "runtime"  // 导入 runtime 包，用于访问 Go 运行时的信息
    "sort"  // 导入 sort 包，用于排序
    "strings"  // 导入 strings 包，用于处理字符串
    "sync"  // 导入 sync 包，用于实现同步
    "testing"  // 导入 testing 包，用于编写测试
    "time"  // 导入 time 包，用于处理时间
)

type event struct {
    timestamp time.Time  // 定义事件结构体，包含时间戳和消息
    msg       string
}

type events []*event  // 定义事件切片类型

func (e events) Len() int           { return len(e) }  // 实现 Len 方法，返回事件切片的长度
func (e events) Less(i, j int) bool { return e[i].timestamp.Before(e[j].timestamp) }  // 实现 Less 方法，比较两个事件的时间戳
func (e events) Swap(i, j int)      { e[i], e[j] = e[j], e[i] }  // 实现 Swap 方法，交换两个事件的位置

// TestLogger 是测试用的日志记录器
// 它会缓冲输出，只有在测试失败或显式打开输出时才会写入输出
// 这个日志记录器的目的是允许 Go 测试在打开 verbose 标志的情况下运行，而不打印日志
// verbose 标志很有用，因为它会输出测试进度，但打印日志会使输出过于冗长
//
// 您还可以添加前缀，以便在每条日志消息之前添加额外的日志上下文
//
// 这是一个日志记录器的层次结构实现，子记录器会将日志条目刷新回父记录器
// 这是因为 t.Cleanup() 以 LIFO 顺序处理条目，所以子记录器总是先刷新
//
// 显然，这个日志记录器不应该在生产系统中使用
type TestLogger struct {
    parent        *TestLogger  // 父记录器
    children      []*TestLogger  // 子记录器
    prefixes      []string  // 前缀列表
    prefixesIface []any  // 前缀接口列表
    t             *testing.T  // 测试对象
    buf           events  // 事件切片
    m             sync.Mutex  // 互斥锁
    logsEnabled   bool  // 日志是否启用
}

func NewTestLogger(t *testing.T) *TestLogger {
    l := &TestLogger{t: t, buf: make(events, 0)}  // 创建新的测试日志记录器对象
    t.Cleanup(l.flush)  // 在测试结束时刷新日志
    return l
}

func (t *TestLogger) buildPrefix(timestamp time.Time) string {
    d := timestamp.Format("2006-01-02T15:04:05.999999")  // 格式化时间戳
    _, file, lineno, _ := runtime.Caller(2)  // 获取调用者的信息
    file = filepath.Base(file)  // 获取文件名
    caller := fmt.Sprintf("%s:%d", file, lineno)  // 格式化调用者信息

    if len(t.prefixes) == 0 {
        return fmt.Sprintf("%s\t%s\t", d, caller)  // 返回格式化后的前缀
    }

    prefixes := strings.Join(t.prefixes, ":")  // 将前缀列表连接成字符串
    return fmt.Sprintf("%s\t%s\t%s: ", d, caller, prefixes)  // 返回格式化后的前缀
}
func (t *TestLogger) Log(args ...any) {
    // 获取当前时间戳
    timestamp := time.Now()
    // 构建日志前缀并将参数转换为字符串拼接在后面
    e := t.buildPrefix(timestamp) + fmt.Sprint(args...)
    // 将日志事件添加到日志缓冲区
    t.add(&event{timestamp: timestamp, msg: e})
}

func (t *TestLogger) Logf(format string, args ...any) {
    // 获取当前时间戳
    timestamp := time.Now()
    // 使用格式化字符串构建日志消息
    e := t.buildPrefix(timestamp) + fmt.Sprintf(format, args...)
    // 将日志事件添加到日志缓冲区
    t.add(&event{timestamp: timestamp, msg: e})
}

func (t *TestLogger) Fatal(args ...any) {
    // 获取当前时间戳
    timestamp := time.Now()
    // 构建带有 "fatal: " 前缀的日志消息
    e := t.buildPrefix(timestamp) + fmt.Sprint(append([]any{"fatal: "}, args...)...)
    // 将日志事件添加到日志缓冲区
    t.add(&event{timestamp: timestamp, msg: e})
    // 立即终止测试
    t.t.FailNow()
}

func (t *TestLogger) Fatalf(format string, args ...any) {
    // 获取当前时间戳
    timestamp := time.Now()
    // 使用格式化字符串构建带有 "fatal: " 前缀的日志消息
    e := t.buildPrefix(timestamp) + fmt.Sprintf(fmt.Sprintf("fatal: %s", format), args...)
    // 将日志事件添加到日志缓冲区
    t.add(&event{timestamp: timestamp, msg: e})
    // 立即终止测试
    t.t.FailNow()
}

func (t *TestLogger) add(e *event) {
    // 加锁以确保并发安全
    t.m.Lock()
    defer t.m.Unlock()
    // 将事件添加到日志缓冲区
    t.buf = append(t.buf, e)
}

func (t *TestLogger) AddPrefix(prefix string) *TestLogger {
    // 创建一个新的 TestLogger 实例，继承父级的属性并添加新的前缀
    l := &TestLogger{
        prefixes:      append(t.prefixes, prefix),
        prefixesIface: append(t.prefixesIface, prefix),
        t:             t.t,
        parent:        t,
        logsEnabled:   t.logsEnabled,
    }
    // 加锁以确保并发安全
    t.m.Lock()
    defer t.m.Unlock()
    // 将新的子日志记录器添加到父级的子记录器列表中
    t.children = append(t.children, l)
    // 在测试结束时清理子日志记录器的日志缓冲区
    t.t.Cleanup(l.flush)
    // 返回新的子日志记录器
    return l
}

func (t *TestLogger) EnableLogs() {
    // 加锁以确保并发安全
    t.m.Lock()
    defer t.m.Unlock()
    // 启用当前日志记录器的日志记录
    t.logsEnabled = true
    // 如果父级日志记录器也启用了日志记录，则递归启用父级日志记录器
    if t.parent != nil {
        if t.parent.logsEnabled {
            t.parent.EnableLogs()
        }
    }
    // 打印启用日志记录的子记录器数量
    fmt.Printf("enabling %d children\n", len(t.children))
    // 递归启用所有子记录器的日志记录
    for _, c := range t.children {
        if !c.logsEnabled {
            c.EnableLogs()
        }
    }
}

func (t *TestLogger) flush() {
    // 清空日志缓冲区的操作
}
    // 如果测试失败或者日志启用，加锁并在函数结束时解锁
    if t.t.Failed() || t.logsEnabled {
        t.m.Lock()
        defer t.m.Unlock()
        // 如果这是一个子测试，将事件发送给父测试
        // 根父测试将按排序顺序打印所有事件
        if t.parent != nil {
            for _, e := range t.buf {
                t.parent.add(e)
            }
        } else {
            // 如果是根测试，对所有事件进行排序，然后打印它们
            sort.Sort(t.buf)
            fmt.Println()
            fmt.Printf("Logs for test %q:\n\n", t.t.Name())
            for _, e := range t.buf {
                fmt.Println(e.msg)
            }
            fmt.Println()
        }
        // 清空缓冲区
        t.buf = nil
    }
# 闭合前面的函数定义
```