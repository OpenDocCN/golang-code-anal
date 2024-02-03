# `v2ray-core\common\mux\server.go`

```go
package mux

import (
    "context"
    "io"

    "v2ray.com/core"
    "v2ray.com/core/common"
    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/errors"
    "v2ray.com/core/common/log"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol"
    "v2ray.com/core/common/session"
    "v2ray.com/core/features/routing"
    "v2ray.com/core/transport"
    "v2ray.com/core/transport/pipe"
)

type Server struct {
    dispatcher routing.Dispatcher
}

// NewServer creates a new mux.Server.
func NewServer(ctx context.Context) *Server {
    s := &Server{}
    core.RequireFeatures(ctx, func(d routing.Dispatcher) {
        s.dispatcher = d
    })
    return s
}

// Type implements common.HasType.
func (s *Server) Type() interface{} {
    return s.dispatcher.Type()
}

// Dispatch implements routing.Dispatcher
func (s *Server) Dispatch(ctx context.Context, dest net.Destination) (*transport.Link, error) {
    if dest.Address != muxCoolAddress {
        return s.dispatcher.Dispatch(ctx, dest)
    }

    opts := pipe.OptionsFromContext(ctx)
    uplinkReader, uplinkWriter := pipe.New(opts...)
    downlinkReader, downlinkWriter := pipe.New(opts...)

    _, err := NewServerWorker(ctx, s.dispatcher, &transport.Link{
        Reader: uplinkReader,
        Writer: downlinkWriter,
    })
    if err != nil {
        return nil, err
    }

    return &transport.Link{Reader: downlinkReader, Writer: uplinkWriter}, nil
}

// Start implements common.Runnable.
func (s *Server) Start() error {
    return nil
}

// Close implements common.Closable.
func (s *Server) Close() error {
    return nil
}

type ServerWorker struct {
    dispatcher     routing.Dispatcher
    link           *transport.Link
    sessionManager *SessionManager
}

func NewServerWorker(ctx context.Context, d routing.Dispatcher, link *transport.Link) (*ServerWorker, error) {
    worker := &ServerWorker{
        dispatcher:     d,
        link:           link,
        sessionManager: NewSessionManager(),
    }
    # 调用 worker 对象的 run 方法，并传入上下文参数 ctx
    go worker.run(ctx)
    # 返回 worker 对象和空指针
    return worker, nil
}

// 处理会话的函数，接收上下文、会话和输出缓冲区作为参数
func handle(ctx context.Context, s *Session, output buf.Writer) {
    // 创建响应写入器对象
    writer := NewResponseWriter(s.ID, output, s.transferType)
    // 将输入流内容拷贝到响应写入器中
    if err := buf.Copy(s.input, writer); err != nil {
        // 如果发生错误，记录错误信息并设置标志
        newError("session ", s.ID, " ends.").Base(err).WriteToLog(session.ExportIDToError(ctx))
        writer.hasError = true
    }

    // 关闭响应写入器
    writer.Close()
    // 关闭会话
    s.Close()
}

// 返回活动连接数的函数
func (w *ServerWorker) ActiveConnections() uint32 {
    return uint32(w.sessionManager.Size())
}

// 返回服务器是否已关闭的函数
func (w *ServerWorker) Closed() bool {
    return w.sessionManager.Closed()
}

// 处理状态保持活动的函数，接收帧元数据和缓冲读取器作为参数
func (w *ServerWorker) handleStatusKeepAlive(meta *FrameMetadata, reader *buf.BufferedReader) error {
    // 如果帧元数据中包含数据选项，则将读取器内容拷贝到丢弃缓冲区中
    if meta.Option.Has(OptionData) {
        return buf.Copy(NewStreamReader(reader), buf.Discard)
    }
    return nil
}

// 处理新状态的函数，接收上下文、帧元数据和缓冲读取器作为参数
func (w *ServerWorker) handleStatusNew(ctx context.Context, meta *FrameMetadata, reader *buf.BufferedReader) error {
    // 记录接收到的请求信息
    newError("received request for ", meta.Target).WriteToLog(session.ExportIDToError(ctx))
    {
        // 创建访问消息对象
        msg := &log.AccessMessage{
            To:     meta.Target,
            Status: log.AccessAccepted,
            Reason: "",
        }
        // 如果上下文中包含入站信息，并且来源有效，则设置消息的来源和电子邮件
        if inbound := session.InboundFromContext(ctx); inbound != nil && inbound.Source.IsValid() {
            msg.From = inbound.Source
            msg.Email = inbound.User.Email
        }
        // 将访问消息添加到上下文中
        ctx = log.ContextWithAccessMessage(ctx, msg)
    }
    // 调度请求并获取链接
    link, err := w.dispatcher.Dispatch(ctx, meta.Target)
    if err != nil {
        // 如果发生错误，并且帧元数据中包含数据选项，则将读取器内容拷贝到丢弃缓冲区中
        if meta.Option.Has(OptionData) {
            buf.Copy(NewStreamReader(reader), buf.Discard)
        }
        // 返回调度请求失败的错误信息
        return newError("failed to dispatch request.").Base(err)
    }
    // 创建会话对象
    s := &Session{
        input:        link.Reader,
        output:       link.Writer,
        parent:       w.sessionManager,
        ID:           meta.SessionID,
        transferType: protocol.TransferTypeStream,
    }
    // 如果目标网络是 UDP，则设置传输类型为数据包
    if meta.Target.Network == net.Network_UDP {
        s.transferType = protocol.TransferTypePacket
    }
    // 将会话添加到会话管理器中
    w.sessionManager.Add(s)
}
    # 调用 handle 函数处理上下文、状态和写入链接的数据
    go handle(ctx, s, w.link.Writer)
    # 如果元数据选项中不包含数据选项，则返回空
    if !meta.Option.Has(OptionData) {
        return nil
    }
    # 从状态中获取一个新的读取器
    rr := s.NewReader(reader)
    # 如果从 rr 读取数据并将其写入 s.output 时出现错误，则执行以下操作
    if err := buf.Copy(rr, s.output); err != nil {
        # 将从 rr 读取的数据写入到 buf.Discard 中
        buf.Copy(rr, buf.Discard)
        # 中断输入流
        common.Interrupt(s.input)
        # 关闭状态
        return s.Close()
    }
    # 返回空
    return nil
}
// 处理保持会话状态的方法
func (w *ServerWorker) handleStatusKeep(meta *FrameMetadata, reader *buf.BufferedReader) error {
    // 如果元数据中没有数据选项，则返回空
    if !meta.Option.Has(OptionData) {
        return nil
    }

    // 从会话管理器中获取会话
    s, found := w.sessionManager.Get(meta.SessionID)
    if !found {
        // 通知远程对等方关闭此会话
        closingWriter := NewResponseWriter(meta.SessionID, w.link.Writer, protocol.TransferTypeStream)
        closingWriter.Close()

        return buf.Copy(NewStreamReader(reader), buf.Discard)
    }

    // 从会话中获取读取器
    rr := s.NewReader(reader)
    // 将数据从读取器复制到输出流
    err := buf.Copy(rr, s.output)

    // 如果发生写入错误
    if err != nil && buf.IsWriteError(err) {
        newError("failed to write to downstream writer. closing session ", s.ID).Base(err).WriteToLog()

        // 通知远程对等方关闭此会话
        closingWriter := NewResponseWriter(meta.SessionID, w.link.Writer, protocol.TransferTypeStream)
        closingWriter.Close()

        // 将剩余数据从读取器复制到丢弃流
        drainErr := buf.Copy(rr, buf.Discard)
        // 中断输入流
        common.Interrupt(s.input)
        // 关闭会话
        s.Close()
        return drainErr
    }

    return err
}

// 处理会话结束状态的方法
func (w *ServerWorker) handleStatusEnd(meta *FrameMetadata, reader *buf.BufferedReader) error {
    // 如果从会话管理器中获取会话成功
    if s, found := w.sessionManager.Get(meta.SessionID); found {
        // 如果元数据中包含错误选项
        if meta.Option.Has(OptionError) {
            // 中断输入流和输出流
            common.Interrupt(s.input)
            common.Interrupt(s.output)
        }
        // 关闭会话
        s.Close()
    }
    // 如果元数据中包含数据选项，则将数据从读取器复制到丢弃流
    if meta.Option.Has(OptionData) {
        return buf.Copy(NewStreamReader(reader), buf.Discard)
    }
    return nil
}

// 处理帧的方法
func (w *ServerWorker) handleFrame(ctx context.Context, reader *buf.BufferedReader) error {
    var meta FrameMetadata
    // 反序列化元数据
    err := meta.Unmarshal(reader)
    if err != nil {
        return newError("failed to read metadata").Base(err)
    }

    switch meta.SessionStatus {
    case SessionStatusKeepAlive:
        err = w.handleStatusKeepAlive(&meta, reader)
    case SessionStatusEnd:
        err = w.handleStatusEnd(&meta, reader)
    case SessionStatusNew:
        err = w.handleStatusNew(ctx, &meta, reader)
    # 根据会话状态执行不同的操作
    case SessionStatusKeep:
        # 处理保持状态的会话
        err = w.handleStatusKeep(&meta, reader)
    # 如果会话状态不是保持状态
    default:
        # 获取会话状态
        status := meta.SessionStatus
        # 返回一个新的错误，指示未知的会话状态
        return newError("unknown status: ", status).AtError()
    }

    # 如果发生错误
    if err != nil:
        # 返回一个新的错误，指示数据处理失败，并将原始错误作为基础错误
        return newError("failed to process data").Base(err)
    # 如果没有发生错误
    return nil
// 在 ServerWorker 结构体的 run 方法中，获取输入流和读取器
func (w *ServerWorker) run(ctx context.Context) {
    input := w.link.Reader
    reader := &buf.BufferedReader{Reader: input}

    // 延迟关闭会话管理器，忽略错误检查
    defer w.sessionManager.Close() // nolint: errcheck

    // 无限循环，监听上下文的取消信号
    for {
        select {
        case <-ctx.Done():
            return
        default:
            // 处理帧数据，返回错误
            err := w.handleFrame(ctx, reader)
            if err != nil {
                // 如果错误不是 EOF，则记录日志并中断输入流
                if errors.Cause(err) != io.EOF {
                    newError("unexpected EOF").Base(err).WriteToLog(session.ExportIDToError(ctx))
                    common.Interrupt(input)
                }
                return
            }
        }
    }
}
```