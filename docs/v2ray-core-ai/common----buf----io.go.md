# `v2ray-core\common\buf\io.go`

```
package buf

import (
    "io"
    "net"
    "os"
    "syscall"
    "time"
)

// Reader extends io.Reader with MultiBuffer.
type Reader interface {
    // ReadMultiBuffer reads content from underlying reader, and put it into a MultiBuffer.
    ReadMultiBuffer() (MultiBuffer, error)
}

// ErrReadTimeout is an error that happens with IO timeout.
var ErrReadTimeout = newError("IO timeout")

// TimeoutReader is a reader that returns error if Read() operation takes longer than the given timeout.
type TimeoutReader interface {
    ReadMultiBufferTimeout(time.Duration) (MultiBuffer, error)
}

// Writer extends io.Writer with MultiBuffer.
type Writer interface {
    // WriteMultiBuffer writes a MultiBuffer into underlying writer.
    WriteMultiBuffer(MultiBuffer) error
}

// WriteAllBytes ensures all bytes are written into the given writer.
func WriteAllBytes(writer io.Writer, payload []byte) error {
    for len(payload) > 0 {
        n, err := writer.Write(payload)
        if err != nil {
            return err
        }
        payload = payload[n:]
    }
    return nil
}

func isPacketReader(reader io.Reader) bool {
    // Check if the reader is a net.PacketConn
    _, ok := reader.(net.PacketConn)
    return ok
}

// NewReader creates a new Reader.
// The Reader instance doesn't take the ownership of reader.
func NewReader(reader io.Reader) Reader {
    if mr, ok := reader.(Reader); ok {
        return mr
    }

    if isPacketReader(reader) {
        // If the reader is a net.PacketConn, create a new PacketReader
        return &PacketReader{
            Reader: reader,
        }
    }

    _, isFile := reader.(*os.File)
    if !isFile && useReadv {
        if sc, ok := reader.(syscall.Conn); ok {
            // If the reader is a syscall.Conn and useReadv is true, create a new ReadVReader
            rawConn, err := sc.SyscallConn()
            if err != nil {
                newError("failed to get sysconn").Base(err).WriteToLog()
            } else {
                return NewReadVReader(reader, rawConn)
            }
        }
    }

    // If none of the above conditions are met, create a new SingleReader
    return &SingleReader{
        Reader: reader,
    }
}

// NewPacketReader creates a new PacketReader based on the given reader.
# 创建一个新的数据包读取器，接受一个io.Reader类型的参数，并返回一个Reader接口
func NewPacketReader(reader io.Reader) Reader {
    # 如果reader已经实现了Reader接口，则直接返回reader
    if mr, ok := reader.(Reader); ok {
        return mr
    }

    # 如果reader没有实现Reader接口，则创建一个PacketReader对象并返回
    return &PacketReader{
        Reader: reader,
    }
}

# 判断是否为数据包写入器
func isPacketWriter(writer io.Writer) bool {
    # 如果writer实现了net.PacketConn接口，则返回true
    if _, ok := writer.(net.PacketConn); ok {
        return true
    }

    # 如果writer没有实现syscall.Conn接口，则可能不是TCP连接
    if _, ok := writer.(syscall.Conn); !ok {
        return true
    }
    return false
}

# 创建一个新的数据包写入器
func NewWriter(writer io.Writer) Writer {
    # 如果writer已经实现了Writer接口，则直接返回writer
    if mw, ok := writer.(Writer); ok {
        return mw
    }

    # 如果writer没有实现Writer接口，但是满足isPacketWriter条件，则创建一个SequentialWriter对象并返回
    if isPacketWriter(writer) {
        return &SequentialWriter{
            Writer: writer,
        }
    }

    # 如果writer既不是Writer接口，也不满足isPacketWriter条件，则创建一个BufferToBytesWriter对象并返回
    return &BufferToBytesWriter{
        Writer: writer,
    }
}
```