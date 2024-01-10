# `v2ray-core\common\mux\frame_test.go`

```
package mux_test

import (
    "testing"

    "v2ray.com/core/common" // 导入通用工具包
    "v2ray.com/core/common/buf" // 导入缓冲区工具包
    "v2ray.com/core/common/mux" // 导入多路复用工具包
    "v2ray.com/core/common/net" // 导入网络工具包
)

func BenchmarkFrameWrite(b *testing.B) {
    // 创建一个帧元数据对象
    frame := mux.FrameMetadata{
        Target:        net.TCPDestination(net.DomainAddress("www.v2ray.com"), net.Port(80)), // 设置目标地址和端口
        SessionID:     1, // 设置会话ID
        SessionStatus: mux.SessionStatusNew, // 设置会话状态为新建
    }
    writer := buf.New() // 创建一个新的缓冲区
    defer writer.Release() // 在函数返回前释放缓冲区资源

    for i := 0; i < b.N; i++ { // 循环执行 b.N 次
        common.Must(frame.WriteTo(writer)) // 将帧元数据写入缓冲区
        writer.Clear() // 清空缓冲区
    }
}
```