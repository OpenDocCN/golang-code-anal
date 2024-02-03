# `v2ray-core\common\protocol\address_test.go`

```go
package protocol_test

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/google/go-cmp/cmp"  // 导入第三方包，用于比较数据

    "v2ray.com/core/common"  // 导入 v2ray 核心通用包
    "v2ray.com/core/common/buf"  // 导入 v2ray 核心通用缓冲包
    "v2ray.com/core/common/net"  // 导入 v2ray 核心通用网络包
    . "v2ray.com/core/common/protocol"  // 导入 v2ray 核心通用协议包
)

func TestAddressReading(t *testing.T) {
    data := []struct {  // 定义结构体切片 data
        Options []AddressOption  // 结构体字段 Options 为 AddressOption 切片
        Input   []byte  // 结构体字段 Input 为字节切片
        Address net.Address  // 结构体字段 Address 为网络地址
        Port    net.Port  // 结构体字段 Port 为网络端口
        Error   bool  // 结构体字段 Error 为布尔值
    }

    for _, tc := range data {  // 遍历结构体切片 data
        b := buf.New()  // 创建新的缓冲区
        parser := NewAddressParser(tc.Options...)  // 使用 AddressOption 创建新的地址解析器
        addr, port, err := parser.ReadAddressPort(b, bytes.NewReader(tc.Input))  // 从缓冲区和输入数据中读取地址和端口
        b.Release()  // 释放缓冲区
        if tc.Error {  // 如果预期有错误
            if err == nil {  // 如果没有错误
                t.Errorf("Expect error but not: %v", tc)  // 输出错误信息
            }
        } else {  // 如果不预期有错误
            if err != nil {  // 如果有错误
                t.Errorf("Expect no error but: %s %v", err.Error(), tc)  // 输出错误信息
            }

            if addr != tc.Address {  // 如果读取的地址不等于预期地址
                t.Error("Got address ", addr.String(), " want ", tc.Address.String())  // 输出错误信息
            }

            if tc.Port != port {  // 如果读取的端口不等于预期端口
                t.Error("Got port ", port, " want ", tc.Port)  // 输出错误信息
            }
        }
    }
}

func TestAddressWriting(t *testing.T) {
    data := []struct {  // 定义结构体切片 data
        Options []AddressOption  // 结构体字段 Options 为 AddressOption 切片
        Address net.Address  // 结构体字段 Address 为网络地址
        Port    net.Port  // 结构体字段 Port 为网络端口
        Bytes   []byte  // 结构体字段 Bytes 为字节切片
        Error   bool  // 结构体字段 Error 为布尔值
    }{
        {
            Options: []AddressOption{AddressFamilyByte(0x01, net.AddressFamilyIPv4)},  // 设置 Options 字段为指定的 AddressOption 切片
            Address: net.LocalHostIP,  // 设置 Address 字段为本地主机 IP 地址
            Port:    net.Port(80),  // 设置 Port 字段为 80
            Bytes:   []byte{1, 127, 0, 0, 1, 0, 80},  // 设置 Bytes 字段为指定的字节切片
        },
    }
    # 遍历数据集合中的每个测试用例
    for _, tc := range data:
        # 根据给定的选项创建地址解析器
        parser := NewAddressParser(tc.Options...)

        # 创建一个新的缓冲区
        b := buf.New()
        
        # 使用地址解析器将地址和端口写入缓冲区
        err := parser.WriteAddressPort(b, tc.Address, tc.Port)
        
        # 如果期望出现错误
        if tc.Error:
            # 如果错误为nil，则输出错误信息
            if err == nil:
                t.Error("Expect error but nil")
        # 如果不期望出现错误
        else:
            # 必须处理错误
            common.Must(err)
            # 比较测试用例的字节数据和缓冲区的字节数据，如果不一致则输出错误信息
            if diff := cmp.Diff(tc.Bytes, b.Bytes()); diff != "":
                t.Error(err)
# 对 IPv4 地址读取性能进行基准测试
func BenchmarkAddressReadingIPv4(b *testing.B):
    # 创建地址解析器，指定地址族为 IPv4
    parser := NewAddressParser(AddressFamilyByte(0x01, net.AddressFamilyIPv4))
    # 创建缓冲区
    cache := buf.New()
    # 在函数返回时释放缓冲区
    defer cache.Release()

    # 创建负载数据缓冲区
    payload := buf.New()
    # 在函数返回时释放负载数据缓冲区
    defer payload.Release()

    # 定义原始数据
    raw := []byte{1, 0, 0, 0, 0, 0, 53}
    # 将原始数据写入负载数据缓冲区
    payload.Write(raw)

    # 重置计时器
    b.ResetTimer()
    # 循环执行基准测试
    for i := 0; i < b.N; i++:
        # 读取地址和端口
        _, _, err := parser.ReadAddressPort(cache, payload)
        # 检查错误
        common.Must(err)
        # 清空缓冲区
        cache.Clear()
        # 清空负载数据缓冲区
        payload.Clear()
        # 扩展负载数据缓冲区大小
        payload.Extend(int32(len(raw)))

# 对 IPv6 地址读取性能进行基准测试
func BenchmarkAddressReadingIPv6(b *testing.B):
    # 创建地址解析器，指定地址族为 IPv6
    parser := NewAddressParser(AddressFamilyByte(0x04, net.AddressFamilyIPv6))
    # 创建缓冲区
    cache := buf.New()
    # 在函数返回时释放缓冲区
    defer cache.Release()

    # 创建负载数据缓冲区
    payload := buf.New()
    # 在函数返回时释放负载数据缓冲区
    defer payload.Release()

    # 定义原始数据
    raw := []byte{4, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 0, 80}
    # 将原始数据写入负载数据缓冲区
    payload.Write(raw)

    # 重置计时器
    b.ResetTimer()
    # 循环执行基准测试
    for i := 0; i < b.N; i++:
        # 读取地址和端口
        _, _, err := parser.ReadAddressPort(cache, payload)
        # 检查错误
        common.Must(err)
        # 清空缓冲区
        cache.Clear()
        # 清空负载数据缓冲区
        payload.Clear()
        # 扩展负载数据缓冲区大小
        payload.Extend(int32(len(raw)))

# 对域名地址读取性能进行基准测试
func BenchmarkAddressReadingDomain(b *testing.B):
    # 创建地址解析器，指定地址族为域名
    parser := NewAddressParser(AddressFamilyByte(0x03, net.AddressFamilyDomain))
    # 创建缓冲区
    cache := buf.New()
    # 在函数返回时释放缓冲区
    defer cache.Release()

    # 创建负载数据缓冲区
    payload := buf.New()
    # 在函数返回时释放负载数据缓冲区
    defer payload.Release()

    # 定义原始数据
    raw := []byte{3, 9, 118, 50, 114, 97, 121, 46, 99, 111, 109, 0, 80}
    # 将原始数据写入负载数据缓冲区
    payload.Write(raw)

    # 重置计时器
    b.ResetTimer()
    # 循环执行基准测试
    for i := 0; i < b.N; i++:
        # 读取地址和端口
        _, _, err := parser.ReadAddressPort(cache, payload)
        # 检查错误
        common.Must(err)
        # 清空缓冲区
        cache.Clear()
        # 清空负载数据缓冲区
        payload.Clear()
        # 扩展负载数据缓冲区大小
        payload.Extend(int32(len(raw)))

# 对 IPv4 地址写入性能进行基准测试
func BenchmarkAddressWritingIPv4(b *testing.B):
    # 创建地址解析器，指定地址族为 IPv4
    parser := NewAddressParser(AddressFamilyByte(0x01, net.AddressFamilyIPv4))
    # 创建缓冲区
    writer := buf.New()
    # 在函数返回时释放缓冲区
    defer writer.Release()

    # 重置计时器
    b.ResetTimer()
    # 循环执行基准测试
    for i := 0; i < b.N; i++:
        # 写入地址和端口
        common.Must(parser.WriteAddressPort(writer, net.LocalHostIP, net.Port(80)))
        # 清空缓冲区
        writer.Clear()
# 用于测试 IPv6 地址写入性能的基准测试函数
func BenchmarkAddressWritingIPv6(b *testing.B) {
    # 创建一个新的地址解析器，指定地址族为 IPv6
    parser := NewAddressParser(AddressFamilyByte(0x04, net.AddressFamilyIPv6))
    # 创建一个新的缓冲区写入器
    writer := buf.New()
    # 在函数返回时释放写入器资源
    defer writer.Release()

    # 重置计时器
    b.ResetTimer()
    # 循环执行 b.N 次
    for i := 0; i < b.N; i++ {
        # 强制写入地址和端口到写入器
        common.Must(parser.WriteAddressPort(writer, net.LocalHostIPv6, net.Port(80)))
        # 清空写入器
        writer.Clear()
    }
}

# 用于测试域名地址写入性能的基准测试函数
func BenchmarkAddressWritingDomain(b *testing.B) {
    # 创建一个新的地址解析器，指定地址族为域名
    parser := NewAddressParser(AddressFamilyByte(0x02, net.AddressFamilyDomain))
    # 创建一个新的缓冲区写入器
    writer := buf.New()
    # 在函数返回时释放写入器资源
    defer writer.Release()

    # 重置计时器
    b.ResetTimer()
    # 循环执行 b.N 次
    for i := 0; i < b.N; i++ {
        # 强制写入地址和端口到写入器
        common.Must(parser.WriteAddressPort(writer, net.DomainAddress("www.v2ray.com"), net.Port(80)))
        # 清空写入器
        writer.Clear()
    }
}
```