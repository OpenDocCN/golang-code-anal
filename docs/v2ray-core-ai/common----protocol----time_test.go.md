# `v2ray-core\common\protocol\time_test.go`

```go
package protocol_test

import (
    "testing"  // 导入测试包
    "time"     // 导入时间包

    . "v2ray.com/core/common/protocol"  // 导入协议包，使用 . 表示在当前文件中可以直接使用其中的函数和变量
)

func TestGenerateRandomInt64InRange(t *testing.T) {
    base := time.Now().Unix()  // 获取当前时间的 Unix 时间戳
    delta := 100  // 设置时间范围
    generator := NewTimestampGenerator(Timestamp(base), delta)  // 使用时间戳和时间范围创建时间戳生成器

    for i := 0; i < 100; i++ {  // 循环 100 次
        val := int64(generator())  // 生成随机的时间戳
        if val > base+int64(delta) || val < base-int64(delta) {  // 判断生成的随机时间戳是否在指定范围内
            t.Error(val, " not between ", base-int64(delta), " and ", base+int64(delta))  // 如果不在范围内，则输出错误信息
        }
    }
}
```