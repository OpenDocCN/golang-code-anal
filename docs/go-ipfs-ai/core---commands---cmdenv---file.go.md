# `kubo\core\commands\cmdenv\file.go`

```
// 导入 cmdenv 包
package cmdenv

// 导入 fmt 包
import (
    "fmt"

    // 导入 files 包
    "github.com/ipfs/boxo/files"
)

// GetFileArg 从目录中返回下一个文件或错误
func GetFileArg(it files.DirIterator) (files.File, error) {
    // 如果没有下一个文件，则返回错误
    if !it.Next() {
        // 获取迭代器的错误
        err := it.Err()
        // 如果错误为空，则创建一个新的错误
        if err == nil {
            err = fmt.Errorf("expected a file argument")
        }
        // 返回错误
        return nil, err
    }
    // 从迭代器中获取文件
    file := files.FileFromEntry(it)
    // 如果文件为空，则返回错误
    if file == nil {
        return nil, fmt.Errorf("file argument was nil")
    }
    // 返回文件和空错误
    return file, nil
}
```