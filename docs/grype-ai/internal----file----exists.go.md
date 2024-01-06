# `grype\internal\file\exists.go`

```
// 导入文件操作相关的包
package file

// 导入所需的包
import (
	"os" // 导入操作系统相关的包
	"github.com/spf13/afero" // 导入文件系统操作相关的包
)

// 判断文件或目录是否存在
func Exists(fs afero.Fs, path string) (bool, error) {
	// 获取文件或目录的信息
	info, err := fs.Stat(path)
	// 如果文件或目录不存在，则返回 false 和 nil
	if os.IsNotExist(err) {
		return false, nil
	} else if err != nil { // 如果出现其他错误，则返回 false 和错误信息
		return false, err
	}

	// 如果是文件，则返回 true 和 nil；如果是目录，则返回 false 和 nil
	return !info.IsDir(), nil
}
```