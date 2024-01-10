# `v2ray-core\transport\internet\headers\http\http.go`

```
package http

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
    "bufio" // 导入 bufio 包，提供了缓冲读取功能
    "bytes" // 导入 bytes 包，提供了操作字节切片的函数
    "context" // 导入 context 包，提供了跟踪请求的上下文
    "io" // 导入 io 包，提供了基本的 I/O 接口
    "net" // 导入 net 包，提供了网络相关的函数和接口
    "net/http" // 导入 net/http 包，提供了 HTTP 客户端和服务端的实现
    "strings" // 导入 strings 包，提供了操作字符串的函数
    "time" // 导入 time 包，提供了时间相关的函数

    "v2ray.com/core/common" // 导入 v2ray.com/core/common 包
    "v2ray.com/core/common/buf" // 导入 v2ray.com/core/common/buf 包
)

const (
    // CRLF 是 HTTP 头部的行结束符
    CRLF = "\r\n"

    // ENDING 是 HTTP 头部和主体之间的双行结束符
    ENDING = CRLF + CRLF

    // HTTP 头部的最大长度。用于防范 DDoS 攻击
    maxHeaderLength = 8192
)

var (
    ErrHeaderToLong = newError("Header too long.") // 定义一个错误变量，表示头部过长
    ErrHeaderMisMatch = newError("Header Mismatch.") // 定义一个错误变量，表示头部不匹配
)

type Reader interface {
    Read(io.Reader) (*buf.Buffer, error) // 定义一个接口，包含读取方法
}

type Writer interface {
    Write(io.Writer) error // 定义一个接口，包含写入方法
}

type NoOpReader struct{} // 定义一个空的读取器结构体

func (NoOpReader) Read(io.Reader) (*buf.Buffer, error) { // 实现空的读取器的读取方法
    return nil, nil
}

type NoOpWriter struct{} // 定义一个空的写入器结构体

func (NoOpWriter) Write(io.Writer) error { // 实现空的写入器的写入方法
    return nil
}

type HeaderReader struct {
    req            *http.Request // HTTP 请求对象
    expectedHeader *RequestConfig // 期望的请求头部配置
}

func (h *HeaderReader) ExpectThisRequest(expectedHeader *RequestConfig) *HeaderReader { // 设置期望的请求头部配置
    h.expectedHeader = expectedHeader
    return h
}

func (h *HeaderReader) Read(reader io.Reader) (*buf.Buffer, error) { // 读取方法
    buffer := buf.New() // 创建一个新的缓冲区
    totalBytes := int32(0) // 总字节数
    endingDetected := false // 是否检测到结束标志

    var headerBuf bytes.Buffer // 创建一个字节缓冲区
    // 当 totalBytes 小于 maxHeaderLength 时执行循环
    for totalBytes < maxHeaderLength {
        // 从 reader 中读取数据到 buffer 中
        _, err := buffer.ReadFrom(reader)
        // 如果读取出错，则释放 buffer 并返回错误
        if err != nil {
            buffer.Release()
            return nil, err
        }
        // 在 buffer 中查找 ENDING 的位置
        if n := bytes.Index(buffer.Bytes(), []byte(ENDING)); n != -1 {
            // 将 buffer 中的数据写入 headerBuf 中，并更新 buffer
            headerBuf.Write(buffer.BytesRange(0, int32(n+len(ENDING))))
            buffer.Advance(int32(n + len(ENDING)))
            endingDetected = true
            break
        }
        lenEnding := int32(len(ENDING))
        // 如果 buffer 中的长度大于等于 lenEnding
        if buffer.Len() >= lenEnding {
            // 更新 totalBytes，并将 buffer 中的数据写入 headerBuf 中
            totalBytes += buffer.Len() - lenEnding
            headerBuf.Write(buffer.BytesRange(0, buffer.Len()-lenEnding))
            leftover := buffer.BytesFrom(-lenEnding)
            buffer.Clear()
            copy(buffer.Extend(lenEnding), leftover)

            // 从 headerBuf 中读取请求并检查是否出现意外的 EOF 错误
            if _, err := readRequest(bufio.NewReader(bytes.NewReader(headerBuf.Bytes())), false); err != io.ErrUnexpectedEOF {
                return nil, err
            }
        }
    }

    // 如果未检测到结束标志，则释放 buffer 并返回 ErrHeaderToLong 错误
    if !endingDetected {
        buffer.Release()
        return nil, ErrHeaderToLong
    }

    // 如果 expectedHeader 为 nil
    if h.expectedHeader == nil {
        // 如果 buffer 为空，则释放 buffer 并返回 nil
        if buffer.IsEmpty() {
            buffer.Release()
            return nil, nil
        }
        // 返回 buffer
        return buffer, nil
    }

    // 解析请求
    if req, err := readRequest(bufio.NewReader(bytes.NewReader(headerBuf.Bytes())), false); err != nil {
        return nil, err
    } else {
        h.req = req
    }

    // 检查请求
    path := h.req.URL.Path
    hasThisUri := false
    for _, u := range h.expectedHeader.Uri {
        if u == path {
            hasThisUri = true
        }
    }

    // 如果请求的路径不匹配，则返回 ErrHeaderMisMatch 错误
    if !hasThisUri {
        return nil, ErrHeaderMisMatch
    }

    // 如果 buffer 为空，则释放 buffer 并返回 nil
    if buffer.IsEmpty() {
        buffer.Release()
        return nil, nil
    }

    // 返回 buffer
    return buffer, nil
}

type HeaderWriter struct {
    header *buf.Buffer
}

func NewHeaderWriter(header *buf.Buffer) *HeaderWriter {
    return &HeaderWriter{
        header: header,
    }
}

func (w *HeaderWriter) Write(writer io.Writer) error {
    if w.header == nil {
        return nil
    }
    // 将头部数据写入到指定的 writer 中
    err := buf.WriteAllBytes(writer, w.header.Bytes())
    // 释放头部数据的缓冲区
    w.header.Release()
    w.header = nil
    return err
}

type HttpConn struct {
    net.Conn

    readBuffer          *buf.Buffer
    oneTimeReader       Reader
    oneTimeWriter       Writer
    errorWriter         Writer
    errorMismatchWriter Writer
    errorTooLongWriter  Writer

    errReason error
}

func NewHttpConn(conn net.Conn, reader Reader, writer Writer, errorWriter Writer, errorMismatchWriter Writer, errorTooLongWriter Writer) *HttpConn {
    return &HttpConn{
        Conn:                conn,
        oneTimeReader:       reader,
        oneTimeWriter:       writer,
        errorWriter:         errorWriter,
        errorMismatchWriter: errorMismatchWriter,
        errorTooLongWriter:  errorTooLongWriter,
    }
}

func (c *HttpConn) Read(b []byte) (int, error) {
    if c.oneTimeReader != nil {
        // 从连接中读取数据
        buffer, err := c.oneTimeReader.Read(c.Conn)
        if err != nil {
            c.errReason = err
            return 0, err
        }
        c.readBuffer = buffer
        c.oneTimeReader = nil
    }

    if !c.readBuffer.IsEmpty() {
        // 从读取缓冲区中读取数据到指定的字节数组中
        nBytes, _ := c.readBuffer.Read(b)
        if c.readBuffer.IsEmpty() {
            // 释放读取缓冲区
            c.readBuffer.Release()
            c.readBuffer = nil
        }
        return nBytes, nil
    }

    return c.Conn.Read(b)
}

// Write implements io.Writer.
func (c *HttpConn) Write(b []byte) (int, error) {
    if c.oneTimeWriter != nil {
        // 将数据写入到连接中
        err := c.oneTimeWriter.Write(c.Conn)
        c.oneTimeWriter = nil
        if err != nil {
            return 0, err
        }
    }

    return c.Conn.Write(b)
}

// Close implements net.Conn.Close().
func (c *HttpConn) Close() error {
    # 检查是否存在一次性写入器和错误写入器
    if c.oneTimeWriter != nil && c.errorWriter != nil:
        # 连接正在关闭，但是头部还没有发送。这意味着客户端请求可能无效。在这种情况下发送一个服务器错误头部。

        # 根据错误原因写入响应
        if c.errReason == ErrHeaderMisMatch:
            # 如果错误原因是头部不匹配，则使用错误不匹配写入器写入到连接中
            c.errorMismatchWriter.Write(c.Conn)
        elif c.errReason == ErrHeaderToLong:
            # 如果错误原因是头部过长，则使用错误过长写入器写入到连接中
            c.errorTooLongWriter.Write(c.Conn)
        else:
            # 否则使用一般错误写入器写入到连接中
            c.errorWriter.Write(c.Conn)
    
    # 关闭连接
    return c.Conn.Close()
}

// 根据配置生成响应头信息
func formResponseHeader(config *ResponseConfig) *HeaderWriter {
    // 创建一个缓冲区用于存储响应头信息
    header := buf.New()
    // 将版本号、状态码和状态原因拼接成响应头的第一行
    common.Must2(header.WriteString(strings.Join([]string{config.GetFullVersion(), config.GetStatusValue().Code, config.GetStatusValue().Reason}, " ")))
    common.Must2(header.WriteString(CRLF))

    // 获取配置中的响应头信息
    headers := config.PickHeaders()
    // 遍历响应头信息并添加到响应头中
    for _, h := range headers {
        common.Must2(header.WriteString(h))
        common.Must2(header.WriteString(CRLF))
    }
    // 如果响应头中没有包含日期信息，则添加当前日期
    if !config.HasHeader("Date") {
        common.Must2(header.WriteString("Date: "))
        common.Must2(header.WriteString(time.Now().Format(http.TimeFormat))
        common.Must2(header.WriteString(CRLF))
    }
    common.Must2(header.WriteString(CRLF))
    // 返回响应头信息
    return &HeaderWriter{
        header: header,
    }
}

// HTTP 认证结构体
type HttpAuthenticator struct {
    config *Config
}

// 获取客户端写入器
func (a HttpAuthenticator) GetClientWriter() *HeaderWriter {
    // 创建一个缓冲区用于存储客户端请求头信息
    header := buf.New()
    config := a.config.Request
    // 将请求方法、URI 和版本号拼接成请求头的第一行
    common.Must2(header.WriteString(strings.Join([]string{config.GetMethodValue(), config.PickUri(), config.GetFullVersion()}, " "))
    common.Must2(header.WriteString(CRLF))

    // 获取配置中的请求头信息
    headers := config.PickHeaders()
    // 遍历请求头信息并添加到请求头中
    for _, h := range headers {
        common.Must2(header.WriteString(h))
        common.Must2(header.WriteString(CRLF))
    }
    common.Must2(header.WriteString(CRLF))
    // 返回请求头信息
    return &HeaderWriter{
        header: header,
    }
}

// 获取服务器端写入器
func (a HttpAuthenticator) GetServerWriter() *HeaderWriter {
    // 根据配置生成响应头信息
    return formResponseHeader(a.config.Response)
}

// 客户端认证方法
func (a HttpAuthenticator) Client(conn net.Conn) net.Conn {
    // 如果请求和响应配置都为空，则直接返回连接
    if a.config.Request == nil && a.config.Response == nil {
        return conn
    }
    var reader Reader = NoOpReader{}
    // 如果请求配置不为空，则使用 HeaderReader 作为读取器
    if a.config.Request != nil {
        reader = new(HeaderReader)
    }

    var writer Writer = NoOpWriter{}
    // 如果响应配置不为空，则使用 GetClientWriter 方法生成写入器
    if a.config.Response != nil {
        writer = a.GetClientWriter()
    }
    // 创建一个新的 HTTP 连接并返回
    return NewHttpConn(conn, reader, writer, NoOpWriter{}, NoOpWriter{}, NoOpWriter{})
}
# 定义一个方法，用于处理服务器端的连接
func (a HttpAuthenticator) Server(conn net.Conn) net.Conn {
    # 如果配置中的请求和响应都为空，则直接返回连接
    if a.config.Request == nil && a.config.Response == nil {
        return conn
    }
    # 使用配置中的请求和服务器写入器创建一个新的 HTTP 连接
    return NewHttpConn(conn, new(HeaderReader).ExpectThisRequest(a.config.Request), a.GetServerWriter(),
        # 格式化 400 响应头
        formResponseHeader(resp400),
        # 格式化 404 响应头
        formResponseHeader(resp404),
        # 再次格式化 400 响应头
        formResponseHeader(resp400))
}

# 定义一个新的 HTTP 认证器，接受上下文和配置作为参数
func NewHttpAuthenticator(ctx context.Context, config *Config) (HttpAuthenticator, error) {
    # 返回一个新的 HTTP 认证器实例和空的错误
    return HttpAuthenticator{
        config: config,
    }, nil
}

# 初始化函数
func init() {
    # 注册配置并返回一个新的 HTTP 认证器实例
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return NewHttpAuthenticator(ctx, config.(*Config))
    }))
}
```