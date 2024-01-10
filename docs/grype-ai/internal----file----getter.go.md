# `grype\internal\file\getter.go`

```
package file

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"  // 导入 io 包，用于实现 I/O 操作
    "net/http"  // 导入 net/http 包，用于发送 HTTP 请求

    "github.com/hashicorp/go-getter"  // 导入 go-getter 包，用于获取远程资源
    "github.com/hashicorp/go-getter/helper/url"  // 导入 url 包，用于处理 URL
    "github.com/wagoodman/go-progress"  // 导入 go-progress 包，用于显示下载进度条

    "github.com/anchore/grype/internal/stringutil"  // 导入 stringutil 包，用于处理字符串
)

var (
    archiveExtensions   = getterDecompressorNames()  // 定义变量 archiveExtensions，并初始化为 getterDecompressorNames() 的返回值
    ErrNonArchiveSource = fmt.Errorf("non-archive sources are not supported for directory destinations")  // 定义变量 ErrNonArchiveSource，并初始化为指定的错误信息
)

type Getter interface {
    // GetFile downloads the give URL into the given path. The URL must reference a single file.
    GetFile(dst, src string, monitor ...*progress.Manual) error  // 定义接口 Getter，包含 GetFile 方法，用于下载单个文件

    // GetToDir downloads the resource found at the `src` URL into the given `dst` directory.
    // The directory must already exist, and the remote resource MUST BE AN ARCHIVE (e.g. `.tar.gz`).
    GetToDir(dst, src string, monitor ...*progress.Manual) error  // 定义接口 Getter，包含 GetToDir 方法，用于下载资源到指定目录
}

type HashiGoGetter struct {
    httpGetter getter.HttpGetter  // 定义结构体 HashiGoGetter，包含 httpGetter 字段
}

// NewGetter creates and returns a new Getter. Providing an http.Client is optional. If one is provided,
// it will be used for all HTTP(S) getting; otherwise, go-getter's default getters will be used.
func NewGetter(httpClient *http.Client) *HashiGoGetter {
    return &HashiGoGetter{  // 创建并返回新的 HashiGoGetter 实例
        httpGetter: getter.HttpGetter{  // 初始化 httpGetter 字段
            Client: httpClient,  // 使用提供的 http.Client，或者使用 go-getter 的默认获取器
        },
    }
}

func (g HashiGoGetter) GetFile(dst, src string, monitors ...*progress.Manual) error {
    if len(monitors) > 1 {  // 如果提供了多个监视器，则返回错误
        return fmt.Errorf("multiple monitors provided, which is not allowed")
    }

    return getterClient(dst, src, false, g.httpGetter, monitors).Get()  // 调用 getterClient 方法下载文件
}

func (g HashiGoGetter) GetToDir(dst, src string, monitors ...*progress.Manual) error {
    // though there are multiple getters, only the http/https getter requires extra validation
    if err := validateHTTPSource(src); err != nil {  // 验证 HTTP 源的有效性
        return err  // 如果验证失败，则返回错误
    }
    if len(monitors) > 1 {  // 如果提供了多个监视器，则返回错误
        return fmt.Errorf("multiple monitors provided, which is not allowed")
    }
}
    # 调用getterClient函数，传入目标地址(dst)、源地址(src)、是否使用HTTPS(true)、HTTP获取器(g.httpGetter)和监视器列表(monitors)，并获取数据
    return getterClient(dst, src, true, g.httpGetter, monitors).Get()
}

// 验证 HTTP 源是否有效，如果不是以 http:// 或 https:// 开头，则忽略
func validateHTTPSource(src string) error {
    if !stringutil.HasAnyOfPrefixes(src, "http://", "https://") {
        return nil
    }

    // 解析 URL，检查是否为有效的 URL
    u, err := url.Parse(src)
    if err != nil {
        return fmt.Errorf("bad URL provided %q: %w", src, err)
    }
    // 只允许具有存档扩展名的源
    if !stringutil.HasAnyOfSuffixes(u.Path, archiveExtensions...) {
        return ErrNonArchiveSource
    }
    return nil
}

// 创建并返回一个 getter 客户端
func getterClient(dst, src string, dir bool, httpGetter getter.HttpGetter, monitors []*progress.Manual) *getter.Client {
    client := &getter.Client{
        Src: src,
        Dst: dst,
        Dir: dir,
        Getters: map[string]getter.Getter{
            "http":  &httpGetter,
            "https": &httpGetter,
            // 注意：这些是来自 https://github.com/hashicorp/go-getter/blob/v1.5.9/get.go#L68-L74 的默认 getter
            // 可能需要考虑其他实现需要自定义 httpclient 注入，但目前还没有考虑到
            "file": new(getter.FileGetter),
            "git":  new(getter.GitGetter),
            "gcs":  new(getter.GCSGetter),
            "hg":   new(getter.HgGetter),
            "s3":   new(getter.S3Getter),
        },
        Options: mapToGetterClientOptions(monitors),
    }

    return client
}

// 返回带有进度监控的 getter 客户端
func withProgress(monitor *progress.Manual) func(client *getter.Client) error {
    return getter.WithProgress(
        &progressAdapter{monitor: monitor},
    )
}

// 将进度监控映射为 getter 客户端选项
func mapToGetterClientOptions(monitors []*progress.Manual) []getter.ClientOption {
    // TODO: 一旦有通用的 `map` 方法可用，此函数将不再需要

    var result []getter.ClientOption

    for _, monitor := range monitors {
        result = append(result, withProgress(monitor))
    }

    return result
}

// 定义一个带有进度监控的读取器
type readCloser struct {
    progress.Reader
}
# 实现了readCloser接口的Close方法，返回nil表示关闭成功
func (c *readCloser) Close() error { return nil }

# 定义了progressAdapter结构体，包含一个progress.Manual类型的monitor字段
type progressAdapter struct {
    monitor *progress.Manual
}

# 实现了TrackProgress方法，用于跟踪进度并返回一个io.ReadCloser接口
func (a *progressAdapter) TrackProgress(_ string, currentSize, totalSize int64, stream io.ReadCloser) io.ReadCloser {
    # 设置当前进度
    a.monitor.Set(currentSize)
    # 设置总进度
    a.monitor.SetTotal(totalSize)
    # 返回一个readCloser类型的对象，其中包含了一个经过进度监控的stream
    return &readCloser{
        Reader: *progress.NewProxyReader(stream, a.monitor),
    }
}

# 定义了getterDecompressorNames函数，用于获取getter.Decompressors中的所有解压器名称
func getterDecompressorNames() (names []string) {
    # 遍历getter.Decompressors中的所有解压器名称，并添加到names切片中
    for name := range getter.Decompressors {
        names = append(names, name)
    }
    # 返回解压器名称切片
    return names
}
```