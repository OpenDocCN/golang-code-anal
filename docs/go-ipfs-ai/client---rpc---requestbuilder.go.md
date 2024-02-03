# `kubo\client\rpc\requestbuilder.go`

```go
package rpc

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "context"  // 导入 context 包，用于控制请求的上下文
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"  // 导入 io 包，用于进行输入输出操作
    "strconv"  // 导入 strconv 包，用于字符串和基本数据类型之间的转换
    "strings"  // 导入 strings 包，用于处理字符串

    "github.com/blang/semver/v4"  // 导入第三方包，用于处理语义化版本
    "github.com/ipfs/boxo/files"  // 导入第三方包，用于处理文件
)

// RequestBuilder 接口定义了构建请求的方法
type RequestBuilder interface {
    Arguments(args ...string) RequestBuilder  // 添加参数到请求中
    BodyString(body string) RequestBuilder  // 设置请求体为给定的字符串
    BodyBytes(body []byte) RequestBuilder  // 设置请求体为给定的字节流
    Body(body io.Reader) RequestBuilder  // 设置请求体为给定的读取器
    FileBody(body io.Reader) RequestBuilder  // 设置请求体为给定的读取器，并将其包装成 multipartreader
    Option(key string, value interface{}) RequestBuilder  // 设置请求的选项
    Header(name, value string) RequestBuilder  // 设置请求头
    Send(ctx context.Context) (*Response, error)  // 发送请求并返回响应
    Exec(ctx context.Context, res interface{}) error  // 发送请求并将响应解析到给定的接口
}

// encodedAbsolutePathVersion 是 multipart 请求中绝对路径头的版本，从该版本开始使用 %-encoded。在此版本之前，使用原始路径。
var encodedAbsolutePathVersion = semver.MustParse("0.23.0-dev")

// requestBuilder 是一个 IPFS 命令的请求构建器
type requestBuilder struct {
    command    string  // 命令
    args       []string  // 参数
    opts       map[string]string  // 选项
    headers    map[string]string  // 请求头
    body       io.Reader  // 请求体
    buildError error  // 构建错误

    shell *HttpApi  // HTTP API
}

// Arguments 将参数添加到请求的参数列表中
func (r *requestBuilder) Arguments(args ...string) RequestBuilder {
    r.args = append(r.args, args...)
    return r
}

// BodyString 将请求体设置为给定的字符串
func (r *requestBuilder) BodyString(body string) RequestBuilder {
    return r.Body(strings.NewReader(body))
}

// BodyBytes 将请求体设置为给定的字节流
func (r *requestBuilder) BodyBytes(body []byte) RequestBuilder {
    return r.Body(bytes.NewReader(body))
}

// Body 将请求体设置为给定的读取器
func (r *requestBuilder) Body(body io.Reader) RequestBuilder {
    r.body = body
    return r
}

// FileBody 将请求体设置为给定的读取器，并将其包装成 multipartreader
func (r *requestBuilder) FileBody(body io.Reader) RequestBuilder {
    pr, _ := files.NewReaderPathFile("/dev/stdin", io.NopCloser(body), nil)
    // ...
}
    // 创建一个新的映射目录，包含一个空字符串和 pr 对象
    d := files.NewMapDirectory(map[string]files.Node{"": pr})

    // 加载远程版本信息
    version, err := r.shell.loadRemoteVersion()
    // 如果加载远程版本信息出错
    if err != nil {
        // 保存错误信息
        r.buildError = err
        // 返回 RequestBuilder 对象
        return r
    }

    // 判断是否使用编码后的绝对路径
    useEncodedAbsPaths := version.LT(encodedAbsolutePathVersion)
    // 设置请求体，包含映射目录 d，不压缩，使用编码后的绝对路径
    r.body = files.NewMultiFileReader(d, false, useEncodedAbsPaths)

    // 返回 RequestBuilder 对象
    return r
// Option sets the given option.
// 设置给定选项
func (r *requestBuilder) Option(key string, value interface{}) RequestBuilder {
    var s string
    switch v := value.(type) {
    case bool:
        s = strconv.FormatBool(v)
    case string:
        s = v
    case []byte:
        s = string(v)
    default:
        // slow case.
        // 慢速情况
        s = fmt.Sprint(value)
    }
    if r.opts == nil {
        r.opts = make(map[string]string, 1)
    }
    r.opts[key] = s
    return r
}

// Header sets the given header.
// 设置给定的头部
func (r *requestBuilder) Header(name, value string) RequestBuilder {
    if r.headers == nil {
        r.headers = make(map[string]string, 1)
    }
    r.headers[name] = value
    return r
}

// Send sends the request and return the response.
// 发送请求并返回响应
func (r *requestBuilder) Send(ctx context.Context) (*Response, error) {
    if r.buildError != nil {
        return nil, r.buildError
    }

    r.shell.applyGlobal(r)

    req := NewRequest(ctx, r.shell.url, r.command, r.args...)
    req.Opts = r.opts
    req.Headers = r.headers
    req.Body = r.body
    return req.Send(&r.shell.httpcli)
}

// Exec sends the request a request and decodes the response.
// 发送请求并解码响应
func (r *requestBuilder) Exec(ctx context.Context, res interface{}) error {
    httpRes, err := r.Send(ctx)
    if err != nil {
        return err
    }

    if res == nil {
        lateErr := httpRes.Close()
        if httpRes.Error != nil {
            return httpRes.Error
        }
        return lateErr
    }

    return httpRes.decode(res)
}

// _ RequestBuilder = &requestBuilder{}
```