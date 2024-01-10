# `kubo\client\rpc\response.go`

```
package rpc

import (
    "encoding/json"  // 导入 JSON 编解码包
    "errors"  // 导入错误处理包
    "fmt"  // 导入格式化包
    "io"  // 导入输入输出包
    "mime"  // 导入 MIME 类型处理包
    "net/http"  // 导入 HTTP 包
    "net/url"  // 导入 URL 处理包
    "os"  // 导入操作系统功能包

    "github.com/ipfs/boxo/files"  // 导入文件处理包
    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入 IPFS 命令包
    cmdhttp "github.com/ipfs/go-ipfs-cmds/http"  // 导入 IPFS HTTP 包
)

type Error = cmds.Error  // 定义 Error 类型为 cmds.Error

type trailerReader struct {  // 定义 trailerReader 结构体
    resp *http.Response  // HTTP 响应对象
}

func (r *trailerReader) Read(b []byte) (int, error) {  // trailerReader 结构体的 Read 方法
    n, err := r.resp.Body.Read(b)  // 读取响应体内容
    if err != nil {  // 如果有错误
        if e := r.resp.Trailer.Get(cmdhttp.StreamErrHeader); e != "" {  // 获取 Trailer 中的错误信息
            err = errors.New(e)  // 将 Trailer 中的错误信息转换为错误对象
        }
    }
    return n, err  // 返回读取的字节数和错误
}

func (r *trailerReader) Close() error {  // trailerReader 结构体的 Close 方法
    return r.resp.Body.Close()  // 关闭响应体
}

type Response struct {  // 定义 Response 结构体
    Output io.ReadCloser  // 输出为可读关闭的流
    Error  *Error  // 错误对象
}

func (r *Response) Close() error {  // Response 结构体的 Close 方法
    if r.Output != nil {  // 如果输出不为空
        // drain output (response body)
        _, err1 := io.Copy(io.Discard, r.Output)  // 读取并丢弃输出内容
        err2 := r.Output.Close()  // 关闭输出流
        if err1 != nil {  // 如果读取出错
            return err1  // 返回读取错误
        }
        return err2  // 返回关闭错误
    }
    return nil  // 返回空
}

// Cancel aborts running request (without draining request body).
func (r *Response) Cancel() error {  // Response 结构体的 Cancel 方法
    if r.Output != nil {  // 如果输出不为空
        return r.Output.Close()  // 关闭输出流
    }

    return nil  // 返回空
}

// Decode reads request body and decodes it as json.
func (r *Response) decode(dec interface{}) error {  // Response 结构体的 decode 方法
    if r.Error != nil {  // 如果有错误
        return r.Error  // 返回错误
    }

    err := json.NewDecoder(r.Output).Decode(dec)  // 读取并解码输出内容为 JSON
    err2 := r.Close()  // 关闭输出流
    if err != nil {  // 如果解码出错
        return err  // 返回解码错误
    }

    return err2  // 返回关闭错误
}

func (r *Request) Send(c *http.Client) (*Response, error) {  // Request 结构体的 Send 方法
    url := r.getURL()  // 获取请求的 URL
    req, err := http.NewRequest("POST", url, r.Body)  // 创建 POST 请求
    if err != nil {  // 如果有错误
        return nil, err  // 返回空和错误
    }

    req = req.WithContext(r.Ctx)  // 设置请求上下文

    // Add any headers that were supplied via the requestBuilder.
    for k, v := range r.Headers {  // 遍历请求头
        req.Header.Add(k, v)  // 添加请求头
    }
    # 检查请求体是否为多文件读取器，如果是则设置请求头的内容类型和内容分隔符
    if fr, ok := r.Body.(*files.MultiFileReader); ok:
        req.Header.Set("Content-Type", "multipart/form-data; boundary="+fr.Boundary())
        req.Header.Set("Content-Disposition", "form-data; name=\"files\"")
    
    # 发送 HTTP 请求并获取响应
    resp, err := c.Do(req)
    if err != nil:
        return nil, err
    
    # 解析响应头中的内容类型
    contentType, _, err := mime.ParseMediaType(resp.Header.Get("Content-Type"))
    if err != nil:
        return nil, err
    
    # 创建新的响应对象
    nresp := new(Response)
    
    # 设置新响应对象的输出为 trailerReader 类型的响应
    nresp.Output = &trailerReader{resp}
    // 如果响应状态码大于或等于 400，表示出现错误
    if resp.StatusCode >= http.StatusBadRequest {
        // 创建一个新的 Error 对象
        e := new(Error)
        // 根据不同的状态码设置错误消息
        switch {
        case resp.StatusCode == http.StatusNotFound:
            e.Message = "command not found"
        case contentType == "text/plain":
            // 读取响应体的内容
            out, err := io.ReadAll(resp.Body)
            if err != nil {
                // 输出错误信息到标准错误输出
                fmt.Fprintf(os.Stderr, "ipfs-shell: warning! response (%d) read error: %s\n", resp.StatusCode, err)
            }
            e.Message = string(out)

            // 根据特殊的状态码设置错误代码
            switch resp.StatusCode {
            case http.StatusNotFound, http.StatusBadRequest:
                e.Code = cmds.ErrClient
            case http.StatusTooManyRequests:
                e.Code = cmds.ErrRateLimited
            case http.StatusForbidden:
                e.Code = cmds.ErrForbidden
            }
        case contentType == "application/json":
            // 解析 JSON 格式的响应体内容到 Error 对象
            if err = json.NewDecoder(resp.Body).Decode(e); err != nil {
                // 输出错误信息到标准错误输出
                fmt.Fprintf(os.Stderr, "ipfs-shell: warning! response (%d) unmarshall error: %s\n", resp.StatusCode, err)
            }
        default:
            // 这是一个服务器端的 bug（可能）
            e.Code = cmds.ErrImplementation
            // 输出错误信息到标准错误输出
            fmt.Fprintf(os.Stderr, "ipfs-shell: warning! unhandled response (%d) encoding: %s", resp.StatusCode, contentType)
            // 读取响应体的内容
            out, err := io.ReadAll(resp.Body)
            if err != nil {
                // 输出错误信息到标准错误输出
                fmt.Fprintf(os.Stderr, "ipfs-shell: response (%d) read error: %s\n", resp.StatusCode, err)
            }
            e.Message = fmt.Sprintf("unknown ipfs-shell error encoding: %q - %q", contentType, out)
        }
        // 将 Error 对象设置为新响应的错误
        nresp.Error = e
        // 清空输出内容
        nresp.Output = nil

        // 读取并关闭响应体
        _, _ = io.Copy(io.Discard, resp.Body)
        _ = resp.Body.Close()
    }

    // 返回新的响应对象和空错误
    return nresp, nil
# 定义一个方法，用于获取请求的URL地址
func (r *Request) getURL() string {
    # 创建一个空的URL参数值对象
    values := make(url.Values)
    # 遍历请求的参数列表，将参数添加到URL参数值对象中
    for _, arg := range r.Args {
        values.Add("arg", arg)
    }
    # 遍历请求的选项列表，将选项添加到URL参数值对象中
    for k, v := range r.Opts {
        values.Add(k, v)
    }
    # 格式化URL地址，包括API基础地址、命令和参数值对象的编码结果
    return fmt.Sprintf("%s/%s?%s", r.ApiBase, r.Command, values.Encode())
}
```