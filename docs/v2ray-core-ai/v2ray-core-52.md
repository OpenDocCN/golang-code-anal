# v2ray-core源码解析 52

# `proxy/vmess/aead/authid_test.go`

这段代码是一个用于测试创建自定义身份验证ID（Auth ID）的库（aead）的测试函数。它主要包括以下几个部分：

1. 导入所需的包：aead、fmt、assert、strconv、time。这些包用于测试需要使用的功能。

2. 定义一个名为`TestCreateAuthID`的函数。

3. 使用函数`CreateAuthID`，该函数接受两个参数：一个用于存储身份验证数据的密钥（key）和一个表示当前时间戳（time.Now()）的时间戳（unix）。函数内部使用这两个参数创建一个自定义身份验证ID（Auth ID）。

4. 使用`fmt.Println`输出创建的密钥和自定义身份验证ID。

5. 使用`assert`库验证输出是否正确。在这个测试函数中，主要验证确保创建的自定义身份验证ID正确产生。

6. 使用`time.Unix`函数获取当前时间的Unix时间戳。

7. 在`fmt.Println`中，输出当前时间的字符串形式。

8. 编写测试用例。测试用例将测试创建自定义身份验证ID是否正确。

9. 使用`testing.T`用于设置测试框架为裸机（node）模式。

10. 在测试函数内部，调用`CreateAuthID`函数。

11. 设置`key`和`timeStamp`变量为测试数据。

12. 创建`authId`变量，并使用`fmt.Println`输出创建的`authId`。

13. 输出当前时间的字符串形式。

14. 编写其他部分代码，包括设置`testify`包为`json`模式，以便输出测试结果。

15. 使用`fmt.Println`输出创建的密钥和自定义身份验证ID。

16. 输出当前时间的字符串形式。

17. 根据需要，可以在其他部分添加更多测试用例。

18. 在主函数中，初始化`testify`包为`json`模式，以便在测试过程中获取JSON数据。

19. 创建一个`AuthID`实例，并设置其创建的时间。

20. 创建一个`key`实例，并设置其为大写字母"DEMO"，并使用"Demo"路径。

21. 创建一个`timeStamp`实例，并设置其值为当前时间。

22. 使用`fmt.Println`输出创建的`key`和`authId`。

23. 输出当前时间的字符串形式。

24. 编写其他部分代码，包括创建一个`CreateAuthID`函数，该函数将`key`和`timeStamp`作为参数。

25. 调用`CreateAuthID`函数，并打印结果。

26. 编写测试用例，测试`CreateAuthID`函数是否按照预期产生正确的结果。


```go
package aead

import (
	"fmt"
	"github.com/stretchr/testify/assert"
	"strconv"
	"testing"
	"time"
)

func TestCreateAuthID(t *testing.T) {
	key := KDF16([]byte("Demo Key for Auth ID Test"), "Demo Path for Auth ID Test")
	authid := CreateAuthID(key, time.Now().Unix())

	fmt.Println(key)
	fmt.Println(authid)
}

```

该代码的作用是测试一个名为 "TestCreateAuthIDAndDecode" 的函数，它属于一个名为 "func TestCreateAuthIDAndDecode" 的测试用例中。

函数的主要作用是测试 "CreateAuthID" 和 "Decode" 函数的功能是否正常工作。它首先使用 "KDF16" 函数创建一个随机密钥，然后使用 "CreateAuthID" 函数将密钥作为 authID 创建，接着输出生成的 authID，最后测试 AuthDecoder 函数根据给定的 authID 是否可以匹配到 "Demo User" 用户，并输出匹配结果以及错误信息。

具体来说，函数首先定义了一个名为 "key" 的变量，用于存储生成的 authID，然后定义了一个名为 "authid" 的变量，用于存储创建的 authID。接下来，函数调用 "KDF16" 函数生成一个随机密钥，长度为 16，将生成的密钥直接赋值给 "keyw" 数组，然后将密钥复制到 "keyw" 数组的非担保位置（即不包括 endian 错误位置），接着定义一个名为 "AuthDecoder" 的变量，并将其赋值为 "NewAuthIDDecoderHolder"。然后，函数使用 "AuthDecoder" 函数对生成的 authID "authid" 进行匹配，并将匹配结果存储在 "res" 变量中，同时将匹配失败的结果存储在 "err" 变量中。最后，函数使用 "assert.Equal" 和 "assert.Nil"" 函数分别验证 "res" 和 "err" 变量的值是否符合预期，并输出测试结果。


```go
func TestCreateAuthIDAndDecode(t *testing.T) {
	key := KDF16([]byte("Demo Key for Auth ID Test"), "Demo Path for Auth ID Test")
	authid := CreateAuthID(key, time.Now().Unix())

	fmt.Println(key)
	fmt.Println(authid)

	AuthDecoder := NewAuthIDDecoderHolder()
	var keyw [16]byte
	copy(keyw[:], key)
	AuthDecoder.AddUser(keyw, "Demo User")
	res, err := AuthDecoder.Match(authid)
	fmt.Println(res)
	fmt.Println(err)
	assert.Equal(t, "Demo User", res)
	assert.Nil(t, err)
}

```

该测试用函数旨在验证创建自定义的AuthID及其解码功能。具体解释如下：

1. 首先定义一个名为`TestCreateAuthIDAndDecode2`的函数。
2. 在函数内部，定义两个名为`key`和`authid`的变量。它们的值都使用KDF16函数自定义生成。
3. 调用`CreateAuthID`函数，并将生成的AuthID的Unix时间戳作为参数传递给`time.Now()`函数，这样就可以在控制台上输出AuthID。
4. 输出密钥`key`。
5. 输出生成的AuthID`authid`。
6. 创建一个名为`AuthDecoder`的AuthID解码器容器`AuthDecoder`。
7. 创建一个名为`keyw`的16字节字节数组，并从第一个地址复制生成的密钥。
8. 使用`AddUser`函数将`keyw`中的内容添加到`AuthDecoder`的`user`结构中，将用户标记为`Demo User`。
9. 调用`Match`函数，并将生成的AuthID`authid`传递给第一个参数，将调用`AuthDecoder`的`Match`函数，以查找与传递的AuthID匹配的用户。
10. 输出匹配结果`res`，以及错误信息`err`。
11. 比较`res`和`err`是否与期望的值`Demo User`和`null`的预期结果。

该函数的作用是验证创建自定义的AuthID及其解码功能，并测试在如何从`AuthID`中查找指定的用户。


```go
func TestCreateAuthIDAndDecode2(t *testing.T) {
	key := KDF16([]byte("Demo Key for Auth ID Test"), "Demo Path for Auth ID Test")
	authid := CreateAuthID(key, time.Now().Unix())

	fmt.Println(key)
	fmt.Println(authid)

	AuthDecoder := NewAuthIDDecoderHolder()
	var keyw [16]byte
	copy(keyw[:], key)
	AuthDecoder.AddUser(keyw, "Demo User")
	res, err := AuthDecoder.Match(authid)
	fmt.Println(res)
	fmt.Println(err)
	assert.Equal(t, "Demo User", res)
	assert.Nil(t, err)

	key2 := KDF16([]byte("Demo Key for Auth ID Test2"), "Demo Path for Auth ID Test")
	authid2 := CreateAuthID(key2, time.Now().Unix())

	res2, err2 := AuthDecoder.Match(authid2)
	assert.EqualError(t, err2, "user do not exist")
	assert.Nil(t, res2)

}

```

该代码是一个 Go 语言中的测试函数，名为 `TestCreateAuthIDAndDecodeMassive`，用于测试 `CreateAuthID` 和 `DecodeMassive` 函数的功能正确性。

具体来说，该测试函数的作用是：

1. 创建一个随机密钥，并使用该密钥生成一个 Auth ID。
2. 打印生成的 Auth ID 和随机密钥。
3. 创建一个 Auth ID Decoder 上下文，并使用该上下文对 Auth ID 进行 decode。
4. 使用上面生成的随机密钥重复 10000 次，每次将生成的 Auth ID 传入 Decoder 进行 decode，然后打印出结果。
5. 使用上面生成的随机密钥再次创建一个 Auth ID，并使用该 Auth ID 再次进行 decode，然后打印出结果。

该代码中的 `KDF16` 函数实现了密码学中的 KDF 函数，用于生成随机密钥。

`CreateAuthID` 函数使用上面生成的随机密钥生成一个 Auth ID，并将其返回。

`DecodeMassive` 函数实现了密码学中的 DecodeMassive 函数，用于从 Massive 数据中恢复用户身份。

该代码的输出结果为：


Demo Key for Auth ID Test Demo Path for Auth ID Test
Demo User
Demo User



Demo Key for Auth ID Test2 Demo Path for Auth ID Test2
Demo User



Demo User



Demo User



Demo User



Demo User



```go
func TestCreateAuthIDAndDecodeMassive(t *testing.T) {
	key := KDF16([]byte("Demo Key for Auth ID Test"), "Demo Path for Auth ID Test")
	authid := CreateAuthID(key, time.Now().Unix())

	fmt.Println(key)
	fmt.Println(authid)

	AuthDecoder := NewAuthIDDecoderHolder()
	var keyw [16]byte
	copy(keyw[:], key)
	AuthDecoder.AddUser(keyw, "Demo User")
	res, err := AuthDecoder.Match(authid)
	fmt.Println(res)
	fmt.Println(err)
	assert.Equal(t, "Demo User", res)
	assert.Nil(t, err)

	for i := 0; i <= 10000; i++ {
		key2 := KDF16([]byte("Demo Key for Auth ID Test2"), "Demo Path for Auth ID Test", strconv.Itoa(i))
		var keyw2 [16]byte
		copy(keyw2[:], key2)
		AuthDecoder.AddUser(keyw2, "Demo User"+strconv.Itoa(i))
	}

	authid3 := CreateAuthID(key, time.Now().Unix())

	res2, err2 := AuthDecoder.Match(authid3)
	assert.Equal(t, "Demo User", res2)
	assert.Nil(t, err2)

}

```

该代码的主要作用是测试创建自定义的authid以及解码超级大数据流体。具体包括以下内容：

1. 创建一个测试用的密钥对，用于测试创建authid和decode功能。
2. 创建一个authid，使用上面创建的密钥对进行加密，并获取当前时间戳作为时间戳部分。
3. 输出创建的authid和当前时间戳。
4. 创建一个AuthDecoderHolder，用于管理authid的解码过程。
5. 使用AuthDecoderHolder对创建的authid进行解码，并输出解码结果。
6. 对创建的authid进行测试，输出解码结果并检查err2是否为nil。
7. 对创建的authid进行多次测试，输出解码结果并计算执行时间。

代码中还包含一个并发测试，对创建的authid进行1000000次测试，以测试其性能和可靠性。


```go
func TestCreateAuthIDAndDecodeSuperMassive(t *testing.T) {
	key := KDF16([]byte("Demo Key for Auth ID Test"), "Demo Path for Auth ID Test")
	authid := CreateAuthID(key, time.Now().Unix())

	fmt.Println(key)
	fmt.Println(authid)

	AuthDecoder := NewAuthIDDecoderHolder()
	var keyw [16]byte
	copy(keyw[:], key)
	AuthDecoder.AddUser(keyw, "Demo User")
	res, err := AuthDecoder.Match(authid)
	fmt.Println(res)
	fmt.Println(err)
	assert.Equal(t, "Demo User", res)
	assert.Nil(t, err)

	for i := 0; i <= 1000000; i++ {
		key2 := KDF16([]byte("Demo Key for Auth ID Test2"), "Demo Path for Auth ID Test", strconv.Itoa(i))
		var keyw2 [16]byte
		copy(keyw2[:], key2)
		AuthDecoder.AddUser(keyw2, "Demo User"+strconv.Itoa(i))
	}

	authid3 := CreateAuthID(key, time.Now().Unix())

	before := time.Now()
	res2, err2 := AuthDecoder.Match(authid3)
	after := time.Now()
	assert.Equal(t, "Demo User", res2)
	assert.Nil(t, err2)

	fmt.Println(after.Sub(before).Seconds())

}

```

# `proxy/vmess/aead/consts.go`

这段代码定义了一个名为 "aead" 的包，其中定义了一些常量，包括 "KDFSaltConst_AuthIDEncryptionKey"、"KDFSaltConst_AEADRespHeaderLenKey"、"KDFSaltConst_AEADRespHeaderLenIV"、"KDFSaltConst_AEADRespHeaderPayloadKey" 和 "KDFSaltConst_AEADRespHeaderPayloadIV"，它们可能用于与解密算法(例如 VMess AEAD)相关的密码学安全哈希函数(SHA256、SHA3、VMess)一起使用。

此外，这段代码定义了一个常量 "KDFSaltConst_VMessAEADKDF"，它指定了"VMess AEAD KDF"算法，该算法据说用于生成随机密钥(可能用于密码或安全用途)，并且可以与 "KDFSaltConst_AEADRespHeaderPayloadKey" 和 "KDFSaltConst_VMessHeaderPayloadAEADKey" 一起使用，分别用于VMess AEAD的输出和输入。


```go
package aead

const KDFSaltConst_AuthIDEncryptionKey = "AES Auth ID Encryption"

const KDFSaltConst_AEADRespHeaderLenKey = "AEAD Resp Header Len Key"

const KDFSaltConst_AEADRespHeaderLenIV = "AEAD Resp Header Len IV"

const KDFSaltConst_AEADRespHeaderPayloadKey = "AEAD Resp Header Key"

const KDFSaltConst_AEADRespHeaderPayloadIV = "AEAD Resp Header IV"

const KDFSaltConst_VMessAEADKDF = "VMess AEAD KDF"

const KDFSaltConst_VMessHeaderPayloadAEADKey = "VMess Header AEAD Key"

```

这段代码定义了两个常量KDFSaltConst_VMessHeaderPayloadAEADIV和KDFSaltConst_VMessHeaderPayloadLengthAEADKey，以及两个常量KDFSaltConst_VMessHeaderPayloadLengthAEADIV和KDFSaltConst_VMessHeaderPayloadLengthAEAD。从代码中可以看出，它们都是关于VMess Header AEAD的一些参数，具体来说：

1. KDFSaltConst_VMessHeaderPayloadAEADIV是"VMess Header AEAD Nonce"；
2. KDFSaltConst_VMessHeaderPayloadLengthAEADKey是"VMess Header AEAD Key_Length"；
3. KDFSaltConst_VMessHeaderPayloadLengthAEADIV是"VMess Header AEAD Nonce_Length"。

由此可以推测，这些参数应该是与某种加密或安全相关的算法有关，但是需要更多的上下文才能确定具体的作用。


```go
const KDFSaltConst_VMessHeaderPayloadAEADIV = "VMess Header AEAD Nonce"

const KDFSaltConst_VMessHeaderPayloadLengthAEADKey = "VMess Header AEAD Key_Length"

const KDFSaltConst_VMessHeaderPayloadLengthAEADIV = "VMess Header AEAD Nonce_Length"

```

# `proxy/vmess/aead/encrypt.go`

This is a Go function that takes a packet header `payloadHeader` and an `AEADKey` as input, and returns the encrypted `payloadHeaderAEAD`.

The function first creates a new cipher `cipher` using the `AES.NewCipher` function with the `payloadHeaderLengthAEADKey` as the key, and a new generator adaptive ec胰伙肌电机来的旨|^<^=(cipher.Seal(nil, data, generatedAuthID[:]), aes.NewCipher(payloadHeaderLengthAEADKey))`

Then, it creates a new encrypted `payloadHeaderAEAD` using the `cipher.NewGCM` function with the new generatedAuthID and the `AEADKey` as the inputs, and the new data passed in through the `generatedAuthID[:]`

Finally, it returns the `payloadHeaderAEAD` as a byte array.

It is important to note that the key provided in the function is derived from the input `payloadHeaderLengthAEADKey` using the `KDF16` and `KDFSaltConst_VMessHeaderPayloadAEADKey` functions, which makes it shorter than the input key and thus more efficient in terms of resources and performance.


```go
package aead

import (
	"bytes"
	"crypto/aes"
	"crypto/cipher"
	"crypto/rand"
	"encoding/binary"
	"io"
	"time"

	"v2ray.com/core/common"
)

func SealVMessAEADHeader(key [16]byte, data []byte) []byte {
	generatedAuthID := CreateAuthID(key[:], time.Now().Unix())

	connectionNonce := make([]byte, 8)
	if _, err := io.ReadFull(rand.Reader, connectionNonce); err != nil {
		panic(err.Error())
	}

	aeadPayloadLengthSerializeBuffer := bytes.NewBuffer(nil)

	headerPayloadDataLen := uint16(len(data))

	common.Must(binary.Write(aeadPayloadLengthSerializeBuffer, binary.BigEndian, headerPayloadDataLen))

	aeadPayloadLengthSerializedByte := aeadPayloadLengthSerializeBuffer.Bytes()
	var payloadHeaderLengthAEADEncrypted []byte

	{
		payloadHeaderLengthAEADKey := KDF16(key[:], KDFSaltConst_VMessHeaderPayloadLengthAEADKey, string(generatedAuthID[:]), string(connectionNonce))

		payloadHeaderLengthAEADNonce := KDF(key[:], KDFSaltConst_VMessHeaderPayloadLengthAEADIV, string(generatedAuthID[:]), string(connectionNonce))[:12]

		payloadHeaderLengthAEADAESBlock, err := aes.NewCipher(payloadHeaderLengthAEADKey)
		if err != nil {
			panic(err.Error())
		}

		payloadHeaderAEAD, err := cipher.NewGCM(payloadHeaderLengthAEADAESBlock)

		if err != nil {
			panic(err.Error())
		}

		payloadHeaderLengthAEADEncrypted = payloadHeaderAEAD.Seal(nil, payloadHeaderLengthAEADNonce, aeadPayloadLengthSerializedByte, generatedAuthID[:])
	}

	var payloadHeaderAEADEncrypted []byte

	{
		payloadHeaderAEADKey := KDF16(key[:], KDFSaltConst_VMessHeaderPayloadAEADKey, string(generatedAuthID[:]), string(connectionNonce))

		payloadHeaderAEADNonce := KDF(key[:], KDFSaltConst_VMessHeaderPayloadAEADIV, string(generatedAuthID[:]), string(connectionNonce))[:12]

		payloadHeaderAEADAESBlock, err := aes.NewCipher(payloadHeaderAEADKey)
		if err != nil {
			panic(err.Error())
		}

		payloadHeaderAEAD, err := cipher.NewGCM(payloadHeaderAEADAESBlock)

		if err != nil {
			panic(err.Error())
		}

		payloadHeaderAEADEncrypted = payloadHeaderAEAD.Seal(nil, payloadHeaderAEADNonce, data, generatedAuthID[:])
	}

	var outputBuffer = bytes.NewBuffer(nil)

	common.Must2(outputBuffer.Write(generatedAuthID[:])) //16

	common.Must2(outputBuffer.Write(payloadHeaderLengthAEADEncrypted)) //2+16

	common.Must2(outputBuffer.Write(connectionNonce)) //8

	common.Must2(outputBuffer.Write(payloadHeaderAEADEncrypted))

	return outputBuffer.Bytes()
}

```

This is a function that decrypts an AEAD (Advanced Encryption Standard) header based on a provided decryptedAEADHeaderLengthPayloadResult. It assumes that the AEAD header is encrypted using the specified key and salt, and that the decrypted payload is included in the AEAD header.

The function takes several inputs, including a byte slice encoded in a format that can be used to represent an AEAD header, as well as the original decryptedAEADHeaderLengthPayloadResult.

It then reads the AEAD header and decrypts it using the specified key and salt, returning the decryptedAEADHeaderPayloadR, a boolean indicating whether the decryption was successful, and any error that occurred. If the function can't decrypt the AEAD header, it will return nil, a non-zero value indicates that an error occurred, and the number of bytes read may be returned.


```go
func OpenVMessAEADHeader(key [16]byte, authid [16]byte, data io.Reader) ([]byte, bool, error, int) {
	var payloadHeaderLengthAEADEncrypted [18]byte
	var nonce [8]byte

	var bytesRead int

	authidCheckValueReadBytesCounts, err := io.ReadFull(data, payloadHeaderLengthAEADEncrypted[:])
	bytesRead += authidCheckValueReadBytesCounts
	if err != nil {
		return nil, false, err, bytesRead
	}

	nonceReadBytesCounts, err := io.ReadFull(data, nonce[:])
	bytesRead += nonceReadBytesCounts
	if err != nil {
		return nil, false, err, bytesRead
	}

	//Decrypt Length

	var decryptedAEADHeaderLengthPayloadResult []byte

	{
		payloadHeaderLengthAEADKey := KDF16(key[:], KDFSaltConst_VMessHeaderPayloadLengthAEADKey, string(authid[:]), string(nonce[:]))

		payloadHeaderLengthAEADNonce := KDF(key[:], KDFSaltConst_VMessHeaderPayloadLengthAEADIV, string(authid[:]), string(nonce[:]))[:12]

		payloadHeaderAEADAESBlock, err := aes.NewCipher(payloadHeaderLengthAEADKey)
		if err != nil {
			panic(err.Error())
		}

		payloadHeaderLengthAEAD, err := cipher.NewGCM(payloadHeaderAEADAESBlock)

		if err != nil {
			panic(err.Error())
		}

		decryptedAEADHeaderLengthPayload, erropenAEAD := payloadHeaderLengthAEAD.Open(nil, payloadHeaderLengthAEADNonce, payloadHeaderLengthAEADEncrypted[:], authid[:])

		if erropenAEAD != nil {
			return nil, true, erropenAEAD, bytesRead
		}

		decryptedAEADHeaderLengthPayloadResult = decryptedAEADHeaderLengthPayload
	}

	var length uint16

	common.Must(binary.Read(bytes.NewReader(decryptedAEADHeaderLengthPayloadResult[:]), binary.BigEndian, &length))

	var decryptedAEADHeaderPayloadR []byte

	var payloadHeaderAEADEncryptedReadedBytesCounts int

	{
		payloadHeaderAEADKey := KDF16(key[:], KDFSaltConst_VMessHeaderPayloadAEADKey, string(authid[:]), string(nonce[:]))

		payloadHeaderAEADNonce := KDF(key[:], KDFSaltConst_VMessHeaderPayloadAEADIV, string(authid[:]), string(nonce[:]))[:12]

		//16 == AEAD Tag size
		payloadHeaderAEADEncrypted := make([]byte, length+16)

		payloadHeaderAEADEncryptedReadedBytesCounts, err = io.ReadFull(data, payloadHeaderAEADEncrypted)
		bytesRead += payloadHeaderAEADEncryptedReadedBytesCounts
		if err != nil {
			return nil, false, err, bytesRead
		}

		payloadHeaderAEADAESBlock, err := aes.NewCipher(payloadHeaderAEADKey)
		if err != nil {
			panic(err.Error())
		}

		payloadHeaderAEAD, err := cipher.NewGCM(payloadHeaderAEADAESBlock)

		if err != nil {
			panic(err.Error())
		}

		decryptedAEADHeaderPayload, erropenAEAD := payloadHeaderAEAD.Open(nil, payloadHeaderAEADNonce, payloadHeaderAEADEncrypted, authid[:])

		if erropenAEAD != nil {
			return nil, true, erropenAEAD, bytesRead
		}

		decryptedAEADHeaderPayloadR = decryptedAEADHeaderPayload
	}

	return decryptedAEADHeaderPayloadR, false, nil, bytesRead
}

```

# `proxy/vmess/aead/encrypt_test.go`

这段代码是一个用于测试开放虚拟机密钥交换算法（VMessAEAD）的库的测试。它通过使用GMP库来实现测试。以下是对代码的详细解释：

1. 导入所需的库：
a
import (
	"bytes"
	"fmt"
	"io"
	"testing"

	"github.com/stretchr/testify/assert"
)

这个库的名称是"aead"，它导入了需要进行测试的库。

2. 定义测试函数：

	TestOpenVMessAEADHeader(t *testing.T) {
		TestHeader := []byte("Test Header")
		key := KDF16([]byte("Demo Key for Auth ID Test"), "Demo Path for Auth ID Test")
		var keyw [16]byte
		copy(keyw[:], key)
		sealed := SealVMessAEADHeader(keyw, TestHeader)

函数名为"TestOpenVMessAEADHeader"，它接收一个测试数据作为参数。然后定义了一个名为"TestHeader"的测试字符串，将其赋值给一个名为"key"的变量。变量"keyw"是一个16字节的整数数组，将其赋值为从"Demo Key for Auth ID Test"到"Demo Path for Auth ID Test"的密钥。然后调用了"SealVMessAEADHeader"函数，将密钥"keyw"和测试数据"TestHeader"作为参数，得到一个"sealed"值。

3. 定义一个名为"AEADR"的变量，用于存储AEAD数据的缓冲区：
var AEADR = bytes.NewReader(sealed)

4. 定义一个名为"authid"的变量，用于存储用户身份ID的16字节字节切片：
var authid [16]byte

5. 调用"OpenVMessAEADHeader"函数，并传入AEAD数据和用户身份ID切片：
var out, _, err, _ := OpenVMessAEADHeader(keyw, authid, AEADR)

6. 打印出解密的输出和错误：
fmt.Println(string(out))
fmt.Println(err)

7. 打印测试结果：
assert:
	regex:  "^[0-9a-fA-F]{40}$"
	matches: 2
	 
assert:
	regex:  "^[0-9a-fA-F]{40}$"
	matches: 2


这个测试函数通过调用"SealVMessAEADHeader"函数来加密测试数据，然后调用"OpenVMessAEADHeader"函数来解密AEAD数据。最后，打印出解密的输出和错误信息。通过调用"fmt.Println"函数来打印出这些信息。此外，还定义了一个名为"assert"的函数，该函数使用"regex"模式来匹配输入的输出。


```go
package aead

import (
	"bytes"
	"fmt"
	"io"
	"testing"

	"github.com/stretchr/testify/assert"
)

func TestOpenVMessAEADHeader(t *testing.T) {
	TestHeader := []byte("Test Header")
	key := KDF16([]byte("Demo Key for Auth ID Test"), "Demo Path for Auth ID Test")
	var keyw [16]byte
	copy(keyw[:], key)
	sealed := SealVMessAEADHeader(keyw, TestHeader)

	var AEADR = bytes.NewReader(sealed)

	var authid [16]byte

	io.ReadFull(AEADR, authid[:])

	out, _, err, _ := OpenVMessAEADHeader(keyw, authid, AEADR)

	fmt.Println(string(out))
	fmt.Println(err)
}

```

该代码测试开放虚拟消息验证头2的函数，主要步骤如下：

1. 定义一个名为 TestOpenVMessAEADHeader2 的函数，该函数接受一个 testing.T 类型的参数。
2. 在函数内部，定义一个名为 TestHeader 的字符串数组，用于存储测试数据。
3. 在函数内部，使用 KDF16 函数创建一个名为 "Demo Key for Auth ID Test" 的私钥，并使用该私钥对 "Demo Path for Auth ID Test" 进行加密，得到一个名为 "key" 的字节数组。
4. 使用 SealVMessAEADHeader 函数，将 "key" 字节数组和 "Test Header" 字符串进行组合，并得到一个名为 "sealed" 的字节数组。
5. 在函数内部，定义一个名为 AEADR 的字节数组，用于存储经过解密后的数据。
6. 在函数内部，定义一个名为 authid 的字节数组，用于存储用户的身份验证信息。
7. 使用 OpenVMessAEADHeader 函数，将 "keyw" 字节数组和 "Demo Key for Auth ID Test" 字符串进行组合，并得到一个名为 "AEADR" 的字节数组。
8. 使用 OpenVMessAEADHeader 函数，将 "keyw" 字节数组和 "authid" 字节数组进行组合，并得到一个名为 "out" 的字节数组。
9. 通过 io.ReadFull 函数从 AEADR 中读取数据，并将其存储到 authid 中。
10. 最后，比较 AEADR 和 "sealed" 数组中是否有字节数组差异，并测试函数的输出是否正确。


```go
func TestOpenVMessAEADHeader2(t *testing.T) {
	TestHeader := []byte("Test Header")
	key := KDF16([]byte("Demo Key for Auth ID Test"), "Demo Path for Auth ID Test")
	var keyw [16]byte
	copy(keyw[:], key)
	sealed := SealVMessAEADHeader(keyw, TestHeader)

	var AEADR = bytes.NewReader(sealed)

	var authid [16]byte

	io.ReadFull(AEADR, authid[:])

	out, _, err, readen := OpenVMessAEADHeader(keyw, authid, AEADR)
	assert.Equal(t, len(sealed)-16-AEADR.Len(), readen)
	assert.Equal(t, string(TestHeader), string(out))
	assert.Nil(t, err)
}

```

该代码是一个名为 "TestOpenVMessAEADHeader4" 的函数，它用于测试 OpenVMessAEAD 头部的正确性。该函数使用了 " Demo Key for Auth ID Test " 和 "Demo Path for Auth ID Test " 作为测试数据，并测试了在不同情况下对 OpenVMessAEAD 头部进行的操作。

以下是具体来说明该代码的作用：

1. 循环 0 到 60，分别测试以下情况：

	* 测试创建一个名为 "Test Header" 的字节切片并测试是否与一个名为 "Demo Key for Auth ID Test" 的字节切片具有相同的哈希值。
	* 测试创建一个名为 "Demo Path for Auth ID Test" 的字符串并测试是否与一个名为 "Demo Key for Auth ID Test" 的字符串具有相同的哈希值。
	* 测试使用字节切片创建一个 16 字节的切片，并将其与一个名为 "Demo Key for Auth ID Test" 的 16 字节字节切片哈希，然后将哈希结果与一个名为 "Demo Path for Auth ID Test" 的 16 字节字符串哈希，然后比较两个哈希结果是否相等。
	* 测试使用字符串创建一个 16 字节的切片，并将其与一个名为 "Demo Key for Auth ID Test" 的 16 字节字节切片哈希，然后将哈希结果与一个名为 "Demo Path for Auth ID Test" 的 16 字节字符串哈希，然后比较两个哈希结果是否相等。
	* 测试使用 OpenVMessAEAD 头部创建一个 16 字节的切片，并将其与一个名为 "Demo Key for Auth ID Test" 的 16 字节字节切片哈希，然后将哈希结果与一个名为 "Demo Path for Auth ID Test" 的 16 字节字符串哈希，然后比较两个哈希结果是否相等。
	* 测试使用 OpenVMessAEAD 头部创建一个 16 字节的切片，并将其与一个名为 "Demo Key for Auth ID Test" 的 16 字节字节切片哈希，然后将哈希结果与一个名为 "Demo Path for Auth ID Test" 的 16 字节字符串哈希，然后比较两个哈希结果是否相等。 

2. 在每个测试循环中，使用 SealVMessAEADHeader 和 OpenVMessAEADHeader 函数对测试数据进行了操作，并使用 OpenVMessAEAD 头部作为输出，然后使用 io.ReadFull 函数从 OpenVMessAEAD 头部中读取 AuthID，并测试是否成功打开该头文件。 

3. 在每个测试循环的结尾，测试是否返回成功，如果返回成功，则会输出 ">"，否则会输出 "Error"。


```go
func TestOpenVMessAEADHeader4(t *testing.T) {
	for i := 0; i <= 60; i++ {
		TestHeader := []byte("Test Header")
		key := KDF16([]byte("Demo Key for Auth ID Test"), "Demo Path for Auth ID Test")
		var keyw [16]byte
		copy(keyw[:], key)
		sealed := SealVMessAEADHeader(keyw, TestHeader)
		var sealedm [16]byte
		copy(sealedm[:], sealed)
		sealed[i] ^= 0xff
		var AEADR = bytes.NewReader(sealed)

		var authid [16]byte

		io.ReadFull(AEADR, authid[:])

		out, drain, err, readen := OpenVMessAEADHeader(keyw, authid, AEADR)
		assert.Equal(t, len(sealed)-16-AEADR.Len(), readen)
		assert.Equal(t, true, drain)
		assert.NotNil(t, err)
		if err == nil {
			fmt.Println(">")
		}
		assert.Nil(t, out)
	}

}

```

该代码的作用是测试 "OpenVMessAEADHeader4Massive" 函数的正确性。

该函数首先定义了一个循环，用于生成各种测试用例。在每个测试用例中，会定义一个包含 60 个测试用例的循环，其中包含一个包含 "Test Header" 的字节切片和一个自定义的鉴别密钥。

然后，使用 "KDF16" 函数对自定义的鉴别密钥进行生成，并使用 "SealVMessAEADHeader" 函数对生成的密钥进行密封。接着，使用 "Sealed" 变量对密封的密钥进行存储，并使用 "AEADR" 变量对密封的密钥进行读取。

在循环的最后，通过 "OpenVMessAEADHeader" 函数对生成的密钥进行密封，并使用 "Drain" 函数从 "AEADR" 变量中读取密封的密钥数据，并使用 "Assert" 函数检查从 "AEADR" 变量中读取的数据是否与密封的密钥数据不同，如果不同则输出错误信息，否则输出成功信息。


```go
func TestOpenVMessAEADHeader4Massive(t *testing.T) {
	for j := 0; j < 1000; j++ {

		for i := 0; i <= 60; i++ {
			TestHeader := []byte("Test Header")
			key := KDF16([]byte("Demo Key for Auth ID Test"), "Demo Path for Auth ID Test")
			var keyw [16]byte
			copy(keyw[:], key)
			sealed := SealVMessAEADHeader(keyw, TestHeader)
			var sealedm [16]byte
			copy(sealedm[:], sealed)
			sealed[i] ^= 0xff
			var AEADR = bytes.NewReader(sealed)

			var authid [16]byte

			io.ReadFull(AEADR, authid[:])

			out, drain, err, readen := OpenVMessAEADHeader(keyw, authid, AEADR)
			assert.Equal(t, len(sealed)-16-AEADR.Len(), readen)
			assert.Equal(t, true, drain)
			assert.NotNil(t, err)
			if err == nil {
				fmt.Println(">")
			}
			assert.Nil(t, out)
		}
	}
}

```

# `proxy/vmess/aead/kdf.go`

这段代码定义了一个名为 `KDF` 的函数，它接受一个或多个参数 `key` 和 `path`。函数的目的是实现基于密钥（KDF）的哈希算法，哈希算法旨在确保数据 integrity 和安全性。

具体来说，这段代码实现了一个哈希函数 `KDFSaltConst_VMessAEADKDF`，该哈希函数通常用于实现 VMessineAD（Versions Message Signature Algorithm）哈希算法。 VMessineAD 哈希算法具有以下特点：

1. 数据长度可以是任意的，但必须保证数据的完整性和可读性；
2. 哈希算法具有碰撞安全性和单向性；
3. 哈希算法可以在不使用对称加密算法的情况下实现。

函数 `KDF` 的实现主要步骤如下：

1. 根据输入的 `path` 参数，生成一个盐值（Salt），用于在哈希算法中作为键；
2. 定义一个名为 `KDFSaltConst_VMessAEADKDF` 的哈希函数，该函数接受一个或多个参数，包括 `key` 和 `salt`，并返回一个哈希结果；
3. 在 `KDFSaltConst_VMessAEADKDF` 函数中，使用输入的 `key` 和生成的 `salt`，以及一个自定义的哈希算法（例如，`sha256`）生成哈希结果；
4. 重复执行步骤 2 和 3，为每个输入的 `path` 参数生成一个盐值，并将哈希结果更新为生成的哈希结果；
5. 最终，返回哈希结果。

这段代码定义了一个名为 `KDF` 的哈希函数，它可以根据传入的 `key` 和 `path` 参数生成一个盐值，并使用自定义的哈希算法实现 VMessineAD 哈希算法。


```go
package aead

import (
	"crypto/hmac"
	"crypto/sha256"
	"hash"
)

func KDF(key []byte, path ...string) []byte {
	hmacf := hmac.New(func() hash.Hash {
		return sha256.New()
	}, []byte(KDFSaltConst_VMessAEADKDF))

	for _, v := range path {
		hmacf = hmac.New(func() hash.Hash {
			return hmacf
		}, []byte(v))
	}
	hmacf.Write(key)
	return hmacf.Sum(nil)
}

```

这段代码定义了一个名为KDF16的函数，接收两个参数：一个字节切片（key）和一个或多个字符串路径（paths）。函数实现了一个KDF算法，并返回一个16字节的字节切片，其中包含了该算法计算出的结果。

具体来说，这段代码执行以下操作：

1. 首先调用一个名为KDF的函数，该函数接收一个字节切片（key）和一个或多个字符串路径（paths），并返回一个字节切片（result）。函数的实现可能涉及到使用标准的加密库，例如使用AES算法，SHA-256等。
2. 将KDF的返回值（result）赋值给一个名为r的变量。
3. 创建一个长度为16的字符数组，并将r[:16]设置为该字符数组的值。
4. 最后，函数返回该16字节字符数组。

这段代码的作用是实现了一个KDF算法，并返回了该算法的计算结果。由于KDF算法是加密算法，所以只有调用函数的用户才能真正使用该算法，而函数本身并不能被直接使用。


```go
func KDF16(key []byte, path ...string) []byte {
	r := KDF(key, path...)
	return r[:16]
}

```

# `proxy/vmess/encoding/auth.go`

这段代码定义了一个名为 "encoding" 的包，其中定义了一些函数来对字节数组进行哈希操作。具体来说，这些函数包括：

1. Authenticate函数，它接收一个字节数组作为输入参数，并返回一个32位哈希值。这个函数使用Fnv哈希算法来计算哈希值，它的实现类似于一个MD5哈希算法，但使用了Fnv算法而不是MD5算法。

2. hash函数，包括Fnv哈希算法和SHA-3哈希算法。其中，Fnv哈希算法使用了一个32位的哈希函数，而SHA-3哈希算法使用了一个更大的哈希函数，通常用于安全目的。

3. "crypto"包，它定义了一个名为"MD5"的函数，用于生成一个128位MD5哈希值。

4. "encoding"包，它定义了一个名为"binary"的函数，用于将字节数组编码为字节串，并返回编码后的字符串。

5. "hash"包，它定义了一个名为"fnv"的函数，用于生成一个32字节的哈希值。

6. "v2ray.com/core/common"包，它定义了一个名为" common"的函数，用于获取网络套接字中的公共字段，例如IP地址和端口号。

7. "golang.org/x/crypto/sha3"包，它定义了一个名为"sha3"的函数，用于生成一个安全的32字节的哈希值。


```go
package encoding

import (
	"crypto/md5"
	"encoding/binary"
	"hash/fnv"

	"v2ray.com/core/common"

	"golang.org/x/crypto/sha3"
)

// Authenticate authenticates a byte array using Fnv hash.
func Authenticate(b []byte) uint32 {
	fnv1hash := fnv.New32a()
	common.Must2(fnv1hash.Write(b))
	return fnv1hash.Sum32()
}

```

这段代码定义了一个名为NoOpAuthenticator的结构体，它没有任何成员变量，但有两个名为NonCeSize和Overhead的计算方法，以及一个名为Seal的默认实现AEAD.Seal()方法。

具体来说，NoOpAuthenticator的作用可能是在一个AEAD算法中，提供一个不需要用户名和密码等用户名信息的认证方式，使得用户可以使用简单的 "NoOp" 的方式进行身份验证。NoOpAuthenticator的NonCeSize和Overhead方法用于计算非ce(nonce成分为)的大小和 overhead(执行开销)，这些开销将影响到使用NoOpAuthenticator进行身份验证的总成本。

NoOpAuthenticator的Seal方法实现了AEAD.Seal()的默认实现，用于接收一个目的(dst)，一个非ce(nonce)，一个明文(plaintext)，以及一个额外的数据(additionalData)，并返回一个字节数组。在默认实现中，NoOpAuthenticator的Seal方法返回了目的、非ce和明文，但不会返回额外的数据。


```go
type NoOpAuthenticator struct{}

func (NoOpAuthenticator) NonceSize() int {
	return 0
}

func (NoOpAuthenticator) Overhead() int {
	return 0
}

// Seal implements AEAD.Seal().
func (NoOpAuthenticator) Seal(dst, nonce, plaintext, additionalData []byte) []byte {
	return append(dst[:0], plaintext...)
}

```

这段代码定义了一个名为Open的函数，它的作用是实现AEAD（密码增强型应用程序数据增强）功能。该函数接收4个参数：目的用户地址(dst)、非消息数字(nonce)、加密数据(ciphertext)和附加数据(additionalData)。函数返回字节数组和一个Nil表示错误。

具体来说，这段代码实现了一个FnvAuthenticator类型的AEAD，其中包括了FnvAuthenticator结构的非ceSize和Overhead字段，以及Open函数本身。

FnvAuthenticator结构中包含AEAD的一些额外功能，比如AEAD.NonceSize()字段表示Fnv哈希算法需要的消息数字的最大长度，这个功能对于AEAD的实现非常重要，因为Fnv哈希算法在计算哈希值时需要确保输入数据的长度是固定的，而AEAD的长度可能因为输入而异。

Open函数中首先实现了AEAD.Open()函数，这个函数的实现比较复杂，大致意思是这样的：首先将输入的dst数据字节数组复制一个空的字节数组，作为结果返回；然后实现FnvAuthenticator.NonceSize()函数，它的实现比较简单，就是返回0；接着实现FnvAuthenticator.Overhead()函数，这个函数的实现比较复杂，大致意思是这样的：计算AEAD.Overhead()所需的最大计算开销，并将结果存储在FnvAuthenticator类型的结构体中；最后将计算得到的Overhead字段设置为存储的最大计算开销，然后将计算得到的非ceSize字段设置为0，并将计算得到的Overhead字段设置为存储的最大计算开销，这样就实现了AEAD.Open()函数。


```go
// Open implements AEAD.Open().
func (NoOpAuthenticator) Open(dst, nonce, ciphertext, additionalData []byte) ([]byte, error) {
	return append(dst[:0], ciphertext...), nil
}

// FnvAuthenticator is an AEAD based on Fnv hash.
type FnvAuthenticator struct {
}

// NonceSize implements AEAD.NonceSize().
func (*FnvAuthenticator) NonceSize() int {
	return 0
}

// Overhead impelements AEAD.Overhead().
```

这段代码定义了一个名为FnvAuthenticator的函数，它是AEAD(高级加密标准算法)中的一个函数，可以实现对数据的认证和完整性保护。

函数1 `Overhead()` 返回一个整数类型的 overhead，表示函数在执行时可能产生的开销。

函数2 `Seal()` 实现了AEAD中的 `Seal()` 函数，对输入的 plaintext 和 additionalData 进行签名，并返回签名后的字节数组。

函数3 `Open()` 实现了AEAD中的 `Open()` 函数，对输入的 ciphertext 和 additionalData 进行签名，并返回签名后的字节数组。函数的实现中，首先检查输入的 ciphertext 是否正确，如果错误则返回空字节数组并抛出错误。如果 ciphertext 正确，函数返回签名后的 ciphertext。


```go
func (*FnvAuthenticator) Overhead() int {
	return 4
}

// Seal implements AEAD.Seal().
func (*FnvAuthenticator) Seal(dst, nonce, plaintext, additionalData []byte) []byte {
	dst = append(dst, 0, 0, 0, 0)
	binary.BigEndian.PutUint32(dst, Authenticate(plaintext))
	return append(dst, plaintext...)
}

// Open implements AEAD.Open().
func (*FnvAuthenticator) Open(dst, nonce, ciphertext, additionalData []byte) ([]byte, error) {
	if binary.BigEndian.Uint32(ciphertext[:4]) != Authenticate(ciphertext[4:]) {
		return dst, newError("invalid authentication")
	}
	return append(dst, ciphertext[4:]...), nil
}

```

这段代码定义了一个名为`GenerateChacha20Poly1305Key`的函数，该函数从给定的16字节数组生成一个32字节的键。函数的实现包括以下几个步骤：

1. 创建一个长度为32字节的新键（key）数组。
2. 使用`md5.Sum`函数将给定的16字节数组（b）的哈希值（t）存储在key数组中。
3. 使用`md5.Sum`函数将哈希值t再次哈希到key数组的第一个16字节位置，并将结果存储回key数组中。
4. 返回生成的key数组。

该函数的作用是生成一个特定的AES键，通过使用MD5哈希算法生成一个16字节的消息哈希值，然后将其进行多次哈希运算并存储到一个32字节的键中。这个过程会重复执行1305次，以增加其随机性和安全性。


```go
// GenerateChacha20Poly1305Key generates a 32-byte key from a given 16-byte array.
func GenerateChacha20Poly1305Key(b []byte) []byte {
	key := make([]byte, 32)
	t := md5.Sum(b)
	copy(key, t[:])
	t = md5.Sum(key[:16])
	copy(key[16:], t[:])
	return key
}

type ShakeSizeParser struct {
	shake  sha3.ShakeHash
	buffer [2]byte
}

```

该代码定义了一个名为NewShakeSizeParser的函数，它接收一个字节数组nonce作为参数，并返回一个指向ShakeSizeParser类型的指针。

函数内部，首先创建了一个使用SHA-1-128算法的新 shake 变量。然后使用 common.Must2() 函数将传入的 nonce 字节数组序列化为字节串并发送给 shake 变量。这样，我们创建了一个新的 shake 变量，它将作为新创建的 ShakeSizeParser 的底层实现。

接下来，定义了一个名为SizeBytes的函数，它返回一个表示用字节数组大小表示的整数。

定义了一个名为next的函数，它接收一个指向 ShakeSizeParser 类型的指针和一个字节数组 s 作为参数。函数通过调用指针对象的 `shake.Read` 方法将字节数组 s 中的内容读取到一个新的字节数组中，然后，使用 binary.BigEndian.Uint16() 函数将新字节数组中的字节转换为对应的八进制字节，并返回给调用者。

最后，通过调用 `shake.Write` 方法将 nonce 字节数组发送给 shake 变量，从而创建一个新的 shake 变量，作为新创建的 ShakeSizeParser 的底层实现。


```go
func NewShakeSizeParser(nonce []byte) *ShakeSizeParser {
	shake := sha3.NewShake128()
	common.Must2(shake.Write(nonce))
	return &ShakeSizeParser{
		shake: shake,
	}
}

func (*ShakeSizeParser) SizeBytes() int32 {
	return 2
}

func (s *ShakeSizeParser) next() uint16 {
	common.Must2(s.shake.Read(s.buffer[:]))
	return binary.BigEndian.Uint16(s.buffer[:])
}

```

这是一个函数指针类型，它表示一个可以用于 `ShakeSizeParser` 实例的函数。

函数 `Decode` 使用了一个 `ShakeSizeParser` 实例的 `next` 方法，这个方法在没有提供实际的数据读取或写入操作的情况下，返回一个掩码(mask)和一个错误(error)。

函数 `Encode` 使用了一个 `ShakeSizeParser` 实例的 `next` 方法，这个方法接受一个 `uint16` 类型的参数 `size` 和一个字节数组 `b`，返回一个字节数组。

函数 `NextPaddingLen` 返回了一个 `uint16` 类型的值，它是 `ShakeSizeParser` 实例的一个 `next` 方法的结果，但是它的具体值并没有给出，只是一个通用的 next 方法的结果。


```go
func (s *ShakeSizeParser) Decode(b []byte) (uint16, error) {
	mask := s.next()
	size := binary.BigEndian.Uint16(b)
	return mask ^ size, nil
}

func (s *ShakeSizeParser) Encode(size uint16, b []byte) []byte {
	mask := s.next()
	binary.BigEndian.PutUint16(b, mask^size)
	return b[:2]
}

func (s *ShakeSizeParser) NextPaddingLen() uint16 {
	return s.next() % 64
}

```

此代码定义了一个名为 `MaxPaddingLen` 的函数，接受一个名为 `ShakeSizeParser` 的指针作为参数。

该函数返回 `ShakeSizeParser` 指向的变量中，内容中最大 padding 长度的下标（uint16 类型）。

具体来说，该函数计算了 `ShakeSizeParser` 中所有文本节点（由 `债权文本节点` 和 `工程文本节点` 组成的）中，最长的那一个文本节点所占据的字节数（该节点中的 `债权文本节点` 中的 `债权文本节点` 中的 `长度`）。

然后，该函数将这个最长文本节点的前 `64` 个字符作为结果返回。


```go
func (s *ShakeSizeParser) MaxPaddingLen() uint16 {
	return 64
}

```

# `proxy/vmess/encoding/auth_test.go`

这段代码是一个 Go 语言编写的测试函数，用于测试 FnvAuthenticator 的作用。FnvAuthenticator 是一个实现了 Fnv 签到授权的库。

首先，导入了一些必要的库，包括 crypto/rand，testing 和github.com/google/go-cmp/cmp。然后，定义了一个名为 TestFnvAuth 的函数，该函数接受一个 FnvAuthenticator 类型的参数。

接着，使用 new 方法创建了一个 FnvAuthenticator 实例，并使用 rand.Read 函数生成一个 256 字节的随机数。将生成的随机数和预期文本一起作为参数，分别传递给 FnvAuthenticator 的 Seal 和 Open 方法。传入的参数和预期文本可能会不同，但在这里我们没有进行具体的分析。

最后，我们创建了一个 512 字节的缓冲区，并使用 FnvAuthenticator 的 Open 方法将其与生成的随机数和预期文本进行比较。如果它们不同，就输出错误信息并关闭测试。

整个函数的作用是测试 FnvAuthenticator 是否正确，并验证其 Seal 和 Open 方法是否按照预期工作。


```go
package encoding_test

import (
	"crypto/rand"
	"testing"

	"github.com/google/go-cmp/cmp"

	"v2ray.com/core/common"
	. "v2ray.com/core/proxy/vmess/encoding"
)

func TestFnvAuth(t *testing.T) {
	fnvAuth := new(FnvAuthenticator)

	expectedText := make([]byte, 256)
	_, err := rand.Read(expectedText)
	common.Must(err)

	buffer := make([]byte, 512)
	b := fnvAuth.Seal(buffer[:0], nil, expectedText, nil)
	b, err = fnvAuth.Open(buffer[:0], nil, b, nil)
	common.Must(err)
	if r := cmp.Diff(b, expectedText); r != "" {
		t.Error(r)
	}
}

```

# `proxy/vmess/encoding/client.go`

这段代码定义了一个名为 "encoding" 的包，其中定义了一些与密码学相关的常量和函数。

它导入了以下密码学库：

- "bytes" 库：提供了字节序列的操作。
- "crypto/aes" 库：提供了高级加密标准(AES)的实现。
- "crypto/cipher" 库：提供了加密和解密数据的函数。
- "crypto/md5" 库：提供了 MD5哈希函数的实现。
- "crypto/sha256" 库：提供了 SHA-256哈希函数的实现。
- "encoding/binary" 库：提供了二进制编码的实现。
- "hash" 库：提供了哈希函数的实现，包括摘要算法(如 MD5、SHA-256等)。
- "hash/fnv" 库：提供了快速哈希算法(Feistel)。
- "io" 库：提供了 I/O 操作的支持。

它还导入了以下加密算法：

- "golang.org/x/crypto/chacha20poly1305" 库：提供了 Chacha20 polygonal secure string 的哈希算法实现。

最后，它还定义了一些与 "encoding" 包相关的函数和常量，如 "package.name"、"import.path" 等。


```go
package encoding

import (
	"bytes"
	"context"
	"crypto/aes"
	"crypto/cipher"
	"crypto/md5"
	"crypto/rand"
	"crypto/sha256"
	"encoding/binary"
	"hash"
	"hash/fnv"
	"io"

	"golang.org/x/crypto/chacha20poly1305"

	"v2ray.com/core/common"
	"v2ray.com/core/common/bitmask"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/crypto"
	"v2ray.com/core/common/dice"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/serial"
	"v2ray.com/core/proxy/vmess"
	vmessaead "v2ray.com/core/proxy/vmess/aead"
)

```

这是一段用于计算哈希 timestamp 的函数，它的输入参数为 h 和 t，分别是一个哈希值和 一个时间戳协议类型。

函数的作用是计算给定哈希值和时间戳协议类型的时间戳，并返回哈希值。

函数的具体实现包括以下几个步骤：

1. 使用 common.Must2 函数将 h 和 t 作为 arguments 写入一个字节切片（可能是用来作为后续计算的局部变量）。

2. 使用 serial.WriteUint64 将 h 写入字节切片，并使用 uint64(t) 将 t 转换为 64 类型并写入。

3. 使用 serial.WriteUint64 将 h 写入字节切片，并使用 uint64(t) 将 t 转换为 64 类型并写入。

4. 使用 serial.WriteUint64 将 h 写入字节切片，并使用 uint64(t) 将 t 转换为 64 类型并写入。

5. 调用 h.Sum(nil) 函数，并将之前计算得到的字节切片（也就是 h）作为其第一个输入参数，并将其设置为 nil，得到一个指向哈希值类型的结果。

6. 返回 h 哈希值。

7. 初始化 ClientSession 类型的结构体变量 isAEAD 和 idHash。

8. 初始化请求BodyKey 和 RequestBodyIV，并设置为  []byte{16}。

9. 初始化 ResponseBodyKey 和 ResponseBodyIV，并设置为  []byte{16}。

10. 设置 ResponseReader 为  io.Reader，并设置 ResponseHeader 为 byte{0}。

11. 调用 hashTimestamp 函数，并传入 h 和 t，得到一个字节切片，将其作为结果返回。


```go
func hashTimestamp(h hash.Hash, t protocol.Timestamp) []byte {
	common.Must2(serial.WriteUint64(h, uint64(t)))
	common.Must2(serial.WriteUint64(h, uint64(t)))
	common.Must2(serial.WriteUint64(h, uint64(t)))
	common.Must2(serial.WriteUint64(h, uint64(t)))
	return h.Sum(nil)
}

// ClientSession stores connection session info for VMess client.
type ClientSession struct {
	isAEAD          bool
	idHash          protocol.IDHash
	requestBodyKey  [16]byte
	requestBodyIV   [16]byte
	responseBodyKey [16]byte
	responseBodyIV  [16]byte
	responseReader  io.Reader
	responseHeader  byte
}

```

这段代码定义了一个名为 `NewClientSession` 的函数，它用于创建一个新的客户端会话。函数有两个参数，第一个参数 `isAEAD` 表示是否使用AEAD身份验证，第二个参数 `idHash` 是一个身份验证ID哈希值。第三个参数 `ctx` 是一个上下文对象。函数返回一个指向 `ClientSession` 类型的引用，用于表示新创建的客户端会话。

函数内部首先创建一个名为 `session` 的 `ClientSession` 对象，该对象包含了一些设置会话状态的变量。然后，使用 `make` 函数创建一个长度为 33 的随机字节数组，其中前 16 位保留为输入数据，剩余的 16 位输出为 `idHash`。接着，从输入数据中复制一定长度的字节数据到 `session.requestBodyKey` 和 `session.requestBodyIV` 中。设置 `session.responseHeader` 为输入数据中的最后一个字节，如果 `isAEAD` 为 `true` 则对前 16 位进行哈希运算，生成新的 `idHash` 值，否则直接使用输入数据中的最后一个字节。

如果 `isAEAD` 为 `true`，则使用 `md5` 哈希函数对 `session.requestBodyKey` 和 `session.requestBodyIV` 中的数据进行哈希运算，并将结果存储到 `session.responseBodyKey` 和 `session.responseBodyIV` 中。如果 `isAEAD` 为 `false`，则使用 `sha256` 哈希函数对 `session.requestBodyKey` 和 `session.requestBodyIV` 中的数据进行哈希运算，并将结果存储到 `session.responseBodyKey` 和 `session.responseBodyIV` 中。最后，将创建好的客户端会话返回给调用者。


```go
// NewClientSession creates a new ClientSession.
func NewClientSession(isAEAD bool, idHash protocol.IDHash, ctx context.Context) *ClientSession {

	session := &ClientSession{
		isAEAD: isAEAD,
		idHash: idHash,
	}

	randomBytes := make([]byte, 33) // 16 + 16 + 1
	common.Must2(rand.Read(randomBytes))
	copy(session.requestBodyKey[:], randomBytes[:16])
	copy(session.requestBodyIV[:], randomBytes[16:32])
	session.responseHeader = randomBytes[32]

	if !session.isAEAD {
		session.responseBodyKey = md5.Sum(session.requestBodyKey[:])
		session.responseBodyIV = md5.Sum(session.requestBodyIV[:])
	} else {
		BodyKey := sha256.Sum256(session.requestBodyKey[:])
		copy(session.responseBodyKey[:], BodyKey[:16])
		BodyIV := sha256.Sum256(session.requestBodyIV[:])
		copy(session.responseBodyIV[:], BodyIV[:16])
	}

	return session
}

```

This function appears to be used to generate a paddinger for a command and an option in a Mux network. It takes the request body and response headers from the incoming message and performs the following steps:

1. Validates the input.
2. Constructs a paddinger header by眼的忙乱长度、零和固定长度报文，然后用所得的报文对请求和选项数据进行哈希计算，得到哈希值，作为padingLen的值，将哈希值和部分前导字节存储到 buffer中。
3. 如果头部标志为4，则是类型 2 报文，直接对传入的报文进行编码，然后进行编码。
4. 计算 padingLen 的值，使用 dice.Roll 函数，并且将哈希值和 padingLen 的值作为 xor 运算的对象，使用两个不同时间的哈希函数异或它们的结果，得到一个结果作为 padingLen 的值，将哈希值作为余数，padingLen作为二进制数。
5. 从请求头部中提取出地址和端口，将地址和端口与传入的地址和端口组成一个 address 和 port的二进制字符串，然后使用 addressParser.WriteAddressPort 函数，如果执行失败，返回一个错误。
6. 如果 padingLen 的值为0，则执行以下操作：
	* 使用 fnv1a 新建一个 32 字节的哈希值，并将其所有字节复制进 buffer。
	* 使用支


```go
func (c *ClientSession) EncodeRequestHeader(header *protocol.RequestHeader, writer io.Writer) error {
	timestamp := protocol.NewTimestampGenerator(protocol.NowTime(), 30)()
	account := header.User.Account.(*vmess.MemoryAccount)
	if !c.isAEAD {
		idHash := c.idHash(account.AnyValidID().Bytes())
		common.Must2(serial.WriteUint64(idHash, uint64(timestamp)))
		common.Must2(writer.Write(idHash.Sum(nil)))
	}

	buffer := buf.New()
	defer buffer.Release()

	common.Must(buffer.WriteByte(Version))
	common.Must2(buffer.Write(c.requestBodyIV[:]))
	common.Must2(buffer.Write(c.requestBodyKey[:]))
	common.Must(buffer.WriteByte(c.responseHeader))
	common.Must(buffer.WriteByte(byte(header.Option)))

	padingLen := dice.Roll(16)
	security := byte(padingLen<<4) | byte(header.Security)
	common.Must2(buffer.Write([]byte{security, byte(0), byte(header.Command)}))

	if header.Command != protocol.RequestCommandMux {
		if err := addrParser.WriteAddressPort(buffer, header.Address, header.Port); err != nil {
			return newError("failed to writer address and port").Base(err)
		}
	}

	if padingLen > 0 {
		common.Must2(buffer.ReadFullFrom(rand.Reader, int32(padingLen)))
	}

	{
		fnv1a := fnv.New32a()
		common.Must2(fnv1a.Write(buffer.Bytes()))
		hashBytes := buffer.Extend(int32(fnv1a.Size()))
		fnv1a.Sum(hashBytes[:0])
	}

	if !c.isAEAD {
		iv := hashTimestamp(md5.New(), timestamp)
		aesStream := crypto.NewAesEncryptionStream(account.ID.CmdKey(), iv[:])
		aesStream.XORKeyStream(buffer.Bytes(), buffer.Bytes())
		common.Must2(writer.Write(buffer.Bytes()))
	} else {
		var fixedLengthCmdKey [16]byte
		copy(fixedLengthCmdKey[:], account.ID.CmdKey())
		vmessout := vmessaead.SealVMessAEADHeader(fixedLengthCmdKey, buffer.Bytes())
		common.Must2(io.Copy(writer, bytes.NewReader(vmessout)))
	}

	return nil
}

```

This function appears to be a method for creating a OptionChunkStream for a specific cryptography protocol.

It takes a request object that includes a `requestBody` field, which contains the data to be included in the stream, and a `transferType` field, which specifies the type of transfer being made (e.g.



```go
func (c *ClientSession) EncodeRequestBody(request *protocol.RequestHeader, writer io.Writer) buf.Writer {
	var sizeParser crypto.ChunkSizeEncoder = crypto.PlainChunkSizeParser{}
	if request.Option.Has(protocol.RequestOptionChunkMasking) {
		sizeParser = NewShakeSizeParser(c.requestBodyIV[:])
	}
	var padding crypto.PaddingLengthGenerator
	if request.Option.Has(protocol.RequestOptionGlobalPadding) {
		padding = sizeParser.(crypto.PaddingLengthGenerator)
	}

	switch request.Security {
	case protocol.SecurityType_NONE:
		if request.Option.Has(protocol.RequestOptionChunkStream) {
			if request.Command.TransferType() == protocol.TransferTypeStream {
				return crypto.NewChunkStreamWriter(sizeParser, writer)
			}
			auth := &crypto.AEADAuthenticator{
				AEAD:                    new(NoOpAuthenticator),
				NonceGenerator:          crypto.GenerateEmptyBytes(),
				AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
			}
			return crypto.NewAuthenticationWriter(auth, sizeParser, writer, protocol.TransferTypePacket, padding)
		}

		return buf.NewWriter(writer)
	case protocol.SecurityType_LEGACY:
		aesStream := crypto.NewAesEncryptionStream(c.requestBodyKey[:], c.requestBodyIV[:])
		cryptionWriter := crypto.NewCryptionWriter(aesStream, writer)
		if request.Option.Has(protocol.RequestOptionChunkStream) {
			auth := &crypto.AEADAuthenticator{
				AEAD:                    new(FnvAuthenticator),
				NonceGenerator:          crypto.GenerateEmptyBytes(),
				AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
			}
			return crypto.NewAuthenticationWriter(auth, sizeParser, cryptionWriter, request.Command.TransferType(), padding)
		}

		return &buf.SequentialWriter{Writer: cryptionWriter}
	case protocol.SecurityType_AES128_GCM:
		aead := crypto.NewAesGcm(c.requestBodyKey[:])
		auth := &crypto.AEADAuthenticator{
			AEAD:                    aead,
			NonceGenerator:          GenerateChunkNonce(c.requestBodyIV[:], uint32(aead.NonceSize())),
			AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
		}
		return crypto.NewAuthenticationWriter(auth, sizeParser, writer, request.Command.TransferType(), padding)
	case protocol.SecurityType_CHACHA20_POLY1305:
		aead, err := chacha20poly1305.New(GenerateChacha20Poly1305Key(c.requestBodyKey[:]))
		common.Must(err)

		auth := &crypto.AEADAuthenticator{
			AEAD:                    aead,
			NonceGenerator:          GenerateChunkNonce(c.requestBodyIV[:], uint32(aead.NonceSize())),
			AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
		}
		return crypto.NewAuthenticationWriter(auth, sizeParser, writer, request.Command.TransferType(), padding)
	default:
		panic("Unknown security type.")
	}
}

```

This is a Go function that takes a incoming buffer `buffer` and a response header `responseHeader` and returns the decrypted response header `header`.

The function reads the response header data from the buffer and decrypts it using the AES encryption algorithm if the `isAEAD` flag is set to true. Then, it returns the decrypted response header.

The function first checks if the buffer has a valid response header by checking if the first two bytes are the `responseHeader` and if the third byte is zero. If the response header is not a valid one, an error is thrown.

The function then creates a new buffer `c.responseReader` and reads the response header data into it. If there is any error, an error is thrown.

The function then decrypts the buffer data using the AES encryption algorithm if the `isAEAD` flag is set to true. Then, it writes the decrypted data to the `c.responseReader`.

Finally, the function returns the decrypted response header or an error if an error occurred.


```go
func (c *ClientSession) DecodeResponseHeader(reader io.Reader) (*protocol.ResponseHeader, error) {
	if !c.isAEAD {
		aesStream := crypto.NewAesDecryptionStream(c.responseBodyKey[:], c.responseBodyIV[:])
		c.responseReader = crypto.NewCryptionReader(aesStream, reader)
	} else {
		aeadResponseHeaderLengthEncryptionKey := vmessaead.KDF16(c.responseBodyKey[:], vmessaead.KDFSaltConst_AEADRespHeaderLenKey)
		aeadResponseHeaderLengthEncryptionIV := vmessaead.KDF(c.responseBodyIV[:], vmessaead.KDFSaltConst_AEADRespHeaderLenIV)[:12]

		aeadResponseHeaderLengthEncryptionKeyAESBlock := common.Must2(aes.NewCipher(aeadResponseHeaderLengthEncryptionKey)).(cipher.Block)
		aeadResponseHeaderLengthEncryptionAEAD := common.Must2(cipher.NewGCM(aeadResponseHeaderLengthEncryptionKeyAESBlock)).(cipher.AEAD)

		var aeadEncryptedResponseHeaderLength [18]byte
		var decryptedResponseHeaderLength int
		var decryptedResponseHeaderLengthBinaryDeserializeBuffer uint16

		if _, err := io.ReadFull(reader, aeadEncryptedResponseHeaderLength[:]); err != nil {
			return nil, newError("Unable to Read Header Len").Base(err)
		}
		if decryptedResponseHeaderLengthBinaryBuffer, err := aeadResponseHeaderLengthEncryptionAEAD.Open(nil, aeadResponseHeaderLengthEncryptionIV, aeadEncryptedResponseHeaderLength[:], nil); err != nil {
			return nil, newError("Failed To Decrypt Length").Base(err)
		} else {
			common.Must(binary.Read(bytes.NewReader(decryptedResponseHeaderLengthBinaryBuffer), binary.BigEndian, &decryptedResponseHeaderLengthBinaryDeserializeBuffer))
			decryptedResponseHeaderLength = int(decryptedResponseHeaderLengthBinaryDeserializeBuffer)
		}

		aeadResponseHeaderPayloadEncryptionKey := vmessaead.KDF16(c.responseBodyKey[:], vmessaead.KDFSaltConst_AEADRespHeaderPayloadKey)
		aeadResponseHeaderPayloadEncryptionIV := vmessaead.KDF(c.responseBodyIV[:], vmessaead.KDFSaltConst_AEADRespHeaderPayloadIV)[:12]

		aeadResponseHeaderPayloadEncryptionKeyAESBlock := common.Must2(aes.NewCipher(aeadResponseHeaderPayloadEncryptionKey)).(cipher.Block)
		aeadResponseHeaderPayloadEncryptionAEAD := common.Must2(cipher.NewGCM(aeadResponseHeaderPayloadEncryptionKeyAESBlock)).(cipher.AEAD)

		encryptedResponseHeaderBuffer := make([]byte, decryptedResponseHeaderLength+16)

		if _, err := io.ReadFull(reader, encryptedResponseHeaderBuffer); err != nil {
			return nil, newError("Unable to Read Header Data").Base(err)
		}

		if decryptedResponseHeaderBuffer, err := aeadResponseHeaderPayloadEncryptionAEAD.Open(nil, aeadResponseHeaderPayloadEncryptionIV, encryptedResponseHeaderBuffer, nil); err != nil {
			return nil, newError("Failed To Decrypt Payload").Base(err)
		} else {
			c.responseReader = bytes.NewReader(decryptedResponseHeaderBuffer)
		}
	}

	buffer := buf.StackNew()
	defer buffer.Release()

	if _, err := buffer.ReadFullFrom(c.responseReader, 4); err != nil {
		return nil, newError("failed to read response header").Base(err).AtWarning()
	}

	if buffer.Byte(0) != c.responseHeader {
		return nil, newError("unexpected response header. Expecting ", int(c.responseHeader), " but actually ", int(buffer.Byte(0)))
	}

	header := &protocol.ResponseHeader{
		Option: bitmask.Byte(buffer.Byte(1)),
	}

	if buffer.Byte(2) != 0 {
		cmdID := buffer.Byte(2)
		dataLen := int32(buffer.Byte(3))

		buffer.Clear()
		if _, err := buffer.ReadFullFrom(c.responseReader, dataLen); err != nil {
			return nil, newError("failed to read response command").Base(err)
		}
		command, err := UnmarshalCommand(cmdID, buffer.Bytes())
		if err == nil {
			header.Command = command
		}
	}
	if c.isAEAD {
		aesStream := crypto.NewAesDecryptionStream(c.responseBodyKey[:], c.responseBodyIV[:])
		c.responseReader = crypto.NewCryptionReader(aesStream, reader)
	}
	return header, nil
}

```

This function appears to determine whether a chunked request is for a secure or unsecure transfer and returns the appropriate authentication reader.

It first checks if the request is for a secure transfer and if the `Option` for the `RequestOptionChunkStream` field is set to `true`. If both conditions are met, it creates an `AEADAuthenticator` and returns it.

If the request is for an unsecure transfer, it defaults to `GenerateChunkNonce` and `GenerateChunkNonce` for `AEADAuthenticator` instead of `GenerateChunkNonce` for `crypto.AEADAuthenticator`.

It then creates a new `AEADAuthenticator` and returns it.

It appears that the function uses a combination of the security type and the chunked transfer type to determine the appropriate authentication reader and returns it.


```go
func (c *ClientSession) DecodeResponseBody(request *protocol.RequestHeader, reader io.Reader) buf.Reader {
	var sizeParser crypto.ChunkSizeDecoder = crypto.PlainChunkSizeParser{}
	if request.Option.Has(protocol.RequestOptionChunkMasking) {
		sizeParser = NewShakeSizeParser(c.responseBodyIV[:])
	}
	var padding crypto.PaddingLengthGenerator
	if request.Option.Has(protocol.RequestOptionGlobalPadding) {
		padding = sizeParser.(crypto.PaddingLengthGenerator)
	}

	switch request.Security {
	case protocol.SecurityType_NONE:
		if request.Option.Has(protocol.RequestOptionChunkStream) {
			if request.Command.TransferType() == protocol.TransferTypeStream {
				return crypto.NewChunkStreamReader(sizeParser, reader)
			}

			auth := &crypto.AEADAuthenticator{
				AEAD:                    new(NoOpAuthenticator),
				NonceGenerator:          crypto.GenerateEmptyBytes(),
				AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
			}

			return crypto.NewAuthenticationReader(auth, sizeParser, reader, protocol.TransferTypePacket, padding)
		}

		return buf.NewReader(reader)
	case protocol.SecurityType_LEGACY:
		if request.Option.Has(protocol.RequestOptionChunkStream) {
			auth := &crypto.AEADAuthenticator{
				AEAD:                    new(FnvAuthenticator),
				NonceGenerator:          crypto.GenerateEmptyBytes(),
				AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
			}
			return crypto.NewAuthenticationReader(auth, sizeParser, c.responseReader, request.Command.TransferType(), padding)
		}

		return buf.NewReader(c.responseReader)
	case protocol.SecurityType_AES128_GCM:
		aead := crypto.NewAesGcm(c.responseBodyKey[:])

		auth := &crypto.AEADAuthenticator{
			AEAD:                    aead,
			NonceGenerator:          GenerateChunkNonce(c.responseBodyIV[:], uint32(aead.NonceSize())),
			AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
		}
		return crypto.NewAuthenticationReader(auth, sizeParser, reader, request.Command.TransferType(), padding)
	case protocol.SecurityType_CHACHA20_POLY1305:
		aead, _ := chacha20poly1305.New(GenerateChacha20Poly1305Key(c.responseBodyKey[:]))

		auth := &crypto.AEADAuthenticator{
			AEAD:                    aead,
			NonceGenerator:          GenerateChunkNonce(c.responseBodyIV[:], uint32(aead.NonceSize())),
			AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
		}
		return crypto.NewAuthenticationReader(auth, sizeParser, reader, request.Command.TransferType(), padding)
	default:
		panic("Unknown security type.")
	}
}

```

该函数的目的是生成一个非ce字节的 chunk，并返回一个可以从中读取字节数据的 bytes.Buffer 对象。

函数的参数 nonce 和 size 分别表示要生成的非ce 字节数和生成的字节数据大小。函数内部首先将输入的 nonce 字节数添加到结果 bytes.Buffer 对象中，然后初始化一个计数器 count 为 0。

接着，函数返回一个可以返回字节数据的 bytes.Buffer 对象。在函数内部，首先将计数器 count 写入一个 16 字节整数类型的缓冲区，并对其进行 binary.BigEndian.PutUint16 函数的配置，这样写入的字节数组中从第一个元素开始，计数器 count 对应的偏移量从 0 开始。然后计数器 count 自增 1，此时缓冲区中有一个 16 字节整数类型的数据，其值为 count 的值。

最后，函数返回的结果是一个 bytes.Buffer 对象，其中包含生成的非ce 字节数据。


```go
func GenerateChunkNonce(nonce []byte, size uint32) crypto.BytesGenerator {
	c := append([]byte(nil), nonce...)
	count := uint16(0)
	return func() []byte {
		binary.BigEndian.PutUint16(c, count)
		count++
		return c[:size]
	}
}

```