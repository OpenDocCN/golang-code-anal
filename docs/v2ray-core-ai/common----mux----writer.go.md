# `v2ray-core\common\mux\writer.go`

```go
package mux

import (
    "v2ray.com/core/common" // 导入 common 包
    "v2ray.com/core/common/buf" // 导入 buf 包
    "v2ray.com/core/common/net" // 导入 net 包
    "v2ray.com/core/common/protocol" // 导入 protocol 包
    "v2ray.com/core/common/serial" // 导入 serial 包
)

type Writer struct { // 定义 Writer 结构体
    dest         net.Destination // 目标地址
    writer       buf.Writer // 写入缓冲区
    id           uint16 // ID
    followup     bool // 是否跟进
    hasError     bool // 是否有错误
    transferType protocol.TransferType // 传输类型
}

func NewWriter(id uint16, dest net.Destination, writer buf.Writer, transferType protocol.TransferType) *Writer { // 创建新的 Writer 对象
    return &Writer{ // 返回 Writer 对象
        id:           id, // 设置 ID
        dest:         dest, // 设置目标地址
        writer:       writer, // 设置写入缓冲区
        followup:     false, // 设置为不跟进
        transferType: transferType, // 设置传输类型
    }
}

func NewResponseWriter(id uint16, writer buf.Writer, transferType protocol.TransferType) *Writer { // 创建新的响应 Writer 对象
    return &Writer{ // 返回 Writer 对象
        id:           id, // 设置 ID
        writer:       writer, // 设置写入缓冲区
        followup:     true, // 设置为跟进
        transferType: transferType, // 设置传输类型
    }
}

func (w *Writer) getNextFrameMeta() FrameMetadata { // 获取下一个帧的元数据
    meta := FrameMetadata{ // 创建 FrameMetadata 对象
        SessionID: w.id, // 设置会话 ID
        Target:    w.dest, // 设置目标地址
    }

    if w.followup { // 如果是跟进
        meta.SessionStatus = SessionStatusKeep // 设置会话状态为保持
    } else {
        w.followup = true // 设置为跟进
        meta.SessionStatus = SessionStatusNew // 设置会话状态为新建
    }

    return meta // 返回元数据
}

func (w *Writer) writeMetaOnly() error { // 仅写入元数据
    meta := w.getNextFrameMeta() // 获取下一个帧的元数据
    b := buf.New() // 创建新的缓冲区
    if err := meta.WriteTo(b); err != nil { // 如果写入元数据到缓冲区出错
        return err // 返回错误
    }
    return w.writer.WriteMultiBuffer(buf.MultiBuffer{b}) // 写入多个缓冲区到写入缓冲区
}

func writeMetaWithFrame(writer buf.Writer, meta FrameMetadata, data buf.MultiBuffer) error { // 使用帧和元数据写入
    frame := buf.New() // 创建新的缓冲区
    if err := meta.WriteTo(frame); err != nil { // 如果写入元数据到缓冲区出错
        return err // 返回错误
    }
    if _, err := serial.WriteUint16(frame, uint16(data.Len())); err != nil { // 如果写入数据长度到缓冲区出错
        return err // 返回错误
    }

    mb2 := make(buf.MultiBuffer, 0, len(data)+1) // 创建新的多缓冲区
    mb2 = append(mb2, frame) // 添加帧到多缓冲区
    mb2 = append(mb2, data...) // 添加数据到多缓冲区
    return writer.WriteMultiBuffer(mb2) // 写入多个缓冲区到写入缓冲区
}

func (w *Writer) writeData(mb buf.MultiBuffer) error { // 写入数据
    meta := w.getNextFrameMeta() // 获取下一个帧的元数据
    # 设置选项数据到元数据对象
    meta.Option.Set(OptionData)
    # 调用函数将元数据和帧数据写入到写入器中，并返回结果
    return writeMetaWithFrame(w.writer, meta, mb)
// WriteMultiBuffer 实现了 buf.Writer 接口
func (w *Writer) WriteMultiBuffer(mb buf.MultiBuffer) error {
    // 延迟释放多重缓冲区
    defer buf.ReleaseMulti(mb)

    // 如果多重缓冲区为空，则只写入元数据
    if mb.IsEmpty() {
        return w.writeMetaOnly()
    }

    // 循环处理多重缓冲区直到为空
    for !mb.IsEmpty() {
        var chunk buf.MultiBuffer
        // 如果传输类型为流，则按照指定大小分割多重缓冲区
        if w.transferType == protocol.TransferTypeStream {
            mb, chunk = buf.SplitSize(mb, 8*1024)
        } else {
            // 否则，按照首个缓冲区分割多重缓冲区
            mb2, b := buf.SplitFirst(mb)
            mb = mb2
            chunk = buf.MultiBuffer{b}
        }
        // 写入数据块
        if err := w.writeData(chunk); err != nil {
            return err
        }
    }

    return nil
}

// Close 实现了 common.Closable 接口
func (w *Writer) Close() error {
    // 构建帧元数据
    meta := FrameMetadata{
        SessionID:     w.id,
        SessionStatus: SessionStatusEnd,
    }
    // 如果存在错误标记，则设置选项为错误
    if w.hasError {
        meta.Option.Set(OptionError)
    }

    // 创建新的缓冲区用于存储帧元数据
    frame := buf.New()
    // 必须将帧元数据写入缓冲区
    common.Must(meta.WriteTo(frame))

    // 将帧元数据写入底层写入器，忽略错误检查
    w.writer.WriteMultiBuffer(buf.MultiBuffer{frame}) // nolint: errcheck
    return nil
}
```