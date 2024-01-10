# `kubo\repo\fsrepo\migrations\httpfetcher.go`

```
package migrations

import (
    "context" // 导入上下文包
    "fmt" // 导入格式化包
    "io" // 导入输入输出包
    "net/http" // 导入网络请求包
    "path" // 导入路径处理包
    "strings" // 导入字符串处理包
)

const (
    // 默认网关 URL，与一些 ISP 阻止的 ipfs.io 不同
    defaultGatewayURL = "https://dweb.link"
    // 默认最大下载大小
    defaultFetchLimit = 1024 * 1024 * 512
)

// HttpFetcher 通过 HTTP 获取文件
type HttpFetcher struct { //nolint
    distPath  string
    gateway   string
    limit     int64
    userAgent string
}

var _ Fetcher = (*HttpFetcher)(nil)

// NewHttpFetcher 创建一个新的 HttpFetcher
//
// 如果 distPath 为 ""，则设置为默认的 IPNS 路径
// 如果 gateway 为 ""，则设置为默认值
// 如果 fetchLimit 为 0，则设置为默认值，-1 表示无限制
func NewHttpFetcher(distPath, gateway, userAgent string, fetchLimit int64) *HttpFetcher { //nolint
    f := &HttpFetcher{
        distPath: LatestIpfsDist, // 设置默认的 IPFS 分发路径
        gateway:  defaultGatewayURL, // 设置默认网关
        limit:    defaultFetchLimit, // 设置默认下载限制
    }

    if distPath != "" {
        if !strings.HasPrefix(distPath, "/") {
            distPath = "/" + distPath
        }
        f.distPath = distPath
    }

    if gateway != "" {
        f.gateway = strings.TrimRight(gateway, "/")
    }

    if fetchLimit != 0 {
        if fetchLimit < 0 {
            fetchLimit = 0
        }
        f.limit = fetchLimit
    }

    return f
}

// Fetch 尝试从配置为此 HttpFetcher 的分发站点获取给定路径的文件
func (f *HttpFetcher) Fetch(ctx context.Context, filePath string) ([]byte, error) {
    gwURL := f.gateway + path.Join(f.distPath, filePath) // 构建完整的 URL
    fmt.Printf("Fetching with HTTP: %q\n", gwURL) // 打印 HTTP 获取信息

    req, err := http.NewRequestWithContext(ctx, http.MethodGet, gwURL, nil) // 创建 HTTP 请求
    if err != nil {
        return nil, fmt.Errorf("http.NewRequest error: %w", err) // 返回请求错误
    }

    if f.userAgent != "" {
        req.Header.Set("User-Agent", f.userAgent) // 设置用户代理
    }

    resp, err := http.DefaultClient.Do(req) // 发送 HTTP 请求
    # 如果发生错误，则返回空和错误信息
    if err != nil:
        return nil, fmt.Errorf("http.DefaultClient.Do error: %w", err)

    # 如果响应状态码大于等于400，则关闭响应体，读取错误信息并返回错误
    if resp.StatusCode >= 400:
        defer resp.Body.Close()
        mes, err := io.ReadAll(resp.Body)
        if err != nil:
            return nil, fmt.Errorf("error reading error body: %w", err)
        return nil, fmt.Errorf("GET %s error: %s: %s", gwURL, resp.Status, string(mes))

    # 根据限制条件创建读取关闭器
    var rc io.ReadCloser
    if f.limit != 0:
        rc = NewLimitReadCloser(resp.Body, f.limit)
    else:
        rc = resp.Body
    # 延迟关闭读取关闭器
    defer rc.Close()

    # 读取并返回读取关闭器中的所有数据
    return io.ReadAll(rc)
# 关闭 HttpFetcher 对象的方法
func (f *HttpFetcher) Close() error:
    # 返回空值，表示关闭操作成功
    return nil
}
```