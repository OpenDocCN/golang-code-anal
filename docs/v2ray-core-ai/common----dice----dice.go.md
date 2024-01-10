# `v2ray-core\common\dice\dice.go`

```
// Package dice contains common functions to generate random number.
// It also initialize math/rand with the time in seconds at launch time.
package dice // import "v2ray.com/core/common/dice"

import (
    "math/rand"  // 导入 math/rand 包
    "time"       // 导入 time 包
)

// Roll returns a non-negative number between 0 (inclusive) and n (exclusive).
func Roll(n int) int {
    if n == 1 {
        return 0
    }
    return rand.Intn(n)  // 返回一个介于 0（包括）和 n（不包括）之间的非负数
}

// Roll returns a non-negative number between 0 (inclusive) and n (exclusive).
func RollDeterministic(n int, seed int64) int {
    if n == 1 {
        return 0
    }
    return rand.New(rand.NewSource(seed)).Intn(n)  // 使用指定的种子创建一个新的随机数生成器，并返回一个介于 0（包括）和 n（不包括）之间的非负数
}

// RollUint16 returns a random uint16 value.
func RollUint16() uint16 {
    return uint16(rand.Intn(65536))  // 返回一个介于 0（包括）和 65536（不包括）之间的随机 uint16 值
}

func RollUint64() uint64 {
    return rand.Uint64()  // 返回一个随机的 uint64 值
}

func NewDeterministicDice(seed int64) *deterministicDice {
    return &deterministicDice{rand.New(rand.NewSource(seed))}  // 使用指定的种子创建一个新的随机数生成器，并返回一个 deterministicDice 结构体指针
}

type deterministicDice struct {
    *rand.Rand  // deterministicDice 结构体包含一个 rand.Rand 类型的指针
}

func (dd *deterministicDice) Roll(n int) int {
    if n == 1 {
        return 0
    }
    return dd.Intn(n)  // 使用 deterministicDice 结构体中的随机数生成器返回一个介于 0（包括）和 n（不包括）之间的非负数
}

func init() {
    rand.Seed(time.Now().Unix())  // 初始化随机数生成器的种子为当前时间的秒数
}
```