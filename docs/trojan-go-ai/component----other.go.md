# `trojan-go\component\other.go`

```
// 标记当前文件适用于 other 或 full 构建标记
// +build other full

// 定义包名为 build
package build

// 导入 github.com/p4gefau1t/trojan-go/easy 包，但不使用其中的任何变量或函数
import (
    _ "github.com/p4gefau1t/trojan-go/easy"
    // 导入 github.com/p4gefau1t/trojan-go/url 包，但不使用其中的任何变量或函数
    _ "github.com/p4gefau1t/trojan-go/url"
)
```