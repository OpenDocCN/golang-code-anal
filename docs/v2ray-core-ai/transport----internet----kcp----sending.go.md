# `v2ray-core\transport\internet\kcp\sending.go`

```go
// +build !confonly

package kcp

import (
    "container/list"
    "sync"

    "v2ray.com/core/common/buf"
)

type SendingWindow struct {
    cache             *list.List  // 用于缓存数据段的链表
    totalInFlightSize uint32      // 当前发送窗口中所有数据段的总大小
    writer            SegmentWriter  // 用于发送数据段的接口
    onPacketLoss      func(uint32)  // 数据丢失时的回调函数
}

func NewSendingWindow(writer SegmentWriter, onPacketLoss func(uint32)) *SendingWindow {
    window := &SendingWindow{
        cache:        list.New(),  // 初始化数据段缓存链表
        writer:       writer,  // 设置数据段发送接口
        onPacketLoss: onPacketLoss,  // 设置数据丢失回调函数
    }
    return window
}

func (sw *SendingWindow) Release() {
    if sw == nil {
        return
    }
    for sw.cache.Len() > 0 {
        seg := sw.cache.Front().Value.(*DataSegment)  // 获取链表头部的数据段
        seg.Release()  // 释放数据段
        sw.cache.Remove(sw.cache.Front())  // 移除链表头部的数据段
    }
}

func (sw *SendingWindow) Len() uint32 {
    return uint32(sw.cache.Len())  // 返回缓存中数据段的数量
}

func (sw *SendingWindow) IsEmpty() bool {
    return sw.cache.Len() == 0  // 判断缓存是否为空
}

func (sw *SendingWindow) Push(number uint32, b *buf.Buffer) {
    seg := NewDataSegment()  // 创建新的数据段
    seg.Number = number  // 设置数据段序号
    seg.payload = b  // 设置数据段的载荷

    sw.cache.PushBack(seg)  // 将数据段加入缓存链表的尾部
}

func (sw *SendingWindow) FirstNumber() uint32 {
    return sw.cache.Front().Value.(*DataSegment).Number  // 返回缓存链表头部数据段的序号
}

func (sw *SendingWindow) Clear(una uint32) {
    for !sw.IsEmpty() {
        seg := sw.cache.Front().Value.(*DataSegment)  // 获取链表头部的数据段
        if seg.Number >= una {  // 如果数据段序号大于等于 una
            break
        }
        seg.Release()  // 释放数据段
        sw.cache.Remove(sw.cache.Front())  // 移除链表头部的数据段
    }
}

func (sw *SendingWindow) HandleFastAck(number uint32, rto uint32) {
    if sw.IsEmpty() {
        return
    }

    sw.Visit(func(seg *DataSegment) bool {
        if number == seg.Number || number-seg.Number > 0x7FFFFFFF {
            return false
        }

        if seg.transmit > 0 && seg.timeout > rto/3 {
            seg.timeout -= rto / 3
        }
        return true
    })
}

func (sw *SendingWindow) Visit(visitor func(seg *DataSegment) bool) {
    if sw.IsEmpty() {
        return
    }
    // 遍历缓存中的数据段，并对每个数据段执行访问者函数
}
    # 从缓存的链表头部开始遍历链表，直到链表末尾
    for e := sw.cache.Front(); e != nil; e = e.Next() {
        # 获取链表节点的值，并将其转换为 DataSegment 类型
        seg := e.Value.(*DataSegment)
        # 如果 visitor 函数返回 false，则跳出循环
        if !visitor(seg) {
            break
        }
    }
}



func (sw *SendingWindow) Flush(current uint32, rto uint32, maxInFlightSize uint32) {
    // 如果发送窗口为空，则直接返回
    if sw.IsEmpty() {
        return
    }

    var lost uint32
    var inFlightSize uint32

    sw.Visit(func(segment *DataSegment) bool {
        // 如果当前时间减去段超时时间大于等于0x7FFFFFFF，则返回true
        if current-segment.timeout >= 0x7FFFFFFF {
            return true
        }
        // 如果传输次数为0，则表示第一次传输，总传输大小加1
        if segment.transmit == 0 {
            // First time
            sw.totalInFlightSize++
        } else {
            lost++
        }
        // 设置段的超时时间为当前时间加上重传超时时间
        segment.timeout = current + rto

        segment.Timestamp = current
        segment.transmit++
        sw.writer.Write(segment)
        inFlightSize++
        return inFlightSize < maxInFlightSize
    })

    // 如果有丢包回调函数，并且当前传输大小大于0且总传输大小不为0，则计算丢包率并调用丢包回调函数
    if sw.onPacketLoss != nil && inFlightSize > 0 && sw.totalInFlightSize != 0 {
        rate := lost * 100 / sw.totalInFlightSize
        sw.onPacketLoss(rate)
    }
}

func (sw *SendingWindow) Remove(number uint32) bool {
    // 如果发送窗口为空，则返回false
    if sw.IsEmpty() {
        return false
    }

    // 遍历发送窗口中的数据段
    for e := sw.cache.Front(); e != nil; e = e.Next() {
        seg := e.Value.(*DataSegment)
        // 如果数据段的序号大于指定序号，则返回false
        if seg.Number > number {
            return false
        } else if seg.Number == number {
            // 如果总传输大小大于0，则总传输大小减1
            if sw.totalInFlightSize > 0 {
                sw.totalInFlightSize--
            }
            seg.Release()
            sw.cache.Remove(e)
            return true
        }
    }

    return false
}

type SendingWorker struct {
    sync.RWMutex
    conn                       *Connection
    window                     *SendingWindow
    firstUnacknowledged        uint32
    nextNumber                 uint32
    remoteNextNumber           uint32
    controlWindow              uint32
    fastResend                 uint32
    windowSize                 uint32
    firstUnacknowledgedUpdated bool
    closed                     bool
}

func NewSendingWorker(kcp *Connection) *SendingWorker {
    # 创建一个名为worker的SendingWorker结构体实例，并初始化其字段
    worker := &SendingWorker{
        conn:             kcp,  # 将kcp赋值给conn字段
        fastResend:       2,    # 将2赋值给fastResend字段
        remoteNextNumber: 32,   # 将32赋值给remoteNextNumber字段
        controlWindow:    kcp.Config.GetSendingInFlightSize(),  # 调用kcp.Config.GetSendingInFlightSize()方法并将结果赋值给controlWindow字段
        windowSize:       kcp.Config.GetSendingBufferSize(),    # 调用kcp.Config.GetSendingBufferSize()方法并将结果赋值给windowSize字段
    }
    # 创建一个名为window的SendingWindow实例，并传入worker和worker.OnPacketLoss作为参数
    worker.window = NewSendingWindow(worker, worker.OnPacketLoss)
    # 返回worker实例
    return worker
func (w *SendingWorker) Release() {
    // 加锁，确保线程安全
    w.Lock()
    // 释放窗口资源
    w.window.Release()
    // 标记为已关闭
    w.closed = true
    // 解锁
    w.Unlock()
}

func (w *SendingWorker) ProcessReceivingNext(nextNumber uint32) {
    // 加锁
    w.Lock()
    // 延迟解锁
    defer w.Unlock()
    // 调用无锁处理接收下一个数据包的方法
    w.ProcessReceivingNextWithoutLock(nextNumber)
}

func (w *SendingWorker) ProcessReceivingNextWithoutLock(nextNumber uint32) {
    // 清除窗口中小于等于指定序号的数据包
    w.window.Clear(nextNumber)
    // 查找第一个未被确认的数据包
    w.FindFirstUnacknowledged()
}

func (w *SendingWorker) FindFirstUnacknowledged() {
    // 保存当前的第一个未被确认的数据包序号
    first := w.firstUnacknowledged
    // 如果窗口不为空，更新第一个未被确认的数据包序号为窗口中的第一个序号，否则更新为下一个序号
    if !w.window.IsEmpty() {
        w.firstUnacknowledged = w.window.FirstNumber()
    } else {
        w.firstUnacknowledged = w.nextNumber
    }
    // 如果更新后的第一个未被确认的数据包序号与之前不一致，标记为已更新
    if first != w.firstUnacknowledged {
        w.firstUnacknowledgedUpdated = true
    }
}

func (w *SendingWorker) processAck(number uint32) bool {
    // 判断收到的确认号是否有效
    if number-w.firstUnacknowledged > 0x7FFFFFFF || number-w.nextNumber < 0x7FFFFFFF {
        return false
    }
    // 从窗口中移除指定序号的数据包
    removed := w.window.Remove(number)
    // 如果移除成功，重新查找第一个未被确认的数据包
    if removed {
        w.FindFirstUnacknowledged()
    }
    return removed
}

func (w *SendingWorker) ProcessSegment(current uint32, seg *AckSegment, rto uint32) {
    // 延迟释放段资源
    defer seg.Release()
    // 加锁
    w.Lock()
    // 延迟解锁
    defer w.Unlock()
    // 如果已关闭，直接返回
    if w.closed {
        return
    }
    // 更新远端下一个期望接收的数据包序号
    if w.remoteNextNumber < seg.ReceivingWindow {
        w.remoteNextNumber = seg.ReceivingWindow
    }
    // 无锁处理接收下一个数据包
    w.ProcessReceivingNextWithoutLock(seg.ReceivingNext)
    // 如果段为空，直接返回
    if seg.IsEmpty() {
        return
    }
    // 初始化最大确认号和是否已移除最大确认号的标志
    var maxack uint32
    var maxackRemoved bool
    // 遍历确认号列表
    for _, number := range seg.NumberList {
        // 处理确认号，并返回是否已移除的标志
        removed := w.processAck(number)
        // 更新最大确认号和是否已移除最大确认号的标志
        if maxack < number {
            maxack = number
            maxackRemoved = removed
        }
    }
    // 如果已移除最大确认号
    if maxackRemoved {
        // 处理快速确认
        w.window.HandleFastAck(maxack, rto)
        // 如果当前时间与段时间差小于10000，更新往返时间
        if current-seg.Timestamp < 10000 {
            w.conn.roundTrip.Update(current-seg.Timestamp, current)
        }
    }
}

func (w *SendingWorker) Push(b *buf.Buffer) bool {
    // 加锁
    w.Lock()
    # 释放写锁
    defer w.Unlock()

    # 如果写入器已关闭，则返回 false
    if w.closed {
        return false
    }

    # 如果窗口中的数据长度超过窗口大小，则返回 false
    if w.window.Len() > w.windowSize {
        return false
    }

    # 将数据推入窗口，并递增下一个数据的编号
    w.window.Push(w.nextNumber, b)
    w.nextNumber++
    # 返回 true，表示成功写入数据
    return true
# 写入数据段到连接
func (w *SendingWorker) Write(seg Segment) error:
    # 将传入的段转换为数据段
    dataSeg := seg.(*DataSegment)
    # 设置数据段的会话和发送下一个序号
    dataSeg.Conv = w.conn.meta.Conversation
    dataSeg.SendingNext = w.firstUnacknowledged
    dataSeg.Option = 0
    # 如果连接状态为准备关闭，则设置数据段选项为关闭选项
    if w.conn.State() == StateReadyToClose:
        dataSeg.Option = SegmentOptionClose
    # 将数据段写入连接的输出
    return w.conn.output.Write(dataSeg)

# 处理丢包情况
func (w *SendingWorker) OnPacketLoss(lossRate uint32):
    # 如果不使用拥塞控制或往返超时时间为0，则直接返回
    if !w.conn.Config.Congestion || w.conn.roundTrip.Timeout() == 0:
        return
    # 根据丢包率调整控制窗口大小
    if lossRate >= 15:
        w.controlWindow = 3 * w.controlWindow / 4
    elif lossRate <= 5:
        w.controlWindow += w.controlWindow / 4
    # 确保控制窗口不小于16，不大于发送窗口的两倍
    if w.controlWindow < 16:
        w.controlWindow = 16
    if w.controlWindow > 2*w.conn.Config.GetSendingInFlightSize():
        w.controlWindow = 2 * w.conn.Config.GetSendingInFlightSize()

# 刷新发送窗口
func (w *SendingWorker) Flush(current uint32):
    # 加锁
    w.Lock()
    # 如果连接已关闭，则解锁并返回
    if w.closed:
        w.Unlock()
        return
    # 计算拥塞窗口大小
    cwnd := w.firstUnacknowledged + w.conn.Config.GetSendingInFlightSize()
    if cwnd > w.remoteNextNumber:
        cwnd = w.remoteNextNumber
    # 如果使用拥塞控制且拥塞窗口大于控制窗口，则将拥塞窗口设置为控制窗口
    if w.conn.Config.Congestion && cwnd > w.firstUnacknowledged+w.controlWindow:
        cwnd = w.firstUnacknowledged + w.controlWindow
    # 如果发送窗口不为空，则刷新发送窗口，并标记已更新
    if !w.window.IsEmpty():
        w.window.Flush(current, w.conn.roundTrip.Timeout(), cwnd)
        w.firstUnacknowledgedUpdated = false
    updated := w.firstUnacknowledgedUpdated
    w.firstUnacknowledgedUpdated = false
    w.Unlock()
    # 如果发送窗口已更新，则发送 Ping 命令
    if updated:
        w.conn.Ping(current, CommandPing)

# 关闭写入
func (w *SendingWorker) CloseWrite():
    # 加锁
    w.Lock()
    defer w.Unlock()
    # 清空发送窗口
    w.window.Clear(0xFFFFFFFF)

# 判断发送窗口是否为空
func (w *SendingWorker) IsEmpty() bool:
    # 加读锁
    w.RLock()
    defer w.RUnlock()
    # 返回发送窗口是否为空的结果
    return w.window.IsEmpty()

# 判断是否需要更新
func (w *SendingWorker) UpdateNecessary() bool:
    # 返回发送窗口是否不为空的结果
    return !w.IsEmpty()

# 获取第一个未确认的序号
func (w *SendingWorker) FirstUnacknowledged() uint32:
    # 加读锁
    w.RLock()
    defer w.RUnlock()
    # 返回变量 w 的属性 firstUnacknowledged 的值
    return w.firstUnacknowledged
# 闭合前面的函数定义
```