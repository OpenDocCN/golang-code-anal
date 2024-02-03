# `v2ray-core\common\mux\client.go`

```go
package mux

import (
    "context" // 上下文包，用于传递请求的上下文信息
    "io" // 输入输出包，提供了基本的输入输出接口
    "sync" // 同步包，提供了并发安全的锁和条件变量等基本同步原语
    "time" // 时间包，提供了时间的表示和计算

    "v2ray.com/core/common" // V2Ray 核心通用包
    "v2ray.com/core/common/buf" // V2Ray 核心缓冲区包
    "v2ray.com/core/common/errors" // V2Ray 核心错误处理包
    "v2ray.com/core/common/net" // V2Ray 核心网络包
    "v2ray.com/core/common/protocol" // V2Ray 核心协议包
    "v2ray.com/core/common/session" // V2Ray 核心会话包
    "v2ray.com/core/common/signal/done" // V2Ray 核心信号处理包
    "v2ray.com/core/common/task" // V2Ray 核心任务包
    "v2ray.com/core/proxy" // V2Ray 代理包
    "v2ray.com/core/transport" // V2Ray 传输包
    "v2ray.com/core/transport/internet" // V2Ray 传输协议包
    "v2ray.com/core/transport/pipe" // V2Ray 传输管道包
)

type ClientManager struct {
    Enabled bool // 是否从用户配置中启用了多路复用
    Picker  WorkerPicker // 工作器选择器
}

func (m *ClientManager) Dispatch(ctx context.Context, link *transport.Link) error {
    for i := 0; i < 16; i++ {
        worker, err := m.Picker.PickAvailable() // 选择可用的工作器
        if err != nil {
            return err
        }
        if worker.Dispatch(ctx, link) { // 如果工作器成功分发请求
            return nil
        }
    }

    return newError("unable to find an available mux client").AtWarning() // 返回错误信息
}

type WorkerPicker interface {
    PickAvailable() (*ClientWorker, error) // 选择可用的工作器接口
}

type IncrementalWorkerPicker struct {
    Factory ClientWorkerFactory // 工作器工厂

    access      sync.Mutex // 互斥锁
    workers     []*ClientWorker // 客户端工作器列表
    cleanupTask *task.Periodic // 定时清理任务
}

func (p *IncrementalWorkerPicker) cleanupFunc() error {
    p.access.Lock() // 加锁
    defer p.access.Unlock() // 延迟解锁

    if len(p.workers) == 0 {
        return newError("no worker") // 返回错误信息
    }

    p.cleanup() // 执行清理操作
    return nil
}

func (p *IncrementalWorkerPicker) cleanup() {
    var activeWorkers []*ClientWorker
    for _, w := range p.workers {
        if !w.Closed() { // 如果工作器未关闭
            activeWorkers = append(activeWorkers, w) // 将工作器添加到活跃工作器列表中
        }
    }
    p.workers = activeWorkers // 更新工作器列表
}

func (p *IncrementalWorkerPicker) findAvailable() int {
    for idx, w := range p.workers {
        if !w.IsFull() { // 如果工作器未满
            return idx // 返回可用工作器的索引
        }
    }

    return -1 // 返回无可用工作器的标识
}

func (p *IncrementalWorkerPicker) pickInternal() (*ClientWorker, bool, error) {
    p.access.Lock() // 加锁
    defer p.access.Unlock() // 延迟解锁

    idx := p.findAvailable() // 查找可用的工作器
    # 如果索引大于等于0
    if idx >= 0 {
        # 获取工作者列表的长度
        n := len(p.workers)
        # 如果工作者数量大于1并且索引不是最后一个
        if n > 1 && idx != n-1 {
            # 交换最后一个工作者和指定索引的工作者
            p.workers[n-1], p.workers[idx] = p.workers[idx], p.workers[n-1]
        }
        # 返回指定索引的工作者和false，表示工作者已存在
        return p.workers[idx], false, nil
    }

    # 清理工作者
    p.cleanup()

    # 创建新的工作者
    worker, err := p.Factory.Create()
    # 如果创建过程中出现错误
    if err != nil {
        # 返回nil，false和错误信息
        return nil, false, err
    }
    # 将新工作者添加到工作者列表中
    p.workers = append(p.workers, worker)

    # 如果清理任务为空
    if p.cleanupTask == nil {
        # 创建一个定时任务，每30秒执行一次清理函数
        p.cleanupTask = &task.Periodic{
            Interval: time.Second * 30,
            Execute:  p.cleanupFunc,
        }
    }

    # 返回新工作者和true，表示工作者是新创建的
    return worker, true, nil
}

// PickAvailable 从增量工作选择器中选择可用的客户端工作者
func (p *IncrementalWorkerPicker) PickAvailable() (*ClientWorker, error) {
    // 从内部选择器中选择工作者、开始标志和错误
    worker, start, err := p.pickInternal()
    // 如果开始标志为真
    if start {
        // 强制执行清理任务的开始
        common.Must(p.cleanupTask.Start())
    }

    // 返回工作者和错误
    return worker, err
}

// ClientWorkerFactory 客户端工作者工厂接口
type ClientWorkerFactory interface {
    Create() (*ClientWorker, error)
}

// DialingWorkerFactory 拨号工作者工厂结构
type DialingWorkerFactory struct {
    Proxy    proxy.Outbound
    Dialer   internet.Dialer
    Strategy ClientStrategy
}

// Create 根据拨号工作者工厂创建客户端工作者
func (f *DialingWorkerFactory) Create() (*ClientWorker, error) {
    // 创建管道选项
    opts := []pipe.Option{pipe.WithSizeLimit(64 * 1024)}
    // 创建上行和下行管道
    uplinkReader, upLinkWriter := pipe.New(opts...)
    downlinkReader, downlinkWriter := pipe.New(opts...)

    // 创建新的客户端工作者
    c, err := NewClientWorker(transport.Link{
        Reader: downlinkReader,
        Writer: upLinkWriter,
    }, f.Strategy)

    // 如果有错误则返回空和错误
    if err != nil {
        return nil, err
    }

    // 启动处理器处理客户端连接
    go func(p proxy.Outbound, d internet.Dialer, c common.Closable) {
        // 创建带有出站信息的上下文
        ctx := session.ContextWithOutbound(context.Background(), &session.Outbound{
            Target: net.TCPDestination(muxCoolAddress, muxCoolPort),
        })
        ctx, cancel := context.WithCancel(ctx)

        // 如果处理出现错误，则记录错误日志
        if err := p.Process(ctx, &transport.Link{Reader: uplinkReader, Writer: downlinkWriter}, d); err != nil {
            errors.New("failed to handler mux client connection").Base(err).WriteToLog()
        }
        // 强制关闭
        common.Must(c.Close())
        cancel()
    }(f.Proxy, f.Dialer, c.done)

    // 返回客户端工作者和空
    return c, nil
}

// ClientStrategy 客户端策略结构
type ClientStrategy struct {
    MaxConcurrency uint32
    MaxConnection  uint32
}

// ClientWorker 客户端工作者结构
type ClientWorker struct {
    sessionManager *SessionManager
    link           transport.Link
    done           *done.Instance
    strategy       ClientStrategy
}

// muxCoolAddress 客户端连接地址
var muxCoolAddress = net.DomainAddress("v1.mux.cool")
// muxCoolPort 客户端连接端口
var muxCoolPort = net.Port(9527)

// NewClientWorker 创建一个新的 mux.Client
func NewClientWorker(stream transport.Link, s ClientStrategy) (*ClientWorker, error) {
    # 创建一个新的ClientWorker对象，并初始化其sessionManager、link、done和strategy属性
    c := &ClientWorker{
        sessionManager: NewSessionManager(),  # 初始化sessionManager属性为一个新的SessionManager对象
        link:           stream,  # 初始化link属性为stream对象
        done:           done.New(),  # 初始化done属性为一个新的Done对象
        strategy:       s,  # 初始化strategy属性为s对象
    }

    # 启动一个goroutine来执行fetchOutput方法
    go c.fetchOutput()
    # 启动一个goroutine来执行monitor方法
    go c.monitor()

    # 返回创建的ClientWorker对象和nil
    return c, nil
// TotalConnections 返回当前客户端的总连接数
func (m *ClientWorker) TotalConnections() uint32 {
    return uint32(m.sessionManager.Count())
}

// ActiveConnections 返回当前客户端的活跃连接数
func (m *ClientWorker) ActiveConnections() uint32 {
    return uint32(m.sessionManager.Size())
}

// Closed 如果客户端已关闭，则返回 true
func (m *ClientWorker) Closed() bool {
    return m.done.Done()
}

// monitor 监控客户端连接状态
func (m *ClientWorker) monitor() {
    // 创建一个定时器，每 16 秒触发一次
    timer := time.NewTicker(time.Second * 16)
    defer timer.Stop()

    for {
        select {
        case <-m.done.Wait():
            // 当客户端关闭时，关闭所有会话管理器和连接
            m.sessionManager.Close()
            common.Close(m.link.Writer)     // nolint: errcheck
            common.Interrupt(m.link.Reader) // nolint: errcheck
            return
        case <-timer.C:
            // 检查会话管理器中的连接数，如果为 0 并且会话管理器关闭成功，则关闭客户端
            size := m.sessionManager.Size()
            if size == 0 && m.sessionManager.CloseIfNoSession() {
                common.Must(m.done.Close())
            }
        }
    }
}

// writeFirstPayload 向目标写入第一个有效负载
func writeFirstPayload(reader buf.Reader, writer *Writer) error {
    // 从 reader 中复制数据到 writer，超时时间为 100 毫秒
    err := buf.CopyOnceTimeout(reader, writer, time.Millisecond*100)
    if err == buf.ErrNotTimeoutReader || err == buf.ErrReadTimeout {
        return writer.WriteMultiBuffer(buf.MultiBuffer{})
    }

    if err != nil {
        return err
    }

    return nil
}

// fetchInput 从会话中获取输入数据并发送到目标
func fetchInput(ctx context.Context, s *Session, output buf.Writer) {
    dest := session.OutboundFromContext(ctx).Target
    transferType := protocol.TransferTypeStream
    if dest.Network == net.Network_UDP {
        transferType = protocol.TransferTypePacket
    }
    s.transferType = transferType
    writer := NewWriter(s.ID, dest, output, transferType)
    defer s.Close()      // nolint: errcheck
    defer writer.Close() // nolint: errcheck

    newError("dispatching request to ", dest).WriteToLog(session.ExportIDToError(ctx))
    if err := writeFirstPayload(s.input, writer); err != nil {
        newError("failed to write first payload").Base(err).WriteToLog(session.ExportIDToError(ctx))
        writer.hasError = true
        common.Interrupt(s.input)
        return
    }
}
    # 如果从输入流复制到写入器时发生错误，则执行以下操作
    if err := buf.Copy(s.input, writer); err != nil:
        # 创建新的错误消息并将其基础错误设置为err，然后将其写入日志
        newError("failed to fetch all input").Base(err).WriteToLog(session.ExportIDToError(ctx))
        # 将写入器的hasError属性设置为true
        writer.hasError = true
        # 中断输入流
        common.Interrupt(s.input)
        # 返回
        return
// 检查当前客户端工作器是否正在关闭
func (m *ClientWorker) IsClosing() bool {
    sm := m.sessionManager
    // 如果策略规定最大连接数大于0，并且当前会话数量达到最大连接数，则返回true
    if m.strategy.MaxConnection > 0 && sm.Count() >= int(m.strategy.MaxConnection) {
        return true
    }
    // 否则返回false
    return false
}

// 检查当前客户端工作器是否已满
func (m *ClientWorker) IsFull() bool {
    // 如果正在关闭或已关闭，则返回true
    if m.IsClosing() || m.Closed() {
        return true
    }

    sm := m.sessionManager
    // 如果策略规定最大并发数大于0，并且当前会话数量达到最大并发数，则返回true
    if m.strategy.MaxConcurrency > 0 && sm.Size() >= int(m.strategy.MaxConcurrency) {
        return true
    }
    // 否则返回false
    return false
}

// 分发任务给客户端工作器
func (m *ClientWorker) Dispatch(ctx context.Context, link *transport.Link) bool {
    // 如果已满或已关闭，则返回false
    if m.IsFull() || m.Closed() {
        return false
    }

    sm := m.sessionManager
    // 分配一个会话
    s := sm.Allocate()
    if s == nil {
        return false
    }
    s.input = link.Reader
    s.output = link.Writer
    // 启动一个协程来处理输入数据
    go fetchInput(ctx, s, m.link.Writer)
    return true
}

// 处理状态为保持的情况
func (m *ClientWorker) handleStatueKeepAlive(meta *FrameMetadata, reader *buf.BufferedReader) error {
    // 如果元数据中包含数据选项，则将数据从reader复制到buf.Discard
    if meta.Option.Has(OptionData) {
        return buf.Copy(NewStreamReader(reader), buf.Discard)
    }
    // 否则返回nil
    return nil
}

// 处理状态为新的情况
func (m *ClientWorker) handleStatusNew(meta *FrameMetadata, reader *buf.BufferedReader) error {
    // 如果元数据中包含数据选项，则将数据从reader复制到buf.Discard
    if meta.Option.Has(OptionData) {
        return buf.Copy(NewStreamReader(reader), buf.Discard)
    }
    // 否则返回nil
    return nil
}

// 处理状态为保持的情况
func (m *ClientWorker) handleStatusKeep(meta *FrameMetadata, reader *buf.BufferedReader) error {
    // 如果元数据中不包含数据选项，则返回nil
    if !meta.Option.Has(OptionData) {
        return nil
    }

    // 从会话管理器中获取会话
    s, found := m.sessionManager.Get(meta.SessionID)
    if !found {
        // 通知远程对等端关闭此会话
        closingWriter := NewResponseWriter(meta.SessionID, m.link.Writer, protocol.TransferTypeStream)
        closingWriter.Close()

        // 将数据从reader复制到buf.Discard
        return buf.Copy(NewStreamReader(reader), buf.Discard)
    }

    // 从会话中获取一个新的reader，并将数据从reader复制到会话的输出
    rr := s.NewReader(reader)
    err := buf.Copy(rr, s.output)
    // 如果 err 不为空并且是写入错误
    if err != nil && buf.IsWriteError(err) {
        // 创建新的错误消息，记录关闭会话的信息，并将错误信息写入日志
        newError("failed to write to downstream. closing session ", s.ID).Base(err).WriteToLog()

        // 通知远程对等端关闭此会话
        closingWriter := NewResponseWriter(meta.SessionID, m.link.Writer, protocol.TransferTypeStream)
        closingWriter.Close()

        // 将剩余的数据从 rr 复制到 Discard，返回可能出现的错误
        drainErr := buf.Copy(rr, buf.Discard)
        // 中断输入流
        common.Interrupt(s.input)
        // 关闭会话
        s.Close()
        // 返回可能出现的错误
        return drainErr
    }

    // 返回错误
    return err
// 处理会话结束状态的方法
func (m *ClientWorker) handleStatusEnd(meta *FrameMetadata, reader *buf.BufferedReader) error {
    // 根据会话ID从会话管理器中获取会话
    if s, found := m.sessionManager.Get(meta.SessionID); found {
        // 如果元数据中包含错误选项，则中断输入和输出流
        if meta.Option.Has(OptionError) {
            common.Interrupt(s.input)
            common.Interrupt(s.output)
        }
        // 关闭会话
        s.Close()
    }
    // 如果元数据中包含数据选项，则将数据从reader复制到Discard中
    if meta.Option.Has(OptionData) {
        return buf.Copy(NewStreamReader(reader), buf.Discard)
    }
    // 否则返回空
    return nil
}

// 获取输出数据的方法
func (m *ClientWorker) fetchOutput() {
    // 延迟关闭done通道
    defer func() {
        common.Must(m.done.Close())
    }()

    // 创建一个reader对象，用于读取数据
    reader := &buf.BufferedReader{Reader: m.link.Reader}

    // 定义一个meta变量，用于存储帧元数据
    var meta FrameMetadata
    // 循环读取数据
    for {
        // 解析元数据
        err := meta.Unmarshal(reader)
        if err != nil {
            // 如果出现错误且不是EOF，则记录日志并退出循环
            if errors.Cause(err) != io.EOF {
                newError("failed to read metadata").Base(err).WriteToLog()
            }
            break
        }

        // 根据会话状态进行不同的处理
        switch meta.SessionStatus {
        case SessionStatusKeepAlive:
            err = m.handleStatueKeepAlive(&meta, reader)
        case SessionStatusEnd:
            err = m.handleStatusEnd(&meta, reader)
        case SessionStatusNew:
            err = m.handleStatusNew(&meta, reader)
        case SessionStatusKeep:
            err = m.handleStatusKeep(&meta, reader)
        default:
            // 如果会话状态未知，则记录日志并退出
            status := meta.SessionStatus
            newError("unknown status: ", status).AtError().WriteToLog()
            return
        }

        // 如果处理过程中出现错误，则记录日志并退出
        if err != nil {
            newError("failed to process data").Base(err).WriteToLog()
            return
        }
    }
}
```