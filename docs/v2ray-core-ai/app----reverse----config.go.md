# `v2ray-core\app\reverse\config.go`

```go
// +build !confonly
// 定义了一个构建标签，表示该文件不仅仅是用于配置

package reverse
// 声明了一个名为 reverse 的包

import (
    "crypto/rand"
    "io"

    "v2ray.com/core/common/dice"
)
// 导入了 crypto/rand、io 和 v2ray.com/core/common/dice 包

func (c *Control) FillInRandom() {
    // 声明了一个名为 FillInRandom 的方法，接收者是类型为 Control 的指针
    randomLength := dice.Roll(64)
    // 生成一个 64 位的随机数
    c.Random = make([]byte, randomLength)
    // 根据随机数长度创建一个字节数组
    io.ReadFull(rand.Reader, c.Random)
    // 从随机数生成器中读取随机数据填充到字节数组中
}
```