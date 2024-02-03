# `v2ray-core\common\dice\dice_test.go`

```go
// 定义 dice_test 包
package dice_test

// 导入 math/rand 和 testing 包
import (
    "math/rand"
    "testing"

    . "v2ray.com/core/common/dice"  // 导入 v2ray.com/core/common/dice 包，并使用 . 符号表示可以直接使用该包里的函数
)

// 定义 BenchmarkRoll1 函数，用于测试 Roll 函数对一个骰子的性能
func BenchmarkRoll1(b *testing.B) {
    // 循环执行 Roll 函数 b.N 次
    for i := 0; i < b.N; i++ {
        Roll(1)  // 调用 Roll 函数，传入参数1
    }
}

// 定义 BenchmarkRoll20 函数，用于测试 Roll 函数对二十个骰子的性能
func BenchmarkRoll20(b *testing.B) {
    // 循环执行 Roll 函数 b.N 次
    for i := 0; i < b.N; i++ {
        Roll(20)  // 调用 Roll 函数，传入参数20
    }
}

// 定义 BenchmarkIntn1 函数，用于测试 rand.Intn 函数生成1以内随机数的性能
func BenchmarkIntn1(b *testing.B) {
    // 循环执行 rand.Intn 函数 b.N 次
    for i := 0; i < b.N; i++ {
        rand.Intn(1)  // 调用 rand.Intn 函数，传入参数1
    }
}

// 定义 BenchmarkIntn20 函数，用于测试 rand.Intn 函数生成20以内随机数的性能
func BenchmarkIntn20(b *testing.B) {
    // 循环执行 rand.Intn 函数 b.N 次
    for i := 0; i < b.N; i++ {
        rand.Intn(20)  // 调用 rand.Intn 函数，传入参数20
    }
}
```