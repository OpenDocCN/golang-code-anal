# `grype\cmd\grype\cli\commands\update.go`

```
// 导入命令包
package commands

// 导入所需的包
import (
	"fmt" // 格式化输出
	"io" // 输入输出
	"net/http" // HTTP 请求
	"strings" // 字符串操作

	hashiVersion "github.com/anchore/go-version" // 导入版本比较包
	"github.com/anchore/grype/cmd/grype/internal" // 导入内部包
)

// 定义最新应用版本的 URL 结构体
var latestAppVersionURL = struct {
	host string // 主机地址
	path string // 路径
}{
	host: "https://toolbox-data.anchore.io", // 主机地址
	path: "/grype/releases/latest/VERSION", // 版本路径
}
# 判断是否为生产版本构建
func isProductionBuild(version string) bool:
    # 如果版本中包含"SNAPSHOT"或者未提供的标记，则不是生产版本构建
    if strings.Contains(version, "SNAPSHOT") || strings.Contains(version, internal.NotProvided):
        return false
    # 否则是生产版本构建
    return true

# 判断是否有可用的更新版本
func isUpdateAvailable(version string) (bool, string, error):
    # 如果不是生产版本构建，则不允许检查更新版本
    if !isProductionBuild(version):
        return false, "", nil
    # 解析当前应用程序版本
    currentVersion, err := hashiVersion.NewVersion(version)
    if err != nil:
        return false, "", fmt.Errorf("failed to parse current application version: %w", err)
    # 获取最新的应用程序版本
    latestVersion, err := fetchLatestApplicationVersion()
    if err != nil:
        return false, "", err
// 如果最新版本大于当前版本，则返回 true 和最新版本号，否则返回 false 和空字符串
if latestVersion.GreaterThan(currentVersion) {
    return true, latestVersion.String(), nil
}

// 创建 HTTP 请求，获取最新应用程序版本
func fetchLatestApplicationVersion() (*hashiVersion.Version, error) {
    // 创建 HTTP 请求
    req, err := http.NewRequest(http.MethodGet, latestAppVersionURL.host+latestAppVersionURL.path, nil)
    if err != nil {
        return nil, fmt.Errorf("failed to create request for latest version: %w", err)
    }

    // 创建 HTTP 客户端并发送请求
    client := http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        return nil, fmt.Errorf("failed to fetch latest version: %w", err)
    }
# 关闭 HTTP 响应体
defer resp.Body.Close()

# 如果 HTTP 响应状态码不是 200，返回错误信息
if resp.StatusCode != http.StatusOK:
    return nil, fmt.Errorf("HTTP %d on fetching latest version: %s", resp.StatusCode, resp.Status)

# 读取 HTTP 响应体的内容
versionBytes, err := io.ReadAll(resp.Body)
if err != nil:
    return nil, fmt.Errorf("failed to read latest version: %w", err)

# 去除内容末尾的换行符，并转换为字符串
versionStr := strings.TrimSuffix(string(versionBytes), "\n")

# 如果版本字符串长度超过 50，返回错误信息
if len(versionStr) > 50:
    return nil, fmt.Errorf("version too long: %q", versionStr[:50])

# 返回版本字符串对应的版本对象
return hashiVersion.NewVersion(versionStr)
```