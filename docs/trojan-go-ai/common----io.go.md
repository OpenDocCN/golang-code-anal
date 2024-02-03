# `trojan-go\common\io.go`

```go
package common

import (
    "io"
    "net"
    "sync"

    "github.com/p4gefau1t/trojan-go/log"
)

type RewindReader struct {
    mu         sync.Mutex  // 互斥锁，用于保护共享资源
    rawReader  io.Reader    // 原始的读取器
    buf        []byte       // 缓冲区
    bufReadIdx int          // 缓冲区读取索引
    rewound    bool         // 标记是否已经回溯
    buffering  bool         // 标记是否正在缓冲
    bufferSize int          // 缓冲区大小
}

func (r *RewindReader) Read(p []byte) (int, error) {
    r.mu.Lock()  // 加锁
    defer r.mu.Unlock()  // 延迟解锁

    if r.rewound {  // 如果已经回溯
        if len(r.buf) > r.bufReadIdx {  // 如果缓冲区还有数据
            n := copy(p, r.buf[r.bufReadIdx:])  // 将缓冲区的数据拷贝到 p 中
            r.bufReadIdx += n  // 更新缓冲区读取索引
            return n, nil  // 返回拷贝的字节数和空错误
        }
        r.rewound = false  // 所有缓冲内容已经读取完毕
    }
    n, err := r.rawReader.Read(p)  // 从原始读取器中读取数据
    if r.buffering {  // 如果正在缓冲
        r.buf = append(r.buf, p[:n]...)  // 将读取的数据追加到缓冲区
        if len(r.buf) > r.bufferSize*2 {  // 如果缓冲区大小超过两倍的缓冲区大小
            log.Debug("read too many bytes!")  // 输出调试信息
        }
    }
    return n, err  // 返回读取的字节数和可能的错误
}

func (r *RewindReader) ReadByte() (byte, error) {
    buf := [1]byte{}  // 创建一个长度为 1 的字节数组
    _, err := r.Read(buf[:])  // 读取一个字节到 buf 中
    return buf[0], err  // 返回读取的字节和可能的错误
}

func (r *RewindReader) Discard(n int) (int, error) {
    buf := [128]byte{}  // 创建一个长度为 128 的字节数组
    if n < 128 {  // 如果需要丢弃的字节数小于 128
        return r.Read(buf[:n])  // 直接从缓冲区中读取 n 个字节
    }
    for discarded := 0; discarded+128 < n; discarded += 128 {  // 循环直到丢弃的字节数大于等于 n
        _, err := r.Read(buf[:])  // 从缓冲区中读取 128 个字节
        if err != nil {  // 如果出现错误
            return discarded, err  // 返回已经丢弃的字节数和错误
        }
    }
    if rest := n % 128; rest != 0 {  // 如果还有剩余的字节数
        return r.Read(buf[:rest])  // 从缓冲区中读取剩余的字节数
    }
    return n, nil  // 返回需要丢弃的字节数和空错误
}

func (r *RewindReader) Rewind() {
    r.mu.Lock()  // 加锁
    if r.bufferSize == 0 {  // 如果缓冲区大小为 0
        panic("no buffer")  // 抛出异常
    }
    r.rewound = true  // 标记已经回溯
    r.bufReadIdx = 0  // 重置缓冲区读取索引
    r.mu.Unlock()  // 解锁
}

func (r *RewindReader) StopBuffering() {
    r.mu.Lock()  // 加锁
    r.buffering = false  // 停止缓冲
    r.mu.Unlock()  // 解锁
}

func (r *RewindReader) SetBufferSize(size int) {
    r.mu.Lock()  // 加锁
    if size == 0 {  // 如果缓冲区大小为 0
        if !r.buffering {  // 如果不在缓冲
            panic("reader is disabled")  // 抛出异常
        }
        r.buffering = false  // 停止缓冲
        r.buf = nil  // 清空缓冲区
        r.bufReadIdx = 0  // 重置缓冲区读取索引
        r.bufferSize = 0  // 缓冲区大小设为 0
    } else {
        # 如果条件不满足，则执行以下代码块
        if r.buffering {
            # 如果reader正在缓冲，则抛出panic
            panic("reader is buffering")
        }
        # 将reader的缓冲状态设置为true
        r.buffering = true
        # 重置缓冲读取索引为0
        r.bufReadIdx = 0
        # 设置缓冲区大小为size
        r.bufferSize = size
        # 创建一个切片，长度为0，容量为size，作为缓冲区
        r.buf = make([]byte, 0, size)
    }
    # 释放互斥锁
    r.mu.Unlock()
# 定义一个结构体 RewindConn，包含 net.Conn 类型的字段和 *RewindReader 类型的字段
type RewindConn struct {
    net.Conn
    *RewindReader
}

# 为 RewindConn 结构体定义 Read 方法，用于读取数据
func (c *RewindConn) Read(p []byte) (int, error) {
    return c.RewindReader.Read(p)
}

# 定义一个函数 NewRewindConn，用于创建 RewindConn 结构体实例
func NewRewindConn(conn net.Conn) *RewindConn {
    return &RewindConn{
        Conn: conn,
        RewindReader: &RewindReader{
            rawReader: conn,
        },
    }
}

# 定义一个结构体 StickyWriter，包含 io.Writer 类型的字段和 []byte 类型的字段
type StickyWriter struct {
    rawWriter   io.Writer
    writeBuffer []byte
    MaxBuffered int
}

# 为 StickyWriter 结构体定义 Write 方法，用于写入数据
func (w *StickyWriter) Write(p []byte) (int, error) {
    # 如果最大缓冲大于0，则将数据追加到缓冲区中
    if w.MaxBuffered > 0 {
        w.MaxBuffered--
        w.writeBuffer = append(w.writeBuffer, p...)
        # 如果最大缓冲不为0，则返回写入的数据长度和nil
        if w.MaxBuffered != 0 {
            return len(p), nil
        }
        w.MaxBuffered = 0
        # 将缓冲区中的数据写入到原始写入器中
        _, err := w.rawWriter.Write(w.writeBuffer)
        w.writeBuffer = nil
        return len(p), err
    }
    # 如果最大缓冲为0，则直接将数据写入到原始写入器中
    return w.rawWriter.Write(p)
}
```