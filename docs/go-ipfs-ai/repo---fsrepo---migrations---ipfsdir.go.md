# `kubo\repo\fsrepo\migrations\ipfsdir.go`

```
package migrations

import (
    "errors"  // 导入 errors 包，用于处理错误
    "fmt"  // 导入 fmt 包，用于格式化输出
    "os"  // 导入 os 包，用于操作系统功能
    "path/filepath"  // 导入 filepath 包，用于处理文件路径
    "strconv"  // 导入 strconv 包，用于字符串和基本数据类型之间的转换
    "strings"  // 导入 strings 包，用于处理字符串

    "github.com/mitchellh/go-homedir"  // 导入第三方包 go-homedir
)

const (
    envIpfsPath = "IPFS_PATH"  // 定义环境变量 IPFS_PATH
    defIpfsDir  = ".ipfs"  // 定义默认的 IPFS 目录
    versionFile = "version"  // 定义版本文件名
)

func init() {
    homedir.DisableCache = true  // 初始化时禁用 homedir 缓存
}

// IpfsDir 返回 IPFS 目录的路径。如果指定了 dir，则返回扩展版本的 dir。如果 dir 为空，则返回由 IPFS_PATH 设置的目录，如果 IPFS_PATH 未设置，则返回家目录中的默认位置。
func IpfsDir(dir string) (string, error) {
    var err error
    if dir == "" {
        dir = os.Getenv(envIpfsPath)  // 获取环境变量 IPFS_PATH 的值
    }
    if dir != "" {
        dir, err = homedir.Expand(dir)  // 扩展路径中的波浪号（~）
        if err != nil {
            return "", err
        }
        return dir, nil
    }

    home, err := homedir.Dir()  // 获取家目录
    if err != nil {
        return "", err
    }
    if home == "" {
        return "", errors.New("could not determine IPFS_PATH, home dir not set")  // 如果家目录未设置，则返回错误
    }

    return filepath.Join(home, defIpfsDir), nil  // 返回默认的 IPFS 目录路径
}

// CheckIpfsDir 获取 IPFS 目录并检查该目录是否存在。
func CheckIpfsDir(dir string) (string, error) {
    var err error
    dir, err = IpfsDir(dir)  // 获取 IPFS 目录路径
    if err != nil {
        return "", err
    }

    _, err = os.Stat(dir)  // 检查目录是否存在
    if err != nil {
        return "", err
    }

    return dir, nil
}

// RepoVersion 返回 IPFS 目录中存储库的版本。如果未指定 IPFS 目录，则使用默认位置。
func RepoVersion(ipfsDir string) (int, error) {
    ipfsDir, err := CheckIpfsDir(ipfsDir)  // 获取 IPFS 目录路径并检查是否存在
    if err != nil {
        return 0, err
    }
    return repoVersion(ipfsDir)  // 返回存储库版本
}

// WriteRepoVersion 将指定的存储库版本写入位于 ipfsDir 中的存储库。如果未指定 ipfsDir，则使用默认位置。
func WriteRepoVersion(ipfsDir string, version int) error {
    ipfsDir, err := IpfsDir(ipfsDir)  // 获取 IPFS 目录路径
    if err != nil {
        return err
    }
    # 使用 filepath.Join 将 ipfsDir 和 versionFile 连接成完整的文件路径
    vFilePath := filepath.Join(ipfsDir, versionFile)
    # 将版本号格式化为字符串，转换为字节数组，写入文件，设置文件权限为 644
    return os.WriteFile(vFilePath, []byte(fmt.Sprintf("%d\n", version)), 0o644)
# 定义一个名为 repoVersion 的函数，接受一个类型为 string 的参数 ipfsDir，返回一个整数和一个错误
func repoVersion(ipfsDir string) (int, error) {
    # 读取指定路径下的文件内容，返回读取的字节切片和可能出现的错误
    c, err := os.ReadFile(filepath.Join(ipfsDir, versionFile))
    # 如果出现错误，返回整数 0 和该错误
    if err != nil {
        return 0, err
    }

    # 将读取的字节切片转换为字符串并去除两端的空白字符，再将其转换为整数
    ver, err := strconv.Atoi(strings.TrimSpace(string(c)))
    # 如果出现错误，返回整数 0 和一个新的错误，说明仓库版本文件中包含无效数据
    if err != nil {
        return 0, errors.New("invalid data in repo version file")
    }
    # 返回转换后的整数和空错误
    return ver, nil
}
```