# `kubo\test\cli\harness\http_client.go`

```go
package harness

import (
    "io"
    "net/http"
    "strings"
    "text/template"
    "time"
)

// HTTPClient is an HTTP client with some conveniences for testing.
// URLs are constructed from a base URL.
// The response body is buffered into a string.
// Internal errors cause panics so that tests don't need to check errors.
// The paths are evaluated as Go templates for readable string interpolation.
type HTTPClient struct {
    Client  *http.Client
    BaseURL string

    Timeout      time.Duration
    TemplateData any
}

type HTTPResponse struct {
    Body       string
    StatusCode int
    Headers    http.Header

    // The raw response. The body will be closed on this response.
    Resp *http.Response
}

func (c *HTTPClient) WithHeader(k, v string) func(h *http.Request) {
    return func(h *http.Request) {
        h.Header.Add(k, v)
    }
}

func (c *HTTPClient) DisableRedirects() *HTTPClient {
    c.Client.CheckRedirect = func(req *http.Request, via []*http.Request) error {
        return http.ErrUseLastResponse
    }
    return c
}

// Do executes the request unchanged.
func (c *HTTPClient) Do(req *http.Request) *HTTPResponse {
    log.Debugf("making HTTP req %s to %q with headers %+v", req.Method, req.URL.String(), req.Header)
    resp, err := c.Client.Do(req)
    if resp != nil && resp.Body != nil {
        defer resp.Body.Close()
    }
    if err != nil {
        panic(err)
    }
    bodyStr, err := io.ReadAll(resp.Body)
    if err != nil {
        panic(err)
    }

    return &HTTPResponse{
        Body:       string(bodyStr),
        StatusCode: resp.StatusCode,
        Headers:    resp.Header,
        Resp:       resp,
    }
}

// BuildURL constructs a request URL from the given path by interpolating the string and then appending it to the base URL.
func (c *HTTPClient) BuildURL(urlPath string) string {
    sb := &strings.Builder{}
    // 解析模板并将结果写入字符串构建器
    err := template.Must(template.New("test").Parse(urlPath)).Execute(sb, c.TemplateData)
    # 如果错误不为空，触发 panic
    if err != nil:
        panic(err)
    # 将渲染后的路径转换为字符串
    renderedPath := sb.String()
    # 返回基本 URL 和渲染后的路径的组合
    return c.BaseURL + renderedPath
# 定义 HTTPClient 结构体的 Get 方法，用于发送 HTTP GET 请求
func (c *HTTPClient) Get(urlPath string, opts ...func(*http.Request)) *HTTPResponse {
    # 创建一个 HTTP GET 请求
    req, err := http.NewRequest(http.MethodGet, c.BuildURL(urlPath), nil)
    # 如果创建请求时出现错误，则触发 panic
    if err != nil {
        panic(err)
    }
    # 遍历传入的选项函数，并将其应用到请求上
    for _, o := range opts {
        o(req)
    }
    # 发送请求并返回响应
    return c.Do(req)
}

# 定义 HTTPClient 结构体的 Post 方法，用于发送 HTTP POST 请求
func (c *HTTPClient) Post(urlPath string, body io.Reader, opts ...func(*http.Request)) *HTTPResponse {
    # 创建一个 HTTP POST 请求
    req, err := http.NewRequest(http.MethodPost, c.BuildURL(urlPath), body)
    # 如果创建请求时出现错误，则触发 panic
    if err != nil {
        panic(err)
    }
    # 遍历传入的选项函数，并将其应用到请求上
    for _, o := range opts {
        o(req)
    }
    # 发送请求并返回响应
    return c.Do(req)
}

# 定义 HTTPClient 结构体的 PostStr 方法，用于发送 HTTP POST 请求，参数为字符串类型的 body
func (c *HTTPClient) PostStr(urlpath, body string, opts ...func(*http.Request)) *HTTPResponse {
    # 将字符串类型的 body 转换为 io.Reader 接口类型
    r := strings.NewReader(body)
    # 调用 Post 方法发送 HTTP POST 请求并返回响应
    return c.Post(urlpath, r, opts...)
}

# 定义 HTTPClient 结构体的 Head 方法，用于发送 HTTP HEAD 请求
func (c *HTTPClient) Head(urlPath string, opts ...func(*http.Request)) *HTTPResponse {
    # 创建一个 HTTP HEAD 请求
    req, err := http.NewRequest(http.MethodHead, c.BuildURL(urlPath), nil)
    # 如果创建请求时出现错误，则触发 panic
    if err != nil {
        panic(err)
    }
    # 遍历传入的选项函数，并将其应用到请求上
    for _, o := range opts {
        o(req)
    }
    # 发送请求并返回响应
    return c.Do(req)
}
```