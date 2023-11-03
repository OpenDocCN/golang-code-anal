# v2ray-core源码解析 21

# `common/crypto/chunk_test.go`

这段代码的作用是测试一个名为 "ChunkStreamIO" 的名为 "crypto_test" 的 package 中的功能。

它实现了 chunk 存储器（chunk stream）I/O 操作，并在这些操作中使用了 "v2ray.com/core/common" 包中的 "bytes" 和 "io" 包。

具体来说，这段代码创建了一个 8192 字节的缓冲区（cache），并使用 "NewChunkStreamWriter" 和 "NewChunkStreamReader" 函数来创建一个 chunk stream  writer 和读者。

在 `TestChunkStreamIO` 函数中，我们分别向 writer 和 reader 写了两个字符串 "abcd" 和 "efg"，然后向 writer 写了两个空字符串。

我们还创建了一个 "MultiBuffer" 类型的缓冲区 `b`，并向 writer 写了这三个字符串。

在 `reader` 的 `ReadMultiBuffer` 函数中，我们读取了多个字符串，并打印了这些字符串的值。我们还测试了 reader 是否能够正确地读取一个空字符串。

最后，在 `cache.Len()` 函数中，我们检查了缓冲区中的字符串是否足够长，如果不够长，就会输出一个错误。


```go
package crypto_test

import (
	"bytes"
	"io"
	"testing"

	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	. "v2ray.com/core/common/crypto"
)

func TestChunkStreamIO(t *testing.T) {
	cache := bytes.NewBuffer(make([]byte, 0, 8192))

	writer := NewChunkStreamWriter(PlainChunkSizeParser{}, cache)
	reader := NewChunkStreamReader(PlainChunkSizeParser{}, cache)

	b := buf.New()
	b.WriteString("abcd")
	common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{b}))

	b = buf.New()
	b.WriteString("efg")
	common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{b}))

	common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{}))

	if cache.Len() != 13 {
		t.Fatalf("Cache length is %d, want 13", cache.Len())
	}

	mb, err := reader.ReadMultiBuffer()
	common.Must(err)

	if s := mb.String(); s != "abcd" {
		t.Error("content: ", s)
	}

	mb, err = reader.ReadMultiBuffer()
	common.Must(err)

	if s := mb.String(); s != "efg" {
		t.Error("content: ", s)
	}

	_, err = reader.ReadMultiBuffer()
	if err != io.EOF {
		t.Error("error: ", err)
	}
}

```

# `common/crypto/crypto.go`

这段代码是一个 Go 语言中的第三方库，名为 "crypto"，它提供了用于 V2Ray 的 common crypto libraries。

首先，它导入了 "v2ray.com/core/common/crypto" 包，这是一个常用的加密库，用于与 V2Ray 一起使用。

接下来，它定义了一个名为 "errordev" 的函数，它导入了 "v2ray.com/core/common/errors" 包，用于错误处理。这个函数没有返回值，说明它是一个默认的错误函数。

最后，它通过调用 "generate" 函数来自动生成一个 "errorgen" 的函数，这个函数会在编译时生成一个可以输出 "errorgen" 的函数声明。


```go
// Package crypto provides common crypto libraries for V2Ray.
package crypto // import "v2ray.com/core/common/crypto"

//go:generate go run v2ray.com/core/common/errors/errorgen

```

# `common/crypto/errors.generated.go`

这段代码是一个名为 "crypto" 的包，其中包含了一些定义、函数和类型声明。下面是对这些部分的解释：

1. 定义了一个名为 "errPathObjHolder" 的类型，该类型有一个无参构造函数和一个名为 "newError" 的函数。函数接收多个参数，可以是任意类型的 value，包括 int、string、bool、byte、interface{}，并返回一个 errors.Error 类型的对象。这些对象有一个挂载到 errPathObjHolder 类型的路径偏移量，因此可以用来设置错误的位置。

2. 引入了 "v2ray.com/core/common/errors" 包，该包可能包含了用于错误处理和转发的常用功能。

3. 定义了一个名为 "errPathObjHolder" 的类型，该类型包含一个空路径偏移量。这个类型被命名为 "errPathObjHolder"，因此可以猜测它是一个具有 "err" 和 "path/obj" 两部分组成的关键字类型。

4. 定义了一个名为 "newError" 的函数，该函数接收多个参数，并返回一个带有 errPathObjHolder 类型的对象。函数的作用是在不输出参数的情况下创建一个错误对象，这个对象使用了values...参数传递，因此可以假设这些参数是错误的类型。

5. 在 "newError" 函数中，使用了 `errors.New` 函数来创建一个新的错误对象。该函数有一个接收者类型为 "error" 的类型参数，以及一个字符串参数，用于指定错误消息。最后，使用 `WithPathObj` 方法来设置错误对象的路径偏移量，该方法将 `errPathObjHolder` 类型的对象与错误消息和错误对象的路径偏移量连接起来。

综上所述，这段代码定义了一个名为 "errPathObjHolder" 的类型，以及一个名为 "newError" 的函数。函数使用 `errors.New` 和 `WithPathObj` 方法来创建一个新的错误对象，该对象包含一个错误消息和一个路径偏移量，该偏移量指向一个名为 "errPathObjHolder" 的类型对象。


```go
package crypto

import "v2ray.com/core/common/errors"

type errPathObjHolder struct{}

func newError(values ...interface{}) *errors.Error {
	return errors.New(values...).WithPathObj(errPathObjHolder{})
}

```

# `common/crypto/io.go`

这段代码定义了一个名为 `CryptionReader` 的结构体，它包含一个名为 `stream` 的协程读写流和一个名为 `reader` 的 I/O 读写流。

`NewCryptionReader` 函数接受两个参数，一个是 `stream` 一个是 `reader`。它返回一个指向 `CryptionReader` 类型的实例，初始化时会将 `stream` 和 `reader` 都指向默认的上下文，即 `cipher.NewCipher()` 和 `io.NullReader`。

`CipherReader` 的作用是接受一个加密协程和一个 I/O 读写流，并将它们连接起来，实现加密和解密的功能。具体来说，`CipherReader` 通过 `cipher.NewCipher()` 创建一个加密上下文，然后使用 `io.ReadAll` 读取输入数据，并使用 `cipher.Gather` 函数将输入数据包装到一个缓冲区 `buf` 中，最后发送出去。接收端，它从 `io.ReadAll` 读取数据，然后使用 `cipher.NewCipher()` 创建一个解密上下文，将 `buf` 中的数据解码为明文，并返回解密的字面。

整个 `package crypto` 是导入了一些与加密相关的库，包括 `crypto/cipher` 和 `v2ray.com/core/common/buf`。


```go
package crypto

import (
	"crypto/cipher"
	"io"

	"v2ray.com/core/common/buf"
)

type CryptionReader struct {
	stream cipher.Stream
	reader io.Reader
}

func NewCryptionReader(stream cipher.Stream, reader io.Reader) *CryptionReader {
	return &CryptionReader{
		stream: stream,
		reader: reader,
	}
}

```

该函数`func (r *CryptionReader) Read(data []byte) (int, error)`接收一个加密读取器`CryptionReader`类型的参数`r`，并返回从`r`的读者中读取的数据字节数和可能的错误。

函数首先使用`r.reader.Read`方法从读取器中读取数据字节数`nBytes`，如果`nBytes`大于零，则执行以下操作：

1. 在输入数据字节数`data`中执行XOR操作，对`data`字节数中的所有元素进行异或操作，得到一个新的字节数`output`。
2. 返回读取器读取的数据字节数`nBytes`和XOR操作的结果。

如果XOR操作出现错误，则在返回时会增加一个`error`。

该函数的作用是读取加密数据并将其存储在缓冲区中，以便后续的写入操作。


```go
func (r *CryptionReader) Read(data []byte) (int, error) {
	nBytes, err := r.reader.Read(data)
	if nBytes > 0 {
		r.stream.XORKeyStream(data[:nBytes], data[:nBytes])
	}
	return nBytes, err
}

var (
	_ buf.Writer = (*CryptionWriter)(nil)
)

type CryptionWriter struct {
	stream    cipher.Stream
	writer    io.Writer
	bufWriter buf.Writer
}

```

这段代码定义了一个名为NewCryptionWriter的函数，它创建了一个新的CryptionsWriter。

函数接受两个参数，第一个参数是一个流(stream)，第二个参数是一个写入器(Writer)。函数返回一个指向CryptionWriter的引用，初始化时会将第一个参数的流赋给变量stream，将第二个参数的写入器赋给变量writer。

函数的另一个实现io.Writer.Write()函数，该函数将给定的数据写入到缓冲区中，并返回写入的数据字节数和错误。首先，函数将数据通过与流中的数据进行异或操作来更新数据。然后，函数通过调用buf.WriteAllBytes()函数将数据写入到指定的写入器中。如果错误发生，函数返回0并错误。最后，函数返回数据的长度。


```go
// NewCryptionWriter creates a new CryptionWriter.
func NewCryptionWriter(stream cipher.Stream, writer io.Writer) *CryptionWriter {
	return &CryptionWriter{
		stream:    stream,
		writer:    writer,
		bufWriter: buf.NewWriter(writer),
	}
}

// Write implements io.Writer.Write().
func (w *CryptionWriter) Write(data []byte) (int, error) {
	w.stream.XORKeyStream(data, data)

	if err := buf.WriteAllBytes(w.writer, data); err != nil {
		return 0, err
	}
	return len(data), nil
}

```

这段代码定义了一个名为 `WriteMultiBuffer` 的函数，它接受一个名为 `mb` 的 `MultiBuffer` 参数，代表一个 多缓冲的字节数组。这个函数的作用是将 `mb` 中的所有字节通过输出流 `w.stream` 进行异或操作，并将结果写入 `mb` 中。

具体实现可以分为两个步骤：

1. 遍历 `mb` 中的所有字节，并将其字节与相邻的字节字节异或在一起。
2. 将处理过的字节重新写入 `mb` 中的对应位置，并返回结果。

这个函数是 `buf.Writer` 的子类，因此可以调用 `w.bufWriter.WriteMultiBuffer` 函数来同时写入多个缓冲区。


```go
// WriteMultiBuffer implements buf.Writer.
func (w *CryptionWriter) WriteMultiBuffer(mb buf.MultiBuffer) error {
	for _, b := range mb {
		w.stream.XORKeyStream(b.Bytes(), b.Bytes())
	}

	return w.bufWriter.WriteMultiBuffer(mb)
}

```

# `common/crypto/internal/chacha.go`

这段代码定义了一个名为“internal”的包，其中包含一个名为“ChaCha20Stream”的结构体。

该结构体定义了一个“ChaCha20Stream”类型的变量，该变量包含一个长度为4的“wordSize”字段，表示每个单词的大小；一个长度为16的“stateSize”字段，表示每个状态字的大小，在16个32位单词中；一个长度为blockSize的“block”字段，表示一个包含64个字节的状态块；以及一个名为“offset”的整数字段，表示在使用状态块时，从块的起始位置开始的字节数。

该结构体还定义了一个名为“ rounds”的整数字段，表示进行多少轮ChaCha20加密。

最后，该结构体还定义了一个名为“generate”的函数，该函数会生成一个长度为“chacha_core_gen.go”中定义的函数的源代码。


```go
package internal

//go:generate go run chacha_core_gen.go

import (
	"encoding/binary"
)

const (
	wordSize  = 4                    // the size of ChaCha20's words
	stateSize = 16                   // the size of ChaCha20's state, in words
	blockSize = stateSize * wordSize // the size of ChaCha20's block, in bytes
)

type ChaCha20Stream struct {
	state  [stateSize]uint32 // the state as an array of 16 32-bit words
	block  [blockSize]byte   // the keystream as an array of 64 bytes
	offset int               // the offset of used bytes in block
	rounds int
}

```

这段代码定义了一个名为`NewChaCha20Stream`的函数，它接受一个字节数组`key`，一个字节数组`nonce`，和一个整数`rounds`作为参数。函数返回一个指向`ChaCha20Stream`类型的`*ChaCha20Stream`类型的指针。

函数内部首先创建一个空的`ChaCha20Stream`类型的`s`变量，然后执行一些状态转换操作，将`key`字节数组转换为256位密钥的初始化状态，并将`nonce`字节数组转换为与`key`相同长度和类型的初始化状态。

接下来，函数循环执行一次，将`key`中的4个字节解码为256位密钥的初始化状态，将`nonce`中当前长度与`key`相同的情况解码为需要的状态，否则产生`bad nonce length`的错误。

最后，函数根据`rounds`的值设置`s.rounds`为所需的`rounds`个周期，然后调用一个名为`ChaCha20Block`的函数，将`s.state`和`s.block`作为参数，执行所需的`rounds`个周期。最后，函数返回`s`指向的`ChaCha20Stream`类型的指针。


```go
func NewChaCha20Stream(key []byte, nonce []byte, rounds int) *ChaCha20Stream {
	s := new(ChaCha20Stream)
	// the magic constants for 256-bit keys
	s.state[0] = 0x61707865
	s.state[1] = 0x3320646e
	s.state[2] = 0x79622d32
	s.state[3] = 0x6b206574

	for i := 0; i < 8; i++ {
		s.state[i+4] = binary.LittleEndian.Uint32(key[i*4 : i*4+4])
	}

	switch len(nonce) {
	case 8:
		s.state[14] = binary.LittleEndian.Uint32(nonce[0:])
		s.state[15] = binary.LittleEndian.Uint32(nonce[4:])
	case 12:
		s.state[13] = binary.LittleEndian.Uint32(nonce[0:4])
		s.state[14] = binary.LittleEndian.Uint32(nonce[4:8])
		s.state[15] = binary.LittleEndian.Uint32(nonce[8:12])
	default:
		panic("bad nonce length")
	}

	s.rounds = rounds
	ChaCha20Block(&s.state, s.block[:], s.rounds)
	return s
}

```

这段代码的作用是实现了一个XORKeyStream函数，接收两个ChaCha20Stream对象作为参数，并对其中一个Stream的输入数据进行异或操作，然后输出结果。

函数接收两个字节切片（src）作为输入数据，并输出一个字节切片（dst）。在函数内部，首先定义了一个变量i，用于跟踪输入数据中需要在循环中使用的最后一个块（gap）的偏移量。

接下来定义了一个变量max，用于存储输入数据的长度，并在循环中计算出gap的最大值，即输入数据长度除以64并向下取整，以确保循环使用的块大小是64的倍数。

在循环中，定义了一个变量o，用于跟踪已经使用的最后一个块的偏移量。然后，在循环中从src的第二个块开始，对每个块进行异或操作，并将结果存储到dst中。

接着，循环变量i，在每次循环中更新了gap，并根据gap的值更新了s.offset和s.state。最后，如果在循环中发现了小于gap的最后一个块，则将s.offset设置为gap，并将s.state的第12个字段（即循环计数器）加1，然后执行ChaCha20Block函数对输入数据进行异或操作。

总的来说，这段代码实现了一个异或操作，用于对一个ChaCha20Stream对象的输入数据进行异或操作，并输出了结果。


```go
func (s *ChaCha20Stream) XORKeyStream(dst, src []byte) {
	// Stride over the input in 64-byte blocks, minus the amount of keystream
	// previously used. This will produce best results when processing blocks
	// of a size evenly divisible by 64.
	i := 0
	max := len(src)
	for i < max {
		gap := blockSize - s.offset

		limit := i + gap
		if limit > max {
			limit = max
		}

		o := s.offset
		for j := i; j < limit; j++ {
			dst[j] = src[j] ^ s.block[o]
			o++
		}

		i += gap
		s.offset = o

		if o == blockSize {
			s.offset = 0
			s.state[12]++
			ChaCha20Block(&s.state, s.block[:], s.rounds)
		}
	}
}

```

# `common/crypto/internal/chacha_core.generated.go`

This code appears to implement the function `putItherwise` for two input parameters `in` and `out`, which are both slices of integers of the same length `32`. The function must put the first `32` uncommitted bytes of the input `in` slice into the output `out` slice, regardless of their order. The order of the bytes in the input `in` slice is not guaranteed.

The `LittleEndian` package is used to handle the endianness of the input `in` slice. The `PutUint32` method is used to write the first 32 uncommitted bytes of the input slice into the output `out` slice. The order in which these bytes are written may be different than the order in which they were received, since the `LittleEndian` package ensures that the order is consistent.


```go
package internal

import "encoding/binary"

func ChaCha20Block(s *[16]uint32, out []byte, rounds int) {
	var x0, x1, x2, x3, x4, x5, x6, x7, x8, x9, x10, x11, x12, x13, x14, x15 = s[0], s[1], s[2], s[3], s[4], s[5], s[6], s[7], s[8], s[9], s[10], s[11], s[12], s[13], s[14], s[15]
	for i := 0; i < rounds; i += 2 {
		var x uint32

		x0 += x4
		x = x12 ^ x0
		x12 = (x << 16) | (x >> (32 - 16))
		x8 += x12
		x = x4 ^ x8
		x4 = (x << 12) | (x >> (32 - 12))
		x0 += x4
		x = x12 ^ x0
		x12 = (x << 8) | (x >> (32 - 8))
		x8 += x12
		x = x4 ^ x8
		x4 = (x << 7) | (x >> (32 - 7))
		x1 += x5
		x = x13 ^ x1
		x13 = (x << 16) | (x >> (32 - 16))
		x9 += x13
		x = x5 ^ x9
		x5 = (x << 12) | (x >> (32 - 12))
		x1 += x5
		x = x13 ^ x1
		x13 = (x << 8) | (x >> (32 - 8))
		x9 += x13
		x = x5 ^ x9
		x5 = (x << 7) | (x >> (32 - 7))
		x2 += x6
		x = x14 ^ x2
		x14 = (x << 16) | (x >> (32 - 16))
		x10 += x14
		x = x6 ^ x10
		x6 = (x << 12) | (x >> (32 - 12))
		x2 += x6
		x = x14 ^ x2
		x14 = (x << 8) | (x >> (32 - 8))
		x10 += x14
		x = x6 ^ x10
		x6 = (x << 7) | (x >> (32 - 7))
		x3 += x7
		x = x15 ^ x3
		x15 = (x << 16) | (x >> (32 - 16))
		x11 += x15
		x = x7 ^ x11
		x7 = (x << 12) | (x >> (32 - 12))
		x3 += x7
		x = x15 ^ x3
		x15 = (x << 8) | (x >> (32 - 8))
		x11 += x15
		x = x7 ^ x11
		x7 = (x << 7) | (x >> (32 - 7))
		x0 += x5
		x = x15 ^ x0
		x15 = (x << 16) | (x >> (32 - 16))
		x10 += x15
		x = x5 ^ x10
		x5 = (x << 12) | (x >> (32 - 12))
		x0 += x5
		x = x15 ^ x0
		x15 = (x << 8) | (x >> (32 - 8))
		x10 += x15
		x = x5 ^ x10
		x5 = (x << 7) | (x >> (32 - 7))
		x1 += x6
		x = x12 ^ x1
		x12 = (x << 16) | (x >> (32 - 16))
		x11 += x12
		x = x6 ^ x11
		x6 = (x << 12) | (x >> (32 - 12))
		x1 += x6
		x = x12 ^ x1
		x12 = (x << 8) | (x >> (32 - 8))
		x11 += x12
		x = x6 ^ x11
		x6 = (x << 7) | (x >> (32 - 7))
		x2 += x7
		x = x13 ^ x2
		x13 = (x << 16) | (x >> (32 - 16))
		x8 += x13
		x = x7 ^ x8
		x7 = (x << 12) | (x >> (32 - 12))
		x2 += x7
		x = x13 ^ x2
		x13 = (x << 8) | (x >> (32 - 8))
		x8 += x13
		x = x7 ^ x8
		x7 = (x << 7) | (x >> (32 - 7))
		x3 += x4
		x = x14 ^ x3
		x14 = (x << 16) | (x >> (32 - 16))
		x9 += x14
		x = x4 ^ x9
		x4 = (x << 12) | (x >> (32 - 12))
		x3 += x4
		x = x14 ^ x3
		x14 = (x << 8) | (x >> (32 - 8))
		x9 += x14
		x = x4 ^ x9
		x4 = (x << 7) | (x >> (32 - 7))
	}
	binary.LittleEndian.PutUint32(out[0:4], s[0]+x0)
	binary.LittleEndian.PutUint32(out[4:8], s[1]+x1)
	binary.LittleEndian.PutUint32(out[8:12], s[2]+x2)
	binary.LittleEndian.PutUint32(out[12:16], s[3]+x3)
	binary.LittleEndian.PutUint32(out[16:20], s[4]+x4)
	binary.LittleEndian.PutUint32(out[20:24], s[5]+x5)
	binary.LittleEndian.PutUint32(out[24:28], s[6]+x6)
	binary.LittleEndian.PutUint32(out[28:32], s[7]+x7)
	binary.LittleEndian.PutUint32(out[32:36], s[8]+x8)
	binary.LittleEndian.PutUint32(out[36:40], s[9]+x9)
	binary.LittleEndian.PutUint32(out[40:44], s[10]+x10)
	binary.LittleEndian.PutUint32(out[44:48], s[11]+x11)
	binary.LittleEndian.PutUint32(out[48:52], s[12]+x12)
	binary.LittleEndian.PutUint32(out[52:56], s[13]+x13)
	binary.LittleEndian.PutUint32(out[56:60], s[14]+x14)
	binary.LittleEndian.PutUint32(out[60:64], s[15]+x15)
}

```

# `common/crypto/internal/chacha_core_gen.go`

这段代码是一个名为`writeQuarterRound`的函数，它接受四个整数参数`a`、`b`、`c`和`d`，并写入到一个名为`file`的文件中。

函数内部的逻辑主要分为以下几个步骤：

1. 对四个输入参数`a`、`b`、`c`和`d`进行某种数学运算，然后将结果写入到`file`中。
2. 对四个输入参数`a`、`b`、`c`和`d`中的任意一个数进行某种数学运算，然后将结果写入到`file`中。
3. 对四个输入参数`a`、`b`、`c`和`d`中的任意一个数进行某种数学运算，然后将结果写入到`file`中。
4. 对四个输入参数`a`、`b`、`c`和`d`中的任意一个数进行某种数学运算，然后将结果写入到`file`中。

具体地，这段代码实现了一个类似于“打补丁”的技巧，对输入参数`a`、`b`、`c`和`d`进行了一些奇偶性的处理，然后将结果进行了一些数学运算，最后将结果写入到`file`中。这些数学运算包括加法、异或、左移和右移等操作。


```go
// +build generate

package main

import (
	"fmt"
	"log"
	"os"
)

func writeQuarterRound(file *os.File, a, b, c, d int) {
	add := "x%d+=x%d\n"
	xor := "x=x%d^x%d\n"
	rotate := "x%d=(x << %d) | (x >> (32 - %d))\n"

	fmt.Fprintf(file, add, a, b)
	fmt.Fprintf(file, xor, d, a)
	fmt.Fprintf(file, rotate, d, 16, 16)

	fmt.Fprintf(file, add, c, d)
	fmt.Fprintf(file, xor, b, c)
	fmt.Fprintf(file, rotate, b, 12, 12)

	fmt.Fprintf(file, add, a, b)
	fmt.Fprintf(file, xor, d, a)
	fmt.Fprintf(file, rotate, d, 8, 8)

	fmt.Fprintf(file, add, c, d)
	fmt.Fprintf(file, xor, b, c)
	fmt.Fprintf(file, rotate, b, 7, 7)
}

```

This is a function definition that takes an input file object (os.File) and returns a ChaCha20 block (file.ChaCha20Block) in the form of a Go slice (file.ChaCha20Block切片).

The ChaCha20 block has a fixed format with a header that includes the 16 input values, their offsets and an index. The function uses the ChaCha20 block to encrypt the input data.

The ChaCha20 block encrypts data by XORing it with a series of xors, which are based on the input data. The rounds of the ChaCha20 block are used to provide redundancy and improve the security of the encryption.

The function also provides code for an iterative implementation of the ChaCha20 block encryption, which can be used to encrypt the data without rounding.


```go
func writeChacha20Block(file *os.File) {
	fmt.Fprintln(file, `
func ChaCha20Block(s *[16]uint32, out []byte, rounds int) {
  var x0,x1,x2,x3,x4,x5,x6,x7,x8,x9,x10,x11,x12,x13,x14,x15 = s[0],s[1],s[2],s[3],s[4],s[5],s[6],s[7],s[8],s[9],s[10],s[11],s[12],s[13],s[14],s[15]
	for i := 0; i < rounds; i+=2 {
    var x uint32
    `)

	writeQuarterRound(file, 0, 4, 8, 12)
	writeQuarterRound(file, 1, 5, 9, 13)
	writeQuarterRound(file, 2, 6, 10, 14)
	writeQuarterRound(file, 3, 7, 11, 15)
	writeQuarterRound(file, 0, 5, 10, 15)
	writeQuarterRound(file, 1, 6, 11, 12)
	writeQuarterRound(file, 2, 7, 8, 13)
	writeQuarterRound(file, 3, 4, 9, 14)
	fmt.Fprintln(file, "}")
	for i := 0; i < 16; i++ {
		fmt.Fprintf(file, "binary.LittleEndian.PutUint32(out[%d:%d], s[%d]+x%d)\n", i*4, i*4+4, i, i)
	}
	fmt.Fprintln(file, "}")
	fmt.Fprintln(file)
}

```

这段代码是一个 Go 语言程序，主要作用是生成一个名为 "chacha_core.generated.go" 的文件。这个文件是由 Go 语言的 "go generate" 命令生成的。

具体来说，这段代码打开一个名为 "chacha_core.generated.go" 的文件，并在这 file 中写入了以下内容：

1. "package internal" 输出了一个名为 "internal" 的包。
2. "package internal" 输出了一行。
3. "fmt.Fprintln(file, "import \"encoding/binary\"") 输出了一行，并在这行中打印了 "fmt.Fprintln" 和 "encoding/binary""。
4. "fmt.Fprintln(file, "import \"os\"") 输出了一行，并在这行中打印了 "fmt.Fprintln" 和 "os"。
5. "writeChacha20Block(file)" 这一行可能是一个 Async/Await 调用，用来将一个名为 "writeChacha20Block" 的函数作用于 file。不过，从代码的开放性来看，这个函数可能是用来将 "chacha_core.go" 文件的内容复制到 "chacha_core.generated.go" 文件中。


```go
func main() {
	file, err := os.OpenFile("chacha_core.generated.go", os.O_WRONLY|os.O_TRUNC|os.O_CREATE, 0644)
	if err != nil {
		log.Fatalf("Failed to generate chacha_core.go: %v", err)
	}
	defer file.Close()

	fmt.Fprintln(file, "package internal")
	fmt.Fprintln(file)
	fmt.Fprintln(file, "import \"encoding/binary\"")
	fmt.Fprintln(file)
	writeChacha20Block(file)
}

```

# `common/dice/dice.go`

这段代码是一个Dice包，它提供了用于生成随机整数的函数和初始化math/rand函数的接口。

具体来说，代码中的两个函数roll()和rand()都是接受一个整数参数n，并返回一个非负整数在0到n之间的随机整数。roll()函数使用rand.Intn()函数生成随机整数，rand()函数使用math/rand包中的intn()函数来生成随机整数。

package dice包的作用是提供一个简单的、易于使用的随机数生成器和种子函数。通过使用rand.Intn()函数和math/rand包中的intn()函数，可以生成0到生成随机整数n之间的随机整数。


```go
// Package dice contains common functions to generate random number.
// It also initialize math/rand with the time in seconds at launch time.
package dice // import "v2ray.com/core/common/dice"

import (
	"math/rand"
	"time"
)

// Roll returns a non-negative number between 0 (inclusive) and n (exclusive).
func Roll(n int) int {
	if n == 1 {
		return 0
	}
	return rand.Intn(n)
}

```

这三段代码属于Roll库，用于生成随机数。Roll库提供了三种不同的生成随机数的方式：RollDeterministic、RollUint16和RollUint64。

第一段代码RollDeterministic的作用是生成一个非负的随机整数，范围是0到n, inclusive。具体来说，如果n是1，那么直接返回0。否则，使用Rand.New(rand.NewSource(seed))构造随机数种子，并使用rand.Intn(n)生成一个0到n之间的随机整数，其中n表示输入参数n的值。

第二段代码RollUint16的作用是生成一个0到65535之间的随机uint16值。具体来说，使用rand.Intn(65536)生成一个0到65535之间的随机整数，然后将其转换为uint16类型。

第三段代码RollUint64的作用是生成一个0到4294967295之间的随机uint64值。具体来说，使用rand.Uint64()生成一个0到4294967295之间的随机整数，然后将其转换为uint64类型。


```go
// Roll returns a non-negative number between 0 (inclusive) and n (exclusive).
func RollDeterministic(n int, seed int64) int {
	if n == 1 {
		return 0
	}
	return rand.New(rand.NewSource(seed)).Intn(n)
}

// RollUint16 returns a random uint16 value.
func RollUint16() uint16 {
	return uint16(rand.Intn(65536))
}

func RollUint64() uint64 {
	return rand.Uint64()
}

```

这段代码定义了一个名为NewDeterministicDice的函数，以及一个名为deterministicDice的结构体。函数返回一个指向deterministicDice结构体的指针，这个结构体包含一个使用随机数生成器(rand.New)创建的随机数生成器实例。

deterministicDice结构体包含一个指向rand.Rand的指针，以及一个int64类型的变量，表示下一次随机 roll()方法计算出的结果。

函数中的主要部分是一个if语句，如果n的值为1，直接返回0，否则使用int64类型的变量d来实现Intn()函数，Intn()函数会根据参数n来返回一个介于0和n之间的整数。


```go
func NewDeterministicDice(seed int64) *deterministicDice {
	return &deterministicDice{rand.New(rand.NewSource(seed))}
}

type deterministicDice struct {
	*rand.Rand
}

func (dd *deterministicDice) Roll(n int) int {
	if n == 1 {
		return 0
	}
	return dd.Intn(n)
}

```

这段代码是使用Go编程语言中的一个匿名函数。它定义了一个名为"init"的函数，在内置的"math/rand"库中进行了一个名为"Seed"的函数调用，并传入了一个名为"time.Now().Unix()"的函数作为参数。通过调用"time.Now().Unix()"获取当前时间的Unix时间戳，并将其作为参数传递给"rand.Seed"函数，用于随机数生成器的初始化。

具体来说，这段代码的作用是用于设置随机数生成器的种子，以保证每次运行程序时生成的随机数序列都不同。这个函数通常用于密码学和随机数测试等应用中，以确保数据生成的随机性和一致性。


```go
func init() {
	rand.Seed(time.Now().Unix())
}

```

# `common/dice/dice_test.go`

这段代码是一个 Go 语言编写的测试套件，用于测试掷骰子的功能。具体来说，它以下划线分成了两个部分：

1. 引入外部的 "math/rand" 和 "testing" 包，分别用于生成随机数和进行测试。
2. 定义了一个名为 "BenchmarkRoll1" 的函数，它接收一个 "testing.B" 类型的参数，代表测试套件中的一个测试用例。函数的作用是在循环中生成随机整数，然后循环多次测试这个随机整数是否符合期望。
3. 在 "BenchmarkRoll1" 函数中，使用嵌套的 "for" 循环来生成随机整数。每次循环都会调用 "Roll" 函数，这个函数会在每次循环中生成一个 1 到 6 之间的随机整数。
4. 循环结束后，使用下划线的 "N" 参数传递给 "testing.B" 类型的参数，表示生成随机整数的次数。这样就可以在循环中多次测试生成不同随机整数的实际情况。


```go
package dice_test

import (
	"math/rand"
	"testing"

	. "v2ray.com/core/common/dice"
)

func BenchmarkRoll1(b *testing.B) {
	for i := 0; i < b.N; i++ {
		Roll(1)
	}
}

```

这三段代码都是用于测试 "BenchmarkRoll20" 和 "BenchmarkIntn1" 函数的一些参数。

"func BenchmarkRoll20(b *testing.B) "是一个函数名为 "BenchmarkRoll20"，参数 "b" 是一个 testing.B 类型的参数。

这段代码的作用是用于测试 "Roll" 函数在每次循环中随机生成的 "20" 包的随机数是否符合某种预期的分布。

具体来说，代码会对于每次循环中的 "20" 包随机生成一个随机数，然后将生成的随机数放入一个名为 "b" 的 testing.B 类型的参数中，以便于对 "Roll" 函数的输出结果进行测试。

对于 "func BenchmarkIntn1(b *testing.B) "，代码的作用与上面两段代码类似，都是用于测试 "Intn" 函数在每次循环中随机生成的 "1" 或 "20" 包的随机数是否符合某种预期的分布。

具体来说，代码会对于每次循环中的 "1" 或 "20" 包随机生成一个随机数，然后将生成的随机数放入一个名为 "b" 的 testing.B 类型的参数中，以便于对 "Intn" 函数的输出结果进行测试。


```go
func BenchmarkRoll20(b *testing.B) {
	for i := 0; i < b.N; i++ {
		Roll(20)
	}
}

func BenchmarkIntn1(b *testing.B) {
	for i := 0; i < b.N; i++ {
		rand.Intn(1)
	}
}

func BenchmarkIntn20(b *testing.B) {
	for i := 0; i < b.N; i++ {
		rand.Intn(20)
	}
}

```

# `common/errors/errors.go`

这段代码定义了一个名为 "errors" 的包，其作用是提供 Golang 中错误包的drop-in替换。该包通过导入 "v2ray.com/core/common/errors" 来引入该包的依赖。

具体来说，这段代码实现了以下功能：

1. 定义了一个名为 "hasInnerError" 的接口，该接口的实现者可以返回其内部错误。
2. 在 "errors" 包的导入中，通过调用 "v2ray.com/core/common/errors" 中的 "isError" 函数，将 "errors" 包导入到 "errors" 包中。
3. 在 "errors" 包的实现中，定义了一个名为 "innerError" 的函数，该函数接收一个 "hasInnerError" 类型的参数，并返回其内部错误。
4. 在 "innerError" 函数中，首先调用 "Inner" 函数（缺少参数的 "Inner" 函数，根据 Golang 官方文档，应该是 "inner" 函数的别名）获取错误 inner 对象，然后使用获取的 inner 对象调用 "strings.hasP" 函数（根据官方文档，这个函数会检查给定的字符串是否存在于给定的查找子串中），如果该字符串存在，则返回 "true"，否则返回 "false"。
5. 在 "innerError" 函数中，使用反射调用 "Inner" 函数的 "()" 方法获取错误对象，然后使用 "is" 函数获取错误对象是否为 "no error"。如果是，"innerError" 函数会输出 "Error: <该错误对象的定义>"，否则会继续执行下面的代码。

总结起来，这段代码定义了一个 "errors" 包，通过该包可以方便地在程序中使用类似于 Golang 中 "errors" 包的错误处理方式。


```go
// Package errors is a drop-in replacement for Golang lib 'errors'.
package errors // import "v2ray.com/core/common/errors"

import (
	"os"
	"reflect"
	"strings"

	"v2ray.com/core/common/log"
	"v2ray.com/core/common/serial"
)

type hasInnerError interface {
	// Inner returns the underlying error of this one.
	Inner() error
}

```

这段代码定义了一个名为Error的结构体，该结构体包含一个pathObj字段，类型为interface{}，该字段用于访问该错误对象的路径。

另外，还定义了一个Error接口，该接口包含一个Severity字段，类型为log.Severity，该字段表示该错误对象的严重程度。

然后，定义了一个函数WithPathObj，该函数接收一个pathObj类型的参数，将其赋值给err结构体中的pathObj字段，并返回修改后的err结构体。

最后，还定义了一个函数inner，该函数将一个错误对象与其严重程度一起作为参数传递给该函数，并返回一个新的错误对象，其严重程度由传入的严重程度和当前严重程度比较后取较小值。


```go
type hasSeverity interface {
	Severity() log.Severity
}

// Error is an error object with underlying error.
type Error struct {
	pathObj  interface{}
	prefix   []interface{}
	message  []interface{}
	inner    error
	severity log.Severity
}

func (err *Error) WithPathObj(obj interface{}) *Error {
	err.pathObj = obj
	return err
}

```

这段代码定义了两个函数：`func (err *Error) pkgPath() string` 和 `func (err *Error) Error() string`。

函数 `pkgPath()` 接收一个指向 `Error` 类型对象的 `err.pathObj` 变量。如果 `err.pathObj` 为空，函数返回一个空字符串。否则，函数使用 `reflect.TypeOf()` 函数获取 `err.pathObj` 的类型，并返回该类型的包名。

函数 `Error()` 接收一个指向 `Error` 类型对象的 `err` 变量。首先，函数创建一个空字符串 `builder`。然后，使用循环遍历 `err.prefix` 切片。对于每个元素，函数创建一个字符串，并将其附加到 `builder` 字符串的开头。接着，函数获取前一个字符串的类型，并将其附加到 `builder` 字符串的尾部。然后，函数将 `err.pkgPath()` 返回的路径附加到 `builder` 字符串的末尾。接下来，函数创建一个字符串切片，并将其附加到 `builder` 字符串的开头。然后，函数创建一个字符串，并将其附加到 `builder` 字符串的尾部。最后，如果 `err.inner` 为非 ` nil`，函数创建一个字符串，并将其附加到 `builder` 字符串的末尾。函数最终返回 `builder.String()`，即构建的错误字符串。


```go
func (err *Error) pkgPath() string {
	if err.pathObj == nil {
		return ""
	}
	return reflect.TypeOf(err.pathObj).PkgPath()
}

// Error implements error.Error().
func (err *Error) Error() string {
	builder := strings.Builder{}
	for _, prefix := range err.prefix {
		builder.WriteByte('[')
		builder.WriteString(serial.ToString(prefix))
		builder.WriteString("] ")
	}

	path := err.pkgPath()
	if len(path) > 0 {
		builder.WriteString(path)
		builder.WriteString(": ")
	}

	msg := serial.Concat(err.message...)
	builder.WriteString(msg)

	if err.inner != nil {
		builder.WriteString(" > ")
		builder.WriteString(err.inner.Error())
	}

	return builder.String()
}

```

这段代码定义了三个函数，它们都接受一个名为“err”的指针变量，并返回一个指向“Error”类型的新值。

第一个函数名为“Inner”，它尝试调用另一个函数“Inner()”，这个函数接收一个错误对象“err.inner”的值。如果“err.inner”为空，函数将返回一个空的错误对象。否则，函数将返回“err.inner”的值。

第二个函数名为“Base”，它接收一个名为“e”的错误对象作为参数，并将其传递给另一个函数“Inner()”。

第三个函数名为“atSeverity”，它接收一个名为“s”的日志 severity 参数，并将它设置为错误对象的 severity。

总的来说，这段代码定义了三个函数，它们接受一个名为“err”的指针变量，并尝试通过不同的方式来获取该错误对象的值和信息，以便在日志中更好地描述错误情况。


```go
// Inner implements hasInnerError.Inner()
func (err *Error) Inner() error {
	if err.inner == nil {
		return nil
	}
	return err.inner
}

func (err *Error) Base(e error) *Error {
	err.inner = e
	return err
}

func (err *Error) atSeverity(s log.Severity) *Error {
	err.severity = s
	return err
}

```

这段代码定义了一个名为 "func" 的函数，它接收一个名为 "err" 的错误对象作为参数，并返回一个名为 "Severity" 的接口类型。

函数的实现分为以下几个步骤：

1. 如果 err.inner 是一个 nil 值，那么直接返回 err.severity，因为 nil 值与任何的严重性级别都为 0 相似，所以返回的 severity 也是 0。

2. 如果 err.inner 是一个具有 "hasSeverity" 类型的对象，那么先尝试从该对象中获取严重性级别，然后计算当前严重性级别(如果存在)，最后比较两个严重性级别的大小，如果当前严重性级别小于 err.severity 级别，那么就返回当前严重性级别，否则就返回 err.severity 级别。

3. 如果 err.inner 不包含 "hasSeverity" 类型的对象，那么就直接返回 err.severity 作为严重性级别。

4. 最后，函数返回 err.severity 作为严重性级别，这样就可以在日志中使用该严重性级别了。


```go
func (err *Error) Severity() log.Severity {
	if err.inner == nil {
		return err.severity
	}

	if s, ok := err.inner.(hasSeverity); ok {
		as := s.Severity()
		if as < err.severity {
			return as
		}
	}

	return err.severity
}

```

这段代码定义了三个名为 `AtDebug`、`AtInfo` 和 `AtWarning` 的函数，它们都接受一个名为 `err` 的错误对象作为参数，并返回一个指向 `Error` 类型的变量。

每个函数都使用 `log.Severity_` 宏定义了 severity(严重程度)，其中 `_` 表示该属性的默认值。这些函数的作用是为 `err` 对象设置不同的严重程度，以便在输出日志时根据情况显示不同的错误信息。

具体来说，`AtDebug` 函数会将 `err` 对象的严重程度设置为 `log.Severity_Debug`，这意味着它将输出 "DEBUG" 格式的错误信息。`AtInfo` 函数将 `err` 对象的严重程度设置为 `log.Severity_Info`，这意味着它将输出 "INFO" 格式的错误信息。`AtWarning` 函数将 `err` 对象的严重程度设置为 `log.Severity_Warning`，这意味着它将输出 "WARNING" 格式的错误信息。

这些函数可以用来设置不同严重程度下的错误信息输出，以便在程序开发和测试中更加方便和灵活地输出错误信息。


```go
// AtDebug sets the severity to debug.
func (err *Error) AtDebug() *Error {
	return err.atSeverity(log.Severity_Debug)
}

// AtInfo sets the severity to info.
func (err *Error) AtInfo() *Error {
	return err.atSeverity(log.Severity_Info)
}

// AtWarning sets the severity to warning.
func (err *Error) AtWarning() *Error {
	return err.atSeverity(log.Severity_Warning)
}

```

这段代码定义了三个函数，分别是：

1. `AtError()`函数，接受一个 `Error` 类型的参数，并返回一个 `Error` 类型的引用。这个函数的作用是，在出错时返回一个具有 `AtError` 严重性的 `Error` 对象。
2. `String()` 函数，接受一个 `Error` 类型的参数，并返回该错误对象的 `String` 形式。这个函数的作用是，返回错误对象的 `String` 形式。
3. `WriteToLog()` 函数，接受一个包含 `ExportOption` 类型的参数，并写入当前的错误到日志中。这个函数的作用是，将当前的 `Error` 对象写入到日志中，可以指定日志的 severity、关键字、格式等选项。


```go
// AtError sets the severity to error.
func (err *Error) AtError() *Error {
	return err.atSeverity(log.Severity_Error)
}

// String returns the string representation of this error.
func (err *Error) String() string {
	return err.Error()
}

// WriteToLog writes current error into log.
func (err *Error) WriteToLog(opts ...ExportOption) {
	var holder ExportOptionHolder

	for _, opt := range opts {
		opt(&holder)
	}

	if holder.SessionID > 0 {
		err.prefix = append(err.prefix, holder.SessionID)
	}

	log.Record(&log.GeneralMessage{
		Severity: GetSeverity(err),
		Content:  err,
	})
}

```

这段代码定义了一个名为 ExportOptionHolder 的结构体，它包含一个名为 SessionID 的 uint32 字段。

定义了一个名为 ExportOption 的函数类型，该函数接收一个名为 ExportOptionHolder 的参数，然后返回一个指向一个名为 ExportOptionHolder 的类型对象的指针。

定义了一个名为 New 的函数，该函数接收任意数量的字符串参数，然后返回一个名为 Error 的错误对象。函数内部使用统皇宫使得函数可以设置错误消息以及日志级别。

最后，在定义了 ExportOption 和 New 函数之后，没有其他代码。


```go
type ExportOptionHolder struct {
	SessionID uint32
}

type ExportOption func(*ExportOptionHolder)

// New returns a new error object with message formed from given arguments.
func New(msg ...interface{}) *Error {
	return &Error{
		message:  msg,
		severity: log.Severity_Info,
	}
}

// Cause returns the root cause of this error.
```

此代码定义了一个名为Cause的函数类型，其参数为error类型。该函数函数在内部实现了异常处理，对于每个生成的错误，函数都会尝试捕获并处理该错误。

函数首先检查给定的错误是否为空，如果是，则返回一个空错误。否则，函数会遍历给定的错误，尝试将其包装为某个具有Inner()方法的对象。

如果给定的错误是一个InnerError类型的对象，函数会尝试捕获并处理该错误。如果尝试处理错误后，仍然没有捕获到InnerError类型的对象，则会跳出循环，继续尝试处理下一个生成的错误。

如果给定的错误是一个*os.PathError类型的对象，则会尝试捕获并处理该错误。如果尝试处理错误后，仍然没有捕获到*os.PathError类型的对象，则会跳出循环，继续尝试处理下一个生成的错误。

如果给定的错误是一个*os.SyscallError类型的对象，则会尝试捕获并处理该错误。如果尝试处理错误后，仍然没有捕获到*os.SyscallError类型的对象，则会跳出循环，继续尝试处理下一个生成的错误。

如果给定的错误无法被任何类型的对象捕获，则会返回一个错误。


```go
func Cause(err error) error {
	if err == nil {
		return nil
	}
L:
	for {
		switch inner := err.(type) {
		case hasInnerError:
			if inner.Inner() == nil {
				break L
			}
			err = inner.Inner()
		case *os.PathError:
			if inner.Err == nil {
				break L
			}
			err = inner.Err
		case *os.SyscallError:
			if inner.Err == nil {
				break L
			}
			err = inner.Err
		default:
			break L
		}
	}
	return err
}

```

这段代码定义了一个名为 `GetSeverity` 的函数，它接收一个名为 `err` 的错误参数。函数的作用是获取给定错误对象的严重程度，包括其内部错误。

具体来说，如果给定的 `err` 对象具有 `hasSeverity` 字段，那么函数将返回该对象的 `Severity` 值。否则，函数将返回 `log.Severity_Info` 类型，表示严重程度为 `info` 的警告。

例如，如果给定一个错误的结构体 `Error` 并将其传递给 `GetSeverity` 函数，该函数将返回一个名为 `M轻微错误` 的严重程度，其值为 `log.Severity_Info`。


```go
// GetSeverity returns the actual severity of the error, including inner errors.
func GetSeverity(err error) log.Severity {
	if s, ok := err.(hasSeverity); ok {
		return s.Severity()
	}
	return log.Severity_Info
}

```

# `common/errors/errors_test.go`

这段代码是针对一个名为 "errors_test" 的包进行的测试。

首先引入了 v2ray.com/core/common/errors 和 io.test，然后定义了一个名为 New 的函数，接收一个错误类型参数，并返回一个自定义的错误类型实例。

在 TestError 函数中，通过调用 GetSeverity 函数，获取错误的严重程度，并与给定的严重程度进行比较。如果两个结果不同，则会输出错误信息。

在第二段中，分别创建了两个自定义的错误实例，一个错误类型为 TestError2，另一个错误类型为 TestError3，并调用 Base 函数和 AtWarning 函数（即使用 warning 级别）。

在第三段中，再次创建了一个自定义的错误实例，并使用 Base 函数，同时使用 AtWarning 函数。

最后，在 TestError4 和 TestError5 中，创建了两个自定义的错误实例，一个错误类型为 TestError4，另一个错误类型为 TestError5，并使用 Base 函数。如果此时仍然获取到 warning 级别，则输出警告信息，否则输出错误信息。同时，还检查错误实例的错误消息是否包含 "EOF"，如果是，则输出错误消息。


```go
package errors_test

import (
	"io"
	"strings"
	"testing"

	"github.com/google/go-cmp/cmp"

	. "v2ray.com/core/common/errors"
	"v2ray.com/core/common/log"
)

func TestError(t *testing.T) {
	err := New("TestError")
	if v := GetSeverity(err); v != log.Severity_Info {
		t.Error("severity: ", v)
	}

	err = New("TestError2").Base(io.EOF)
	if v := GetSeverity(err); v != log.Severity_Info {
		t.Error("severity: ", v)
	}

	err = New("TestError3").Base(io.EOF).AtWarning()
	if v := GetSeverity(err); v != log.Severity_Warning {
		t.Error("severity: ", v)
	}

	err = New("TestError4").Base(io.EOF).AtWarning()
	err = New("TestError5").Base(err)
	if v := GetSeverity(err); v != log.Severity_Warning {
		t.Error("severity: ", v)
	}
	if v := err.Error(); !strings.Contains(v, "EOF") {
		t.Error("error: ", v)
	}
}

```

这段代码定义了一个名为 e 的结构体，它包含一个空字符串类型的字段。接下来，定义了一个名为 TestErrorMessage 的测试函数，该函数使用了 cmp.Diff 函数来比较传入的错误消息和实际的错误消息之间的差异。

在 TestErrorMessage 的实现中，使用了一个包含多个异常的切片（slice） data。对于每个数据元素，它创建了一个包含两个元素的切片（slice） err 和消息。然后，使用 for 循环遍历 data 切片，并使用 cmp.Diff 函数比较每个数据元素的消息（message）和错误（err）之间的差异。如果差异不为空，则执行 t.Error 函数，并将差异打印出来。


```go
type e struct{}

func TestErrorMessage(t *testing.T) {
	data := []struct {
		err error
		msg string
	}{
		{
			err: New("a").Base(New("b")).WithPathObj(e{}),
			msg: "v2ray.com/core/common/errors_test: a > b",
		},
		{
			err: New("a").Base(New("b").WithPathObj(e{})),
			msg: "a > v2ray.com/core/common/errors_test: b",
		},
	}

	for _, d := range data {
		if diff := cmp.Diff(d.msg, d.err.Error()); diff != "" {
			t.Error(diff)
		}
	}
}

```

# `common/errors/multi_error.go`

这段代码定义了一个名为`multiError`的包，它包含一个名为`multierr`的数组，该数组中包含一个或多个错误。

该包还定义了一个名为`Error`的函数，它接受一个名为`multiError`的数组参数。此函数返回一个字符串，其中包含与给定数组中每个错误相关的信息。

最后，该包定义了一个名为`strings.Builder`的类型，该类型代表一个字符串缓冲区，可以用来构建字符串。


```go
package errors

import (
	"strings"
)

type multiError []error

func (e multiError) Error() string {
	var r strings.Builder
	r.WriteString("multierr: ")
	for _, err := range e {
		r.WriteString(err.Error())
		r.WriteString(" | ")
	}
	return r.String()
}

```

该函数`Combine`接受一个或多个`MaybeError`类型的参数，并返回一个`Error`类型的结果。

函数内部，首先定义了一个`multiError`类型的变量`errs`，用于存储多个`MaybeError`类型的参数的错误。

接着使用一个`for`循环，遍历一个包含多个`MaybeError`类型的参数的数组`maybeError`。对于每个`MaybeError`类型的参数`err`，如果该参数不是`nil`类型，则将其添加到`errs`数组中。

然后函数检查`errs`数组中的元素个数，如果为`0`，则返回`nil`。否则，函数返回`errs`数组中的第一个错误类型的`Error`。

最后，函数返回`errs`数组中的最后一个错误类型的`Error`。


```go
func Combine(maybeError ...error) error {
	var errs multiError
	for _, err := range maybeError {
		if err != nil {
			errs = append(errs, err)
		}
	}
	if len(errs) == 0 {
		return nil
	}
	return errs
}

```

# `common/errors/errorgen/main.go`

该代码的主要作用是生成一个名为"errors.generated.go"的文件，该文件包含一个自定义的错误类型"errPathObjHolder"，以及一个新函数"newError"。

函数参数是一个包含多个"interface{}"，该类型代表着程序运行时需要的错误信息，以及一个表示错误路径对象的"errPathObjHolder"。函数的实现首先尝试从当前目录获取一个名为"v2ray-core"的包名称，如果失败则打印错误并退出程序。如果成功，则将包名更改为"core"，因为"v2ray-core"是"v2ray-core"的别名。

接下来，函数创建一个名为"errors.generated.go"的文件，并向其中写入函数头部和特定类型的注释。函数头部包含错误工具包所需的导入，以及一个指向包含错误信息的"errPathObjHolder"的类型声明。函数体实现了"newError"函数，该函数接受一个或多个"interface{}"，以及一个表示错误路径对象的"errPathObjHolder"。函数的实现创建了一个新的"errPathObjHolder"对象，该对象包含一个名为"err"的错误，以及一个指向错误路径对象的引用，该对象实现了"errors.Error"接口并提供了错误路径对象的元组类型。

最后，函数使用"os.Create"函数打开一个名为"errors.generated.go"的文件，并向其中写入所有指定的内容。函数调用"fmt.Fprintln"函数将函数名称和一个新的，包含函数实现的行。函数最后使用"os.Exit"函数来设置退出状态，如果错误处理程序出现任何错误，则会打印错误并退出程序。


```go
package main

import (
	"fmt"
	"log"
	"os"
	"path/filepath"

	"v2ray.com/core/common"
)

func main() {
	pwd, err := os.Getwd()
	if err != nil {
		fmt.Println("can not get current working directory")
		os.Exit(1)
	}
	pkg := filepath.Base(pwd)
	if pkg == "v2ray-core" {
		pkg = "core"
	}

	moduleName, gmnErr := common.GetModuleName(pwd)
	if gmnErr != nil {
		fmt.Println("can not get module path", gmnErr)
		os.Exit(1)
	}

	file, err := os.OpenFile("errors.generated.go", os.O_WRONLY|os.O_TRUNC|os.O_CREATE, 0644)
	if err != nil {
		log.Fatalf("Failed to generate errors.generated.go: %v", err)
		os.Exit(1)
	}
	defer file.Close()

	fmt.Fprintln(file, "package", pkg)
	fmt.Fprintln(file, "")
	fmt.Fprintln(file, "import \""+moduleName+"/common/errors\"")
	fmt.Fprintln(file, "")
	fmt.Fprintln(file, "type errPathObjHolder struct{}")
	fmt.Fprintln(file, "")
	fmt.Fprintln(file, "func newError(values ...interface{}) *errors.Error {")
	fmt.Fprintln(file, "	return errors.New(values...).WithPathObj(errPathObjHolder{})")
	fmt.Fprintln(file, "}")
}

```

# `common/log/access.go`

这段代码定义了一个名为“log”的包，其中包含了一些定义、导入和函数。

1. 定义了一个名为“logKey”的类型，该类型是一个整数类型。

2. 定义了一个名为“accessMessageKey”的类型，该类型与“logKey”类型相同。

3. 定义了一个名为“accessMessage”的函数，该函数接受一个名为“ctx”的参数，该参数是一个上下文（context）类型的参数。

4. 定义了一个名为“formatMessage”的函数，该函数接受一个名为“msg”的参数，该参数是一个字符串类型的参数。

5. 定义了一个名为“setAccessMessage”的函数，该函数接受一个名为“key”的参数，该参数是一个整数类型的参数。

6. 导入了一个名为“context”的类型，该类型可能用于在函数中传递上下文。

7. 导入了一个名为“strings”的类型，该类型可能用于格式化字符串。

8. 在一个名为“v2ray.com/core/common/serial”的包的定义中，定义了一个名为“accessMessageKey”的类型，该类型与“logKey”类型相同。

9. 在“v2ray.com/core/common/serial”包的定义中，定义了一个名为“accessMessage”的函数，该函数接受一个名为“ctx”的参数，该参数是一个上下文（context）类型的参数。

10. 在“v2ray.com/core/common/serial”包的定义中，定义了一个名为“formatMessage”的函数，该函数接受一个名为“msg”的参数，该参数是一个字符串类型的参数。

11. 在“v2ray.com/core/common/serial”包的定义中，定义了一个名为“setAccessMessage”的函数，该函数接受一个名为“key”的参数，该参数是一个整数类型的参数。


```go
package log

import (
	"context"
	"strings"

	"v2ray.com/core/common/serial"
)

type logKey int

const (
	accessMessageKey logKey = iota
)

```

这段代码定义了一个名为 AccessMessage 的结构体类型，它包含以下字段：

* From：发送消息的人的接口类型，这里可以认为是 "user"。
* To：接收消息的人的接口类型，这里可以认为是 "user"。
* Status：状态字段，表示消息是否被接受或者被拒绝，这里使用了 AccessStatus 类型来表示。
* Reason：消息说明字段，可以包含任何描述消息的文本。
* Email：电子邮件字段，可以用于发送一封解释消息的电子邮件。
* Detour：如果消息被拒绝，这里可以记录一下级步骤或者操作步骤。

这个结构体类型表示了一个用户（AccessMessage）发送给另一个用户（AccessMessage）的消息，包含了消息发送者、接收者、状态、说明和可能的附件。


```go
type AccessStatus string

const (
	AccessAccepted = AccessStatus("accepted")
	AccessRejected = AccessStatus("rejected")
)

type AccessMessage struct {
	From   interface{}
	To     interface{}
	Status AccessStatus
	Reason interface{}
	Email  string
	Detour string
}

```

这段代码定义了一个名为"func"的函数，接受一个名为"m"的整数型参数"AccessMessage"。函数返回一个字符串类型的值，表示包含参数"m"的所有属性的字符串表示形式。

具体来说，这段代码实现了一个 serialize 和 deserialize 函数，将一个 AccessMessage 对象序列化为字符串，并将其反序列化为 AccessMessage 对象。这个函数通过构建字符串并打印一些字节数组，以及通过一些条件判断来生成字符串。

函数首先将参数 m 从 AccessMessage 中提取出来，并使用 serialize.String() 函数将其从类型为 AccessMessage 的对象中序列化为字符串。然后，它将这个字符串打印到字符串Builder 上。接着，函数循环遍历 m 中的所有属性，并使用不同的字段名来打印它们。如果 m 中包含一个名为 "Detour" 的属性，函数将打印到字符串Builder上，并使用 "[" 和 "]" 字符来表示数组。

接下来，如果 m 中包含一个名为 "Email" 的属性，函数将打印到字符串Builder上，并使用 "email:" 和 " " 字符来表示电子邮件地址。

最后，函数使用 strings.Builder() 对象的 String() 方法来获取生成的字符串，并将其返回。


```go
func (m *AccessMessage) String() string {
	builder := strings.Builder{}
	builder.WriteString(serial.ToString(m.From))
	builder.WriteByte(' ')
	builder.WriteString(string(m.Status))
	builder.WriteByte(' ')
	builder.WriteString(serial.ToString(m.To))
	builder.WriteByte(' ')
	if len(m.Detour) > 0 {
		builder.WriteByte('[')
		builder.WriteString(m.Detour)
		builder.WriteString("] ")
	}
	builder.WriteString(serial.ToString(m.Reason))

	if len(m.Email) > 0 {
		builder.WriteString("email:")
		builder.WriteString(m.Email)
		builder.WriteByte(' ')
	}
	return builder.String()
}

```

这两个函数都与一个名为 "ctx" 的上下文相关的函数。函数 "ContextWithAccessMessage" 接收一个名为 "ctx" 的上下文和一个名为 "accessMessage" 的参数 "*AccessMessage"。它返回一个名为 "ctx" 的上下文，带有名为 "accessMessageKey" 的键和参数 "accessMessage"。

函数 "AccessMessageFromContext" 则接收一个名为 "ctx" 的上下文，并返回一个名为 "AccessMessage" 的参数。它首先检查给定的上下文是否具有名为 "accessMessageKey" 的键，并且是否有一条值为 "accessMessage" 的键的值被绑定到上下文中。如果上下文包含名为 "accessMessageKey" 的键并且该键的值被绑定到上下文中，那么函数返回作为 "accessMessage" 参数的值。否则，函数返回 nil。


```go
func ContextWithAccessMessage(ctx context.Context, accessMessage *AccessMessage) context.Context {
	return context.WithValue(ctx, accessMessageKey, accessMessage)
}

func AccessMessageFromContext(ctx context.Context) *AccessMessage {
	if accessMessage, ok := ctx.Value(accessMessageKey).(*AccessMessage); ok {
		return accessMessage
	}
	return nil
}

```

# `common/log/log.go`

这段代码定义了一个名为 "log" 的包，其中包含以下几个导入和定义：

1. 导入 "v2ray.com/core/common/log" 中的 "Message" 和 "Handler" 类型；
2. 定义了一个名为 "Message" 的接口，该接口包含一个名为 "String" 的方法，用于获取消息的字符串表示；
3. 定义了一个名为 "Handler" 的接口，该接口包含一个名为 "Handle" 的方法，用于处理接收到的日志消息；
4. 在 "Handler" 类型的 "Handle" 方法中，使用了 "sync.Once" 类型的变量 "handler" 来存储处理消息的函数，以及一个名为 "msg" 的参数，该参数用于获取消息对象；
5. 最后，在 "log.Handler" 函数中，使用了一个名为 "handler" 的变量，该变量存储了一个 "Handler" 类型的函数，该函数接收一个 "Message" 类型的参数，用于获取并处理日志消息。


```go
package log // import "v2ray.com/core/common/log"

import (
	"sync"

	"v2ray.com/core/common/serial"
)

// Message is the interface for all log messages.
type Message interface {
	String() string
}

// Handler is the interface for log handler.
type Handler interface {
	Handle(msg Message)
}

```

这段代码定义了一个名为 `GeneralMessage` 的结构体，它包含一个 `Severity` 字段和一个 `Content` 字段。这个结构体定义了一种通用的日志消息类型，可以包含各种类型的内容。

接着，定义了一个名为 `String` 的方法，它实现了 `Message` 接口，这个方法将 `GeneralMessage` 类型的结构体转换为字符串并返回。

最后，定义了一个名为 `Record` 的函数，它接受一个名为 `msg` 的 `Message` 类型的参数，将其写入日志处理程序的流中。


```go
// GeneralMessage is a general log message that can contain all kind of content.
type GeneralMessage struct {
	Severity Severity
	Content  interface{}
}

// String implements Message.
func (m *GeneralMessage) String() string {
	return serial.Concat("[", m.Severity, "] ", m.Content)
}

// Record writes a message into log stream.
func Record(msg Message) {
	logHandler.Handle(msg)
}

```

该代码定义了一个名为 `syncHandler` 的变量，其类型为 `sync.Handler`。

该代码还定义了一个名为 `RegisterHandler` 的函数，其接收一个 `Handler` 类型的参数。

函数的作用是将传入的 `Handler` 类型存储在 `logHandler` 变量上，并将之前注册的 `Handler` 类型（如果有的话）丢弃。

函数的实现原理是先检查传入的 `Handler` 类型是否为 `nil`，如果是，则输出一个错误信息。然后创建一个 `RWMutex` 类型的变量 `handler`，并将其设置为传入的 `Handler` 类型。最后，将 `handler` 存储在 `logHandler` 变量上，这样每次调用 `RegisterHandler` 函数时，都会将之前注册的 `Handler` 类型丢弃，并只保留当前注册的 `Handler` 类型。


```go
var (
	logHandler syncHandler
)

// RegisterHandler register a new handler as current log handler. Previous registered handler will be discarded.
func RegisterHandler(handler Handler) {
	if handler == nil {
		panic("Log handler is nil")
	}
	logHandler.Set(handler)
}

type syncHandler struct {
	sync.RWMutex
	Handler
}

```

这是一段使用Go语言编写的同步处理函数，该函数实现了RLock和RUnlock同步机制。

函数接收一个同步处理器h和一个消息类型为Message的消息参数，函数的作用是在消息处理完后解锁并通知给定的处理器。

函数首先执行h.RLock()操作，获取RLock对象。然后使用defer h.RUnlock()方法，在函数退出时释放RLock对象。

函数判断h.Handler是否为空，如果是，则执行h.Handler.Handle(msg)方法，将消息消息传递给处理器。

函数还实现了Set函数，用于设置新的处理器。该函数需要获取当前同步器h并调用其Lock函数。在获取到新的处理器之后，使用Set函数将其设置同步处理器。最后，使用Unlock函数通知给定的同步处理器。


```go
func (h *syncHandler) Handle(msg Message) {
	h.RLock()
	defer h.RUnlock()

	if h.Handler != nil {
		h.Handler.Handle(msg)
	}
}

func (h *syncHandler) Set(handler Handler) {
	h.Lock()
	defer h.Unlock()

	h.Handler = handler
}

```