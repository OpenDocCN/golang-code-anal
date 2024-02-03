# `trojan-go\component\server.go`

```go
// 根据条件编译标签，当满足 server、full 或 mini 任一条件时，进行编译
// 导入 github.com/p4gefau1t/trojan-go/proxy/server 包，但不使用其中的任何内容
package build

import (
    _ "github.com/p4gefau1t/trojan-go/proxy/server"
)
```