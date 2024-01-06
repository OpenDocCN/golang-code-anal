# `grype\grype\pkg\context.go`

```
# 导入包
package pkg

# 导入所需的包
import (
	"github.com/anchore/syft/syft/linux"  # 导入 Linux 包
	"github.com/anchore/syft/syft/source"  # 导入 source 包
)

# 定义 Context 结构体
type Context struct {
	Source *source.Description  # Context 结构体包含一个指向 source.Description 结构体的指针
	Distro *linux.Release  # Context 结构体包含一个指向 linux.Release 结构体的指针
}
```