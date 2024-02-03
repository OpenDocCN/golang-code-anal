# `kubo\repo\fsrepo\migrations\retryfetcher.go`

```go
package migrations

import (
    "context"  // 导入上下文包
    "fmt"      // 导入格式化包
)

type RetryFetcher struct {
    Fetcher   // 重试获取器包含一个获取器
    MaxTries int  // 最大重试次数
}

var _ Fetcher = (*RetryFetcher)(nil)  // 确保 RetryFetcher 实现了 Fetcher 接口

func (r *RetryFetcher) Fetch(ctx context.Context, filePath string) ([]byte, error) {
    var lastErr error  // 定义最后一个错误
    for i := 0; i < r.MaxTries; i++ {  // 循环最大重试次数
        out, err := r.Fetcher.Fetch(ctx, filePath)  // 获取数据
        if err == nil {  // 如果没有错误
            return out, nil  // 返回数据和空错误
        }

        if ctx.Err() != nil {  // 如果上下文出错
            return nil, ctx.Err()  // 返回空和上下文错误
        }
        lastErr = err  // 记录错误
    }
    return nil, fmt.Errorf("exceeded number of retries. last error was %w", lastErr)  // 返回错误信息
}

func (r *RetryFetcher) Close() error {
    return r.Fetcher.Close()  // 关闭获取器
}
```