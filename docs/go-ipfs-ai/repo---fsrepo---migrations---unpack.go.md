# `kubo\repo\fsrepo\migrations\unpack.go`

```
package migrations

import (
    "archive/tar"  // 导入tar包，用于处理tar文件
    "archive/zip"  // 导入zip包，用于处理zip文件
    "compress/gzip"  // 导入gzip包，用于处理gzip压缩
    "errors"  // 导入errors包，用于处理错误
    "fmt"  // 导入fmt包，用于格式化输入输出
    "io"  // 导入io包，用于进行输入输出操作
    "os"  // 导入os包，提供了对操作系统功能的访问
)

func unpackArchive(arcPath, atype, root, name, out string) error {
    var err error
    switch atype {
    case "tar.gz":
        err = unpackTgz(arcPath, root, name, out)  // 解压tar.gz文件
    case "zip":
        err = unpackZip(arcPath, root, name, out)  // 解压zip文件
    default:
        err = fmt.Errorf("unrecognized archive type: %s", atype)  // 返回错误信息，表示未识别的压缩类型
    }
    if err != nil {
        return err  // 如果有错误，则返回错误
    }
    return nil  // 返回空值
}

func unpackTgz(arcPath, root, name, out string) error {
    fi, err := os.Open(arcPath)  // 打开tar.gz文件
    if err != nil {
        return fmt.Errorf("cannot open archive file: %w", err)  // 返回无法打开文件的错误信息
    }
    defer fi.Close()  // 延迟关闭文件

    gzr, err := gzip.NewReader(fi)  // 创建gzip读取器
    if err != nil {
        return fmt.Errorf("error opening gzip reader: %w", err)  // 返回打开gzip读取器错误信息
    }
    defer gzr.Close()  // 延迟关闭gzip读取器

    var bin io.Reader  // 声明一个io.Reader类型的变量bin
    tarr := tar.NewReader(gzr)  // 创建tar文件读取器

    lookFor := root + "/" + name  // 拼接要查找的文件路径
    for {
        th, err := tarr.Next()  // 获取tar文件的下一个文件头
        if err != nil {
            if err == io.EOF {
                break  // 如果到达文件末尾，则退出循环
            }
            return fmt.Errorf("cannot read archive: %w", err)  // 返回无法读取文件的错误信息
        }

        if th.Name == lookFor {
            bin = tarr  // 如果找到目标文件，则将文件读取器赋值给bin
            break
        }
    }

    if bin == nil {
        return errors.New("no binary found in archive")  // 如果bin为空，则返回未在压缩文件中找到目标文件的错误信息
    }

    return writeToPath(bin, out)  // 调用writeToPath函数，将文件内容写入指定路径
}

func unpackZip(arcPath, root, name, out string) error {
    zipr, err := zip.OpenReader(arcPath)  // 打开zip文件
    if err != nil {
        return fmt.Errorf("error opening zip reader: %w", err)  // 返回打开zip读取器错误信息
    }
    defer zipr.Close()  // 延迟关闭zip读取器

    lookFor := root + "/" + name  // 拼接要查找的文件路径
    var bin io.ReadCloser  // 声明一个io.ReadCloser类型的变量bin
    for _, fis := range zipr.File {
        if fis.Name == lookFor {
            rc, err := fis.Open()  // 打开zip文件中的目标文件
            if err != nil {
                return fmt.Errorf("error extracting binary from archive: %w", err)  // 返回从压缩文件中提取目标文件的错误信息
            }

            bin = rc  // 将打开的文件赋值给bin
            break
        }
    }

    if bin == nil {
        return errors.New("no binary found in archive")  // 如果bin为空，则返回未在压缩文件中找到目标文件的错误信息
    }
}
    # 调用 writeToPath 函数，将 bin 写入到 out 指定的路径
    return writeToPath(bin, out)
# 将读取的数据写入指定路径的文件
func writeToPath(rc io.Reader, out string) error:
    # 创建指定路径的文件
    binfi, err := os.Create(out)
    # 如果创建文件出错，则返回错误信息
    if err != nil:
        return fmt.Errorf("error creating output file '%s': %w", out, err)
    # 延迟关闭文件
    defer binfi.Close()

    # 将读取的数据拷贝到文件中
    _, err = io.Copy(binfi, rc)

    # 返回可能出现的错误
    return err
```