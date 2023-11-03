# v2ray-core源码解析 69

# `transport/internet/kcp/segment_test.go`

这段代码是一个 Go 语言编写的测试用例，用于测试一个名为 "kcp" 的包。主要目的是通过验证 "BadSegment" 函数是否正确来评估 "kcp" 包的功能。

具体来说，这段代码以下几个步骤：

1. 导入 "testing" 和 "github.com/google/go-cmp/cmp" 和 "github.com/google/go-cmp/cmp/cmpopts" 包。

2. 定义一个名为 "TestBadSegment" 的函数。

3. 使用 "ReadSegment" 函数读取一个无序的 segment 并存储到 "seg" 和 "buf" 变量中。

4. 检查 "seg" 变量是否为空，如果不是，则输出一条错误消息。

5. 检查 "buf" 变量是否为空，如果不是，则输出一条错误消息。

6. 调用 "BadSegment" 函数，并传入一个有效的 segment 作为参数。

7. 检查 "BadSegment" 函数是否正确地处理了传入的参数，如果函数成功，则跳过这一步。

8. 没有其他代码，直接开始测试。


```go
package kcp_test

import (
	"testing"

	"github.com/google/go-cmp/cmp"
	"github.com/google/go-cmp/cmp/cmpopts"

	. "v2ray.com/core/transport/internet/kcp"
)

func TestBadSegment(t *testing.T) {
	seg, buf := ReadSegment(nil)
	if seg != nil {
		t.Error("non-nil seg")
	}
	if len(buf) != 0 {
		t.Error("buf len: ", len(buf))
	}
}

```

这段代码是一个名为 `TestDataSegment` 的函数，是用于测试数据段(DataSegment)的功能的。

具体来说，该函数的作用是创建一个数据段 `seg`，其中包含一个 `Conv` 为 1、`Timestamp` 为 3、`Number` 为 4、`SendingNext` 为 5 的数据。然后，`seg` 的数据域被写入一个字节数组 `bytes`。

接着，函数使用 `ReadSegment` 函数从 `bytes` 数组中读取一个 `DataSegment` 类型的值，并将其存储在 `seg2` 变量中。

最后，函数使用 `cmpopts.IgnoreUnexported(DataSegment{})` 选项，比较 `seg2` 和 `seg` 的内存分配和元素内容是否相等。如果两个 `DataSegment` 对象的内存分配不同或者它们的元素内容不相等，函数就会输出错误并返回 `t` 标志位。


```go
func TestDataSegment(t *testing.T) {
	seg := &DataSegment{
		Conv:        1,
		Timestamp:   3,
		Number:      4,
		SendingNext: 5,
	}
	seg.Data().Write([]byte{'a', 'b', 'c', 'd'})

	nBytes := seg.ByteSize()
	bytes := make([]byte, nBytes)
	seg.Serialize(bytes)

	iseg, _ := ReadSegment(bytes)
	seg2 := iseg.(*DataSegment)
	if r := cmp.Diff(seg2, seg, cmpopts.IgnoreUnexported(DataSegment{})); r != "" {
		t.Error(r)
	}
	if r := cmp.Diff(seg2.Data().Bytes(), seg.Data().Bytes()); r != "" {
		t.Error(r)
	}
}

```

该代码段是一个名为 "Test1ByteDataSegment" 的函数，旨在测试数据段（DataSegment）类型的函数。

具体来说，该函数创建了一个名为 "seg" 的 DataSegment 实例，该实例包含以下字段：

- Conv: 1，表示使用基数为 1 的二进制编码。
- Timestamp: 3，表示该数据段的创建时间。
- Number: 4，表示该数据段的编号。
- SendingNext: 5，表示数据段是否为发送方，如果是，则该字段为 1，否则为 0。

然后，该函数创建了一个名为 "bytes" 的字节数组，该数组大小等于 DataSegment 的 ByteSize() 字段，即 4 字节。

接下来，该函数使用该 DataSegment 实例的 Serialize() 函数将数据段序列化为字节数组，并将字节数组递归地传递给函数 cmp.Diff() 和 cmpopts.IgnoreUnexported()。

然后，该函数使用 ReadSegment() 函数从文件中读取一个名为 "seg.data" 的数据段实例，并将其存储在 "seg2" 变量中。

最后，该函数使用 cmp.Diff() 函数比较 "seg2" 和 "seg" 之间的差异，并在差异为空字符串时打印错误消息。如果差异不为空字符串，则打印错误消息并退出函数。


```go
func Test1ByteDataSegment(t *testing.T) {
	seg := &DataSegment{
		Conv:        1,
		Timestamp:   3,
		Number:      4,
		SendingNext: 5,
	}
	seg.Data().WriteByte('a')

	nBytes := seg.ByteSize()
	bytes := make([]byte, nBytes)
	seg.Serialize(bytes)

	iseg, _ := ReadSegment(bytes)
	seg2 := iseg.(*DataSegment)
	if r := cmp.Diff(seg2, seg, cmpopts.IgnoreUnexported(DataSegment{})); r != "" {
		t.Error(r)
	}
	if r := cmp.Diff(seg2.Data().Bytes(), seg.Data().Bytes()); r != "" {
		t.Error(r)
	}
}

```

该代码定义了一个名为 "TestACKSegment" 的函数，用于对一个ACKSegment对象的测试。

函数接收一个 testing.T 类型的参数，表示测试框架。

函数内部创建了一个 AckSegment 类型的实例，该实例包含以下字段：

- Conv: 1，表示segment 的构造函数。
- ReceivingWindow: 2，表示窗口大小，即正在等待确认的连续字节数。
- ReceivingNext: 3，表示连续确认的下一个字节的位置。
- Timestamp: 10，表示 segments 的时间戳。
- NumberList: []uint32{1, 3, 5, 7, 9}，表示 segment 中包含的数字列表。

该实例创建后，使用了 Serialize() 方法将其序列化为字节数组，然后使用 Bytes() 方法将其转换为 isegment 变量类型的字节数组。

接下来，函数使用 ReadSegment() 函数从源系统读取一个连续的字节数组，然后将其转换为 isegment 类型的变量。

然后，函数创建了一个 isegment 类型的变量 seg2，并将其与读取的连续字节数组进行比较，以检查它们是否相等。如果两个实例不相等，函数将打印错误并退出。


```go
func TestACKSegment(t *testing.T) {
	seg := &AckSegment{
		Conv:            1,
		ReceivingWindow: 2,
		ReceivingNext:   3,
		Timestamp:       10,
		NumberList:      []uint32{1, 3, 5, 7, 9},
	}

	nBytes := seg.ByteSize()
	bytes := make([]byte, nBytes)
	seg.Serialize(bytes)

	iseg, _ := ReadSegment(bytes)
	seg2 := iseg.(*AckSegment)
	if r := cmp.Diff(seg2, seg); r != "" {
		t.Error(r)
	}
}

```

这段代码的作用是测试一个名为 "TestCmdSegment" 的函数。该函数使用 "CmdOnlySegment" 类型的参数 "seg" 作为输入，并输出一个测试用例。具体来说，该函数创建了一个 "CmdOnlySegment" 类型的实例 "seg"，设置了其 "Conv" 为 1, "Cmd" 为 "CommandPing", "Option" 为 "SegmentOptionClose", "SendingNext" 为 11, "ReceivingNext" 为 13，以及 "PeerRTO" 为 15。然后，它创建了一个包含 "nBytes" 大小的字节数组 "bytes"，并将 "seg" 实例的 "Serialize" 方法应用于 "bytes" 数组，以将 "seg" 的参数传递给 "ReadSegment" 函数。接下来，它创建了一个名为 "iseg" 的整数类型变量，该变量存储了 "ReadSegment" 函数返回的整数类型值。最后，它使用 "cmp.Diff" 函数比较 "seg2" 和 "seg" 之间的差异，并将结果传递给 "t.Error" 函数。如果差异为空或两个参数之间的差异为 0，则函数不会输出任何错误信息。否则，函数会将错误信息传递给 "t.Error" 函数。


```go
func TestCmdSegment(t *testing.T) {
	seg := &CmdOnlySegment{
		Conv:          1,
		Cmd:           CommandPing,
		Option:        SegmentOptionClose,
		SendingNext:   11,
		ReceivingNext: 13,
		PeerRTO:       15,
	}

	nBytes := seg.ByteSize()
	bytes := make([]byte, nBytes)
	seg.Serialize(bytes)

	iseg, _ := ReadSegment(bytes)
	seg2 := iseg.(*CmdOnlySegment)
	if r := cmp.Diff(seg2, seg); r != "" {
		t.Error(r)
	}
}

```

# `transport/internet/kcp/sending.go`

这段代码定义了一个名为 `SendingWindow` 的结构体，它包含以下字段：

1. `cache`：一个 `list.List` 类型的变量，用于缓存已经发送的数据包。
2. `totalInFlightSize`：一个 `uint32` 类型的变量，用于记录正在发送的数据包总数。
3. `writer`：一个 `SegmentWriter` 类型的变量，用于向信源写入数据。
4. `onPacketLoss`：一个函数类型，用于在数据包丢失时执行。

`+build` 和 `!confonly` 是两个构建选项，用于编译时生成二进制文件和不产生配置文件。

具体来说，`SendingWindow` 结构体定义了一个 `SegmentWriter` 类型的变量 `writer`，用于向信源写入数据。`


```go
// +build !confonly

package kcp

import (
	"container/list"
	"sync"

	"v2ray.com/core/common/buf"
)

type SendingWindow struct {
	cache             *list.List
	totalInFlightSize uint32
	writer            SegmentWriter
	onPacketLoss      func(uint32)
}

```

该代码定义了一个名为 NewSendingWindow 的函数，它接收一个名为 writer 的参数和一个名为 onPacketLoss 的函数参数。函数返回一个指向 SendingWindow 的指针。

函数内部创建了一个 SendingWindow 对象，并初始化了其 cache、writer 和 onPacketLoss 函数。函数的另一个参数是一个指向 SendingWindow 的指针，该指针在函数中被解引用并输出。

函数的 Release 函数用于释放 SendingWindow 对象中的资源。函数首先检查 SendingWindow 是否为 nil，如果是，则直接返回。否则，函数会遍历 SendingWindow 的 cache 并删除front元素的值，然后删除 front 元素。最后，函数会清空 cache。


```go
func NewSendingWindow(writer SegmentWriter, onPacketLoss func(uint32)) *SendingWindow {
	window := &SendingWindow{
		cache:        list.New(),
		writer:       writer,
		onPacketLoss: onPacketLoss,
	}
	return window
}

func (sw *SendingWindow) Release() {
	if sw == nil {
		return
	}
	for sw.cache.Len() > 0 {
		seg := sw.cache.Front().Value.(*DataSegment)
		seg.Release()
		sw.cache.Remove(sw.cache.Front())
	}
}

```

这段代码定义了一个名为 `SendingWindow` 的结构体，它有两个方法 `Len()` 和 `IsEmpty()`，以及两个方法 `Push()` 和 `Pop()`。

`Len()` 方法返回发送方窗口的缓存长度，它将缓存中的所有元素一个一个地转换为 `uint32` 类型并返回。

`IsEmpty()` 方法返回发送方窗口是否为空，如果缓存中没有元素，则返回 `true`，否则返回 `false`。

`Push()` 方法将一个 `uint32` 类型的数据和一个 `buf.Buffer` 类型的缓冲区 `b` 压入到发送方窗口的缓存中。它使用 `NewDataSegment()` 函数创建一个新的 `DataSegment` 对象，其中 `number` 字段存储 `uint32` 类型的数据，`payload` 字段存储 `buf.Buffer` 类型的缓冲区。然后，它将 `DataSegment` 对象添加到缓存中，使用 `sw.cache.PushBack()` 方法将数据添加到缓存中。

`Pop()` 方法从发送方窗口的缓存中弹出一个 `uint32` 类型的数据，并将其存储在变量 `number` 中。如果缓存为空，它将返回 `nil`。


```go
func (sw *SendingWindow) Len() uint32 {
	return uint32(sw.cache.Len())
}

func (sw *SendingWindow) IsEmpty() bool {
	return sw.cache.Len() == 0
}

func (sw *SendingWindow) Push(number uint32, b *buf.Buffer) {
	seg := NewDataSegment()
	seg.Number = number
	seg.payload = b

	sw.cache.PushBack(seg)
}

```

这两函数接收一个 SendingWindow 类型的参数，并返回一个 uint32 类型的变量 FirstNumber 和一个函数 Clear。

函数 FirstNumber() 接收一个 SendingWindow 类型的参数，返回该参数的 cache 中 front 位置的 DataSegment 类型的 Number 变量。

函数 Clear() 接收一个 uint32 类型的变量 一个 UnAskedUint32，用于指示要删除的数据包序号。该函数会遍历 SendingWindow 的 cache，若当前要删除的数据包序号已经存在于 cache 中，则跳出循环。否则，会从 cache 的 front 位置移除该数据包，并更新 front 位置指针。


```go
func (sw *SendingWindow) FirstNumber() uint32 {
	return sw.cache.Front().Value.(*DataSegment).Number
}

func (sw *SendingWindow) Clear(una uint32) {
	for !sw.IsEmpty() {
		seg := sw.cache.Front().Value.(*DataSegment)
		if seg.Number >= una {
			break
		}
		seg.Release()
		sw.cache.Remove(sw.cache.Front())
	}
}

```

这段代码定义了一个名为 `HandleFastAck` 的函数，接收两个uint32类型的参数 `sw` 和 `rto`。

该函数的主要作用是处理 fast ack 请求。fast ack 是一种在传输过程中允许发送方在收到确认前重传数据的技术，可以提高网络传输的可靠性。

函数内部首先检查传递给 `HandleFastAck` 的 `sw` 是否为空，如果是，则直接返回，因为为空时没有数据需要处理 fast ack 请求。

如果 `sw` 不是空，函数会遍历 `sw` 中的所有 `DataSegment` 结构体，并且对于每个 `DataSegment`，函数会递归地执行以下操作：

1. 如果 `number` 等于 `seg.Number` 或 `number - seg.Number` 大于或等于 0x7FFFFFFF，函数返回 `false`。这是 fast ack 请求的一个条件，发送方知道接收方已经接收到了数据，可以开始发送新的数据。
2. 如果 `seg.transmit` 大于 0 且 `seg.timeout` 大于 `rto / 3`，函数会减小 `rto` 除以 3，以便发送方知道 fast ack 请求的延迟时间。
3. 函数返回 `true`，表示已经处理完了 fast ack 请求。

该函数可以在使用 fast ack 技术时，提高网络传输的可靠性。


```go
func (sw *SendingWindow) HandleFastAck(number uint32, rto uint32) {
	if sw.IsEmpty() {
		return
	}

	sw.Visit(func(seg *DataSegment) bool {
		if number == seg.Number || number-seg.Number > 0x7FFFFFFF {
			return false
		}

		if seg.transmit > 0 && seg.timeout > rto/3 {
			seg.timeout -= rto / 3
		}
		return true
	})
}

```

这两函数是用于 SendingWindow 的 Visit 和 Flush 方法。

Visit 函数接收一个 visitor 函数作为参数，该函数接收一个 DataSegment 对象作为参数。如果 visitor 函数返回 false，则立即退出循环。否则，继续循环并访问下一个 DataSegment 对象。

Flush 函数用于将数据包从发送窗口中异步发送到接收窗口。发送窗口维护当前正在发送的数据包和已发送但还未确认的数据包，当发送窗口为空时，函数将返回。

函数接收一个 current uint32 表示发送窗口的当前大小，一个 rto uint32 表示发送窗口中的最大数据包大小，和一个 maxInFlightSize uint32 表示发送窗口可以同时发送的最大数据包数量。

如果发送窗口为空，函数直接返回。否则，函数首先计算丢失的数据包数量，然后使用 visitor 函数处理每个 DataSegment 对象。如果 DataSegment 对象发送失败，函数将增加丢失的数据包数量，并使用 onPacketLoss 函数通知相关人员。最后，函数将 maxInFlightSize 内的数据包发送到接收窗口。


```go
func (sw *SendingWindow) Visit(visitor func(seg *DataSegment) bool) {
	if sw.IsEmpty() {
		return
	}

	for e := sw.cache.Front(); e != nil; e = e.Next() {
		seg := e.Value.(*DataSegment)
		if !visitor(seg) {
			break
		}
	}
}

func (sw *SendingWindow) Flush(current uint32, rto uint32, maxInFlightSize uint32) {
	if sw.IsEmpty() {
		return
	}

	var lost uint32
	var inFlightSize uint32

	sw.Visit(func(segment *DataSegment) bool {
		if current-segment.timeout >= 0x7FFFFFFF {
			return true
		}
		if segment.transmit == 0 {
			// First time
			sw.totalInFlightSize++
		} else {
			lost++
		}
		segment.timeout = current + rto

		segment.Timestamp = current
		segment.transmit++
		sw.writer.Write(segment)
		inFlightSize++
		return inFlightSize < maxInFlightSize
	})

	if sw.onPacketLoss != nil && inFlightSize > 0 && sw.totalInFlightSize != 0 {
		rate := lost * 100 / sw.totalInFlightSize
		sw.onPacketLoss(rate)
	}
}

```

该函数名为 `func (sw *SendingWindow) Remove(number uint32) bool`，其作用是移除传入的 `sw` 对象中标识为 `number` 的数据报文，并返回是否成功。

函数首先判断 `sw` 对象是否为空，如果是，则直接返回 `false`。接下来，函数遍历 `sw` 对象中的所有数据报文，并为每个数据报文创建一个 `DataSegment` 对象。然后，函数会比较当前数据报文的 `number` 值是否大于传入的 `number` 参数，如果是，则返回 `false`，否则继续执行后续操作。

如果当前数据报文的 `number` 值与传入参数相同，那么函数会检查 `sw` 对象中已经存在的数据报文是否已经被处理。如果是，则移除它，并更新 `sw.totalInFlightSize` 变量；如果还没有被处理，则继续执行接下来的操作，并返回 `true`。最后，函数会返回 `false`，表示无法从 `sw` 对象中移除标识为 `number` 的数据报文。


```go
func (sw *SendingWindow) Remove(number uint32) bool {
	if sw.IsEmpty() {
		return false
	}

	for e := sw.cache.Front(); e != nil; e = e.Next() {
		seg := e.Value.(*DataSegment)
		if seg.Number > number {
			return false
		} else if seg.Number == number {
			if sw.totalInFlightSize > 0 {
				sw.totalInFlightSize--
			}
			seg.Release()
			sw.cache.Remove(e)
			return true
		}
	}

	return false
}

```

该代码定义了一个名为 `SendingWorker` 的结构体，表示一个发送方 workers 组件。该组件包含以下字段：

- `conn` 字段是一个指针，指向一个 `Connection` 对象。
- `window` 字段是一个指针，指向一个 `SendingWindow` 对象。
- `firstUnacknowledged` 字段是一个计数器，用于跟踪已经发送的数据量。
- `nextNumber` 字段是一个计数器，用于跟踪下一个需要发送的包的数量。
- `remoteNextNumber` 字段是一个计数器，用于跟踪远程下一个需要发送的包的数量。
- `controlWindow` 字段是一个整数，用于指示是否应该接收控制消息并启动控制窗口。
- `fastResend` 字段是一个整数，用于指示是否应该尽快重发丢失的数据包。
- `windowSize` 字段是一个整数，用于指示窗口中可以存储的最大数据量。
- `closed` 字段是一个布尔，用于指示worker是否已经关闭。

此外，该代码还实现了一个名为 `NewSendingWorker` 的函数，该函数接受一个 `Connection` 对象作为参数，并返回一个指向新 `SendingWorker` 对象的指针。该函数还设置了一些默认值，例如 `fastResend` 为 2,`remoteNextNumber` 为 32,`controlWindow` 为 `kcp.Config.GetSendingInFlightSize()`。


```go
type SendingWorker struct {
	sync.RWMutex
	conn                       *Connection
	window                     *SendingWindow
	firstUnacknowledged        uint32
	nextNumber                 uint32
	remoteNextNumber           uint32
	controlWindow              uint32
	fastResend                 uint32
	windowSize                 uint32
	firstUnacknowledgedUpdated bool
	closed                     bool
}

func NewSendingWorker(kcp *Connection) *SendingWorker {
	worker := &SendingWorker{
		conn:             kcp,
		fastResend:       2,
		remoteNextNumber: 32,
		controlWindow:    kcp.Config.GetSendingInFlightSize(),
		windowSize:       kcp.Config.GetSendingBufferSize(),
	}
	worker.window = NewSendingWindow(worker, worker.OnPacketLoss)
	return worker
}

```

这段代码定义了两个函数：Release()和ProcessReceivingNext()。

Release()函数的作用是确保所有资源都被正确释放，并使Worker关闭。具体来说，它使用双重锁定模式，确保在期间任何其他进程正在访问资源。然后，它关闭了Worker的窗口，并设置closed变量为true，这将在Worker关闭时通知所有连接的客户端。最后，它使用Unlock()方法解除双重锁定。

ProcessReceivingNext()函数的作用是在Worker窗口中处理接收下一个数据包的任务。具体来说，它使用双重锁定模式，确保在期间任何其他进程都没有访问窗口。然后，它调用窗口的Clear()方法，清空窗口的内容，并使用FindFirstUnacknowledged()方法查找第一个未acknowledged的窗口。

由于没有设置该函数的权限，因此它无法输出任何信息。


```go
func (w *SendingWorker) Release() {
	w.Lock()
	w.window.Release()
	w.closed = true
	w.Unlock()
}

func (w *SendingWorker) ProcessReceivingNext(nextNumber uint32) {
	w.Lock()
	defer w.Unlock()

	w.ProcessReceivingNextWithoutLock(nextNumber)
}

func (w *SendingWorker) ProcessReceivingNextWithoutLock(nextNumber uint32) {
	w.window.Clear(nextNumber)
	w.FindFirstUnacknowledged()
}

```

这两函数的主要目的是在发送worker中处理acknowledgment。函数1func (w *SendingWorker) FindFirstUnacknowledged()会尝试从窗口中删除编号最小的acknowledgment，如果删除成功则将第一次acknowledgment设置为当前acknowledgment。如果删除失败，则将nextNumber设置为当前acknowledgment。函数2func (w *SendingWorker) processAck(number uint32)会检查给定的acknowledgment是否已删除或者可以删除。如果acknowledgment已删除，则返回false。否则，函数将尝试从nextNumber中删除给定编号的acknowledgment。


```go
func (w *SendingWorker) FindFirstUnacknowledged() {
	first := w.firstUnacknowledged
	if !w.window.IsEmpty() {
		w.firstUnacknowledged = w.window.FirstNumber()
	} else {
		w.firstUnacknowledged = w.nextNumber
	}
	if first != w.firstUnacknowledged {
		w.firstUnacknowledgedUpdated = true
	}
}

func (w *SendingWorker) processAck(number uint32) bool {
	// number < v.firstUnacknowledged || number >= v.nextNumber
	if number-w.firstUnacknowledged > 0x7FFFFFFF || number-w.nextNumber < 0x7FFFFFFF {
		return false
	}

	removed := w.window.Remove(number)
	if removed {
		w.FindFirstUnacknowledged()
	}
	return removed
}

```

该函数的作用是处理数据传输中的一个 segment。具体来说，它接收一个接收窗口（AckSegment）和一个发送窗口（SendingWorker）。当接收到数据时，函数会根据接收窗口的大小和当前已接收到的数据量来更新发送窗口。

以下是函数的详细解释：

1. 函数接收一个接收窗口（AckSegment）和一个发送窗口（SendingWorker）。
2. 函数首先检查发送窗口是否关闭。如果是，函数返回，因为关闭的发送窗口不会接收数据。
3. 如果发送窗口没有关闭，函数将继续检查接收到数据后是否可以更新发送窗口。
4. 如果可以更新发送窗口，函数会根据接收窗口的大小和当前已接收到的数据量来更新发送窗口。
5. 如果发送窗口已经更新，函数接下来会处理接收窗口。
6. 如果接收窗口为空，函数会返回，因为接收窗口为空表示没有新数据可以接收。
7. 最后，函数会处理接收窗口。如果接收窗口为空，函数会使用函数内置的 `w.processAck` 函数来接收之前发送的数据，然后发送回给发送窗口。如果接收窗口不为空，函数会将新数据添加到发送窗口。
8. 函数最后，如果发送窗口已更新，函数会将更新后的发送窗口发送给发送端。

综上所述，该函数的作用是更新发送窗口以接收新的数据，并处理接收窗口。


```go
func (w *SendingWorker) ProcessSegment(current uint32, seg *AckSegment, rto uint32) {
	defer seg.Release()

	w.Lock()
	defer w.Unlock()

	if w.closed {
		return
	}

	if w.remoteNextNumber < seg.ReceivingWindow {
		w.remoteNextNumber = seg.ReceivingWindow
	}
	w.ProcessReceivingNextWithoutLock(seg.ReceivingNext)

	if seg.IsEmpty() {
		return
	}

	var maxack uint32
	var maxackRemoved bool
	for _, number := range seg.NumberList {
		removed := w.processAck(number)
		if maxack < number {
			maxack = number
			maxackRemoved = removed
		}
	}

	if maxackRemoved {
		w.window.HandleFastAck(maxack, rto)
		if current-seg.Timestamp < 10000 {
			w.conn.roundTrip.Update(current-seg.Timestamp, current)
		}
	}
}

```

该函数名为 `Push`，定义在指针变量 `w` 上，其作用是确保 `SendingWorker` 对象 `w` 正确地接收到了 `buf.Buffer` 类型的数据。

具体来说，函数的实现分为以下几个步骤：

1. 首先，函数创建了一个名为 `w` 的 `SendingWorker` 对象，并获取了锁 `w.Lock()`，以确保在函数内部对 `w` 进行任何读或写操作时，其他协程程序员不能访问 `w`。

2. 接着，函数检查 `w` 对象是否已经关闭，如果是，则直接返回 `false`，因为关闭的 `SendingWorker` 对象无法接收新数据。

3. 然后，函数检查 `w` 对象的 `window` 数组是否已经满了，如果是，则返回 `false`，因为满了的 `window` 数组无法再接收新数据。

4. 接下来，函数确保 `w.nextNumber` 加上 `w.windowSize` 等于 `w.window` 数组的下一个元素，然后将 `w.nextNumber` 和 `w.windowSize` 指针都自增 1，并将 `w.window` 数组中的元素 Push 过去。

5. 最后，函数返回 `true`，表示 `w` 对象正确接收到了 `buf.Buffer` 类型的数据。


```go
func (w *SendingWorker) Push(b *buf.Buffer) bool {
	w.Lock()
	defer w.Unlock()

	if w.closed {
		return false
	}

	if w.window.Len() > w.windowSize {
		return false
	}

	w.window.Push(w.nextNumber, b)
	w.nextNumber++
	return true
}

```

这两函数分别定义在函数式编程风格中，主要作用是实现发送方确认包（Segment）的发送和丢失处理逻辑。

第一个函数 `func (w *SendingWorker) Write(seg Segment) error` 接收一个 `Segment` 类型的参数，将其包装并输出到 `w` 的 `conn` 连接上。`Write` 函数的行为取决于以下几个参数：

1. `dataSeg`：作为 `Segment` 类型的参数，可能是接收方确认包（Segment）。
2. `conn`：`Write` 函数的输出目标 `conn`。
3. `w`：`SendingWorker` 类型的参数，表示正在工作的发送方客户端。

函数的主要逻辑包括：

1. 解析接收方确认包（`Segment`）。
2. 更新发送方控制窗口（Control Window）。
3. 如果发送方连接处于关闭状态，设置控制窗口为 `SegmentOptionClose`。
4. 如果连接没有设置为关闭状态，且超时，设置控制窗口为 `SegmentOptionHangUp`。
5. 调用 `conn.output.Write` 并输出接收方确认包（`Segment`）。

第二个函数 `func (w *SendingWorker) OnPacketLoss(lossRate uint32)` 接收一个损失率（`uint32`）参数，然后执行以下操作：

1. 如果连接没有设置为关闭状态，或者没有超时，则执行以下操作：
	* 如果损失率小于或等于 15，则不做任何操作。
	* 如果损失率大于 15，设置 `w.controlWindow` 为 3 * `w.controlWindow` / 4。
	* 如果 `w.controlWindow` 小于 16，设置为 16。
	* 如果 `w.controlWindow` 大于或等于 2 * `w.conn.Config.GetSendingInFlightSize()`，设置为 2 * `w.conn.Config.GetSendingInFlightSize()`。
2. 如果连接已设置为关闭状态，或者已经超时，执行以下操作：
	* 设置 `w.controlWindow` 为 2 * `w.conn.Config.GetSendingInFlightSize()`。
	* 设置 `w.controlWindow` 为 16。
	* 停止执行其他操作。


```go
func (w *SendingWorker) Write(seg Segment) error {
	dataSeg := seg.(*DataSegment)

	dataSeg.Conv = w.conn.meta.Conversation
	dataSeg.SendingNext = w.firstUnacknowledged
	dataSeg.Option = 0
	if w.conn.State() == StateReadyToClose {
		dataSeg.Option = SegmentOptionClose
	}

	return w.conn.output.Write(dataSeg)
}

func (w *SendingWorker) OnPacketLoss(lossRate uint32) {
	if !w.conn.Config.Congestion || w.conn.roundTrip.Timeout() == 0 {
		return
	}

	if lossRate >= 15 {
		w.controlWindow = 3 * w.controlWindow / 4
	} else if lossRate <= 5 {
		w.controlWindow += w.controlWindow / 4
	}
	if w.controlWindow < 16 {
		w.controlWindow = 16
	}
	if w.controlWindow > 2*w.conn.Config.GetSendingInFlightSize() {
		w.controlWindow = 2 * w.conn.Config.GetSendingInFlightSize()
	}
}

```

此函数的作用是处理发送者窗口中的数据并将其传递给接收者。以下是函数的步骤：

1. 首先获取发送者窗口中当前要发送的数据量，并检查是否已关闭并尝试获取输入操作权限。
2. 如果正在运行且已经关闭，则允许函数退出并返回。
3. 如果正在运行，则获取当前正在传输到下一个服务器的主机上的数据量，并检查是否已收到延迟数据。
4. 如果正在接收延迟数据，则增加当前发送的数据量，以确保发送者窗口中的数据量不会超过可接受的最大值。
5. 如果发送者窗口为空，则将当前数据量发送给接收者。
6. 更新发送者窗口中已发送的数据量。
7. 如果还没有发送数据，则发送一个心跳包给发送者以获取确认消息。

函数的实现允许发送者窗口中的数据按照延迟数据和当前数据量的更优先级进行传递，以确保接收者能够及时收到数据。通过检查是否正在接收延迟数据，函数可以避免不必要的数据丢失。


```go
func (w *SendingWorker) Flush(current uint32) {
	w.Lock()

	if w.closed {
		w.Unlock()
		return
	}

	cwnd := w.firstUnacknowledged + w.conn.Config.GetSendingInFlightSize()
	if cwnd > w.remoteNextNumber {
		cwnd = w.remoteNextNumber
	}
	if w.conn.Config.Congestion && cwnd > w.firstUnacknowledged+w.controlWindow {
		cwnd = w.firstUnacknowledged + w.controlWindow
	}

	if !w.window.IsEmpty() {
		w.window.Flush(current, w.conn.roundTrip.Timeout(), cwnd)
		w.firstUnacknowledgedUpdated = false
	}

	updated := w.firstUnacknowledgedUpdated
	w.firstUnacknowledgedUpdated = false

	w.Unlock()

	if updated {
		w.conn.Ping(current, CommandPing)
	}
}

```

这是一个 Go 语言中的函数指针类型，它接收一个名为 SendingWorker 的 *SendingWorker 类型的参数。

func (w *SendingWorker) CloseWrite() {
	w.Lock()
	defer w.Unlock()

	w.window.Clear(0xFFFFFFFF)
}

func (w *SendingWorker) IsEmpty() bool {
	w.RLock()
	defer w.RUnlock()

	return w.window.IsEmpty()
}

func (w *SendingWorker) UpdateNecessary() bool {
	return !w.IsEmpty()
}

这个函数指针是一个指针变量 w，它代表一个 SendingWorker 类型的对象。

该函数指针使用Lock和Unlock函数来保护对 SendingWorker 对象的引用。

- CloseWrite() 函数使用 w.Lock() 函数获取当前锁定状态，使用 w.Unlock() 函数释放当前锁定状态。然后，它调用 w.window.Clear() 函数来清空屏幕，并返回一个 bool 值以表示是否已写入数据。
- IsEmpty() 函数使用 w.RLock() 函数获取当前锁定状态，使用 w.RUnlock() 函数释放当前锁定状态。然后，它返回一个 bool 值以表示是否已写入数据。
- UpdateNecessary() 函数使用 w.RLock() 函数获取当前锁定状态，使用 w.RUnlock() 函数释放当前锁定状态。然后，它返回一个 bool 值以表示是否需要更新屏幕。


```go
func (w *SendingWorker) CloseWrite() {
	w.Lock()
	defer w.Unlock()

	w.window.Clear(0xFFFFFFFF)
}

func (w *SendingWorker) IsEmpty() bool {
	w.RLock()
	defer w.RUnlock()

	return w.window.IsEmpty()
}

func (w *SendingWorker) UpdateNecessary() bool {
	return !w.IsEmpty()
}

```

该函数名为`FirstUnacknowledged`，接收一个名为`w`的`SendingWorker`类型的参数。

函数的作用是获取`SendingWorker`中第一个未被确认的`Unacknowledged`消息的`acknowledged`时间戳（即`Unacknowledged`消息被确认的时间戳减去`send`的时间戳）。

具体实现包括：

1. 使用`RLock`函数对`w`对象与其`RLock`对象（即`SendingWorker`对象的锁）进行互斥锁操作，以确保在函数中只有一个`Unacknowledged`消息被返回。
2. 使用`defer`关键字，确保在函数中执行的操作是在`Unacknowledged`消息被返回之前完成的（不会卡在函数中）。
3. 函数返回`w.firstUnacknowledged`，即`SendingWorker`中第一个未被确认的`Unacknowledged`消息的`acknowledged`时间戳。


```go
func (w *SendingWorker) FirstUnacknowledged() uint32 {
	w.RLock()
	defer w.RUnlock()

	return w.firstUnacknowledged
}

```

# `transport/internet/kcp/xor.go`

这两函数一起作用于一个字节数组x，实现了字符串中的XOR操作。

具体来说，这两个函数都是对数组x中的每个元素，进行一系列XOR操作，然后将结果赋值回原来的元素。

这两个函数实现不同的是，一个是对向进行操作，另一个是反向进行操作。

函数`xorfwd`实现的是向量XOR操作，具体来说，它对数组x中的每个元素，从第二个元素开始，向前走四个元素，然后将结果赋值回原来的元素。

函数`xorbkd`实现的是反向的XOR操作，具体来说，它对数组x中的每个元素，从最后一个元素开始，向前走四个元素，然后将结果赋值回原来的元素。

由于这两个函数都涉及到对数组x的元素进行XOR操作，因此它们的结果也是相同的，都是对数组x中的元素进行了交换操作。


```go
// +build !amd64

package kcp

// xorfwd performs XOR forwards in words, x[i] ^= x[i-4], i from 0 to len
func xorfwd(x []byte) {
	for i := 4; i < len(x); i++ {
		x[i] ^= x[i-4]
	}
}

// xorbkd performs XOR backwords in words, x[i] ^= x[i-4], i from len to 0
func xorbkd(x []byte) {
	for i := len(x) - 1; i >= 4; i-- {
		x[i] ^= x[i-4]
	}
}

```

# `transport/internet/kcp/xor_amd64.go`

这两函数是实现 XML 转 ASCII 编码的库函数。

函数 xorfwd 将输入的 x 字节数组进行 XML 编码，并输出结果。

函数 xorjkd 将输入的 x 字节数组进行 ASCII 编码，并输出结果。

这两函数的实现较为简单，主要通过创建一个子函数来实现。在函数内部，使用 go:noescape 修饰符来禁止输出函数内部的字符串，防止意外的输出发生。


```go
package kcp

//go:noescape
func xorfwd(x []byte)

//go:noescape
func xorbkd(x []byte)

```

# `transport/internet/quic/config.go`

这段代码是一个 Go 语言编写的 build 命令，用于构建quic 加密协议的源代码。它使用了以下一些工具：

1. `build` 命令：这将使用 Go 的 `go build` 工具构建 Go 语言项目。
2. `!confonly`：这是一个实验性选项，用于仅在计算机上编译 Go 语言源代码，而不是将其包装成一个依赖项。

更具体地说，这个 build 命令将在当前目录下创建一个名为 `quic-build.go` 的二进制文件，这个文件将包含 Go 语言编译器编写的二进制文件。这个文件将作为 `quic-build` 包的入口点，用于编译 quic 加密协议的源代码。


```go
// +build !confonly

package quic

import (
	"crypto/aes"
	"crypto/cipher"
	"crypto/sha256"

	"golang.org/x/crypto/chacha20poly1305"
	"v2ray.com/core/common"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/transport/internet"
)

```

这段代码定义了一个名为 `getAuth` 的函数，接收一个名为 `config` 的字符串参数。函数的作用是获取一个加密认证，并返回加密认证和错误。

函数首先根据 `config` 中的安全策略类型来判断是否支持加密认证。如果安全策略类型为 `protocol.SecurityType_NONE`，那么直接返回 `nil` 和 `nil`。否则，函数会执行以下步骤：

1. 从 `config` 中提取一个名为 `salted` 的字节数组，该数组包含了一个 V2Ray 加密隧道信息。
2. 计算一个 SHA256 哈希值，该哈希值将 `salted` 数组作为输入，并采用一个特定的安全策略。
3. 如果安全策略类型为 `protocol.SecurityType_AES128_GCM`，那么使用 AES-128-GCM 算法创建一个加密器。
4. 如果安全策略类型为 `protocol.SecurityType_CHACHA20_POLY1305`，那么使用 CHAKE20-POLY1305 算法创建一个加密器。
5. 如果安全策略类型为 `protocol.SecurityType_NONE`，或者是上面步骤计算得到的哈希值无法匹配任何一个安全策略，那么返回一个错误并 `nil`。

函数的实现使用了多种安全策略，包括 AES-128-GCM、CHAKE20-POLY1305 和 V2Ray 加密隧道。函数的作用是帮助用户根据他们的安全策略选择一个适当的加密认证。


```go
func getAuth(config *Config) (cipher.AEAD, error) {
	security := config.Security.GetSecurityType()
	if security == protocol.SecurityType_NONE {
		return nil, nil
	}

	salted := []byte(config.Key + "v2ray-quic-salt")
	key := sha256.Sum256(salted)

	if security == protocol.SecurityType_AES128_GCM {
		block, err := aes.NewCipher(key[:16])
		common.Must(err)
		return cipher.NewGCM(block)
	}

	if security == protocol.SecurityType_CHACHA20_POLY1305 {
		return chacha20poly1305.New(key[:])
	}

	return nil, newError("unsupported security type")
}

```

这段代码定义了一个名为 `getHeader` 的函数，接受一个名为 `config` 的 pointer参数，并返回一个名为 `internet.PacketHeader` 的结构体和一个名为 `error` 的整型变量。

函数首先检查传入的 `config` 参数是否为 `nil`，如果是，则返回 `nil` 和 `nil`，表示函数无法提供任何有用的信息。否则，函数将尝试从传入的 `config` 参数的 `Header` 字段中获取数据，如果失败，则返回 `nil` 和 `err`，表示函数在获取 Header 时遇到了问题。

接下来，函数调用了一个名为 `config.Header.GetInstance` 的函数，这个函数会尝试从 `config.Header` 的字节切片中获取数据，并返回一个指向 `internet.PacketHeader` 类型的变量。如果这个函数返回 `nil`，则代表 Header 数据不可用，函数将返回 `nil` 和 `err`，表示函数在获取 Header 时遇到了问题。如果这个函数返回一个有效的 `internet.PacketHeader` 类型，则代表 Header 数据可用，函数将返回该 `internet.PacketHeader` 类型的实例。

最后，函数调用 `internet.CreatePacketHeader` 函数，将 `internet.PacketHeader` 类型的数据创建成 `internet.PacketHeader` 类型，并返回它。这个 `internet.PacketHeader` 类型包含了 `Message` 和 `Header` 字段，其中 `Message` 字段包含了原始数据，`Header` 字段包含了 Header 信息。


```go
func getHeader(config *Config) (internet.PacketHeader, error) {
	if config.Header == nil {
		return nil, nil
	}

	msg, err := config.Header.GetInstance()
	if err != nil {
		return nil, err
	}

	return internet.CreatePacketHeader(msg)
}

```

# `transport/internet/quic/config.pb.go`

这段代码定义了一个名为 "quic" 的包，其作用是创建一个名为 "transport/internet/quic/config" 的配置 protobuf 文件。

具体来说，这个包通过导入 "protoc-gen-go" 和 "protoc" 两个包，来生成go语言中的配置文件。然后，通过使用 reflect 和 sync 包来处理生成的配置文件中的语法和结构，最后导出 "v2ray.com/core/common/protocol" 和 "v2ray.com/core/common/serial" 两个包。

这个quic包主要使用了protobuf协议来定义网络通信中的配置信息，通过protoc-gen-go来生成go语言中的.proto文件，然后通过反射和sync包来处理生成的配置文件中的语法和结构，最后通过protocol和serial包来将这些配置信息应用到实际的网络通信中。


```go
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
// 	protoc-gen-go v1.25.0
// 	protoc        v3.13.0
// source: transport/internet/quic/config.proto

package quic

import (
	proto "github.com/golang/protobuf/proto"
	protoreflect "google.golang.org/protobuf/reflect/protoreflect"
	protoimpl "google.golang.org/protobuf/runtime/protoimpl"
	reflect "reflect"
	sync "sync"
	protocol "v2ray.com/core/common/protocol"
	serial "v2ray.com/core/common/serial"
)

```

这段代码是一个高亮代码，它表达了两个关键点：

1. 验证所编写的代码是否与所依赖的依赖项兼容。
2. 验证运行时/protoimpl是否足够新。

具体来说，代码中使用了两个不同的 protobuf 版本：

- `protoimpl.MinVersion` 和 `protoimpl.MaxVersion` 确定所依赖的旧版本和版本。
- `_ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)` 和 `_ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)` 分别用于验证最低和最高依赖版本。

此外，代码中定义了一个 `Config` 类型，其中包含了一些 protobuf 字段，包括 `key`、`security` 和 `header`。这些字段定义了文档中的配置项。


```go
const (
	// Verify that this generated code is sufficiently up-to-date.
	_ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
	// Verify that runtime/protoimpl is sufficiently up-to-date.
	_ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// This is a compile-time assertion that a sufficiently up-to-date version
// of the legacy proto package is being used.
const _ = proto.ProtoPackageIsVersion4

type Config struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Key      string                   `protobuf:"bytes,1,opt,name=key,proto3" json:"key,omitempty"`
	Security *protocol.SecurityConfig `protobuf:"bytes,2,opt,name=security,proto3" json:"security,omitempty"`
	Header   *serial.TypedMessage     `protobuf:"bytes,3,opt,name=header,proto3" json:"header,omitempty"`
}

```

这段代码定义了两个函数：

1. `Reset()` 函数接收一个指向 `Config` 类型对象的 `x` 参数，并将其赋值为 `Config{}`。然后，它检查 `protoimpl.UnsafeEnabled` 是否为真，如果是，则执行以下操作：

  - 创建一个空的 `file_transport_internet_quic_config_proto_msgTypes` 类型对象，并将其赋值给 `mi` 变量。
  - 使用 `protoimpl.X.MessageStateOf` 函数将 `x` 对象的状态存储到 `ms` 变量中。
  - 最后，将 `mi` 和 `ms` 对象存储的消息信息存储到 `x` 参数的复制中。

2. `String()` 函数接收一个指向 `Config` 类型对象的 `x` 参数，并返回 `protoimpl.X.MessageStringOf(x)` 的结果。这个函数主要用来将 `x` 对象的字符串表示出来。

3. `ProtoMessage()` 函数是一个通用函数，它接收一个指向 `Config` 类型对象的 `x` 参数，并返回一个空对象。这个函数用于将 `x` 对象转换为字节序列，以便将其传输到远程，并将其转换为 JSON 对象。


```go
func (x *Config) Reset() {
	*x = Config{}
	if protoimpl.UnsafeEnabled {
		mi := &file_transport_internet_quic_config_proto_msgTypes[0]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *Config) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*Config) ProtoMessage() {}

```

这段代码定义了两个函数，分别接收一个*Config类型的参数x，并返回相应的函数值。

第一个函数名为func (x *Config) ProtoReflect() protoreflect.Message {，它接收一个*Config类型的参数x，并尝试使用*protoimpl.UnsafeEnabled选项来创建一个Message类型的指针。如果*protoimpl.UnsafeEnabled为真，则尝试使用x的MessageStateOf函数来获取Message类型的信息，并将其存储为mi。最后，如果mi已经被创建，则直接返回mi，否则创建一个新的Message类型的指针mi，并将其存储为mi。

第二个函数名为func (*Config) Descriptor() ([]byte, []int)，它接收一个*Config类型的参数x，并返回一个描述符，其中第一个元素是描述符的编码类型，第二个元素是描述符的长度。它通过调用file_transport_internet_quic_config_proto_rawDescGZIP函数来获取描述符的原始代码，然后将其转换为字节切片并返回。


```go
func (x *Config) ProtoReflect() protoreflect.Message {
	mi := &file_transport_internet_quic_config_proto_msgTypes[0]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use Config.ProtoReflect.Descriptor instead.
func (*Config) Descriptor() ([]byte, []int) {
	return file_transport_internet_quic_config_proto_rawDescGZIP(), []int{0}
}

```

这段代码定义了三个函数，分别接收一个`Config`类型的参数`x`，并返回其相应字段的值。

第一个函数`GetKey()`接收一个`Config`类型的参数`x`，并尝试获取`x`中存储的键的值。如果`x`不等于`nil`，则返回`x`中存储的键的值；如果`x`为`nil`，则返回一个空字符串。

第二个函数`GetSecurity()`与第一个函数类似，尝试获取`x`中存储的安全性配置。如果`x`不等于`nil`，则返回`x`中存储的安全性配置；如果`x`为`nil`，则返回一个`nil`类型的`SecurityConfig`。

第三个函数`GetHeader()`与第一个函数类似，尝试获取`x`中存储的头部信息。如果`x`不等于`nil`，则返回`x`中存储的头部信息；如果`x`为`nil`，则返回一个`nil`类型的`serial.TypedMessage`。


```go
func (x *Config) GetKey() string {
	if x != nil {
		return x.Key
	}
	return ""
}

func (x *Config) GetSecurity() *protocol.SecurityConfig {
	if x != nil {
		return x.Security
	}
	return nil
}

func (x *Config) GetHeader() *serial.TypedMessage {
	if x != nil {
		return x.Header
	}
	return nil
}

```

It looks like a hexadecimal representation of a Unicode character. Without further context, it\'s difficult to determine what this represents.



```go
var File_transport_internet_quic_config_proto protoreflect.FileDescriptor

var file_transport_internet_quic_config_proto_rawDesc = []byte{
	0x0a, 0x24, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65,
	0x72, 0x6e, 0x65, 0x74, 0x2f, 0x71, 0x75, 0x69, 0x63, 0x2f, 0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67,
	0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x22, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f,
	0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74,
	0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x71, 0x75, 0x69, 0x63, 0x1a, 0x21, 0x63, 0x6f, 0x6d, 0x6d,
	0x6f, 0x6e, 0x2f, 0x73, 0x65, 0x72, 0x69, 0x61, 0x6c, 0x2f, 0x74, 0x79, 0x70, 0x65, 0x64, 0x5f,
	0x6d, 0x65, 0x73, 0x73, 0x61, 0x67, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x1a, 0x1d, 0x63,
	0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2f, 0x68,
	0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x22, 0xa2, 0x01, 0x0a,
	0x06, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x10, 0x0a, 0x03, 0x6b, 0x65, 0x79, 0x18, 0x01,
	0x20, 0x01, 0x28, 0x09, 0x52, 0x03, 0x6b, 0x65, 0x79, 0x12, 0x46, 0x0a, 0x08, 0x73, 0x65, 0x63,
	0x75, 0x72, 0x69, 0x74, 0x79, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x2a, 0x2e, 0x76, 0x32,
	0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e,
	0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2e, 0x53, 0x65, 0x63, 0x75, 0x72, 0x69, 0x74,
	0x79, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x52, 0x08, 0x73, 0x65, 0x63, 0x75, 0x72, 0x69, 0x74,
	0x79, 0x12, 0x3e, 0x0a, 0x06, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x18, 0x03, 0x20, 0x01, 0x28,
	0x0b, 0x32, 0x26, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63,
	0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x73, 0x65, 0x72, 0x69, 0x61, 0x6c, 0x2e, 0x54, 0x79, 0x70,
	0x65, 0x64, 0x4d, 0x65, 0x73, 0x73, 0x61, 0x67, 0x65, 0x52, 0x06, 0x68, 0x65, 0x61, 0x64, 0x65,
	0x72, 0x42, 0x77, 0x0a, 0x26, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
	0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e,
	0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x71, 0x75, 0x69, 0x63, 0x50, 0x01, 0x5a, 0x26, 0x76,
	0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x74, 0x72,
	0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74,
	0x2f, 0x71, 0x75, 0x69, 0x63, 0xaa, 0x02, 0x22, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f,
	0x72, 0x65, 0x2e, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x49, 0x6e, 0x74,
	0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x51, 0x75, 0x69, 0x63, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74,
	0x6f, 0x33,
}

```

这段代码定义了一个名为file_transport_internet_quic_config_proto_rawDescOnce的变量，其类型为sync.Once，作用是确保同一时刻只有一个实例存在。

同时，它还定义了一个名为file_transport_internet_quic_config_proto_rawDescData的变量，其类型为v2ray.core.transport.internet.quic.ConfigProtos.Config，作用是返回一个与file_transport_internet_quic_config_proto_rawDesc类型相同的字节切片。

接着，它实现了一个名为file_transport_internet_quic_config_proto_rawDescGZIP的函数，该函数使用CompressGZIP压缩将file_transport_internet_quic_config_proto_rawDescData字节切片返回，其作用是返回一个与file_transport_internet_quic_config_proto_rawDesc类型相同的字节切片。

最后，它还定义了一个名为file_transport_internet_quic_config_proto_msgTypes的变量，其类型为protoimpl.MessageInfo，作用是用于定义与file_transport_internet_quic_config_proto_rawDesc类型相同的消息类型。同时，它还定义了一个名为file_transport_internet_quic_config_proto_goTypes的变量，其类型为[]interface{}，作用是定义go.对应的消息类型。


```go
var (
	file_transport_internet_quic_config_proto_rawDescOnce sync.Once
	file_transport_internet_quic_config_proto_rawDescData = file_transport_internet_quic_config_proto_rawDesc
)

func file_transport_internet_quic_config_proto_rawDescGZIP() []byte {
	file_transport_internet_quic_config_proto_rawDescOnce.Do(func() {
		file_transport_internet_quic_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_quic_config_proto_rawDescData)
	})
	return file_transport_internet_quic_config_proto_rawDescData
}

var file_transport_internet_quic_config_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
var file_transport_internet_quic_config_proto_goTypes = []interface{}{
	(*Config)(nil),                  // 0: v2ray.core.transport.internet.quic.Config
	(*protocol.SecurityConfig)(nil), // 1: v2ray.core.common.protocol.SecurityConfig
	(*serial.TypedMessage)(nil),     // 2: v2ray.core.common.serial.TypedMessage
}
```

This is a function that initializes the `file_transport_internet_quic_config_proto` struct. This struct is used to represent the configuration of the Internet-trained椭圆加密/解密协议的`file_transport_internet_quic_config_proto.proto` file.

If the `File_transport_internet_quic_config_proto` struct is already defined, the function returns without doing anything.

If `File_transport_internet_quic_config_proto` is not defined, the function will create it using the `protoimpl.TypeBuilder` type, and then call the `Exporter` field of the `file_transport_internet_quic_config_proto.proto` message to define the initial exported function signature, which should return an instance of the `Config` struct.

The `Config` struct represents the configuration parameters of the HTTP/3 encrypted network, including the allowed initial window size (`sizeCache`), the maximum initial window size (`initialWindow`), and the maximum incremental window size (`maxIncrementalWindow`).

The `file_transport_internet_quic_config_proto_init()` function is called only if the `File_transport_internet_quic_config_proto` struct is not defined.


```go
var file_transport_internet_quic_config_proto_depIdxs = []int32{
	1, // 0: v2ray.core.transport.internet.quic.Config.security:type_name -> v2ray.core.common.protocol.SecurityConfig
	2, // 1: v2ray.core.transport.internet.quic.Config.header:type_name -> v2ray.core.common.serial.TypedMessage
	2, // [2:2] is the sub-list for method output_type
	2, // [2:2] is the sub-list for method input_type
	2, // [2:2] is the sub-list for extension type_name
	2, // [2:2] is the sub-list for extension extendee
	0, // [0:2] is the sub-list for field type_name
}

func init() { file_transport_internet_quic_config_proto_init() }
func file_transport_internet_quic_config_proto_init() {
	if File_transport_internet_quic_config_proto != nil {
		return
	}
	if !protoimpl.UnsafeEnabled {
		file_transport_internet_quic_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*Config); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
	}
	type x struct{}
	out := protoimpl.TypeBuilder{
		File: protoimpl.DescBuilder{
			GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
			RawDescriptor: file_transport_internet_quic_config_proto_rawDesc,
			NumEnums:      0,
			NumMessages:   1,
			NumExtensions: 0,
			NumServices:   0,
		},
		GoTypes:           file_transport_internet_quic_config_proto_goTypes,
		DependencyIndexes: file_transport_internet_quic_config_proto_depIdxs,
		MessageInfos:      file_transport_internet_quic_config_proto_msgTypes,
	}.Build()
	File_transport_internet_quic_config_proto = out.File
	file_transport_internet_quic_config_proto_rawDesc = nil
	file_transport_internet_quic_config_proto_goTypes = nil
	file_transport_internet_quic_config_proto_depIdxs = nil
}

```

# `transport/internet/quic/conn.go`

这段代码是一个名为 `quic-bootstrap.go` 的文件，它是 quic-go 库中的一个初始化函数。具体来说，这段代码的作用是启动一个基于 Quic 的 TCP 代理，提供给定的端口号，并在启动时使用 `--no-hello-世界` 选项来避免发送任何的握手消息。

它主要包含以下几个步骤：

1. 导入需要的依赖项：包括 `crypto/cipher`、`crypto/rand`、`errors` 和 `time`。

2. 定义了 Quic 代理的一些基本设置，包括使用默认的证书、连接超时时间和 Keepalive 选项。

3. 加载 Quic 代理的实现，这个实现是在 `quic-transport.go` 文件中定义的。

4. 设置代理的监听端口，这个端口是用户传递给程序的选项 `--listen-port` 的值。

5. 如果没有传递给程序的选项，就默认使用 `--no-hello-世界` 选项，这个选项不发送任何握手消息，可以避免在 Quic 启动时出现一些问题。

6. 启动 Quic 代理。

这段代码的目的是提供一个默认的 Quic 代理，可以用来进行网络通信，但是它并不包含任何安全和加密相关的功能。如果需要更完整的 quic-go 库的功能，可以参考 `quic-transport.go` 和 `quic-bootstrap.go` 文件。


```go
// +build !confonly

package quic

import (
	"crypto/cipher"
	"crypto/rand"
	"errors"
	"time"

	"github.com/lucas-clemente/quic-go"
	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/net"
	"v2ray.com/core/transport/internet"
)

```

该代码定义了一个名为 `sysConn` 的结构体，其中包含一个 `net.PacketConn` 类型的 `conn` 字段，一个 `internet.PacketHeader` 类型的 `header` 字段，一个 `cipher.AEAD` 类型的 `auth` 字段。

函数 `wrapSysConn` 接收一个 `net.PacketConn` 类型的 `rawConn` 和一个 `Config` 类型的参数 `config`。该函数首先调用 `getHeader` 函数获取 `header` 字段的值，然后调用 `getAuth` 函数获取 `auth` 字段的值。最后，将获取到的值作为 `sysConn` 结构的 `conn`、`header` 和 `auth` 字段的值，返回新的 `sysConn` 结构体。

如果调用 `wrapSysConn` 时传递的 `rawConn` 和 `config` 参数不合法，函数将返回 `nil` 和相应的错误。


```go
type sysConn struct {
	conn   net.PacketConn
	header internet.PacketHeader
	auth   cipher.AEAD
}

func wrapSysConn(rawConn net.PacketConn, config *Config) (*sysConn, error) {
	header, err := getHeader(config)
	if err != nil {
		return nil, err
	}
	auth, err := getAuth(config)
	if err != nil {
		return nil, err
	}
	return &sysConn{
		conn:   rawConn,
		header: header,
		auth:   auth,
	}, nil
}

```

这段代码的作用是实现了一个名为 `sysConn` 的网络连接类，其中的 `readFromInternal` 函数用于从远程服务器接收数据并返回给客户端。

函数接收一个字节数组 `p` 作为参数，表示远程服务器发送的数据。函数首先创建一个缓冲区 `buffer`，用于存储从远程服务器读取的数据，然后使用 `getBuffer` 函数获取一个可用的缓冲区。

接着，函数使用 `defer putBuffer` 函数将缓冲区中的数据传回主缓冲区，确保在函数结束时，缓冲区中的数据不会丢失。

函数中，首先使用 `c.conn.ReadFrom` 函数从远程服务器读取一个字节数组 `buffer`。如果读取过程中出现错误，函数会返回一个错误对象 `errInvalidPacket`，并将其传递给调用方。

如果远程服务器返回的数据成功读取，函数会将读取到的数据 `payload` 存储到缓冲区中，并使用 `c.header` 变量检查远程服务器发送的数据是否包含头部信息。如果头部信息存在，函数会检查当前读取到的数据是否小于头部信息的大小，如果小于头部信息的大小，函数会直接返回 `0` 和 `addr` 两个变量，因为已经读取了所有需要的信息。

如果头部信息存在，并且当前读取到的数据长度大于等于头部信息的大小，函数会将从远程服务器读取的数据 `payload` 存储到缓冲区中，并使用 `c.header.NonceSize` 计算需要非ce的数量，然后使用 `c.auth.Open` 函数尝试打开auth协程，使用已经计算好的非ce数据和非ce数量，以及当前读取到的数据，尝试获取数据。如果auth协程成功打开，函数就可以返回已经读取到的数据长度 `len(p)` 和 `addr` 两个变量，表示数据读取成功。

如果上述过程中出现错误，函数会使用 `errInvalidPacket` 错误对象返回一个错误。


```go
var errInvalidPacket = errors.New("invalid packet")

func (c *sysConn) readFromInternal(p []byte) (int, net.Addr, error) {
	buffer := getBuffer()
	defer putBuffer(buffer)

	nBytes, addr, err := c.conn.ReadFrom(buffer)
	if err != nil {
		return 0, nil, err
	}

	payload := buffer[:nBytes]
	if c.header != nil {
		if len(payload) <= int(c.header.Size()) {
			return 0, nil, errInvalidPacket
		}
		payload = payload[c.header.Size():]
	}

	if c.auth == nil {
		n := copy(p, payload)
		return n, addr, nil
	}

	if len(payload) <= c.auth.NonceSize() {
		return 0, nil, errInvalidPacket
	}

	nonce := payload[:c.auth.NonceSize()]
	payload = payload[c.auth.NonceSize():]

	p, err = c.auth.Open(p[:0], nonce, payload, nil)
	if err != nil {
		return 0, nil, errInvalidPacket
	}

	return len(p), addr, nil
}

```

该函数的作用是读取来自网络连接的数据并返回。它接收一个字符串类型的参数 `c` 和一个字节切片类型的参数 `p`。

首先，它检查 `c` 和 `auth` 是否都为 `nil`。如果是，那么它将尝试从 `c.conn` 通道中读取数据。如果 `c` 或 `auth` 中有一个不是 `nil`，那么它将调用 `c.readFromInternal` 函数，该函数尝试从 `c` 的内部读取数据。

如果 `c.header` 或 `c.auth` 错误，函数将返回一个 `0`、一个 `net.Addr` 类型的错误对象和一个 `err` 参数。

函数还使用一个循环来读取数据。它每次调用 `c.readFromInternal` 函数并检查是否出现错误。如果错误不是由于 `c.header` 或 `c.auth` 错误，那么函数将返回读取的数据和网络地址。如果错误是 `errInvalidPacket`，那么函数将返回一个非 `nil` 的错误对象。


```go
func (c *sysConn) ReadFrom(p []byte) (int, net.Addr, error) {
	if c.header == nil && c.auth == nil {
		return c.conn.ReadFrom(p)
	}

	for {
		n, addr, err := c.readFromInternal(p)
		if err != nil && err != errInvalidPacket {
			return 0, nil, err
		}
		if err == nil {
			return n, addr, nil
		}
	}
}

```

该函数的作用是向目标网络地址发送数据帧。传送的数据帧中包含以下部分：

1. 从源连接器（sysConn）中获取数据缓冲区和目标地址。
2. 如果网络连接器和认证信息都为空，则将数据直接发送。
3. 如果网络连接器有数据可发送，则将数据连接到数据缓冲区，并获取数据缓冲区的大小。
4. 如果认证信息不为空，则使用认证信息对数据进行签名，并确保不重复签名。
5. 如果同时存在数据缓冲区和认证信息，则将数据缓冲区的内容与目标地址进行连接，并返回实际发送的数据长度。

函数的实现依赖于网络传输框架和操作系统提供的库。


```go
func (c *sysConn) WriteTo(p []byte, addr net.Addr) (int, error) {
	if c.header == nil && c.auth == nil {
		return c.conn.WriteTo(p, addr)
	}

	buffer := getBuffer()
	defer putBuffer(buffer)

	payload := buffer
	n := 0
	if c.header != nil {
		c.header.Serialize(payload)
		n = int(c.header.Size())
	}

	if c.auth == nil {
		nBytes := copy(payload[n:], p)
		n += nBytes
	} else {
		nounce := payload[n : n+c.auth.NonceSize()]
		common.Must2(rand.Read(nounce))
		n += c.auth.NonceSize()
		pp := c.auth.Seal(payload[:n], nounce, p, nil)
		n = len(pp)
	}

	return c.conn.WriteTo(payload[:n], addr)
}

```

这段代码定义了三个函数，分别作用于一个名为`sysConn`的`sys.conn.Transport`结构体的`*c`参数。

1. `func (c *sysConn) Close() error` 函数表示关闭与`sysConn`关联的TCP连接。它首先调用`c.conn.Close()`，然后返回一个错误。如果连接无法关闭，该函数将抛出。

2. `func (c *sysConn) LocalAddr() net.Addr` 函数返回与`sysConn`关联的TCP连接的本地地址。

3. `func (c *sysConn) SetDeadline(t time.Time) error` 函数设置TCP连接的 deadline。它首先调用`c.conn.SetDeadline(t)`，然后返回一个错误。如果设置的deadline超出了当前时间，该函数将抛出。

4. `func (c *sysConn) SetReadDeadline(t time.Time) error` 函数设置TCP连接的 read deadline。它首先调用`c.conn.SetReadDeadline(t)`，然后返回一个错误。如果设置的deadline超出了当前时间，该函数将抛出。


```go
func (c *sysConn) Close() error {
	return c.conn.Close()
}

func (c *sysConn) LocalAddr() net.Addr {
	return c.conn.LocalAddr()
}

func (c *sysConn) SetDeadline(t time.Time) error {
	return c.conn.SetDeadline(t)
}

func (c *sysConn) SetReadDeadline(t time.Time) error {
	return c.conn.SetReadDeadline(t)
}

```

这段代码定义了一个名为`func`的函数接收一个`sysConn`类型的参数，并设置一个`time.Time`类型的参数`t`为写入数据的超时时间。函数的作用是返回一个`error`类型的变量。

函数内部，首先通过调用`conn.SetWriteDeadline`函数，将写入超时时间设置为`t`。然后，返回`conn.SetWriteDeadline`的返回值作为整数类型。

函数内部，还定义了一个名为`interConn`的`struct`类型，该类型包含一个`quic.Stream`类型的`c`字段，一个`net.Addr`类型的`local`字段和一个`net.Addr`类型的`remote`字段。

函数内部，定义了一个名为`Read`的函数和一个名为`WriteMultiBuffer`的函数。

`Read`函数接收一个`[]byte`类型的参数`b`，并返回一个`int`类型的参数，表示成功读取的数据量。函数内部，通过调用`stream.Read`函数，将数据读取到`b`中。

`WriteMultiBuffer`函数接收一个`buf.MultiBuffer`类型的参数`mb`，并返回一个`error`类型的变量`err`。函数内部，将`mb`通过`buf.Compact`函数压入到一个`buf.MultiBuffer`中，然后调用`buf.WriteMultiBuffer`函数，将数据写入到`mb`中。接着，通过调用`buf.ReleaseMulti`函数，释放出对`mb`的写入操作。最后，将`err`赋值为`err`的返回值，以便在函数失败时进行错误处理。


```go
func (c *sysConn) SetWriteDeadline(t time.Time) error {
	return c.conn.SetWriteDeadline(t)
}

type interConn struct {
	stream quic.Stream
	local  net.Addr
	remote net.Addr
}

func (c *interConn) Read(b []byte) (int, error) {
	return c.stream.Read(b)
}

func (c *interConn) WriteMultiBuffer(mb buf.MultiBuffer) error {
	mb = buf.Compact(mb)
	mb, err := buf.WriteMultiBuffer(c, mb)
	buf.ReleaseMulti(mb)
	return err
}

```

以上代码定义了两个函数，一个是`func (c *interConn) Write(b []byte) (int, error)`，另一个是`func (c *interConn) Close() error`。

这两个函数都在一个名为`interConn`的传输层信号线路上进行操作。

函数1 `func (c *interConn) Write(b []byte) (int, error)`接收一个字节数组`b`，并返回信号线路上写入字节数组`b`的返回字节数和错误。

函数2 `func (c *interConn) Close() error`关闭连接并返回错误。

函数1和函数2都使用了`c.stream`引用，通过`c.stream.Write`方法向信号线路上写入数据。

函数2中的`c.local`和`c.remote`变量用于获取客户端的本地地址和远程地址，用于在客户端和服务器之间传输数据。


```go
func (c *interConn) Write(b []byte) (int, error) {
	return c.stream.Write(b)
}

func (c *interConn) Close() error {
	return c.stream.Close()
}

func (c *interConn) LocalAddr() net.Addr {
	return c.local
}

func (c *interConn) RemoteAddr() net.Addr {
	return c.remote
}

```

以上代码定义了两个函数，分别针对interConn类型的网络客户端连接设置 deadlines。

具体来说，SetDeadline函数接受一个时间参数t，表示设置数据的最终 deadline 到来的时间，然后将该设置并返回给调用者。如果设置的时间早于当前时间，不会产生影响，仍然返回成功。如果设置的时间晚于当前时间，将会设置为一个新的 deadline，并在新的 deadline 下尝试发送数据。如果设置成功，则返回错误。

SetReadDeadline函数与SetDeadline函数类似，只是设置的是读取数据的deadline，即设置为从当前时间开始，到设置的下一个时间点的时间。

SetWriteDeadline函数与SetDeadline函数类似，只是设置的是写入数据的deadline，即设置为从当前时间开始，到设置的下一个时间点的时间。


```go
func (c *interConn) SetDeadline(t time.Time) error {
	return c.stream.SetDeadline(t)
}

func (c *interConn) SetReadDeadline(t time.Time) error {
	return c.stream.SetReadDeadline(t)
}

func (c *interConn) SetWriteDeadline(t time.Time) error {
	return c.stream.SetWriteDeadline(t)
}

```