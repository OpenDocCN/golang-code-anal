# v2ray-core源码解析 19

# `common/buf/reader.go`

这段代码定义了一个名为"buf"的包，其中包含一个名为"readOneUDP"的函数，其接收一个IO读取器(io.Reader)作为参数，并返回一个名为"Buffer"的缓冲区对象，或者错误。

函数内部，首先创建一个名为"b"的缓冲区对象，然后使用循环读取64个数据单元(即64字节)，并将读取到的数据读取到缓冲区中。循环条件为：b.IsEmpty() && err == nil。如果缓冲区为空或者没有错误，则说明数据读取成功，返回缓冲区对象，否则关闭缓冲区并返回错误。

函数中的UDP类型未定义，但根据代码中的引用，可以推断出该函数接受一个UDP套接字的读取器作为输入，并返回一个缓冲区对象。


```go
package buf

import (
	"io"

	"v2ray.com/core/common"
	"v2ray.com/core/common/errors"
)

func readOneUDP(r io.Reader) (*Buffer, error) {
	b := New()
	for i := 0; i < 64; i++ {
		_, err := b.ReadFrom(r)
		if !b.IsEmpty() {
			return b, nil
		}
		if err != nil {
			b.Release()
			return nil, err
		}
	}

	b.Release()
	return nil, newError("Reader returns too many empty payloads.")
}

```

这段代码定义了一个名为 `BufferedReader` 的类，它是一个 `Reader` 类型，用于从给定的 `Reader` 中读取数据并将其存储在内存中的一个 `Buffer` 中。

`BufferedReader` 中的 `Reader` 是一个 `io.Reader` 类型，表示要读取的数据来源。在 `ReadBuffer` 函数中，通过创建一个空的 `Buffer`，然后调用 `Reader` 中的 `ReadFrom` 函数来从 `Reader` 中读取数据，并将其存储在 `Buffer` 中。

如果读取成功，`ReadBuffer` 函数将返回一个指向 `Buffer` 对象的引用，同时不产生任何错误。如果读取失败，`ReadBuffer` 函数将返回一个 `nil` 表示的错误对象。

`BufferedReader` 中的 `Buffer` 是一个 `MultiBuffer` 类型，表示一个可以存储多个字节数据的缓冲区。`Spliter` 是一个函数，用于将 `MultiBuffer` 中的所有数据分割成多个连续的字节数组。


```go
// ReadBuffer reads a Buffer from the given reader.
func ReadBuffer(r io.Reader) (*Buffer, error) {
	b := New()
	n, err := b.ReadFrom(r)
	if n > 0 {
		return b, err
	}
	b.Release()
	return nil, err
}

// BufferedReader is a Reader that keeps its internal buffer.
type BufferedReader struct {
	// Reader is the underlying reader to be read from
	Reader Reader
	// Buffer is the internal buffer to be read from first
	Buffer MultiBuffer
	// Spliter is a function to read bytes from MultiBuffer
	Spliter func(MultiBuffer, []byte) (MultiBuffer, int)
}

```

这段代码是一个名为 `BufferedBytes` 的函数，它返回一个缓冲区 `Reader` 对象中缓存的字节数。

它实现了 `io.ByteReader` 接口，提供了从缓冲区 `Reader` 中读取字节并返回给调用者的功能。

具体来说，这段代码包含以下步骤：

1. 定义了一个名为 `BufferedReader` 的接口，它包含一个名为 `Buffer` 的字段和一个名为 `Spliter` 的字段（也是一个 `Reader` 接口的实现）。
2. 在 `BufferedReader` 的 `BufferedBytes` 函数中，通过调用 `r.Read` 函数来读取缓冲区中的字节，并返回读取的字节数。
3. 在 `BufferedReader` 的 `ReadByte` 函数中，通过调用 `r.Read` 函数来读取缓冲区中的第一个字节，并返回可能产生的错误。
4. 在 `BufferedReader` 的 `Read` 函数中，首先尝试从缓冲区中读取字节，如果无法成功，则执行从底层 `Reader` 对象中读取字节并返回给调用者。在这里，我们使用了 `io.ReadMulti` 函数来同时读取多个字节，并使用 `Spliter` 来分割缓冲区和底层 `Reader` 对象。
5. 在 `BufferedReader` 的 `BufferedBytes` 函数中，还实现了 `io.ByteReader` 接口的 `IsEmpty` 方法，用于判断缓冲区是否为空。


```go
// BufferedBytes returns the number of bytes that is cached in this reader.
func (r *BufferedReader) BufferedBytes() int32 {
	return r.Buffer.Len()
}

// ReadByte implements io.ByteReader.
func (r *BufferedReader) ReadByte() (byte, error) {
	var b [1]byte
	_, err := r.Read(b[:])
	return b[0], err
}

// Read implements io.Reader. It reads from internal buffer first (if available) and then reads from the underlying reader.
func (r *BufferedReader) Read(b []byte) (int, error) {
	spliter := r.Spliter
	if spliter == nil {
		spliter = SplitBytes
	}

	if !r.Buffer.IsEmpty() {
		buffer, nBytes := spliter(r.Buffer, b)
		r.Buffer = buffer
		if r.Buffer.IsEmpty() {
			r.Buffer = nil
		}
		return nBytes, nil
	}

	mb, err := r.Reader.ReadMultiBuffer()
	if err != nil {
		return 0, err
	}

	mb, nBytes := spliter(mb, b)
	if !mb.IsEmpty() {
		r.Buffer = mb
	}
	return nBytes, nil
}

```

这段代码实现了一个名为 ReadMultiBuffer 的函数，它是Reader接口的实现，允许从缓冲区中读取多个数据。

首先，代码检查缓冲区是否为空，如果是，则读取数据并以MB的形式返回，否则，代码会将缓冲区清空，并返回错误。

其次，代码定义了一个名为 ReadAtMost 的函数，它允许从缓冲区中读取至多指定大小的数据，并返回一个MB类型的变量。

最后，代码定义了一个名为 SplitSize 的函数，它将缓冲区数据按照指定大小进行分割并返回两个参数，其中一个参数是分割后剩余的数据，另一个参数是分割后的数据大小。如果缓冲区为空，函数将返回错误。


```go
// ReadMultiBuffer implements Reader.
func (r *BufferedReader) ReadMultiBuffer() (MultiBuffer, error) {
	if !r.Buffer.IsEmpty() {
		mb := r.Buffer
		r.Buffer = nil
		return mb, nil
	}

	return r.Reader.ReadMultiBuffer()
}

// ReadAtMost returns a MultiBuffer with at most size.
func (r *BufferedReader) ReadAtMost(size int32) (MultiBuffer, error) {
	if r.Buffer.IsEmpty() {
		mb, err := r.Reader.ReadMultiBuffer()
		if mb.IsEmpty() && err != nil {
			return nil, err
		}
		r.Buffer = mb
	}

	rb, mb := SplitSize(r.Buffer, size)
	r.Buffer = rb
	if r.Buffer.IsEmpty() {
		r.Buffer = nil
	}
	return mb, nil
}

```

该函数的作用是读取一个二进制缓冲区（BufferedReader）对象（r），将其内容写入一个指定的内部缓冲区（Writer）对象（writer）。如果缓冲区（r）不为空，则先将其内容复制到一个大小计数器（SizeCounter）中，然后将缓冲区（r）的内容以多个字节（io.Byte）写入到写入缓冲区（writer）中。如果写入过程中出现错误，函数返回一个错误值（error）。函数返回写入成功后的大小计数器（SizeCounter）的值，如果写入失败，则返回错误值（error）。


```go
func (r *BufferedReader) writeToInternal(writer io.Writer) (int64, error) {
	mbWriter := NewWriter(writer)
	var sc SizeCounter
	if r.Buffer != nil {
		sc.Size = int64(r.Buffer.Len())
		if err := mbWriter.WriteMultiBuffer(r.Buffer); err != nil {
			return 0, err
		}
		r.Buffer = nil
	}

	err := Copy(r.Reader, mbWriter, CountSize(&sc))
	return sc.Size, err
}

```

这两段代码定义了两个函数：`WriteTo` 和 `Interrupt`。它们的作用如下：

1. `WriteTo` 函数：

该函数接收一个 `Writer` 类型的参数，它是 `io.WriterTo` 接口的实现。这个函数的作用是将 `BufferedReader` 内部的数据写入到 `Writer` 中。它将返回写入的字节数和可能的错误。

如果调用 `WriteTo` 时出现错误，可能是由于以下原因：

- 读取操作尚未完成，但已经写入了数据，这时候可能会导致写入的字节数超过Writer的字节数，产生错误。
- 写入操作出现了EOF错误，这将导致整个读取操作失败。

2. `Interrupt` 函数：

该函数是一个 `common.Interruptible` 的实现了 `Interrupt` 接口。它的作用是在 `BufferedReader` 的 `Reader` 上执行写入操作时，通知操作系统的调度器进行干预。这可能会导致读取操作暂停，但不会挂起整个进程。

当 `Interrupt` 被调用时，它会立即停止 `BufferedReader` 的写入操作，并将任何 pending的字节数丢弃。这将导致读取操作暂停，但不会挂起整个进程。


```go
// WriteTo implements io.WriterTo.
func (r *BufferedReader) WriteTo(writer io.Writer) (int64, error) {
	nBytes, err := r.writeToInternal(writer)
	if errors.Cause(err) == io.EOF {
		return nBytes, nil
	}
	return nBytes, err
}

// Interrupt implements common.Interruptible.
func (r *BufferedReader) Interrupt() {
	common.Interrupt(r.Reader)
}

// Close implements io.Closer.
```

此代码定义了一个名为`func`的函数，它接收一个名为`r`的`BufferedReader`类型的参数，并返回一个`Close`错误。

`BufferedReader`是一个用于读取大型数据块的`Reader`类型，它可以同时从多个缓冲区中读取数据。

`SingleReader`是一个`Reader`类型，它每读取一个缓冲区就会读取一次数据。它包含一个`Reader`指针和一个`MultiBuffer`类型的参数`MultiBuffer`，用于将多个缓冲区中的数据组合成一个大块并返回。

`func`函数的作用是处理一个`BufferedReader`类型的参数`r`，并使用`common.Close`函数关闭`r`所代表的读取器。如果关闭成功，则返回一个`Close`错误，否则返回一个未定义的错误。

`Close`函数是一个通用的函数，用于关闭任何`Reader`。它接收一个`Reader`指针和一个错误类型参数，并尝试关闭读取器以返回一个错误。如果关闭成功，则返回一个错误，否则返回一个未定义的错误。由于没有明确指定错误类型，因此函数将返回一个成功关闭读取器的错误。


```go
func (r *BufferedReader) Close() error {
	return common.Close(r.Reader)
}

// SingleReader is a Reader that read one Buffer every time.
type SingleReader struct {
	io.Reader
}

// ReadMultiBuffer implements Reader.
func (r *SingleReader) ReadMultiBuffer() (MultiBuffer, error) {
	b, err := ReadBuffer(r.Reader)
	return MultiBuffer{b}, err
}

```

这段代码定义了一个名为 PacketReader 的 struct 类型，该类型包含一个 io.Reader 类型的 field，表示读者可以从一个缓冲区中读取数据。

接着，定义了一个名为 PacketReader 的函数，该函数实现了 Reader 接口，用于从 UDP 套接字中读取数据。该函数返回一个名为 MultiBuffer 的元组类型和一个名为 error 的错误类型。

对于 PacketReader.ReadMultiBuffer() 函数，实现了从 UDP 套接字中读取多个数据包的功能。函数接收一个 PacketReader 类型的实例作为参数，首先调用 readOneUDP() 函数尝试从 UDP 套接字中读取一个数据包，如果失败则返回 nil 和错误。如果成功，则返回一个名为 MultiBuffer 的元组类型，其中包含读取到的数据包，以及一个名为 nil 的错误类型。


```go
// PacketReader is a Reader that read one Buffer every time.
type PacketReader struct {
	io.Reader
}

// ReadMultiBuffer implements Reader.
func (r *PacketReader) ReadMultiBuffer() (MultiBuffer, error) {
	b, err := readOneUDP(r.Reader)
	if err != nil {
		return nil, err
	}
	return MultiBuffer{b}, nil
}

```

# `common/buf/reader_test.go`

这段代码的主要目的是测试一个名为 "buf\_test" 的包。这个包中包含了一个名为 "TestBytesReaderWriteTo" 的函数，它的作用是测试如何将多个字符串字节读取并写到缓冲区中。

具体来说，这段代码创建了一个缓冲区（即字节数组） "b1"，然后向其中添加两个字符串 "abc" 和 "efg"。接着，它创建了两个管道 "pReader" 和 "pWriter"，并使用 "pipe.WithSizeLimit" 方法设置每个管道的大小限制为 1024字节。然后，它使用 "BufferedReader" 类型的 "reader" 从 "pReader" 读取数据，并将读取到的数据写入 "pWriter" 中。

在 "TestBytesReaderWriteTo" 函数内部，首先创建了一个缓冲区 "b1"，并向其中写入了两个字符串。然后，它创建了两个新的管道 "pWriter2" 和 "pReader2"，并使用 "pipe.New" 方法创建了这两个管道。由于 "pWriter2" 设置为 "false"，因此无法写入数据到 "pWriter" 中。然后，它使用 "io.Copy" 函数从 "reader" 读取数据，并将其复制到 "pWriter2" 中。最后，它使用 "pReader2" 读取了 "b1" 中的数据，并将其解码为字符串。如果解码后的字符串不是 "abcefg"，那么函数就会输出错误。

综合来看，这段代码的主要目的是测试 "buf\_test" 包中的 "TestBytesReaderWriteTo" 函数，以验证它是否正确地将两个字符串字节读取并写到缓冲区中。


```go
package buf_test

import (
	"bytes"
	"io"
	"strings"
	"testing"

	"v2ray.com/core/common"
	. "v2ray.com/core/common/buf"
	"v2ray.com/core/transport/pipe"
)

func TestBytesReaderWriteTo(t *testing.T) {
	pReader, pWriter := pipe.New(pipe.WithSizeLimit(1024))
	reader := &BufferedReader{Reader: pReader}
	b1 := New()
	b1.WriteString("abc")
	b2 := New()
	b2.WriteString("efg")
	common.Must(pWriter.WriteMultiBuffer(MultiBuffer{b1, b2}))
	pWriter.Close()

	pReader2, pWriter2 := pipe.New(pipe.WithSizeLimit(1024))
	writer := NewBufferedWriter(pWriter2)
	writer.SetBuffered(false)

	nBytes, err := io.Copy(writer, reader)
	common.Must(err)
	if nBytes != 6 {
		t.Error("copy: ", nBytes)
	}

	mb, err := pReader2.ReadMultiBuffer()
	common.Must(err)
	if s := mb.String(); s != "abcefg" {
		t.Error("content: ", s)
	}
}

```

该代码的作用是测试一个名为 `TestBytesReaderMultiBuffer` 的函数。该函数使用 `pipe.New` 创建了一个带缓冲区限制的通道（或管道），并创建了一个 `BufferedReader` 对象和一个 `MultiBuffer` 对象。然后，它向该 `MultiBuffer` 对象添加了两个 `BufferedReader` 对象，并使用 `pipe.WriteMultiBuffer` 方法将多个字符串读取到一个缓冲区中。

具体来说，该函数的作用是：

1. 创建一个带缓冲区限制的通道（或管道） `pReader` 和一个 `BufferedWriter` 对象 `pWriter`。
2. 创建两个新的 `BufferedReader` 对象 `b1` 和 `b2`，并将它们的内容设置为 "abc" 和 "efg"。
3. 使用 `pipe.WriteMultiBuffer` 方法将多个字符串读取到一个缓冲区中，并关闭 `pWriter`。
4. 创建一个名为 `mbReader` 的 `MultiBuffer` 对象 `mb`。
5. 使用 `mbReader.ReadMultiBuffer` 方法读取多个字符串到一个缓冲区中，并关闭 `mbReader`。
6. 检查 `mb.String()` 是否等于 "abcefg"，如果是，则输出 "content: ", `mb.String()` 的内容。否则，输出 "test error: ".


```go
func TestBytesReaderMultiBuffer(t *testing.T) {
	pReader, pWriter := pipe.New(pipe.WithSizeLimit(1024))
	reader := &BufferedReader{Reader: pReader}
	b1 := New()
	b1.WriteString("abc")
	b2 := New()
	b2.WriteString("efg")
	common.Must(pWriter.WriteMultiBuffer(MultiBuffer{b1, b2}))
	pWriter.Close()

	mbReader := NewReader(reader)
	mb, err := mbReader.ReadMultiBuffer()
	common.Must(err)
	if s := mb.String(); s != "abcefg" {
		t.Error("content: ", s)
	}
}

```

这段代码在测试一个名为 `TestReadByte` 的函数。函数的参数是一个名为 `t` 的 testing.T 类型的切片，函数内部通过 `NewReader` 创建了一个 `BufferedReader` 对象，该对象通过 `Reader` 属性从字符串 `"abcd"` 中读取一个字符并返回。如果返回值不是字符 'a', 则函数会输出该错误并关闭错误输出。如果返回三个字符， 函数会检查 `BufferedReader` 的缓冲区是否还有其他字符， 如果是， 则函数会输出错误并关闭错误输出。如果函数无法在缓冲区中写入任何字符， 函数会输出错误并关闭错误输出。


```go
func TestReadByte(t *testing.T) {
	sr := strings.NewReader("abcd")
	reader := &BufferedReader{
		Reader: NewReader(sr),
	}
	b, err := reader.ReadByte()
	common.Must(err)
	if b != 'a' {
		t.Error("unexpected byte: ", b, " want a")
	}
	if reader.BufferedBytes() != 3 { // 3 bytes left in buffer
		t.Error("unexpected buffered Bytes: ", reader.BufferedBytes())
	}

	nBytes, err := reader.WriteTo(DiscardBytes)
	common.Must(err)
	if nBytes != 3 {
		t.Error("unexpect bytes written: ", nBytes)
	}
}

```

这两段代码都是 Go 语言中的测试函数，旨在测试某个名为 "ReadBuffer" 的函数的实现是否正确。

第一段代码 "TestReadBuffer" 的作用是测试 "ReadBuffer" 函数的正确性。函数接受一个 testing.T 类型的参数，意味着它将作为断言的接受者。函数内部，首先创建一个字符串串 a 的 newReader，然后使用 ReadBuffer 函数读取 a 的字节并将其存储在缓冲区中。接下来，使用 .String() 方法将缓冲区中的字节转换为字符串，并将其存储在一个名为 s 的变量中。最后，比较生成的字符串和预期的字符串是否相等，如果不相等，则使用 t.Error 函数输出错误信息。

第二段代码 "TestReadAtMost" 的作用是测试 "ReadAtMost" 函数的正确性。函数同样接受一个 testing.T 类型的参数，并将其作为断言的接受者。函数内部，首先创建一个字符串 sr，然后创建一个名为 reader 的 BufferedReader，并将 reader 的 Reader 属性设置为 sr，这样读者就可以从 sr 中逐行读取数据。接下来，使用 ReadAtMost 函数从 reader 中读取不超过指定数量的字符，并将其存储在一个名为 mb 的变量中。最后，使用 .String() 方法将 mb 转换的字符串存储在变量 s 中，并比较生成的字符串和预期的字符串是否相等，如果不相等，则使用 t.Error 函数输出错误信息。

由于这两段代码都使用了 .Must() 函数，表示它们对传入的错误信息不敏感，因此即使它们的逻辑失败，它们也不会输出错误信息。


```go
func TestReadBuffer(t *testing.T) {
	{
		sr := strings.NewReader("abcd")
		buf, err := ReadBuffer(sr)
		common.Must(err)

		if s := buf.String(); s != "abcd" {
			t.Error("unexpected str: ", s, " want abcd")
		}
		buf.Release()
	}

}

func TestReadAtMost(t *testing.T) {
	sr := strings.NewReader("abcd")
	reader := &BufferedReader{
		Reader: NewReader(sr),
	}

	mb, err := reader.ReadAtMost(3)
	common.Must(err)
	if s := mb.String(); s != "abc" {
		t.Error("unexpected read result: ", s)
	}

	nBytes, err := reader.WriteTo(DiscardBytes)
	common.Must(err)
	if nBytes != 1 {
		t.Error("unexpect bytes written: ", nBytes)
	}
}

```

这两段代码都是 Go 语言中的测试用例，主要目的是测试不同类型的读者（PacketReader，ReaderInterface）对字符串（alpha）的读取是否正确。

1. `func TestPacketReader_ReadMultiBuffer(t *testing.T)` 是针对 PacketReader 类型设置的测试用例。主要作用是在不输出源代码的情况下，验证能否正确从给定的字符串中读取多个字符。具体实现包括：定义一个代表字符串 alpha 的字节缓冲区（`const alpha = "abcefg"`），创建一个 PacketReader 实例（`const reader = &PacketReader{buf}`），然后循环从缓冲区中读取字符，直至遇到空字符（`buf.String() == ""`）。接着将每读取到一个字符（`common.Must(err)`）将其转换成字符串（`s := mb.String()`），并验证结果是否与 alpha 相同。最后，若验证失败，将输出错误信息（`t.Error("content: ", s)`）。

2. `func TestReaderInterface(t *testing.T)` 是针对 ReaderInterface 类型设置的测试用例。主要作用是在不输出源代码的情况下，验证能否正确从不同类型的读者中读取字符（如 PacketReader，Reader）。具体实现包括：定义一个代表字符串 alpha 的字节缓冲区（`const alpha = "abcefg"`），创建一个 PacketReader 实例（`const reader = &PacketReader{buf}`），然后循环从缓冲区中读取字符（`common.Must(err)`）。接着，分别创建一个 Reader 实例（`func TestReaderInterface_Reader(t *testing.T)`），一个 BufferedReader 实例（`func TestReaderInterface_BufferedReader(t *testing.T)`）和一个 ByteReader 实例（`func TestReaderInterface_ByteReader(t *testing.T)`），分别从不同的缓冲区或字节读取字符。最后，验证每个实例是否能正确读取字符（`t.True()`）。


```go
func TestPacketReader_ReadMultiBuffer(t *testing.T) {
	const alpha = "abcefg"
	buf := bytes.NewBufferString(alpha)
	reader := &PacketReader{buf}
	mb, err := reader.ReadMultiBuffer()
	common.Must(err)
	if s := mb.String(); s != alpha {
		t.Error("content: ", s)
	}
}

func TestReaderInterface(t *testing.T) {
	_ = (io.Reader)(new(ReadVReader))
	_ = (Reader)(new(ReadVReader))

	_ = (Reader)(new(BufferedReader))
	_ = (io.Reader)(new(BufferedReader))
	_ = (io.ByteReader)(new(BufferedReader))
	_ = (io.WriterTo)(new(BufferedReader))
}

```

# `common/buf/readv_posix.go`

这段代码是一个 Go 语言编写的构建函数，它的作用是编译不同的目标，并输出不同的文件头。

具体来说，接下来的三条注释（// +build !windows // +build !wasm // +build !illumos）都是通过 `!` 结尾的 build 函数选项，用来分别编译针对 Windows、Wasm 和 Illumos 平台的目标。

1. `// +build !windows`：编译针对 Windows 平台的 target。
2. `// +build !wasm`：编译针对 Wasm 平台的 target。
3. `// +build !illumos`：编译针对 Illumos 平台的 target。

这里 `!` 是一个通配符，表示只要输入的名称中包含 `!`，就执行该选项对应的编译操作。


```go
// +build !windows
// +build !wasm
// +build !illumos

package buf

import (
	"syscall"
	"unsafe"
)

type posixReader struct {
	iovecs []syscall.Iovec
}

```

这段代码定义了一个名为`func`的函数接收一个名为`r`的指针和一个字符串数组`bs`作为参数。函数的作用是初始化`r`指向的`posixReader`对象的`iovecs`字段，并将`bs`中的每个元素读入到`r.iovecs`字段中。

具体来说，函数首先检查`iovecs`是否为空。如果是，则创建一个大小为`len(bs)`的`iovec`数组，并将每个`iovec`对象设置为`syscall.Iovec`类型的`base`字段加上`len(bs)`的长度`Size`。然后，遍历`bs`中的每个元素，将其类型设置为`iovec`类型，并将`len`字段设置为`int(Size)`。最后，将`iovecs`字段赋值给`r.iovecs`，并返回从`syscall.Syscall`函数返回的读取字节数。

接下来是函数的第二个实现：`func (r *posixReader) Read(fd uintptr) int32`。这个函数的作用是读取来自`fd`的`posixReader`对象中的数据，并返回数据的长度。它通过调用`syscall.Syscall`函数实现，并使用`r.iovecs`字段中的数据。

需要注意的是，由于这些函数使用了操作系统调用，因此需要确保程序在调用时具有相应的权限。


```go
func (r *posixReader) Init(bs []*Buffer) {
	iovecs := r.iovecs
	if iovecs == nil {
		iovecs = make([]syscall.Iovec, 0, len(bs))
	}
	for idx, b := range bs {
		iovecs = append(iovecs, syscall.Iovec{
			Base: &(b.v[0]),
		})
		iovecs[idx].SetLen(int(Size))
	}
	r.iovecs = iovecs
}

func (r *posixReader) Read(fd uintptr) int32 {
	n, _, e := syscall.Syscall(syscall.SYS_READV, fd, uintptr(unsafe.Pointer(&r.iovecs[0])), uintptr(len(r.iovecs)))
	if e != 0 {
		return -1
	}
	return int32(n)
}

```

这两段代码定义了一个名为`func`的函数，它接收一个名为`r`的整数类型的指针变量，代表一个`posixReader`类型的读取器。

函数的内容如下：


func (r *posixReader) Clear() {
	for idx := range r.iovecs {
		r.iovecs[idx].Base = nil
	}
	r.iovecs = r.iovecs[:0]
}


这个函数的作用是清空`r.iovecs`数组，即将`r.iovecs`中的所有元素都置为`nil`。

接下来是另一个函数定义，接收一个名为`newMultiReader`的整数类型的指针变量，代表一个`multiReader`类型的读取器。


func newMultiReader() multiReader {
	return &posixReader{}
}


这个函数的作用是返回一个`multiReader`类型的读取器，这个读取器包含一个空的`posixReader`类型的读取器。

总的来说，这两段代码定义了一个简单的函数，对传入的`posixReader`类型的读取器进行操作，实现了清空读取器中所有元素的操作。同时，还定义了一个新的`multiReader`类型的函数，用于创建一个空的`posixReader`类型的读取器。


```go
func (r *posixReader) Clear() {
	for idx := range r.iovecs {
		r.iovecs[idx].Base = nil
	}
	r.iovecs = r.iovecs[:0]
}

func newMultiReader() multiReader {
	return &posixReader{}
}

```

# `common/buf/readv_reader.go`

这段代码是一个 Rust 代码，它定义了一个名为 "buf" 的包。这个包通过两个函数 "build" 和 "!wasm" 来构建和运行程序。

下面是对代码中使用的几个关键部分的解释：

1. `//`：这是一个名为 "!wasm" 的函数指针，它告诉编译器在编译时不要将这个文件编译成 WebAssembly 机器码，而是直接执行。
2. `package buf`：这是一个名为 "buf" 的包名称，它告诉编译器如何打包和分发这个包。
3. `import ( "io" "runtime" "syscall" )`：这是一个导入列表，它告诉编译器要导入 "io"、"runtime" 和 "syscall" 这三个库。
4. `type allocStrategy struct { current uint32 }`：这是一个名为 "allocStrategy" 的用户自定义类型，它有一个名为 "current" 的字段，类型为 "uint32"。
5. `runtime.绑载`：`runtime.绑定` 函数允许您为您的程序提供高性能的内存管理。通过 `runtime.绑定`，您可以向您的程序中添加对内存的更高级别的管理。
6. `syscall. syscall ...`：这是一个调用 "syscall" 函数的引用，`syscall.syscall` 函数是一个系统调用，允许您执行操作系统级别的操作，如文件 I/O 和网络调用。


```go
// +build !wasm

package buf

import (
	"io"
	"runtime"
	"syscall"

	"v2ray.com/core/common/platform"
)

type allocStrategy struct {
	current uint32
}

```

这两段代码定义了两个名为"func"的函数，接收一个名为"s"的指针变量作为参数，并返回一个名为"Current"的uint32类型的局部变量。

第一个函数名为"Current"，函数体中首先定义了一个名为"s.current"的局部变量，并将其返回，函数体中还有一行代码，将"s.current"的值乘以4，这是通过将"s.current"乘以 4来扩大当前分配的内存空间大小，扩大之后如果这个值大于等于32，则将值限制为32，否则将值设置为1，最后将结果返回。

第二个函数名为"Adjust"，函数体中首先定义了一个名为"n"的局部变量，并将其设置为一个uint32类型的参数，函数体中还有一行代码，将"s.current"的值乘以4，这是通过将"s.current"乘以 4来扩大当前分配的内存空间大小，扩大之后如果这个值大于等于32，则将值限制为32，否则将值设置为n，最后将结果返回。


```go
func (s *allocStrategy) Current() uint32 {
	return s.current
}

func (s *allocStrategy) Adjust(n uint32) {
	if n >= s.current {
		s.current *= 4
	} else {
		s.current = n
	}

	if s.current > 32 {
		s.current = 32
	}

	if s.current == 0 {
		s.current = 1
	}
}

```

该函数是一个名为`multiReader`的接口，它定义了一个`Init`方法和一个`Read`方法。它还有一个名为`sallocStrategy`的参数，该参数是一个指向`AllocStrategy`类型的指针。

函数`Alloc`的目的是分配一个与`sallocStrategy`指向的`AllocStrategy`相同数量的内存空间，并返回该内存空间。函数内部使用一个循环来创建一个与`sallocStrategy`相同数量的`Buffer`类型的切片，并为切片分配内存。

函数`multiReader`定义了一个接口，该接口包含三个方法：

1. `Init`方法：初始化一个包含给定`Buffer`类型的切片，并设置一个缓冲区`fd`的读取器。
2. `Read`方法：使用`readv` syscall将一个包含指定`Buffer`类型的数据和一个指定文件描述符`fd`的数据读取到缓冲区中，并返回读取的数据个数。
3. `Clear`方法：清除缓冲区。

该函数的作用是创建一个可以读取多个文件的`multiReader`类型的实例，并提供了一个初始化和读取数据的接口。


```go
func (s *allocStrategy) Alloc() []*Buffer {
	bs := make([]*Buffer, s.current)
	for i := range bs {
		bs[i] = New()
	}
	return bs
}

type multiReader interface {
	Init([]*Buffer)
	Read(fd uintptr) int32
	Clear()
}

// ReadVReader is a Reader that uses readv(2) syscall to read data.
```

这是一段使用Go编程语言实现的代码，定义了一个名为ReadVReader的类型。该类型包含一个io.Reader类型的字段和一个syscall.RawConn类型的字段，以及一个名为multiReader的multiReader类型字段和一个名为alloc的allocStrategy类型字段。

该代码首先定义了一个名为NewReadVReader的函数，该函数接收两个参数，一个是io.Reader类型的字段和一个syscall.RawConn类型的字段。函数创建一个新的ReadVReader实例，并返回该实例的指针。

函数创建新ReadVReader实例的过程如下：

1. 从Reader字段中获取一个io.Reader类型的实例。
2. 从rawConn字段中获取一个syscall.RawConn类型的实例。
3. 创建一个名为multiReader的multiReader类型实例，并将其赋值为 allocStrategy类型的字段。
4. 将当前的multiReader实例的值设置为1，以确保能够使用该实例的最大数量。
5. 返回新创建的multiReader实例的实例。


```go
type ReadVReader struct {
	io.Reader
	rawConn syscall.RawConn
	mr      multiReader
	alloc   allocStrategy
}

// NewReadVReader creates a new ReadVReader.
func NewReadVReader(reader io.Reader, rawConn syscall.RawConn) *ReadVReader {
	return &ReadVReader{
		Reader:  reader,
		rawConn: rawConn,
		alloc: allocStrategy{
			current: 1,
		},
		mr: newMultiReader(),
	}
}

```

该函数的作用是读取一个ReadVReader对象中的多个数据包。它将首先分配一个缓冲区(bs)并初始化ReadVReader对象的mr成员。

然后，它使用ReadVReader对象的rawConn来读取数据包。对于每个数据包，它首先检查ReadVReader对象的mr是否已满，如果是，就继续读取。如果不是，它读取数据包并将其存储在bs中，然后清空mr以准备读取下一个数据包。

如果ReadVReader对象在尝试读取数据包时出现错误，它将释放bs中的所有数据包并返回一个错误。

最后，它检查bs中是否有任何可用数据包，如果有，它将其返回。如果没有，它返回一个空的多缓冲区。


```go
func (r *ReadVReader) readMulti() (MultiBuffer, error) {
	bs := r.alloc.Alloc()

	r.mr.Init(bs)
	var nBytes int32
	err := r.rawConn.Read(func(fd uintptr) bool {
		n := r.mr.Read(fd)
		if n < 0 {
			return false
		}

		nBytes = n
		return true
	})
	r.mr.Clear()

	if err != nil {
		ReleaseMulti(MultiBuffer(bs))
		return nil, err
	}

	if nBytes == 0 {
		ReleaseMulti(MultiBuffer(bs))
		return nil, io.EOF
	}

	nBuf := 0
	for nBuf < len(bs) {
		if nBytes <= 0 {
			break
		}
		end := nBytes
		if end > Size {
			end = Size
		}
		bs[nBuf].end = end
		nBytes -= end
		nBuf++
	}

	for i := nBuf; i < len(bs); i++ {
		bs[i].Release()
		bs[i] = nil
	}

	return MultiBuffer(bs[:nBuf]), nil
}

```

这段代码定义了一个名为`ReadMultiBuffer`的函数，它实现了`Reader`接口。这个函数的参数是一个`ReadVReader`类型的指针`r`，它有一个`ReadBuffer`函数和一个`readMulti`函数作为辅助函数。

如果`r.alloc.Current()`的值为1，说明正在读取一个独立的缓冲区，`ReadBuffer`函数将被调用，并且如果缓冲区为空，将会自动分配一个大小为`r.Reader.Alloc()`的缓冲区。如果缓冲区不为空，则返回一个`MultiBuffer`类型的变量`mb`，其中包含`r.Reader`缓冲区中的数据。如果错误，函数将返回一个`nil`值。

如果`r.readMulti`函数在尝试读取多个缓冲区时出现错误，函数将返回一个`nil`值。否则，函数将尝试分配适当长度的内存来存储缓冲区中的数据，并返回`MultiBuffer`类型的变量`mb`。如果分配错误，函数将返回一个`nil`值。


```go
// ReadMultiBuffer implements Reader.
func (r *ReadVReader) ReadMultiBuffer() (MultiBuffer, error) {
	if r.alloc.Current() == 1 {
		b, err := ReadBuffer(r.Reader)
		if b.IsFull() {
			r.alloc.Adjust(1)
		}
		return MultiBuffer{b}, err
	}

	mb, err := r.readMulti()
	if err != nil {
		return nil, err
	}
	r.alloc.Adjust(uint32(len(mb)))
	return mb, nil
}

```

这段代码是一个用于设置或取消一个名为 "useReadv" 的环境变量的 Go 语言程序。它包含两个函数，分别是 `init` 和 `run`。

1. `init` 函数的作用是在程序启动时设置 "useReadv" 环境变量的默认值。它首先调用 `platform.NewEnvFlag("v2ray.buf.readv").GetValue` 函数，这个函数会获取一个名为 `v2ray.buf.readv` 的环境变量。如果这个环境变量已经被定义，函数将返回它的值，否则，函数将返回一个默认值，这个默认值是 "NOT_DEFINED_AT_ALL"。函数接着使用 `const defaultFlagValue = "NOT_DEFINED_AT_ALL"` 来定义一个名为 `defaultFlagValue` 的常量，它的值也是 "NOT_DEFINED_AT_ALL"。最后，函数将 `defaultFlagValue` 和 `defaultFlagValue` 中的第一个值作为 `value` 变量，这样无论哪一个值被使用，都会将 `useReadv` 设置为 `true`。

2. `run` 函数的作用是监控 `useReadv` 环境变量是否被设置为 `true`。它首先检查运行时使用的架构和操作系统。如果使用的操作系统是 Linux、macOS 或 Windows 中的任意一个，并且使用的 Go 语言版本是 380 或更高版本，那么 `useReadv` 将被设置为 `true`。否则，`run` 函数将返回一个名为 `useReadv` 的布尔值，这个值将根据 `runtime.GOARCH` 和 `runtime.GOOS` 环境变量来确定是否启用读取缓冲区。


```go
var useReadv = false

func init() {
	const defaultFlagValue = "NOT_DEFINED_AT_ALL"
	value := platform.NewEnvFlag("v2ray.buf.readv").GetValue(func() string { return defaultFlagValue })
	switch value {
	case defaultFlagValue, "auto":
		if (runtime.GOARCH == "386" || runtime.GOARCH == "amd64" || runtime.GOARCH == "s390x") && (runtime.GOOS == "linux" || runtime.GOOS == "darwin" || runtime.GOOS == "windows") {
			useReadv = true
		}
	case "enable":
		useReadv = true
	}
}

```

# `common/buf/readv_reader_wasm.go`

这是一个 Go 语言中的 package 名为 "buf" 的函数，其作用是定义了一个名为 "NewReadVReader" 的函数，接收两个参数：一个 io.Reader 类型的读者和一个 syscall.RawConn 类型的操作系统调用。

函数的作用是创建一个名为 "reader" 的 io.Reader 类型实例，可以使用 "io.NewReader" 函数创建，该函数会根据提供的Reader类型创建一个新的 io.Reader 实例。然后函数使用 syscall.RawConn 类型将读者与操作系统调用 "syscall.Read" 组合在一起，实现从操作系统读取数据并返回给函数调用者。最后一个参数是一个名为 "rawConn" 的 syscall.RawConn 类型实例，它用于将 io.Reader 类型与操作系统调用 "syscall.Read" 组合在一起。

函数内部没有实现具体的逻辑，只是简单地将创建的 io.Reader 类型实例与 syscall.RawConn 类型实例作为参数传递给函数外部。


```go
// +build wasm

package buf

import (
	"io"
	"syscall"
)

const useReadv = false

func NewReadVReader(reader io.Reader, rawConn syscall.RawConn) Reader {
	panic("not implemented")
}

```

# `common/buf/readv_test.go`

这段代码是一个Go语言编写的测试框架，用于测试一个名为"buf_test"的包。其主要作用是创建一个名为"buf_test_临时服务器"的服务器，该服务器使用Go标准库中的"sync"包中的"errgroup"子包来处理测试中的错误。

具体来说，代码的作用是：

1. 使用Go标准库中的"build"工具，构建一个名为"buf_test_临时服务器.go"的文件。
2. 通过使用"!wasm"参数，告诉编译器不要将任何Wasm模块打包到生成的可执行文件中。
3. 导入所需的第三方库：net、testing和crypto/rand。
4. 实现一个名为"buf_test"的测试类，该类使用net.Listen函数创建一个TCP服务器，然后使用服务器接收端的缓冲区接收数据。
5. 在测试类中，实现一个名为"run"的函数，该函数使用一个名为"buf_test_临时服务器"的服务器，并将测试数据发送给它，然后从服务器接收端缓冲区读取数据并打印到控制台。
6. 在"buf_test"测试类的顶部的main函数中，创建一个名为"test_run"的函数，该函数会调用"buf_test"测试类中的"run"函数，并输出测试结果。

综上所述，这段代码的作用是创建一个用于测试一个名为"buf_test"的包的测试服务器，并运行测试以验证它是否能够正常工作。


```go
// +build !wasm

package buf_test

import (
	"crypto/rand"
	"net"
	"testing"

	"github.com/google/go-cmp/cmp"

	"golang.org/x/sync/errgroup"
	"v2ray.com/core/common"
	. "v2ray.com/core/common/buf"
	"v2ray.com/core/testing/servers/tcp"
)

```

这段代码的作用是测试一个名为`ReadvReader`的函数，它接受一个名为`tcpServer`的TCP服务器实例作为参数。`tcpServer`通过一个名为`MsgProcessor`的函数来处理接收到的数据，该函数将接收到的数据直接返回。

具体来说，这段代码创建了一个TCP服务器实例，然后使用该服务器实例来建立一个TCP连接。接着，使用一个名为`ReadvReader`的函数来从连接中读取数据。在函数内部，使用一个名为`NewReader`的函数创建一个新读取器实例，然后使用该实例从连接中读取数据。

接下来，使用一个名为`MergeBytes`的函数来合并数据和读取器之间的数据。最后，比较合并后的数据和读取器读取的数据，以验证它们是否相等。如果它们不相等，那么函数的行为就会失败，并输出错误信息。


```go
func TestReadvReader(t *testing.T) {
	tcpServer := &tcp.Server{
		MsgProcessor: func(b []byte) []byte {
			return b
		},
	}
	dest, err := tcpServer.Start()
	common.Must(err)
	defer tcpServer.Close() // nolint: errcheck

	conn, err := net.Dial("tcp", dest.NetAddr())
	common.Must(err)
	defer conn.Close() // nolint: errcheck

	const size = 8192
	data := make([]byte, 8192)
	common.Must2(rand.Read(data))

	var errg errgroup.Group
	errg.Go(func() error {
		writer := NewWriter(conn)
		mb := MergeBytes(nil, data)

		return writer.WriteMultiBuffer(mb)
	})

	defer func() {
		if err := errg.Wait(); err != nil {
			t.Error(err)
		}
	}()

	rawConn, err := conn.(*net.TCPConn).SyscallConn()
	common.Must(err)

	reader := NewReadVReader(conn, rawConn)
	var rmb MultiBuffer
	for {
		mb, err := reader.ReadMultiBuffer()
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}
		rmb, _ = MergeMulti(rmb, mb)
		if rmb.Len() == size {
			break
		}
	}

	rdata := make([]byte, size)
	SplitBytes(rmb, rdata)

	if r := cmp.Diff(data, rdata); r != "" {
		t.Fatal(r)
	}
}

```

# `common/buf/readv_unix.go`

这段代码定义了一个名为"buf"的包，其中包含一个名为"unixReader"的结构体，该结构体实现了一个非阻塞的文件读取器。

该代码的作用是创建一个可以读取文件的读取器实例，允许用户在使用完毕后能够自动关闭文件。它通过在内存中分配一块字节数组，将该数组初始化为要读取的数据，然后循环读取数组中的每个元素，将其存储在非阻塞读取器实例的"iovs"数组中。最后，将"iovs"数组中的所有元素都存储在"unixReader"实例的"iovs"数组中，从而实现了非阻塞文件读取。


```go
// +build illumos

package buf

import "golang.org/x/sys/unix"

type unixReader struct {
	iovs [][]byte
}

func (r *unixReader) Init(bs []*Buffer) {
	iovs := r.iovs
	if iovs == nil {
		iovs = make([][]byte, 0, len(bs))
	}
	for _, b := range bs {
		iovs = append(iovs, b.v)
	}
	r.iovs = iovs
}

```

该代码定义了一个名为 multiPhone器（multiPhone器）的接口，该接口包含两个方法：

1. func (r *unixReader) Read(fd uintptr) int32：这个方法接受一个文件描述符（fd）作为参数，然后调用 unix.Readv 函数从文件中读取数据，并存储到传入的 `r.iovs` 数组中。如果读取过程中出现错误，函数返回 -1。

2. func (r *unixReader) Clear()：这个方法清空了 `r.iovs` 数组，将所有数据都置为 0。

3. func newMultiReader() multiReader：这个方法返回一个名为 `multiPhone器` 的接口，该接口包含上述两个方法。

4. 最后，该代码没有定义一个具体的 `multiPhone器` 实例，但通过 `newMultiReader` 方法，可以创建多个实例并调用它们的方法。


```go
func (r *unixReader) Read(fd uintptr) int32 {
	n, e := unix.Readv(int(fd), r.iovs)
	if e != nil {
		return -1
	}
	return int32(n)
}

func (r *unixReader) Clear() {
	r.iovs = r.iovs[:0]
}

func newMultiReader() multiReader {
	return &unixReader{}
}

```

# `common/buf/readv_windows.go`

这段代码定义了一个名为`buf`的包，包含了一个名为`windowsReader`的结构体。

`windowsReader`结构体包含一个名为`Init`的函数，该函数接收一个包含`BS`类型的切片（`[]*Buffer`）作为参数。函数在初始化时创建一个空的`WSABuf`数组，然后循环遍历输入的`BS`切片，为每个元素分配一个`WSABuf`实例并将其添加到内部缓冲区数组中。最后，函数返回这个内部缓冲区数组。

这个`windowsReader`结构体可能是用于在Windows系统上读取文件内容的。`Init`函数用于初始化缓冲区数组，并在每个`WSABuf`实例都被分配给缓冲区后，将缓冲区数组的大小设置为输入的`BS`切片的大小。


```go
package buf

import (
	"syscall"
)

type windowsReader struct {
	bufs []syscall.WSABuf
}

func (r *windowsReader) Init(bs []*Buffer) {
	if r.bufs == nil {
		r.bufs = make([]syscall.WSABuf, 0, len(bs))
	}
	for _, b := range bs {
		r.bufs = append(r.bufs, syscall.WSABuf{Len: uint32(Size), Buf: &b.v[0]})
	}
}

```

该代码的作用是读取一个名为 "windowsReader" 的输入流 "r"，并对其进行清理和重置。

函数 "Clear()" 清理的作用是遍历输入流中的所有缓冲区，并将其中的数据复制到输入流中创建的新的缓冲区中。这样可以确保输入流中的所有数据都被保留，即使正在从其他缓冲区中读取数据。

函数 "Read()" 则允许从输入流中读取数据，并将其存储在 "r.bufs" 数组中的第一个元素中。函数使用了 "syscall.WSARecv()" 函数来读取数据，并将其存储在 "r.bufs" 数组中。如果函数存在任何错误，它将返回一个负值，因此需要在调用它之前进行错误处理。

最后，函数 "windowsReader.Read()" 的作用是读取数据并返回读取的数据字节数。


```go
func (r *windowsReader) Clear() {
	for idx := range r.bufs {
		r.bufs[idx].Buf = nil
	}
	r.bufs = r.bufs[:0]
}

func (r *windowsReader) Read(fd uintptr) int32 {
	var nBytes uint32
	var flags uint32
	err := syscall.WSARecv(syscall.Handle(fd), &r.bufs[0], uint32(len(r.bufs)), &nBytes, &flags, nil, nil)
	if err != nil {
		return -1
	}
	return int32(nBytes)
}

```

这段代码定义了一个名为 `newMultiReader` 的函数，它返回一个名为 `multiReader` 的对象。

在这个函数中，`new` 操作符返回一个指向 `windowsReader` 的引用。然后，使用 `return` 关键字返回这个引用，这样就可以将 `windowsReader` 赋值给 `multiReader` 了。

换句话说，这个函数的作用是创建一个 `multiReader` 对象，它包含多个 `windowsReader`。当你调用 `newMultiReader` 时，它会创建一个新的 `multiReader` 对象，然后将多个 `windowsReader` 赋值给它。


```go
func newMultiReader() multiReader {
	return new(windowsReader)
}

```

# `common/buf/writer.go`

这段代码定义了一个名为“buf”的包，该包包含了一些与缓冲区操作相关的函数。

首先，导入了两个名为“io”和“net”的包，这两个包提供了输入和输出操作所需的接口。

接着，导入了“buf”包自己的一个名为“BufferToBytesWriter”的类型，这个类型有一个“io.Writer”接口和一个“[]byte”类型的“cache”字段，表示缓存区。

然后，定义了一个名为“BufferToBytesWriter”的结构体，该结构体实现了“io.Writer”接口，因为它有一个“Writer”字段和一个“cache”字段，其中“Writer”字段提供了一个写入数据到缓冲区的接口，“cache”字段是一个由“[]byte”组成的缓冲区，用于存储已经写入的数据。

最后，定义了一个名为“main”的函数，该函数没有参数并且返回一个非负整数，表示缓冲区操作的正确性。


```go
package buf

import (
	"io"
	"net"
	"sync"

	"v2ray.com/core/common"
	"v2ray.com/core/common/errors"
)

// BufferToBytesWriter is a Writer that writes alloc.Buffer into underlying writer.
type BufferToBytesWriter struct {
	io.Writer

	cache [][]byte
}

```

该函数实现了 `Writer` 接口，用于将多个缓冲区（`MultiBuffer`）的数据写入到目标缓冲区（`BufferToBytesWriter`）中。它接受一个 `MultiBuffer` 参数，并在写入数据之前或之后释放它。

具体来说，该函数的实现包括以下步骤：

1. 检查缓冲区的大小，如果为空，则返回。
2. 如果缓冲区只有一个元素，则将其字节数组全部写入目标缓冲区，并返回。
3. 如果缓冲区的大小大于 `cap` 数组的大小，则扩充数组。
4. 遍历缓冲区的所有元素，并将它们字节数组添加到目标缓冲区。
5. 如果缓冲区已经写入目标缓冲区，则检查是否已经写入了所有数据，如果是，则返回前 `int32` 字节的数据。
6. 返回 ` nil`，表示没有错误发生。

该函数的实现使用了 Net `buf` 包，它提供了一个高性能、可扩展的缓冲区操作库，可以方便地处理各种缓冲区操作。


```go
// WriteMultiBuffer implements Writer. This method takes ownership of the given buffer.
func (w *BufferToBytesWriter) WriteMultiBuffer(mb MultiBuffer) error {
	defer ReleaseMulti(mb)

	size := mb.Len()
	if size == 0 {
		return nil
	}

	if len(mb) == 1 {
		return WriteAllBytes(w.Writer, mb[0].Bytes())
	}

	if cap(w.cache) < len(mb) {
		w.cache = make([][]byte, 0, len(mb))
	}

	bs := w.cache
	for _, b := range mb {
		bs = append(bs, b.Bytes())
	}

	defer func() {
		for idx := range bs {
			bs[idx] = nil
		}
	}()

	nb := net.Buffers(bs)

	for size > 0 {
		n, err := nb.WriteTo(w.Writer)
		if err != nil {
			return err
		}
		size -= int32(n)
	}

	return nil
}

```

这段代码定义了一个名为 `BufferedWriter` 的 struct 类型，该类型使用一个内部缓冲区 `Buffer`，并且可以提供 `BufferToBytesWriter` 接口的函数 `ReadFrom`。

`BufferedWriter` 的 `ReadFrom` 函数接受一个 `Reader` 对象作为参数，然后返回一个整数 `sc` 和一个错误 `err`。`sc` 是一个 `SizeCounter` 类型，用于跟踪读取的字节数，而 `err` 是传递给 `ReadFrom` 的错误对象。

`BufferedWriter` 的 `CountSize` 函数用于计算 `sc` 的值，该函数将 `Reader` 对象传递给 `BufferedWriter` 的 `ReadFrom` 函数，并返回一个指向 `sc` 变量的新指针。

`BufferedWriter` 的 `Buffered` 函数返回一个布尔值，表示缓冲区是否已满。如果缓冲区已满，则返回 `true`，否则返回 `false`。

`BufferedWriter` 的 `ReadFrom` 函数实现了一个 `BufferToBytesWriter` 接口的函数，用于从 `Reader` 对象中读取字节。该函数创建一个新的 `Reader` 对象，将其传递给 `BufferedWriter` 的 `ReadFrom` 函数，并将读取的字节数存储在 `sc` 变量中。如果发生了错误，该函数将返回一个非 `nil` 的错误对象。

由于 `BufferToBytesWriter` 实现了 `io.ReaderFrom` 接口，因此 `BufferedWriter` 可以直接使用 `ReaderFrom` 函数，而不需要自己实现 `ReadFrom` 函数。


```go
// ReadFrom implements io.ReaderFrom.
func (w *BufferToBytesWriter) ReadFrom(reader io.Reader) (int64, error) {
	var sc SizeCounter
	err := Copy(NewReader(reader), w, CountSize(&sc))
	return sc.Size, err
}

// BufferedWriter is a Writer with internal buffer.
type BufferedWriter struct {
	sync.Mutex
	writer   Writer
	buffer   *Buffer
	buffered bool
}

```

这段代码定义了一个名为`NewBufferedWriter`的函数，该函数接受一个`Writer`类型的参数，并返回一个指向`BufferedWriter`类型的引用。

`NewBufferedWriter`函数的作用是在不创建缓冲区的情况下创建一个新的`BufferedWriter`。它将`Writer`类型的参数和一个空的缓冲区作为构造函数的参数，并在函数内部将`writer`字段设置为传入的`Writer`，将`buffer`字段设置为一个新的缓冲区，并将`buffered`字段设置为`true`，表示创建了一个新的缓冲区。

函数有两个方法：

* `WriteByte`：实现了`io.ByteWriter.`该方法接受一个`byte`类型的参数，并返回一个`Write`方法的错误。它通过调用`writer`字段的`Write`方法来写入数据到缓冲区，并返回一个非错误类型的值。
* `Write`：实现了`io.Writer.`该方法与`WriteByte`类似，但是将`Write`方法接受的数据类型指定为`[]byte`而不是`io.ByteWriter.`该方法返回一个错误类型的值。它调用`writer`字段的`Write`方法来写入数据到缓冲区，并返回一个非错误类型的值。


```go
// NewBufferedWriter creates a new BufferedWriter.
func NewBufferedWriter(writer Writer) *BufferedWriter {
	return &BufferedWriter{
		writer:   writer,
		buffer:   New(),
		buffered: true,
	}
}

// WriteByte implements io.ByteWriter.
func (w *BufferedWriter) WriteByte(c byte) error {
	return common.Error2(w.Write([]byte{c}))
}

// Write implements io.Writer.
```

该函数的作用是向一个缓冲写入器(BufferedWriter)中写入一个字节数组(b []byte)。函数的实现包括以下步骤：

1. 如果字节数组为空，函数返回0并抛出错误。

2. 在函数内部，首先检查缓冲区是否已充满，如果是，则调用一个 writer.io.Writer 类型的参数，使用 Write 函数向字节数组写入数据。如果缓冲区没有充满，则遍历缓冲区并使用 New 函数创建一个新的缓冲区，使用 Write 函数将数据写入到缓冲区中。

3. 在循环中，首先检查缓冲区是否为空，如果是，则调用 New 函数创建一个新的缓冲区并将其赋值为 b。

4. 在循环中，使用 for 循环遍历字节数组，如果当前缓冲区有数据，则使用 Write 函数向当前缓冲区写入数据。

5. 如果当前缓冲区已充满或者当前循环中的数据写入失败，则调用一个 flushInternal 函数将当前缓冲区中的数据刷写到磁盘上，并返回前一次返回的错误。

6. 最后，返回总共写入的字节数并检查是否成功。


```go
func (w *BufferedWriter) Write(b []byte) (int, error) {
	if len(b) == 0 {
		return 0, nil
	}

	w.Lock()
	defer w.Unlock()

	if !w.buffered {
		if writer, ok := w.writer.(io.Writer); ok {
			return writer.Write(b)
		}
	}

	totalBytes := 0
	for len(b) > 0 {
		if w.buffer == nil {
			w.buffer = New()
		}

		nBytes, err := w.buffer.Write(b)
		totalBytes += nBytes
		if err != nil {
			return totalBytes, err
		}
		if !w.buffered || w.buffer.IsFull() {
			if err := w.flushInternal(); err != nil {
				return totalBytes, err
			}
		}
		b = b[nBytes:]
	}

	return totalBytes, nil
}

```

该函数实现了 `Writer` 接口，用于将多个 `MultiBuffer` 对象写入到缓冲区中。它接受一个 `MultiBuffer` 对象作为参数，并返回一个错误。以下是函数的实现细节：

1. 如果传递的 `MultiBuffer` 是空的，那么立即返回 `nil`，因为空 `MultiBuffer` 不会对任何错误产生任何影响。
2. 在函数开始时，对 `BufferedWriter` 类型的 `Writer` 进行锁定，以确保在函数的任何部分都能够对 `MultiBuffer` 对象进行写入。
3. 如果 `BufferedWriter` 当前正在写入数据，那么直接调用 `writer` 实例，将 `MultiBuffer` 对象写入到缓冲区中。
4. 创建一个名为 `reader` 的 `MultiBufferContainer` 对象，将其设置为当前正在读取的 `MultiBuffer` 对象，并使用循环从 `MultiBuffer` 对象中读取数据。
5. 如果 `BufferedWriter` 的缓冲区为空，那么调用 `flushInternal` 函数将缓冲区内容写入到文件中，并返回一个错误。

整个函数的目的是将多个 `MultiBuffer` 对象写入到一个缓冲区中，并在写入完成后关闭文件。它确保了 `BufferedWriter` 具有足够的数据来写入 `MultiBuffer` 对象，即使缓冲区为空。


```go
// WriteMultiBuffer implements Writer. It takes ownership of the given MultiBuffer.
func (w *BufferedWriter) WriteMultiBuffer(b MultiBuffer) error {
	if b.IsEmpty() {
		return nil
	}

	w.Lock()
	defer w.Unlock()

	if !w.buffered {
		return w.writer.WriteMultiBuffer(b)
	}

	reader := MultiBufferContainer{
		MultiBuffer: b,
	}
	defer reader.Close()

	for !reader.MultiBuffer.IsEmpty() {
		if w.buffer == nil {
			w.buffer = New()
		}
		common.Must2(w.buffer.ReadFrom(&reader))
		if w.buffer.IsFull() {
			if err := w.flushInternal(); err != nil {
				return err
			}
		}
	}

	return nil
}

```

这段代码定义了一个名为 `BufferedWriter` 的类，它实现了 `io.Writer` 接口，用于向底层 writer 中的缓冲内容进行写入。

该类包含两个方法：`Flush` 和 `flushInternal`。

`Flush` 方法的作用是，通过获取 `BufferedWriter` 类型的 `w` 实例的锁，然后调用 `w.flushInternal` 方法将缓冲内容写入底层 writer。如果缓冲内容为空，该方法返回一个 `nil`。

`flushInternal` 方法的作用是，获取 `BufferedWriter` 类型的 `w` 实例的 `writer` 字段，如果它实现了 `io.Writer` 接口，那么它尝试调用 `WriteAllBytes` 方法将 `w.buffer` 缓冲区中的内容写入底层 writer。如果 `writer` 不是 `io.Writer`，那么它尝试将 `w.buffer` 缓冲区中的内容一次性写入底层 writer。如果 `w.writer` 实现了 `io.Writer` 接口，那么 `flushInternal` 的返回值就是 `WriteAllBytes` 方法的返回值。如果 `w.writer` 不是 `io.Writer`，那么该方法会尝试从底层 writer 中的缓冲区读取数据，并将其写入 `w.buffer` 缓冲区，直到缓冲区为空或者底层 writer 已经没有数据可以写入。


```go
// Flush flushes buffered content into underlying writer.
func (w *BufferedWriter) Flush() error {
	w.Lock()
	defer w.Unlock()

	return w.flushInternal()
}

func (w *BufferedWriter) flushInternal() error {
	if w.buffer.IsEmpty() {
		return nil
	}

	b := w.buffer
	w.buffer = nil

	if writer, ok := w.writer.(io.Writer); ok {
		err := WriteAllBytes(writer, b.Bytes())
		b.Release()
		return err
	}

	return w.writer.WriteMultiBuffer(MultiBuffer{b})
}

```

这段代码定义了一个名为 `BufferedWriter` 的类，用于 buffering writes。内部缓冲区是否被使用可以通过 `SetBuffered` 函数进行设置，如果设置为 `true`，则调用 `Flush()` 函数清除缓冲区；如果设置为 `false`，则不会调用 `Flush()` 函数。

`ReadFrom` 函数使用一个 `io.ReaderFrom` 类型的参数，它从缓冲区中读取数据。如果缓冲区未被设置为 `true`，则会先调用 `SetBuffered` 函数将缓冲区中的数据清空，然后调用 `ReadFrom` 函数从 `Reader` 类型对应的通道中读取数据。

如果设置为 `true`，则不会调用 `SetBuffered` 函数，直接从 `Reader` 类型对应的通道中读取数据。


```go
// SetBuffered sets whether the internal buffer is used. If set to false, Flush() will be called to clear the buffer.
func (w *BufferedWriter) SetBuffered(f bool) error {
	w.Lock()
	defer w.Unlock()

	w.buffered = f
	if !f {
		return w.flushInternal()
	}
	return nil
}

// ReadFrom implements io.ReaderFrom.
func (w *BufferedWriter) ReadFrom(reader io.Reader) (int64, error) {
	if err := w.SetBuffered(false); err != nil {
		return 0, err
	}

	var sc SizeCounter
	err := Copy(NewReader(reader), w, CountSize(&sc))
	return sc.Size, err
}

```

这段代码定义了一个名为`Close`的函数，该函数实现了`io.Closable`接口的`Close`方法。

该函数的行为是在缓冲区Writer上调用`Flush`函数并关闭写入时，如果出现错误，则返回错误。如果`Flush`函数正常运行并且关闭写入时没有错误，则调用`common.Close`函数关闭写入缓冲区。

接下来定义了一个名为`SequentialWriter`的结构体，表示一个将多个缓冲区 sequentially 写入底层`io.Writer`的Writer。

`WriteMultiBuffer`函数是一个实现了`Writer`接口的函数，用于将多个缓冲区写入底层`io.Writer`。此函数接受一个`MultiBuffer`类型的参数，代表一个写入缓冲区。函数首先调用一个名为`WriteMultiBuffer`的内部函数，将`MultiBuffer`对象传递给该函数，然后使用`ReleaseMulti`函数释放`MultiBuffer`对象。最后，函数返回写入缓冲区的错误。

在上述代码中，还定义了一个名为`Close`的函数，该函数实现了`io.Closable`接口的`Close`方法。该函数接受一个`Writer`对象和一个`Closable`接口类型的参数。函数首先调用`Flush`函数并关闭写入时，如果出现错误，则返回错误。如果`Flush`函数正常运行并且关闭写入时没有错误，则调用`common.Close`函数关闭写入缓冲区。


```go
// Close implements io.Closable.
func (w *BufferedWriter) Close() error {
	if err := w.Flush(); err != nil {
		return err
	}
	return common.Close(w.writer)
}

// SequentialWriter is a Writer that writes MultiBuffer sequentially into the underlying io.Writer.
type SequentialWriter struct {
	io.Writer
}

// WriteMultiBuffer implements Writer.
func (w *SequentialWriter) WriteMultiBuffer(mb MultiBuffer) error {
	mb, err := WriteMultiBuffer(w.Writer, mb)
	ReleaseMulti(mb)
	return err
}

```

这段代码定义了一个名为noOpWriter的类型，该类型有一个名为WriteMultiBuffer的函数和一个名为Write的函数和一个名为ReadFrom的函数。

noOpWriter是一个没有实现的类型，它没有提供任何实际的操作。它的类型参数noOpWriter是一个byte类型的变量，但这个变量没有被定义，我们无法对其进行使用。

下面来具体解释一下函数的行为：

1. WriteMultiBuffer函数接收一个名为MultiBuffer的参数，并尝试将其复用。

2. Write函数接收一个字节切片（[]byte）。函数首先创建一个名为b的变量，并使用io.Reader从该Reader中读取字节。然后，函数使用noOpWriter的WriteMultiBuffer函数将多个字节写入MultiBuffer。

3. ReadFrom函数接收一个io.Reader，并从该Reader中读取字节。函数创建一个名为b的变量，并使用noOpWriter的ReadFrom函数从Reader中读取字节。然后，函数使用Write函数将读取的字节写入noOpWriter。最后，函数使用int64类型将读取的字节大小返回。

noOpWriter函数的作用是定义了如何向MultiBuffer写入数据，以及如何从Reader中读取数据，但是它们没有实际的内容。


```go
type noOpWriter byte

func (noOpWriter) WriteMultiBuffer(b MultiBuffer) error {
	ReleaseMulti(b)
	return nil
}

func (noOpWriter) Write(b []byte) (int, error) {
	return len(b), nil
}

func (noOpWriter) ReadFrom(reader io.Reader) (int64, error) {
	b := New()
	defer b.Release()

	totalBytes := int64(0)
	for {
		b.Clear()
		_, err := b.ReadFrom(reader)
		totalBytes += int64(b.Len())
		if err != nil {
			if errors.Cause(err) == io.EOF {
				return totalBytes, nil
			}
			return totalBytes, err
		}
	}
}

```

这段代码定义了两个变量 `Discard` 和 `DiscardBytes`，它们都是 `Writer` 类型。

`noOpWriter(0)` 是一个接受 `Writer` 类型作为参数的函数，返回一个空的 `Writer`。这个空的 `Writer` 并不会做任何写入操作，可以用来确保在某些操作成功后，不会输出任何内容。

`noOpWriter(0)` 是另一个接受 `Writer` 类型作为参数的函数，返回一个接受所有写入内容的 `Writer`。这个接受所有写入内容的 `Writer` 可以在需要时写入数据，但不会输出任何内容。

这两个变量都被声明为 `var` 类型，意味着它们可以被赋值。不过，在这段代码中，它们都被赋值为 `noOpWriter(0)`，也就是一个空的 `Writer`。


```go
var (
	// Discard is a Writer that swallows all contents written in.
	Discard Writer = noOpWriter(0)

	// DiscardBytes is an io.Writer that swallows all contents written in.
	DiscardBytes io.Writer = noOpWriter(0)
)

```