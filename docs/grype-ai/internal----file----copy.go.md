# `grype\internal\file\copy.go`

```
package file

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"io"  // 导入 io 包，用于实现 I/O 操作
	"os"  // 导入 os 包，提供操作系统功能
	"path"  // 导入 path 包，用于处理文件路径

	"github.com/spf13/afero"  // 导入第三方包 afero，用于文件系统操作
)

func CopyDir(fs afero.Fs, src string, dst string) error {
	var err error  // 定义错误变量
	var fds []os.DirEntry  // 定义目录条目切片
	var srcinfo os.FileInfo  // 定义源文件信息变量

	if srcinfo, err = fs.Stat(src); err != nil {  // 获取源文件信息，如果出错则返回错误
		return err
	}

# 如果目标目录不存在，则创建目标目录，保持与源目录相同的权限
if err = fs.MkdirAll(dst, srcinfo.Mode()); err != nil:
    return err

# 读取源目录下的文件和子目录信息
if fds, err = os.ReadDir(src); err != nil:
    return err

# 遍历源目录下的文件和子目录
for _, fd := range fds:
    # 构建源文件/目录的完整路径
    srcPath := path.Join(src, fd.Name())
    # 构建目标文件/目录的完整路径
    dstPath := path.Join(dst, fd.Name())

    # 如果是子目录，则递归调用CopyDir函数复制子目录
    if fd.IsDir():
        if err = CopyDir(fs, srcPath, dstPath); err != nil:
            return fmt.Errorf("could not copy dir (%s -> %s): %w", srcPath, dstPath, err)
    # 如果是文件，则调用CopyFile函数复制文件
    else:
        if err = CopyFile(fs, srcPath, dstPath); err != nil:
            return fmt.Errorf("could not copy file (%s -> %s): %w", srcPath, dstPath, err)
	}
	// 如果没有错误，返回空值
	return nil
}

// 复制文件
func CopyFile(fs afero.Fs, src, dst string) error {
	var err error
	var srcFd afero.File
	var dstFd afero.File
	var srcinfo os.FileInfo

	// 打开源文件
	if srcFd, err = fs.Open(src); err != nil {
		return err
	}
	// 延迟关闭源文件
	defer srcFd.Close()

	// 创建目标文件
	if dstFd, err = fs.Create(dst); err != nil {
		return err
	}
	// 延迟关闭目标文件
	defer dstFd.Close()
# 将源文件的内容拷贝到目标文件中，返回拷贝过程中的错误
if _, err = io.Copy(dstFd, srcFd); err != nil:
    return err
# 获取源文件的信息，如果出现错误则返回错误
if srcinfo, err = fs.Stat(src); err != nil:
    return err
# 修改目标文件的权限为源文件的权限，返回修改过程中的错误
return fs.Chmod(dst, srcinfo.Mode())
```