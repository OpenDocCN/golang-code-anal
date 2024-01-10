# `v2ray-core\transport\pipe\impl.go`

```
package pipe

import (
    "errors" // 引入 errors 包，用于定义错误
    "io" // 引入 io 包，用于实现 I/O 操作
    "runtime" // 引入 runtime 包，用于访问 Go 运行时的信息
    "sync" // 引入 sync 包，用于实现同步操作
    "time" // 引入 time 包，用于处理时间相关操作

    "v2ray.com/core/common" // 引入 common 包
    "v2ray.com/core/common/buf" // 引入 buf 包，用于处理缓冲区
    "v2ray.com/core/common/signal" // 引入 signal 包，用于处理信号
    "v2ray.com/core/common/signal/done" // 引入 done 包，用于处理完成信号
)

type state byte // 定义状态类型

const (
    open state = iota // 定义状态常量 open
    closed // 定义状态常量 closed
    errord // 定义状态常量 errord
)

type pipeOption struct { // 定义管道选项结构
    limit           int32 // 最大缓冲区大小（以字节为单位）
    discardOverflow bool // 是否丢弃溢出数据
}

func (o *pipeOption) isFull(curSize int32) bool { // 判断缓冲区是否已满
    return o.limit >= 0 && curSize > o.limit
}

type pipe struct { // 定义管道结构
    sync.Mutex // 匿名嵌入 sync.Mutex，实现互斥锁
    data        buf.MultiBuffer // 缓冲区，存储多个数据块
    readSignal  *signal.Notifier // 读取信号通知器
    writeSignal *signal.Notifier // 写入信号通知器
    done        *done.Instance // 完成实例
    option      pipeOption // 管道选项
    state       state // 当前状态
}

var errBufferFull = errors.New("buffer full") // 定义缓冲区已满的错误
var errSlowDown = errors.New("slow down") // 定义减速错误

func (p *pipe) getState(forRead bool) error { // 获取管道状态
    switch p.state { // 根据当前状态进行判断
    case open: // 如果状态为 open
        if !forRead && p.option.isFull(p.data.Len()) { // 如果是写入操作且缓冲区已满
            return errBufferFull // 返回缓冲区已满的错误
        }
        return nil // 返回空错误
    case closed: // 如果状态为 closed
        if !forRead { // 如果是写入操作
            return io.ErrClosedPipe // 返回管道已关闭的错误
        }
        if !p.data.IsEmpty() { // 如果缓冲区不为空
            return nil // 返回空错误
        }
        return io.EOF // 返回文件结束错误
    case errord: // 如果状态为 errord
        return io.ErrClosedPipe // 返回管道已关闭的错误
    default: // 其他情况
        panic("impossible case") // 抛出异常
    }
}

func (p *pipe) readMultiBufferInternal() (buf.MultiBuffer, error) { // 读取多个数据块（内部方法）
    p.Lock() // 加锁
    defer p.Unlock() // 延迟解锁

    if err := p.getState(true); err != nil { // 获取管道状态，如果有错误
        return nil, err // 返回空数据块和错误
    }

    data := p.data // 将缓冲区数据赋值给 data
    p.data = nil // 清空缓冲区
    return data, nil // 返回数据块和空错误
}

func (p *pipe) ReadMultiBuffer() (buf.MultiBuffer, error) { // 读取多个数据块
    for { // 循环
        data, err := p.readMultiBufferInternal() // 调用内部方法读取数据块
        if data != nil || err != nil { // 如果数据块不为空或者有错误
            p.writeSignal.Signal() // 发送写入信号
            return data, err // 返回数据块和错误
        }

        select { // 选择
        case <-p.readSignal.Wait(): // 等待读取信号
        case <-p.done.Wait(): // 等待完成信号
        }
    }
}

func (p *pipe) ReadMultiBufferTimeout(d time.Duration) (buf.MultiBuffer, error) { // 读取多个数据块（带超时）
    timer := time.NewTimer(d) // 创建定时器
    defer timer.Stop() // 延迟停止定时器
    // 无限循环，用于读取数据
    for {
        // 调用 readMultiBufferInternal 方法读取数据
        data, err := p.readMultiBufferInternal()
        // 如果数据不为空或者出现错误，则发送信号并返回数据和错误
        if data != nil || err != nil {
            p.writeSignal.Signal()
            return data, err
        }

        // 使用 select 语句监听多个 channel 的事件
        select {
        // 等待读取信号
        case <-p.readSignal.Wait():
        // 等待结束信号
        case <-p.done.Wait():
        // 等待定时器超时信号
        case <-timer.C:
            return nil, buf.ErrReadTimeout
        }
    }
# 写入多个缓冲区到管道内部，使用管道的锁进行同步
func (p *pipe) writeMultiBufferInternal(mb buf.MultiBuffer) error {
    # 获取管道的锁
    p.Lock()
    # 在函数返回时释放管道的锁
    defer p.Unlock()

    # 获取管道的状态，如果有错误则返回
    if err := p.getState(false); err != nil {
        return err
    }

    # 如果管道数据为空，则直接将传入的多个缓冲区写入管道
    if p.data == nil {
        p.data = mb
        return nil
    }

    # 如果管道数据不为空，则合并传入的多个缓冲区和管道中的数据
    p.data, _ = buf.MergeMulti(p.data, mb)
    return errSlowDown
}

# 将多个缓冲区写入管道
func (p *pipe) WriteMultiBuffer(mb buf.MultiBuffer) error {
    # 如果传入的多个缓冲区为空，则直接返回
    if mb.IsEmpty() {
        return nil
    }

    # 循环写入多个缓冲区到管道
    for {
        # 调用写入多个缓冲区的内部方法
        err := p.writeMultiBufferInternal(mb)
        # 如果写入成功，则发送读取信号并返回
        if err == nil {
            p.readSignal.Signal()
            return nil
        }

        # 如果写入速度过慢，则发送读取信号并暂停当前 goroutine，让读取端有机会读取数据
        if err == errSlowDown {
            p.readSignal.Signal()
            runtime.Gosched()
            return nil
        }

        # 如果缓冲区已满且设置了丢弃溢出，则释放多个缓冲区并返回
        if err == errBufferFull && p.option.discardOverflow {
            buf.ReleaseMulti(mb)
            return nil
        }

        # 如果错误不是缓冲区已满，则释放多个缓冲区并发送读取信号，然后返回错误
        if err != errBufferFull {
            buf.ReleaseMulti(mb)
            p.readSignal.Signal()
            return err
        }

        # 通过 select 语句等待写入信号或者管道关闭信号
        select {
        case <-p.writeSignal.Wait():
        case <-p.done.Wait():
            return io.ErrClosedPipe
        }
    }
}

# 关闭管道
func (p *pipe) Close() error {
    # 获取管道的锁
    p.Lock()
    # 在函数返回时释放管道的锁
    defer p.Unlock()

    # 如果管道状态为关闭或错误，则直接返回
    if p.state == closed || p.state == errord {
        return nil
    }

    # 设置管道状态为关闭，并关闭管道的 done 信号
    p.state = closed
    common.Must(p.done.Close())
    return nil
}

# 中断管道操作，实现 common.Interruptible 接口
func (p *pipe) Interrupt() {
    # 获取管道的锁
    p.Lock()
    # 在函数返回时释放管道的锁
    defer p.Unlock()

    # 如果管道状态为关闭或错误，则直接返回
    if p.state == closed || p.state == errord {
        return
    }

    # 设置管道状态为错误，并释放管道中的数据，然后关闭管道的 done 信号
    p.state = errord

    if !p.data.IsEmpty() {
        buf.ReleaseMulti(p.data)
        p.data = nil
    }

    common.Must(p.done.Close())
}
```