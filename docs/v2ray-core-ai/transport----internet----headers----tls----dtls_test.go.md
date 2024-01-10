# `v2ray-core\transport\internet\headers\tls\dtls_test.go`

```
package tls_test

import (
    "context"  // 导入上下文包，用于处理请求的上下文信息
    "testing"  // 导入测试包，用于编写和运行测试函数

    "v2ray.com/core/common"  // 导入通用包，用于处理通用功能
    "v2ray.com/core/common/buf"  // 导入缓冲区包，用于处理数据缓冲区
    . "v2ray.com/core/transport/internet/headers/tls"  // 导入 TLS 头部包，用于处理 TLS 头部

)

func TestDTLSWrite(t *testing.T) {
    content := []byte{'a', 'b', 'c', 'd', 'e', 'f', 'g'}  // 定义一个包含字节的切片
    dtlsRaw, err := New(context.Background(), &PacketConfig{})  // 使用上下文和数据包配置创建一个新的 DTLS 对象
    common.Must(err)  // 检查错误，如果有错误则终止程序

    dtls := dtlsRaw.(*DTLS)  // 将 DTLS 原始对象转换为 DTLS 对象

    payload := buf.New()  // 创建一个新的缓冲区
    dtls.Serialize(payload.Extend(dtls.Size()))  // 将 DTLS 对象序列化到缓冲区中
    payload.Write(content)  // 将内容写入缓冲区

    if payload.Len() != int32(len(content))+dtls.Size() {  // 检查缓冲区长度是否符合预期
        t.Error("payload len: ", payload.Len(), " want ", int32(len(content))+dtls.Size())  // 输出错误信息
    }
}
```