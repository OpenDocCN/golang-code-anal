# `v2ray-core\common\protocol\time.go`

```
# 定义 protocol 包
package protocol

# 导入 time 包
import (
    "time"

    # 导入 v2ray.com/core/common/dice 包
    "v2ray.com/core/common/dice"
)

# 定义 Timestamp 类型为 int64
type Timestamp int64

# 定义 TimestampGenerator 函数类型
type TimestampGenerator func() Timestamp

# 返回当前时间的时间戳
func NowTime() Timestamp {
    return Timestamp(time.Now().Unix())
}

# 创建一个新的时间戳生成器
func NewTimestampGenerator(base Timestamp, delta int) TimestampGenerator {
    # 返回一个匿名函数，生成一个在指定范围内的时间戳
    return func() Timestamp {
        # 生成一个在 -delta 到 delta 范围内的随机数
        rangeInDelta := dice.Roll(delta*2) - delta
        # 返回基础时间加上随机数得到的时间戳
        return base + Timestamp(rangeInDelta)
    }
}
```