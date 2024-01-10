# `v2ray-core\app\stats\counter_test.go`

```
package stats_test

import (
    "context"  // 导入上下文包，用于处理请求的上下文信息
    "testing"  // 导入测试包，用于编写和运行测试函数

    . "v2ray.com/core/app/stats"  // 导入 stats 包，并将其所有公开的符号导入当前包的命名空间
    "v2ray.com/core/common"  // 导入 common 包，用于通用的功能
    "v2ray.com/core/features/stats"  // 导入 stats 功能包，用于统计功能
)

func TestStatsCounter(t *testing.T) {
    // 创建一个新的对象实例，返回原始对象和错误信息
    raw, err := common.CreateObject(context.Background(), &Config{})
    common.Must(err)  // 如果错误不为空，则触发 panic

    // 将原始对象转换为 stats.Manager 接口类型
    m := raw.(stats.Manager)
    // 注册一个名为 "test.counter" 的计数器，返回计数器和错误信息
    c, err := m.RegisterCounter("test.counter")
    common.Must(err)  // 如果错误不为空，则触发 panic

    // 如果 c.Add(1) 的返回值不等于 1，则触发测试失败
    if v := c.Add(1); v != 1 {
        t.Fatal("unpexcted Add(1) return: ", v, ", wanted ", 1)
    }

    // 如果 c.Set(0) 的返回值不等于 1，则触发测试失败
    if v := c.Set(0); v != 1 {
        t.Fatal("unexpected Set(0) return: ", v, ", wanted ", 1)
    }

    // 如果 c.Value() 的返回值不等于 0，则触发测试失败
    if v := c.Value(); v != 0 {
        t.Fatal("unexpected Value() return: ", v, ", wanted ", 0)
    }
}
```