# v2ray-core源码解析 75

# `transport/pipe/pipe_test.go`

这段代码是一个 Go 语言编写的测试套件，用于测试 v2ray.com 项目中的 Pipe 组件。具体来说，这段代码以下面几个主要部分：

1. 导入所需的第三方库：
	* `errors`：用于处理测试中可能出现的错误；
	* `io`：用于测试文件 I/O 操作；
	* `testing`：用于测试框架；
	* `time`：用于测试时间相关的操作；
	* `github.com/google/go-cmp/cmp`：用于比较测试结果；
	* `golang.org/x/sync/errgroup`：用于并行测试中可能出现的错误。
2. 定义测试函数：
	* `Run` 函数：设置测试的延时时间，并开始一个新测试；
	* `Test` 函数：测试 `test_pipe_client`；
	* `Fail` 函数：测试 `test_pipe_client_err` 和 `test_pipe_client_timeout`；
	* `Shutdown` 函数：关闭服务器，用于测试在客户端关闭时服务器是否能正常關閉。
3. 设置测试的配置：
	* `CreateServer` 函数：设置服务器端的一个实例；
	* `CreateClient` 函数：设置客户端的一个实例；
	* `Server` 和 `Client` 变量：分别绑定服务端和客户端的实例。

这段代码的主要目的是提供一个测试框架，用于测试 Pipe 组件在不同场景下的行为。通过调用 `Run`、`Test`、`Fail` 和 `Shutdown` 函数，可以很方便地进行测试，并输出测试结果。


```go
package pipe_test

import (
	"errors"
	"io"
	"testing"
	"time"

	"github.com/google/go-cmp/cmp"
	"golang.org/x/sync/errgroup"

	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	. "v2ray.com/core/transport/pipe"
)

```

该代码的主要目的是测试一个名为 `TestPipeReadWrite` 的函数。该函数使用 `buf` 包从指定大小的文件中读取和写入数据。

具体来说，该函数首先创建了一个 `pReader` 和一个 `pWriter` 变量。`pReader` 变量是一个 pipe reader，它可以从 `test.T` 类型的临时变量 `t` 接收数据。`pWriter` 变量是一个 pipe writer，它可以向 `t` 发送数据。

接下来，函数创建了一个名为 `b` 的缓冲区字符串，并将其内容设置为 "abcd"。然后，函数创建了一个名为 `b2` 的缓冲区字符串，并将其内容设置为 "efg"。

接着，函数调用 `pWriter.WriteMultiBuffer` 方法，将 `buf.MultiBuffer` 对象传递给 `pWriter` 作为第一个和第二个缓冲区。函数使用 `pReader.ReadMultiBuffer` 方法从 `pReader` 接收数据，并将其存储在 `rb` 变量中。

最后，函数比较 `rb.String()` 返回的字符串和 "abcdefg" 是否相等。如果不相等，函数将输出错误信息并调用 `t.Error` 函数。

由于该函数没有对 `buf` 包进行任何验证，因此如果 `buf` 包不正确，函数的行为可能会不可预测。


```go
func TestPipeReadWrite(t *testing.T) {
	pReader, pWriter := New(WithSizeLimit(1024))

	b := buf.New()
	b.WriteString("abcd")
	common.Must(pWriter.WriteMultiBuffer(buf.MultiBuffer{b}))

	b2 := buf.New()
	b2.WriteString("efg")
	common.Must(pWriter.WriteMultiBuffer(buf.MultiBuffer{b2}))

	rb, err := pReader.ReadMultiBuffer()
	common.Must(err)
	if r := cmp.Diff(rb.String(), "abcdefg"); r != "" {
		t.Error(r)
	}
}

```

该代码测试了一个名为 "TestPipeInterrupt" 的函数，它旨在测试是否能够安全地从一个带缓冲区的管道中读取数据。具体来说，该函数的作用如下：

1. 创建一个带缓冲区的管道，大小限制为 1024。
2. 创建一个包含 "a"、"b"、"c" 和 "d" 的字节数组。
3. 创建一个缓冲区 (buf) 并将其初始化为上述字节数组。
4. 使用缓冲区将数据写入管道。
5. 使用带缓冲区的管道读取数据。
6. 如果读取数据成功，则管道没有关闭错误，函数将不输出此错误。
7. 如果没有成功，则函数将输出错误。


```go
func TestPipeInterrupt(t *testing.T) {
	pReader, pWriter := New(WithSizeLimit(1024))
	payload := []byte{'a', 'b', 'c', 'd'}
	b := buf.New()
	b.Write(payload)
	common.Must(pWriter.WriteMultiBuffer(buf.MultiBuffer{b}))
	pWriter.Interrupt()

	rb, err := pReader.ReadMultiBuffer()
	if err != io.ErrClosedPipe {
		t.Fatal("expect io.ErrClosePipe, but got ", err)
	}
	if !rb.IsEmpty() {
		t.Fatal("expect empty buffer, but got ", rb.Len())
	}
}

```

这段代码的作用是测试一个名为 `PipeClose` 的函数，它关闭了一个带缓冲能力的 `PipeReader` 和一个 `PipeWriter`，然后向 `PipeWriter` 中写入数据，并关闭了 `PipeReader`。通过使用 `buf.New()` 创建了一个带缓冲的 `Buffer` 对象，并使用 `common.Must2` 函数向 `PipeWriter` 中写入数据，然后使用 `common.Must` 函数关闭了 `PipeWriter`。接下来，使用 `pReader.ReadMultiBuffer()` 函数读取带缓冲的 `Reader` 中的数据，并使用 `common.Must2` 函数检查读取到的内容是否与预期相同。最后，关闭 `PipeReader` 和 `PipeWriter`。


```go
func TestPipeClose(t *testing.T) {
	pReader, pWriter := New(WithSizeLimit(1024))
	payload := []byte{'a', 'b', 'c', 'd'}
	b := buf.New()
	common.Must2(b.Write(payload))
	common.Must(pWriter.WriteMultiBuffer(buf.MultiBuffer{b}))
	common.Must(pWriter.Close())

	rb, err := pReader.ReadMultiBuffer()
	common.Must(err)
	if rb.String() != string(payload) {
		t.Fatal("expect content ", string(payload), " but actually ", rb.String())
	}

	rb, err = pReader.ReadMultiBuffer()
	if err != io.EOF {
		t.Fatal("expected EOF, but got ", err)
	}
	if !rb.IsEmpty() {
		t.Fatal("expect empty buffer, but got ", rb.String())
	}
}

```

这段代码的作用是测试一个名为 `TestPipeLimitZero` 的函数，它会对缓冲区 `buf` 进行测试。具体来说，这段代码会创建一个缓冲区 `bb`，并向其中写入两个字符 `'a'` 和 `'b'`。然后，它创建一个名为 `pReader` 的读缓冲区和一个名为 `pWriter` 的写缓冲区，并将它们都设置为允许的最大大小。

接下来，代码会使用一个名为 `errg` 的错误组。对于每个写入操作，代码会向 `pWriter` 中写入缓冲区 `bb` 的内容，然后调用 `pWriter.WriteMultiBuffer(buf.MultiBuffer{bb})` 方法将缓冲区内容作为 `MultiBuffer` 对象写入 `pWriter`。最后，代码会进入一个条件语句，等待一段时间后测试缓冲区 `bb` 中的内容是否正确。如果测试失败，则会输出错误并等待 `errg` 错误组中的所有错误。




```go
func TestPipeLimitZero(t *testing.T) {
	pReader, pWriter := New(WithSizeLimit(0))
	bb := buf.New()
	common.Must2(bb.Write([]byte{'a', 'b'}))
	common.Must(pWriter.WriteMultiBuffer(buf.MultiBuffer{bb}))

	var errg errgroup.Group
	errg.Go(func() error {
		b := buf.New()
		b.Write([]byte{'c', 'd'})
		return pWriter.WriteMultiBuffer(buf.MultiBuffer{b})
	})
	errg.Go(func() error {
		time.Sleep(time.Second)

		var container buf.MultiBufferContainer
		if err := buf.Copy(pReader, &container); err != nil {
			return err
		}

		if r := cmp.Diff(container.String(), "abcd"); r != "" {
			return errors.New(r)
		}
		return nil
	})
	errg.Go(func() error {
		time.Sleep(time.Second * 2)
		return pWriter.Close()
	})
	if err := errg.Wait(); err != nil {
		t.Error(err)
	}
}

```

这段代码是一个名为 "TestPipeWriteMultiThread" 的测试函数，属于 "testing" 包。它的作用是测试一个名为 "MultiThread" 的函数，并确保它能在正确的条件下正常运行。

具体来说，这段代码以下几种方式实现了测试：

1. 首先，定义了一个名为 "pReader" 的 "Reader" 实例和一个名为 "pWriter" 的 "Writer" 实例。
2. 然后，定义了一个名为 "errg" 的 "Error" 错误组的变量。
3. 使用一个循环 "for" 该循环会迭代10次，并执行以下操作：
	1. 创建一个名为 "b" 的 "Buffer" 实例，并将其清空。
	2. 使用 "pWriter.WriteMultiBuffer" 方法将 "b" 中的数据写入一个 "MultiBuffer" 类型的 "Writer" 实例。
	3. 调用 "errg.Go" 函数，该函数执行以下操作：
		1. 创建一个名为 "buf.MultiBuffer" 的 "MultiBuffer" 类型。
		2. 使用 "pWriter.WriteMultiBuffer" 方法将 "buf" 中的数据写入 "MultiBuffer" 类型的 "Writer" 实例。
		3. 返回 "Writer" 实例。
	4. 使用 "time.Sleep" 函数暂停执行100毫秒，然后再使用 "pReader.ReadMultiBuffer" 方法从 "Reader" 实例中读取数据。
	5. 调用 "errg.Wait" 函数，等待 "errg" 错误组中所有延迟的错误。
	6. 使用 " common.Must" 函数，确保从 "Reader" 实例中读取的数据不小于两个字节。
	7. 最后，检查 "err" 变量，如果该变量不等于 "nil"，说明 "MultiThread" 函数运行成功。


```go
func TestPipeWriteMultiThread(t *testing.T) {
	pReader, pWriter := New(WithSizeLimit(0))

	var errg errgroup.Group
	for i := 0; i < 10; i++ {
		errg.Go(func() error {
			b := buf.New()
			b.WriteString("abcd")
			return pWriter.WriteMultiBuffer(buf.MultiBuffer{b})
		})
	}
	time.Sleep(time.Millisecond * 100)
	pWriter.Close()
	errg.Wait()

	b, err := pReader.ReadMultiBuffer()
	common.Must(err)
	if r := cmp.Diff(b[0].Bytes(), []byte{'a', 'b', 'c', 'd'}); r != "" {
		t.Error(r)
	}
}

```

这两段代码是用于测试一个名为`TestInterfaces`的函数，该函数旨在测试读写接口的实现。具体来说，第一段代码定义了一个名为`buf.Reader`的接口类型，以及一个名为`buf.TimeoutReader`的接口类型，都实现了`Reader`接口。同时，该函数还创建了一个名为`common.Interruptible`的接口类型，以及一个名为`common.Closable`的接口类型。这些接口类型都实现了`testing.Fatalf`函数的`SetFatBehavior`方法，用于在发生错误时模拟操作系统的行为。

第二段代码定义了一个名为`func BenchmarkPipeReadWrite`的函数，该函数创建了一个带有一个缓冲区读写接口的`testing.B`类型变量`b`。在该函数中，使用了一个名为`buf.New`的函数来创建一个空缓冲区。然后，使用`buf.MultiBuffer`创建了一个带缓冲区读写接口的`buf.Reader`类型变量`a`，并将该缓冲区扩展到与缓冲区大小相同的值。

接着，在该函数中定义了一个`for`循环，该循环使用了一个`common.Must`函数来保证写入操作的顺利进行。循环还定义了一个`reader.ReadMultiBuffer`函数，用于从读写接口中读取多个数据包。在该函数中，使用了一个`common.Must`函数来保证读取操作的顺利进行，并在需要时丢弃任何读取的数据。最后，将读取到的数据打印到控制台上，以便进行测试。


```go
func TestInterfaces(t *testing.T) {
	_ = (buf.Reader)(new(Reader))
	_ = (buf.TimeoutReader)(new(Reader))

	_ = (common.Interruptible)(new(Reader))
	_ = (common.Interruptible)(new(Writer))
	_ = (common.Closable)(new(Writer))
}

func BenchmarkPipeReadWrite(b *testing.B) {
	reader, writer := New(WithoutSizeLimit())
	a := buf.New()
	a.Extend(buf.Size)
	c := buf.MultiBuffer{a}

	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		common.Must(writer.WriteMultiBuffer(c))
		d, err := reader.ReadMultiBuffer()
		common.Must(err)
		c = d
	}
}

```

# `transport/pipe/reader.go`

这段代码定义了一个名为 "Pipe" 的包，它包含了一个名为 "Reader" 的结构体和多个名为 "ReadMultiBuffer" 的函数。

"Reader" 结构体包含一个指向 "pipe" 类型的变量，它是一个从 "pipe" 中读取内容的缓冲区 "Reader"。

"ReadMultiBuffer" 函数是一个 "buf.Reader"，它从 "Reader" 类型的缓冲区中读取多个字节并返回。

整个包的作用是提供一个名为 "Pipe" 的包，其中包含一个 "Reader" 类型的结构体和多个 "ReadMultiBuffer" 函数，用于从 "pipe" 中读取内容并返回缓冲区。


```go
package pipe

import (
	"time"

	"v2ray.com/core/common/buf"
)

// Reader is a buf.Reader that reads content from a pipe.
type Reader struct {
	pipe *pipe
}

// ReadMultiBuffer implements buf.Reader.
func (r *Reader) ReadMultiBuffer() (buf.MultiBuffer, error) {
	return r.pipe.ReadMultiBuffer()
}

```

这两段代码定义了一个名为 `ReadMultiBufferTimeout` 的函数和一个名为 `Interrupt` 的函数。它们属于一个名为 `Reader` 的类。

`ReadMultiBufferTimeout` 函数的作用是从给定的 `time.Duration` 时间间隔内读取数据。如果超时，它将返回一个名为 `buf.MultiBuffer` 的错误 `MultiBuffer` 对象和一个名为 `error` 的错误 `Reader` 对象。如果没有超时，它将返回 `Reader` 对象的 `MultiBuffer` 字段和一个没有错误的状态 `Reader` 对象。

`Interrupt` 函数的作用是暂时中断正在进行的读取操作，并返回一个不会丢弃数据的情况下中断 `Reader` 和 `pipe` 之间的连接。


```go
// ReadMultiBufferTimeout reads content from a pipe within the given duration, or returns buf.ErrTimeout otherwise.
func (r *Reader) ReadMultiBufferTimeout(d time.Duration) (buf.MultiBuffer, error) {
	return r.pipe.ReadMultiBufferTimeout(d)
}

// Interrupt implements common.Interruptible.
func (r *Reader) Interrupt() {
	r.pipe.Interrupt()
}

```

# `transport/pipe/writer.go`

这段代码定义了一个名为Pipe的包，该包包含一个名为Writer的接口，以及一个实现了Writer接口的类。

具体来说，这段代码描述了一个可以写入数据的缓冲区（pipe）的Writer类。通过使用 implements 关键字，该类实现了名为WriteMultiBuffer的buf.Writer接口。这个接口要求实现者写入的数据必须是多缓冲区的数据，而且数据写入后，writer应该继续写入数据。

因此，这段代码的作用是描述了一个可以写入多个缓冲区的数据的缓冲区Writer类。


```go
package pipe

import (
	"v2ray.com/core/common/buf"
)

// Writer is a buf.Writer that writes data into a pipe.
type Writer struct {
	pipe *pipe
}

// WriteMultiBuffer implements buf.Writer.
func (w *Writer) WriteMultiBuffer(mb buf.MultiBuffer) error {
	return w.pipe.WriteMultiBuffer(mb)
}

```

这两段代码定义了一个名为 `Close` 的函数，它实现了 `io.Closer` 接口，用于关闭管道并返回相应的错误。

具体来说，第一段代码 `Close` 函数的作用是关闭管道，通过调用 `w.pipe.Close` 来实现的。在管道关闭后，如果写入管道，会返回一个 `io.ErrClosedPipe` 错误，而如果读取管道，则会返回一个 `io.EOF` 错误。这些错误分别表示管道已关闭和文件已经结尾。

第二段代码 `Interrupt` 函数的作用是中断正在进行的写入操作，通过调用 `w.pipe.Interrupt` 来实现的。这个函数会在写入管道时暂停所有写入操作，并等待管道读写操作完成。如果在这个函数被调用后，管道仍然开放，那么所有写入操作都不会成功，并且可能会导致系统崩溃。


```go
// Close implements io.Closer. After the pipe is closed, writing to the pipe will return io.ErrClosedPipe, while reading will return io.EOF.
func (w *Writer) Close() error {
	return w.pipe.Close()
}

// Interrupt implements common.Interruptible.
func (w *Writer) Interrupt() {
	w.pipe.Interrupt()
}

```