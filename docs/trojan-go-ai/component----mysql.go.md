# `trojan-go\component\mysql.go`

```go
// 标记当前文件适用于 mysql、full 或 mini 构建标记
// +build mysql full mini

// 定义包名为 build
package build

// 导入 github.com/p4gefau1t/trojan-go/statistic/mysql 包，但不使用其中的任何变量或函数
import (
    _ "github.com/p4gefau1t/trojan-go/statistic/mysql"
)
```