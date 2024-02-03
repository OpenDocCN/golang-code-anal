# `v2ray-core\proxy\shadowsocks\protocol_test.go`

```go
package shadowsocks_test

import (
    "testing"

    "github.com/google/go-cmp/cmp"

    "v2ray.com/core/common"
    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol"
    . "v2ray.com/core/proxy/shadowsocks"
)

func toAccount(a *Account) protocol.Account {
    // 将 Account 对象转换为 protocol.Account 对象
    account, err := a.AsAccount()
    common.Must(err)
    return account
}

func TestUDPEncoding(t *testing.T) {
    // 创建一个测试用的请求头
    request := &protocol.RequestHeader{
        Version: Version,
        Command: protocol.RequestCommandUDP,
        Address: net.LocalHostIP,
        Port:    1234,
        User: &protocol.MemoryUser{
            Email: "love@v2ray.com",
            Account: toAccount(&Account{
                Password:   "shadowsocks-password",
                CipherType: CipherType_AES_128_CFB,
            }),
        },
    }

    // 创建一个新的数据缓冲区
    data := buf.New()
    // 向数据缓冲区写入字符串
    common.Must2(data.WriteString("test string"))
    // 对请求头和数据进行 UDP 数据包编码
    encodedData, err := EncodeUDPPacket(request, data.Bytes())
    common.Must(err)

    // 对 UDP 数据包进行解码
    decodedRequest, decodedData, err := DecodeUDPPacket(request.User, encodedData)
    common.Must(err)

    // 检查解码后的数据是否与原始数据一致
    if r := cmp.Diff(decodedData.Bytes(), data.Bytes()); r != "" {
        t.Error("data: ", r)
    }

    // 检查解码后的请求头是否与原始请求头一致
    if r := cmp.Diff(decodedRequest, request); r != "" {
        t.Error("request: ", r)
    }
}

func TestTCPRequest(t *testing.T) {
    cases := []struct {
        request *protocol.RequestHeader
        payload []byte
    # 第一个请求对象，包含请求头和负载数据
    {
        # 请求头对象，包含版本、命令、地址、端口和用户信息
        request: &protocol.RequestHeader{
            Version: Version,
            Command: protocol.RequestCommandTCP,
            Address: net.LocalHostIP,
            Port:    1234,
            User: &protocol.MemoryUser{
                Email: "love@v2ray.com",
                Account: toAccount(&Account{
                    Password:   "tcp-password",
                    CipherType: CipherType_CHACHA20,
                }),
            },
        },
        # 负载数据，以字节数组形式表示
        payload: []byte("test string"),
    },
    # 第二个请求对象，包含请求头和负载数据
    {
        # 请求头对象，包含版本、命令、地址、端口和用户信息
        request: &protocol.RequestHeader{
            Version: Version,
            Command: protocol.RequestCommandTCP,
            Address: net.LocalHostIPv6,
            Port:    1234,
            User: &protocol.MemoryUser{
                Email: "love@v2ray.com",
                Account: toAccount(&Account{
                    Password:   "password",
                    CipherType: CipherType_AES_256_CFB,
                }),
            },
        },
        # 负载数据，以字节数组形式表示
        payload: []byte("test string"),
    },
    # 第三个请求对象，包含请求头和负载数据
    {
        # 请求头对象，包含版本、命令、地址、端口和用户信息
        request: &protocol.RequestHeader{
            Version: Version,
            Command: protocol.RequestCommandTCP,
            Address: net.DomainAddress("v2ray.com"),
            Port:    1234,
            User: &protocol.MemoryUser{
                Email: "love@v2ray.com",
                Account: toAccount(&Account{
                    Password:   "password",
                    CipherType: CipherType_CHACHA20_IETF,
                }),
            },
        },
        # 负载数据，以字节数组形式表示
        payload: []byte("test string"),
    },
}
    # 定义一个名为 runTest 的函数，接受一个请求头和一个数据载荷作为参数
    runTest := func(request *protocol.RequestHeader, payload []byte) {
        # 创建一个新的缓冲区
        data := buf.New()
        # 将数据载荷写入缓冲区
        common.Must2(data.Write(payload))
    
        # 创建一个新的缓冲区作为缓存
        cache := buf.New()
        # 延迟释放缓存
        defer cache.Release()
    
        # 使用 WriteTCPRequest 函数将请求头和缓存写入缓存中
        writer, err := WriteTCPRequest(request, cache)
        common.Must(err)
    
        # 使用 writer 将数据载荷写入缓存
        common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{data}))
    
        # 从缓存中读取 TCP 会话，并解码为请求头和读取器
        decodedRequest, reader, err := ReadTCPSession(request.User, cache)
        common.Must(err)
        # 如果解码后的请求头与原始请求头不同，则输出错误信息
        if r := cmp.Diff(decodedRequest, request); r != "" {
            t.Error("request: ", r)
        }
    
        # 从读取器中读取解码后的数据
        decodedData, err := reader.ReadMultiBuffer()
        common.Must(err)
        # 如果解码后的数据与原始数据不同，则输出错误信息
        if r := cmp.Diff(decodedData[0].Bytes(), payload); r != "" {
            t.Error("data: ", r)
        }
    }
    
    # 遍历测试用例数组，并对每个测试用例运行 runTest 函数
    for _, test := range cases {
        runTest(test.request, test.payload)
    }
func TestUDPReaderWriter(t *testing.T) {
    // 创建一个内存用户对象
    user := &protocol.MemoryUser{
        // 将账户信息转换为用户账户
        Account: toAccount(&Account{
            Password:   "test-password",
            CipherType: CipherType_CHACHA20_IETF,
        }),
    }
    // 创建一个缓冲区
    cache := buf.New()
    // 在函数返回时释放缓冲区
    defer cache.Release()

    // 创建一个顺序写入器，用于UDP写入
    writer := &buf.SequentialWriter{Writer: &UDPWriter{
        // 将数据写入缓冲区
        Writer: cache,
        // 创建一个请求头
        Request: &protocol.RequestHeader{
            Version: Version,
            Address: net.DomainAddress("v2ray.com"),
            Port:    123,
            User:    user,
        },
    }}

    // 创建一个UDP读取器
    reader := &UDPReader{
        // 从缓冲区读取数据
        Reader: cache,
        // 使用的用户
        User:   user,
    }

    {
        // 创建一个新的缓冲区
        b := buf.New()
        // 将字符串写入缓冲区
        common.Must2(b.WriteString("test payload"))
        // 将缓冲区的数据写入UDP连接
        common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{b}))

        // 读取UDP连接的数据
        payload, err := reader.ReadMultiBuffer()
        // 检查是否有错误发生
        common.Must(err)
        // 检查读取的数据是否符合预期
        if payload[0].String() != "test payload" {
            t.Error("unexpected output: ", payload[0].String())
        }
    }

    {
        // 创建一个新的缓冲区
        b := buf.New()
        // 将字符串写入缓冲区
        common.Must2(b.WriteString("test payload 2"))
        // 将缓冲区的数据写入UDP连接
        common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{b}))

        // 读取UDP连接的数据
        payload, err := reader.ReadMultiBuffer()
        // 检查是否有错误发生
        common.Must(err)
        // 检查读取的数据是否符合预期
        if payload[0].String() != "test payload 2" {
            t.Error("unexpected output: ", payload[0].String())
        }
    }
}
```