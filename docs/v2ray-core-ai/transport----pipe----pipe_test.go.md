# `v2ray-core\transport\pipe\pipe_test.go`

```
package pipe_test

import (
    "errors"  // 导入 errors 包
    "io"  // 导入 io 包
    "testing"  // 导入 testing 包
    "time"  // 导入 time 包

    "github.com/google/go-cmp/cmp"  // 导入 github.com/google/go-cmp/cmp 包
    "golang.org/x/sync/errgroup"  // 导入 golang.org/x/sync/errgroup 包

    "v2ray.com/core/common"  // 导入 v2ray.com/core/common 包
    "v2ray.com/core/common/buf"  // 导入 v2ray.com/core/common/buf 包
    . "v2ray.com/core/transport/pipe"  // 导入 v2ray.com/core/transport/pipe 包，并将其导入的函数和类型放在当前包的命名空间中
)

func TestPipeReadWrite(t *testing.T) {
    pReader, pWriter := New(WithSizeLimit(1024))  // 创建一个大小限制为 1024 的管道读写器

    b := buf.New()  // 创建一个新的缓冲区
    b.WriteString("abcd")  // 向缓冲区写入字符串 "abcd"
    common.Must(pWriter.WriteMultiBuffer(buf.MultiBuffer{b}))  // 使用 pWriter 将缓冲区中的数据写入管道

    b2 := buf.New()  // 创建另一个新的缓冲区
    b2.WriteString("efg")  // 向缓冲区写入字符串 "efg"
    common.Must(pWriter.WriteMultiBuffer(buf.MultiBuffer{b2}))  // 使用 pWriter 将缓冲区中的数据写入管道

    rb, err := pReader.ReadMultiBuffer()  // 从管道中读取数据到缓冲区 rb
    common.Must(err)  // 检查是否有错误发生
    if r := cmp.Diff(rb.String(), "abcdefg"); r != "" {  // 比较读取的数据和预期的数据是否一致
        t.Error(r)  // 如果不一致，则输出错误信息
    }
}

func TestPipeInterrupt(t *testing.T) {
    pReader, pWriter := New(WithSizeLimit(1024))  // 创建一个大小限制为 1024 的管道读写器
    payload := []byte{'a', 'b', 'c', 'd'}  // 创建一个字节数组
    b := buf.New()  // 创建一个新的缓冲区
    b.Write(payload)  // 向缓冲区写入字节数组
    common.Must(pWriter.WriteMultiBuffer(buf.MultiBuffer{b}))  // 使用 pWriter 将缓冲区中的数据写入管道
    pWriter.Interrupt()  // 中断写入操作

    rb, err := pReader.ReadMultiBuffer()  // 从管道中读取数据到缓冲区 rb
    if err != io.ErrClosedPipe {  // 检查是否返回了预期的错误
        t.Fatal("expect io.ErrClosePipe, but got ", err)  // 如果不是预期的错误，则输出错误信息
    }
    if !rb.IsEmpty() {  // 检查读取的缓冲区是否为空
        t.Fatal("expect empty buffer, but got ", rb.Len())  // 如果不是空的，则输出错误信息
    }
}

func TestPipeClose(t *testing.T) {
    pReader, pWriter := New(WithSizeLimit(1024))  // 创建一个大小限制为 1024 的管道读写器
    payload := []byte{'a', 'b', 'c', 'd'}  // 创建一个字节数组
    b := buf.New()  // 创建一个新的缓冲区
    common.Must2(b.Write(payload))  // 向缓冲区写入字节数组
    common.Must(pWriter.WriteMultiBuffer(buf.MultiBuffer{b}))  // 使用 pWriter 将缓冲区中的数据写入管道
    common.Must(pWriter.Close())  // 关闭写入操作

    rb, err := pReader.ReadMultiBuffer()  // 从管道中读取数据到缓冲区 rb
    common.Must(err)  // 检查是否有错误发生
    if rb.String() != string(payload) {  // 检查读取的数据是否和预期的数据一致
        t.Fatal("expect content ", string(payload), " but actually ", rb.String())  // 如果不一致，则输出错误信息
    }

    rb, err = pReader.ReadMultiBuffer()  // 再次从管道中读取数据到缓冲区 rb
    if err != io.EOF {  // 检查是否返回了预期的错误
        t.Fatal("expected EOF, but got ", err)  // 如果不是预期的错误，则输出错误信息
    }
    if !rb.IsEmpty() {  // 检查读取的缓冲区是否为空
        t.Fatal("expect empty buffer, but got ", rb.String())  // 如果不是空的，则输出错误信息
    }
}

func TestPipeLimitZero(t *testing.T) {
    pReader, pWriter := New(WithSizeLimit(0))  // 创建一个大小限制为 0 的管道读写器
    bb := buf.New()  // 创建一个新的缓冲区
    # 调用 common.Must2 函数，将字节流写入 bb 中
    common.Must2(bb.Write([]byte{'a', 'b'}))
    # 调用 common.Must 函数，将多个字节流写入 pWriter 中
    common.Must(pWriter.WriteMultiBuffer(buf.MultiBuffer{bb}))

    # 创建错误组
    var errg errgroup.Group
    # 向错误组中添加一个任务
    errg.Go(func() error {
        # 创建一个新的字节流
        b := buf.New()
        # 将字节流写入 b 中
        b.Write([]byte{'c', 'd'})
        # 将多个字节流写入 pWriter 中
        return pWriter.WriteMultiBuffer(buf.MultiBuffer{b})
    })
    # 向错误组中添加一个任务
    errg.Go(func() error {
        # 休眠一秒
        time.Sleep(time.Second)

        # 创建一个多字节流容器
        var container buf.MultiBufferContainer
        # 从 pReader 中复制数据到容器中
        if err := buf.Copy(pReader, &container); err != nil {
            return err
        }

        # 比较容器中的数据和字符串 "abcd" 是否相同
        if r := cmp.Diff(container.String(), "abcd"); r != "" {
            return errors.New(r)
        }
        return nil
    })
    # 向错误组中添加一个任务
    errg.Go(func() error {
        # 休眠两秒
        time.Sleep(time.Second * 2)
        # 关闭 pWriter
        return pWriter.Close()
    })
    # 等待错误组中的所有任务完成，如果有错误则输出错误信息
    if err := errg.Wait(); err != nil {
        t.Error(err)
    }
func TestPipeWriteMultiThread(t *testing.T) {
    // 创建管道读写器
    pReader, pWriter := New(WithSizeLimit(0))

    // 创建错误组
    var errg errgroup.Group
    // 循环10次
    for i := 0; i < 10; i++ {
        // 启动并发任务
        errg.Go(func() error {
            // 创建缓冲区
            b := buf.New()
            // 写入数据到缓冲区
            b.WriteString("abcd")
            // 将缓冲区数据写入管道
            return pWriter.WriteMultiBuffer(buf.MultiBuffer{b})
        })
    }
    // 等待100毫秒
    time.Sleep(time.Millisecond * 100)
    // 关闭管道写入器
    pWriter.Close()
    // 等待所有并发任务完成
    errg.Wait()

    // 从管道读取多个缓冲区数据
    b, err := pReader.ReadMultiBuffer()
    // 必须处理错误
    common.Must(err)
    // 比较读取的数据和预期数据是否一致
    if r := cmp.Diff(b[0].Bytes(), []byte{'a', 'b', 'c', 'd'}); r != "" {
        t.Error(r)
    }
}

func TestInterfaces(t *testing.T) {
    // 接口断言，判断Reader是否实现了buf.Reader接口
    _ = (buf.Reader)(new(Reader))
    // 接口断言，判断Reader是否实现了buf.TimeoutReader接口
    _ = (buf.TimeoutReader)(new(Reader))

    // 接口断言，判断Reader是否实现了common.Interruptible接口
    _ = (common.Interruptible)(new(Reader))
    // 接口断言，判断Writer是否实现了common.Interruptible接口
    _ = (common.Interruptible)(new(Writer))
    // 接口断言，判断Writer是否实现了common.Closable接口
    _ = (common.Closable)(new(Writer))
}

func BenchmarkPipeReadWrite(b *testing.B) {
    // 创建管道读写器
    reader, writer := New(WithoutSizeLimit())
    // 创建缓冲区
    a := buf.New()
    // 扩展缓冲区大小
    a.Extend(buf.Size)
    // 创建多个缓冲区
    c := buf.MultiBuffer{a}

    // 重置计时器
    b.ResetTimer()
    // 循环执行基准测试
    for i := 0; i < b.N; i++ {
        // 写入多个缓冲区数据到管道
        common.Must(writer.WriteMultiBuffer(c))
        // 从管道读取多个缓冲区数据
        d, err := reader.ReadMultiBuffer()
        // 必须处理错误
        common.Must(err)
        // 更新待写入的缓冲区数据
        c = d
    }
}
```