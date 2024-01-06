# `grype\internal\file\getter.go`

```
package file

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"io"  // 导入 io 包，用于实现 I/O 操作
	"net/http"  // 导入 net/http 包，用于处理 HTTP 请求

	"github.com/hashicorp/go-getter"  // 导入 go-getter 包，用于文件下载
	"github.com/hashicorp/go-getter/helper/url"  // 导入 go-getter 的 URL 辅助包
	"github.com/wagoodman/go-progress"  // 导入 go-progress 包，用于显示下载进度条

	"github.com/anchore/grype/internal/stringutil"  // 导入 stringutil 包，用于字符串操作
)

var (
	archiveExtensions   = getterDecompressorNames()  // 定义变量 archiveExtensions，并初始化为 getterDecompressorNames() 的返回值
	ErrNonArchiveSource = fmt.Errorf("non-archive sources are not supported for directory destinations")  // 定义变量 ErrNonArchiveSource，并初始化为一个错误信息
)

type Getter interface {  // 定义接口 Getter
// GetFile函数用于下载指定URL的文件到指定路径。URL必须指向单个文件。
GetFile(dst, src string, monitor ...*progress.Manual) error

// GetToDir函数用于将位于src URL的资源下载到指定的dst目录中。
// 目录必须已经存在，并且远程资源必须是一个存档文件（例如.tar.gz）。
GetToDir(dst, src string, monitor ...*progress.Manual) error
}

type HashiGoGetter struct {
	httpGetter getter.HttpGetter
}

// NewGetter函数创建并返回一个新的Getter。提供一个http.Client是可选的。如果提供了一个，
// 它将用于所有的HTTP(S)获取；否则，go-getter的默认获取器将被使用。
func NewGetter(httpClient *http.Client) *HashiGoGetter {
	return &HashiGoGetter{
		httpGetter: getter.HttpGetter{
			Client: httpClient,
		},
	}
```
// 上面的代码是一个Go语言程序的一部分，包含了一些函数和结构体的定义。每个函数和结构体都有注释来解释其作用。
// GetFile 方法用于从源地址获取文件到目标地址，可以传入监控器参数
func (g HashiGoGetter) GetFile(dst, src string, monitors ...*progress.Manual) error {
	// 如果传入的监控器参数数量大于1，返回错误
	if len(monitors) > 1 {
		return fmt.Errorf("multiple monitors provided, which is not allowed")
	}
	// 调用 getterClient 方法，使用 httpGetter 获取文件
	return getterClient(dst, src, false, g.httpGetter, monitors).Get()
}

// GetToDir 方法用于从源地址获取文件到目标目录，可以传入监控器参数
func (g HashiGoGetter) GetToDir(dst, src string, monitors ...*progress.Manual) error {
	// 验证 HTTP 源地址的有效性
	if err := validateHTTPSource(src); err != nil {
		return err
	}
	// 如果传入的监控器参数数量大于1，返回错误
	if len(monitors) > 1 {
		return fmt.Errorf("multiple monitors provided, which is not allowed")
	}
	// 调用 getterClient 方法，使用 httpGetter 获取文件到目标目录
	return getterClient(dst, src, true, g.httpGetter, monitors).Get()
}
}

// validateHTTPSource 校验 HTTP 源
func validateHTTPSource(src string) error {
	// 忽略不是使用 http getter 对象的源
	if !stringutil.HasAnyOfPrefixes(src, "http://", "https://") {
		return nil
	}

	// 解析 URL
	u, err := url.Parse(src)
	if err != nil {
		return fmt.Errorf("bad URL provided %q: %w", src, err)
	}
	// 只允许带有存档扩展名的源
	if !stringutil.HasAnyOfSuffixes(u.Path, archiveExtensions...) {
		return ErrNonArchiveSource
	}
	return nil
}

// getterClient 获取客户端
func getterClient(dst, src string, dir bool, httpGetter getter.HttpGetter, monitors []*progress.Manual) *getter.Client {
// 创建一个getter.Client对象，设置其属性值为src、dst、dir
// 设置其Getters属性为一个map，包含http、https、file、git、gcs、hg、s3等键值对
// 注意：这些是https://github.com/hashicorp/go-getter/blob/v1.5.9/get.go#L68-L74中默认的getter实现
// 可能其他实现需要考虑自定义httpclient注入，但目前还未考虑
// 设置其Options属性为monitors转换为GetterClientOptions的结果
// 返回创建的client对象
}

func withProgress(monitor *progress.Manual) func(client *getter.Client) error {
	return getter.WithProgress(
		&progressAdapter{monitor: monitor},  // 使用进度适配器创建带有进度监视器的客户端
	)
}

func mapToGetterClientOptions(monitors []*progress.Manual) []getter.ClientOption {
	// TODO: This function is no longer needed once a generic `map` method is available.
	// TODO: 一旦有通用的 `map` 方法可用，这个函数就不再需要了。

	var result []getter.ClientOption

	for _, monitor := range monitors {
		result = append(result, withProgress(monitor))  // 将每个进度监视器转换为客户端选项并添加到结果中
	}

	return result  // 返回结果数组
}
# 定义一个结构体 readCloser，包含 progress.Reader 的属性
type readCloser struct {
    progress.Reader
}

# 实现 readCloser 结构体的 Close 方法，返回值为 error 类型
func (c *readCloser) Close() error { return nil }

# 定义一个结构体 progressAdapter，包含 progress.Manual 的属性
type progressAdapter struct {
    monitor *progress.Manual
}

# 实现 progressAdapter 结构体的 TrackProgress 方法，参数为文件名、当前大小、总大小和读取流，返回值为读取流
func (a *progressAdapter) TrackProgress(_ string, currentSize, totalSize int64, stream io.ReadCloser) io.ReadCloser {
    # 设置当前进度
    a.monitor.Set(currentSize)
    # 设置总进度
    a.monitor.SetTotal(totalSize)
    # 返回一个 readCloser 结构体，其中 Reader 属性为经过进度监控的流
    return &readCloser{
        Reader: *progress.NewProxyReader(stream, a.monitor),
    }
}

# 定义一个函数 getterDecompressorNames，返回值为字符串数组
func getterDecompressorNames() (names []string) {
    # 遍历 getter.Decompressors 中的名称
    for name := range getter.Decompressors {
# 将 name 添加到 names 切片中
names = append(names, name)
# 返回 names 切片
}
return names
```