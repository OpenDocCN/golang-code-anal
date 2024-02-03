# `v2ray-core\common\net\port_test.go`

```go
// 声明 net_test 包
package net_test

// 导入 testing 包
import (
    "testing"

    // 导入 v2ray.com/core/common/net 包，并使用 . 符号将其命名为当前包的一部分
    . "v2ray.com/core/common/net"
)

// 定义测试函数 TestPortRangeContains
func TestPortRangeContains(t *testing.T) {
    // 创建一个 PortRange 结构体实例，包含 From 和 To 两个字段
    portRange := &PortRange{
        From: 53,
        To:   53,
    }

    // 如果 portRange 不包含端口 53，则输出错误信息
    if !portRange.Contains(Port(53)) {
        t.Error("expected port range containing 53, but actually not")
    }
}
```