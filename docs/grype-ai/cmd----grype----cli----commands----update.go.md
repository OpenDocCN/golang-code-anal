# `grype\cmd\grype\cli\commands\update.go`

```
package commands

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"   // 导入 io 包，用于实现 I/O 操作
    "net/http"  // 导入 net/http 包，用于发送 HTTP 请求
    "strings"   // 导入 strings 包，用于处理字符串

    hashiVersion "github.com/anchore/go-version"  // 导入第三方包，用于处理版本号
    "github.com/anchore/grype/cmd/grype/internal"  // 导入内部包
)

var latestAppVersionURL = struct {
    host string
    path string
}{
    host: "https://toolbox-data.anchore.io",  // 定义最新应用版本的 URL 主机地址
    path: "/grype/releases/latest/VERSION",   // 定义最新应用版本的 URL 路径
}

func isProductionBuild(version string) bool {
    if strings.Contains(version, "SNAPSHOT") || strings.Contains(version, internal.NotProvided) {
        return false  // 如果版本号中包含 "SNAPSHOT" 或者是未提供的版本，则不是生产版本
    }
    return true  // 否则是生产版本
}

func isUpdateAvailable(version string) (bool, string, error) {
    if !isProductionBuild(version) {
        // 不允许非生产版本检查更新
        return false, "", nil
    }
    currentVersion, err := hashiVersion.NewVersion(version)  // 解析当前应用版本号
    if err != nil {
        return false, "", fmt.Errorf("failed to parse current application version: %w", err)  // 解析失败，返回错误
    }

    latestVersion, err := fetchLatestApplicationVersion()  // 获取最新应用版本号
    if err != nil {
        return false, "", err  // 获取失败，返回错误
    }

    if latestVersion.GreaterThan(currentVersion) {  // 如果最新版本号大于当前版本号
        return true, latestVersion.String(), nil  // 返回需要更新，以及最新版本号
    }

    return false, "", nil  // 否则不需要更新
}

func fetchLatestApplicationVersion() (*hashiVersion.Version, error) {
    req, err := http.NewRequest(http.MethodGet, latestAppVersionURL.host+latestAppVersionURL.path, nil)  // 创建获取最新版本号的 HTTP 请求
    if err != nil {
        return nil, fmt.Errorf("failed to create request for latest version: %w", err)  // 创建请求失败，返回错误
    }

    client := http.Client{}  // 创建 HTTP 客户端
    resp, err := client.Do(req)  // 发送 HTTP 请求
    if err != nil {
        return nil, fmt.Errorf("failed to fetch latest version: %w", err)  // 获取最新版本号失败，返回错误
    }
    defer resp.Body.Close()  // 延迟关闭响应体

    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf("HTTP %d on fetching latest version: %s", resp.StatusCode, resp.Status)  // 获取最新版本号的 HTTP 状态码不是 200，返回错误
    }

    versionBytes, err := io.ReadAll(resp.Body)  // 读取响应体中的最新版本号数据
    if err != nil {
        return nil, fmt.Errorf("failed to read latest version: %w", err)  // 读取最新版本号失败，返回错误
    }
}
    # 使用 TrimSuffix 函数去除 versionBytes 末尾的换行符，并转换为字符串
    versionStr := strings.TrimSuffix(string(versionBytes), "\n")
    # 如果 versionStr 的长度超过50，则返回错误信息
    if len(versionStr) > 50 {
        return nil, fmt.Errorf("version too long: %q", versionStr[:50])
    }
    # 使用 NewVersion 函数创建一个 hashiVersion 版本对象，并返回
    return hashiVersion.NewVersion(versionStr)
# 闭合前面的函数定义
```