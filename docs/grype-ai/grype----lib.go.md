# `grype\grype\lib.go`

```
# 导入 grype 包
package grype

# 导入 go-partybus 包
import (
    "github.com/wagoodman/go-partybus"

    # 导入 go-logger 包
    "github.com/anchore/go-logger"
    # 导入 grype 内部的 bus 模块
    "github.com/anchore/grype/internal/bus"
    # 导入 grype 内部的 log 模块
    "github.com/anchore/grype/internal/log"
)

# 设置日志记录器
func SetLogger(l logger.Logger) {
    # 调用 log 模块的 Set 方法，设置日志记录器
    log.Set(l)
}

# 设置消息总线
func SetBus(b *partybus.Bus) {
    # 调用 bus 模块的 Set 方法，设置消息总线
    bus.Set(b)
}
```