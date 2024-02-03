# `kubo\repo\fsrepo\migrations\unpack_test.go`

```go
package migrations

import (
    "archive/tar"  // 导入tar包，用于处理tar文件
    "archive/zip"  // 导入zip包，用于处理zip文件
    "bufio"  // 导入bufio包，用于缓冲IO
    "compress/gzip"  // 导入gzip包，用于处理gzip文件
    "io"  // 导入io包，提供IO原语
    "os"  // 导入os包，提供操作系统函数
    "path"  // 导入path包，提供对斜杠分隔的路径的操作
    "path/filepath"  // 导入filepath包，提供操作文件路径的函数
    "strings"  // 导入strings包，提供字符串操作函数
    "testing"  // 导入testing包，提供单元测试功能
)

func TestUnpackArchive(t *testing.T) {
    // 检查未识别的存档类型
    err := unpackArchive("", "no-arch-type", "", "", "")
    if err == nil || err.Error() != "unrecognized archive type: no-arch-type" {
        t.Fatal("expected 'unrecognized archive type' error")
    }

    // 测试无法打开的错误
    err = unpackArchive("no-archive", "tar.gz", "", "", "")
    if err == nil || !strings.HasPrefix(err.Error(), "cannot open archive file") {
        t.Fatal("expected 'cannot open' error, got:", err)
    }
    err = unpackArchive("no-archive", "zip", "", "", "")
    if err == nil || !strings.HasPrefix(err.Error(), "error opening zip reader") {
        t.Fatal("expected 'cannot open' error, got:", err)
    }
}

func TestUnpackTgz(t *testing.T) {
    tmpDir := t.TempDir()

    badTarGzip := filepath.Join(tmpDir, "bad.tar.gz")
    err := os.WriteFile(badTarGzip, []byte("bad-data\n"), 0o644)
    if err != nil {
        panic(err)
    }
    err = unpackTgz(badTarGzip, "", "abc", "abc")
    if err == nil || !strings.HasPrefix(err.Error(), "error opening gzip reader") {
        t.Fatal("expected error opening gzip reader, got:", err)
    }

    testTarGzip := filepath.Join(tmpDir, "test.tar.gz")
    testData := "some data"
    err = writeTarGzipFile(testTarGzip, "testroot", "testfile", testData)
    if err != nil {
        panic(err)
    }

    out := filepath.Join(tmpDir, "out.txt")

    // 测试查找不在存档中的文件
    err = unpackTgz(testTarGzip, "testroot", "abc", out)
    if err == nil || err.Error() != "no binary found in archive" {
        t.Fatal("expected 'no binary found in archive' error, got:", err)
    }

    // 测试解压是否正常工作
    err = unpackTgz(testTarGzip, "testroot", "testfile", out)
    if err != nil {
        t.Fatal(err)
    }
}
    // 获取文件信息，包括文件大小等
    fi, err := os.Stat(out)
    // 如果出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }
    // 检查解压后的文件大小是否与预期大小相符
    if fi.Size() != int64(len(testData)) {
        t.Fatal("unpacked file size is", fi.Size(), "expected", len(testData))
    }
}
// 测试解压缩 ZIP 文件
func TestUnpackZip(t *testing.T) {
    // 创建临时目录
    tmpDir := t.TempDir()

    // 创建一个损坏的 ZIP 文件
    badZip := filepath.Join(tmpDir, "bad.zip")
    err := os.WriteFile(badZip, []byte("bad-data\n"), 0o644)
    if err != nil {
        panic(err)
    }
    // 解压缩损坏的 ZIP 文件，预期会出现错误
    err = unpackZip(badZip, "", "abc", "abc")
    if err == nil || !strings.HasPrefix(err.Error(), "error opening zip reader") {
        t.Fatal("expected error opening zip reader, got:", err)
    }

    // 创建一个测试用的 ZIP 文件
    testZip := filepath.Join(tmpDir, "test.zip")
    testData := "some data"
    err = writeZipFile(testZip, "testroot", "testfile", testData)
    if err != nil {
        panic(err)
    }

    out := filepath.Join(tmpDir, "out.txt")

    // 测试查找 ZIP 文件中不存在的文件
    err = unpackZip(testZip, "testroot", "abc", out)
    if err == nil || err.Error() != "no binary found in archive" {
        t.Fatal("expected 'no binary found in archive' error, got:", err)
    }

    // 测试解压缩 ZIP 文件
    err = unpackZip(testZip, "testroot", "testfile", out)
    if err != nil {
        t.Fatal(err)
    }

    // 获取解压后文件的信息
    fi, err := os.Stat(out)
    if err != nil {
        t.Fatal(err)
    }
    // 检查解压后文件的大小是否符合预期
    if fi.Size() != int64(len(testData)) {
        t.Fatal("unpacked file size is", fi.Size(), "expected", len(testData))
    }
}

// 创建一个 tar.gz 文件
func writeTarGzipFile(archName, root, fileName, data string) error {
    // 创建 tar.gz 文件
    archFile, err := os.Create(archName)
    if err != nil {
        return err
    }
    defer archFile.Close()
    w := bufio.NewWriter(archFile)

    // 写入数据到 tar.gz 文件
    err = writeTarGzip(root, fileName, data, w)
    if err != nil {
        return err
    }
    // 刷新缓冲区数据到文件
    if err = w.Flush(); err != nil {
        return err
    }
    // 关闭 tar 文件
    if err = archFile.Close(); err != nil {
        return err
    }
    return nil
}

// 写入数据到 tar 文件并压缩为 gzip 格式
func writeTarGzip(root, fileName, data string, w io.Writer) error {
    // 创建 gzip 写入器
    gzw := gzip.NewWriter(w)
    defer gzw.Close()
    // 创建 tar 写入器
    tw := tar.NewWriter(gzw)
    // 延迟关闭 tar.Writer，确保在函数返回前关闭
    defer tw.Close()

    var err error
    // 如果文件名不为空
    if fileName != "" {
        // 创建一个 tar 文件头
        hdr := &tar.Header{
            Name: path.Join(root, fileName),  // 设置文件名
            Mode: 0o600,  // 设置文件权限
            Size: int64(len(data)),  // 设置文件大小
        }
        // 写入文件头
        if err = tw.WriteHeader(hdr); err != nil {
            return err
        }
        // 写入文件内容
        if _, err := tw.Write([]byte(data)); err != nil {
            return err
        }
    }

    // 关闭 tar.Writer
    if err = tw.Close(); err != nil {
        return err
    }
    // 关闭 gzip.Writer，完成写入 gzip 数据到缓冲区
    if err = gzw.Close(); err != nil {
        return err
    }
    // 返回空错误，表示操作成功
    return nil
// 写入 ZIP 文件
func writeZipFile(archName, root, fileName, data string) error {
    // 创建 ZIP 文件
    archFile, err := os.Create(archName)
    if err != nil {
        return err
    }
    // 延迟关闭 ZIP 文件
    defer archFile.Close()
    // 创建 ZIP 文件的缓冲写入器
    w := bufio.NewWriter(archFile)

    // 写入 ZIP 文件
    err = writeZip(root, fileName, data, w)
    if err != nil {
        return err
    }
    // 刷新缓冲数据到文件
    if err = w.Flush(); err != nil {
        return err
    }
    // 关闭 ZIP 文件
    if err = archFile.Close(); err != nil {
        return err
    }
    return nil
}

// 写入 ZIP 文件
func writeZip(root, fileName, data string, w io.Writer) error {
    // 创建 ZIP 写入器
    zw := zip.NewWriter(w)
    // 延迟关闭 ZIP 写入器
    defer zw.Close()

    // 写入文件名
    f, err := zw.Create(path.Join(root, fileName))
    if err != nil {
        return err
    }
    // 写入文件数据
    _, err = f.Write([]byte(data))
    if err != nil {
        return err
    }

    // 关闭 ZIP 写入器
    if err = zw.Close(); err != nil {
        return err
    }
    return nil
}
```