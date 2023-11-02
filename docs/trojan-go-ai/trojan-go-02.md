# trojan-go源码解析 2

# `common/common.go`

这段代码定义了一个名为"common"的包，然后导入了几个用于处理数据哈希的函数和一个用于格式化输出信息的函数。

具体来说，这段代码以下几个主要部分：

1. 定义了一个名为"Runnable"的接口，该接口定义了一个函数"Run"和一个函数"Close"。这些函数都在同一"Runnable"接口中，但具体的实现可能会因哈希算法等因素而有所不同。

2. 导入了一些与哈希算法相关的库，包括"crypto/sha256"和"github.com/p4gefau1t/trojan-go/util/concurrent"。

3. 定义了一个名为"os"的包，它似乎包含一些通用的操作系统功能。

4. 定义了一个名为"filepath"的包，它从filepath.pkgs中导入了一些通用的文件操作函数。

5. 在"common"包中定义了一个名为"Runnable"的接口，该接口定义了一个包含两个函数的组合。这两个函数分别是"Run"和"Close"。

6. 在"Runnable"接口中定义了一个名为"Run"的函数，它接受一个错误作为参数，并在运行时将其返回。

7. 在"Runnable"接口中定义了一个名为"Close"的函数，它接受一个错误作为参数，并在关闭时将其返回。

8. 在"common"包的其他部分，可能还定义了一些其他函数或变量，但上述部分是这段代码中最重要的部分。


```go
package common

import (
	"crypto/sha256"
	"fmt"
	"os"
	"path/filepath"

	"github.com/p4gefau1t/trojan-go/log"
)

type Runnable interface {
	Run() error
	Close() error
}

```

这两段代码都是使用Go语言编写的，目的是什么呢？让我们一步一步来分析。

首先看第一段代码，它定义了一个名为`func SHA224String(password string) string`的函数。这个函数的作用是获取一个经过SHA-224哈希算法加密后的密码字符串。函数接收一个`string`类型的参数`password`，然后创建一个名为`hash`的`sha256.New224()`实例。接着，使用`hash.Write([]byte(password))`将密码字节串编码为哈希值，然后使用`hash.Sum(nil)`计算哈希值，最后将哈希值转换为一个字符串并返回。

接下来看第二段代码，它定义了一个名为`func GetProgramDir() string`的函数。这个函数的作用是获取程序目录，如果目录不存在，则输出错误并退出程序。函数首先使用`filepath.Abs(filepath.Dir(os.Args[0]))`获取程序目录，如果目录不存在，则输出错误并退出程序。

总的来说，这两段代码的主要目的是实现一个简单的加密函数和一个获取程序目录的函数。加密函数主要用于在测试中保证密码的安全性，而获取程序目录的函数则用于在程序运行时获取用户输入的目录并执行相应的操作。


```go
func SHA224String(password string) string {
	hash := sha256.New224()
	hash.Write([]byte(password))
	val := hash.Sum(nil)
	str := ""
	for _, v := range val {
		str += fmt.Sprintf("%02x", v)
	}
	return str
}

func GetProgramDir() string {
	dir, err := filepath.Abs(filepath.Dir(os.Args[0]))
	if err != nil {
		log.Fatal(err)
	}
	return dir
}

```

这段代码定义了一个名为 GetAssetLocation 的函数，用于获取资产文件（asset file）的位置（location）。函数的输入参数为 fileString 类型，表示要获取位置的文件名。

函数首先检查 file 是否绝对路径（即不包含从 "." 或 ".." 开始的位置）。如果是，则返回 file 本身。如果不是，函数接下来尝试使用 os 系统的 Getenv 函数获取一个名为 "TROJAN_GO_LOCATION_ASSET" 的环境变量，如果这个环境变量存在，则使用 filepath.Abs 函数将路径转换为绝对路径，然后返回 filepath.Join 函数将文件路径与文件名连接起来。如果这个环境变量不存在，则函数使用 filepath.Join 函数将文件名连接到程序目录（GetProgramDir）的相对路径，并返回该路径。

如果函数在获取资产文件位置时遇到错误，例如无法访问环境变量或文件路径不存在，函数将记录错误并返回文件名本身。


```go
func GetAssetLocation(file string) string {
	if filepath.IsAbs(file) {
		return file
	}
	if loc := os.Getenv("TROJAN_GO_LOCATION_ASSET"); loc != "" {
		absPath, err := filepath.Abs(loc)
		if err != nil {
			log.Fatal(err)
		}
		log.Debugf("env set: TROJAN_GO_LOCATION_ASSET=%s", absPath)
		return filepath.Join(absPath, file)
	}
	return filepath.Join(GetProgramDir(), file)
}

```

# `common/error.go`

这段代码定义了一个名为`Error`的结构体，该结构体包含一个字符串类型的`info`字段和一个指向`error`类型类型的`e`变量。

该代码还定义了一个名为`Base`的函数，该函数接收一个`error`类型的参数，并创建一个指向`Error`类型类型的`e`变量。如果传递的`error`不空，则会将其添加到`info`中，并将结果返回。

该代码最后没有输出任何东西，但是它定义了一个可以用于输出错误信息的`Error`类型，该类型可以使用`fmt.Printf`函数进行输出。


```go
package common

import (
	"fmt"
)

type Error struct {
	info string
}

func (e *Error) Error() string {
	return e.info
}

func (e *Error) Base(err error) *Error {
	if err != nil {
		e.info += " | " + err.Error()
	}
	return e
}

```

这三段代码都是使用Go语言中的语言特定的错误处理函数。

第一段代码定义了一个名为NewError的函数，该函数创建了一个名为info的错误对象，并返回一个指向Error类型的引用。函数的作用是在创建新的错误对象时记录下所传递的信息。

第二段代码定义了一个名为Must的函数，该函数接收一个err参数，如果err不等于零，则打印出err并崩溃。Must的作用是验证传入的err是否为零，如果不是，则打印错误并崩溃。

第三段代码定义了一个名为Must2的函数，该函数接收两个参数，一个是接口类型，一个是err参数。如果err不等于零，则打印出err并崩溃。Must2的作用是验证接口类型的err是否为零，如果不是，则打印错误并崩溃。


```go
func NewError(info string) *Error {
	return &Error{
		info: info,
	}
}

func Must(err error) {
	if err != nil {
		fmt.Println(err)
		panic(err)
	}
}

func Must2(_ interface{}, err error) {
	if err != nil {
		fmt.Println(err)
		panic(err)
	}
}

```

# `common/io.go`

该代码定义了一个名为 "RewindReader" 的类，用于从网络中读取数据，并具有以下特点：

1. 通过 "net" 包从网络中读取数据，使用 "io.Reader" 类型将数据读取到一个缓冲区 "buf" 中。
2. 使用 "sync.Mutex" 类型的 "mu" 变量，用于在缓冲区中的数据读取操作的互斥同步。
3. 在 "mu.sync" 命名上下文中使用 "RewindReader" 类型的实例，确保任何情况下只有一个 "RewindReader" 实例在运行。
4. "bufReadIdx" 变量用于跟踪缓冲区中正在读取的数据位置，如果是从头开始读取，则它的值为 0。
5. "rewound" 变量指示是否已重置读取计数的器，如果是，则它设置为 true，否则设置为 false。
6. "buffering" 变量指示是否正在使用缓冲区，如果是，则设置为 true，否则设置为 false。
7. "bufferSize" 变量定义了缓冲区的大小，如果缓冲区大小需要在应用程序中进行调整，则该变量将用于设置。
8. "log" 包中的 "log.Fatal" 函数用于在出现错误时输出致命错误，并返回错误信息。


```go
package common

import (
	"io"
	"net"
	"sync"

	"github.com/p4gefau1t/trojan-go/log"
)

type RewindReader struct {
	mu         sync.Mutex
	rawReader  io.Reader
	buf        []byte
	bufReadIdx int
	rewound    bool
	buffering  bool
	bufferSize int
}

```

该函数的作用是读取一个RewindReader类型的数据源，并将其缓冲区中的数据写入一个目标缓冲区。以下是函数的步骤：

1. 对读取操作进行加锁操作，以确保在函数返回前对读取操作进行完成。
2. 如果RewindReader中的缓冲区长度大于正在读取的数据长度，则函数将会从RewindReader的缓冲区中读取比正在读取的数据长度更多的数据，并将它们写入目标缓冲区。
3. 如果正在使用缓冲区，则函数将会检查缓冲区是否已满，如果是，则函数将会记录下读取的数据长度并返回，以便在需要时进行提示。
4. 如果使用了RewindReader，则函数将从原始数据源中连续读取数据，而不是逐个读取。
5. 在函数返回之前，必须确保对读取操作进行完成。


```go
func (r *RewindReader) Read(p []byte) (int, error) {
	r.mu.Lock()
	defer r.mu.Unlock()

	if r.rewound {
		if len(r.buf) > r.bufReadIdx {
			n := copy(p, r.buf[r.bufReadIdx:])
			r.bufReadIdx += n
			return n, nil
		}
		r.rewound = false // all buffering content has been read
	}
	n, err := r.rawReader.Read(p)
	if r.buffering {
		r.buf = append(r.buf, p[:n]...)
		if len(r.buf) > r.bufferSize*2 {
			log.Debug("read too many bytes!")
		}
	}
	return n, err
}

```

这两函数是`RewindReader`类的两个方法，用于读取字节数组和 discard读取中的数据。

函数`ReadByte()`接收一个`RewindReader`对象`r`，返回一个字节数组和一个错误。函数会从`r`的当前读取位置开始读取字节数组中的第一个字节，并返回该字节和相应的错误。如果当前读取位置没有可读取的字节，函数将返回一个空的字节数组。

函数`Discard()`同样接收一个`RewindReader`对象`r`，返回一个已消费的字节数和一个错误。函数会尝试读取`n`个字节，如果消费的字节数少于`n`，函数将返回未消费的字节数。在消费剩余的字节时，函数会从读取中尝试消费剩余的字节，直到消费的字节数与剩余的字节数相等或者剩余的字节数为0。如果函数在消费剩余的字节时遇到错误，则会返回该错误。如果剩余的字节数为0，函数不会返回任何错误，因为在那样的情况下，函数已经没有剩余的字节可以读取。

这两个函数都是使用`Read()`方法来读取数据，并使用`Discard()`方法来控制消费的字节数。 `ReadByte()`方法在消费字节完成后，将调用`Read()`方法读取剩余的字节，而`Discard()`方法则没有这样的行为。


```go
func (r *RewindReader) ReadByte() (byte, error) {
	buf := [1]byte{}
	_, err := r.Read(buf[:])
	return buf[0], err
}

func (r *RewindReader) Discard(n int) (int, error) {
	buf := [128]byte{}
	if n < 128 {
		return r.Read(buf[:n])
	}
	for discarded := 0; discarded+128 < n; discarded += 128 {
		_, err := r.Read(buf[:])
		if err != nil {
			return discarded, err
		}
	}
	if rest := n % 128; rest != 0 {
		return r.Read(buf[:rest])
	}
	return n, nil
}

```

这两函数的作用是管理一个缓冲区为新生成的数据缓冲区，实现数据缓冲区功能。

函数1:`func (r *RewindReader) Rewind()`
此函数用于设置读取器的位置，并开启重新开始读取数据的操作。

具体操作流程如下：

1. 使用`r.mu.Lock()`获取读取器当前缓冲区的锁，确保在函数执行期间只有一个线程访问该缓冲区。

2. 如果当前缓冲区为空，执行以下操作：panic("no buffer") 错误，因为此时函数中没有对当前缓冲区进行写入或读取。

3. 设置`r.rewound`为`true`，指示是否已经准备好开始重新开始读取数据。

4. 将`r.bufReadIdx`设置为当前缓冲区的开始索引，确保在重新开始读取数据时从正确的位置开始。

5. 使用`r.mu.Unlock()`释放当前缓冲区的锁。

函数2:`func (r *RewindReader) StopBuffering()`
此函数用于停止读取数据，并释放当前缓冲区。

具体操作流程如下：

1. 使用`r.mu.Lock()`获取读取器当前缓冲区的锁，确保在函数执行期间只有一个线程访问该缓冲区。

2. 将`r.buffering`设置为`false`，指示当前缓冲区是否正在使用。

3. 使用`r.mu.Unlock()`释放当前缓冲区的锁。


```go
func (r *RewindReader) Rewind() {
	r.mu.Lock()
	if r.bufferSize == 0 {
		panic("no buffer")
	}
	r.rewound = true
	r.bufReadIdx = 0
	r.mu.Unlock()
}

func (r *RewindReader) StopBuffering() {
	r.mu.Lock()
	r.buffering = false
	r.mu.Unlock()
}

```

此函数的作用是控制循环读取器中的缓冲区大小。它接受一个整数参数 `size`，表示要设置的缓冲区大小。函数内部首先尝试禁用缓冲区，如果已经禁用但传入的缓冲区大小为零，则会输出一条错误消息。如果传入的缓冲区大小不为零，则会创建一个新的缓冲区，并将其设置为输入缓冲区的大小。最后，函数会释放内存并更新 `bufferSize` 和 `buf` 变量，以便在循环读取器中正确地使用缓冲区。


```go
func (r *RewindReader) SetBufferSize(size int) {
	r.mu.Lock()
	if size == 0 { // disable buffering
		if !r.buffering {
			panic("reader is disabled")
		}
		r.buffering = false
		r.buf = nil
		r.bufReadIdx = 0
		r.bufferSize = 0
	} else {
		if r.buffering {
			panic("reader is buffering")
		}
		r.buffering = true
		r.bufReadIdx = 0
		r.bufferSize = size
		r.buf = make([]byte, 0, size)
	}
	r.mu.Unlock()
}

```

以上代码定义了一个名为RewindConn的结构体，它包含一个net.Conn类型的网络连接和一个RewindReader类型的恢复套接字。

RewindConn的作用是为了解决有时候网络连接不可用或者需要重置连接的情况下，通过将当前连接的流量读取到控制台或者打印机等设备上，以便进行调试或记录。

具体来说，当RewindConn的网络连接不可用时，它可以通过调用Read函数，将当前连接的流量读取到RewindReader中，然后将RewindReader的流量输出到控制台或者打印机等设备上。当RewindConn的网络连接恢复时，它可以通过调用Read函数，从RewindReader中读取之前记录的流量。

RewindConn还提供了一个名为NewRewindConn的函数，该函数接受一个net.Conn类型的网络连接参数，并返回一个指向RewindConn的引用。通过调用NewRewindConn函数，可以方便地创建一个新的RewindConn实例，从而实现连接到网络设备的功能。


```go
type RewindConn struct {
	net.Conn
	*RewindReader
}

func (c *RewindConn) Read(p []byte) (int, error) {
	return c.RewindReader.Read(p)
}

func NewRewindConn(conn net.Conn) *RewindConn {
	return &RewindConn{
		Conn: conn,
		RewindReader: &RewindReader{
			rawReader: conn,
		},
	}
}

```

该 StickyWriter 结构体是一个用于写入文件的接口类型。它包括一个 rawWriter 字段，一个 writeBuffer 字段和一个 MaxBuffered 字段。

函数 Write 对传入的 slice 参数 p 进行写入，并返回写入的个数或者是错误的错误。如果缓冲区还有空闲空间，则将 p 写入缓冲区，并将 buffer 长度减一。如果缓冲区满，则将 buffer 中的内容写入 rawWriter，并清空 buffer。如果缓冲区仍然为空，则将 maxBuffered 字段减一，并尝试将 p 写入 rawWriter。如果写入失败，则返回错误。最后，清空缓冲区和 maxBuffered 字段，以便准备写入新的数据。


```go
type StickyWriter struct {
	rawWriter   io.Writer
	writeBuffer []byte
	MaxBuffered int
}

func (w *StickyWriter) Write(p []byte) (int, error) {
	if w.MaxBuffered > 0 {
		w.MaxBuffered--
		w.writeBuffer = append(w.writeBuffer, p...)
		if w.MaxBuffered != 0 {
			return len(p), nil
		}
		w.MaxBuffered = 0
		_, err := w.rawWriter.Write(w.writeBuffer)
		w.writeBuffer = nil
		return len(p), err
	}
	return w.rawWriter.Write(p)
}

```

# `common/io_test.go`

该代码的作用是测试一个名为 "BufferedReader" 的功能。该功能可以用于从缓冲区读取数据，并保证读取的数据是准确的。

具体来说，该代码创建了一个名为 "common" 的包，其中包含了一个名为 "BufferedReader" 的函数。该函数接受一个 "bytes" 类型的参数 "payload"，表示要读取的数据。函数使用 "crypto/rand" 包中的 "rand" 函数生成一个随机读取数据的长度为 1024 的字节切片，并将其赋值给 "payload" 变量。然后，函数创建一个名为 "rawReader" 的字节缓冲区 "rawReader"，其中包含一个长度为 1024 的字节切片和一个长度为 512 的字节切片，分别用于读取数据和缓冲数据。接着，函数创建一个名为 "rewardedReader" 的 "RewindReader" 类型的对象，其中包含一个 "rawReader" 类型的 "rawReader" 和一个缓冲区大小为 2048 的 "buf"。最后，函数使用 "BufferedReader" 函数读取 "rawReader" 中的数据，并将其存储在名为 "buf1" 和 "buf2" 的缓冲区中。然后，函数比较两个缓冲区中的数据是否相等，如果不相等，则测试失败。最后，函数再读取一次数据，并将其存储在名为 "buf3" 的缓冲区中，并检查是否与之前的读取数据相等。如果相等，则测试成功。


```go
package common

import (
	"bytes"
	"crypto/rand"
	"testing"

	"github.com/v2fly/v2ray-core/v4/common"
)

func TestBufferedReader(t *testing.T) {
	payload := [1024]byte{}
	rand.Reader.Read(payload[:])
	rawReader := bytes.NewBuffer(payload[:])
	r := RewindReader{
		rawReader: rawReader,
	}
	r.SetBufferSize(2048)
	buf1 := make([]byte, 512)
	buf2 := make([]byte, 512)
	common.Must2(r.Read(buf1))
	r.Rewind()
	common.Must2(r.Read(buf2))
	if !bytes.Equal(buf1, buf2) {
		t.Fail()
	}
	buf3 := make([]byte, 512)
	common.Must2(r.Read(buf3))
	if !bytes.Equal(buf3, payload[512:]) {
		t.Fail()
	}
	r.Rewind()
	buf4 := make([]byte, 1024)
	common.Must2(r.Read(buf4))
	if !bytes.Equal(payload[:], buf4) {
		t.Fail()
	}
}

```

# `common/net.go`

该代码包的作用是执行以下操作：

1. 导入一些标准库中的函数和变量，包括 `fmt` 用于格式化输出，`io` 用于输入/输出流操作，`net` 用于网络通信，`strconv` 用于字符串转换，`os` 用于操作系统相关操作，`time` 用于时间相关操作，`strings` 用于字符串操作等。

2. 定义了一些变量，包括 `url.Values` 类型的 `project` 变量，用于在 HTTP 请求中传递参数，`http.Response` 类型的 `response` 变量，用于从 HTTP 服务器获取响应数据，以及一些自定义的变量 `common.randomNumber` 和 `common.encoding.utf8` 等。

3. 实现了 HTTP 请求的发送，首先通过 `net/http` 包发送 HTTP 请求，获取指定 URL 并执行 GET 请求，然后解析响应内容，最后输出响应内容。具体实现包括：

  - 创建一个 HTTP 请求对象 `http.Request` 并设置请求方法、URL 和请求头等参数；
  - 设置请求参数，使用 `url.Values` 中的 `project` 变量将参数传递给服务器，其中 `project` 的值是 `"param1=value1&param2=value2"` 形式，`param1` 和 `param2` 是参数名，`value1` 和 `value2` 是参数值；
  - 发送请求并获取响应，使用 `net/http` 包发送 HTTP 请求，获取响应对象 `http.Response` 并解析响应内容；
  - 输出响应内容，使用 `fmt` 包中的 `Printf` 函数将响应内容输出到控制台。

4. 通过 `os` 包执行一些操作系统相关操作，包括：

  - 获取当前时间，并将其格式化为 `time.Time` 类型；
  - 生成一个随机的字符串，并将其编码为 `utf8` 编码的字符串；
  - 将当前时间和随机生成的字符串组合成 URL 参数，并将它们作为参数传递给 `http.Request` 对象，最终发送 HTTP 请求。


```go
package common

import (
	"fmt"
	"io"
	"io/ioutil"
	"net"
	"net/http"
	"net/url"
	"os"
	"strconv"
	"strings"
	"time"
)

```

这段代码定义了一个名为HumanFriendlyTraffic的函数，该函数接受一个字节数组uint64类型的参数bytes，并返回一个描述流量单位（例如KiB、MiB或GiB）和该流量单位的数量。

KiB是1024，MiB是KiB的1024倍，GiB是MiB的1024倍。所以，如果bytes的值小于等于KiB，函数将返回该bytes字节数；如果bytes的值小于等于MiB，函数将返回流量单位为KiB的数量；如果bytes的值小于等于GiB，函数将返回流量单位为MiB的数量。

函数的实现采用了人类可读的格式，使用了两个浮点数（float32）来计算流量单位。这种格式将数字转换为带有两个小数点的十进制数字，方便人们理解和比较不同大小的流量单位。


```go
const (
	KiB = 1024
	MiB = KiB * 1024
	GiB = MiB * 1024
)

func HumanFriendlyTraffic(bytes uint64) string {
	if bytes <= KiB {
		return fmt.Sprintf("%d B", bytes)
	}
	if bytes <= MiB {
		return fmt.Sprintf("%.2f KiB", float32(bytes)/KiB)
	}
	if bytes <= GiB {
		return fmt.Sprintf("%.2f MiB", float32(bytes)/MiB)
	}
	return fmt.Sprintf("%.2f GiB", float32(bytes)/GiB)
}

```

该函数的作用是尝试连接到指定的网络主机，并返回一个随机的TCP或UDP端口号，以便连接到远程计算机。它采用递归方式，在每次尝试连接时增加尝试次数，直到成功连接或所有尝试次数都为止。如果连接建立成功，函数将返回所选端口号的数字形式，否则返回0。


```go
func PickPort(network string, host string) int {
	switch network {
	case "tcp":
		for retry := 0; retry < 16; retry++ {
			l, err := net.Listen("tcp", host+":0")
			if err != nil {
				continue
			}
			defer l.Close()
			_, port, err := net.SplitHostPort(l.Addr().String())
			Must(err)
			p, err := strconv.ParseInt(port, 10, 32)
			Must(err)
			return int(p)
		}
	case "udp":
		for retry := 0; retry < 16; retry++ {
			conn, err := net.ListenPacket("udp", host+":0")
			if err != nil {
				continue
			}
			defer conn.Close()
			_, port, err := net.SplitHostPort(conn.LocalAddr().String())
			Must(err)
			p, err := strconv.ParseInt(port, 10, 32)
			Must(err)
			return int(p)
		}
	default:
		return 0
	}
	return 0
}

```

这两段代码都是函数，名为`WriteAllBytes`和`WriteFile`，作用是向指定文件或字符串写入字节数组`payload`。

`WriteAllBytes`函数接受一个`Writer`和一个字节数组`payload`作为参数。首先，该函数遍历`payload`数组，将`payload`中所有元素写入到接受写入的`Writer`中。然后，它将`payload`中所有元素刷写到接收写入的`Writer`中之后的所有元素复制到一个新的`payload`数组中。最后，函数返回一个`n`表示`payload`数组中元素个数，`err`表示发生错误的时间。

`WriteFile`函数接受一个文件路径和一个字节数组`payload`作为参数。首先，该函数创建一个新文件并写入`payload`字节数组。然后，它调用`WriteAllBytes`函数将`payload`中所有元素写入到接受写入的`Writer`中。最后，函数返回一个`n`表示`payload`数组中元素个数，`err`表示发生错误的时间。


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

func WriteFile(path string, payload []byte) error {
	writer, err := os.Create(path)
	if err != nil {
		return err
	}
	defer writer.Close()

	return WriteAllBytes(writer, payload)
}

```

此函数的作用是获取目标字符串（通过调用url.Parse函数解析）对应的HTTP或HTTPS链接，并返回该链接的内容。函数首先检查给定的目标字符串是否符合预期的URL schema，如果不符合，则会返回错误信息。然后，使用一个`http.Client`实例发送HTTP GET请求，获取响应，并检查响应状态码是否为HTTP的200状态码（表示成功）。如果响应状态码不是200，则会返回错误信息。最后，使用`ioutil.ReadAll`函数从响应主体中读取所有内容，并返回该内容。


```go
func FetchHTTPContent(target string) ([]byte, error) {
	parsedTarget, err := url.Parse(target)
	if err != nil {
		return nil, fmt.Errorf("invalid URL: %s", target)
	}

	if s := strings.ToLower(parsedTarget.Scheme); s != "http" && s != "https" {
		return nil, fmt.Errorf("invalid scheme: %s", parsedTarget.Scheme)
	}

	client := &http.Client{
		Timeout: 30 * time.Second,
	}
	resp, err := client.Do(&http.Request{
		Method: "GET",
		URL:    parsedTarget,
		Close:  true,
	})
	if err != nil {
		return nil, fmt.Errorf("failed to dial to %s", target)
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		return nil, fmt.Errorf("unexpected HTTP status code: %d", resp.StatusCode)
	}

	content, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		return nil, fmt.Errorf("failed to read HTTP response")
	}

	return content, nil
}

```

# `common/sync.go`

这段代码定义了一个名为 "common" 的包，其中包含一个名为 "Notifier" 的结构体及其相关方法。

"Notifier" 结构体代表一个用于通知变化的消息代理。它包含一个名为 "c" 的通道，用于代表变化的内容。每当 producers 变化时，它会向 channel "c" 发送一个新数据元素，通知消费者。消费者可以订阅 "Notifier" 结构体，以异步方式接收通知。

"NewNotifier" 函数用于创建一个 "Notifier" 实例。它返回一个指向 "Notifier" 实例的指针，该实例初始化时包含一个包含一个空通道的 "channel"。

"Signal" 函数用于通知生产者有变化发生，并不会阻塞任何操作。它将变化内容作为参数传递给 "c" 通道的发送方。


```go
package common

// Notifier is a utility for notifying changes. The change producer may notify changes multiple time, and the consumer may get notified asynchronously.
type Notifier struct {
	c chan struct{}
}

// NewNotifier creates a new Notifier.
func NewNotifier() *Notifier {
	return &Notifier{
		c: make(chan struct{}, 1),
	}
}

// Signal signals a change, usually by producer. This method never blocks.
```

这两函数定义在了一个名为 `Notifier` 的接口上。它们的作用是为 `Signal` 和 `Wait` 函数提供实现了 `Signal` 和 `Wait` 接口的实现。

1. `Signal` 函数的作用是接收一个整数类型的参数 `n`，并输出一个 `struct{}` 类型的值。它会根据传入的 `n` 值，如果是 `<-` 传递了一个 `struct{}` 类型的值，那么直接返回。否则，会执行一个 `select` 语句，根据 `n` 的值选择执行不同的代码块。

2. `Wait` 函数的作用是返回一个 channel，用于等待 `Notifier` 中的 `c` 发生变化。它会返回一个 `<-` 通道，当 `Notifier` 中的 `c` 发生变化时，会将值 `<-` 发送到通道中。

综合来看，这两函数的主要作用是为 `Signal` 和 `Wait` 函数提供了一个简单的、可读性高的接口。


```go
func (n *Notifier) Signal() {
	select {
	case n.c <- struct{}{}:
	default:
	}
}

// Wait returns a channel for waiting for changes. The returned channel never gets closed.
func (n *Notifier) Wait() <-chan struct{} {
	return n.c
}

```

# `common/geodata/cache.go`

这段代码定义了一个名为"geodata"的包，它导入了两个相关的开源项目："github.com/v2fly/v2ray-core"和"github.com/p4gefau1t/trojan-go"。

首先，代码导入了"io/ioutil"和"strings"两个标准库，可能用于文件I/O操作和字符串处理。

接着，代码引入了"github.com/v2fly/v2ray-core/v4/app/router"以及"google.golang.org/protobuf/proto"，这些库可能用于构建应用程序的内部逻辑和外部API。

接下来，代码定义了一个名为"geoipCache"的类型，它接受一个键值对类型"map[string]*v2router.GeoIP"，该类型键为"string"，值为一个"v2router.GeoIP"类型的指针。

通过创建一个名为"geoipCache"的Map，可以存储一组"v2router.GeoIP"类型的数据，这些数据将用于缓存从外部API获取到的"GeoIP"数据，以提高应用程序的性能。


```go
package geodata

import (
	"io/ioutil"
	"strings"

	v2router "github.com/v2fly/v2ray-core/v4/app/router"
	"google.golang.org/protobuf/proto"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/log"
)

type geoipCache map[string]*v2router.GeoIP

```

这段代码是一个 Go 语言编写的函数，它实现了基于键的地理信息缓存功能。

具体来说，这段代码定义了三个函数，分别用于判断键是否存在、获取指定键的值并返回、以及设置指定键的值。这三个函数的作用如下：

1. `func (g geoipCache) Has(key string) bool`：判断给定的键是否存在于缓存中，如果存在返回 `true`，否则返回 `false`。

2. `func (g geoipCache) Get(key string) *v2router.GeoIP`：从缓存中获取指定键的值并返回，如果缓存为空或给定的键不存在，返回 `nil`。

3. `func (g geoipCache) Set(key string, value *v2router.GeoIP)`：将给定的键的值设置为缓存中的指定键的值，如果缓存为空或给定的键不存在，将缓存设置为一个新的空缓存。


```go
func (g geoipCache) Has(key string) bool {
	return !(g.Get(key) == nil)
}

func (g geoipCache) Get(key string) *v2router.GeoIP {
	if g == nil {
		return nil
	}
	return g[key]
}

func (g geoipCache) Set(key string, value *v2router.GeoIP) {
	if g == nil {
		g = make(map[string]*v2router.GeoIP)
	}
	g[key] = value
}

```

This is a Go function that retrieves a GeoIP location from a file using its asset and code sections. It first checks if it has already retrieved the location from the database, and if it has, it returns the associated geoip and nil. If it has not retrieved the location, it attempts to read the file's contents and decode the geoip data from it.

If the function fails to read the file's contents for any reason (e.g. an error with the file's readability), it will return nil. If it succeeds in reading the file's contents, it will decode the geoip data and store it in the database. It will also check if the decoded geoip matches the one expected by the database (based on the country code). If the match is found, it will return the associated geoip and nil. If the match is not found, it will return an error.


```go
func (g geoipCache) Unmarshal(filename, code string) (*v2router.GeoIP, error) {
	asset := common.GetAssetLocation(filename)
	idx := strings.ToLower(asset + ":" + code)
	if g.Has(idx) {
		log.Debugf("geoip cache HIT: %s -> %s", code, idx)
		return g.Get(idx), nil
	}

	geoipBytes, err := Decode(asset, code)
	switch err {
	case nil:
		var geoip v2router.GeoIP
		if err := proto.Unmarshal(geoipBytes, &geoip); err != nil {
			return nil, err
		}
		g.Set(idx, &geoip)
		return &geoip, nil

	case ErrCodeNotFound:
		return nil, common.NewError("country code " + code + " not found in " + filename)

	case ErrFailedToReadBytes, ErrFailedToReadExpectedLenBytes,
		ErrInvalidGeodataFile, ErrInvalidGeodataVarintLength:
		log.Warnf("failed to decode geoip file: %s, fallback to the original ReadFile method", filename)
		geoipBytes, err = ioutil.ReadFile(asset)
		if err != nil {
			return nil, err
		}
		var geoipList v2router.GeoIPList
		if err := proto.Unmarshal(geoipBytes, &geoipList); err != nil {
			return nil, err
		}
		for _, geoip := range geoipList.GetEntry() {
			if strings.EqualFold(code, geoip.GetCountryCode()) {
				g.Set(idx, geoip)
				return geoip, nil
			}
		}

	default:
		return nil, err
	}

	return nil, common.NewError("country code " + code + " not found in " + filename)
}

```

这段代码定义了一个名为 `geoiseCache` 的类型，该类型使用一个键值对 `map` 类型来存储 `v2router.GeoSite` 类型的值。

该代码还定义了三个函数：`Has`、`Get` 和 `Set`。

`Has` 函数用于检查给定的键是否存在于 `geoiseCache` 中，如果不存在，则返回 `false`，否则返回 `true`。

`Get` 函数用于从 `geoiseCache` 中获取指定键的值，并返回该值的引用（如果该键存在）。如果 `geoiseCache` 为 `nil`，则返回 `nil`。

`Set` 函数用于将指定键的值设置为给定值的 `geoiseCache`，如果 `geoiseCache` 为 `nil`，则创建一个新的 `map` 并将其设置为给定值的 `map`。


```go
type geositeCache map[string]*v2router.GeoSite

func (g geositeCache) Has(key string) bool {
	return !(g.Get(key) == nil)
}

func (g geositeCache) Get(key string) *v2router.GeoSite {
	if g == nil {
		return nil
	}
	return g[key]
}

func (g geositeCache) Set(key string, value *v2router.GeoSite) {
	if g == nil {
		g = make(map[string]*v2router.GeoSite)
	}
	g[key] = value
}

```

This is a Go function that retrieves a GeoIP file by its filename and attempts to resolve it using the Google GeoIP cache or an online API if the file cannot be found. It takes the GeoIP file as input, decodes it into a v2router.GeoSite object if


```go
func (g geositeCache) Unmarshal(filename, code string) (*v2router.GeoSite, error) {
	asset := common.GetAssetLocation(filename)
	idx := strings.ToLower(asset + ":" + code)
	if g.Has(idx) {
		log.Debugf("geosite cache HIT: %s -> %s", code, idx)
		return g.Get(idx), nil
	}

	geositeBytes, err := Decode(asset, code)
	switch err {
	case nil:
		var geosite v2router.GeoSite
		if err := proto.Unmarshal(geositeBytes, &geosite); err != nil {
			return nil, err
		}
		g.Set(idx, &geosite)
		return &geosite, nil

	case ErrCodeNotFound:
		return nil, common.NewError("list " + code + " not found in " + filename)

	case ErrFailedToReadBytes, ErrFailedToReadExpectedLenBytes,
		ErrInvalidGeodataFile, ErrInvalidGeodataVarintLength:
		log.Warnf("failed to decode geoip file: %s, fallback to the original ReadFile method", filename)
		geositeBytes, err = ioutil.ReadFile(asset)
		if err != nil {
			return nil, err
		}
		var geositeList v2router.GeoSiteList
		if err := proto.Unmarshal(geositeBytes, &geositeList); err != nil {
			return nil, err
		}
		for _, geosite := range geositeList.GetEntry() {
			if strings.EqualFold(code, geosite.GetCountryCode()) {
				g.Set(idx, geosite)
				return geosite, nil
			}
		}

	default:
		return nil, err
	}

	return nil, common.NewError("list " + code + " not found in " + filename)
}

```

# `common/geodata/decode.go`

这段代码定义了一个名为 "geodata" 的包，其中包含一些用于解码和解析地理IP和地理坐标数据文件的工具。它依赖于名为 "GeoIPList" 和 "GeoSiteList" 的接口结构，该接口定义了地理IP和地理坐标的规范。

具体来说，这段代码实现了以下功能：

1. 通过导入 "google.golang.org/protobuf/encoding/protowire" 包，它允许程序在解析地理数据时使用 Google Protocol Buffers 的编码。
2. 定义了如何解析包含地理IP列表和地理坐标数据的文本文件。
3. 通过 "strings" 包（似乎无法在 Go 语言环境中使用，但在 Go 语言环境中应该可以使用 "golang.org/x/text/language/strings" 包）的 "error" 类型，实现对包含大量文本数据的文件进行处理。
4. 构造函数 "decode" 用于解析包含地理坐标数据的 GeoIP 文件。它通过 "google.golang.org/protobuf/encoding/protowire" 包中的 "ReadMessage" 函数读取原始消息，然后使用 "error" 类型的 "decode" 函数将原始消息转换为实际的地理坐标数据。
5. 定义了一个名为 "write" 的函数，用于将地理坐标数据写入文本文件。


```go
// Package geodata includes utilities to decode and parse the geoip & geosite dat files.
//
// It relies on the proto structure of GeoIP, GeoIPList, GeoSite and GeoSiteList in
// github.com/v2fly/v2ray-core/v4/app/router/config.proto to comply with following rules:
//
// 1. GeoIPList and GeoSiteList cannot be changed
// 2. The country_code in GeoIP and GeoSite must be
//    a length-delimited `string`(wired type) and has field_number set to 1
//
package geodata

import (
	"errors"
	"io"
	"os"
	"strings"

	"google.golang.org/protobuf/encoding/protowire"
)

```

这段代码是一个错误处理函数，它的作用是在函数调用时处理可能抛出的不同错误。

具体来说，这个函数的参数是一个读字节切片（io.ReadSeeker）和一个字符串参数code。函数内部首先定义了四个变量，并使用它们来根据错误类型设置计数器、判断是否是内部错误、创建一个临时数据容器以及根据传入的参数长度来设置相应的变量。

接着，函数使用for循环来对传入的code字符串进行遍历，根据varint类型长度来处理每种错误类型。在循环内部，使用var advancedN来记录一个计数器，表示已经处理过的varint类型长度。然后，使用isInner来判断当前遍历的类型是否是内部错误类型，如果是，则执行相应的错误处理代码。

最后，函数返回处理结果，并将计数器加1。


```go
var (
	ErrFailedToReadBytes            = errors.New("failed to read bytes")
	ErrFailedToReadExpectedLenBytes = errors.New("failed to read expected length of bytes")
	ErrInvalidGeodataFile           = errors.New("invalid geodata file")
	ErrInvalidGeodataVarintLength   = errors.New("invalid geodata varint length")
	ErrCodeNotFound                 = errors.New("code not found")
)

func EmitBytes(f io.ReadSeeker, code string) ([]byte, error) {
	count := 1
	isInner := false
	tempContainer := make([]byte, 0, 5)

	var result []byte
	var advancedN uint64 = 1
	var geoDataVarintLength, codeVarintLength, varintLenByteLen uint64 = 0, 0, 0

```

This is a function that reads a GeoIP/GeoSite list from a `tempContainer` of type `geoip.Varint` and returns the container, the number of matching varint values found, and the offset to the next varint value.

The function uses two loops to iterate through the `tempContainer` and compare the contents to the code stored in the `codeVarint`. If there is a match, the function advances to the next read offset, advances the `advancedN` counter, and skips the unmatched variable. If there is no match, the function skips the unmatched variable and continues to the next round of reading.

The function returns the container, the number of matching varint values found, and the offset to the next varint value. If an error occurs, the function returns an error with the appropriate message.


```go
Loop:
	for {
		container := make([]byte, advancedN)
		bytesRead, err := f.Read(container)
		if err == io.EOF {
			return nil, ErrCodeNotFound
		}
		if err != nil {
			return nil, ErrFailedToReadBytes
		}
		if bytesRead != len(container) {
			return nil, ErrFailedToReadExpectedLenBytes
		}

		switch count {
		case 1, 3: // data type ((field_number << 3) | wire_type)
			if container[0] != 10 { // byte `0A` equals to `10` in decimal
				return nil, ErrInvalidGeodataFile
			}
			advancedN = 1
			count++
		case 2, 4: // data length
			tempContainer = append(tempContainer, container...)
			if container[0] > 127 { // max one-byte-length byte `7F`(0FFF FFFF) equals to `127` in decimal
				advancedN = 1
				goto Loop
			}
			lenVarint, n := protowire.ConsumeVarint(tempContainer)
			if n < 0 {
				return nil, ErrInvalidGeodataVarintLength
			}
			tempContainer = nil
			if !isInner {
				isInner = true
				geoDataVarintLength = lenVarint
				advancedN = 1
			} else {
				isInner = false
				codeVarintLength = lenVarint
				varintLenByteLen = uint64(n)
				advancedN = codeVarintLength
			}
			count++
		case 5: // data value
			if strings.EqualFold(string(container), code) {
				count++
				offset := -(1 + int64(varintLenByteLen) + int64(codeVarintLength))
				f.Seek(offset, 1)               // back to the start of GeoIP or GeoSite varint
				advancedN = geoDataVarintLength // the number of bytes to be read in next round
			} else {
				count = 1
				offset := int64(geoDataVarintLength) - int64(codeVarintLength) - int64(varintLenByteLen) - 1
				f.Seek(offset, 1) // skip the unmatched GeoIP or GeoSite varint
				advancedN = 1     // the next round will be the start of another GeoIPList or GeoSiteList
			}
		case 6: // matched GeoIP or GeoSite varint
			result = container
			break Loop
		}
	}

	return result, nil
}

```

这段代码定义了一个名为 "Decode" 的函数，它接收两个参数：一个文件名和一个字符串 "code"。它的功能是解码一个 GeoJSON 编码的 JSON 数据。

函数首先使用 `os.Open` 函数打开文件行，如果失败，返回一个空字符串和错误。然后，它调用 `EmitBytes` 函数读取文件内容并编码为 GeoJSON 字节数组。如果这个步骤出现错误，函数也会返回一个空字符串和错误。

最后，函数返回解码后的 GeoJSON 字节数组，如果没有错误。函数的实现使用了操作系统接口库，比如 `os` 和 `google.golang.org/grpc`。


```go
func Decode(filename, code string) ([]byte, error) {
	f, err := os.Open(filename)
	if err != nil {
		return nil, err
	}
	defer f.Close()

	geoBytes, err := EmitBytes(f, code)
	if err != nil {
		return nil, err
	}
	return geoBytes, nil
}

```

# `common/geodata/decode_test.go`

这段代码是一个 Go 语言编写的测试包，用于测试 Geodata 包的功能。

具体来说，这段代码包括以下几个主要部分：

1. 导入必要的包：
	* "bytes" 包用于字符串操作，例如创建字节切片并获取缓冲区字符串；
	* "errors" 包用于错误处理，例如自定义错误类；
	* "io/fs" 包用于文件系统操作，例如打开文件和关闭文件；
	* "os" 包用于操作系统操作，例如获取当前路径；
	* "path/filepath" 包用于文件路径操作，例如生成文件路径并获取文件后缀。

2. 定义测试函数：
	* "test_get_point" 函数用于测试 Geodata.GetPoint 函数的功能，该函数用于获取指定点的经纬度坐标并返回；
	* "test_set_point" 函数用于测试 Geodata.SetPoint 函数的功能，该函数用于设置指定点的经纬度坐标；
	* "test_get_multi_points" 函数用于测试 Geodata.GetMultiPoints 函数的功能，该函数用于获取指定多点的经纬度坐标并返回。

3. 打印测试结果：
	* "test_get_point" 函数中的 "Expected: true" 和 "Actual: true" 行打印结果，验证获取指定点的经纬度坐标是否正确；
	* "test_set_point" 函数中的 "Expected: true" 和 "Actual: true" 行打印结果，验证设置指定点的经纬度坐标是否正确；
	* "test_get_multi_points" 函数中的 "Expected: 4 [1 2 3 4]" 和 "Actual: 4 [1 2 3 4]" 行打印结果，验证获取指定多点的经纬度坐标是否正确。


```go
package geodata_test

import (
	"bytes"
	"errors"
	"io/fs"
	"os"
	"path/filepath"
	"runtime"
	"testing"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/common/geodata"
)

```

这段代码是一个函数，名为 `init()`，它会在创建一个名为 `test` 目录的新目录中，下载并存储一个地理信息系统（GeoIP）文件和一个域名列表文件（GeoSite）的副本。

具体来说，它做了以下几件事情：

1. 初始化函数创建了一个名为 `test` 的新目录，并设置了一个名为 `TROJAN_GO_LOCATION_ASSET` 的环境变量，这个变量可能用于操作系统中的定位资产。

2. 下载并存储地理信息系统（GeoIP）文件。它使用 `geoipURL` 变量下载了 `geoip.dat` 文件，并使用 `os.Getwd()` 函数获取了当前工作目录。然后，它将文件存储在 `tempPath` 目录中，这个目录的权限为 755（rwxr-x---）。

3. 下载并存储域名列表文件（GeoSite）的副本。它使用 `geotypeURL` 变量下载了 `dlc.dat` 文件，并使用 `os.Getwd()` 函数获取了当前工作目录。然后，它将文件存储在 `tempPath` 目录中，这个目录的权限为 755（rwxr-x---）。

这些函数的作用可能与本项目的目的有关，但需要更多的上下文才能确定。


```go
func init() {
	const (
		geoipURL   = "https://raw.githubusercontent.com/v2fly/geoip/release/geoip.dat"
		geositeURL = "https://raw.githubusercontent.com/v2fly/domain-list-community/release/dlc.dat"
	)

	wd, err := os.Getwd()
	common.Must(err)

	tempPath := filepath.Join(wd, "..", "..", "test", "temp")
	os.Setenv("TROJAN_GO_LOCATION_ASSET", tempPath)

	geoipPath := common.GetAssetLocation("geoip.dat")
	geositePath := common.GetAssetLocation("geosite.dat")

	if _, err := os.Stat(geoipPath); err != nil && errors.Is(err, fs.ErrNotExist) {
		common.Must(os.MkdirAll(tempPath, 0o755))
		geoipBytes, err := common.FetchHTTPContent(geoipURL)
		common.Must(err)
		common.Must(common.WriteFile(geoipPath, geoipBytes))
	}
	if _, err := os.Stat(geositePath); err != nil && errors.Is(err, fs.ErrNotExist) {
		common.Must(os.MkdirAll(tempPath, 0o755))
		geositeBytes, err := common.FetchHTTPContent(geositeURL)
		common.Must(err)
		common.Must(common.WriteFile(geositePath, geositeBytes))
	}
}

```

这两个函数旨在测试一个名为 "decodeGeoIP" 的函数和一个名为 "decodeGeoSite" 的函数。它们都使用了 "geodata.Decode" 函数来从不同文件中的地理数据中检索数据。

这两个函数的差异在于它们测试的文件类型和预期输出。 "TestDecodeGeoIP" 函数测试了 "geoip.dat" 文件中的数据，而 "decodeGeoSite" 函数测试了 "geotype.dat" 文件中的数据。

函数内部首先获取 "geoip.dat" 文件的位置并尝试使用 "geodata.Decode" 函数加载数据。如果出现错误，函数将打印错误并输出错误信息。

如果数据加载成功，函数将检查 "geoip.dat" 文件中返回的数据是否与预期数据相等。如果数据不匹配，函数将打印错误并输出错误信息。

如果 "geotype.dat" 文件的数据与 "geoip.dat" 文件的数据不匹配，函数将不会输出任何错误信息，而是直接跳过。


```go
func TestDecodeGeoIP(t *testing.T) {
	filename := common.GetAssetLocation("geoip.dat")
	result, err := geodata.Decode(filename, "test")
	if err != nil {
		t.Error(err)
	}

	expected := []byte{10, 4, 84, 69, 83, 84, 18, 8, 10, 4, 127, 0, 0, 0, 16, 8}
	if !bytes.Equal(result, expected) {
		t.Errorf("failed to load geoip:test, expected: %v, got: %v", expected, result)
	}
}

func TestDecodeGeoSite(t *testing.T) {
	filename := common.GetAssetLocation("geosite.dat")
	result, err := geodata.Decode(filename, "test")
	if err != nil {
		t.Error(err)
	}

	expected := []byte{10, 4, 84, 69, 83, 84, 18, 20, 8, 3, 18, 16, 116, 101, 115, 116, 46, 101, 120, 97, 109, 112, 108, 101, 46, 99, 111, 109}
	if !bytes.Equal(result, expected) {
		t.Errorf("failed to load geosite:test, expected: %v, got: %v", expected, result)
	}
}

```

这段代码是一个名为 "BenchmarkLoadGeoIP" 的函数，属于 "runtime" 包。它的目的是测试一个名为 "geodata" 的库中 "NewGeodataLoader" 和 "LoadGeoIP" 函数的行为。为了实现这个目的，它的主要步骤如下：

1. 创建两个名为 "m1" 和 "m2" 的内存统计器，用于存储函数运行时创建的内存对象。
2. 创建一个名为 "loader" 的 "geodata.NewGeodataLoader" 实例。
3. 使用 "loader.LoadGeoIP" 函数分别加载名为 "cn" 和 "private" 的数据集。
4. 使用 "CN" 和 "private" 存储数据集的 ID。
5. 循环读取内存中的数据，直到所有数据加载完成。
6. 使用 "m2.Alloc" 和 "m1.Alloc" 分别记录每个数据集的内存分配和释放情况。
7. 使用 "m2.TotalAlloc" 和 "m1.TotalAlloc" 分别记录每个数据集的内存分配和释放总大小。
8. 计算函数运行时创建的内存对象之和与总内存对象之和之差，得到两个度量值（KiB 和 MiB）。
9. 调用 "BenchmarkLoadGeoIP" 函数并传入两个度量值，然后输出结果。

整个函数的作用是测试 "geodata" 包中的 "NewGeodataLoader" 和 "LoadGeoIP" 函数的正确性。它并不会对数据存储进行任何操作，只是一个测试函数。


```go
func BenchmarkLoadGeoIP(b *testing.B) {
	m1 := runtime.MemStats{}
	m2 := runtime.MemStats{}

	loader := geodata.NewGeodataLoader()

	runtime.ReadMemStats(&m1)
	cn, _ := loader.LoadGeoIP("cn")
	private, _ := loader.LoadGeoIP("private")
	runtime.KeepAlive(cn)
	runtime.KeepAlive(private)
	runtime.ReadMemStats(&m2)

	b.ReportMetric(float64(m2.Alloc-m1.Alloc)/1024, "KiB(GeoIP-Alloc)")
	b.ReportMetric(float64(m2.TotalAlloc-m1.TotalAlloc)/1024/1024, "MiB(GeoIP-TotalAlloc)")
}

```

这段代码是一个名为"BenchmarkLoadGeoSite"的函数，属于"testing"测试框架中的"runtime"包。它的目的是测试"GeoSite"类的加载性能。

具体来说，这段代码实现了以下功能：

1. 定义了一个名为"m3"的内存统计器和一个名为"m4"的内存统计器。
2. 使用"newGeodataLoader"方法创建了一个名为"loader"的Geo数据加载器。
3. 使用"LoadGeoSite"方法分别从"cn"、"geolocation-!cn"和"private"三个Geo数据源中加载数据。
4. 使用"KeepAlive"方法确保不会释放之前分配的内存。
5. 使用"ReadMemStats"方法记录Geo数据加载器中内存的使用情况。
6. 创建了一个"m4.Alloc"的指标，用于测量从"cn"数据源加载的内存数量。"m4.TotalAlloc"指标用于测量从"private"数据源加载的内存数量。
7. 运行基准测试套件，并输出"GeoSite-Alloc"和"GeoSite-TotalAlloc"指标的内存使用情况。

总之，这段代码实现了基准测试套件中"GeoSite"类的加载性能测试。


```go
func BenchmarkLoadGeoSite(b *testing.B) {
	m3 := runtime.MemStats{}
	m4 := runtime.MemStats{}

	loader := geodata.NewGeodataLoader()

	runtime.ReadMemStats(&m3)
	cn, _ := loader.LoadGeoSite("cn")
	notcn, _ := loader.LoadGeoSite("geolocation-!cn")
	private, _ := loader.LoadGeoSite("private")
	runtime.KeepAlive(cn)
	runtime.KeepAlive(notcn)
	runtime.KeepAlive(private)
	runtime.ReadMemStats(&m4)

	b.ReportMetric(float64(m4.Alloc-m3.Alloc)/1024/1024, "MiB(GeoSite-Alloc)")
	b.ReportMetric(float64(m4.TotalAlloc-m3.TotalAlloc)/1024/1024, "MiB(GeoSite-TotalAlloc)")
}

```

# `common/geodata/interface.go`

这段代码定义了一个名为 "GeodataLoader" 的接口，它表示可以加载地理数据（GeoIP 和 Gesite）的代码。这个接口使用了 "github.com/v2fly/v2ray-core/v4/app/router" 包中的 v2router。

接下来，我们为这个接口提供了一些方法，用于分别加载每个类型的地理数据：

- `LoadIP(filename, country) ([]*v2router.CIDR, error)`：加载一个国家（通过 "country" 参数）的 IP 地址。
- `LoadSite(filename, list) ([]*v2router.Domain, error)`：加载一个国家（通过 "list" 参数）的域名。
- `LoadGeoIP(country) ([]*v2router.CIDR, error)`：加载一个国家的 IP 地址。
- `LoadGeoSite(list, country) ([]*v2router.Domain, error)`：加载一个国家（通过 "country" 参数）的域名。


```go
package geodata

import v2router "github.com/v2fly/v2ray-core/v4/app/router"

type GeodataLoader interface {
	LoadIP(filename, country string) ([]*v2router.CIDR, error)
	LoadSite(filename, list string) ([]*v2router.Domain, error)
	LoadGeoIP(country string) ([]*v2router.CIDR, error)
	LoadGeoSite(list string) ([]*v2router.Domain, error)
}

```

# `common/geodata/loader.go`

这段代码定义了一个名为“geodata”的包，该包中包含了一个名为“GeodataLoader”的类型，以及一些函数和变量。

“GeodataLoader”类型有一个指向一个包含“*v2router.GeoIP”和“*v2router.GeoSite”的“geodataCache”的构造函数。

此外，还有两个名为“NewGeodataLoader”和“GeodataCache”的函数。

函数“NewGeodataLoader”返回一个“GeodataLoader”类型的变量，该变量使用上面定义的构造函数创建一个空的“geodataCache”结构体，然后返回该结构体。

函数“GeodataCache”返回一个包含“*v2router.GeoIP”和“*v2router.GeoSite”的“geodataCache”结构体，该结构体使用上面定义的构造函数初始化。


```go
package geodata

import (
	"runtime"

	v2router "github.com/v2fly/v2ray-core/v4/app/router"
)

type geodataCache struct {
	geoipCache
	geositeCache
}

func NewGeodataLoader() GeodataLoader {
	return &geodataCache{
		make(map[string]*v2router.GeoIP),
		make(map[string]*v2router.GeoSite),
	}
}

```

这两函数的作用是加载IP地址和网站信息到geodataCache中的geoip和geotable中。

`LoadIP`函数将输入的filename中的IP地址解析并返回，如果输入的文件名或解析IP地址时出现错误，函数返回 nil 和错误。

`LoadSite`函数将输入的filename中的网站信息解析并返回，如果输入的文件名或解析网站信息时出现错误，函数返回 nil 和错误。


```go
func (g *geodataCache) LoadIP(filename, country string) ([]*v2router.CIDR, error) {
	geoip, err := g.geoipCache.Unmarshal(filename, country)
	if err != nil {
		return nil, err
	}
	runtime.GC()
	return geoip.Cidr, nil
}

func (g *geodataCache) LoadSite(filename, list string) ([]*v2router.Domain, error) {
	geosite, err := g.geositeCache.Unmarshal(filename, list)
	if err != nil {
		return nil, err
	}
	runtime.GC()
	return geosite.Domain, nil
}

```

这两个函数接收一个名为 "g" 的指针变量和一个 "country" 参数 "string"。它们的作用是分别从 geodataCache 结构体中加载 IP 和站点数据，并将数据返回。

具体来说，这两个函数都在 cache 中查找相应的数据。如果数据存在，它们就会被加载到 g 指向的内存区域。如果 cache 中没有数据，这两个函数都会返回一个带有 error 的空切片。然后，这些数据将被用于返回给调用者。


```go
func (g *geodataCache) LoadGeoIP(country string) ([]*v2router.CIDR, error) {
	return g.LoadIP("geoip.dat", country)
}

func (g *geodataCache) LoadGeoSite(list string) ([]*v2router.Domain, error) {
	return g.LoadSite("geosite.dat", list)
}

```

# `component/api.go`

这段代码是一个 Go 语言中的 `package build` 包。它包含了一个开发者在生产环境构建应用程序时需要用到的构建步骤。

首先，它定义了一个名为 `api` 的变量，其值为 `full`。这里 `full` 是一个覆盖变量，它会覆盖 `api` 变量的默认值。所以，无论开发者在生产环境中是否定义了 `api` 变量，该变量的值都将为 `full`。

接下来，这个代码片段定义了一个名为 `build` 的函数。该函数将在应用程序的构建过程中执行。

然后，该函数导入了两个外部库的依赖：`github.com/p4gefau1t/trojan-go/api/control` 和 `github.com/p4gefau1t/trojan-go/api/service`。这两个库在构建应用程序时用于提供控制和服务的 API。

最后，该函数没有做任何其他事情，所以它的作用就是在应用程序的构建过程中提供一些必要的依赖。


```go
//go:build api || full
// +build api full

package build

import (
	_ "github.com/p4gefau1t/trojan-go/api/control"
	_ "github.com/p4gefau1t/trojan-go/api/service"
)

```

# `component/base.go`

这段代码是 Go 语言中的一个包，包含了以下内容：

1. 导入了一个名为 "github.com/p4gefau1t/trojan-go/log/golog" 的外部依赖，可能是一个用于记录日志的库。
2. 导入了一个名为 "github.com/p4gefau1t/trojan-go/statistic/memory" 的外部依赖，可能是一个用于记录内存使用情况的数据库。
3. 导入了一个名为 "github.com/p4gefau1t/trojan-go/version" 的外部依赖，可能是一个用于记录版本信息的地方。

根据这些依赖，可以推测出这个包可能是一个用于记录和统计应用程序的工具，它可能记录了应用程序的日志、内存使用情况以及版本信息。


```go
package build

import (
	_ "github.com/p4gefau1t/trojan-go/log/golog"
	_ "github.com/p4gefau1t/trojan-go/statistic/memory"
	_ "github.com/p4gefau1t/trojan-go/version"
)

```

# `component/client.go`

这段代码是一个 Go 语言中的 package.晴天( build ) 函数，用于构建 Go 语言中的 GoCD( Go Code Daemon ) 工具链。它通过运行一个名为 "build" 的函数，创建一个名为 "build" 的目录，并在其中生成一个名为 "client"、"full" 和 "mini" 的目录。这个目录通常会包含 GoCD 的一些依赖库，以便在构建时使用。

首先，该代码导入了github.com/p4gefau1t/trojan-go/proxy/client库，以便在构建过程中使用该库的代理。然后，接下来的两个星号( //* build client full mini\*/ 表示注释该代码块的开始位置和结束位置。这个注释区域的目的是让代码在这里提供一个简短的说明，但不会输出任何内容。

最后，该代码使用 build.sh 脚本来创建一个名为 "build" 的目录，并使用 createir 函数在该目录下创建 "client"、"full" 和 "mini" 目录。这个目录结构会在构建过程中被使用，因此，在运行 "build" 函数之前，这些目录可能已经存在，并且其中的文件可能已经被编译过。


```go
//go:build client || full || mini
// +build client full mini

package build

import (
	_ "github.com/p4gefau1t/trojan-go/proxy/client"
)

```

# `component/custom.go`

这段代码是一个 Go 语言编程语言的构建脚本，它定义了一个名为 "build" 的包。通过运行 "go build" 命令，将会编译并构建这个包。

这个包的目的是定义了 Go 语言的一些第三方库的导入，以及一些默认的导入。其中包括：

* github.com/p4gefau1t/trojan-go/proxy/custom
* github.com/p4gefau1t/trojan-go/tunnel/adapter
* github.com/p4gefau1t/trojan-go/tunnel/dokodemo
* github.com/p4gefau1t/trojan-go/tunnel/freedom
* github.com/p4gefau1t/trojan-go/tunnel/http
* github.com/p4gefau1t/trojan-go/tunnel/mux
* github.com/p4gefau1t/trojan-go/tunnel/router
* github.com/p4gefau1t/trojan-go/tunnel/shadowsocks
* github.com/p4gefau1t/trojan-go/tunnel/simplesocks
* github.com/p4gefau1t/trojan-go/tunnel/socks
* github.com/p4gefau1t/trojan-go/tunnel/tls
* github.com/p4gefau1t/trojan-go/tunnel/tproxy
* github.com/p4gefau1t/trojan-go/tunnel/transport
* github.com/p4gefau1t/trojan-go/tunnel/trojan
* github.com/p4gefau1t/trojan-go/tunnel/websocket

运行 "go build" 命令，将会编译并构建这个包，最终输出一个名为 "build" 的目录，里面会有上述库的依赖文件。


```go
//go:build custom || full
// +build custom full

package build

import (
	_ "github.com/p4gefau1t/trojan-go/proxy/custom"
	_ "github.com/p4gefau1t/trojan-go/tunnel/adapter"
	_ "github.com/p4gefau1t/trojan-go/tunnel/dokodemo"
	_ "github.com/p4gefau1t/trojan-go/tunnel/freedom"
	_ "github.com/p4gefau1t/trojan-go/tunnel/http"
	_ "github.com/p4gefau1t/trojan-go/tunnel/mux"
	_ "github.com/p4gefau1t/trojan-go/tunnel/router"
	_ "github.com/p4gefau1t/trojan-go/tunnel/shadowsocks"
	_ "github.com/p4gefau1t/trojan-go/tunnel/simplesocks"
	_ "github.com/p4gefau1t/trojan-go/tunnel/socks"
	_ "github.com/p4gefau1t/trojan-go/tunnel/tls"
	_ "github.com/p4gefau1t/trojan-go/tunnel/tproxy"
	_ "github.com/p4gefau1t/trojan-go/tunnel/transport"
	_ "github.com/p4gefau1t/trojan-go/tunnel/trojan"
	_ "github.com/p4gefau1t/trojan-go/tunnel/websocket"
)

```

# `component/forward.go`

这段代码是一个 Go 语言的构建命令，它使用了 `go build` 命令来构建 Go 语言项目。

这里 `//go:build forward || full || mini` 是语法注释，告诉 Go 语言编译器使用其中注释的分支来编译源代码。`||` 是一个逻辑 OR 运算符，用于处理编译器无法找到的命令行选项。

具体来说，这段代码会执行以下操作：

1. 如果当前目录下存在名为 `full` 的 Build 文件，则使用该文件编译。
2. 如果当前目录下存在名为 `mini` 的 Build 文件，则使用该文件编译。
3. 如果当前目录下既存在名为 `full` 的 Build 文件，又存在名为 `mini` 的 Build 文件，则使用其中名为 `full` 的文件编译。

如果当前目录下不存在上述三种 Build 文件，则默认使用 `go build` 命令编译源代码。


```go
//go:build forward || full || mini
// +build forward full mini

package build

import (
	_ "github.com/p4gefau1t/trojan-go/proxy/forward"
)

```