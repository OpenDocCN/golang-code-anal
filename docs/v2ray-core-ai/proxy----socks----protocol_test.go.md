# `v2ray-core\proxy\socks\protocol_test.go`

```
package socks_test

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/google/go-cmp/cmp"  // 导入 cmp 包，用于比较数据

    "v2ray.com/core/common"  // 导入 common 包，V2Ray 核心通用功能
    "v2ray.com/core/common/buf"  // 导入 buf 包，V2Ray 核心缓冲区功能
    "v2ray.com/core/common/net"  // 导入 net 包，V2Ray 核心网络功能
    "v2ray.com/core/common/protocol"  // 导入 protocol 包，V2Ray 核心协议功能
    . "v2ray.com/core/proxy/socks"  // 导入 socks 包，并将其所有公开的符号都导入当前包的命名空间
)

func TestUDPEncoding(t *testing.T) {
    b := buf.New()  // 创建一个新的缓冲区

    request := &protocol.RequestHeader{  // 创建一个请求头对象
        Address: net.IPAddress([]byte{1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6}),  // 设置请求头的地址
        Port:    1024,  // 设置请求头的端口
    }
    writer := &buf.SequentialWriter{Writer: NewUDPWriter(request, b)}  // 创建一个顺序写入器，用于写入 UDP 数据

    content := []byte{'a'}  // 创建一个字节切片
    payload := buf.New()  // 创建一个新的缓冲区
    payload.Write(content)  // 将内容写入缓冲区
    common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{payload}))  // 使用 writer 写入缓冲区中的数据

    reader := NewUDPReader(b)  // 创建一个 UDP 读取器，用于读取 UDP 数据

    decodedPayload, err := reader.ReadMultiBuffer()  // 读取 UDP 数据
    common.Must(err)  // 检查是否有错误发生
    if r := cmp.Diff(decodedPayload[0].Bytes(), content); r != "" {  // 比较读取的数据和原始数据
        t.Error(r)  // 输出比较结果
    }
}

func TestReadUsernamePassword(t *testing.T) {
    testCases := []struct {  // 创建测试用例切片
        Input    []byte  // 输入字节切片
        Username string  // 用户名
        Password string  // 密码
        Error    bool  // 是否有错误
    }{
        {
            Input:    []byte{0x05, 0x01, 'a', 0x02, 'b', 'c'},  // 设置输入数据
            Username: "a",  // 设置用户名
            Password: "bc",  // 设置密码
        },
        {
            Input: []byte{0x05, 0x18, 'a', 0x02, 'b', 'c'},  // 设置输入数据
            Error: true,  // 标记有错误发生
        },
    }
    # 遍历测试用例切片
    for _, testCase := range testCases {
        # 根据测试用例的输入创建字节流读取器
        reader := bytes.NewReader(testCase.Input)
        # 调用 ReadUsernamePassword 函数读取用户名和密码，并返回可能的错误
        username, password, err := ReadUsernamePassword(reader)
        # 如果测试用例期望有错误
        if testCase.Error {
            # 如果实际错误为 nil，则输出错误信息
            if err == nil {
                t.Error("for input: ", testCase.Input, " expect error, but actually nil")
            }
        } else {
            # 如果测试用例期望没有错误
            if err != nil {
                # 如果实际错误不为 nil，则输出错误信息
                t.Error("for input: ", testCase.Input, " expect no error, but actually ", err.Error())
            }
            # 检查实际用户名和期望用户名是否一致，不一致则输出错误信息
            if testCase.Username != username {
                t.Error("for input: ", testCase.Input, " expect username ", testCase.Username, " but actually ", username)
            }
            # 检查实际密码和期望密码是否一致，不一致则输出错误信息
            if testCase.Password != password {
                t.Error("for input: ", testCase.Input, " expect passowrd ", testCase.Password, " but actually ", password)
            }
        }
    }
func TestReadUntilNull(t *testing.T) {
    // 定义测试用例，包括输入字节流、预期输出字符串和是否期望出错
    testCases := []struct {
        Input  []byte
        Output string
        Error  bool
    }{
        {
            Input:  []byte{'a', 'b', 0x00},
            Output: "ab",
        },
        {
            Input: []byte{'a'},
            Error: true,
        },
    }

    // 遍历测试用例
    for _, testCase := range testCases {
        // 创建字节流读取器
        reader := bytes.NewReader(testCase.Input)
        // 调用 ReadUntilNull 函数，获取返回值和错误
        value, err := ReadUntilNull(reader)
        // 检查是否期望出错
        if testCase.Error {
            // 如果期望出错但实际没有出错，输出错误信息
            if err == nil {
                t.Error("for input: ", testCase.Input, " expect error, but actually nil")
            }
        } else {
            // 如果不期望出错但实际出错，输出错误信息
            if err != nil {
                t.Error("for input: ", testCase.Input, " expect no error, but actually ", err.Error())
            }
            // 如果输出值与预期输出不符，输出错误信息
            if testCase.Output != value {
                t.Error("for input: ", testCase.Input, " expect output ", testCase.Output, " but actually ", value)
            }
        }
    }
}

func BenchmarkReadUsernamePassword(b *testing.B) {
    // 定义输入字节流
    input := []byte{0x05, 0x01, 'a', 0x02, 'b', 'c'}
    // 创建字节缓冲区
    buffer := buf.New()
    // 将输入写入缓冲区
    buffer.Write(input)

    // 重置计时器
    b.ResetTimer()
    // 循环执行基准测试
    for i := 0; i < b.N; i++ {
        // 调用 ReadUsernamePassword 函数，获取返回值和错误
        _, _, err := ReadUsernamePassword(buffer)
        // 确保没有错误发生
        common.Must(err)
        // 清空缓冲区
        buffer.Clear()
        // 扩展缓冲区大小
        buffer.Extend(int32(len(input)))
    }
}
```