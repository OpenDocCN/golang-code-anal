# `trojan-go\component\base.go`

```go
# 导入 build 包
package build

# 导入日志模块
import (
    _ "github.com/p4gefau1t/trojan-go/log/golog"
    # 导入内存统计模块
    _ "github.com/p4gefau1t/trojan-go/statistic/memory"
    # 导入版本模块
    _ "github.com/p4gefau1t/trojan-go/version"
)
```