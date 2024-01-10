# `v2ray-core\proxy\blackhole\blackhole_test.go`

```
package blackhole_test

import (
    "context"  // 导入 context 包
    "testing"  // 导入 testing 包

    "v2ray.com/core/common"  // 导入 common 包
    "v2ray.com/core/common/buf"  // 导入 buf 包
    "v2ray.com/core/common/serial"  // 导入 serial 包
    "v2ray.com/core/proxy/blackhole"  // 导入 blackhole 包
    "v2ray.com/core/transport"  // 导入 transport 包
    "v2ray.com/core/transport/pipe"  // 导入 pipe 包
)

func TestBlackholeHTTPResponse(t *testing.T) {
    // 创建 blackhole handler
    handler, err := blackhole.New(context.Background(), &blackhole.Config{
        Response: serial.ToTypedMessage(&blackhole.HTTPResponse{}),
    })
    common.Must(err)  // 检查错误

    // 创建管道的读写端
    reader, writer := pipe.New(pipe.WithoutSizeLimit())

    var mb buf.MultiBuffer  // 声明一个多缓冲区变量
    var rerr error  // 声明一个错误变量
    go func() {
        b, e := reader.ReadMultiBuffer()  // 从管道中读取多缓冲区
        mb = b  // 将读取的多缓冲区赋值给变量 mb
        rerr = e  // 将读取过程中的错误赋值给变量 rerr
    }()

    // 创建传输链路
    link := transport.Link{
        Reader: reader,
        Writer: writer,
    }
    common.Must(handler.Process(context.Background(), &link, nil))  // 处理传输链路
    common.Must(rerr)  // 检查错误
    if mb.IsEmpty() {  // 如果多缓冲区为空
        t.Error("expect http response, but nothing")  // 输出错误信息
    }
}
```