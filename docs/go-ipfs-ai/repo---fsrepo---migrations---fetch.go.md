# `kubo\repo\fsrepo\migrations\fetch.go`

```go
package migrations

import (
    "bufio" // 导入 bufio 包，提供了缓冲 I/O 的功能
    "bytes" // 导入 bytes 包，提供了操作字节切片的函数
    "context" // 导入 context 包，提供了跟踪请求的上下文
    "fmt" // 导入 fmt 包，提供了格式化 I/O 的功能
    "io" // 导入 io 包，提供了基本的 I/O 接口
    "os" // 导入 os 包，提供了操作系统函数
    "os/exec" // 导入 os/exec 包，提供了执行外部命令的函数
    "path/filepath" // 导入 path/filepath 包，提供了操作文件路径的函数
    "runtime" // 导入 runtime 包，提供了与 Go 运行时环境交互的函数
    "strings" // 导入 strings 包，提供了操作字符串的函数
)

// DownloadDirectory 可以设置为 FetchBinary 保存下载的存档文件的位置
// 如果未设置，则 FetchBinary 会将存档保存在一个临时目录中，并在提取存档内容后将其删除
var DownloadDirectory string

// FetchBinary 从分发站点下载存档并解压缩
//
// 存档中的二进制文件的基本名称可能与存档的基本名称不同。如果不同，则需要指定 binName。
// 例如，由于存档 "go-ipfs_v0.7.0_linux-amd64.tar.gz" 包含一个名为 "ipfs" 的二进制文件，因此需要以下操作
//
//    FetchBinary(ctx, fetcher, "go-ipfs", "v0.7.0", "ipfs", tmpDir)
//
// 如果 out 是一个目录，则将二进制文件写入该目录，并使用存档中的相同名称。
// 否则，将二进制文件写入由 out 指定的文件。
func FetchBinary(ctx context.Context, fetcher Fetcher, dist, ver, binName, out string) (string, error) {
    // 存档文件名是 dist 的基本名称。这是为了支持 dist 中可能的子目录，例如："ipfs-repo-migrations/fs-repo-11-to-12"
    arcName := filepath.Base(dist)
    // 如果未指定二进制基本名称，则与存档基本名称相同
    if binName == "" {
        binName = arcName
    }

    // 存档中存在的二进制文件的名称
    binName = ExeName(binName)

    // 如果文件存在或 stat 失败的原因不是不存在，则返回错误。如果 out 是一个目录，则将提取的二进制文件写入该目录
    fi, err := os.Stat(out)
    // 如果文件不存在，则执行以下操作
    if !os.IsNotExist(err) {
        // 如果出现其他错误，则返回错误
        if err != nil {
            return "", err
        }
        // 如果文件存在但不是目录，则返回错误
        if !fi.IsDir() {
            return "", &os.PathError{
                Op:   "FetchBinary",
                Path: out,
                Err:  os.ErrExist,
            }
        }
        // 如果out存在且是目录，则组合最终名称
        out = filepath.Join(out, binName)
        // 检查目录中是否已经存在该二进制文件
        _, err = os.Stat(out)
        // 如果文件存在，则返回错误
        if !os.IsNotExist(err) {
            if err != nil {
                return "", err
            }
            return "", &os.PathError{
                Op:   "FetchBinary",
                Path: out,
                Err:  os.ErrExist,
            }
        }
    }

    // 设置临时目录为下载目录
    tmpDir := DownloadDirectory
    // 如果临时目录不为空，则执行以下操作
    if tmpDir != "" {
        fi, err = os.Stat(tmpDir)
        // 如果出现错误，则返回错误
        if err != nil {
            return "", err
        }
        // 如果临时目录不是目录，则返回错误
        if !fi.IsDir() {
            return "", &os.PathError{
                Op:   "FetchBinary",
                Path: tmpDir,
                Err:  os.ErrExist,
            }
        }
    } else {
        // 创建临时目录以存储下载内容
        tmpDir, err = os.MkdirTemp("", arcName)
        // 如果出现错误，则返回错误
        if err != nil {
            return "", err
        }
        // 延迟删除临时目录
        defer os.RemoveAll(tmpDir)
    }

    // 设置默认压缩类型为tar.gz
    atype := "tar.gz"
    // 如果运行环境为windows，则设置压缩类型为zip
    if runtime.GOOS == "windows" {
        atype = "zip"
    }

    // 创建存储归档数据的文件
    arcDistPath, arcFullName := makeArchivePath(dist, arcName, ver, atype)
    arcPath := filepath.Join(tmpDir, arcFullName)
    arcFile, err := os.Create(arcPath)
    // 如果出现错误，则返回错误
    if err != nil {
        return "", err
    }
    // 延迟关闭文件
    defer arcFile.Close()

    // 打开连接以从ipfs路径下载归档并写入文件
    arcBytes, err := fetcher.Fetch(ctx, arcDistPath)
    // 如果出现错误，则返回错误
    if err != nil {
        return "", err
    }

    // 写入下载数据
    _, err = io.Copy(arcFile, bytes.NewReader(arcBytes))
    // 如果出现错误，则返回错误
    if err != nil {
        return "", err
    }
    // 关闭归档文件
    arcFile.Close()

    // 解压归档并将二进制数据写入输出文件
    err = unpackArchive(arcPath, atype, dist, binName, out)
    if err != nil {
        return "", err
    }

    // 将输出文件的权限设置为可执行
    err = os.Chmod(out, 0o755)
    if err != nil {
        return "", err
    }

    // 返回输出文件路径和空错误
    return out, nil
// osWithVariant返回带有可选变体的操作系统名称。
// 当前返回runtime.GOOS，或者"linux-musl"。
func osWithVariant() (string, error) {
    if runtime.GOOS != "linux" {
        return runtime.GOOS, nil
    }

    // ldd输出系统的libc类型。
    // - 在标准ubuntu上：ldd (Ubuntu GLIBC 2.23-0ubuntu5) 2.23
    // - 在alpine上：musl libc (x86_64)
    //
    // 我们使用合并的stdout+stderr，
    // 因为ldd --version在不同的操作系统上打印不同的内容。
    // - 在标准ubuntu上：stdout
    // - 在alpine上：stderr（它可能不知道--version标志）
    //
    // 我们抑制非零的退出码（参见关于alpine的最后一点）。
    out, err := exec.Command("sh", "-c", "ldd --version || true").CombinedOutput()
    if err != nil {
        return "", err
    }

    // 现在只需查看输出中是否可以找到"musl"
    scan := bufio.NewScanner(bytes.NewBuffer(out))
    for scan.Scan() {
        if strings.Contains(scan.Text(), "musl") {
            return "linux-musl", nil
        }
    }

    return "linux", nil
}

// makeArchivePath从分发站点相对于下载二进制文件的路径。
// 返回的路径不包含分发站点路径，例如"/ipns/dist.ipfs.tech/"，因为这是知道的获取者。
//
// 返回存档路径和基本名称。
//
// ipfs路径格式为：distribution/version/archiveName
//   - distribution是分发的名称，例如"go-ipfs"
//   - version是要获取的版本，例如"v0.8.0-rc2"
//   - archiveName格式为name_version_osv-GOARCH.atype，例如"go-ipfs_v0.8.0-rc2_linux-amd64.tar.gz"
//
// 这将形成路径：
// go-ipfs/v0.8.0/go-ipfs_v0.8.0_linux-amd64.tar.gz
func makeArchivePath(dist, name, ver, atype string) (string, string) {
    arcName := fmt.Sprintf("%s_%s_%s-%s.%s", name, ver, runtime.GOOS, runtime.GOARCH, atype)
    # 使用格式化字符串函数将 dist、ver 和 arcName 拼接成路径，并返回拼接后的路径和 arcName
    return fmt.Sprintf("%s/%s/%s", dist, ver, arcName), arcName
# 闭合前面的函数定义
```