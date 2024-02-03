# `trojan-go\component\client.go`

```go
// 标记该文件适用于 client、full 或 mini 构建标记
// +build client full mini

// 定义包名为 build
package build

// 导入 github.com/p4gefau1t/trojan-go/proxy/client 包，但不使用其中的任何变量或函数
import (
    _ "github.com/p4gefau1t/trojan-go/proxy/client"
)
```