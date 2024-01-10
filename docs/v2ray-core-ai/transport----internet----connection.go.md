# `v2ray-core\transport\internet\connection.go`

```
package internet

import (
    "net"  // 导入 net 包，提供了基本的网络 I/O 接口
    "v2ray.com/core/features/stats"  // 导入 stats 包，提供了统计功能
)

type Connection interface {
    net.Conn  // 定义 Connection 接口，包含 net.Conn 接口
}

type StatCouterConnection struct {
    Connection  // 定义 StatCouterConnection 结构体，包含 Connection 接口
    ReadCounter  stats.Counter  // 读取数据的统计计数器
    WriteCounter stats.Counter  // 写入数据的统计计数器
}

func (c *StatCouterConnection) Read(b []byte) (int, error) {
    nBytes, err := c.Connection.Read(b)  // 调用 Connection 接口的 Read 方法，读取数据
    if c.ReadCounter != nil {  // 如果读取计数器不为空
        c.ReadCounter.Add(int64(nBytes))  // 将读取的字节数添加到读取计数器中
    }

    return nBytes, err  // 返回读取的字节数和可能的错误
}

func (c *StatCouterConnection) Write(b []byte) (int, error) {
    nBytes, err := c.Connection.Write(b)  // 调用 Connection 接口的 Write 方法，写入数据
    if c.WriteCounter != nil {  // 如果写入计数器不为空
        c.WriteCounter.Add(int64(nBytes))  // 将写入的字节数添加到写入计数器中
    }
    return nBytes, err  // 返回写入的字节数和可能的错误
}
```