# `kubo\core\corehttp\logs.go`

```go
package corehttp

import (
    "io"
    "net"
    "net/http"

    lwriter "github.com/ipfs/go-log/writer"
    core "github.com/ipfs/kubo/core"
)

type writeErrNotifier struct {
    w    io.Writer
    errs chan error
}

func newWriteErrNotifier(w io.Writer) (io.WriteCloser, <-chan error) {
    ch := make(chan error, 1)
    return &writeErrNotifier{
        w:    w,
        errs: ch,
    }, ch
}

func (w *writeErrNotifier) Write(b []byte) (int, error) {
    n, err := w.w.Write(b)  // 将数据写入到指定的 io.Writer 中
    if err != nil {  // 如果写入过程中出现错误
        select {
        case w.errs <- err:  // 将错误发送到 errs 通道中
        default:
        }
    }
    if f, ok := w.w.(http.Flusher); ok {  // 如果 w 是 http.Flusher 类型
        f.Flush()  // 刷新 HTTP 响应流
    }
    return n, err  // 返回写入的字节数和可能出现的错误
}

func (w *writeErrNotifier) Close() error {
    select {
    case w.errs <- io.EOF:  // 将 io.EOF 错误发送到 errs 通道中
    default:
    }
    return nil  // 返回空错误
}

func LogOption() ServeOption {
    return func(n *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
        mux.HandleFunc("/logs", func(w http.ResponseWriter, r *http.Request) {
            w.WriteHeader(200)  // 设置 HTTP 响应状态码为 200
            wnf, errs := newWriteErrNotifier(w)  // 创建一个新的 writeErrNotifier 实例
            lwriter.WriterGroup.AddWriter(wnf)  // 将 wnf 添加到 WriterGroup 中
            log.Event(n.Context(), "log API client connected") //nolint deprecated  // 记录日志事件
            <-errs  // 从 errs 通道中接收错误
        })
        return mux, nil  // 返回处理后的 ServeMux 和空错误
    }
}
```