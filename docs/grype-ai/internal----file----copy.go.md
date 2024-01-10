# `grype\internal\file\copy.go`

```
package file

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"   // 导入 io 包，用于输入输出操作
    "os"   // 导入 os 包，用于操作系统功能
    "path" // 导入 path 包，用于处理文件路径

    "github.com/spf13/afero" // 导入第三方库 github.com/spf13/afero，用于文件系统操作
)

func CopyDir(fs afero.Fs, src string, dst string) error {
    var err error
    var fds []os.DirEntry  // 声明变量 fds 为 os.DirEntry 类型的切片
    var srcinfo os.FileInfo  // 声明变量 srcinfo 为 os.FileInfo 类型

    if srcinfo, err = fs.Stat(src); err != nil {  // 获取源文件信息
        return err
    }

    if err = fs.MkdirAll(dst, srcinfo.Mode()); err != nil {  // 创建目标目录
        return err
    }

    if fds, err = os.ReadDir(src); err != nil {  // 读取源目录下的文件和子目录
        return err
    }
    for _, fd := range fds {  // 遍历源目录下的文件和子目录
        srcPath := path.Join(src, fd.Name())  // 拼接源文件路径
        dstPath := path.Join(dst, fd.Name())  // 拼接目标文件路径

        if fd.IsDir() {  // 判断是否为子目录
            if err = CopyDir(fs, srcPath, dstPath); err != nil {  // 递归拷贝子目录
                return fmt.Errorf("could not copy dir (%s -> %s): %w", srcPath, dstPath, err)
            }
        } else {
            if err = CopyFile(fs, srcPath, dstPath); err != nil {  // 拷贝文件
                return fmt.Errorf("could not copy file (%s -> %s): %w", srcPath, dstPath, err)
            }
        }
    }
    return nil
}

func CopyFile(fs afero.Fs, src, dst string) error {
    var err error
    var srcFd afero.File  // 声明变量 srcFd 为 afero.File 类型
    var dstFd afero.File  // 声明变量 dstFd 为 afero.File 类型
    var srcinfo os.FileInfo  // 声明变量 srcinfo 为 os.FileInfo 类型

    if srcFd, err = fs.Open(src); err != nil {  // 打开源文件
        return err
    }
    defer srcFd.Close()  // 延迟关闭源文件

    if dstFd, err = fs.Create(dst); err != nil {  // 创建目标文件
        return err
    }
    defer dstFd.Close()  // 延迟关闭目标文件

    if _, err = io.Copy(dstFd, srcFd); err != nil {  // 拷贝文件内容
        return err
    }
    if srcinfo, err = fs.Stat(src); err != nil {  // 获取源文件信息
        return err
    }
    return fs.Chmod(dst, srcinfo.Mode())  // 设置目标文件权限
}
```