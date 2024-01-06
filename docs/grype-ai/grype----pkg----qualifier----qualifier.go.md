# `grype\grype\pkg\qualifier\qualifier.go`

```
// 声明一个名为qualifier的包
package qualifier

// 导入所需的包
import (
	"github.com/anchore/grype/grype/distro"  // 导入distro包
	"github.com/anchore/grype/grype/pkg"      // 导入pkg包
)

// 声明一个接口类型Qualifier
type Qualifier interface {
	// 定义接口方法Satisfied，用于判断给定的发行版和软件包是否满足某些条件
	Satisfied(d *distro.Distro, p pkg.Package) (bool, error)
}
```