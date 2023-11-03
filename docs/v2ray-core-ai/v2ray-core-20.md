# v2ray-core源码解析 20

# `common/buf/writer_test.go`

这段代码是一个 Go 语言编写的测试框架，用于测试buf.io 库。它定义了一个名为"buf_test"的包，并且导入了几个常用的库：bufio、bytes、crypto/rand、io、testing 和github.com/google/go-cmp/cmp。

具体来说，这段代码的作用是提供一个测试框架，用于测试buf.io库的功能。它通过导入了必要的库，创建了一个测试buf.io管道，并提供了测试用例。测试用例包括创建一个缓冲区、发送数据到缓冲区、从缓冲区读取数据等基本操作。通过这些测试用例，可以验证buf.io库的功能是否符合预期。


```go
package buf_test

import (
	"bufio"
	"bytes"
	"crypto/rand"
	"io"
	"testing"

	"github.com/google/go-cmp/cmp"

	"v2ray.com/core/common"
	. "v2ray.com/core/common/buf"
	"v2ray.com/core/transport/pipe"
)

```

这段代码定义了一个名为 TestWriter 的测试函数，用于测试一个名为 writer 的 buffered writer 的功能。

函数内部，首先创建了一个名为 lb 的新的 buffer 对象，然后使用 common.Must2 函数从 rand.Reader 获取随机字节，并将其添加到 lb 的字节数组中。

接着创建了一个名为 writeBuffer 的 bytes.Buffer 对象，用于存储数据。

然后，创建了一个名为 writer 的 buffered writer 对象，并将其缓冲区设置为 writeBuffer。

接着，将 writer 的缓冲区设置为 false，并使用 writer.WriteMultiBuffer 函数向 buffer 对象中写入数据。

最后，比较预期的字节数组和 writeBuffer 的字节数是否相等，如果相等，则表示函数正常运行，否则会输出一个错误。


```go
func TestWriter(t *testing.T) {
	lb := New()
	common.Must2(lb.ReadFrom(rand.Reader))

	expectedBytes := append([]byte(nil), lb.Bytes()...)

	writeBuffer := bytes.NewBuffer(make([]byte, 0, 1024*1024))

	writer := NewBufferedWriter(NewWriter(writeBuffer))
	writer.SetBuffered(false)
	common.Must(writer.WriteMultiBuffer(MultiBuffer{lb}))
	common.Must(writer.Flush())

	if r := cmp.Diff(expectedBytes, writeBuffer.Bytes()); r != "" {
		t.Error(r)
	}
}

```

这段代码的作用是测试一个名为 `func TestBytesWriterReadFrom` 的函数。该函数的主要作用是测试一个名为 `ReadBytesFrom` 的函数，该函数从 `pipe.WithSizeLimit` 约束下的随机读取数据并以字节为单位写入一个 `BufferedWriter` 中。如果写入的字节数与期望的不符，或者函数执行成功但存在错误，则函数将输出错误信息。

具体而言，这段代码创建了一个名为 `pReader` 和 `pWriter` 的 `pipe.Reader` 和 `pipe.Writer` 实例，其中 `size` 变量表示数据读取的大小限制。然后，它创建了一个名为 `reader` 的 `bufio.Reader` 实例，用于从 `pipe.WithSizeLimit` 约束下的随机读取数据并以字节为单位写入 `pWriter`。接着，它创建了一个名为 `writer` 的 `BufferedWriter` 实例，设置其 `SetBuffered` 方法为 `false`，以便在写入数据时不会缓冲数据。

接下来，它使用 `reader` 将数据写入 `pWriter`，并使用 `pReader` 读取缓冲的数据。如果读取的数据字节数与期望的不符，或者函数执行成功但存在错误，则函数将输出错误信息，并捕获相应的错误并返回。


```go
func TestBytesWriterReadFrom(t *testing.T) {
	const size = 50000
	pReader, pWriter := pipe.New(pipe.WithSizeLimit(size))
	reader := bufio.NewReader(io.LimitReader(rand.Reader, size))
	writer := NewBufferedWriter(pWriter)
	writer.SetBuffered(false)
	nBytes, err := reader.WriteTo(writer)
	if nBytes != size {
		t.Fatal("unexpected size of bytes written: ", nBytes)
	}
	if err != nil {
		t.Fatal("expect success, but actually error: ", err.Error())
	}

	mb, err := pReader.ReadMultiBuffer()
	common.Must(err)
	if mb.Len() != size {
		t.Fatal("unexpected size read: ", mb.Len())
	}
}

```

这两段代码是对Go标准库中的“io”和“testing”包进行测试的函数。它们的作用是测试一个名为“DiscardBytes”的函数的正确性。

首先，这两段代码创建了一个名为“b”的包缓冲区。然后，它们使用“New”函数创建了一个新的包缓冲区，并使用“Must2”函数读取一个随机生成的测试数据“rand.Reader”和一个固定大小的数据“Size”（这里设置为10240个字节）。

接下来，这两段代码分别实现了两个名为“TestDiscardBytes”和“TestDiscardBytesMultiBuffer”的测试函数。这些函数分别使用了两种不同的方式来从“DiscardBytes”函数中复制数据。

第一个函数“TestDiscardBytes”的作用是测试从“discard bytes”函数中复制数据时，如果复制失败，应该能够正确地抛出错误。具体来说，这个函数创建了一个新的包缓冲区“b”，并使用“io.Copy”函数从“rand.Reader”和“Size”生成的测试数据中复制数据。然后，它使用“Must2”函数检查复制结果是否与预期大小“Size”相等。如果不相等，函数会输出一个错误并崩溃。

第二个函数“TestDiscardBytesMultiBuffer”的作用是测试从“discard bytes”函数中复制数据时，如果复制多个数据，应该能够正确地复制。具体来说，这个函数创建了一个新的包缓冲区“b”，并使用“io.Copy”函数从“rand.Reader”和“Size”生成的测试数据中复制多个数据。然后，它使用“Must2”函数检查复制结果是否与预期大小“Size”相等。如果相等，函数不会输出任何错误并成功执行。


```go
func TestDiscardBytes(t *testing.T) {
	b := New()
	common.Must2(b.ReadFullFrom(rand.Reader, Size))

	nBytes, err := io.Copy(DiscardBytes, b)
	common.Must(err)
	if nBytes != Size {
		t.Error("copy size: ", nBytes)
	}
}

func TestDiscardBytesMultiBuffer(t *testing.T) {
	const size = 10240*1024 + 1
	buffer := bytes.NewBuffer(make([]byte, 0, size))
	common.Must2(buffer.ReadFrom(io.LimitReader(rand.Reader, size)))

	r := NewReader(buffer)
	nBytes, err := io.Copy(DiscardBytes, &BufferedReader{Reader: r})
	common.Must(err)
	if nBytes != size {
		t.Error("copy size: ", nBytes)
	}
}

```

该代码测试了两个函数接口：Writer和BufferedWriter。主要目的是验证它们是否符合预期的类型，即Writer应该是可以写入数据到内存或文件，而BufferedWriter应该是可以缓冲数据并从缓冲区读取数据。

具体来说，代码首先定义了一个名为TestWriterInterface的函数，该函数接受一个测试框架对象t作为参数。函数内部定义了一个内部类型writer，它是一个实现了BufferToBytesWriter和BufferedWriter接口的对象。然后，通过两次switch语句来判断writer对象的实际类型，如果不符合期望的类型，函数将输出一个错误并导致t被清空。

具体实现可以分为以下几个步骤：

1. 创建一个Writer对象并将其赋值为<BufferToBytesWriter>(nil)，这样它实现了BufferToBytesWriter接口，但不支持缓冲数据。

2. 创建一个BufferedWriter对象并将其赋值为<BufferedWriter>(nil)，这样它实现了BufferedWriter接口，但不支持写入数据到内存或文件。

3. 遍历writer对象可以实现的类型，即Writer、io.Writer、io.ReaderFrom和io.ByteWriter。对于每种类型的writer，使用switch语句判断它是否实现了期望的类型。如果不符合期望类型，函数将输出一个错误并导致t被清空。

4. 调用TestWriterInterface函数，并将测试框架对象t作为参数，这将调用函数内部的两个switch语句，最终输出一个错误并导致t被清空。


```go
func TestWriterInterface(t *testing.T) {
	{
		var writer interface{} = (*BufferToBytesWriter)(nil)
		switch writer.(type) {
		case Writer, io.Writer, io.ReaderFrom:
		default:
			t.Error("BufferToBytesWriter is not Writer, io.Writer or io.ReaderFrom")
		}
	}

	{
		var writer interface{} = (*BufferedWriter)(nil)
		switch writer.(type) {
		case Writer, io.Writer, io.ReaderFrom, io.ByteWriter:
		default:
			t.Error("BufferedWriter is not Writer, io.Writer, io.ReaderFrom or io.ByteWriter")
		}
	}
}

```

# `common/bytespool/pool.go`

这段代码定义了一个名为“bytespool”的包，它提供了用于管理缓冲区的一些函数和变量。

首先，它导入了“sync”包，以便在函数中使用sync.Once。

然后，它定义了一个名为“createAllocFunc”的函数，该函数接收一个整型参数“size”和一个返回类型为“()”的整型参数。函数内部创建一个返回值为“()”的函数，该函数创建一个具有给定大小的缓冲区并返回它。

接着，它定义了一个名为“numPools”的变量，它用于控制缓冲区池的数量。它的值为4，表示从2KB大小开始，每个缓冲区的大小是前一个缓冲区大小的4倍。

然后，定义了一个名为“sizeMulti”的变量，它表示每个缓冲区的大小的系数。它的值为4，与“numPools”的值相同。

最后，通过调用“createAllocFunc”函数来创建一些缓冲区池，并且为了保证缓冲区池不会使用比最大的池更大的缓冲区，该代码使用了一个名为“Pool”的同步结构体。该结构体包含一个名为“pools”的变量，它表示所有当前的缓冲区池，以及一个名为“size”的变量，它用于存储分配给缓冲区池的缓冲区大小。


```go
package bytespool

import "sync"

func createAllocFunc(size int32) func() interface{} {
	return func() interface{} {
		return make([]byte, size)
	}
}

// The following parameters controls the size of buffer pools.
// There are numPools pools. Starting from 2k size, the size of each pool is sizeMulti of the previous one.
// Package buf is guaranteed to not use buffers larger than the largest pool.
// Other packets may use larger buffers.
const (
	numPools  = 4
	sizeMulti = 4
)

```

该代码创建了一个名为`sync.Pool`的同步空间，用于管理一组`int32`类型的数据结构，并赋予了该同步空间一个名为`numPools`的容量限制。

在`init()`函数中，首先定义了一个名为`size`的变量，该变量表示要创建的同步空间的最大容量。然后使用一个`for`循环来遍历`numPools`次，每次创建一个具有相同`sync.Pool`类型的对象。这个对象包含一个名为`createAllocFunc`的函数，用于在创建同步空间时分配内存。然后将`size`变量乘以自身，以便能够在创建同步空间时分配更多内存。

在`createAllocFunc()`函数中，没有做任何实际的内存分配操作，只是返回了一个`sync.Pool`类型的函数指针，用于将`size`变量分配给同步空间。

最后，`init()`函数将`sync.Pool`类型的对象全部初始化完成，开始创建同步空间，分配内存并将其注册到同步空间中。


```go
var (
	pool     [numPools]sync.Pool
	poolSize [numPools]int32
)

func init() {
	size := int32(2048)
	for i := 0; i < numPools; i++ {
		pool[i] = sync.Pool{
			New: createAllocFunc(size),
		}
		poolSize[i] = size
		size *= sizeMulti
	}
}

```

这段代码定义了一个名为 GetPool 的函数，它返回一个名为 sync.Pool 的同步大小池对象。这个函数接收一个整数参数 size，它表示要生成的字节数。如果 size 超过了任何已知池的大小，函数将返回 nil。

函数内部首先遍历池中所有可能大小的池，然后检查传入的 size 是否小于等于池中任意一个池的大小。如果是，函数返回该池的引用。如果不是，函数返回 nil。

该函数还有一个名为 Alloc 的辅助函数，它返回一个至少包含给定大小的字节切片。这个函数没有对输入参数 size 进行校验，因此它会返回一个大小为 2048（1 MB）的字节切片，而不是请求的任意大小。


```go
// GetPool returns a sync.Pool that generates bytes array with at least the given size.
// It may return nil if no such pool exists.
//
// v2ray:api:stable
func GetPool(size int32) *sync.Pool {
	for idx, ps := range poolSize {
		if size <= ps {
			return &pool[idx]
		}
	}
	return nil
}

// Alloc returns a byte slice with at least the given size. Minimum size of returned slice is 2048.
//
```

这两段代码是 v2ray 库中的两个函数，其作用是管理一个动态数据池。

函数 Alloc() 接受一个整型参数 size，返回一个整型数组类型的内存，如果内存池中存在同大小级的内存，则直接返回内存池中的内存，否则创建一个大小为 size 的内存并返回。

函数 Free() 接受一个字节数组参数 b，将其中的所有元素复制到内存池中的对应位置，并释放出内存池中同大小级的内存。

函数在内存池中维护了一组同大小级的内存，如果释放出的内存大小不符合某个内存池的大小要求，则会将内存合并到当前内存池中。

函数的实现遵循了开源代码的承诺，可以在需要时动态地分配和释放内存，从而实现高效的数据池管理。


```go
// v2ray:api:stable
func Alloc(size int32) []byte {
	pool := GetPool(size)
	if pool != nil {
		return pool.Get().([]byte)
	}
	return make([]byte, size)
}

// Free puts a byte slice into the internal pool.
//
// v2ray:api:stable
func Free(b []byte) {
	size := int32(cap(b))
	b = b[0:cap(b)]
	for i := numPools - 1; i >= 0; i-- {
		if size >= poolSize[i] {
			pool[i].Put(b) // nolint: megacheck
			return
		}
	}
}

```

# `common/cmdarg/cmdarg.go`

这段代码定义了一个名为“cmdarg”的包，其中定义了一个名为“Arg”的类型，该类型表示一个或多个字符串类型的参数。

定义了名为“*c”的指针变量，通过该指针变量可以访问“Arg”类型的参数。

定义了一个名为“String”的函数，该函数将给定的“Arg”类型的参数转换为字符串并返回。

定义了一个名为“Set”的函数，该函数接受一个字符串类型的参数，将其设置给“Arg”类型的参数。该函数不会返回错误，而是直接将字符串复制到参数中。


```go
package cmdarg

import "strings"

// Arg is used by flag to accept multiple argument.
type Arg []string

func (c *Arg) String() string {
	return strings.Join([]string(*c), " ")
}

// Set is the method flag package calls
func (c *Arg) Set(value string) error {
	*c = append(*c, value)
	return nil
}

```

# `common/crypto/aes.go`

这段代码定义了一个名为 "crypto" 的包，其中包含了一些与AES加密和 decryption 相关的函数和接口。

1. "import (": 
	* 导入了一些AES相关的标准库头文件和第三方库（如 "crypto/aes" 和 "v2ray.com/core/common"）。

2. "import (": 
	* 导入了一些第三方库头文件（如 "crypto/cipher" 和 "crypto/secretring/pb").

3. "NewAesDecryptionStream": 
	* 创建了一个名为 "NewAesDecryptionStream" 的函数，它接收两个字节切片（即 "key" 和 "iv"）。
	* 这个函数基于传入的 "key" 和 "iv" 创建一个新的AES加密流。
	* 函数创建的加密流的参数包括：
		+ "key"：AES密钥。
		+ "iv"：AES初始向量。
		+ "cipher.NewCFBDecrypter"：AES加密器的一个新的CFB（Counter-File Function-Based）decrypter。
	* 函数返回一个新的AES加密流实例。

4. "NewAesStreamMethod": 
	* 导入了一个名为 "NewAesStreamMethod" 的函数，这个函数创建一个AES流实例。
	* 该函数的第一个参数是 "key" 和 "iv"，第二个参数是一个Cipher的实例（这个Cipher必须是CFBDecrypter类型的实例）。
	* 函数创建的加密流的参数包括：
		+ "key"：AES密钥。
		+ "iv"：AES初始向量。
		+ "cipher.NewCFBDecrypter"：AES加密器的一个新的CFB（Counter-File Function-Based）decrypter。
		+ "cipher.NewCIPhersECPGLower(cipher.NewCIPhersSCD)"：一个Cipher的实例，该实例支持SCD（Single-数字货币）模式。
	* 函数返回一个新的AES流实例。


```go
package crypto

import (
	"crypto/aes"
	"crypto/cipher"

	"v2ray.com/core/common"
)

// NewAesDecryptionStream creates a new AES encryption stream based on given key and IV.
// Caller must ensure the length of key and IV is either 16, 24 or 32 bytes.
func NewAesDecryptionStream(key []byte, iv []byte) cipher.Stream {
	return NewAesStreamMethod(key, iv, cipher.NewCFBDecrypter)
}

```

这段代码定义了三个名为NewAesEncryptionStream、NewAesStreamMethod和NewAesCTRStream的函数，它们的作用是创建一个新的AES描述流。

NewAesEncryptionStream函数接受两个参数，一个是键(key)，另一个是初始化器(IV)，函数内部使用AESStreamMethod函数创建一个新的AES描述流，并返回该流。

NewAesStreamMethod函数接受两个参数，一个是键(key)，另一个是初始化器(IV)，函数内部使用AESStreamMethod函数创建一个新的AES描述流，并返回该流。这里需要注意，函数的第二个参数是一个函数指针，这个函数指针需要使用cipher.NewCFBEncrypter函数进行初始化。

NewAesCTRStream函数与NewAesStreamMethod类似，只是使用了AES-CTR而不是AES描述流。同样，函数内部使用NewAesStreamMethod函数创建一个新的AES描述流，并返回该流。


```go
// NewAesEncryptionStream creates a new AES description stream based on given key and IV.
// Caller must ensure the length of key and IV is either 16, 24 or 32 bytes.
func NewAesEncryptionStream(key []byte, iv []byte) cipher.Stream {
	return NewAesStreamMethod(key, iv, cipher.NewCFBEncrypter)
}

func NewAesStreamMethod(key []byte, iv []byte, f func(cipher.Block, []byte) cipher.Stream) cipher.Stream {
	aesBlock, err := aes.NewCipher(key)
	common.Must(err)
	return f(aesBlock, iv)
}

// NewAesCTRStream creates a stream cipher based on AES-CTR.
func NewAesCTRStream(key []byte, iv []byte) cipher.Stream {
	return NewAesStreamMethod(key, iv, cipher.NewCTR)
}

```

这段代码定义了一个名为NewAesGcm的函数，它接收一个字节切片（key）作为参数并返回一个名为cipher.AEAD的gcm加密器。

函数首先使用aes.NewCipher函数根据传入的key生成一个AES-GCM密码机，并返回该密码机。接着，使用cipher.NewGCM函数创建一个gcm加密器，并将其与之前生成的密码机一起返回。最终，函数返回生成的加密器。


```go
// NewAesGcm creates a AEAD cipher based on AES-GCM.
func NewAesGcm(key []byte) cipher.AEAD {
	block, err := aes.NewCipher(key)
	common.Must(err)
	aead, err := cipher.NewGCM(block)
	common.Must(err)
	return aead
}

```

# `common/crypto/auth.go`

这段代码定义了一个名为“crypto”的包，该包包含了一些与加密有关的函数和变量。下面是对该包中的一些关键部分的解释：

1. 导入了一个名为“crypto/cipher”的包，该包提供了一些加密算法的实现。

2. 导入了“io”包，该包提供了输入/输出操作的支持。

3. 导入了“math/rand”包，该包提供了随机数的生成支持。

4. 定义了一个名为“BytesGenerator”的函数，该函数没有具体的实现，但可以用来创建一个字节切片。

5. 在包的头部定义了一个常量“package crypto”，该常量似乎用于告诉编译器该包的主要作用。

6. 在“crypto/cipher”包中导入了几个函数，包括“aes128”、“aes256”、“aes512”等，这些函数分别提供了AES 128位、AES 256位和AES 512位加密算法的实现。

7. 在“io”包中导入了“net”包，该包提供了网络套接字的支持。

8. 在“math/rand”包中导入了几个函数，包括“genrand”和“随地散列”，这些函数可以用来生成随机数。

9. 在“v2ray.com/core/common”包中导入了几个函数，包括“common/util”和“common/encoding”，这些函数可以用来处理一些通用的任务。

10. 在“crypto/cipher”包中导入了几个函数，包括“crypto/encryptor”和“crypto/decryptor”，这些函数可以用来对字节数据进行加密和解密操作。

11. 在“io”包中导入了“tcp”包，该包提供了对TCP套接字的操作支持。

12. 在“v2ray.com/core/common”包中导入了几个函数，包括“common/iputil”和“common/urlresolv”，这些函数可以用来处理IP地址和其他通


```go
package crypto

import (
	"crypto/cipher"
	"io"
	"math/rand"

	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/bytespool"
	"v2ray.com/core/common/protocol"
)

type BytesGenerator func() []byte

```

这三段代码都是定义了一个名为“BytesGenerator”的接口类型，并实现了该接口的函数。

第一段代码定义了一个名为“GenerateEmptyBytes”的函数，其作用是创建一个长度为1的 byte 数组，并返回该数组的空字符串。该函数的实现是调用了“b[:0]”方法，将数组的开始位置清零，得到一个空字符串。

第二段代码定义了一个名为“GenerateStaticBytes”的函数，其作用是创建一个与传入内容相同长度的 byte 数组，并返回该数组。该函数的实现是直接将传入的内容复制到数组中，并返回该数组。

第三段代码定义了一个名为“GenerateIncreasingNonce”的函数，其作用是在一个已经存在的 byte 数组上递增，并返回递增后的数组。该函数的实现是创建一个新的数组，并将其与传入的数组内容相同，然后将数组的每个元素都递增1，并循环直到数组的最后一个元素不为0，然后返回该数组。


```go
func GenerateEmptyBytes() BytesGenerator {
	var b [1]byte
	return func() []byte {
		return b[:0]
	}
}

func GenerateStaticBytes(content []byte) BytesGenerator {
	return func() []byte {
		return content
	}
}

func GenerateIncreasingNonce(nonce []byte) BytesGenerator {
	c := append([]byte(nil), nonce...)
	return func() []byte {
		for i := range c {
			c[i]++
			if c[i] != 0 {
				break
			}
		}
		return c
	}
}

```

这段代码定义了一个名为AEADNonce的函数，其作用是生成一个自增的、非同构的nonce，并返回给调用者。函数的实现包括两个主要部分：生成nonce和实现AEADAuthenticator类型的Authenticator接口。

1. `GenerateIncreasingNonce([]byte{0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF})`函数用于生成一个长度为16的nonce，通过不断地增加计数器来实现。函数的返回值是一个生成的nonce数组。

2. `type Authenticator interface {`定义了一个名为Authenticator的接口，它包含三个方法：NonceSize、Overhead和Open。

3. `type AEADAuthenticator struct {`定义了一个名为AEADAuthenticator的 struct，它包含一个AEAD类型的cipher.AEAD成员和一个BytesGenerator类型的AdditionalDataGenerator成员。

4. `AEAD`类型表示AEAD（先进证据）密码算法，它可以确保密码的完整性、真实性和可逆性。

5. `NonceGenerator`类型是一个BytesGenerator，用于生成nonce。

6. `AdditionalDataGenerator`类型是一个BytesGenerator，用于生成其他数据，如密钥或消息。

7. `func GenerateInitialAEADNonce() BytesGenerator`函数返回一个名为GenerateInitialAEADNonce的函数，它使用AEADAuthenticator类型的实例生成一个初始的nonce，并返回给调用者。函数创建一个名为AEADNonce的临时函数，实现AEADAuthenticator接口的生成nonce方法，设置AEADAuthenticator实例的AEAD类型的函数生成nonce，同时设置其他类型函数生成其他数据。


```go
func GenerateInitialAEADNonce() BytesGenerator {
	return GenerateIncreasingNonce([]byte{0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF})
}

type Authenticator interface {
	NonceSize() int
	Overhead() int
	Open(dst, cipherText []byte) ([]byte, error)
	Seal(dst, plainText []byte) ([]byte, error)
}

type AEADAuthenticator struct {
	cipher.AEAD
	NonceGenerator          BytesGenerator
	AdditionalDataGenerator BytesGenerator
}

```

这两函数的作用是`AEADAuthenticator`到`Plaintext`的序列化和认证。

`Open`函数接收两个参数，一个是目的地址 `dst` 和一个字符串类型的密文 `cipherText`。然后，它使用AEAD的 `Open` 函数进行序列化和认证。在函数内部，首先生成一个初始化向量 `iv`，其长度应与 `AEAD.NonceSize()` 相等。然后，根据需要的的话，还生成一个 `additionalData` 数组，它可能与 `AEAD.additionalDataSize()` 相等。最后，使用 `AEAD.Open` 函数在传递 `dst`、`iv` 和 `cipherText` 参数的基础上，返回加密后的结果，如果初始化失败或者生成 `additionalData` 失败则返回一个错误。

`Seal` 函数接收两个参数，一个是目的地址 `dst` 和一个字符串类型的明文 `plainText`。然后，它使用AEAD的 `Seal` 函数进行序列化和认证。在函数内部，首先生成一个初始化向量 `iv`，其长度应与 `AEAD.NonceSize()` 相等。然后，根据需要的的话，还生成一个 `additionalData` 数组，它可能与 `AEAD.additionalDataSize()` 相等。最后，使用 `AEAD.Seal` 函数在传递 `dst`、`iv` 和 `plainText` 参数的基础上，返回加密后的结果，如果初始化失败或者生成 `additionalData` 失败则返回一个错误。


```go
func (v *AEADAuthenticator) Open(dst, cipherText []byte) ([]byte, error) {
	iv := v.NonceGenerator()
	if len(iv) != v.AEAD.NonceSize() {
		return nil, newError("invalid AEAD nonce size: ", len(iv))
	}

	var additionalData []byte
	if v.AdditionalDataGenerator != nil {
		additionalData = v.AdditionalDataGenerator()
	}
	return v.AEAD.Open(dst, iv, cipherText, additionalData)
}

func (v *AEADAuthenticator) Seal(dst, plainText []byte) ([]byte, error) {
	iv := v.NonceGenerator()
	if len(iv) != v.AEAD.NonceSize() {
		return nil, newError("invalid AEAD nonce size: ", len(iv))
	}

	var additionalData []byte
	if v.AdditionalDataGenerator != nil {
		additionalData = v.AdditionalDataGenerator()
	}
	return v.AEAD.Seal(dst, iv, plainText, additionalData), nil
}

```

该代码定义了一个名为AuthenticationReader的结构体，用于描述一个读取流的设计和属性。

该结构体包含以下字段：

-   `auth`：一个名为`Authenticator`的类型，用于描述认证提供者。
-   `reader`：一个指向一个`buf.BufferedReader`类型的变量，用于读取数据。
-   `sizeParser`：一个名为`ChunkSizeDecoder`的类型，用于解析输入数据中分块的大小。
-   `sizeBytes`：一个包含输入数据分块的大小的字节数组。
-   `transferType`：一个名为`protocol.TransferType`的类型，用于描述数据传输类型。
-   `padding`：一个名为`PaddingLengthGenerator`的类型，用于生成数据填充的长度。
-   `size`：一个名为`uint16`的类型，用于表示数据分块的大小。
-   `paddingLen`：一个名为`uint16`的类型，用于表示数据填充的长度。
-   `hasSize`：一个名为`bool`的类型，用于表示数据分块的大小是否存在。
-   `done`：一个名为`bool`的类型，表示是否已经完成数据读取。

该结构体还包含一个名为`NewAuthenticationReader`的函数，该函数接受一个`Authenticator`、一个`ChunkSizeDecoder`、一个`Reader`和一个`TransferType`，用于创建一个新的`AuthenticationReader`实例。该函数首先根据传递的参数初始化`AuthenticationReader`实例，然后调用`Reader`函数创建一个新的`BufferedReader`实例，并将其指向输入数据的开始位置。最后，将创建的`AuthenticationReader`实例返回给调用者。

该结构体还包含一个名为`Size`的函数，用于计算数据分块的大小，该函数首先尝试从`Reader`提供的读取位置开始计算数据分块的大小，如果失败，则返回一个固定长度的大小。

该结构体还包含一个名为`PaddingLengthGenerator`的函数，用于生成数据填充的长度，该函数使用传递的`PaddingLengthGenerator`类型来获取设置的长度，并返回填充长度。

该结构体还包含一个名为`TransferType`的函数，用于将数据传输类型转换为字符串。

该结构体还包含一个名为`SizeBytes`的函数，用于将`size`字节数组转换为`ChunkSizeDecoder`类型。

该结构体还包含一个名为`Authenticator`的函数，用于从认证提供者获取用户身份信息。

该结构体还包含一个名为`SizeDecoder`的函数，用于解析`Authenticator`提供的身份信息中的可用字段。

该结构体还包含一个名为`DecodedAuthInfo`的函数，用于将解码后的身份信息存储在内存中。

该结构体还包含一个名为`AuthInfo`的函数，用于将解码后的身份信息存储在内存中。


```go
type AuthenticationReader struct {
	auth         Authenticator
	reader       *buf.BufferedReader
	sizeParser   ChunkSizeDecoder
	sizeBytes    []byte
	transferType protocol.TransferType
	padding      PaddingLengthGenerator
	size         uint16
	paddingLen   uint16
	hasSize      bool
	done         bool
}

func NewAuthenticationReader(auth Authenticator, sizeParser ChunkSizeDecoder, reader io.Reader, transferType protocol.TransferType, paddingLen PaddingLengthGenerator) *AuthenticationReader {
	r := &AuthenticationReader{
		auth:         auth,
		sizeParser:   sizeParser,
		transferType: transferType,
		padding:      paddingLen,
		sizeBytes:    make([]byte, sizeParser.SizeBytes()),
	}
	if breader, ok := reader.(*buf.BufferedReader); ok {
		r.reader = breader
	} else {
		r.reader = &buf.BufferedReader{Reader: buf.NewReader(reader)}
	}
	return r
}

```

此函数的作用是读取AuthenticationReader类型的实例中的输入数据。它接受一个指向该实例的引用变量r，并返回两个uint16类型的值：已读入数据的大小和padding长度。

如果输入已包含足够的数据，函数将返回已读入数据的大小、一个占用该数据额外空间的padding长度，以及一个 nil错误。如果出现错误，函数将返回0、0和错误对象。

如果r.reader是一个已关闭的输入流，函数可能不会读入任何数据，此时已读入数据的大小将为0,padding长度将为r.padding的大小(如果存在)，错误将为nil。

如果r.padding是一个非空字符串，函数将尝试使用io.ReadFull函数从r.reader中读取数据。如果错误发生，函数将返回错误对象。如果数据成功读入，函数将使用r.sizeParser.Decode函数将读入的数据大小解析为uint16类型，并将padding长度设置为下一个padding字符串的长度。最后，函数将返回已读入数据的大小、padding长度和错误对象。


```go
func (r *AuthenticationReader) readSize() (uint16, uint16, error) {
	if r.hasSize {
		r.hasSize = false
		return r.size, r.paddingLen, nil
	}
	if _, err := io.ReadFull(r.reader, r.sizeBytes); err != nil {
		return 0, 0, err
	}
	var padding uint16
	if r.padding != nil {
		padding = r.padding.NextPaddingLen()
	}
	size, err := r.sizeParser.Decode(r.sizeBytes)
	return size, padding, err
}

```

这段代码定义了一个名为`AuthenticationReader`的类型，该类型有一个名为`readBuffer`的函数，接受两个参数：`size`表示每次读取的数据量，`padding`表示在每次读取数据之前需要进行的填充操作。

`readBuffer`函数的作用是读取一个`buf.Buffer`对象，并返回其中的数据或者错误。其具体实现包括以下几个步骤：

1. 创建一个名为`b`的`buf.Buffer`对象。
2. 如果遇到错误，就释放当前`buf.Buffer`对象，并返回错误。
3. 从`r.reader`中读取指定大小的数据并写入到`b`中。
4. 从`r.auth`对象中打开一个可以写入到`b`中的数据缓冲区，并写入到`rb`中。
5. 将`rb`中的数据读取到`b`中，并调整`b`的大小以适应所有读取的数据。
6. 如果`readBuffer`函数没有返回任何值，那么它需要负责的副作用包括：
	* 创建并分配给`b`的内存将被释放。
	* `r.reader`和`r.auth`对象的引用将不再有效。


```go
var errSoft = newError("waiting for more data")

func (r *AuthenticationReader) readBuffer(size int32, padding int32) (*buf.Buffer, error) {
	b := buf.New()
	if _, err := b.ReadFullFrom(r.reader, size); err != nil {
		b.Release()
		return nil, err
	}
	size -= padding
	rb, err := r.auth.Open(b.BytesTo(0), b.BytesTo(size))
	if err != nil {
		b.Release()
		return nil, err
	}
	b.Resize(0, int32(len(rb)))
	return b, nil
}

```

这段代码是一个名为`func`的函数，它是`AuthenticationReader`类型的指针函数。它的作用是读取数据并将其存储在`MultiBuffer`类型的变量`mb`中。以下是该函数的实现步骤：

1. 如果`soft`为`true`并且`r.reader.BufferedBytes()`小于`r.sizeParser.SizeBytes()`，则函数返回一个错误。
2. 如果`r.done`为`true`，则函数返回一个错误。
3. 计算`r.readSize()`，如果出现错误，返回错误。
4. 如果`size`等于`uint16(r.auth.Overhead())+padding`，则函数返回一个错误。
5. 如果`soft`为`true`并且`int32(size)`大于`r.reader.BufferedBytes()`，则函数返回一个错误。
6. 如果`size`小于或等于`buf.Size`，则函数首先尝试从`r.auth`的原始数据中读取数据，并将其存储在`MultiBuffer`类型的变量`mb`中。
7. 最后，函数通过`io.ReadFull`函数从原始数据中读取剩余的数据并将它们存储在`MultiBuffer`类型的变量`mb`中。


```go
func (r *AuthenticationReader) readInternal(soft bool, mb *buf.MultiBuffer) error {
	if soft && r.reader.BufferedBytes() < r.sizeParser.SizeBytes() {
		return errSoft
	}

	if r.done {
		return io.EOF
	}

	size, padding, err := r.readSize()
	if err != nil {
		return err
	}

	if size == uint16(r.auth.Overhead())+padding {
		r.done = true
		return io.EOF
	}

	if soft && int32(size) > r.reader.BufferedBytes() {
		r.size = size
		r.paddingLen = padding
		r.hasSize = true
		return errSoft
	}

	if size <= buf.Size {
		b, err := r.readBuffer(int32(size), int32(padding))
		if err != nil {
			return nil
		}
		*mb = append(*mb, b)
		return nil
	}

	payload := bytespool.Alloc(int32(size))
	defer bytespool.Free(payload)

	if _, err := io.ReadFull(r.reader, payload[:size]); err != nil {
		return err
	}

	size -= padding

	rb, err := r.auth.Open(payload[:0], payload[:size])
	if err != nil {
		return err
	}

	*mb = buf.MergeBytes(*mb, rb)
	return nil
}

```

这段代码是一个名为`func`的函数，其接收一个名为`r`的整数类型的指针变量，代表一个`AuthenticationReader`类型的对象。函数的作用是读取多个`MultiBuffer`类型的数据，并返回一个`buf.MultiBuffer`类型的数据和一个`error`类型的数据。

具体来说，函数首先创建一个长度为`readSize`的`MultiBuffer`类型的变量`mb`，并使用`make`函数创建一个包含`readSize`个元素的`MultiBuffer`。然后，函数调用一个名为`r.readInternal`的内部方法，这个方法以二进制模式读取数据并返回一个`MultiBuffer`类型的数据。如果这个方法返回一个非`MultiBuffer`类型的错误，函数将释放`MultiBuffer`并返回该错误。

接下来，函数使用一个循环来读取多个`MultiBuffer`类型的数据。循环从第二个元素开始，每隔一个元素执行一次`r.readInternal`的函数。如果函数返回一个非`MultiBuffer`类型的错误或者读取到了数据结束，函数将释放`MultiBuffer`并返回该错误。如果函数在循环中没有遇到任何错误，函数将返回`MultiBuffer`类型的数据，并将最后一个读取的数据赋值给`mb`。

最后，函数返回`mb`和`nil`，并将`MultiBuffer`类型的数据传递给`buf.ReleaseMulti`函数，以便释放内存。


```go
func (r *AuthenticationReader) ReadMultiBuffer() (buf.MultiBuffer, error) {
	const readSize = 16
	mb := make(buf.MultiBuffer, 0, readSize)
	if err := r.readInternal(false, &mb); err != nil {
		buf.ReleaseMulti(mb)
		return nil, err
	}

	for i := 1; i < readSize; i++ {
		err := r.readInternal(true, &mb)
		if err == errSoft || err == io.EOF {
			break
		}
		if err != nil {
			buf.ReleaseMulti(mb)
			return nil, err
		}
	}

	return mb, nil
}

```

这段代码定义了一个名为AuthenticationWriter的结构体，它包含以下字段：

-   `auth`：一个Authenticator类型的字段，用于管理身份验证。
-   `writer`：一个buf.Writer类型的字段，用于写入数据到缓冲区。
-   `sizeParser`：一个ChunkSizeEncoder类型的字段，用于解析缓冲区中的数据并计算数据的大小。
-   `transferType`：一个protocol.TransferType类型的字段，用于指定数据传输类型。
-   `padding`：一个PaddingLengthGenerator类型的字段，用于在传输数据之前对其进行填充。

此外，还包含一个名为`NewAuthenticationWriter`的函数，该函数接收一个Authenticator、一个缓冲区大小解析器、一个写入者、一个传输类型和一个填充器作为参数，并返回一个指向新AuthenticationWriter实例的引用。

最后，还包含一个名为`WriteChunkedData`的函数，该函数将缓冲区中的数据按照传输类型进行分片，并逐个写入到写入者中。


```go
type AuthenticationWriter struct {
	auth         Authenticator
	writer       buf.Writer
	sizeParser   ChunkSizeEncoder
	transferType protocol.TransferType
	padding      PaddingLengthGenerator
}

func NewAuthenticationWriter(auth Authenticator, sizeParser ChunkSizeEncoder, writer io.Writer, transferType protocol.TransferType, padding PaddingLengthGenerator) *AuthenticationWriter {
	w := &AuthenticationWriter{
		auth:         auth,
		writer:       buf.NewWriter(writer),
		sizeParser:   sizeParser,
		transferType: transferType,
	}
	if padding != nil {
		w.padding = padding
	}
	return w
}

```

这段代码定义了一个名为`seal`的函数，接受一个名为`w`的`AuthenticationWriter`类型的参数和一个字节切片`b`。函数的作用是加密一个字节切片`b`，并返回加密后的字节切片和错误。

具体来说，这段代码执行以下操作：

1. 计算要存储的数据长度，包括认证开发生成的 overhead，以及可能需要的填充长度。
2. 如果`padding`字段不为空，则计算所需的填充长度。
3. 计算已经编码过的数据长度，加上计算得出的填充长度，得到总长度。
4. 如果生成的数据长度超过缓冲区`buf`的大小，函数返回一个错误，并释放内存。
5. 对填充长度进行随机化，以确保数据的随机性。
6. 如果`padding`字段为`nil`，并且填充长度为`0`，函数仍然会尝试对填充长度进行随机化，即使在这种情况下，也不会创建任何缓冲区，因此仍然会返回一个错误。


```go
func (w *AuthenticationWriter) seal(b []byte) (*buf.Buffer, error) {
	encryptedSize := int32(len(b) + w.auth.Overhead())
	var paddingSize int32
	if w.padding != nil {
		paddingSize = int32(w.padding.NextPaddingLen())
	}

	sizeBytes := w.sizeParser.SizeBytes()
	totalSize := sizeBytes + encryptedSize + paddingSize
	if totalSize > buf.Size {
		return nil, newError("size too large: ", totalSize)
	}

	eb := buf.New()
	w.sizeParser.Encode(uint16(encryptedSize+paddingSize), eb.Extend(sizeBytes))
	if _, err := w.auth.Seal(eb.Extend(encryptedSize)[:0], b); err != nil {
		eb.Release()
		return nil, err
	}
	if paddingSize > 0 {
		// With size of the chunk and padding length encrypted, the content of padding doesn't matter much.
		paddingBytes := eb.Extend(paddingSize)
		common.Must2(rand.Read(paddingBytes))
	}

	return eb, nil
}

```

这段代码定义了一个名为`func`的函数，接收一个名为`w`的`AuthenticationWriter`类型和一个名为`mb`的`MultiBuffer`类型的参数。

函数的作用是帮助`AuthenticationWriter`写入数据到给定的`MultiBuffer`中。首先，它确定最大允许的填充长度，然后创建一个大小为`len(mb)+10`的`MultiBuffer`，用于存放写入的数据。接着，它遍历输入数据，将`MultiBuffer`中的数据与输入数据分割，然后对输入数据进行签名。如果输入数据长度小于设定的最大长度，函数将使用新创建的`MultiBuffer`中的数据覆盖输入数据，并继续遍历。最后，函数调用`writer.WriteMultiBuffer`将写入的`MultiBuffer`中的数据写入到给定的`MultiBuffer`中。

函数的实现要点如下：

1. 函数首先定义了一个`maxPadding`变量，用于记录`AuthenticationWriter`中的最大可填充长度，如果没有设置该值，则默认为`0`。
2. 函数检查`padding`是否为`nil`，如果是，则假设最大填充长度为`w.padding.MaxPaddingLen()`，否则假设最大填充长度为`maxPadding`。
3. 函数根据`padding`的值计算出最大允许的填充长度，然后创建一个大小为`len(mb)+10`的`MultiBuffer`，用于存放写入的数据。
4. 函数遍历输入数据，将`MultiBuffer`中的数据与输入数据分割，然后对输入数据进行签名。如果输入数据长度小于设定的最大长度，函数将使用新创建的`MultiBuffer`中的数据替换输入数据，并继续遍历。
5. 函数调用`writer.WriteMultiBuffer`将写入的`MultiBuffer`中的数据写入到给定的`MultiBuffer`中。


```go
func (w *AuthenticationWriter) writeStream(mb buf.MultiBuffer) error {
	defer buf.ReleaseMulti(mb)

	var maxPadding int32
	if w.padding != nil {
		maxPadding = int32(w.padding.MaxPaddingLen())
	}

	payloadSize := buf.Size - int32(w.auth.Overhead()) - w.sizeParser.SizeBytes() - maxPadding
	mb2Write := make(buf.MultiBuffer, 0, len(mb)+10)

	temp := buf.New()
	defer temp.Release()

	rawBytes := temp.Extend(payloadSize)

	for {
		nb, nBytes := buf.SplitBytes(mb, rawBytes)
		mb = nb

		eb, err := w.seal(rawBytes[:nBytes])

		if err != nil {
			buf.ReleaseMulti(mb2Write)
			return err
		}
		mb2Write = append(mb2Write, eb)
		if mb.IsEmpty() {
			break
		}
	}

	return w.writer.WriteMultiBuffer(mb2Write)
}

```

此函数的作用是创建一个名为 "AuthenticationWriter" 的输出缓冲区 "w"。该函数接受一个名为 "mb" 的缓冲区作为参数，并返回一个错误。

函数中包含一个名为 "writePacket" 的内部函数，该函数接受一个名为 "mb" 的缓冲区作为参数，并返回一个错误。

函数的作用可以总结为以下几个步骤：

1. 创建一个名为 "mb2Write" 的缓冲区，该缓冲区的大小为 len(mb) + 1，其中 len(mb) 是缓冲区中 "mb" 长度参数的输入，+1 是用于容纳 "mb" 中的数据和一个空的 "MultiBuffer"。

2. 遍历 "mb" 中的所有数据包，并对每个数据包执行以下操作：

  a. 如果数据包是空字符串，则跳过。

  b. 将数据包的字节编码并存储到 "mb2Write" 缓冲区中的 "e" 字段中。

3. 如果 "mb2Write" 缓冲区中的所有元素都是空的，则返回一个错误。

4. 调用 "w.writer.WriteMultiBuffer" 函数，并将 "mb2Write" 缓冲区作为参数，这将导致 "w.writer" 函数在缓冲区中的所有元素都被写入 "w" 输出缓冲区中。

5. 如果 "mb2Write" 缓冲区中至少有一个元素，则返回 "w.writer.WriteMultiBuffer" 函数的返回值。

6. 如果没有返回值，则返回一个错误。


```go
func (w *AuthenticationWriter) writePacket(mb buf.MultiBuffer) error {
	defer buf.ReleaseMulti(mb)

	mb2Write := make(buf.MultiBuffer, 0, len(mb)+1)

	for _, b := range mb {
		if b.IsEmpty() {
			continue
		}

		eb, err := w.seal(b.Bytes())
		if err != nil {
			continue
		}

		mb2Write = append(mb2Write, eb)
	}

	if mb2Write.IsEmpty() {
		return nil
	}

	return w.writer.WriteMultiBuffer(mb2Write)
}

```

这段代码定义了一个名为`WriteMultiBuffer`的函数，它实现了`buf.Writer`接口。这个函数接收一个名为`mb`的`buf.MultiBuffer`对象，并返回一个错误。

如果`mb`为空，函数会尝试使用`w.seal`函数来创建一个空的`buf.MultiBuffer`对象。如果这个函数成功，它会将`mb`的`IsEmpty()`方法返回的`true`作为参数传递给`w.writer.WriteMultiBuffer`函数，这个函数会将`mb`中的数据写入到缓冲区中。如果这个函数失败，那么返回一个非空的`buf.MultiBuffer`对象。

如果`w.transferType`的值为`protocol.TransferTypeStream`，那么这个函数会尝试使用`w.writeStream`函数来写入数据到缓冲区中。如果`w.transferType`的值不是`protocol.TransferTypeStream`，那么这个函数会尝试使用`w.writePacket`函数来写入数据到缓冲区中。

如果`mb`为空，函数会创建一个空的`buf.MultiBuffer`对象，然后调用`w.writeStream`函数来写入数据到缓冲区中，如果函数成功。如果`w.transferType`的值为`protocol.TransferTypeStream`，那么这个函数会调用`w.writeStream`函数来写入数据到缓冲区中，如果这个函数失败，那么返回一个非空的`buf.MultiBuffer`对象。如果`w.transferType`的值不是`protocol.TransferTypeStream`，那么这个函数会调用`w.writePacket`函数来写入数据到缓冲区中，如果这个函数失败，那么返回一个非空的`buf.MultiBuffer`对象。


```go
// WriteMultiBuffer implements buf.Writer.
func (w *AuthenticationWriter) WriteMultiBuffer(mb buf.MultiBuffer) error {
	if mb.IsEmpty() {
		eb, err := w.seal([]byte{})
		common.Must(err)
		return w.writer.WriteMultiBuffer(buf.MultiBuffer{eb})
	}

	if w.transferType == protocol.TransferTypeStream {
		return w.writeStream(mb)
	}

	return w.writePacket(mb)
}

```

# `common/crypto/auth_test.go`

该代码是一个用于测试AES密码学算法的CryptoTest包。它包含了一些AES密码学算法的实现，以及一些辅助函数和测试用例。以下是该代码的作用和主要功能：

1. 定义了一些AES密码学算法的帮助函数和测试用例。这些函数和测试用例可以用来测试该代码中实现的AES密码学算法。

2. 实现了一个AES密码学算法的高能见度测试用例。该算法使用了PR劈算法，并使用了随机种子，可以在不同的输入长度和不同的密钥长度下给出预期的性能。

3. 实现了AES密码学算法中的几种常见模式的测试用例。这些模式包括ECP、IDEA和SHA-256。

4. 实现了AES密码学算法中的几种常见情况的测试用例。这些情况包括密码长度为0、最大密码长度、以及不同输入长度下的密码。

5. 实现了随机数生成器和测试用例。这些函数可以生成随机数，并用于测试该代码中实现的AES密码学算法。

6. 实现了AES密码学算法中的多种测试用例。这些测试用例包括不同密钥长度下的加密、解密、以及随机数据的使用。

该代码的主要目的是提供一个可测试AES密码学算法的工具集，以便开发人员可以更轻松地测试和比较不同的AES密码学算法实现。


```go
package crypto_test

import (
	"bytes"
	"crypto/aes"
	"crypto/cipher"
	"crypto/rand"
	"io"
	"testing"

	"github.com/google/go-cmp/cmp"

	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	. "v2ray.com/core/common/crypto"
	"v2ray.com/core/common/protocol"
)

```

This is a function that sends a piece of data of fixed size to a remote endpoint using the Simple End-to-End (SEND) protocol. The function uses the Secure Real-Time Transport (SRT) protocol for SRT and the AES Electronic Signature Protocol (ESP) for encryption.

The function takes two arguments:

* `data`: a piece of data of fixed size to be sent.
* `recipient`: the destination endpoint for the data.

The function first generates a random nonce `iv` of size 12 bytes, and then it sends the data in a `MultiBuffer` named `mb` using the `MultiBuffer` name from the `buf.MultiBuffer` type. The data is then sent to the recipient endpoint using the `reader` and `writer` objects.

The function uses the `AEADAuthenticator` provided by the `AES` package to authenticate the data being sent. The `AEADAuthenticator` generates a static 12-byte nonce `iv` and a dynamic 128-bit payload that is then included in the `MultiBuffer` object. The `writer` then writes the `MultiBuffer` object to the `reader`.

The function uses the `GenerateStaticBytes` and `GenerateEmptyBytes` functions provided by the `Generate` package to generate a static 12-byte nonce and an empty buffer, respectively.

The function uses the `NewAuthenticationWriter` and `NewAuthenticationReader` functions provided by the `Authentication` package to create an `AuthenticationWriter` and `AuthenticationReader` object, respectively. These objects are used to write the data to the `writer` and read it from the `reader`, respectively.

The function uses the `cmp.Diff` function provided by the `diff` package to compare the `MultiBuffer` content with the `rawPayload` to ensure that they match.

The function returns nothing.


```go
func TestAuthenticationReaderWriter(t *testing.T) {
	key := make([]byte, 16)
	rand.Read(key)
	block, err := aes.NewCipher(key)
	common.Must(err)

	aead, err := cipher.NewGCM(block)
	common.Must(err)

	const payloadSize = 1024 * 80
	rawPayload := make([]byte, payloadSize)
	rand.Read(rawPayload)

	payload := buf.MergeBytes(nil, rawPayload)

	cache := bytes.NewBuffer(nil)
	iv := make([]byte, 12)
	rand.Read(iv)

	writer := NewAuthenticationWriter(&AEADAuthenticator{
		AEAD:                    aead,
		NonceGenerator:          GenerateStaticBytes(iv),
		AdditionalDataGenerator: GenerateEmptyBytes(),
	}, PlainChunkSizeParser{}, cache, protocol.TransferTypeStream, nil)

	common.Must(writer.WriteMultiBuffer(payload))
	if cache.Len() <= 1024*80 {
		t.Error("cache len: ", cache.Len())
	}
	common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{}))

	reader := NewAuthenticationReader(&AEADAuthenticator{
		AEAD:                    aead,
		NonceGenerator:          GenerateStaticBytes(iv),
		AdditionalDataGenerator: GenerateEmptyBytes(),
	}, PlainChunkSizeParser{}, cache, protocol.TransferTypeStream, nil)

	var mb buf.MultiBuffer

	for mb.Len() < payloadSize {
		mb2, err := reader.ReadMultiBuffer()
		common.Must(err)

		mb, _ = buf.MergeMulti(mb, mb2)
	}

	if mb.Len() != payloadSize {
		t.Error("mb len: ", mb.Len())
	}

	mbContent := make([]byte, payloadSize)
	buf.SplitBytes(mb, mbContent)
	if r := cmp.Diff(mbContent, rawPayload); r != "" {
		t.Error(r)
	}

	_, err = reader.ReadMultiBuffer()
	if err != io.EOF {
		t.Error("error: ", err)
	}
}

```

This appears to be a function testing the authentication mechanism of an anonymous file transfer protocol (AFT) client. The function uses the OpenSSL crtension library to handle the encryption and decryption.

The function takes a reader and a writer as inputs, and writes a payload to the writer. The payload is generated by the writer using the `GenerateStaticBytes` and `GenerateEmptyBytes` functions, which read data from a given piece of data (`iv`) and return it in chunks. The writer is using the `AEADAuthenticator` struct, which generates an authentication tag (`aad`) and a nonce (`nonceGenerator`).

The function also reads the payload from the reader, which is stored in a `MultiBuffer` object. The payload is then split into two chunks, one containing the data and the other containing the metadata (described by the `AdditionalDataGenerator`).

The function then reads the data from the reader and writes it to the writer. The data is written to the writer in chunks, with the metadata going in the middle of each chunk.

The function then reads the data from the reader again, this time starting from the middle of the last chunk. The function expects that the data at the end of the last chunk should not be an empty buffer.

Note that the function uses the `GenerateStaticBytes` and `GenerateEmptyBytes` functions to read data from the input `iv` and return it in chunks, but does not explain how this data is generated or what it contains.


```go
func TestAuthenticationReaderWriterPacket(t *testing.T) {
	key := make([]byte, 16)
	common.Must2(rand.Read(key))
	block, err := aes.NewCipher(key)
	common.Must(err)

	aead, err := cipher.NewGCM(block)
	common.Must(err)

	cache := buf.New()
	iv := make([]byte, 12)
	rand.Read(iv)

	writer := NewAuthenticationWriter(&AEADAuthenticator{
		AEAD:                    aead,
		NonceGenerator:          GenerateStaticBytes(iv),
		AdditionalDataGenerator: GenerateEmptyBytes(),
	}, PlainChunkSizeParser{}, cache, protocol.TransferTypePacket, nil)

	var payload buf.MultiBuffer
	pb1 := buf.New()
	pb1.Write([]byte("abcd"))
	payload = append(payload, pb1)

	pb2 := buf.New()
	pb2.Write([]byte("efgh"))
	payload = append(payload, pb2)

	common.Must(writer.WriteMultiBuffer(payload))
	if cache.Len() == 0 {
		t.Error("cache len: ", cache.Len())
	}

	common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{}))

	reader := NewAuthenticationReader(&AEADAuthenticator{
		AEAD:                    aead,
		NonceGenerator:          GenerateStaticBytes(iv),
		AdditionalDataGenerator: GenerateEmptyBytes(),
	}, PlainChunkSizeParser{}, cache, protocol.TransferTypePacket, nil)

	mb, err := reader.ReadMultiBuffer()
	common.Must(err)

	mb, b1 := buf.SplitFirst(mb)
	if b1.String() != "abcd" {
		t.Error("b1: ", b1.String())
	}

	mb, b2 := buf.SplitFirst(mb)
	if b2.String() != "efgh" {
		t.Error("b2: ", b2.String())
	}

	if !mb.IsEmpty() {
		t.Error("not empty")
	}

	_, err = reader.ReadMultiBuffer()
	if err != io.EOF {
		t.Error("error: ", err)
	}
}

```

# `common/crypto/benchmark_test.go`

这段代码是一个用于测试加密算法的框架，它包含一个名为`benchmarkStream`的函数。

函数接受两个参数：`b`表示测试计数器，`c`是一个加密套件中的加密函数。

函数内部首先创建一个大小为`benchSize`的缓冲区`input`和一个大小为`benchSize`的缓冲区`output`，用于存储加密前和加密后的数据。

然后，函数开始一个循环，每次循环都会调用`c.XORKeyStream`函数对`input`和`output`进行异或操作，并将结果复送到`output`缓冲区中。

最后，函数使用`testing.B`包中的`b.SetBytes`函数来设置计数器`b`的值到`benchSize`，然后开始运行测试。在每次循环结束后，函数使用`b.ResetTimer`函数来重置计时器，以便下一次循环可以重新开始。


```go
package crypto_test

import (
	"crypto/cipher"
	"testing"

	. "v2ray.com/core/common/crypto"
)

const benchSize = 1024 * 1024

func benchmarkStream(b *testing.B, c cipher.Stream) {
	b.SetBytes(benchSize)
	input := make([]byte, benchSize)
	output := make([]byte, benchSize)
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		c.XORKeyStream(output, input)
	}
}

```

这段代码定义了三个名为"BenchmarkChaCha20"、"BenchmarkChaCha20IETF"和"BenchmarkAESEncryption"的函数，它们都接受一个名为"testing.B"的参数。每个函数内部都创建了一个名为"ChaCha20Stream"的类的实例，并传入一个字节数组作为参数，以及一个字节数组作为非ce参数。

这些函数都实现了"io.Reader"接口的"BenchmarkStream"方法，该方法用于在测试中生成随机数据，并输出这些数据。通过调用"BenchmarkStream"方法，可以测试不同的数据生成和处理函数，例如在这三个函数中，通过传入不同的非ce和密钥参数，可以测试如何生成非ce和密钥，以及如何将数据发送到"ChaCha20Stream"实例中进行加密和解密。


```go
func BenchmarkChaCha20(b *testing.B) {
	key := make([]byte, 32)
	nonce := make([]byte, 8)
	c := NewChaCha20Stream(key, nonce)
	benchmarkStream(b, c)
}

func BenchmarkChaCha20IETF(b *testing.B) {
	key := make([]byte, 32)
	nonce := make([]byte, 12)
	c := NewChaCha20Stream(key, nonce)
	benchmarkStream(b, c)
}

func BenchmarkAESEncryption(b *testing.B) {
	key := make([]byte, 32)
	iv := make([]byte, 16)
	c := NewAesEncryptionStream(key, iv)

	benchmarkStream(b, c)
}

```

这段代码定义了一个名为 "BenchmarkAESDecryption" 的函数，它的作用是测试 AES 数据加密和解密功能。函数接收一个字节数组（即 "b" 参数）作为参数，然后执行一系列测试用例。

具体来说，这段代码执行以下操作：

1. 生成一个 32 字节长的随机字节数组（即 "key" 参数）。
2. 生成一个 16 字节长的随机字节数组（即 "iv" 参数）。
3. 创建一个名为 "c" 的 AES 数据加密流实例，使用上面两个随机生成的字节数组作为参数。
4. 使用 "c" 执行数据加密和解密操作，将测试用例的输入数据作为输入，并输出结果。

这段代码的目的是在给定随机数据的情况下，测试 AES 数据加密和解密功能是否正常工作。


```go
func BenchmarkAESDecryption(b *testing.B) {
	key := make([]byte, 32)
	iv := make([]byte, 16)
	c := NewAesDecryptionStream(key, iv)

	benchmarkStream(b, c)
}

```

# `common/crypto/chacha20.go`

这段代码定义了一个名为“crypto”的包，其中包含了一些用于ChaCha20算法的函数和变量。

首先，引入了“crypto/cipher”和“v2ray.com/core/common/crypto/internal”两个包，可能是用于与ChaCha20算法有关的库和函数。

然后，定义了一个名为“NewChaCha20Stream”的函数，该函数接受两个字节切片（即关键点和IV）。函数创建一个新的基于给定密钥和IV的ChaCha20加密/解密流。函数的第一个参数是关键点字节数组，第二个参数是IV字节数组。函数的第三个参数是加密/解密模式，这里是20字节。

函数内部使用“internal.NewChaCha20Stream”函数创建一个新的ChaCha20流，并设置关键点为传入的密钥字节数组，IV设置为传入的IV字节数组。然后，函数返回一个新的ChaCha20流实例。

总结一下，这段代码定义了一个名为“NewChaCha20Stream”的函数，用于创建一个新的基于给定密钥和IV的ChaCha20加密/解密流。


```go
package crypto

import (
	"crypto/cipher"

	"v2ray.com/core/common/crypto/internal"
)

// NewChaCha20Stream creates a new Chacha20 encryption/descryption stream based on give key and IV.
// Caller must ensure the length of key is 32 bytes, and length of IV is either 8 or 12 bytes.
func NewChaCha20Stream(key []byte, iv []byte) cipher.Stream {
	return internal.NewChaCha20Stream(key, iv, 20)
}

```

# `common/crypto/chacha20_test.go`

这段代码是一个名为 "crypto_test" 的 package，其中包含了一些用于测试加密/解密算法的函数和变量。

1. "crypto/rand"：这是一个标准库 "crypto" 中定义的函数，用于生成随机数。
2. "encoding/hex"：这是一个标准库 "encoding" 中定义的函数，用于将字符串转换为 hex 编码的字节数组。
3. "testing"：这是一个标准库 "testing" 中定义的函数，用于提供测试功能。
4. "github.com/google/go-cmp/cmp"：这是一个名为 "google/go-cmp" 的库，用于测试 "testing" 中的比较功能。
5. "v2ray.com/core/common"：这个库 "v2ray.com/core/common" 中定义了一些变量和函数，用于与 v2ray 客户端进行交互。
6. "github.com/v2ray/v2ray-go-client"：这个库 "github.com/v2ray/v2ray-go-client" 中定义了一些变量和函数，用于与 v2ray 客户端进行交互。


```go
package crypto_test

import (
	"crypto/rand"
	"encoding/hex"
	"testing"

	"github.com/google/go-cmp/cmp"

	"v2ray.com/core/common"
	. "v2ray.com/core/common/crypto"
)

func mustDecodeHex(s string) []byte {
	b, err := hex.DecodeString(s)
	common.Must(err)
	return b
}

```

This appears to be a function written in Go, which tests the encryption strength of the ChaCha20 algorithm. The ChaCha20 algorithm is a widely-used, high-performance cryptographic hashing algorithm that is designed to be more secure than its前身， ChaCha128.

The function takes a single argument, which is a ChaCha20 preset key. The key is generated using the `crypto/rand.仍然是crypto/rand.generate()`函数， and then passed to the `NewChaCha20Stream()` function, which returns a stream that can be used to perform the ChaCha20 algorithm.

The function then reads in the output from the ChaCha20 stream, and compares it to the actual output to ensure that they match. If they do not match, the function prints out the difference and then raises an error.

The function also creates a NewChaCha20Stream from the preset key, and then iterates through all possible output values for the ChaCha20 algorithm, using the `for` loop to check all possible combinations of the input and output values.


```go
func TestChaCha20Stream(t *testing.T) {
	var cases = []struct {
		key    []byte
		iv     []byte
		output []byte
	}{
		{
			key: mustDecodeHex("0000000000000000000000000000000000000000000000000000000000000000"),
			iv:  mustDecodeHex("0000000000000000"),
			output: mustDecodeHex("76b8e0ada0f13d90405d6ae55386bd28bdd219b8a08ded1aa836efcc8b770dc7" +
				"da41597c5157488d7724e03fb8d84a376a43b8f41518a11cc387b669b2ee6586" +
				"9f07e7be5551387a98ba977c732d080dcb0f29a048e3656912c6533e32ee7aed" +
				"29b721769ce64e43d57133b074d839d531ed1f28510afb45ace10a1f4b794d6f"),
		},
		{
			key: mustDecodeHex("5555555555555555555555555555555555555555555555555555555555555555"),
			iv:  mustDecodeHex("5555555555555555"),
			output: mustDecodeHex("bea9411aa453c5434a5ae8c92862f564396855a9ea6e22d6d3b50ae1b3663311" +
				"a4a3606c671d605ce16c3aece8e61ea145c59775017bee2fa6f88afc758069f7" +
				"e0b8f676e644216f4d2a3422d7fa36c6c4931aca950e9da42788e6d0b6d1cd83" +
				"8ef652e97b145b14871eae6c6804c7004db5ac2fce4c68c726d004b10fcaba86"),
		},
		{
			key:    mustDecodeHex("0000000000000000000000000000000000000000000000000000000000000000"),
			iv:     mustDecodeHex("000000000000000000000000"),
			output: mustDecodeHex("76b8e0ada0f13d90405d6ae55386bd28bdd219b8a08ded1aa836efcc8b770dc7da41597c5157488d7724e03fb8d84a376a43b8f41518a11cc387b669b2ee6586"),
		},
	}
	for _, c := range cases {
		s := NewChaCha20Stream(c.key, c.iv)
		input := make([]byte, len(c.output))
		actualOutout := make([]byte, len(c.output))
		s.XORKeyStream(actualOutout, input)
		if r := cmp.Diff(c.output, actualOutout); r != "" {
			t.Fatal(r)
		}
	}
}

```

该代码是一个名为 "TestChaCha20Decoding" 的函数，用于测试 "ChaCha20Decoding" 函数的正确性。函数会生成一个随机长度的字节数组 "key"，一个随机长度的字节数组 "iv"，以及一个随机长度的字节数组 "stream"。然后，使用 NewChaCha20Stream 函数创建一个 "ChaCha20Stream" 对象，将 "key" 和 "iv" 作为其输入，将 "stream" 作为其输入，生成一个长度为 1024 的随机字节数组 "payload"。接下来，使用 stream.XORKeyStream 函数对 "x" 和 "payload" 进行异或操作，然后使用 stream2.XORKeyStream 函数对 "x" 和 "stream2.XORKeyStream" 返回的 "x" 进行异或操作。最后，比较 "x" 和 "payload" 之间的差异，如果两个字节数组之间的差异存在，则函数会输出该差异，并因为在运行时打印错误消息。


```go
func TestChaCha20Decoding(t *testing.T) {
	key := make([]byte, 32)
	common.Must2(rand.Read(key))
	iv := make([]byte, 8)
	common.Must2(rand.Read(iv))
	stream := NewChaCha20Stream(key, iv)

	payload := make([]byte, 1024)
	common.Must2(rand.Read(payload))

	x := make([]byte, len(payload))
	stream.XORKeyStream(x, payload)

	stream2 := NewChaCha20Stream(key, iv)
	stream2.XORKeyStream(x, x)
	if r := cmp.Diff(x, payload); r != "" {
		t.Fatal(r)
	}
}

```

# `common/crypto/chunk.go`

这段代码定义了一个名为“crypto”的包，它包含了一个名为“ChunkSizeDecoder”的接口。这个接口的一个实现是“ChunkSizeDecoderIo”，它使用一个名为“ChunkSizeDecoder”的内部类型来处理字节数组。

具体来说，这段代码导入了两个库：

1. “encoding/binary”库，它提供了对字节数组的编码和解码功能。
2. “v2ray.com/core/common”库，它提供了与V2Ray客户端进行交互的接口。

然后，定义了一个名为“ChunkSizeDecoder”的内部类型，它包含两个方法：

1. “SizeBytes”方法，它使用“encoding/binary”库的“binary.Decode”函数将字节数组转换为int32类型的整数。
2. “Decode”方法，它使用“ChunkSizeDecoderIo”接口的“Decode”函数将字节数组解码为两个uint16类型的值，并返回可能的错误。

最后，在导入了“ChunkSizeDecoderIo”接口的“ChunkSizeDecoder”类型的实例之后，将所有导入的库都赋值给变量“decoder”，这样就可以在需要时使用它们了。


```go
package crypto

import (
	"encoding/binary"
	"io"

	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
)

// ChunkSizeDecoder is a utility class to decode size value from bytes.
type ChunkSizeDecoder interface {
	SizeBytes() int32
	Decode([]byte) (uint16, error)
}

```

这段代码定义了一个名为 `ChunkSizeEncoder` 的类，该类实现了一个将size值编码成字节的方法。

定义了一个名为 `ChunkSizeEncoder` 的接口 `ChunkSizeEncoder`，该接口包含两个方法：`SizeBytes` 和 `Encode`。

定义了一个名为 `PaddingLengthGenerator` 的接口 `PaddingLengthGenerator`，该接口包含两个方法：`MaxPaddingLen` 和 `NextPaddingLen`。

定义了一个名为 `PlainChunkSizeParser` 的 struct 类型 `PlainChunkSizeParser`，该类型实现了一个 `ChunkSizeEncoder`，并包含一个 `SizeBytes` 方法。

该 `PlainChunkSizeParser` 的 `SizeBytes` 方法实现了一个 `ChunkSizeEncoder` 的 `SizeBytes` 方法，将输入的 `uint16` 类型的参数转换成字节并返回。


```go
// ChunkSizeEncoder is a utility class to encode size value into bytes.
type ChunkSizeEncoder interface {
	SizeBytes() int32
	Encode(uint16, []byte) []byte
}

type PaddingLengthGenerator interface {
	MaxPaddingLen() uint16
	NextPaddingLen() uint16
}

type PlainChunkSizeParser struct{}

func (PlainChunkSizeParser) SizeBytes() int32 {
	return 2
}

```

这段代码定义了一个名为AEADChunkSizeParser的 struct类型，以及两个名为func的函数，用于对AEAD数据进行编码和解码。

具体来说，这两个函数的第一个参数PlainChunkSizeParser是一个名为PlainChunkSizeParser的函数，它接收一个size和一个字节数组b，将size转换为字节并返回以字节数组b开始的连续字节。这个函数的作用是将输入的size转换成字节，然后返回以字节数组b开始的连续字节。

这两个函数的第二个参数都是接收一个字节数组b，并返回一个uint16类型的值和一个错误类型的值。具体来说，第一个函数Encode接收一个字节数组b，将b字节数组中的第一个字节设为输入的size，然后返回以字节数组b开始的连续字节。第二个函数Decode接收一个字节数组b，将b字节数组中的第一个字节设为输入的size，然后返回传入的AEADAuthenticator对象的Overhead()方法计算出的uint16类型值和错误类型的值。

AEADChunkSizeParser是一个AEAD数据编码器，其中AEADAuthenticator是一个AEAD认证器，用于对AEAD数据进行编码和解码。函数SizeBytes返回AEADChunkSizeParser的输入和输出字节数。函数使用PlainChunkSizeParser将输入的AEAD数据编码成字节，并使用Decode函数解码AEAD数据，并返回AEADAuthenticator对象的Overhead()计算出的字节数。


```go
func (PlainChunkSizeParser) Encode(size uint16, b []byte) []byte {
	binary.BigEndian.PutUint16(b, size)
	return b[:2]
}

func (PlainChunkSizeParser) Decode(b []byte) (uint16, error) {
	return binary.BigEndian.Uint16(b), nil
}

type AEADChunkSizeParser struct {
	Auth *AEADAuthenticator
}

func (p *AEADChunkSizeParser) SizeBytes() int32 {
	return 2 + int32(p.Auth.Overhead())
}

```

这两函数的作用如下：

1. `func (p *AEADChunkSizeParser) Encode(size uint16, b []byte) []byte` 函数的输入参数为 `size` 和一个字节切片 `b`。它将 `size` 转换为字节，然后将 `size` 以小数位 4 字节为单位进行编码，并将编码后的结果存储到一个字节切片 `b` 中。

2. `func (p *AEADChunkSizeParser) Decode(b []byte) (uint16, error)` 函数的输入参数为 `b` 字节切片。它将 `b` 中的字节读取并传递给 `AEADChunkSizeParser.Decode` 函数，解码结果存储为 `uint16` 类型，并将解码后的结果返回。如果解码过程中出现错误，函数将返回一个非 `nil` 的错误对象。


```go
func (p *AEADChunkSizeParser) Encode(size uint16, b []byte) []byte {
	binary.BigEndian.PutUint16(b, size-uint16(p.Auth.Overhead()))
	b, err := p.Auth.Seal(b[:0], b[:2])
	common.Must(err)
	return b
}

func (p *AEADChunkSizeParser) Decode(b []byte) (uint16, error) {
	b, err := p.Auth.Open(b[:0], b)
	if err != nil {
		return 0, err
	}
	return binary.BigEndian.Uint16(b) + uint16(p.Auth.Overhead()), nil
}

```

该代码定义了一个名为 `ChunkStreamReader` 的结构体，用于处理分块流数据。该结构体包含两个主要成员变量：`sizeDecoder` 和 `reader`，分别用于处理数据大小和数据读取的上下文。

`sizeDecoder` 是一个 `ChunkSizeDecoder` 类型的变量，用于处理数据分块的大小和数量。`reader` 是一个 `io.Reader` 类型的变量，用于读取数据。

该 `ChunkStreamReader` 还包含两个辅助成员函数：`NewChunkStreamReader` 和 `NewChunkStreamReaderWithChunkCount`。

`NewChunkStreamReader` 函数将 `sizeDecoder` 和 `reader` 作为参数，返回一个包含 `ChunkStreamReader` 实例的新结构体。

`NewChunkStreamReaderWithChunkCount` 函数将 `sizeDecoder` 和 `reader` 作为参数，返回一个包含 `ChunkStreamReader` 实例以及最大数据块数量的新结构体。

该 `ChunkStreamReader` 的主要工作是读取数据，并对读取到的数据进行分块处理。它通过 `sizeDecoder` 来确定数据分块的大小和数量，并通过 `reader` 来读取数据。然后，它将数据块缓存到 `buffer` 数组中，并根据 `maxNumChunk` 变量来确定最大数据块数量。当缓存满了之后，它将使用 `reader` 提供的数据读取器读取新的数据，并将新读取到的数据与缓存中的数据进行合并，如果缓存中所有的数据块加上新的数据长度之和大于设定的最大数据块数量，它将删除最早的数据块。


```go
type ChunkStreamReader struct {
	sizeDecoder ChunkSizeDecoder
	reader      *buf.BufferedReader

	buffer       []byte
	leftOverSize int32
	maxNumChunk  uint32
	numChunk     uint32
}

func NewChunkStreamReader(sizeDecoder ChunkSizeDecoder, reader io.Reader) *ChunkStreamReader {
	return NewChunkStreamReaderWithChunkCount(sizeDecoder, reader, 0)
}

func NewChunkStreamReaderWithChunkCount(sizeDecoder ChunkSizeDecoder, reader io.Reader, maxNumChunk uint32) *ChunkStreamReader {
	r := &ChunkStreamReader{
		sizeDecoder: sizeDecoder,
		buffer:      make([]byte, sizeDecoder.SizeBytes()),
		maxNumChunk: maxNumChunk,
	}
	if breader, ok := reader.(*buf.BufferedReader); ok {
		r.reader = breader
	} else {
		r.reader = &buf.BufferedReader{Reader: buf.NewReader(reader)}
	}

	return r
}

```

这两段代码都是PartitionedStream中的ChunkStreamReader相关函数。

第一段代码定义了一个名为func的函数，接收一个ChunkStreamReader类型的参数r，并返回读取到的数据块大小(uint16)以及错误(error)。函数在执行时，首先检查是否已经 read 完全部数据，如果是，则回滚并返回0 和错误。然后尝试从 io 设备读取数据并解码到ChunkStreamReader的buffer，如果出错，则返回0 和错误。

第二段代码定义了一个名为func的函数，与第一段代码类似，接收一个ChunkStreamReader类型的参数r，并返回读取到的数据块大小(uint16)以及错误(error)。函数在执行时，首先检查是否已经 read 完全部数据，如果是，则回滚并返回0 和错误。然后尝试从 io 设备读取数据并解码到ChunkStreamReader的buffer，如果出错，则返回0 和错误。在本函数中，还增加了一个名为r.maxNumChunk的变量，用来记录已经超过的最大数据块数。当已经读取的数据块数达到了 maxNumChunk，则会继续向读取更多数据块，直到达到 io 设备读取限制或者遇到 end of file 等错误。


```go
func (r *ChunkStreamReader) readSize() (uint16, error) {
	if _, err := io.ReadFull(r.reader, r.buffer); err != nil {
		return 0, err
	}
	return r.sizeDecoder.Decode(r.buffer)
}

func (r *ChunkStreamReader) ReadMultiBuffer() (buf.MultiBuffer, error) {
	size := r.leftOverSize
	if size == 0 {
		r.numChunk++
		if r.maxNumChunk > 0 && r.numChunk > r.maxNumChunk {
			return nil, io.EOF
		}
		nextSize, err := r.readSize()
		if err != nil {
			return nil, err
		}
		if nextSize == 0 {
			return nil, io.EOF
		}
		size = int32(nextSize)
	}
	r.leftOverSize = size

	mb, err := r.reader.ReadAtMost(size)
	if !mb.IsEmpty() {
		r.leftOverSize -= mb.Len()
		return mb, nil
	}
	return nil, err
}

```

该代码定义了一个名为 `ChunkStreamWriter` 的 struct，它包含一个 `sizeEncoder` 和一个 `writer` 字段。

`sizeEncoder` 是一个名为 `ChunkSizeEncoder` 的类型，它可能实现了一些字节数组大小的编码逻辑。`writer` 字段是一个 `io.Writer` 类型的字段，它代表了一个可以写入数据到字节缓冲区的目标接口。

该 struct 的主要作用是实现一个可扩展的、多缓冲的输入输出流。它允许将输入数据缓冲区 `mb` 中的数据，通过 `sizeEncoder` 对数据进行编码，然后将其写入一个目标 `writer` 接口。在每次写入数据之前，它会先将数据拆分成多个较小的缓冲区，并使用 `sizeEncoder` 对每个缓冲区进行编码。如果输入数据是空字符串，则退出数据读取循环，保证不会读入任何数据。


```go
type ChunkStreamWriter struct {
	sizeEncoder ChunkSizeEncoder
	writer      buf.Writer
}

func NewChunkStreamWriter(sizeEncoder ChunkSizeEncoder, writer io.Writer) *ChunkStreamWriter {
	return &ChunkStreamWriter{
		sizeEncoder: sizeEncoder,
		writer:      buf.NewWriter(writer),
	}
}

func (w *ChunkStreamWriter) WriteMultiBuffer(mb buf.MultiBuffer) error {
	const sliceSize = 8192
	mbLen := mb.Len()
	mb2Write := make(buf.MultiBuffer, 0, mbLen/buf.Size+mbLen/sliceSize+2)

	for {
		mb2, slice := buf.SplitSize(mb, sliceSize)
		mb = mb2

		b := buf.New()
		w.sizeEncoder.Encode(uint16(slice.Len()), b.Extend(w.sizeEncoder.SizeBytes()))
		mb2Write = append(mb2Write, b)
		mb2Write = append(mb2Write, slice...)

		if mb.IsEmpty() {
			break
		}
	}

	return w.writer.WriteMultiBuffer(mb2Write)
}

```