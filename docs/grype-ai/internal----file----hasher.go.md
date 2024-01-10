# `grype\internal\file\hasher.go`

```
package file

import (
    "crypto/sha256"  // 导入加密算法包
    "encoding/hex"   // 导入十六进制编码包
    "fmt"            // 导入格式化包
    "hash"           // 导入哈希包
    "io"             // 导入输入输出包
    "strings"        // 导入字符串处理包

    "github.com/spf13/afero"  // 导入文件系统操作包
)

func ValidateByHash(fs afero.Fs, path, hashStr string) (bool, string, error) {
    var hasher hash.Hash  // 定义哈希对象
    var hashFn string     // 定义哈希函数名称
    switch {
    case strings.HasPrefix(hashStr, "sha256:"):  // 判断哈希字符串是否以"sha256:"开头
        hashFn = "sha256"  // 设置哈希函数名称为sha256
        hasher = sha256.New()  // 创建sha256哈希对象
    default:
        return false, "", fmt.Errorf("hasher not supported or specified (given: %s)", hashStr)  // 返回错误信息
    }

    hashNoPrefix := strings.Split(hashStr, ":")[1]  // 获取去除前缀后的哈希值

    actualHash, err := HashFile(fs, path, hasher)  // 计算文件的哈希值
    if err != nil {
        return false, "", err  // 返回错误信息
    }

    return actualHash == hashNoPrefix, hashFn + ":" + actualHash, nil  // 返回哈希值比较结果和完整哈希字符串
}

func HashFile(fs afero.Fs, path string, hasher hash.Hash) (string, error) {
    f, err := fs.Open(path)  // 打开文件
    if err != nil {
        return "", fmt.Errorf("failed to open file '%s': %w", path, err)  // 返回错误信息
    }
    defer f.Close()  // 延迟关闭文件

    if _, err := io.Copy(hasher, f); err != nil {  // 将文件内容拷贝到哈希对象中
        return "", fmt.Errorf("failed to hash file '%s': %w", path, err)  // 返回错误信息
    }

    return hex.EncodeToString(hasher.Sum(nil)), nil  // 返回文件的哈希值
}
```