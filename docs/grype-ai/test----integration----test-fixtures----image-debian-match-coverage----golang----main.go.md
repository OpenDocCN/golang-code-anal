# `grype\test\integration\test-fixtures\image-debian-match-coverage\golang\main.go`

```
package main
// 声明包名为 main，表示这是一个可独立执行的程序

import (
    "fmt"
    // 导入 fmt 包，用于格式化输入输出

    "github.com/google/uuid"
    // 导入 github.com/google/uuid 包，用于生成 UUID
)

func main() {
    // 打印生成的 UUID
    fmt.Println(uuid.New())
    // 调用 uuid 包中的 New 函数生成 UUID，并打印输出
}
```