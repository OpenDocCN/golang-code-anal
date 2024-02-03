# `v2ray-core\proxy\vmess\aead\encrypt.go`

```go
package aead

import (
    "bytes"  // 导入 bytes 包，用于操作字节切片
    "crypto/aes"  // 导入 aes 包，用于实现高级加密标准
    "crypto/cipher"  // 导入 cipher 包，用于实现加密和解密
    "crypto/rand"  // 导入 rand 包，用于生成随机数
    "encoding/binary"  // 导入 binary 包，用于对数据进行二进制编解码
    "io"  // 导入 io 包，用于进行 I/O 操作
    "time"  // 导入 time 包，用于处理时间

    "v2ray.com/core/common"  // 导入 common 包，用于通用功能
)

func SealVMessAEADHeader(key [16]byte, data []byte) []byte {
    generatedAuthID := CreateAuthID(key[:], time.Now().Unix())  // 生成认证 ID

    connectionNonce := make([]byte, 8)  // 创建长度为 8 的连接随机数切片
    if _, err := io.ReadFull(rand.Reader, connectionNonce); err != nil {  // 从随机数生成器中读取随机数到连接随机数切片
        panic(err.Error())  // 如果出错，抛出异常
    }

    aeadPayloadLengthSerializeBuffer := bytes.NewBuffer(nil)  // 创建一个字节缓冲区

    headerPayloadDataLen := uint16(len(data))  // 获取数据切片的长度，并转换为 uint16 类型

    common.Must(binary.Write(aeadPayloadLengthSerializeBuffer, binary.BigEndian, headerPayloadDataLen))  // 将数据长度写入字节缓冲区

    aeadPayloadLengthSerializedByte := aeadPayloadLengthSerializeBuffer.Bytes()  // 获取字节缓冲区中的字节切片
    var payloadHeaderLengthAEADEncrypted []byte  // 声明一个字节切片变量

    {
        payloadHeaderLengthAEADKey := KDF16(key[:], KDFSaltConst_VMessHeaderPayloadLengthAEADKey, string(generatedAuthID[:]), string(connectionNonce))  // 使用 KDF16 函数生成密钥

        payloadHeaderLengthAEADNonce := KDF(key[:], KDFSaltConst_VMessHeaderPayloadLengthAEADIV, string(generatedAuthID[:]), string(connectionNonce))[:12]  // 使用 KDF 函数生成随机数

        payloadHeaderLengthAEADAESBlock, err := aes.NewCipher(payloadHeaderLengthAEADKey)  // 创建 AES 加密块
        if err != nil {  // 如果出错
            panic(err.Error())  // 抛出异常
        }

        payloadHeaderAEAD, err := cipher.NewGCM(payloadHeaderLengthAEADAESBlock)  // 创建 GCM 加密块
        if err != nil {  // 如果出错
            panic(err.Error())  // 抛出异常
        }

        payloadHeaderLengthAEADEncrypted = payloadHeaderAEAD.Seal(nil, payloadHeaderLengthAEADNonce, aeadPayloadLengthSerializedByte, generatedAuthID[:])  // 使用 AEAD 加密算法对数据进行加密
    }

    var payloadHeaderAEADEncrypted []byte  // 声明一个字节切片变量
    {
        // 使用密钥派生函数生成用于加密负载头部的 AEAD 密钥
        payloadHeaderAEADKey := KDF16(key[:], KDFSaltConst_VMessHeaderPayloadAEADKey, string(generatedAuthID[:]), string(connectionNonce))
    
        // 使用密钥派生函数生成用于加密负载头部的 AEAD 随机数
        payloadHeaderAEADNonce := KDF(key[:], KDFSaltConst_VMessHeaderPayloadAEADIV, string(generatedAuthID[:]), string(connectionNonce))[:12]
    
        // 使用 AES 加密算法创建负载头部的 AEAD 密钥块
        payloadHeaderAEADAESBlock, err := aes.NewCipher(payloadHeaderAEADKey)
        if err != nil {
            panic(err.Error())
        }
    
        // 使用 AEAD 密钥块创建 AEAD 加密器
        payloadHeaderAEAD, err := cipher.NewGCM(payloadHeaderAEADAESBlock)
    
        if err != nil {
            panic(err.Error())
        }
    
        // 使用 AEAD 加密器对数据进行加密，生成加密后的负载头部
        payloadHeaderAEADEncrypted = payloadHeaderAEAD.Seal(nil, payloadHeaderAEADNonce, data, generatedAuthID[:])
    }
    
    // 创建一个新的字节缓冲区
    var outputBuffer = bytes.NewBuffer(nil)
    
    // 将生成的认证 ID 写入输出缓冲区（长度为 16）
    common.Must2(outputBuffer.Write(generatedAuthID[:]))
    
    // 将加密后的负载头部长度写入输出缓冲区（长度为 2+16）
    common.Must2(outputBuffer.Write(payloadHeaderLengthAEADEncrypted))
    
    // 将连接随机数写入输出缓冲区（长度为 8）
    common.Must2(outputBuffer.Write(connectionNonce))
    
    // 将加密后的负载头部写入输出缓冲区
    common.Must2(outputBuffer.Write(payloadHeaderAEADEncrypted))
    
    // 返回输出缓冲区的字节表示
    return outputBuffer.Bytes()
}
// 打开 VMess AEAD 头部，解密数据
func OpenVMessAEADHeader(key [16]byte, authid [16]byte, data io.Reader) ([]byte, bool, error, int) {
    var payloadHeaderLengthAEADEncrypted [18]byte  // 存储加密的负载头部长度
    var nonce [8]byte  // 存储加密的随机数

    var bytesRead int  // 存储读取的字节数

    authidCheckValueReadBytesCounts, err := io.ReadFull(data, payloadHeaderLengthAEADEncrypted[:])  // 从数据流中读取负载头部长度的加密值
    bytesRead += authidCheckValueReadBytesCounts  // 更新已读取的字节数
    if err != nil {
        return nil, false, err, bytesRead  // 如果读取出错，返回错误信息和已读取的字节数
    }

    nonceReadBytesCounts, err := io.ReadFull(data, nonce[:])  // 从数据流中读取随机数的加密值
    bytesRead += nonceReadBytesCounts  // 更新已读取的字节数
    if err != nil {
        return nil, false, err, bytesRead  // 如果读取出错，返回错误信息和已读取的字节数
    }

    // 解密长度

    var decryptedAEADHeaderLengthPayloadResult []byte  // 存储解密后的负载头部长度结果

    {
        payloadHeaderLengthAEADKey := KDF16(key[:], KDFSaltConst_VMessHeaderPayloadLengthAEADKey, string(authid[:]), string(nonce[:]))  // 使用密钥派生函数生成负载头部长度的 AEAD 密钥
        payloadHeaderLengthAEADNonce := KDF(key[:], KDFSaltConst_VMessHeaderPayloadLengthAEADIV, string(authid[:]), string(nonce[:]))[:12]  // 使用密钥派生函数生成负载头部长度的 AEAD 随机数

        payloadHeaderAEADAESBlock, err := aes.NewCipher(payloadHeaderLengthAEADKey)  // 创建负载头部长度的 AES 加密块
        if err != nil {
            panic(err.Error())  // 如果出错，抛出异常
        }

        payloadHeaderLengthAEAD, err := cipher.NewGCM(payloadHeaderAEADAESBlock)  // 创建负载头部长度的 GCM 加密块
        if err != nil {
            panic(err.Error())  // 如果出错，抛出异常
        }

        decryptedAEADHeaderLengthPayload, erropenAEAD := payloadHeaderLengthAEAD.Open(nil, payloadHeaderLengthAEADNonce, payloadHeaderLengthAEADEncrypted[:], authid[:])  // 使用 AEAD 解密负载头部长度的加密值
        if erropenAEAD != nil {
            return nil, true, erropenAEAD, bytesRead  // 如果解密出错，返回错误信息和已读取的字节数
        }

        decryptedAEADHeaderLengthPayloadResult = decryptedAEADHeaderLengthPayload  // 存储解密后的负载头部长度结果
    }

    var length uint16  // 存储负载头部长度

    common.Must(binary.Read(bytes.NewReader(decryptedAEADHeaderLengthPayloadResult[:]), binary.BigEndian, &length))  // 从解密后的负载头部长度结果中读取长度值

    var decryptedAEADHeaderPayloadR []byte  // 存储解密后的负载头部数据
    var payloadHeaderAEADEncryptedReadedBytesCounts int
}
    # 使用密钥派生函数（KDF）生成16字节的负载头部AEAD密钥
    payloadHeaderAEADKey := KDF16(key[:], KDFSaltConst_VMessHeaderPayloadAEADKey, string(authid[:]), string(nonce[:]))

    # 使用KDF生成负载头部AEAD的12字节随机数
    payloadHeaderAEADNonce := KDF(key[:], KDFSaltConst_VMessHeaderPayloadAEADIV, string(authid[:]), string(nonce[:]))[:12]

    # 创建一个长度为length+16的字节切片，用于存储负载头部AEAD加密后的数据
    payloadHeaderAEADEncrypted := make([]byte, length+16)

    # 从数据流中读取加密后的负载头部AEAD数据，并记录读取的字节数
    payloadHeaderAEADEncryptedReadedBytesCounts, err = io.ReadFull(data, payloadHeaderAEADEncrypted)
    bytesRead += payloadHeaderAEADEncryptedReadedBytesCounts
    if err != nil {
        return nil, false, err, bytesRead
    }

    # 使用AES算法创建负载头部AEAD密钥的块
    payloadHeaderAEADAESBlock, err := aes.NewCipher(payloadHeaderAEADKey)
    if err != nil {
        panic(err.Error())
    }

    # 使用GCM模式创建负载头部AEAD
    payloadHeaderAEAD, err := cipher.NewGCM(payloadHeaderAEADAESBlock)

    if err != nil {
        panic(err.Error())
    }

    # 解密负载头部AEAD加密后的数据
    decryptedAEADHeaderPayload, erropenAEAD := payloadHeaderAEAD.Open(nil, payloadHeaderAEADNonce, payloadHeaderAEADEncrypted, authid[:])

    # 如果解密出错，则返回错误信息和已读取的字节数
    if erropenAEAD != nil {
        return nil, true, erropenAEAD, bytesRead
    }

    # 返回解密后的负载头部AEAD数据、错误标志和nil，以及已读取的字节数
    decryptedAEADHeaderPayloadR = decryptedAEADHeaderPayload
    return decryptedAEADHeaderPayloadR, false, nil, bytesRead
# 闭合前面的函数定义
```