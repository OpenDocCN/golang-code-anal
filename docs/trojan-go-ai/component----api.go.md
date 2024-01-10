# `trojan-go\component\api.go`

```
// 标记这个文件只在构建标签为 api 或 full 时才会被包含进来
// 构建标签为 api 或 full
// 导入 build 包
package build

// 导入外部包，但不使用它们的成员，只是为了执行它们的 init 函数
import (
    _ "github.com/p4gefau1t/trojan-go/api/control"
    _ "github.com/p4gefau1t/trojan-go/api/service"
)
```