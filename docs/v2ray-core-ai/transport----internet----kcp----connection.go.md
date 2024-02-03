# `v2ray-core\transport\internet\kcp\connection.go`

```go
// +build !confonly

package kcp

import (
    "bytes" // 导入 bytes 包，用于操作字节
    "io" // 导入 io 包，用于实现 I/O 操作
    "net" // 导入 net 包，用于网络编程
    "runtime" // 导入 runtime 包，用于访问运行时环境的信息
    "sync" // 导入 sync 包，用于实现并发控制
    "sync/atomic" // 导入 sync/atomic 包，用于原子操作
    "time" // 导入 time 包，用于时间相关操作

    "v2ray.com/core/common/buf" // 导入 v2ray.com/core/common/buf 包
    "v2ray.com/core/common/signal" // 导入 v2ray.com/core/common/signal 包
    "v2ray.com/core/common/signal/semaphore" // 导入 v2ray.com/core/common/signal/semaphore 包
)

var (
    ErrIOTimeout        = newError("Read/Write timeout") // 定义 Read/Write 超时错误
    ErrClosedListener   = newError("Listener closed.") // 定义监听器关闭错误
    ErrClosedConnection = newError("Connection closed.") // 定义连接关闭错误
)

// State of the connection
type State int32 // 定义连接状态类型为 int32

// Is returns true if current State is one of the candidates.
func (s State) Is(states ...State) bool { // 定义 State 类型的方法 Is，用于判断当前状态是否为给定状态之一
    for _, state := range states { // 遍历给定的状态列表
        if s == state { // 如果当前状态等于给定状态之一
            return true // 返回 true
        }
    }
    return false // 如果都不匹配，则返回 false
}

const (
    StateActive          State = 0 // Connection is active，定义连接活跃状态
    StateReadyToClose    State = 1 // Connection is closed locally，定义连接本地关闭状态
    StatePeerClosed      State = 2 // Connection is closed on remote，定义连接远程关闭状态
    StateTerminating     State = 3 // Connection is ready to be destroyed locally，定义连接本地准备销毁状态
    StatePeerTerminating State = 4 // Connection is ready to be destroyed on remote，定义连接远程准备销毁状态
    StateTerminated      State = 5 // Connection is destroyed.，定义连接销毁状态
)

func nowMillisec() int64 { // 定义函数 nowMillisec，用于获取当前时间的毫秒表示
    now := time.Now() // 获取当前时间
    return now.Unix()*1000 + int64(now.Nanosecond()/1000000) // 返回当前时间的毫秒表示
}

type RoundTripInfo struct { // 定义 RoundTripInfo 结构体
    sync.RWMutex // 嵌入 sync.RWMutex，实现读写锁
    variation        uint32 // 变化量
    srtt             uint32 // 平滑往返时间
    rto              uint32 // 超时重传时间
    minRtt           uint32 // 最小往返时间
    updatedTimestamp uint32 // 更新时间戳
}

func (info *RoundTripInfo) UpdatePeerRTO(rto uint32, current uint32) { // 定义方法 UpdatePeerRTO，用于更新对端的超时重传时间
    info.Lock() // 加写锁
    defer info.Unlock() // 延迟释放写锁

    if current-info.updatedTimestamp < 3000 { // 如果当前时间与上次更新时间间隔小于 3000 毫秒
        return // 不进行更新
    }

    info.updatedTimestamp = current // 更新时间戳
    info.rto = rto // 更新超时重传时间
}

func (info *RoundTripInfo) Update(rtt uint32, current uint32) { // 定义方法 Update，用于更新往返时间
    if rtt > 0x7FFFFFFF { // 如果往返时间大于最大值
        return // 不进行更新
    }
    info.Lock() // 加写锁
    defer info.Unlock() // 延迟释放写锁

    // https://tools.ietf.org/html/rfc6298
    if info.srtt == 0 { // 如果平滑往返时间为 0
        info.srtt = rtt // 更新平滑往返时间
        info.variation = rtt / 2 // 更新变化量
    } else {
        // 计算 RTT 的变化值
        delta := rtt - info.srtt
        // 如果当前的 RTT 小于平均 RTT，则取相反数
        if info.srtt > rtt {
            delta = info.srtt - rtt
        }
        // 更新 RTT 的变化值
        info.variation = (3*info.variation + delta) / 4
        // 更新平均 RTT
        info.srtt = (7*info.srtt + rtt) / 8
        // 如果平均 RTT 小于最小 RTT，则将平均 RTT 设置为最小 RTT
        if info.srtt < info.minRtt {
            info.srtt = info.minRtt
        }
    }
    // 计算 RTO
    var rto uint32
    // 如果最小 RTT 小于 4 倍的 RTT 变化值，则使用 srtt + 4*variation
    if info.minRtt < 4*info.variation {
        rto = info.srtt + 4*info.variation
    } else {
        // 否则使用 srtt + variation
        rto = info.srtt + info.variation
    }

    // 如果 RTO 大于 10000，则将 RTO 设置为 10000
    if rto > 10000 {
        rto = 10000
    }
    // 更新 RTO
    info.rto = rto * 5 / 4
    // 更新时间戳
    info.updatedTimestamp = current
}

// 获取 RoundTripInfo 结构体的超时时间
func (info *RoundTripInfo) Timeout() uint32 {
    // 加读锁
    info.RLock()
    // 在函数返回时释放读锁
    defer info.RUnlock()

    // 返回超时时间
    return info.rto
}

// 获取 RoundTripInfo 结构体的平滑时间
func (info *RoundTripInfo) SmoothedTime() uint32 {
    // 加读锁
    info.RLock()
    // 在函数返回时释放读锁
    defer info.RUnlock()

    // 返回平滑时间
    return info.srtt
}

// Updater 结构体
type Updater struct {
    interval        int64
    shouldContinue  func() bool
    shouldTerminate func() bool
    updateFunc      func()
    notifier        *semaphore.Instance
}

// 创建新的 Updater 结构体
func NewUpdater(interval uint32, shouldContinue func() bool, shouldTerminate func() bool, updateFunc func()) *Updater {
    // 初始化 Updater 结构体
    u := &Updater{
        interval:        int64(time.Duration(interval) * time.Millisecond),
        shouldContinue:  shouldContinue,
        shouldTerminate: shouldTerminate,
        updateFunc:      updateFunc,
        notifier:        semaphore.New(1),
    }
    // 返回 Updater 结构体
    return u
}

// 唤醒 Updater 结构体
func (u *Updater) WakeUp() {
    // 选择一个 case 执行
    select {
    // 如果可以从 notifier 中取出数据
    case <-u.notifier.Wait():
        // 启动一个协程来执行 run 方法
        go u.run()
    // 默认情况
    default:
    }
}

// 运行 Updater 结构体
func (u *Updater) run() {
    // 在函数返回时发送信号
    defer u.notifier.Signal()

    // 如果应该终止，则直接返回
    if u.shouldTerminate() {
        return
    }
    // 创建一个定时器
    ticker := time.NewTicker(u.Interval())
    // 循环执行，直到 shouldContinue 返回 false
    for u.shouldContinue() {
        // 执行更新函数
        u.updateFunc()
        // 等待定时器触发
        <-ticker.C
    }
    // 停止定时器
    ticker.Stop()
}

// 获取 Updater 结构体的时间间隔
func (u *Updater) Interval() time.Duration {
    return time.Duration(atomic.LoadInt64(&u.interval))
}

// 设置 Updater 结构体的时间间隔
func (u *Updater) SetInterval(d time.Duration) {
    atomic.StoreInt64(&u.interval, int64(d))
}

// ConnMetadata 结构体
type ConnMetadata struct {
    LocalAddr    net.Addr
    RemoteAddr   net.Addr
    Conversation uint16
}

// Connection 结构体，表示一个基于 UDP 的 KCP 连接
type Connection struct {
    meta       ConnMetadata
    closer     io.Closer
    rd         time.Time
    wd         time.Time // 写入截止时间
    since      int64
    dataInput  *signal.Notifier
    dataOutput *signal.Notifier
    Config     *Config

    state            State
    stateBeginTime   uint32
    lastIncomingTime uint32
    lastPingTime     uint32

    mss       uint32
    roundTrip *RoundTripInfo

    receivingWorker *ReceivingWorker
}
    # 声明一个名为sendingWorker的指针变量，指向SendingWorker类型的对象
    sendingWorker   *SendingWorker
    
    # 声明一个名为output的SegmentWriter类型的变量
    output SegmentWriter
    
    # 声明一个名为dataUpdater的指针变量，指向Updater类型的对象
    dataUpdater *Updater
    
    # 声明一个名为pingUpdater的指针变量，指向Updater类型的对象
    pingUpdater *Updater
// NewConnection 创建本地和远程之间的新 KCP 连接。
func NewConnection(meta ConnMetadata, writer PacketWriter, closer io.Closer, config *Config) *Connection {
    // 记录创建连接的元数据信息到日志
    newError("#", meta.Conversation, " creating connection to ", meta.RemoteAddr).WriteToLog()

    // 创建连接对象
    conn := &Connection{
        meta:       meta,
        closer:     closer,
        since:      nowMillisec(),
        dataInput:  signal.NewNotifier(),
        dataOutput: signal.NewNotifier(),
        Config:     config,
        output:     NewRetryableWriter(NewSegmentWriter(writer)),
        mss:        config.GetMTUValue() - uint32(writer.Overhead()) - DataSegmentOverhead,
        roundTrip: &RoundTripInfo{
            rto:    100,
            minRtt: config.GetTTIValue(),
        },
    }

    // 创建接收数据的工作线程
    conn.receivingWorker = NewReceivingWorker(conn)
    // 创建发送数据的工作线程
    conn.sendingWorker = NewSendingWorker(conn)

    // 判断连接是否正在终止
    isTerminating := func() bool {
        return conn.State().Is(StateTerminating, StateTerminated)
    }
    // 判断连接是否已经终止
    isTerminated := func() bool {
        return conn.State() == StateTerminated
    }
    // 创建数据更新器
    conn.dataUpdater = NewUpdater(
        config.GetTTIValue(),
        func() bool {
            return !isTerminating() && (conn.sendingWorker.UpdateNecessary() || conn.receivingWorker.UpdateNecessary())
        },
        isTerminating,
        conn.updateTask)
    // 创建 ping 更新器
    conn.pingUpdater = NewUpdater(
        5000, // 5 seconds
        func() bool { return !isTerminated() },
        isTerminated,
        conn.updateTask)
    // 唤醒 ping 更新器
    conn.pingUpdater.WakeUp()

    // 返回连接对象
    return conn
}

// Elapsed 返回连接自创建以来经过的时间
func (c *Connection) Elapsed() uint32 {
    return uint32(nowMillisec() - c.since)
}

// ReadMultiBuffer 实现了 buf.Reader 接口
func (c *Connection) ReadMultiBuffer() (buf.MultiBuffer, error) {
    // 如果连接对象为空，则返回错误
    if c == nil {
        return nil, io.EOF
    }
    # 无限循环，直到条件满足才会退出
    for {
        # 如果连接状态为准备关闭、正在终止、已终止，则返回空，表示文件结束
        if c.State().Is(StateReadyToClose, StateTerminating, StateTerminated) {
            return nil, io.EOF
        }
        # 从接收工作线程读取多缓冲区数据
        mb := c.receivingWorker.ReadMultiBuffer()
        # 如果多缓冲区不为空
        if !mb.IsEmpty() {
            # 唤醒数据更新器
            c.dataUpdater.WakeUp()
            # 返回多缓冲区数据和空错误
            return mb, nil
        }

        # 如果连接状态为对等端正在终止，则返回空，表示文件结束
        if c.State() == StatePeerTerminating {
            return nil, io.EOF
        }

        # 等待数据输入，如果出现错误则返回空和错误
        if err := c.waitForDataInput(); err != nil {
            return nil, err
        }
    }
// 等待数据输入的方法，返回错误
func (c *Connection) waitForDataInput() error {
    // 循环16次，等待数据输入
    for i := 0; i < 16; i++ {
        // 使用 select 语句监听数据输入的等待通道
        select {
        case <-c.dataInput.Wait():
            // 如果有数据输入，则返回 nil
            return nil
        default:
            // 如果没有数据输入，则让出当前线程的执行权
            runtime.Gosched()
        }
    }

    // 设置超时时间为16秒
    duration := time.Second * 16
    // 如果读取截止时间不为零，则计算剩余时间
    if !c.rd.IsZero() {
        duration = time.Until(c.rd)
        // 如果剩余时间小于0，则返回超时错误
        if duration < 0 {
            return ErrIOTimeout
        }
    }

    // 创建一个定时器，设置超时时间为 duration
    timeout := time.NewTimer(duration)
    // 延迟关闭定时器
    defer timeout.Stop()

    // 使用 select 语句监听数据输入的等待通道和超时定时器
    select {
    case <-c.dataInput.Wait():
    case <-timeout.C:
        // 如果读取截止时间不为零且在当前时间之前，则返回超时错误
        if !c.rd.IsZero() && c.rd.Before(time.Now()) {
            return ErrIOTimeout
        }
    }

    // 返回 nil
    return nil
}

// 实现 Conn 的 Read 方法
func (c *Connection) Read(b []byte) (int, error) {
    // 如果连接为 nil，则返回0和文件结束错误
    if c == nil {
        return 0, io.EOF
    }

    // 循环读取数据
    for {
        // 如果连接状态为 StateReadyToClose、StateTerminating 或 StateTerminated，则返回0和文件结束错误
        if c.State().Is(StateReadyToClose, StateTerminating, StateTerminated) {
            return 0, io.EOF
        }
        // 从接收工作器中读取数据
        nBytes := c.receivingWorker.Read(b)
        // 如果读取到数据，则唤醒数据更新器并返回读取的字节数和 nil
        if nBytes > 0 {
            c.dataUpdater.WakeUp()
            return nBytes, nil
        }

        // 如果等待数据输入出错，则返回0和错误
        if err := c.waitForDataInput(); err != nil {
            return 0, err
        }
    }
}

// 等待数据输出的方法，返回错误
func (c *Connection) waitForDataOutput() error {
    // 循环16次，等待数据输出
    for i := 0; i < 16; i++ {
        // 使用 select 语句监听数据输出的等待通道
        select {
        case <-c.dataOutput.Wait():
            // 如果有数据输出，则返回 nil
            return nil
        default:
            // 如果没有数据输出，则让出当前线程的执行权
            runtime.Gosched()
        }
    }

    // 设置超时时间为16秒
    duration := time.Second * 16
    // 如果写入截止时间不为零，则计算剩余时间
    if !c.wd.IsZero() {
        duration = time.Until(c.wd)
        // 如果剩余时间小于0，则返回超时错误
        if duration < 0 {
            return ErrIOTimeout
        }
    }

    // 创建一个定时器，设置超时时间为 duration
    timeout := time.NewTimer(duration)
    // 延迟关闭定时器
    defer timeout.Stop()

    // 使用 select 语句监听数据输出的等待通道和超时定时器
    select {
    case <-c.dataOutput.Wait():
    case <-timeout.C:
        // 如果写入截止时间不为零且在当前时间之前，则返回超时错误
        if !c.wd.IsZero() && c.wd.Before(time.Now()) {
            return ErrIOTimeout
        }
    }

    // 返回 nil
    return nil
}

// 实现 io.Writer 的 Write 方法
func (c *Connection) Write(b []byte) (int, error) {
    // 创建一个字节流读取器
    reader := bytes.NewReader(b)
    // 调用内部的写入多缓冲区方法，如果出错则返回0和错误
    if err := c.writeMultiBufferInternal(reader); err != nil {
        return 0, err
    # 返回变量 b 的长度和空值
    return len(b), nil
// WriteMultiBuffer 实现了 buf.Writer 接口
func (c *Connection) WriteMultiBuffer(mb buf.MultiBuffer) error {
    // 创建一个包含多缓冲区的读取器
    reader := &buf.MultiBufferContainer{
        MultiBuffer: mb,
    }
    // 在函数返回时关闭读取器
    defer reader.Close()

    // 调用内部的写多缓冲区函数
    return c.writeMultiBufferInternal(reader)
}

// 内部的写多缓冲区函数
func (c *Connection) writeMultiBufferInternal(reader io.Reader) error {
    // 标记是否有待更新的数据
    updatePending := false
    // 在函数返回时，如果有待更新的数据，则唤醒数据更新器
    defer func() {
        if updatePending {
            c.dataUpdater.WakeUp()
        }
    }()

    var b *buf.Buffer
    // 在函数返回时释放缓冲区
    defer b.Release()

    for {
        for {
            // 如果连接为 nil 或者状态不是活动状态，则返回管道关闭错误
            if c == nil || c.State() != StateActive {
                return io.ErrClosedPipe
            }

            if b == nil {
                // 创建一个新的缓冲区，并从读取器中读取数据
                b = buf.New()
                _, err := b.ReadFrom(io.LimitReader(reader, int64(c.mss)))
                if err != nil {
                    return nil
                }
            }

            if !c.sendingWorker.Push(b) {
                break
            }
            // 标记有待更新的数据
            updatePending = true
            b = nil
        }

        if updatePending {
            // 如果有待更新的数据，则唤醒数据更新器并重置标记
            c.dataUpdater.WakeUp()
            updatePending = false
        }

        // 等待数据输出
        if err := c.waitForDataOutput(); err != nil {
            return err
        }
    }
}

// 设置连接状态
func (c *Connection) SetState(state State) {
    // 获取当前时间
    current := c.Elapsed()
    // 更新连接状态和状态开始时间
    atomic.StoreInt32((*int32)(&c.state), int32(state))
    atomic.StoreUint32(&c.stateBeginTime, current)
    // 记录调试日志
    newError("#", c.meta.Conversation, " entering state ", state, " at ", current).AtDebug().WriteToLog()

    // 根据状态执行相应的操作
    switch state {
    case StateReadyToClose:
        c.receivingWorker.CloseRead()
    case StatePeerClosed:
        c.sendingWorker.CloseWrite()
    case StateTerminating:
        c.receivingWorker.CloseRead()
        c.sendingWorker.CloseWrite()
        c.pingUpdater.SetInterval(time.Second)
    case StatePeerTerminating:
        c.sendingWorker.CloseWrite()
        c.pingUpdater.SetInterval(time.Second)
    }
}
    # 根据状态为 StateTerminated 执行不同操作
    case StateTerminated:
        # 关闭接收工作者的读取通道
        c.receivingWorker.CloseRead()
        # 关闭发送工作者的写入通道
        c.sendingWorker.CloseWrite()
        # 设置 ping 更新器的时间间隔为 1 秒
        c.pingUpdater.SetInterval(time.Second)
        # 唤醒数据更新器
        c.dataUpdater.WakeUp()
        # 唤醒 ping 更新器
        c.pingUpdater.WakeUp()
        # 启动终止操作
        go c.Terminate()
    }
// Close方法用于关闭连接
func (c *Connection) Close() error {
    // 如果连接为空，则返回错误信息
    if c == nil {
        return ErrClosedConnection
    }
    // 发送数据输入信号
    c.dataInput.Signal()
    // 发送数据输出信号
    c.dataOutput.Signal()
    // 根据连接状态进行不同的操作
    switch c.State() {
    case StateReadyToClose, StateTerminating, StateTerminated:
        return ErrClosedConnection
    case StateActive:
        c.SetState(StateReadyToClose)
    case StatePeerClosed:
        c.SetState(StateTerminating)
    case StatePeerTerminating:
        c.SetState(StateTerminated)
    }
    // 记录关闭连接的日志
    newError("#", c.meta.Conversation, " closing connection to ", c.meta.RemoteAddr).WriteToLog()
    // 返回空值
    return nil
}

// LocalAddr方法返回本地网络地址
func (c *Connection) LocalAddr() net.Addr {
    // 如果连接为空，则返回空值
    if c == nil {
        return nil
    }
    // 返回本地地址
    return c.meta.LocalAddr
}

// RemoteAddr方法返回远程网络地址
func (c *Connection) RemoteAddr() net.Addr {
    // 如果连接为空，则返回空值
    if c == nil {
        return nil
    }
    // 返回远程地址
    return c.meta.RemoteAddr
}

// SetDeadline方法设置与监听器关联的截止日期。零时间值会禁用截止日期。
func (c *Connection) SetDeadline(t time.Time) error {
    // 如果设置读取截止日期出错，则返回错误信息
    if err := c.SetReadDeadline(t); err != nil {
        return err
    }
    // 设置写入截止日期
    return c.SetWriteDeadline(t)
}

// SetReadDeadline实现了Conn SetReadDeadline方法
func (c *Connection) SetReadDeadline(t time.Time) error {
    // 如果连接为空或者状态不是活动状态，则返回错误信息
    if c == nil || c.State() != StateActive {
        return ErrClosedConnection
    }
    // 设置读取截止日期
    c.rd = t
    // 返回空值
    return nil
}

// SetWriteDeadline实现了Conn SetWriteDeadline方法
func (c *Connection) SetWriteDeadline(t time.Time) error {
    // 如果连接为空或者状态不是活动状态，则返回错误信息
    if c == nil || c.State() != StateActive {
        return ErrClosedConnection
    }
    // 设置写入截止日期
    c.wd = t
    // 返回空值
    return nil
}

// kcp update, input loop
func (c *Connection) updateTask() {
    // 刷新连接
    c.flush()
}

// Terminate方法用于终止连接
func (c *Connection) Terminate() {
    // 如果 c 为空，则直接返回
    if c == nil {
        return
    }
    // 记录错误日志，表示连接终止
    newError("#", c.meta.Conversation, " terminating connection to ", c.RemoteAddr()).WriteToLog()

    // 通知数据输入线程结束
    c.dataInput.Signal()
    // 通知数据输出线程结束
    c.dataOutput.Signal()

    // 关闭连接
    c.closer.Close()
    // 释放发送数据线程
    c.sendingWorker.Release()
    // 释放接收数据线程
    c.receivingWorker.Release()
// 处理连接选项，根据选项中的关闭标志调用对等方关闭函数
func (c *Connection) HandleOption(opt SegmentOption) {
    // 检查选项中是否包含关闭标志，如果包含则调用对等方关闭函数
    if (opt & SegmentOptionClose) == SegmentOptionClose {
        c.OnPeerClosed()
    }
}

// 对等方关闭函数，根据连接状态进行相应的处理
func (c *Connection) OnPeerClosed() {
    // 根据连接状态进行不同的处理
    switch c.State() {
    case StateReadyToClose:
        c.SetState(StateTerminating)
    case StateActive:
        c.SetState(StatePeerClosed)
    }
}

// 当接收到低级数据包（例如 UDP 数据包）时调用该函数
func (c *Connection) Input(segments []Segment) {
    // 获取当前时间
    current := c.Elapsed()
    // 将当前时间存储到连接的最后接收时间中
    atomic.StoreUint32(&c.lastIncomingTime, current)
}
    # 遍历 segments 列表中的每个元素
    for _, seg := range segments:
        # 如果当前段的会话与元数据中的会话不同，则跳出循环
        if seg.Conversation() != c.meta.Conversation:
            break

        # 根据当前段的类型进行不同的处理
        switch seg := seg.(type):
            # 如果是数据段
            case *DataSegment:
                # 处理选项
                c.HandleOption(seg.Option)
                # 处理接收工作
                c.receivingWorker.ProcessSegment(seg)
                # 如果有可用数据，则发出数据输入信号
                if c.receivingWorker.IsDataAvailable():
                    c.dataInput.Signal()
                # 唤醒数据更新器
                c.dataUpdater.WakeUp()
            # 如果是确认段
            case *AckSegment:
                # 处理选项
                c.HandleOption(seg.Option)
                # 处理发送工作
                c.sendingWorker.ProcessSegment(current, seg, c.roundTrip.Timeout())
                # 发出数据输出信号
                c.dataOutput.Signal()
                # 唤醒数据更新器
                c.dataUpdater.WakeUp()
            # 如果是仅命令段
            case *CmdOnlySegment:
                # 处理选项
                c.HandleOption(seg.Option)
                # 如果命令是终止命令
                if seg.Command() == CommandTerminate:
                    # 根据当前状态进行不同的处理
                    switch c.State():
                        case StateActive, StatePeerClosed:
                            c.SetState(StatePeerTerminating)
                        case StateReadyToClose:
                            c.SetState(StateTerminating)
                        case StateTerminating:
                            c.SetState(StateTerminated)
                # 如果选项是关闭段或者命令是终止命令
                if seg.Option == SegmentOptionClose || seg.Command() == CommandTerminate:
                    # 发出数据输入信号
                    c.dataInput.Signal()
                    # 发出数据输出信号
                    c.dataOutput.Signal()
                # 处理接收下一个段
                c.sendingWorker.ProcessReceivingNext(seg.ReceivingNext)
                # 处理发送下一个段
                c.receivingWorker.ProcessSendingNext(seg.SendingNext)
                # 更新对等方的往返时间
                c.roundTrip.UpdatePeerRTO(seg.PeerRTO, current)
                # 释放当前段
                seg.Release()
            # 默认情况
            default:
        # 结束 switch 语句
        # 结束 for 循环
// 刷新连接状态
func (c *Connection) flush() {
    // 获取当前时间
    current := c.Elapsed()

    // 如果连接状态为终止，则返回
    if c.State() == StateTerminated {
        return
    }
    // 如果连接状态为活跃且当前时间减去上次接收时间大于等于30000毫秒，则关闭连接
    if c.State() == StateActive && current-atomic.LoadUint32(&c.lastIncomingTime) >= 30000 {
        c.Close()
    }
    // 如果连接状态为准备关闭且发送工作队列为空，则设置连接状态为终止中
    if c.State() == StateReadyToClose && c.sendingWorker.IsEmpty() {
        c.SetState(StateTerminating)
    }

    // 如果连接状态为终止中
    if c.State() == StateTerminating {
        // 记录发送终止命令的调试信息
        newError("#", c.meta.Conversation, " sending terminating cmd.").AtDebug().WriteToLog()
        // 发送Ping命令
        c.Ping(current, CommandTerminate)

        // 如果当前时间减去状态开始时间大于8000毫秒，则设置连接状态为已终止
        if current-atomic.LoadUint32(&c.stateBeginTime) > 8000 {
            c.SetState(StateTerminated)
        }
        return
    }
    // 如果连接状态为对端终止中且当前时间减去状态开始时间大于4000毫秒，则设置连接状态为终止中
    if c.State() == StatePeerTerminating && current-atomic.LoadUint32(&c.stateBeginTime) > 4000 {
        c.SetState(StateTerminating)
    }

    // 如果连接状态为准备关闭且当前时间减去状态开始时间大于15000毫秒，则设置连接状态为终止中
    if c.State() == StateReadyToClose && current-atomic.LoadUint32(&c.stateBeginTime) > 15000 {
        c.SetState(StateTerminating)
    }

    // 刷新接收工作队列
    c.receivingWorker.Flush(current)
    // 刷新发送工作队列
    c.sendingWorker.Flush(current)

    // 如果当前时间减去上次Ping时间大于等于3000毫秒，则发送Ping命令
    if current-atomic.LoadUint32(&c.lastPingTime) >= 3000 {
        c.Ping(current, CommandPing)
    }
}

// 获取连接状态
func (c *Connection) State() State {
    return State(atomic.LoadInt32((*int32)(&c.state)))
}

// 发送Ping命令
func (c *Connection) Ping(current uint32, cmd Command) {
    // 创建只包含命令的数据段
    seg := NewCmdOnlySegment()
    seg.Conv = c.meta.Conversation
    seg.Cmd = cmd
    seg.ReceivingNext = c.receivingWorker.NextNumber()
    seg.SendingNext = c.sendingWorker.FirstUnacknowledged()
    seg.PeerRTO = c.roundTrip.Timeout()
    // 如果连接状态为准备关闭，则设置数据段选项为关闭
    if c.State() == StateReadyToClose {
        seg.Option = SegmentOptionClose
    }
    // 将数据段写入输出流
    c.output.Write(seg)
    // 更新上次Ping时间
    atomic.StoreUint32(&c.lastPingTime, current)
    // 释放数据段资源
    seg.Release()
}
```