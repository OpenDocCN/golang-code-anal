# `v2ray-core\proxy\trojan\protocol_test.go`

```
package trojan_test

import (
    "testing"

    "github.com/google/go-cmp/cmp"  // 导入用于比较的包
    "v2ray.com/core/common"  // 导入通用功能包
    "v2ray.com/core/common/buf"  // 导入缓冲区包
    "v2ray.com/core/common/net"  // 导入网络包
    "v2ray.com/core/common/protocol"  // 导入协议包
    . "v2ray.com/core/proxy/trojan"  // 导入 Trojan 代理包
)

func toAccount(a *Account) protocol.Account {
    account, err := a.AsAccount()  // 将 Account 转换为协议账户
    common.Must(err)  // 如果有错误则终止程序
    return account  // 返回账户
}

func TestTCPRequest(t *testing.T) {
    user := &protocol.MemoryUser{  // 创建内存用户
        Email: "love@v2ray.com",  // 设置邮箱
        Account: toAccount(&Account{  // 设置账户信息
            Password: "password",  // 设置密码
        }),
    }
    payload := []byte("test string")  // 设置测试数据
    data := buf.New()  // 创建新的缓冲区
    common.Must2(data.Write(payload))  // 将测试数据写入缓冲区

    buffer := buf.New()  // 创建新的缓冲区
    defer buffer.Release()  // 在函数返回时释放缓冲区

    destination := net.Destination{Network: net.Network_TCP, Address: net.LocalHostIP, Port: 1234}  // 设置目标地址
    writer := &ConnWriter{Writer: buffer, Target: destination, Account: user.Account.(*MemoryAccount)}  // 创建连接写入器
    common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{data}))  // 使用写入器写入数据

    reader := &ConnReader{Reader: buffer}  // 创建连接读取器
    common.Must(reader.ParseHeader())  // 解析头部信息

    if r := cmp.Diff(reader.Target, destination); r != "" {  // 比较读取器的目标地址和预期目标地址
        t.Error("destination: ", r)  // 输出错误信息
    }

    decodedData, err := reader.ReadMultiBuffer()  // 读取解码后的数据
    common.Must(err)  // 如果有错误则终止程序
    if r := cmp.Diff(decodedData[0].Bytes(), payload); r != "" {  // 比较解码后的数据和预期数据
        t.Error("data: ", r)  // 输出错误信息
    }
}

func TestUDPRequest(t *testing.T) {
    user := &protocol.MemoryUser{  // 创建内存用户
        Email: "love@v2ray.com",  // 设置邮箱
        Account: toAccount(&Account{  // 设置账户信息
            Password: "password",  // 设置密码
        }),
    }
    payload := []byte("test string")  // 设置测试数据
    data := buf.New()  // 创建新的缓冲区
    common.Must2(data.Write(payload))  // 将测试数据写入缓冲区

    buffer := buf.New()  // 创建新的缓冲区
    defer buffer.Release()  // 在函数返回时释放缓冲区

    destination := net.Destination{Network: net.Network_UDP, Address: net.LocalHostIP, Port: 1234}  // 设置目标地址
    writer := &PacketWriter{Writer: &ConnWriter{Writer: buffer, Target: destination, Account: user.Account.(*MemoryAccount)}, Target: destination}  // 创建数据包写入器
    common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{data}))  // 使用写入器写入数据
    // 创建一个 ConnReader 对象，用于从 buffer 中读取数据
    connReader := &ConnReader{Reader: buffer}
    // 解析 ConnReader 对象的头部信息
    common.Must(connReader.ParseHeader())

    // 创建一个 PacketReader 对象，用于从 connReader 中读取数据
    packetReader := &PacketReader{Reader: connReader}
    // 读取带有元数据的多缓冲区数据
    p, err := packetReader.ReadMultiBufferWithMetadata()
    common.Must(err)

    // 如果读取的数据为空，则输出错误信息
    if p.Buffer.IsEmpty() {
        t.Error("no request data")
    }

    // 比较读取的目标数据和目的地数据是否一致，如果不一致则输出错误信息
    if r := cmp.Diff(p.Target, destination); r != "" {
        t.Error("destination: ", r)
    }

    // 将读取的数据拆分成多个缓冲区，并释放缓冲区
    mb, decoded := buf.SplitFirst(p.Buffer)
    buf.ReleaseMulti(mb)

    // 比较解码后的数据和有效载荷数据是否一致，如果不一致则输出错误信息
    if r := cmp.Diff(decoded.Bytes(), payload); r != "" {
        t.Error("data: ", r)
    }
# 闭合前面的函数定义
```