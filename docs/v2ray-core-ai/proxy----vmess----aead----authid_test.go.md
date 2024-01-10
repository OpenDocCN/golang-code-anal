# `v2ray-core\proxy\vmess\aead\authid_test.go`

```
package aead

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "github.com/stretchr/testify/assert"  // 导入 testify 包中的 assert 模块，用于断言测试结果
    "strconv"  // 导入 strconv 包，用于字符串和基本数据类型之间的转换
    "testing"  // 导入 testing 包，用于编写测试函数
    "time"  // 导入 time 包，用于处理时间相关操作
)

func TestCreateAuthID(t *testing.T) {
    key := KDF16([]byte("Demo Key for Auth ID Test"), "Demo Path for Auth ID Test")  // 调用 KDF16 函数生成密钥
    authid := CreateAuthID(key, time.Now().Unix())  // 调用 CreateAuthID 函数生成认证 ID

    fmt.Println(key)  // 打印密钥
    fmt.Println(authid)  // 打印认证 ID
}

func TestCreateAuthIDAndDecode(t *testing.T) {
    key := KDF16([]byte("Demo Key for Auth ID Test"), "Demo Path for Auth ID Test")  // 调用 KDF16 函数生成密钥
    authid := CreateAuthID(key, time.Now().Unix())  // 调用 CreateAuthID 函数生成认证 ID

    fmt.Println(key)  // 打印密钥
    fmt.Println(authid)  // 打印认证 ID

    AuthDecoder := NewAuthIDDecoderHolder()  // 创建 AuthIDDecoderHolder 对象
    var keyw [16]byte  // 声明一个长度为 16 的字节数组
    copy(keyw[:], key)  // 将生成的密钥复制到字节数组中
    AuthDecoder.AddUser(keyw, "Demo User")  // 向 AuthIDDecoderHolder 对象中添加用户
    res, err := AuthDecoder.Match(authid)  // 调用 Match 方法匹配认证 ID
    fmt.Println(res)  // 打印匹配结果
    fmt.Println(err)  // 打印错误信息
    assert.Equal(t, "Demo User", res)  // 使用断言判断匹配结果是否符合预期
    assert.Nil(t, err)  // 使用断言判断错误信息是否为空
}

func TestCreateAuthIDAndDecode2(t *testing.T) {
    key := KDF16([]byte("Demo Key for Auth ID Test"), "Demo Path for Auth ID Test")  // 调用 KDF16 函数生成密钥
    authid := CreateAuthID(key, time.Now().Unix())  // 调用 CreateAuthID 函数生成认证 ID

    fmt.Println(key)  // 打印密钥
    fmt.Println(authid)  // 打印认证 ID

    AuthDecoder := NewAuthIDDecoderHolder()  // 创建 AuthIDDecoderHolder 对象
    var keyw [16]byte  // 声明一个长度为 16 的字节数组
    copy(keyw[:], key)  // 将生成的密钥复制到字节数组中
    AuthDecoder.AddUser(keyw, "Demo User")  // 向 AuthIDDecoderHolder 对象中添加用户
    res, err := AuthDecoder.Match(authid)  // 调用 Match 方法匹配认证 ID
    fmt.Println(res)  // 打印匹配结果
    fmt.Println(err)  // 打印错误信息
    assert.Equal(t, "Demo User", res)  // 使用断言判断匹配结果是否符合预期
    assert.Nil(t, err)  // 使用断言判断错误信息是否为空

    key2 := KDF16([]byte("Demo Key for Auth ID Test2"), "Demo Path for Auth ID Test")  // 调用 KDF16 函数生成另一个密钥
    authid2 := CreateAuthID(key2, time.Now().Unix())  // 调用 CreateAuthID 函数生成另一个认证 ID

    res2, err2 := AuthDecoder.Match(authid2)  // 使用 AuthIDDecoderHolder 对象匹配另一个认证 ID
    assert.EqualError(t, err2, "user do not exist")  // 使用断言判断错误信息是否符合预期
    assert.Nil(t, res2)  // 使用断言判断匹配结果是否为空
}

func TestCreateAuthIDAndDecodeMassive(t *testing.T) {
    key := KDF16([]byte("Demo Key for Auth ID Test"), "Demo Path for Auth ID Test")  // 调用 KDF16 函数生成密钥
    authid := CreateAuthID(key, time.Now().Unix())  // 调用 CreateAuthID 函数生成认证 ID

    fmt.Println(key)  // 打印密钥
    fmt.Println(authid)  // 打印认证 ID

    AuthDecoder := NewAuthIDDecoderHolder()  // 创建 AuthIDDecoderHolder 对象
    var keyw [16]byte  // 声明一个长度为 16 的字节数组
    copy(keyw[:], key)  // 将生成的密钥复制到字节数组中
    AuthDecoder.AddUser(keyw, "Demo User")  // 向 AuthIDDecoderHolder 对象中添加用户
}
    // 使用 AuthDecoder 对象匹配 authid，返回结果和错误
    res, err := AuthDecoder.Match(authid)
    // 打印结果
    fmt.Println(res)
    // 打印错误
    fmt.Println(err)
    // 断言结果为 "Demo User"
    assert.Equal(t, "Demo User", res)
    // 断言错误为空
    assert.Nil(t, err)

    // 循环 0 到 10000，生成不同的 key2，并添加用户到 AuthDecoder 对象
    for i := 0; i <= 10000; i++ {
        // 生成 key2
        key2 := KDF16([]byte("Demo Key for Auth ID Test2"), "Demo Path for Auth ID Test", strconv.Itoa(i))
        // 创建长度为 16 的字节数组 keyw2，并将 key2 的内容复制进去
        var keyw2 [16]byte
        copy(keyw2[:], key2)
        // 添加用户到 AuthDecoder 对象
        AuthDecoder.AddUser(keyw2, "Demo User"+strconv.Itoa(i))
    }

    // 创建 authid3
    authid3 := CreateAuthID(key, time.Now().Unix())

    // 使用 AuthDecoder 对象匹配 authid3，返回结果和错误
    res2, err2 := AuthDecoder.Match(authid3)
    // 断言结果为 "Demo User"
    assert.Equal(t, "Demo User", res2)
    // 断言错误为空
    assert.Nil(t, err2)
func TestCreateAuthIDAndDecodeSuperMassive(t *testing.T) {
    // 生成密钥
    key := KDF16([]byte("Demo Key for Auth ID Test"), "Demo Path for Auth ID Test")
    // 创建认证ID
    authid := CreateAuthID(key, time.Now().Unix())

    // 打印密钥
    fmt.Println(key)
    // 打印认证ID
    fmt.Println(authid)

    // 创建认证ID解码器
    AuthDecoder := NewAuthIDDecoderHolder()
    var keyw [16]byte
    // 将密钥复制到数组中
    copy(keyw[:], key)
    // 添加用户到认证ID解码器中
    AuthDecoder.AddUser(keyw, "Demo User")
    // 匹配认证ID
    res, err := AuthDecoder.Match(authid)
    // 打印匹配结果
    fmt.Println(res)
    // 打印错误信息
    fmt.Println(err)
    // 断言匹配结果为"Demo User"
    assert.Equal(t, "Demo User", res)
    // 断言错误信息为空
    assert.Nil(t, err)

    // 循环添加大量用户到认证ID解码器中
    for i := 0; i <= 1000000; i++ {
        // 生成新的密钥
        key2 := KDF16([]byte("Demo Key for Auth ID Test2"), "Demo Path for Auth ID Test", strconv.Itoa(i))
        var keyw2 [16]byte
        // 将新密钥复制到数组中
        copy(keyw2[:], key2)
        // 添加用户到认证ID解码器中
        AuthDecoder.AddUser(keyw2, "Demo User"+strconv.Itoa(i))
    }

    // 创建新的认证ID
    authid3 := CreateAuthID(key, time.Now().Unix())

    // 记录开始时间
    before := time.Now()
    // 匹配新的认证ID
    res2, err2 := AuthDecoder.Match(authid3)
    // 记录结束时间
    after := time.Now()
    // 断言匹配结果为"Demo User"
    assert.Equal(t, "Demo User", res2)
    // 断言错误信息为空
    assert.Nil(t, err2)

    // 打印匹配耗时
    fmt.Println(after.Sub(before).Seconds())
}
```