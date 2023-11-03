# v2ray-core源码解析 18

# `common/buf/buffer_test.go`

这段代码的主要目的是测试一个名为“BufferClear”的函数，它接收一个字节缓冲区（Buffer）并对其内容进行写入和清除操作。通过编写一个名为“TestBufferClear”的函数，我们可以更轻松地编写集成测试来确保代码的正确性。

具体来说，这段代码实现了一个简单的测试用例，即创建一个字节缓冲区，向其中写入一个字符串，然后清空缓冲区并输出缓冲区剩余的字节数。以下是这段代码的主要步骤：

1. 导入所需的包：通过导入“buf_test”、“crypto/rand”、“testing”和“github.com/google/go-cmp/cmp”这些包，确保了测试所需的依赖。

2. 创建一个名为“buffer”的字节缓冲区。

3. 通过调用“New”函数创建一个新的字节缓冲区。

4. 通过调用“Release”函数确保缓冲区在使用完毕后及时释放资源。

5. 写入一个字符串到缓冲区。

6. 使用“cmp.Diff”函数比较缓冲区和传入的字符串之间的差异。如果差异较大，则说明缓冲区的写入操作可能存在问题。

7. 使用“clear”函数清空缓冲区。

8. 通过调用“Len”函数检查缓冲区是否还包含任何元素，如果是，则说明缓冲区已经被清空。

9. 通过调用“Error”函数输出任何错误信息。

需要注意的是，这段代码并没有实现实际的测试用例，只是一个简单的示例，用于说明如何编写测试用例来验证缓冲区的正确性。在实际应用中，需要根据具体需求编写更具体的测试用例。


```go
package buf_test

import (
	"bytes"
	"crypto/rand"
	"testing"

	"github.com/google/go-cmp/cmp"
	"v2ray.com/core/common"
	. "v2ray.com/core/common/buf"
)

func TestBufferClear(t *testing.T) {
	buffer := New()
	defer buffer.Release()

	payload := "Bytes"
	buffer.Write([]byte(payload))
	if diff := cmp.Diff(buffer.Bytes(), []byte(payload)); diff != "" {
		t.Error(diff)
	}

	buffer.Clear()
	if buffer.Len() != 0 {
		t.Error("expect 0 length, but got ", buffer.Len())
	}
}

```

这两个函数是对待测函数测试的代码。

第一个函数 `TestBufferIsEmpty` 创建了一个字符缓冲区 `buffer`, 并使用 `New()` 函数创建一个新的空字符缓冲区。然后使用 `Release()` 函数将缓冲区分配给内存回收器，最后测试 `buffer.IsEmpty()` 是否为真，如果是，则说明创建了一个非空缓冲区，否则会输出 "expect empty buffer, but not"。

第二个函数 `TestBufferString` 创建了一个字符缓冲区 `buffer`, 并使用 `New()` 函数创建一个新的空字符缓冲区。然后使用 `WriteString(payload)` 函数将字符串 `payload` 写入缓冲区，并使用 `Release()` 函数将缓冲区分配给内存回收器。接着测试 `buffer.String()` 是否等于 `payload`，如果是，则输出 "expect buffer content as " `payload` "，否则会输出 "expect buffer content as <buffer> but actually <buffer>".


```go
func TestBufferIsEmpty(t *testing.T) {
	buffer := New()
	defer buffer.Release()

	if buffer.IsEmpty() != true {
		t.Error("expect empty buffer, but not")
	}
}

func TestBufferString(t *testing.T) {
	buffer := New()
	defer buffer.Release()

	const payload = "Test String"
	common.Must2(buffer.WriteString(payload))
	if buffer.String() != payload {
		t.Error("expect buffer content as ", payload, " but actually ", buffer.String())
	}
}

```

以上代码是用于测试一个名为`TestBufferByte`的函数。通过观察代码可以发现，该函数的主要目的是测试不同输入参数下，对`New`和`StackNew`函数的使用，以及测试相关的错误条件。

具体来说，函数`TestBufferByte`接受三个参数：

1. 一个`testing.T`类型的接收者（指针，但并非指针所指向的实参）。
2. 一个函数体，该函数体内部又包含两个`New`函数和一个`Release`函数。
3. 在函数体内部，分别传入一个`New`参数和一个`StackNew`参数。

接下来，通过编写一系列测试用例，来测试上述参数下的函数行为。这些测试用例会发送一系列不同的输入参数，并检查函数的输出是否符合预期。如果输出结果与预期不符，则会产生错误信息，并调用函数内部的错误处理逻辑，以提醒程序员关注相关代码。


```go
func TestBufferByte(t *testing.T) {
	{
		buffer := New()
		common.Must(buffer.WriteByte('m'))
		if buffer.String() != "m" {
			t.Error("expect buffer content as ", "m", " but actually ", buffer.String())
		}
		buffer.Release()
	}
	{
		buffer := StackNew()
		common.Must(buffer.WriteByte('n'))
		if buffer.String() != "n" {
			t.Error("expect buffer content as ", "n", " but actually ", buffer.String())
		}
		buffer.Release()
	}
	{
		buffer := StackNew()
		common.Must2(buffer.WriteString("HELLOWORLD"))
		if b := buffer.Byte(5); b != 'W' {
			t.Error("unexpected byte ", b)
		}

		buffer.SetByte(5, 'M')
		if buffer.String() != "HELLOMORLD" {
			t.Error("expect buffer content as ", "n", " but actually ", buffer.String())
		}
		buffer.Release()
	}
}
```

这段代码是一个名为 `TestBufferResize` 的测试函数，旨在测试一个名为 `Buffer` 的字符串缓冲区。

函数的作用是模拟在向字符串缓冲区中写入字符串后，读取该缓冲区中所有字符并将其打印出来，然后测试缓冲区的操作。

具体来说，代码首先创建一个名为 `buffer` 的字符串缓冲区，并使用 `Release()` 方法将其关闭，以确保在函数结束时释放资源。

接下来，代码使用 `WriteString()` 方法将常数 `payload` 写入缓冲区中。然后，使用 `Must2()` 函数保证 `WriteString()` 方法成功写入字符串。

接着，代码使用 `Resize()` 方法对缓冲区进行大小调整。对于此，`Must2()` 函数确保 `Resize()` 方法成功实现了缓冲区大小调整，即使缓冲区为空或已满。

在 `Resize()` 方法中，代码使用 `Len()` 方法获取当前缓冲区的长度，然后使用 `Int()` 函数将其转换为整数类型。接着，代码尝试使用 `Resize()` 方法将缓冲区大小调整为 `int32` 类型，即尝试将缓冲区大小调整为 `len(payload)` 的两倍。

最后，代码使用 `String()` 方法尝试从缓冲区中读取所有字符，并将其打印出来。如果缓冲区中的字符串与 `payload` 不相等，则函数会输出错误信息。如果缓冲区中的字符串为空，则函数也会输出错误信息。


```go
func TestBufferResize(t *testing.T) {
	buffer := New()
	defer buffer.Release()

	const payload = "Test String"
	common.Must2(buffer.WriteString(payload))
	if buffer.String() != payload {
		t.Error("expect buffer content as ", payload, " but actually ", buffer.String())
	}

	buffer.Resize(-6, -3)
	if l := buffer.Len(); int(l) != 3 {
		t.Error("len error ", l)
	}

	if s := buffer.String(); s != "Str" {
		t.Error("unexpect buffer ", s)
	}

	buffer.Resize(int32(len(payload)), 200)
	if l := buffer.Len(); int(l) != 200-len(payload) {
		t.Error("len error ", l)
	}
}

```

此代码是针对一个名为 "TestBufferSlice" 的测试函数。函数接受一个测试框架 "t"，并在其中执行以下操作：

1. 创建一个名为 "b" 的缓冲区对象并将其初始化为空字符串。
2. 使用 "Write" 函数将一个字符串（'abcd'）写入缓冲区。
3. 从缓冲区中获取一个字符串，并将其转换为字节数组。
4. 使用 "BytesFrom" 函数将从缓冲区中获取的字符串转换为字节数组的位置，并返回该位置的字符数组。
5. 比较从缓冲区获取的字符数组和给定字符串之间的差异，并输出差异。
6. 重复步骤 1-5 中的操作，分别将字符 'a' 和 'b' 放入缓冲区中。

函数的作用是测试缓冲区数组在不同操作下的行为，包括写入、从缓冲区中获取字符串、将字符串转换为字节数组等。


```go
func TestBufferSlice(t *testing.T) {
	{
		b := New()
		common.Must2(b.Write([]byte("abcd")))
		bytes := b.BytesFrom(-2)
		if diff := cmp.Diff(bytes, []byte{'c', 'd'}); diff != "" {
			t.Error(diff)
		}
	}

	{
		b := New()
		common.Must2(b.Write([]byte("abcd")))
		bytes := b.BytesTo(-2)
		if diff := cmp.Diff(bytes, []byte{'a', 'b'}); diff != "" {
			t.Error(diff)
		}
	}

	{
		b := New()
		common.Must2(b.Write([]byte("abcd")))
		bytes := b.BytesRange(-3, -1)
		if diff := cmp.Diff(bytes, []byte{'b', 'c'}); diff != "" {
			t.Error(diff)
		}
	}
}

```

这段代码是用于测试一个名为"TestBufferReadFullFrom"的函数。该函数使用"testing.T"作为参数，因此在函数内部使用了断言语句。

具体来说，该函数的作用是创建一个长度为1024的字节数组"payload"，并使用"rand.Read"生成一个随机字节，然后将其赋值给字节数组。接着，该函数创建一个名为"reader"的读取器对象，并使用"bytes.NewReader"方法将字节数组作为参数传递给该读取器，然后使用"New"函数创建一个名为"b"的空字符串对象，并使用"reader"读取器从字节数组中读取1024个字节，并将其赋值给"b"对象。最后，该函数使用"cmp.Diff"函数比较"payload"和"b.Bytes"字节数组之间的差异，如果两者之间的差异为空字符串，则说明函数正确，否则会输出差异信息并错误提示。

因此，该函数的作用是测试一个名为"TestBufferReadFullFrom"的函数是否正确地读取一个长度为1024的字节数组，并返回读取的字节数。


```go
func TestBufferReadFullFrom(t *testing.T) {
	payload := make([]byte, 1024)
	common.Must2(rand.Read(payload))

	reader := bytes.NewReader(payload)
	b := New()
	n, err := b.ReadFullFrom(reader, 1024)
	common.Must(err)
	if n != 1024 {
		t.Error("expect reading 1024 bytes, but actually ", n)
	}

	if diff := cmp.Diff(payload, b.Bytes()); diff != "" {
		t.Error(diff)
	}
}

```

这段代码定义了两个名为"BenchmarkNewBuffer"和"BenchmarkNewBufferStack"的函数，它们都是testing包中的测试函数。这两个函数都是使用"*testing.B"作为参数传递给func类型的函数。

函数内部创建了一个缓冲区(buffer)，然后执行一系列操作来测试这些缓冲区的行为。下面是具体的实现细节：

1.func BenchmarkNewBuffer(b *testing.B) {
这个函数接收一个名为"b"的参数。它使用for循环来创建一个缓冲区，并执行两次Release()函数来释放缓冲区。

2.func BenchmarkNewBufferStack(b *testing.B) {
这个函数与func BenchmarkNewBuffer(b *testing.B)函数非常类似，只是创建缓冲区的方式使用的是StackNew()函数，而不是New()函数。

3.func BenchmarkWrite2(b *testing.B) {
这个函数创建了一个名为"buffer"的缓冲区，并使用Write()函数将两个字节的数据写入缓冲区。然后，它使用Clear()函数清空缓冲区。

函数的作用是测试缓冲区的Release()函数是否能够正常工作，并保证每次测试都是独立的。


```go
func BenchmarkNewBuffer(b *testing.B) {
	for i := 0; i < b.N; i++ {
		buffer := New()
		buffer.Release()
	}
}

func BenchmarkNewBufferStack(b *testing.B) {
	for i := 0; i < b.N; i++ {
		buffer := StackNew()
		buffer.Release()
	}
}

func BenchmarkWrite2(b *testing.B) {
	buffer := New()

	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		_, _ = buffer.Write([]byte{'a', 'b'})
		buffer.Clear()
	}
}

```

这两段代码都是用来测试一个名为BenchmarkWrite的函数。该函数接受两个参数，一个是*testing.B类型的参数，另一个是空字符串""。函数内部有一个循环，该循环使用了*testing.B参数提供的计时器，在每次循环中调用BenchmarkWrite函数本身，并传入不同的测试数据。

第一个函数BenchmarkWrite8函数的作用是测试字符串"a、b、c、d、e、f、g、h"在缓冲区中的存储和清除。具体来说，该函数创建一个长度为8的字符串缓冲区，然后循环调用缓冲区的Write函数，每次将一个"a"、"b"、"c"、"d"、"e"、"f"、"g"、"h"的字符串数据写入缓冲区。接着，函数清空缓冲区并再次调用Write函数。这样，就可以测试字符串在缓冲区中存储和清除的能力。

第二个函数BenchmarkWrite32函数的作用是测试字符串"a、b、c、d、e、f、g、h"在缓冲区中的存储和清除，但使用了长度为32的负载。具体来说，该函数创建一个长度为32的字符串缓冲区，然后循环调用缓冲区的Write函数，每次将一个"a"、"b"、"c"、"d"、"e"、"f"、"g"、"h"的字符串数据写入缓冲区。接着，函数清空缓冲区并再次调用Write函数。这样，就可以测试字符串在缓冲区中存储和清除的能力，不过此时缓冲区的大小是固定的32字节，无法测试缓冲区大小为8KB或16MB等更大的情况。


```go
func BenchmarkWrite8(b *testing.B) {
	buffer := New()

	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		_, _ = buffer.Write([]byte{'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'})
		buffer.Clear()
	}
}

func BenchmarkWrite32(b *testing.B) {
	buffer := New()
	payload := make([]byte, 32)
	rand.Read(payload)

	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		_, _ = buffer.Write(payload)
		buffer.Clear()
	}
}

```

这两段代码是在测试框架中定义的函数，主要作用是向一个缓冲区（buffer）中输出一系列字符。`BenchmarkWriteByte2`函数主要针对字节数据类型，而`BenchmarkWriteByte8`函数主要针对字符数据类型。

在`BenchmarkWriteByte2`函数中，首先创建了一个空缓冲区（buffer）。接着，使用一个循环来不断地向缓冲区中输出字符。在循环体内，首先调用`buffer.WriteByte('a')`方法向缓冲区中输出字符'a'，然后调用`buffer.WriteByte('b')`方法向缓冲区中输出字符'b'，最后调用`buffer.Clear()`方法清空缓冲区。这样，就完成了一次循环。

在`BenchmarkWriteByte8`函数中，创建了一个空缓冲区（buffer）。然后，使用一个循环来不断地向缓冲区中输出字符。在循环体内，首先调用`buffer.WriteByte('a')`方法向缓冲区中输出字符'a'，然后调用`buffer.WriteByte('b')`方法向缓冲区中输出字符'b'，接着调用`buffer.WriteByte('c')`方法向缓冲区中输出字符'c'，然后调用`buffer.WriteByte('d')`方法向缓冲区中输出字符'd'，接着调用`buffer.WriteByte('e')`方法向缓冲区中输出字符'e'，最后调用`buffer.WriteByte('f')`方法向缓冲区中输出字符'f'，然后调用`buffer.WriteByte('g')`方法向缓冲区中输出字符'g'，最后调用`buffer.WriteByte('h')`方法向缓冲区中输出字符'h'，最后调用`buffer.Clear()`方法清空缓冲区。这样，同样完成了一次循环。


```go
func BenchmarkWriteByte2(b *testing.B) {
	buffer := New()

	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		_ = buffer.WriteByte('a')
		_ = buffer.WriteByte('b')
		buffer.Clear()
	}
}

func BenchmarkWriteByte8(b *testing.B) {
	buffer := New()

	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		_ = buffer.WriteByte('a')
		_ = buffer.WriteByte('b')
		_ = buffer.WriteByte('c')
		_ = buffer.WriteByte('d')
		_ = buffer.WriteByte('e')
		_ = buffer.WriteByte('f')
		_ = buffer.WriteByte('g')
		_ = buffer.WriteByte('h')
		buffer.Clear()
	}
}

```

# `common/buf/copy.go`

这段代码定义了一个名为"buf"的包，其中包含了一些定义、函数和类型声明。

首先，导入了一些标准库中的函数和类型，包括"io"(用于输入/输出)、"time"(用于获取当前时间)、"v2ray.com/core/common/errors"(用于错误处理)、"v2ray.com/core/common/signal"(用于信号处理)。

然后，定义了一个名为"dataHandler"的函数类型，该类型表示一个处理"MultiBuffer"(多缓冲协程)的函数。

接着，定义了一个名为"copyHandler"的 struct类型，该类型包含一个名为"onData"的数组，该数组包含一个处理"MultiBuffer"的函数。

最后，定义了一些常量和函数，但没有函数的实现，因此无法使用。


```go
package buf

import (
	"io"
	"time"

	"v2ray.com/core/common/errors"
	"v2ray.com/core/common/signal"
)

type dataHandler func(MultiBuffer)

type copyHandler struct {
	onData []dataHandler
}

```

这段代码定义了一个名为SizeCounter的结构体，用于计数从Copy().中复制的字节数。

定义了一个名为CopyOption的类型，用于表示复制数据的选项。

定义了一个名为UpdateActivity的CopyOption，用于在每次数据复制操作期间更新活动。

在CloseActivity()函数中，使用CopyOption UpdateActivity()函数来更新活动，通过append来设置活动记录器。因此，每次有新数据被复制时，活动机会被更新，并在活动计时器中通知。


```go
// SizeCounter is for counting bytes copied by Copy().
type SizeCounter struct {
	Size int64
}

// CopyOption is an option for copying data.
type CopyOption func(*copyHandler)

// UpdateActivity is a CopyOption to update activity on each data copy operation.
func UpdateActivity(timer signal.ActivityUpdater) CopyOption {
	return func(handler *copyHandler) {
		handler.onData = append(handler.onData, func(MultiBuffer) {
			timer.Update()
		})
	}
}

```

这段代码定义了一个名为CountSize的CopyOption类型。该CopyOption类型的函数CountSize作用于一个名为SizeCounter的变量，返回一个复制选项函数，该函数可以将SizeCounter中存储的数据复制到给定的handler对象中。

函数的实现部分如下：


func CountSize(sc *SizeCounter) CopyOption {
	return func(handler *copyHandler) {
		handler.onData = append(handler.onData, func(b MultiBuffer) {
			sc.Size += int64(b.Len())
		})
	}
}


该函数接收一个SizeCounter类型的参数sc，并返回一个CopyOption类型的函数。该函数的作用是监听SizeCounter中存储的数据的复制，将每个数据复制到传入的handler对象中。

函数的具体实现包括以下几个步骤：

1. 在函数的参数列表中添加了一个名为handler的参数，该参数是一个指向copyhandler类型的指针。

2. 在函数体中，定义了一个名为handler的onData指针，该指针存储了一个函数，该函数接收一个MultiBuffer类型的参数b。

3. 在函数体中，使用append函数将handler.onData所包含的函数添加到已经定义的onData列表中。

4. 在函数体中，定义了一个名为sc.Size的变量，用于存储SizeCounter中存储的数据的总大小。

5. 在函数体中，使用int64函数将传入的b.Len()的值复制到sc.Size中。

总体而言，该函数的作用是监听SizeCounter中存储的数据的复制，并将每个数据复制到传入的handler对象中。


```go
// CountSize is a CopyOption that sums the total size of data copied into the given SizeCounter.
func CountSize(sc *SizeCounter) CopyOption {
	return func(handler *copyHandler) {
		handler.onData = append(handler.onData, func(b MultiBuffer) {
			sc.Size += int64(b.Len())
		})
	}
}

type readError struct {
	error
}

func (e readError) Error() string {
	return e.error.Error()
}

```

这段代码定义了两个函数：`func (e readError) Inner() error` 和 `func IsReadError(err error) bool`。

第一个函数 `(e readError) Inner() error`接收一个 `readError` 类型的参数 `e`，并返回 `e` 造成的错误。

第二个函数 `IsReadError(err error) bool` 接收一个 `err` 类型的参数，并判断它是否为 `readError` 类型。如果是，函数返回 `true`，否则返回 `false`。

另外，还定义了一个 `writeError` 类型，该类型有一个 `error` 字段和一个 `innerError` 字段，其中 `innerError` 是 `writeError` 类型的实例。


```go
func (e readError) Inner() error {
	return e.error
}

// IsReadError returns true if the error in Copy() comes from reading.
func IsReadError(err error) bool {
	_, ok := err.(readError)
	return ok
}

type writeError struct {
	error
}

func (e writeError) Error() string {
	return e.error.Error()
}

```

这段代码定义了一个名为`copyInternal`的函数，它接收三个参数：

1. 一个`Reader`类型的读者，用于从输入中读取数据。
2. 一个`Writer`类型的写作者，用于向输出中写入数据。
3. 一个指向`copyHandler`类型的引用，用于传递数据处理函数。

函数实现中，首先判断给定的`err`是否为`WriteError`类型。如果是，那么函数返回`true`。否则，继续执行函数内部的数据复制过程。

具体来说，实现中分为以下几个步骤：

1. 从`Reader`中读取数据，并遍历所有的数据处理函数，如果某个函数处理失败，返回一个`writeError`类型的错误。
2. 从`Writer`中写入数据，如果写入过程中出现错误，返回一个`writeError`类型的错误。
3. 如果以上两个步骤都没有发生错误，那么继续执行数据复制过程。

函数的实现非常简单，但考虑到数据读写和错误处理的需要，函数的功能和复杂度都比较高。


```go
func (e writeError) Inner() error {
	return e.error
}

// IsWriteError returns true if the error in Copy() comes from writing.
func IsWriteError(err error) bool {
	_, ok := err.(writeError)
	return ok
}

func copyInternal(reader Reader, writer Writer, handler *copyHandler) error {
	for {
		buffer, err := reader.ReadMultiBuffer()
		if !buffer.IsEmpty() {
			for _, handler := range handler.onData {
				handler(buffer)
			}

			if werr := writer.WriteMultiBuffer(buffer); werr != nil {
				return writeError{werr}
			}
		}

		if err != nil {
			return readError{err}
		}
	}
}

```

这段代码定义了一个名为Copy的函数，它作用于读取器(reader)和写入器(writer)之间的数据传输。

该函数的行为是在读取期间将所有数据传输到写入器，并在发生错误时停止传输或返回错误。当遇到闭字符串（EOF）时，函数将返回 nil。在选项中，函数接受一个或多个复制选项(CopyOption)。

具体来说，该函数首先定义了一个名为handler的类型，它是一个将数据从读取器传输到写入器的处理程序。接着，该函数遍历传递给它的选项，并将每个选项与其关联的handler传递给第一个参数。

然后，该函数调用了一个名为copyInternal的内部函数，该函数将读取器和写入器之间的数据复制到handler。如果从读取器到写入器的传输过程中出现错误，函数将在不尝试重新传输数据的情况下返回错误，并检查错误是否是由未及时关闭输入读取器（如，由于 timeout）导致的。

最后，该函数返回 nil，如果从读取器到写入器传输过程中没有发生错误，或者在选项中没有提供任何具体的处理程序。


```go
// Copy dumps all payload from reader to writer or stops when an error occurs. It returns nil when EOF.
func Copy(reader Reader, writer Writer, options ...CopyOption) error {
	var handler copyHandler
	for _, option := range options {
		option(&handler)
	}
	err := copyInternal(reader, writer, &handler)
	if err != nil && errors.Cause(err) != io.EOF {
		return err
	}
	return nil
}

var ErrNotTimeoutReader = newError("not a TimeoutReader")

```

这是一段 Go 语言中的函数，名为 `CopyOnceTimeout`。它接收三个参数：

1. `reader`：一个 `Reader` 类型的变量，用于读取数据。
2. `writer`：一个 `Writer` 类型的变量，用于写入数据。
3. `timeout`：一个 `time.Duration` 类型的参数，表示超时时间。

函数的作用是复制数据。具体来说，它尝试等待 `timeout` 毫秒，如果读者（Reader）在超时时间内没有返回数据，它将返回一个错误。否则，它将从读者中读取数据，并将其写入到 `writer` 中。

从函数的实现来看，它主要使用了两个已知的外部库：`timeoutReader` 和 `ReadMultiBufferTimeout`。`timeoutReader` 是一个实现了 `timeoutReader` 接口的读者，它尝试在超时时间内从指定位置读取数据。`ReadMultiBufferTimeout` 是另一个已知的外部库，它提供了在 `timeoutReader` 中读取多缓冲区数据的功能。




```go
func CopyOnceTimeout(reader Reader, writer Writer, timeout time.Duration) error {
	timeoutReader, ok := reader.(TimeoutReader)
	if !ok {
		return ErrNotTimeoutReader
	}
	mb, err := timeoutReader.ReadMultiBufferTimeout(timeout)
	if err != nil {
		return err
	}
	return writer.WriteMultiBuffer(mb)
}

```

# `common/buf/copy_test.go`

这段代码是一个 Go 语言编写的测试框架，用于测试一个名为 "buf_test" 的包。通过使用 crypto/rand 和 io 包，该框架模拟了一个 V2Ray 服务器的核心功能，即从客户端接收数据并将其复制到缓冲区。

具体来说，该测试框架包含一个名为 "TestReadError" 的测试函数。函数内部使用 mockCtl（github.com/golang/mock/gomock）包创建了一个客户端模拟器，并使用该模拟器对一个名为 "mockReader" 的虚拟读者进行读取数据。

如果模拟读取数据时出现错误，函数会返回一个带着错误消息的错误。如果期望错误，但实际没有错误，函数会返回一个非错误的消息。如果期望错误，但实际上出现了错误，函数会打印错误消息并崩溃。

该测试函数的作用是验证 "buf_test" 包在处理从客户端读取数据时是否正确地处理了错误情况。


```go
package buf_test

import (
	"crypto/rand"
	"io"
	"testing"

	"github.com/golang/mock/gomock"

	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/errors"
	"v2ray.com/core/testing/mocks"
)

func TestReadError(t *testing.T) {
	mockCtl := gomock.NewController(t)
	defer mockCtl.Finish()

	mockReader := mocks.NewReader(mockCtl)
	mockReader.EXPECT().Read(gomock.Any()).Return(0, errors.New("error"))

	err := buf.Copy(buf.NewReader(mockReader), buf.Discard)
	if err == nil {
		t.Fatal("expected error, but nil")
	}

	if !buf.IsReadError(err) {
		t.Error("expected to be ReadError, but not")
	}

	if err.Error() != "error" {
		t.Fatal("unexpected error message: ", err.Error())
	}
}

```

这段代码是一个名为 `TestWriteError` 的测试函数，它使用 Go mocking 技术来模拟 `write` 函数的行为。

具体来说，这段代码创建了一个名为 `mockCtl` 的控制器，该控制器使用 `mocks.NewController` 创建了一个模拟的 `write` 函数，该函数接收一个 `mockWriter` 作为参数，然后在模拟的 `write` 函数中执行 `EXPECT().Write(any)` 代码，该代码返回一个非空 `int` 和一个错误对象，模拟的 `write` 函数将返回一个错误并输出该错误。

接下来，这段代码创建了一个名为 `buf` 的缓冲区，然后使用 `buf.Copy` 将缓冲区的内容复制到一个名为 `buf.Reader` 的 `Reader` 上，然后使用另一个名为 `mockWriter` 的 `Writer` 将缓冲区的内容写入模拟的 `write` 函数中，将内容复制到期望的输出缓冲区 `buf.Writer` 中。

最后，这段代码使用 `buf.IsWriteError` 检验缓冲区的内容是否为错误，如果是错误，则输出错误并退出测试，否则不输出任何内容并继续测试。如果模拟的 `write` 函数返回非空 `int`，则说明缓冲区的内容为错误，否则继续测试。


```go
func TestWriteError(t *testing.T) {
	mockCtl := gomock.NewController(t)
	defer mockCtl.Finish()

	mockWriter := mocks.NewWriter(mockCtl)
	mockWriter.EXPECT().Write(gomock.Any()).Return(0, errors.New("error"))

	err := buf.Copy(buf.NewReader(rand.Reader), buf.NewWriter(mockWriter))
	if err == nil {
		t.Fatal("expected error, but nil")
	}

	if !buf.IsWriteError(err) {
		t.Error("expected to be WriteError, but not")
	}

	if err.Error() != "error" {
		t.Fatal("unexpected error message: ", err.Error())
	}
}

```

这两行代码定义了一个名为 `TestReader` 的结构体，该结构体有一个名为 `Read` 的方法，接受一个字节切片 `b` 作为参数，并返回两个参数：一个表示已读取的字节数，一个表示错误。

第二行代码定义了一个名为 `BenchmarkCopy` 的函数，该函数接收一个字节切片 `b` 作为参数，并使用一个可读写缓冲区的 `Reader` 对象和一个简单的 `Discard` 函数作为 `writer`。该函数使用一个循环来读取 `b` 切片中的所有字节，并将其复制到可读写缓冲区的 `Writer` 对象中。

该代码的目的是创建一个 `TestReader` 实例，该实例可以被用作测试中读取字节切片。在 `BenchmarkCopy` 函数中，我们创建了一个可读写缓冲区的 `Reader` 对象和一个简单的 `Discard` 函数作为 `writer`。我们使用一个循环来读取一个名为 `b` 的字节切片，并将其复制到可读写缓冲区的 `Writer` 对象中。由于 `Reader` 对象具有 `io.LimitReader` 接口，因此我们可以使用它来读取不超过指定限制的输入数据。


```go
type TestReader struct{}

func (TestReader) Read(b []byte) (int, error) {
	return len(b), nil
}

func BenchmarkCopy(b *testing.B) {
	reader := buf.NewReader(io.LimitReader(TestReader{}, 10240))
	writer := buf.Discard

	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		_ = buf.Copy(reader, writer)
	}
}

```

# `common/buf/errors.generated.go`

这段代码定义了一个名为"buf"的包，其中包含以下功能：

1. 导入了"v2ray.com/core/common/errors"库，以便使用其中的错误对象。

2. 定义了一个名为"errPathObjHolder"的结构体，该结构体只有一个成员变量，类型为"er"。

3. 定义了一个名为"newError"的函数，该函数接收多个参数，可以是数字"..."，也可以是任意数量的用户自定义类型。函数返回一个错误对象，该对象包含与传递给函数的最后一个参数相当多的错误对象，同时还包含一个指向错误对象路径对象的引用。

4. 通过调用包的默认导入设置，从"v2ray.com/core/common/errors"库中导入"errors.Error"类型，以便使用其中的错误对象。


```go
package buf

import "v2ray.com/core/common/errors"

type errPathObjHolder struct{}

func newError(values ...interface{}) *errors.Error {
	return errors.New(values...).WithPathObj(errPathObjHolder{})
}

```

# `common/buf/io.go`

这段代码定义了一个名为"buf"的包，其中包含了一些用于处理网络数据的函数和类型定义。

具体来说，它实现了Reader接口，用于从网络中读取数据。Reader接口定义了一个名为ReadMultiBuffer的类型，它是一个io.Reader接口的扩展，允许读者使用MultiBuffer对读取的数据进行缓冲。

除了Reader接口外，代码还实现了一些与网络数据传输相关的函数，例如向目标IP地址发送数据并设置超时时间和允许连接忘记关闭套接字。通过调用这些函数，可以实现与远程系统的数据传输和通信。


```go
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

```

这段代码定义了一个名为 "ErrReadTimeout" 的错误类型，该错误类型表示在 I/O 操作超时的情况下发生。

然后，定义了一个名为 "ErrReadTimeout" 的变量，它从 "IO timeout" 错误消息中获取错误类型。

接着，定义了一个名为 "TimeoutReader" 的接口，该接口表示一个可以返回错误信息并提供 Read() 操作超时时间的读者。

然后，定义了一个名为 "Writer" 的接口，该接口实现了 "io.Writer" 类型，具有 "WriteMultiBuffer" 和 "WriteAllBytes" 方法，可以在底层写入缓冲区和所有字节并返回错误。

最后，给 "TimeoutReader" 接口实现了 "Writer" 接口的变量 "WriterTimeout" 赋值一个新错误类型 "ErrReadTimeout"，以便在 "TimeoutReader" 接口的 "Read()" 操作中出现超时时，可以返回相应的错误信息。


```go
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
```

这两段代码一起实现了一个网络数据包的接收和发送功能。具体来说，这两段代码定义了两个函数：`WriteAllBytes` 和 `isPacketReader`。

1. `WriteAllBytes` 函数接收一个 `io.Writer` 类型的写入器和一个字节数组 `payload`。该函数将从写入器中连续写入 `payload` 字节，并循环多次。如果写入过程中出现错误，函数将返回一个错误。

2. `isPacketReader` 函数接收一个 `io.Reader` 类型的读入器。函数判断 `reader` 是否为网络数据包连接 `net.PacketConn` 类型的实例。如果是，函数返回一个真，否则返回一个假。

这两段代码的主要目的是实现网络数据包的接收和发送。`WriteAllBytes` 函数用于将接收到的数据包连续发送，而 `isPacketReader` 函数用于判断接收到的数据包是否来自网络数据包连接。


```go
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
	_, ok := reader.(net.PacketConn)
	return ok
}

```

这段代码定义了一个名为NewReader的函数，它接受一个io.Reader类型的参数。这个函数的作用是创建一个新的Reader实例，并将它返回。

函数内部首先检查mr是否是Reader类型，如果是，就直接返回它。如果不是，则检查isPacketReader()函数，如果是，就创建一个PacketReader实例并将Reader设置为它。如果不是文件，且useReadv设置为true，那么会尝试使用操作系统调用中的`useReadv`函数来尝试读取文件内容。如果sc是*os.File类型，就可以通过调用SingleReader函数来创建一个新的SingleReader实例，并将Reader设置为sc。

函数最终返回一个Reader实例，方便后续操作使用。


```go
// NewReader creates a new Reader.
// The Reader instance doesn't take the ownership of reader.
func NewReader(reader io.Reader) Reader {
	if mr, ok := reader.(Reader); ok {
		return mr
	}

	if isPacketReader(reader) {
		return &PacketReader{
			Reader: reader,
		}
	}

	_, isFile := reader.(*os.File)
	if !isFile && useReadv {
		if sc, ok := reader.(syscall.Conn); ok {
			rawConn, err := sc.SyscallConn()
			if err != nil {
				newError("failed to get sysconn").Base(err).WriteToLog()
			} else {
				return NewReadVReader(reader, rawConn)
			}
		}
	}

	return &SingleReader{
		Reader: reader,
	}
}

```

这两段代码一起实现了两个函数：`NewPacketReader` 和 `isPacketWriter`。

1. `NewPacketReader` 函数接收一个 `io.Reader` 类型的参数，然后检查该 `Reader` 是否已经存在。如果已经存在，则直接返回。否则，创建一个新的 `PacketReader` 实例，并将其作为返回值。

2. `isPacketWriter` 函数接收一个 `io.Writer` 类型的参数，然后检查该 `Writer` 是否支持 `net.PacketConn`。如果支持，则返回 `true`。否则，如果该 `Writer` 不属于 `net.PacketConn` 类型，则返回 `true`。如果既不支持也不属于 `net.PacketConn` 类型，则返回 `false`。

这两段代码的主要目的是创建一个 `PacketReader` 实例，用于读取二进制数据包。`NewPacketReader` 函数用于创建一个新的 `PacketReader` 实例，而 `isPacketWriter` 函数用于检查 `Writer` 是否支持 `net.PacketConn`。


```go
// NewPacketReader creates a new PacketReader based on the given reader.
func NewPacketReader(reader io.Reader) Reader {
	if mr, ok := reader.(Reader); ok {
		return mr
	}

	return &PacketReader{
		Reader: reader,
	}
}

func isPacketWriter(writer io.Writer) bool {
	if _, ok := writer.(net.PacketConn); ok {
		return true
	}

	// If the writer doesn't implement syscall.Conn, it is probably not a TCP connection.
	if _, ok := writer.(syscall.Conn); !ok {
		return true
	}
	return false
}

```

这段代码定义了一个名为 `NewWriter` 的函数，它接受一个名为 `writer` 的 `io.Writer` 类型的参数。它的作用是创建一个新的 `Writer`。

函数首先检查 `writer` 是否为已定义的 `Writer` 类型，如果是，则直接返回该 `Writer`。然后，它检查 `writer` 是否为 `io.PacketWriter` 类型，如果是，则返回一个名为 `&SequentialWriter` 的 `Writer` 类型，其中 `Writer` 字段表示 `writer` 是一个 `PacketWriter`。如果不是 `PacketWriter`，则返回一个名为 `&BufferToBytesWriter` 的 `Writer` 类型，其中 `Writer` 字段表示 `writer` 是一个 `BufferToBytesWriter`。

最后，函数返回一个新的 `Writer` 类型的引用，该引用指向一个新的 `Writer` 对象，该对象实现了 `io.Writer` 接口，并且 `writer` 参数被传递给它。


```go
// NewWriter creates a new Writer.
func NewWriter(writer io.Writer) Writer {
	if mw, ok := writer.(Writer); ok {
		return mw
	}

	if isPacketWriter(writer) {
		return &SequentialWriter{
			Writer: writer,
		}
	}

	return &BufferToBytesWriter{
		Writer: writer,
	}
}

```

# `common/buf/io_test.go`

这段代码是一个关于 "buf.test" 包的测试，通过创建不同的 TCP 或 TLS 连接，对 "buf" 包中的 "BufferToBytesWriter" 和 "SequentialWriter" 类型的函数进行测试。

具体来说，这段代码主要实现了以下功能：

1. 创建一个 TCP 服务器，用于测试 "buf.test.Writer" 函数的使用。
2. 创建一个 TLS 服务器，也用于测试 "buf.test.Writer" 函数的使用，但在这个场景中，使用了 "InsecureSkipVerify" 选项，表示不验证 SSL/TLS 证书的有效性。
3. 创建一个 "BufferToBytesWriter" 类型的函数，用于向 "buf" 包中的 "BufferToBytesWriter" 函数进行测试。
4. 创建一个 "SequentialWriter" 类型的函数，用于向 "buf" 包中的 "SequentialWriter" 函数进行测试。
5. 通过 "TcpServer" 和 "tcp.Server" 类型的函数，创建一个 TCP 服务器，用于与远程服务器建立连接。
6. 通过 "net.Dial" 函数，建立一个 TCP 连接，与远程服务器建立通信。
7. 通过 "NewWriter" 函数，创建一个 "BufferToBytesWriter" 和一个 "SequentialWriter" 类型的函数，分别用于 "buf.test.Writer" 函数的实现。
8. 在 "TestWriterCreation" 函数中，分别通过创建 TCP 和 TLS 连接，向 "buf.test.Writer" 函数传递不同的输入参数，如不同的缓冲区或不同的连接方式，以测试 "buf.test.Writer" 函数的实现。


```go
package buf_test

import (
	"crypto/tls"
	"io"
	"testing"

	. "v2ray.com/core/common/buf"
	"v2ray.com/core/common/net"
	"v2ray.com/core/testing/servers/tcp"
)

func TestWriterCreation(t *testing.T) {
	tcpServer := tcp.Server{}
	dest, err := tcpServer.Start()
	if err != nil {
		t.Fatal("failed to start tcp server: ", err)
	}
	defer tcpServer.Close()

	conn, err := net.Dial("tcp", dest.NetAddr())
	if err != nil {
		t.Fatal("failed to dial a TCP connection: ", err)
	}
	defer conn.Close()

	{
		writer := NewWriter(conn)
		if _, ok := writer.(*BufferToBytesWriter); !ok {
			t.Fatal("writer is not a BufferToBytesWriter")
		}

		writer2 := NewWriter(writer.(io.Writer))
		if writer2 != writer {
			t.Fatal("writer is not reused")
		}
	}

	tlsConn := tls.Client(conn, &tls.Config{
		InsecureSkipVerify: true,
	})
	defer tlsConn.Close()

	{
		writer := NewWriter(tlsConn)
		if _, ok := writer.(*SequentialWriter); !ok {
			t.Fatal("writer is not a SequentialWriter")
		}
	}
}

```

# `common/buf/multi_buffer.go`

这段代码定义了一个名为"buf"的包，其中包含了一些与缓冲区操作相关的函数。

1. `import`语句导入了一些外部库，包括"io"、"v2ray.com/core/common"和"v2ray.com/core/common/errors"。

2. `package`语句定义了该包的名称。

3. `import`语句导入了一个名为"ReadAllToBytes"的函数，该函数使用了`io.Reader`类型的读者，从读取器中读取所有内容并存储到字节数组中，直到遇到字符 EndOfFile 为止。

4. `func`语句定义了一个名为"ReadAllToBytes"的函数，其参数为"io.Reader"类型的读者和字节数组。

5. `if`语句检查读取器是否出错，如果出错，则返回 nil 和错误。

6. `return`语句返回字节数组和 nil，如果函数正常运行，则返回字节数组。

7. `func SplitBytes(mb []byte, b []byte) (int, []byte)`函数将一个字节数组mb分割成两个字节数组，并返回它们的下标。如果分割操作出现错误，函数将返回-1和空字节数组。

8. `func ReleaseMulti(mb []byte)`函数释放一个多字节字符串mb，它可以将多字节字符串中的所有字节释放回内存。

9. `func ReadFrom(reader io.Reader) (int, []byte)`函数使用io.Reader读取一个io.Reader类型的读者，并返回读取的字节数和对应的字节数组。如果读取操作出现错误，函数将返回-1和空字节数组。

10. `func WriteTo(writer io.Writer, b []byte)`函数使用io.Writer写入一个io.Writer类型的写入者，并写入字节数组b。如果写入操作出现错误，函数将返回-1和空字节数组。


```go
package buf

import (
	"io"

	"v2ray.com/core/common"
	"v2ray.com/core/common/errors"
	"v2ray.com/core/common/serial"
)

// ReadAllToBytes reads all content from the reader into a byte array, until EOF.
func ReadAllToBytes(reader io.Reader) ([]byte, error) {
	mb, err := ReadFrom(reader)
	if err != nil {
		return nil, err
	}
	if mb.Len() == 0 {
		return nil, nil
	}
	b := make([]byte, mb.Len())
	mb, _ = SplitBytes(mb, b)
	ReleaseMulti(mb)
	return b, nil
}

```

这段代码定义了一个名为MultiBuffer的多缓冲区列表类型，其中每个缓冲区的顺序非常重要。

接着，定义了两个名为MergeMulti和MergeBytes的函数，用于对多个缓冲区进行合并操作。

MergeMulti函数接收两个MultiBuffer类型的参数dest和src，返回一个新的MultiBuffer类型dest和一个MultiBuffer类型src的剩余部分。函数通过将src缓冲区的内容追加到dest缓冲区中，并且将src缓冲区中所有不完整的字节设置为nil来完成合并操作。

MergeBytes函数接收一个MultiBuffer类型类型的参数dest和一个字节切片src，返回一个新的MultiBuffer类型。函数将检查dest缓冲区是否已经满了，如果是，就从dest缓冲区中读取足够的字节将src缓冲区中的内容写入dest缓冲区中。然后，从src缓冲区中读取比dest缓冲区中读取的字节数更大的字节，创建一个新的MultiBuffer类型，并将它加入到dest缓冲区中。这样，所有的src缓冲区内容都会被正确地合并到dest缓冲区中，而不会丢失任何字节。


```go
// MultiBuffer is a list of Buffers. The order of Buffer matters.
type MultiBuffer []*Buffer

// MergeMulti merges content from src to dest, and returns the new address of dest and src
func MergeMulti(dest MultiBuffer, src MultiBuffer) (MultiBuffer, MultiBuffer) {
	dest = append(dest, src...)
	for idx := range src {
		src[idx] = nil
	}
	return dest, src[:0]
}

// MergeBytes merges the given bytes into MultiBuffer and return the new address of the merged MultiBuffer.
func MergeBytes(dest MultiBuffer, src []byte) MultiBuffer {
	n := len(dest)
	if n > 0 && !(dest)[n-1].IsFull() {
		nBytes, _ := (dest)[n-1].Write(src)
		src = src[nBytes:]
	}

	for len(src) > 0 {
		b := New()
		nBytes, _ := b.Write(src)
		src = src[nBytes:]
		dest = append(dest, b)
	}

	return dest
}

```

这两段代码定义了两个名为 `ReleaseMulti` 和 `Copy` 的函数。它们都是 `MultiBuffer` 类的实例方法。

1. `ReleaseMulti` 函数的目的是释放 `MultiBuffer` 对象中的所有内容，并返回一个空的 `MultiBuffer`。它通过一个循环来遍历 `MultiBuffer` 对象的所有元素，并释放每个元素的副本。最后，它返回的是 `MultiBuffer` 对象本身，但要去除所有元素后再返回，所以最终返回的是一个空的 `MultiBuffer`。

2. `Copy` 函数的目的是将 `MultiBuffer` 对象中的一部分复制到给定的字节数组中。它通过一个循环来遍历 `MultiBuffer` 对象的所有元素，并计算出每个元素的实际字节数。然后，它会将这个字节数赋给一个变量 `total`，并从元素中复制这些字节数到给定的字节数组中。最后，它返回的是总共复制的字节数。

这两个函数都在同一个名为 `MultiBuffer` 的类中定义，所以我们可以推断出 `MultiBuffer` 类支持这两种类型的复制操作。


```go
// ReleaseMulti release all content of the MultiBuffer, and returns an empty MultiBuffer.
func ReleaseMulti(mb MultiBuffer) MultiBuffer {
	for i := range mb {
		mb[i].Release()
		mb[i] = nil
	}
	return mb[:0]
}

// Copy copied the beginning part of the MultiBuffer into the given byte array.
func (mb MultiBuffer) Copy(b []byte) int {
	total := 0
	for _, bb := range mb {
		nBytes := copy(b[total:], bb.Bytes())
		total += nBytes
		if int32(nBytes) < bb.Len() {
			break
		}
	}
	return total
}

```

这段代码定义了一个名为 `ReadFrom` 的函数，它接受一个名为 `reader` 的 `io.Reader` 参数，并返回一个名为 `MultiBuffer` 的多个缓冲区以及一个可能的错误。

该函数的主要目的是从 `reader` 中读取所有内容，直到遇到 EOF( end-of-file 断言)。在函数内部，最初创建了一个名为 `mb` 的 `MultiBuffer`，这个 `MultiBuffer` 最初设置为 16 个缓冲区，然后循环从 `reader` 中读取内容，并将其存储在 `b` 变量中。如果 `b` 对象是空的，那么释放它，否则将 `b` 添加到 `mb` 中的缓冲区中。

函数还检查从 `reader` 中读取内容时是否遇到 EOF 或错误。如果是 EOF，函数将返回一个空 `MultiBuffer`，并且不会返回任何错误。如果是错误，函数将返回一个包含错误信息的 `MultiBuffer`，并且不会返回任何内容。


```go
// ReadFrom reads all content from reader until EOF.
func ReadFrom(reader io.Reader) (MultiBuffer, error) {
	mb := make(MultiBuffer, 0, 16)
	for {
		b := New()
		_, err := b.ReadFullFrom(reader, Size)
		if b.IsEmpty() {
			b.Release()
		} else {
			mb = append(mb, b)
		}
		if err != nil {
			if errors.Cause(err) == io.EOF || errors.Cause(err) == io.ErrUnexpectedEOF {
				return mb, nil
			}
			return mb, err
		}
	}
}

```

这段代码的作用是实现了一个名为 `SplitBytes` 的函数，它用于将传入的多缓冲区（MultiBuffer）中的字节按照指定的字节数进行分割，并返回分割后剩余的多缓冲区和写入到输入字节切片（byte slice）中的字节数。

函数接收一个名为 `mb` 的 MultiBuffer 和一个名为 `b` 的字节切片参数。函数首先定义了一个 `totalBytes` 变量和一个 `endIndex` 变量，初始值分别为 0 和 -1。

函数中的循环从 `mb` 的第一个元素开始，逐个读取并累加到 `nBytes`，然后将 `nBytes` 字段的值赋给 `b` 切片，将 `pBuffer` 中的内容从 `mb` 切片中移除，并将 `pBuffer` 设置为 `nil`。这样，当 `pBuffer` 不再拥有任何数据时，函数结束循环并检查 `endIndex` 变量是否为 `-1`。如果是，则表示所有可用的数据都已经处理完毕，此时将 `mb` 的第一个元素赋值给 `mb`，并将 `endIndex` 变量设置为 `-1`。否则，将 `mb` 中 `endIndex` 之后的元素设置为 `nil`，并将 `nBytes` 累加到 `totalBytes` 变量中。

最后，函数返回分割后的 `mb` 和 `totalBytes` 值。


```go
// SplitBytes splits the given amount of bytes from the beginning of the MultiBuffer.
// It returns the new address of MultiBuffer leftover, and number of bytes written into the input byte slice.
func SplitBytes(mb MultiBuffer, b []byte) (MultiBuffer, int) {
	totalBytes := 0
	endIndex := -1
	for i := range mb {
		pBuffer := mb[i]
		nBytes, _ := pBuffer.Read(b)
		totalBytes += nBytes
		b = b[nBytes:]
		if !pBuffer.IsEmpty() {
			endIndex = i
			break
		}
		pBuffer.Release()
		mb[i] = nil
	}

	if endIndex == -1 {
		mb = mb[:0]
	} else {
		mb = mb[endIndex:]
	}

	return mb, totalBytes
}

```

这两段代码定义了两个名为 SplitFirstBytes 和 Compact 的函数。它们的主要目的是在 MultiBuffer 和给定的一维字节切片之间进行数据划分和合并。

首先，我们需要了解 MultiBuffer 和字节切片的一些基本知识。MultiBuffer 是 reflect 包中定义的接口，它代表了一组连续的内存区域。通过 SplitFirstBytes 函数，我们可以将一个 MultiBuffer 对象 SplitFirst 以后得到两个部分：一个 MultiBuffer，其包含前半部分，另一个整数类型的切片。

接下来，我们来看一下 SplitFirstBytes 函数。它接收一个 MultiBuffer 和一个字节切片（即一个整数类型的切片），然后将 MultiBuffer 的第一个缓冲区从 MultiBuffer 中 SplitFirst，并将分出的第一个缓冲区的内容复制到给定的字节切片上。如果分出的第一个缓冲区为空，函数返回 MultiBuffer 和 0。

最后，我们来看一下 Compact 函数。它接收一个 MultiBuffer，然后将其所有内容合并成一个新 MultiBuffer。这个新 MultiBuffer 包含所有输入 MultiBuffer 的内容，没有额外的数据区域。

通过这两个函数，我们可以将一个 MultiBuffer 对象 SplitFirst 以后得到两个部分：一个 MultiBuffer 和一个字节切片。接着，通过对 MultiBuffer 和字节切片的操作，我们可以将数据从一个 MultiBuffer 对象分割成两个部分，并将其第一个缓冲区的内容复制到另一个 MultiBuffer 或将 MultiBuffer 和字节切片的内容合并。


```go
// SplitFirstBytes splits the first buffer from MultiBuffer, and then copy its content into the given slice.
func SplitFirstBytes(mb MultiBuffer, p []byte) (MultiBuffer, int) {
	mb, b := SplitFirst(mb)
	if b == nil {
		return mb, 0
	}
	n := copy(p, b.Bytes())
	b.Release()
	return mb, n
}

// Compact returns another MultiBuffer by merging all content of the given one together.
func Compact(mb MultiBuffer) MultiBuffer {
	if len(mb) == 0 {
		return mb
	}

	mb2 := make(MultiBuffer, 0, len(mb))
	last := mb[0]

	for i := 1; i < len(mb); i++ {
		curr := mb[i]
		if last.Len()+curr.Len() > Size {
			mb2 = append(mb2, last)
			last = curr
		} else {
			common.Must2(last.ReadFrom(curr))
			curr.Release()
		}
	}

	mb2 = append(mb2, last)
	return mb2
}

```

这两段代码定义了两个名为 SplitFirst 和 SplitSize 的函数，它们用于对一个名为 MultiBuffer 的数据结构进行分割。

SplitFirst 函数的输入参数是一个名为 MultiBuffer 的数据结构和一个名为 Buffer 的指针。该函数首先检查输入参数的长度是否为 0，如果是，则返回 MultiBuffer 和 nil。否则，该函数提取 MultiBuffer 的第一个字符串，将其设置为 nil，并将 MultiBuffer 的索引设为 1。这样，函数的返回值是 MultiBuffer 和 nil。

SplitSize 函数的输入参数是一个名为 MultiBuffer 的数据结构和一个名为 int32 的整数。函数首先检查输入参数的长度是否为 0，如果是，则返回 MultiBuffer 和 nil。否则，函数将 MultiBuffer 的第一个字符串的长度与传入的尺寸进行比较。如果是，则创建一个新的字符串并将其复制到 MultiBuffer 的第一个字符串的足够大尺寸内。然后，将 MultiBuffer 的第一个字符串的后一个字符添加到新创建的字符串中。这样，函数的返回值是 MultiBuffer 和新创建的字符串。


```go
// SplitFirst splits the first Buffer from the beginning of the MultiBuffer.
func SplitFirst(mb MultiBuffer) (MultiBuffer, *Buffer) {
	if len(mb) == 0 {
		return mb, nil
	}

	b := mb[0]
	mb[0] = nil
	mb = mb[1:]
	return mb, b
}

// SplitSize splits the beginning of the MultiBuffer into another one, for at most size bytes.
func SplitSize(mb MultiBuffer, size int32) (MultiBuffer, MultiBuffer) {
	if len(mb) == 0 {
		return mb, nil
	}

	if mb[0].Len() > size {
		b := New()
		copy(b.Extend(size), mb[0].BytesTo(size))
		mb[0].Advance(size)
		return mb, MultiBuffer{b}
	}

	totalBytes := int32(0)
	var r MultiBuffer
	endIndex := -1
	for i := range mb {
		if totalBytes+mb[i].Len() > size {
			endIndex = i
			break
		}
		totalBytes += mb[i].Len()
		r = append(r, mb[i])
		mb[i] = nil
	}
	if endIndex == -1 {
		// To reuse mb array
		mb = mb[:0]
	} else {
		mb = mb[endIndex:]
	}
	return mb, r
}

```

这段代码定义了一个名为 `WriteMultiBuffer` 的函数，它接受一个名为 `writer` 的输入参数和一个名为 `mb` 的输入参数，用于在多个缓冲区中连续写入数据。

函数首先通过 `SplitFirst` 函数将传入的多缓冲区 `mb` 分离成两个部分，然后遍历第二个部分（即 `b` 部分）。在遍历过程中，如果发现 `b` 为 `nil`，则表示写入数据结束，可以退出循环。

接着，函数尝试从 `writer` 输入通道中连续写入 `mb2` 中的数据。如果写入成功，则 `mb2` 中的数据将被释放，并返回剩余的多缓冲区 `mb`。如果写入失败，则返回包含错误信息的多缓冲区 `mb`。

函数返回的是一个名为 `MultiBuffer` 的类型，它包含了所有的缓冲区，以及一个错误信息。如果调用 `WriteMultiBuffer` 时出现错误，函数将返回一个非空错误。


```go
// WriteMultiBuffer writes all buffers from the MultiBuffer to the Writer one by one, and return error if any, with leftover MultiBuffer.
func WriteMultiBuffer(writer io.Writer, mb MultiBuffer) (MultiBuffer, error) {
	for {
		mb2, b := SplitFirst(mb)
		mb = mb2
		if b == nil {
			break
		}

		_, err := writer.Write(b.Bytes())
		b.Release()
		if err != nil {
			return mb, err
		}
	}

	return nil, nil
}

```

该代码定义了两个名为mb的多缓冲区MultiBuffer类型的函数。

函数Len()返回MultiBuffer中的总字节数，并检查MultiBuffer是否为空。具体来说，函数处理mb中的所有元素，并计算它们的总字节数。最后，函数返回该总字节数。

函数IsEmpty()返回MultiBuffer是否为空。具体来说，函数遍历mb中的所有元素，并检查它们是否为空。如果是任何一个元素为空，函数返回true，否则返回false。

从函数的名称和参数可以推断出，这些函数都与MultiBuffer有关。MultiBuffer是一个用于同时发送多个数据缓冲区的Java类。因此，这些函数可能是在定义MultiBuffer的操作和检查方法。


```go
// Len returns the total number of bytes in the MultiBuffer.
func (mb MultiBuffer) Len() int32 {
	if mb == nil {
		return 0
	}

	size := int32(0)
	for _, b := range mb {
		size += b.Len()
	}
	return size
}

// IsEmpty return true if the MultiBuffer has no content.
func (mb MultiBuffer) IsEmpty() bool {
	for _, b := range mb {
		if !b.IsEmpty() {
			return false
		}
	}
	return true
}

```

这段代码定义了一个名为MultiBufferContainer的结构体，它包含一个MultiBuffer。MultiBuffer是一个序列化的包，它可以容纳多个类型的数据。

函数MultiBufferContainer的String()函数返回MultiBuffer容器中的所有数据串行组成的字符串。

具体来说，函数首先创建一个大小为len(mb)的数组v，然后循环遍历MultiBufferContainer中的每个元素mb。在循环内部，将当前元素赋值给v数组的对应索引。最后，将v数组中的所有元素通过序列化函数Serial化（Serialization）并返回。

另外，MultiBufferContainer还重写了io.Reader接口，实现了一个io.Reader的功能，可以用来从D例读取数据。


```go
// String returns the content of the MultiBuffer in string.
func (mb MultiBuffer) String() string {
	v := make([]interface{}, len(mb))
	for i, b := range mb {
		v[i] = b
	}
	return serial.Concat(v...)
}

// MultiBufferContainer is a ReadWriteCloser wrapper over MultiBuffer.
type MultiBufferContainer struct {
	MultiBuffer
}

// Read implements io.Reader.
```

这段代码定义了两个函数，一个是`Read`函数，另一个是`ReadMultiBuffer`函数。

`Read`函数接收一个`MultiBufferContainer`类型的参数`c`和一个字节数组`b`，并返回一个整数和一个错误。函数首先检查`MultiBufferContainer`中的缓冲区是否为空，如果是，那么函数返回0并丢弃错误。否则，函数将字节数组`b`拆分为多个`MultiBuffer`类型的子缓冲区，并将它们合并为`MultiBufferContainer`类型的缓冲区`mb`。最后，函数返回`nBytes`和` nil`，其中`nBytes`表示读取的字节数，` nil`表示没有错误。

`ReadMultiBuffer`函数与`Read`函数类似，但它返回一个`MultiBuffer`类型和一个错误。函数创建一个空的`MultiBufferContainer`，创建一个`MultiBuffer`并将它赋值给`MultiBufferContainer`类型的`c`。最后，函数返回`MultiBuffer`类型和` nil`，表示已经完成读取操作。


```go
func (c *MultiBufferContainer) Read(b []byte) (int, error) {
	if c.MultiBuffer.IsEmpty() {
		return 0, io.EOF
	}

	mb, nBytes := SplitBytes(c.MultiBuffer, b)
	c.MultiBuffer = mb
	return nBytes, nil
}

// ReadMultiBuffer implements Reader.
func (c *MultiBufferContainer) ReadMultiBuffer() (MultiBuffer, error) {
	mb := c.MultiBuffer
	c.MultiBuffer = nil
	return mb, nil
}

```

这段代码定义了两个函数，名为`Write`和`WriteMultiBuffer`，它们都接受一个名为`MultiBuffer`的整型参数和一个名为`b`的字节切片。这两个函数实现了`io.Writer`接口，可以用来向多个缓冲区容器中的多个缓冲区写入数据。

具体来说，这两个函数的实现包括以下步骤：

1. 在函数`Write`中，首先将`c.MultiBuffer`和`b`字节切片进行合并，得到一个新的`MultiBuffer`对象。然后，返回`len(b)`作为结果，表示成功写入的字节数，以及一个`nil`错误。

2. 在函数`WriteMultiBuffer`中，首先使用`MergeMulti`函数将`c.MultiBuffer`和`b`字节切片合并，得到一个新的`MultiBuffer`对象。然后，将合并后的`MultiBuffer`对象存储在`c.MultiBuffer`变量中，并返回一个`nil`错误。

3. 在函数`Close`中，首先将`c.MultiBuffer`中的所有数据元素释放，然后关闭桶（close）该桶。


```go
// Write implements io.Writer.
func (c *MultiBufferContainer) Write(b []byte) (int, error) {
	c.MultiBuffer = MergeBytes(c.MultiBuffer, b)
	return len(b), nil
}

// WriteMultiBuffer implement Writer.
func (c *MultiBufferContainer) WriteMultiBuffer(b MultiBuffer) error {
	mb, _ := MergeMulti(c.MultiBuffer, b)
	c.MultiBuffer = mb
	return nil
}

// Close implement io.Closer.
func (c *MultiBufferContainer) Close() error {
	c.MultiBuffer = ReleaseMulti(c.MultiBuffer)
	return nil
}

```

# `common/buf/multi_buffer_test.go`

这段代码是一个 Go 语言编写的测试框架，用于测试buf包中的内容。具体来说，这段代码包括以下几个部分：

1. 导入所需的包：
	* "bytes"
	* "crypto/rand"
	* "io"
	* "testing"
	* "github.com/google/go-cmp/cmp"
	* "io/ioutil"
	* "os"
	* "v2ray.com/core/common"
	* ". "v2ray.com/core/common/buf"
2. 定义一个名为 "test_buf_io_滑动套接字 ..." 的函数。这个函数接收一个 "buf" 类型的参数，然后进行一系列测试：
	* 检查 "buf" 参数是否为空字符串。
	* 通过 "io.ioutil.ReadAll" 函数读取一个空字符串，并打印到 "testing.T" 类型。
	* 通过 "io.ioutil.WriteAll" 函数向一个有缓冲的字符串中写入字符串 "hello"，并打印到 "testing.T" 类型。
	* 通过 "io.ioutil.ReadAll" 函数读取一个有缓冲的字符串中所有的字符，并打印到 "testing.T" 类型。
	* 通过 "io.ioutil.WriteAll" 函数向一个有缓冲的字符串中写入字符串 "world"，并打印到 "testing.T" 类型。
	* 通过 "buf" 包中的 "实现轮询操作" 函数测试是否可以通过 "buf" 包中的 "轮询操作" 函数获取 "buf" 包中的内容。
3. 定义一个名为 "test_buf_滑动套接字 ..." 的函数。这个函数与上面定义的 "test_buf_io_滑动套接字 ..." 函数类似，不同之处在于它使用了 "滑动套接字" 而不是 "轮询操作"。具体来说，这个函数接收一个 "buf" 类型的参数，然后进行一系列测试：
	* 创建一个有缓冲的字符串 "buf_str"。
	* 通过 "len" 函数获取字符串 "buf_str" 中的字符数。
	* 通过 "buf.Pool.Add" 函数向 "buf_str" 中的字符数个连续的缓冲区 "buf" 添加字符。
	* 通过 "buf.Pool.Get" 函数从 "buf_str" 中的字符数个连续的缓冲区 "buf" 中获取字符。
	* 通过 "buf.Pool.Put" 函数将获取的字符串 "buf_str" 中的一个字符 "str" 添加到 "buf" 中的下一个连续缓冲区。
	* 通过 "buf.Pool.Get" 函数从 "buf_str" 中的字符数个连续的缓冲区 "buf" 中获取字符。
	* 通过 "buf.Pool.Put" 函数将获取的字符串 "buf_str" 中的一个字符 "str2" 添加到 "buf" 中的下一个连续缓冲区。
	* 通过 "buf.Pool.Get" 函数从 "buf_str" 中的字符数个连续的缓冲区 "buf" 中获取字符。
	* 通过 "buf.Pool.Put" 函数将获取的字符串 "buf_str" 中的一个字符 "str3" 添加到 "buf" 中的下一个连续缓冲区。
	* 通过 "buf.Pool.Get" 函数从 "buf_str" 中的字符数个连续的缓冲区 "buf" 中获取字符。
	* 通过 "buf.Pool.Put" 函数将获取的字符串 "buf_str" 中的一个字符 "str4" 添加到 "buf" 中的下一个连续缓冲区。
	* 通过 "buf.Pool.Get" 函数从 "buf_str" 中的字符数个连续的缓冲区 "buf" 中获取字符。
	* 通过 "buf.Pool.Put" 函数将获取的字符串 "buf_str" 中的一个字符 "str5" 添加到 "buf" 中的下一个连续缓冲区。
	* 通过 "buf.Pool.Get" 函数从 "buf_str" 中的字符


```go
package buf_test

import (
	"bytes"
	"crypto/rand"
	"io"
	"testing"

	"github.com/google/go-cmp/cmp"
	"io/ioutil"
	"os"

	"v2ray.com/core/common"
	. "v2ray.com/core/common/buf"
)

```

以上代码是为了测试一个名为 "MultiBufferRead" 的函数。该函数接受一个测试用例 "t"，并在测试用例开始时创建了一个名为 "b1" 的缓冲区和一个名为 "mb" 的多缓冲区。

在 "MultiBufferRead" 函数中，首先创建了一个名为 "b1" 的缓冲区，并使用 "WriteString" 函数将字符串 "ab" 写入该缓冲区。然后，创建了一个名为 "b2" 的缓冲区，并使用 "WriteString" 函数将字符串 "cd" 写入该缓冲区。

接下来，使用 "MultiBuffer" 函数创建了一个名为 "mb" 的多缓冲区，将 "b1" 和 "b2" 两个缓冲区链接起来。

接着，使用 "SplitBytes" 函数将多缓冲区 "mb" 分割成一个包含 32 个字节缓冲区的子缓冲区，并将这些子缓冲区复制到 "bs" 数组中。

最后，使用 "cmp" 函数比较 "bs" 数组与一个名为 "abcd" 的字符串之间的差异。如果两个字符串不同，则函数会输出该差异。


```go
func TestMultiBufferRead(t *testing.T) {
	b1 := New()
	common.Must2(b1.WriteString("ab"))

	b2 := New()
	common.Must2(b2.WriteString("cd"))
	mb := MultiBuffer{b1, b2}

	bs := make([]byte, 32)
	_, nBytes := SplitBytes(mb, bs)
	if nBytes != 4 {
		t.Error("expect 4 bytes split, but got ", nBytes)
	}
	if r := cmp.Diff(bs[:nBytes], []byte("abcd")); r != "" {
		t.Error(r)
	}
}

```

这两段代码是对一个名为 "MultiBufferAppend" 和 "MultiBufferSliceBySizeLarge" 的测试函数。它们都使用了 "MultiBuffer" 类型，该类型允许在缓冲区中添加或删除多个数据元素。

第一段代码 "TestMultiBufferAppend" 演示了如何使用 MultiBuffer 类型在缓冲区中添加一个字节数组并返回该缓冲区。它通过使用 "New" 函数创建了一个名为 "mb" 的 MultiBuffer 对象，并使用 "WriteString" 方法将一个字符串 "ab" 写入该对象。然后，它使用 "append" 方法将另一个名为 "b" 的缓冲区对象添加到 MultiBuffer 中。最后，它使用一系列条件判断来检查 MultiBuffer 中的数据元素数量是否符合预期。

第二段代码 "TestMultiBufferSliceBySizeLarge" 则演示了如何使用 MultiBuffer 类型在缓冲区中切片并返回多个数据元素。它首先使用 "make" 函数创建了一个长度为 8 1024 个字节的数据缓冲区 "lb"，并使用 "io.ReadFull" 函数读取该缓冲区并将其存储在 "mb" 变量中。然后，它使用 "MergeBytes" 函数将一个空字符串 "nil" 和缓冲区 "lb" 中的所有数据元素合并成一个 MultiBuffer 对象 "mb2"，并将合并后的数据元素数量存储在 "mb2.Len" 变量中。接下来，它使用 "SplitSize" 函数将 MultiBuffer "mb2" 以 1024 个字节为间隔分割成两个数据元素 "mb" 和 "mb2"。最后，它使用一系列条件判断来检查 MultiBuffer 中的数据元素数量是否符合预期。


```go
func TestMultiBufferAppend(t *testing.T) {
	var mb MultiBuffer
	b := New()
	common.Must2(b.WriteString("ab"))
	mb = append(mb, b)
	if mb.Len() != 2 {
		t.Error("expected length 2, but got ", mb.Len())
	}
}

func TestMultiBufferSliceBySizeLarge(t *testing.T) {
	lb := make([]byte, 8*1024)
	common.Must2(io.ReadFull(rand.Reader, lb))

	mb := MergeBytes(nil, lb)

	mb, mb2 := SplitSize(mb, 1024)
	if mb2.Len() != 1024 {
		t.Error("expect length 1024, but got ", mb2.Len())
	}
	if mb.Len() != 7*1024 {
		t.Error("expect length 7*1024, but got ", mb.Len())
	}

	mb, mb3 := SplitSize(mb, 7*1024)
	if mb3.Len() != 7*1024 {
		t.Error("expect length 7*1024, but got", mb.Len())
	}

	if !mb.IsEmpty() {
		t.Error("expect empty buffer, but got ", mb.Len())
	}
}

```

这段代码定义了一个名为 `TestMultiBufferSplitFirst` 的测试函数，用于测试 `MultiBuffer` 和 `SplitFirst` 函数的行为。

在函数内部，首先创建了两个 `MultiBuffer` 对象 `mb1` 和 `mb2`，分别写入了字符串 "b1" 和 "b2"。然后，使用 `append` 方法将第三个字符串 "b3" 添加到 `mb` 对象中。

接着，使用 `SplitFirst` 函数将 `mb` 对象拆分成两个部分，分别分配给变量 `c1` 和 `c2`，并将拆分后的两个字符串存储到变量中。如果两个字符串不相等，函数会打印错误信息。

接下来，再次使用 `SplitFirst` 函数将 `mb` 对象拆分成两个部分，分别分配给变量 `c3` 和 `c2`，并将拆分后的两个字符串存储到变量中。如果两个字符串不相等，函数会打印错误信息。

最后，函数会打印一个判断条件 `if !mb.IsEmpty()`，如果 `mb` 对象不空，函数会打印错误信息。

整个函数的作用是测试 `MultiBuffer` 和 `SplitFirst` 函数的行为，包括将多个字符串存储到一个 `MultiBuffer` 对象中，并使用 `SplitFirst` 函数将该对象拆分成两个部分，并打印结果。


```go
func TestMultiBufferSplitFirst(t *testing.T) {
	b1 := New()
	b1.WriteString("b1")

	b2 := New()
	b2.WriteString("b2")

	b3 := New()
	b3.WriteString("b3")

	var mb MultiBuffer
	mb = append(mb, b1, b2, b3)

	mb, c1 := SplitFirst(mb)
	if diff := cmp.Diff(b1.String(), c1.String()); diff != "" {
		t.Error(diff)
	}

	mb, c2 := SplitFirst(mb)
	if diff := cmp.Diff(b2.String(), c2.String()); diff != "" {
		t.Error(diff)
	}

	mb, c3 := SplitFirst(mb)
	if diff := cmp.Diff(b3.String(), c3.String()); diff != "" {
		t.Error(diff)
	}

	if !mb.IsEmpty() {
		t.Error("expect empty buffer, but got ", mb.String())
	}
}

```



该代码的主要目的是测试一个名为 `MultiBufferReadAllToByte` 的函数。该函数的行为是在读取一个 8KB 大小的数据流时，它能够正确地将数据读取到内存中并向磁盘读取数据。

具体来说，代码分为两个部分。第一部分，代码使用 `make` 函数创建了一个 8KB 大小的缓冲区 `lb`，并使用 `io.ReadFull` 函数从从随机读取器中读取该缓冲区中的所有数据。然后，代码创建了一个新的字符缓冲器 `rd`，并使用 `ReadAllToBytes` 函数将缓冲区中的所有数据读取到该缓冲器中。如果一切正常，代码会关闭从随机读取器中读取的数据文件。

第二部分，代码打开名为 `data.dat` 的文件，并使用 `ReadAllToBytes` 函数读取该文件中的所有数据。然后，代码关闭文件并使用 `ioutil.ReadFile` 函数读取文件中的所有数据。最后，代码比较读取到的数据和从文件中读取的数据之间的差异，如果不相同，则错误。

该代码的目的是测试 `MultiBufferReadAllToByte` 函数的正确性，并确保它能够在正确的情况下将数据读取到内存中并向磁盘读取数据。


```go
func TestMultiBufferReadAllToByte(t *testing.T) {
	{
		lb := make([]byte, 8*1024)
		common.Must2(io.ReadFull(rand.Reader, lb))
		rd := bytes.NewBuffer(lb)
		b, err := ReadAllToBytes(rd)
		common.Must(err)

		if l := len(b); l != 8*1024 {
			t.Error("unexpceted length from ReadAllToBytes", l)
		}

	}
	{
		const dat = "data/test_MultiBufferReadAllToByte.dat"
		f, err := os.Open(dat)
		common.Must(err)

		buf2, err := ReadAllToBytes(f)
		common.Must(err)
		f.Close()

		cnt, err := ioutil.ReadFile(dat)
		common.Must(err)

		if d := cmp.Diff(buf2, cnt); d != "" {
			t.Error("fail to read from file: ", d)
		}
	}
}

```

这段代码的作用是测试一个名为 `MultiBufferCopy` 的函数。该函数的行为是对一个缓冲区（`[8字节]`）进行复制，并将其内容复制到另一个缓冲区（`[8字节]`）。

具体来说，这段代码首先创建了一个大小为 `8 * 1024` 的缓冲区（`[8字节]`），并将其内容通过 `rand.Reader` 输入到该缓冲区中。然后，代码创建了一个新的缓冲区（`[8字节]`）并将其设置为从原始缓冲区（`[8字节]`）中读取的内容。

接下来，代码调用 `ReadFrom` 函数，该函数从原始缓冲区中读取数据并将其存储在第一个缓冲区（`[8字节]`）中。然后，代码调用 `MultiBufferCopy` 函数，并将第一个缓冲区（`[8字节]`）的内容复制到第二个缓冲区（`[8字节]`）中。

最后，代码比较原始缓冲区和目标缓冲区之间的差异，并输出结果。如果两个缓冲区之间的差异与预期不符，函数将输出错误信息。


```go
func TestMultiBufferCopy(t *testing.T) {
	lb := make([]byte, 8*1024)
	common.Must2(io.ReadFull(rand.Reader, lb))
	reader := bytes.NewBuffer(lb)

	mb, err := ReadFrom(reader)
	common.Must(err)

	lbdst := make([]byte, 8*1024)
	mb.Copy(lbdst)

	if d := cmp.Diff(lb, lbdst); d != "" {
		t.Error("unexpceted different from MultiBufferCopy ", d)
	}
}

```

该代码段是针对Go标准库中的testing包进行编写的测试函数。函数名为TestSplitFirstBytes，它接收一个testing.T类型的参数，代表测试true。

函数的作用是测试splitFirstBytes函数的正确性。该函数需要两个参数，一个是MultiBuffer类型的变量mb，另一个是[byte]类型的变量o。

函数内部先创建两个MultiBuffer类型的变量a和b，分别写入字符串"ab"和"bc"。然后，创建一个MultiBuffer类型的变量mb，将a和b的内容合并到mb中。

接着，调用SplitFirstBytes函数，并将mb和o作为参数传入，得到两个[byte]类型的切片，cnt的值为2。然后，通过cmp.Diff函数比较d和"ab"的差值，期望的结果是空的切片([]byte{0})。如果cnt的值不等于2，或者d不是"ab"，都会导致函数失败并输出调试信息。

最后，函数没有返回任何值，它仅在测试失败时输出调试信息。


```go
func TestSplitFirstBytes(t *testing.T) {
	a := New()
	common.Must2(a.WriteString("ab"))
	b := New()
	common.Must2(b.WriteString("bc"))

	mb := MultiBuffer{a, b}

	o := make([]byte, 2)
	_, cnt := SplitFirstBytes(mb, o)
	if cnt != 2 {
		t.Error("unexpected cnt from SplitFirstBytes ", cnt)
	}
	if d := cmp.Diff(string(o), "ab"); d != "" {
		t.Error("unexpected splited result from SplitFirstBytes ", d)
	}
}

```

这道题目主要测试了两个函数的实现。第一个函数 `TestCompact` 是测试 Compact 函数的正确性。第二个函数 `BenchmarkSplitBytes` 则是测试 SplitBytes 函数的正确性。

具体来说，这两个函数都使用了 MultiBuffer 和 Compact 函数，并对其进行了测试。

MultiBuffer 是用来处理多行数据的包，它可以同时处理多行字符串。在 `TestCompact` 函数中，我们创建了两个 MultiBuffer，然后将它们合并成一个 MultiBuffer，并调用 Compact 函数对其进行压缩。最后，我们测试了 Compact 函数的输出是否正确。

而 `BenchmarkSplitBytes` 函数则测试 SplitBytes 函数的正确性。在测试中，我们创建了一个 MultiBuffer，然后使用 SplitBytes 函数将其行字符串进行分割。接着，我们对 SplitBytes 函数的结果进行测试，以确认其输出是否正确。


```go
func TestCompact(t *testing.T) {
	a := New()
	common.Must2(a.WriteString("ab"))
	b := New()
	common.Must2(b.WriteString("bc"))

	mb := MultiBuffer{a, b}
	cmb := Compact(mb)

	if w := cmb.String(); w != "abbc" {
		t.Error("unexpected Compact result ", w)
	}
}

func BenchmarkSplitBytes(b *testing.B) {
	var mb MultiBuffer
	raw := make([]byte, Size)

	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		buffer := StackNew()
		buffer.Extend(Size)
		mb = append(mb, &buffer)
		mb, _ = SplitBytes(mb, raw)
	}
}

```