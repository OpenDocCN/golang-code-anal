# `grype\grype\pkg\context.go`

```
# 导入 pkg 包
package pkg

# 导入所需的模块
import (
    "github.com/anchore/syft/syft/linux"  # 导入 linux 模块
    "github.com/anchore/syft/syft/source"  # 导入 source 模块
)

# 定义 Context 结构体
type Context struct {
    Source *source.Description  # Context 结构体包含一个指向 source.Description 类型的指针
    Distro *linux.Release  # Context 结构体包含一个指向 linux.Release 类型的指针
}
```