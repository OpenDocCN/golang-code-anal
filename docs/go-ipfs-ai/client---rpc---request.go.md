# `kubo\client\rpc\request.go`

```
package rpc

import (
    "context"   // 导入上下文包
    "io"        // 导入输入输出包
    "strings"   // 导入字符串处理包
)

type Request struct {
    Ctx     context.Context   // 请求上下文
    ApiBase string           // API 基础地址
    Command string           // 命令
    Args    []string         // 参数列表
    Opts    map[string]string  // 选项
    Body    io.Reader         // 请求体
    Headers map[string]string  // 请求头
}

func NewRequest(ctx context.Context, url, command string, args ...string) *Request {
    if !strings.HasPrefix(url, "http") {  // 如果 URL 不是以 "http" 开头
        url = "http://" + url  // 在 URL 前面添加 "http://"
    }

    opts := map[string]string{  // 创建选项映射
        "encoding":        "json",  // 设置编码方式为 JSON
        "stream-channels": "true",  // 设置流通道为真
    }
    return &Request{  // 返回新的请求对象
        Ctx:     ctx,  // 设置请求上下文
        ApiBase: url + "/api/v0",  // 设置 API 基础地址
        Command: command,  // 设置命令
        Args:    args,  // 设置参数列表
        Opts:    opts,  // 设置选项
        Headers: make(map[string]string),  // 创建空的请求头映射
    }
}
```