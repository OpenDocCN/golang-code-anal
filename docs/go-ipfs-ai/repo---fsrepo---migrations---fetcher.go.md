# `kubo\repo\fsrepo\migrations\fetcher.go`

```
package migrations

import (
    "context"  // 上下文包，用于处理请求的取消、超时等
    "fmt"  // 格式化包，用于打印输出
    "io"  // 输入输出包，提供了基本的输入输出功能
    "os"  // 操作系统包，提供了对操作系统功能的访问

    "github.com/hashicorp/go-multierror"  // 多错误包，用于处理多个错误的情况
)

const (
    // 用于获取迁移的当前分发
    CurrentIpfsDist = "/ipfs/QmZPedUiZNe6Gq9oDvoizuuCMVoeb7shwq9xKhysq7exMo" // fs-repo-14-to-15 v1.0.1
    // 最新分发路径，默认用于获取器
    LatestIpfsDist = "/ipns/dist.ipfs.tech"

    // 分发环境变量
    envIpfsDistPath = "IPFS_DIST_PATH"
)

type Fetcher interface {
    // Fetch 尝试获取给定 IPFS 路径的文件
    Fetch(ctx context.Context, filePath string) ([]byte, error)
    // Close 在不再需要获取器时执行任何清理工作
    Close() error
}

// MultiFetcher 包含多个获取器，并提供一个 Fetch 方法，依次尝试每个获取器，直到成功为止
type MultiFetcher struct {
    fetchers []Fetcher
}

type limitReadCloser struct {
    io.Reader
    io.Closer
}

// NewMultiFetcher 使用给定的获取器创建一个 MultiFetcher。获取器按顺序尝试，然后传递给此函数
func NewMultiFetcher(f ...Fetcher) *MultiFetcher {
    mf := &MultiFetcher{
        fetchers: make([]Fetcher, len(f)),
    }
    copy(mf.fetchers, f)
    return mf
}

// Fetch 尝试从每个获取器获取文件，直到成功为止
func (f *MultiFetcher) Fetch(ctx context.Context, ipfsPath string) ([]byte, error) {
    var errs error
    for _, fetcher := range f.fetchers {
        out, err := fetcher.Fetch(ctx, ipfsPath)
        if err == nil {
            return out, nil
        }
        fmt.Printf("Error fetching: %s\n", err.Error())
        errs = multierror.Append(errs, err)
    }
    return nil, errs
}

func (f *MultiFetcher) Close() error {
    var errs error
    for _, fetcher := range f.fetchers {
        if err := fetcher.Close(); err != nil {
            errs = multierror.Append(errs, err)
        }
    }
    return errs
}

func (f *MultiFetcher) Len() int {
    return len(f.fetchers)
}
// 返回 MultiFetcher 结构体中的 fetchers 切片
func (f *MultiFetcher) Fetchers() []Fetcher {
    return f.fetchers
}

// NewLimitReadCloser 返回一个新的 io.ReadCloser，其中的读取器被包装在一个 io.LimitedReader 中，限制读取指定的数量
func NewLimitReadCloser(rc io.ReadCloser, limit int64) io.ReadCloser {
    return limitReadCloser{
        Reader: io.LimitReader(rc, limit),
        Closer: rc,
    }
}

// GetDistPathEnv 返回分发站点的 IPFS 路径，使用由 envIpfsDistPath 指定的环境变量的值。如果环境变量未设置，则返回提供的 distPath，如果也未设置，则返回 IPNS 路径。
//
// 要获取最新分发的 IPFS 路径，如果未被环境变量覆盖：GetDistPathEnv(CurrentIpfsDist)。
func GetDistPathEnv(distPath string) string {
    if dist := os.Getenv(envIpfsDistPath); dist != "" {
        return dist
    }
    if distPath == "" {
        return LatestIpfsDist
    }
    return distPath
}
```