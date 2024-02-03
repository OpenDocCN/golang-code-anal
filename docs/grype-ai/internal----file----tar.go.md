# `grype\internal\file\tar.go`

```go
    // 导入所需的包
    "archive/tar"
    "compress/gzip"
    "fmt"
    "io"
    "os"
    "path/filepath"
    "strings"
)

// 定义常量
const (
    _  = iota
    KB = 1 << (10 * iota)
    MB
    GB
    // 限制 tar 读取器每个文件的大小为 5GB，以防止解压缩炸弹攻击
    decompressionByteReadLimit = 5 * GB
)

// 定义自定义错误类型
type errZipSlipDetected struct {
    Prefix   string
    JoinArgs []string
}

// 实现自定义错误类型的 Error 方法
func (e *errZipSlipDetected) Error() string {
    return fmt.Sprintf("paths are not allowed to resolve outside of the root prefix (%q). Destination: %q", e.Prefix, e.JoinArgs)
}

// safeJoin 确保目标路径不会解析到前缀路径之外
func safeJoin(prefix string, dest ...string) (string, error) {
    // 将目标路径与前缀路径拼接，并清理路径
    joinResult := filepath.Join(append([]string{prefix}, dest...)...)
    cleanJoinResult := filepath.Clean(joinResult)
    // 如果清理后的路径不以前缀路径开头，则返回错误
    if !strings.HasPrefix(cleanJoinResult, filepath.Clean(prefix)) {
        return "", &errZipSlipDetected{
            Prefix:   prefix,
            JoinArgs: dest,
        }
    }
    // 返回拼接后的路径和 nil 错误
    return joinResult, nil
}

// UnTarGz 解压缩 gzip 格式的 tar 文件
func UnTarGz(dst string, r io.Reader) error {
    // 创建 gzip 读取器
    gzr, err := gzip.NewReader(r)
    if err != nil {
        return err
    }
    // 延迟关闭 gzip 读取器
    defer gzr.Close()
    // 创建 tar 读取器
    tr := tar.NewReader(gzr)
    // 无限循环，不断读取 tar 文件的下一个文件头信息
    for {
        // 读取下一个文件头信息
        header, err := tr.Next()

        // 根据不同的错误情况进行处理
        switch {
        // 如果到达文件末尾，返回空值
        case err == io.EOF:
            return nil

        // 如果出现其他错误，直接返回错误
        case err != nil:
            return err

        // 如果文件头信息为空，继续下一次循环
        case header == nil:
            continue
        }

        // 安全地拼接目标路径和文件名
        target, err := safeJoin(dst, header.Name)
        if err != nil {
            return err
        }

        // 根据文件类型进行不同的处理
        switch header.Typeflag {
        // 如果是目录类型，检查目标路径是否存在，不存在则创建
        case tar.TypeDir:
            if _, err := os.Stat(target); err != nil {
                if err := os.MkdirAll(target, 0755); err != nil {
                    return fmt.Errorf("failed to mkdir (%s): %w", target, err)
                }
            }

        // 如果是普通文件类型，打开文件并进行读写操作
        case tar.TypeReg:
            f, err := os.OpenFile(target, os.O_CREATE|os.O_RDWR, os.FileMode(header.Mode))
            if err != nil {
                return fmt.Errorf("failed to open file (%s): %w", target, err)
            }

            // 限制文件读取的字节数，并进行复制操作
            if err := copyWithLimits(f, tr, decompressionByteReadLimit, target); err != nil {
                return err
            }

            // 关闭文件
            if err = f.Close(); err != nil {
                return fmt.Errorf("failed to close file (%s): %w", target, err)
            }
        }
    }
# 复制文件内容到指定的写入器中，限制读取的字节数，并指定路径
func copyWithLimits(writer io.Writer, reader io.Reader, byteReadLimit int64, pathInArchive string) error:
    # 使用io.LimitReader限制读取的字节数，并将内容复制到写入器中
    if numBytes, err := io.Copy(writer, io.LimitReader(reader, byteReadLimit)); err != nil:
        # 如果复制过程中出现错误，则返回带有路径信息的错误信息
        return fmt.Errorf("failed to copy file (%s): %w", pathInArchive, err)
    # 如果复制的字节数达到或超过限制的字节数，则返回带有路径信息的错误信息
    else if numBytes >= byteReadLimit:
        return fmt.Errorf("failed to copy file (%s): read limit (%d bytes) reached ", pathInArchive, byteReadLimit)
    # 如果复制成功，则返回nil
    return nil
```