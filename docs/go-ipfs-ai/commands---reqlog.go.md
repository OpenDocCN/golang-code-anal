# `kubo\commands\reqlog.go`

```go
package commands

import (
    "sync"  // 导入同步包，用于实现并发安全的操作
    "time"  // 导入时间包，用于处理时间相关的操作
)

// ReqLogEntry is an entry in the request log.
type ReqLogEntry struct {
    StartTime time.Time  // 请求开始时间
    EndTime   time.Time  // 请求结束时间
    Active    bool       // 请求是否活跃
    Command   string     // 请求的命令
    Options   map[string]interface{}  // 请求的选项
    Args      []string   // 请求的参数
    ID        int        // 请求的唯一标识

    log *ReqLog  // 请求日志对象
}

// Copy returns a copy of the ReqLogEntry.
func (r *ReqLogEntry) Copy() *ReqLogEntry {
    out := *r  // 复制请求日志对象
    out.log = nil  // 清空日志对象
    return &out  // 返回复制的请求日志对象
}

// ReqLog is a log of requests.
type ReqLog struct {
    Requests []*ReqLogEntry  // 请求日志条目列表
    nextID   int             // 下一个请求的唯一标识
    lock     sync.Mutex      // 互斥锁，用于保护并发操作
    keep     time.Duration   // 请求日志保留时间
}

// AddEntry adds an entry to the log.
func (rl *ReqLog) AddEntry(rle *ReqLogEntry) {
    rl.lock.Lock()  // 加锁
    defer rl.lock.Unlock()  // 延迟解锁

    rle.ID = rl.nextID  // 设置请求日志条目的唯一标识
    rl.nextID++  // 下一个请求的唯一标识加一
    rl.Requests = append(rl.Requests, rle)  // 将请求日志条目添加到请求日志列表中

    if !rle.Active {  // 如果请求不活跃
        rl.maybeCleanup()  // 可能进行清理操作
    }
}

// ClearInactive removes stale entries.
func (rl *ReqLog) ClearInactive() {
    rl.lock.Lock()  // 加锁
    defer rl.lock.Unlock()  // 延迟解锁

    k := rl.keep  // 保存请求日志保留时间
    rl.keep = 0  // 清零请求日志保留时间
    rl.cleanup()  // 执行清理操作
    rl.keep = k  // 恢复请求日志保留时间
}

func (rl *ReqLog) maybeCleanup() {
    // only do it every so often or it might
    // become a perf issue
    if len(rl.Requests)%10 == 0 {  // 每隔一段时间执行清理操作，避免性能问题
        rl.cleanup()  // 执行清理操作
    }
}

func (rl *ReqLog) cleanup() {
    i := 0  // 初始化索引
    now := time.Now()  // 获取当前时间
    for j := 0; j < len(rl.Requests); j++ {  // 遍历请求日志列表
        rj := rl.Requests[j]  // 获取请求日志条目
        if rj.Active || rl.Requests[j].EndTime.Add(rl.keep).After(now) {  // 如果请求活跃或者请求结束时间加上保留时间在当前时间之后
            rl.Requests[i] = rl.Requests[j]  // 将请求日志条目移动到指定位置
            i++  // 索引加一
        }
    }
    rl.Requests = rl.Requests[:i]  // 更新请求日志列表
}

// SetKeepTime sets a duration after which an entry will be considered inactive.
func (rl *ReqLog) SetKeepTime(t time.Duration) {
    rl.lock.Lock()  // 加锁
    defer rl.lock.Unlock()  // 延迟解锁
    rl.keep = t  // 设置请求日志保留时间
}

// Report generates a copy of all the entries in the requestlog.
func (rl *ReqLog) Report() []*ReqLogEntry {
    rl.lock.Lock()  // 加锁
    defer rl.lock.Unlock()  // 延迟解锁
    out := make([]*ReqLogEntry, len(rl.Requests))  // 创建请求日志条目的副本
    # 遍历请求列表中的每个请求，并获取索引和元素值
    for i, e := range rl.Requests {
        # 将每个请求的副本存储到输出列表中的相应位置
        out[i] = e.Copy()
    }
    # 返回包含请求副本的输出列表
    return out
// Finish 标记日志中的条目为已完成
func (rl *ReqLog) Finish(rle *ReqLogEntry) {
    // 获取锁，确保线程安全
    rl.lock.Lock()
    // 延迟释放锁
    defer rl.lock.Unlock()

    // 将条目标记为非活动状态
    rle.Active = false
    // 设置条目的结束时间为当前时间
    rle.EndTime = time.Now()

    // 可能进行清理操作
    rl.maybeCleanup()
}
```