# `v2ray-core\transport\internet\kcp\receiving.go`

```go
// +build !confonly
// 定义了 kcp 包，不包含 confonly 标签

package kcp
// 导入必要的包
import (
    "sync"
    "v2ray.com/core/common/buf"
)

// 定义接收窗口结构体
type ReceivingWindow struct {
    cache map[uint32]*DataSegment
}

// 创建新的接收窗口对象
func NewReceivingWindow() *ReceivingWindow {
    return &ReceivingWindow{
        cache: make(map[uint32]*DataSegment),
    }
}

// 设置接收窗口中的数据段
func (w *ReceivingWindow) Set(id uint32, value *DataSegment) bool {
    _, f := w.cache[id]
    if f {
        return false
    }
    w.cache[id] = value
    return true
}

// 检查接收窗口中是否存在指定的数据段
func (w *ReceivingWindow) Has(id uint32) bool {
    _, f := w.cache[id]
    return f
}

// 从接收窗口中移除指定的数据段
func (w *ReceivingWindow) Remove(id uint32) *DataSegment {
    v, f := w.cache[id]
    if !f {
        return nil
    }
    delete(w.cache, id)
    return v
}

// 定义应答列表结构体
type AckList struct {
    writer     SegmentWriter
    timestamps []uint32
    numbers    []uint32
    nextFlush  []uint32

    flushCandidates []uint32
    dirty           bool
}

// 创建新的应答列表对象
func NewAckList(writer SegmentWriter) *AckList {
    return &AckList{
        writer:          writer,
        timestamps:      make([]uint32, 0, 128),
        numbers:         make([]uint32, 0, 128),
        nextFlush:       make([]uint32, 0, 128),
        flushCandidates: make([]uint32, 0, 128),
    }
}

// 向应答列表中添加数据段的序号和时间戳
func (l *AckList) Add(number uint32, timestamp uint32) {
    l.timestamps = append(l.timestamps, timestamp)
    l.numbers = append(l.numbers, number)
    l.nextFlush = append(l.nextFlush, 0)
    l.dirty = true
}

// 清除应答列表中小于给定序号的数据段
func (l *AckList) Clear(una uint32) {
    count := 0
    for i := 0; i < len(l.numbers); i++ {
        if l.numbers[i] < una {
            continue
        }
        if i != count {
            l.numbers[count] = l.numbers[i]
            l.timestamps[count] = l.timestamps[i]
            l.nextFlush[count] = l.nextFlush[i]
        }
        count++
    }
    if count < len(l.numbers) {
        l.numbers = l.numbers[:count]
        l.timestamps = l.timestamps[:count]
        l.nextFlush = l.nextFlush[:count]
        l.dirty = true
    }
}
func (l *AckList) Flush(current uint32, rto uint32) {
    // 清空刷新候选列表
    l.flushCandidates = l.flushCandidates[:0]

    // 创建一个新的确认段
    seg := NewAckSegment()
    // 遍历序号列表
    for i := 0; i < len(l.numbers); i++ {
        // 如果下次刷新时间大于当前时间
        if l.nextFlush[i] > current {
            // 如果刷新候选列表长度小于容量
            if len(l.flushCandidates) < cap(l.flushCandidates) {
                // 将序号添加到刷新候选列表
                l.flushCandidates = append(l.flushCandidates, l.numbers[i])
            }
            continue
        }
        // 将序号和时间戳添加到确认段
        seg.PutNumber(l.numbers[i])
        seg.PutTimestamp(l.timestamps[i])
        // 计算超时时间
        timeout := rto / 2
        if timeout < 20 {
            timeout = 20
        }
        // 更新下次刷新时间
        l.nextFlush[i] = current + timeout

        // 如果确认段已满
        if seg.IsFull() {
            // 写入确认段
            l.writer.Write(seg)
            // 释放确认段资源
            seg.Release()
            // 创建一个新的确认段
            seg = NewAckSegment()
            // 标记为非脏数据
            l.dirty = false
        }
    }

    // 如果数据脏或者确认段不为空
    if l.dirty || !seg.IsEmpty() {
        // 遍历刷新候选列表
        for _, number := range l.flushCandidates {
            // 如果确认段已满
            if seg.IsFull() {
                break
            }
            // 将序号添加到确认段
            seg.PutNumber(number)
        }
        // 写入确认段
        l.writer.Write(seg)
        // 标记为非脏数据
        l.dirty = false
    }

    // 释放确认段资源
    seg.Release()
}

type ReceivingWorker struct {
    sync.RWMutex
    conn       *Connection
    leftOver   buf.MultiBuffer
    window     *ReceivingWindow
    acklist    *AckList
    nextNumber uint32
    windowSize uint32
}

func NewReceivingWorker(kcp *Connection) *ReceivingWorker {
    // 创建一个接收工作者
    worker := &ReceivingWorker{
        conn:       kcp,
        window:     NewReceivingWindow(),
        windowSize: kcp.Config.GetReceivingInFlightSize(),
    }
    // 创建一个确认列表
    worker.acklist = NewAckList(worker)
    return worker
}

func (w *ReceivingWorker) Release() {
    // 加锁
    w.Lock()
    // 释放剩余数据
    buf.ReleaseMulti(w.leftOver)
    w.leftOver = nil
    // 解锁
    w.Unlock()
}

func (w *ReceivingWorker) ProcessSendingNext(number uint32) {
    // 加锁
    w.Lock()
    defer w.Unlock()

    // 清除指定序号之前的数据
    w.acklist.Clear(number)
}

func (w *ReceivingWorker) ProcessSegment(seg *DataSegment) {
    // 加锁
    w.Lock()
    defer w.Unlock()

    // 获取数据段的序号
    number := seg.Number
    // 计算序号与下一个期望序号的差值
    idx := number - w.nextNumber
    // 如果差值大于等于窗口大小
    if idx >= w.windowSize {
        return
    }
}
    # 清空 acklist 中发送下一个数据包之前的所有数据包的确认信息
    w.acklist.Clear(seg.SendingNext)
    # 向 acklist 中添加指定序号和时间戳的确认信息
    w.acklist.Add(number, seg.Timestamp)

    # 如果窗口中已经存在相同序号的数据包，则不再添加该数据包
    if !w.window.Set(seg.Number, seg) {
        # 释放该数据包的资源
        seg.Release()
    }
# 读取多个缓冲区数据
func (w *ReceivingWorker) ReadMultiBuffer() buf.MultiBuffer {
    # 如果有未处理的数据，直接返回
    if w.leftOver != nil {
        mb := w.leftOver
        w.leftOver = nil
        return mb
    }

    # 创建一个空的多缓冲区
    mb := make(buf.MultiBuffer, 0, 32)

    # 加锁
    w.Lock()
    defer w.Unlock()
    # 循环读取窗口中的数据段，直到窗口为空
    for {
        seg := w.window.Remove(w.nextNumber)
        if seg == nil {
            break
        }
        w.nextNumber++
        mb = append(mb, seg.Detach())
        seg.Release()
    }

    return mb
}

# 读取数据到字节数组
func (w *ReceivingWorker) Read(b []byte) int {
    # 读取多个缓冲区数据
    mb := w.ReadMultiBuffer()
    # 如果数据为空，返回0
    if mb.IsEmpty() {
        return 0
    }
    # 将数据拆分到字节数组中，并返回拷贝的字节数
    mb, nBytes := buf.SplitBytes(mb, b)
    # 如果还有剩余数据，保存到leftOver中
    if !mb.IsEmpty() {
        w.leftOver = mb
    }
    return nBytes
}

# 检查是否有可用数据
func (w *ReceivingWorker) IsDataAvailable() bool {
    # 加读锁
    w.RLock()
    defer w.RUnlock()
    # 检查窗口中是否有下一个数据段
    return w.window.Has(w.nextNumber)
}

# 获取下一个数据段的编号
func (w *ReceivingWorker) NextNumber() uint32 {
    # 加读锁
    w.RLock()
    defer w.RUnlock()
    # 返回下一个数据段的编号
    return w.nextNumber
}

# 刷新数据
func (w *ReceivingWorker) Flush(current uint32) {
    # 加锁
    w.Lock()
    defer w.Unlock()
    # 刷新确认列表
    w.acklist.Flush(current, w.conn.roundTrip.Timeout())
}

# 写入数据段
func (w *ReceivingWorker) Write(seg Segment) error {
    # 转换数据段为确认数据段
    ackSeg := seg.(*AckSegment)
    ackSeg.Conv = w.conn.meta.Conversation
    ackSeg.ReceivingNext = w.nextNumber
    ackSeg.ReceivingWindow = w.nextNumber + w.windowSize
    ackSeg.Option = 0
    # 如果连接状态为准备关闭，则设置选项为关闭
    if w.conn.State() == StateReadyToClose {
        ackSeg.Option = SegmentOptionClose
    }
    # 写入确认数据段
    return w.conn.output.Write(ackSeg)
}

# 关闭读取
func (*ReceivingWorker) CloseRead() {
}

# 检查是否需要更新
func (w *ReceivingWorker) UpdateNecessary() bool {
    # 加读锁
    w.RLock()
    defer w.RUnlock()
    # 检查确认列表中是否有数据
    return len(w.acklist.numbers) > 0
}
```