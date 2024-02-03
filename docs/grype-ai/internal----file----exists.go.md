# `grype\internal\file\exists.go`

```go
// 导入 file 包和所需的依赖包
package file

import (
    "os"  // 导入操作系统相关的包
    "github.com/spf13/afero"  // 导入 afero 包
)

// 判断文件系统中是否存在指定路径的文件或目录
func Exists(fs afero.Fs, path string) (bool, error) {
    // 获取指定路径的文件或目录信息
    info, err := fs.Stat(path)
    // 如果路径不存在，则返回 false 和 nil
    if os.IsNotExist(err) {
        return false, nil
    } else if err != nil {  // 如果出现其他错误，则返回 false 和错误信息
        return false, err
    }

    // 如果路径存在且不是目录，则返回 true 和 nil
    return !info.IsDir(), nil
}
```