# `trojan-go\component\forward.go`

```
// 根据条件编译标记，选择构建 forward、full 或 mini 三种模式之一
// +build forward full mini

// 包声明，指定当前文件属于 build 包
package build

// 导入 forward 包，但不使用其中的任何标识符
import (
    _ "github.com/p4gefau1t/trojan-go/proxy/forward"
)
```