# `grype\internal\file\tar.go`

```
package file

import (
	"archive/tar"  // 导入tar包，用于处理tar文件
	"compress/gzip"  // 导入gzip包，用于处理gzip压缩文件
	"fmt"  // 导入fmt包，用于格式化输出
	"io"  // 导入io包，用于实现输入输出
	"os"  // 导入os包，提供操作系统函数
	"path/filepath"  // 导入filepath包，用于处理文件路径
	"strings"  // 导入strings包，用于处理字符串
)

const (
	_  = iota  // 使用匿名变量占位
	KB = 1 << (10 * iota)  // 定义KB为1左移10位的结果
	MB  // 定义MB为1左移20位的结果
	GB  // 定义GB为1左移30位的结果
	// 限制tar读取器每个文件的大小为5GB，以防止解压缩炸弹攻击。为什么是5GB？这在一定程度上是一个任意的阈值，但是我们需要将其至少设置为2GB以适应可能的grype数据库大小。
	decompressionByteReadLimit = 5 * GB  // 定义解压缩字节读取限制为5GB
// 定义一个自定义错误类型，用于表示检测到 ZIP 文件路径遍历漏洞
type errZipSlipDetected struct {
	Prefix   string      // 存储路径前缀
	JoinArgs []string    // 存储连接参数
}

// 实现错误接口的方法，返回错误信息
func (e *errZipSlipDetected) Error() string {
	return fmt.Sprintf("paths are not allowed to resolve outside of the root prefix (%q). Destination: %q", e.Prefix, e.JoinArgs)
}

// safeJoin 确保任何目的地路径都不会解析到前缀路径之外
func safeJoin(prefix string, dest ...string) (string, error) {
	// 将前缀和目的地连接成一个路径
	joinResult := filepath.Join(append([]string{prefix}, dest...)...)
	// 清理连接结果，去除多余的分隔符和相对路径
	cleanJoinResult := filepath.Clean(joinResult)
	// 检查清理后的连接结果是否以清理后的前缀路径开头，如果不是则表示存在路径遍历漏洞
	if !strings.HasPrefix(cleanJoinResult, filepath.Clean(prefix)) {
		// 返回自定义错误类型，表示检测到路径遍历漏洞
		return "", &errZipSlipDetected{
			Prefix:   prefix,
			JoinArgs: dest,
		}
	}
	// 为什么不返回清晰的路径？调用者可能没有预期这个结果，因为应该只是一个连接操作。
	return joinResult, nil
}

func UnTarGz(dst string, r io.Reader) error {
	// 从输入流创建一个新的 gzip 读取器
	gzr, err := gzip.NewReader(r)
	if err != nil {
		return err
	}
	// 延迟关闭 gzip 读取器
	defer gzr.Close()

	// 创建一个新的 tar 读取器
	tr := tar.NewReader(gzr)

	// 循环读取 tar 文件中的每个文件
	for {
		// 读取下一个文件的头信息
		header, err := tr.Next()

		switch {
		// 如果到达文件末尾，返回 nil
		case err == io.EOF:
			return nil
		// 如果发生错误，返回错误信息
		case err != nil:
			return err

		// 如果文件头为空，继续下一个循环
		case header == nil:
			continue
		}

		// 将目标路径和文件名安全地拼接起来
		target, err := safeJoin(dst, header.Name)
		if err != nil {
			return err
		}

		// 根据文件类型标志进行不同的操作
		switch header.Typeflag {
		// 如果是目录类型
		case tar.TypeDir:
			// 检查目标路径是否存在，如果不存在则创建目录
			if _, err := os.Stat(target); err != nil {
				if err := os.MkdirAll(target, 0755); err != nil {
					return fmt.Errorf("failed to mkdir (%s): %w", target, err)
				}
			}
// 如果是普通文件类型
case tar.TypeReg:
    // 打开或创建目标文件，使用指定的权限
    f, err := os.OpenFile(target, os.O_CREATE|os.O_RDWR, os.FileMode(header.Mode))
    if err != nil {
        return fmt.Errorf("failed to open file (%s): %w", target, err)
    }

    // 从 tar 文件中解压并复制数据到目标文件，同时限制读取的字节数
    if err := copyWithLimits(f, tr, decompressionByteReadLimit, target); err != nil {
        return err
    }

    // 关闭目标文件
    if err = f.Close(); err != nil {
        return fmt.Errorf("failed to close file (%s): %w", target, err)
    }
}

// 限制读取的字节数并将数据复制到目标文件
func copyWithLimits(writer io.Writer, reader io.Reader, byteReadLimit int64, pathInArchive string) error {
    // 使用 io.LimitReader 限制从 reader 中读取的字节数，并将数据复制到 writer
    if numBytes, err := io.Copy(writer, io.LimitReader(reader, byteReadLimit)); err != nil {
# 返回一个带有错误信息的错误对象，包括文件路径和错误信息
return fmt.Errorf("failed to copy file (%s): %w", pathInArchive, err)
# 如果读取的字节数大于等于设定的字节限制，则返回带有错误信息的错误对象
} else if numBytes >= byteReadLimit {
    return fmt.Errorf("failed to copy file (%s): read limit (%d bytes) reached ", pathInArchive, byteReadLimit)
# 如果没有发生错误，则返回空的错误对象
}
return nil
```