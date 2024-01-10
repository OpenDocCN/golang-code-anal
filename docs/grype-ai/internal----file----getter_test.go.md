# `grype\internal\file\getter_test.go`

```
package file

import (
    "archive/tar"  // 导入tar包，用于处理tar文件
    "bytes"  // 导入bytes包，用于操作字节
    "context"  // 导入context包，用于控制goroutine的执行
    "crypto/x509"  // 导入x509包，用于操作证书
    "fmt"  // 导入fmt包，用于格式化输入输出
    "net"  // 导入net包，用于网络通信
    "net/http"  // 导入http包，用于创建HTTP客户端和服务器
    "net/http/httptest"  // 导入httptest包，用于创建HTTP测试服务器
    "net/url"  // 导入url包，用于解析URL
    "path"  // 导入path包，用于处理文件路径
    "testing"  // 导入testing包，用于编写测试函数

    "github.com/stretchr/testify/assert"  // 导入assert包，用于编写断言
)

func TestGetter_GetFile(t *testing.T) {
    testCases := []struct {
        name          string
        prepareClient func(*http.Client)  // 准备HTTP客户端的函数
        assert        assert.ErrorAssertionFunc  // 断言函数
    }{
        {
            name:   "client trusts server's CA",  // 测试用例名称
            assert: assert.NoError,  // 断言函数，期望没有错误
        },
        {
            name:          "client doesn't trust server's CA",  // 测试用例名称
            prepareClient: removeTrustedCAs,  // 准备HTTP客户端的函数，移除信任的CA
            assert:        assertUnknownAuthorityError,  // 断言函数，期望未知授权错误
        },
    }

    for _, tc := range testCases {  // 遍历测试用例
        t.Run(tc.name, func(t *testing.T) {  // 运行测试用例
            requestPath := "/foo"  // 请求路径

            server := newTestServer(t, withResponseForPath(t, requestPath, testFileContent))  // 创建测试服务器
            t.Cleanup(server.Close)  // 在测试结束时关闭服务器

            httpClient := getClient(t, server)  // 获取HTTP客户端
            if tc.prepareClient != nil {  // 如果有准备HTTP客户端的函数
                tc.prepareClient(httpClient)  // 执行准备HTTP客户端的函数
            }

            getter := NewGetter(httpClient)  // 创建文件获取器
            requestURL := createRequestURL(t, server, requestPath)  // 创建请求URL

            tempDir := t.TempDir()  // 创建临时目录
            tempFile := path.Join(tempDir, "some-destination-file")  // 创建临时文件路径

            err := getter.GetFile(tempFile, requestURL)  // 获取文件
            tc.assert(t, err)  // 断言结果
        })
    }
}

func TestGetter_GetToDir_FilterNonArchivesWired(t *testing.T) {
    testCases := []struct {
        name   string
        source string
        assert assert.ErrorAssertionFunc
    }{
        {
            name:   "error out on non-archive sources",  // 测试用例名称
            source: "http://localhost/something.txt",  // 非归档源
            assert: assertErrNonArchiveSource,  // 断言函数，期望非归档源错误
        },
    }

    for _, test := range testCases {  // 遍历测试用例
        t.Run(test.name, func(t *testing.T) {  // 运行测试用例
            test.assert(t, NewGetter(nil).GetToDir(t.TempDir(), test.source))  // 断言结果
        })
    }
}
func TestGetter_validateHttpSource(t *testing.T) {
    // 定义测试用例
    testCases := []struct {
        name   string
        source string
        assert assert.ErrorAssertionFunc
    }{
        {
            name:   "error out on non-archive sources",
            source: "http://localhost/something.txt",
            assert: assertErrNonArchiveSource,
        },
        {
            name:   "filter out non-archive sources with get param",
            source: "https://localhost/vulnerability-db_v3_2021-11-21T08:15:44Z.txt?checksum=sha256%3Ac402d01fa909a3fa85a5c6733ef27a3a51a9105b6c62b9152adbd24c08358911",
            assert: assertErrNonArchiveSource,
        },
        {
            name:   "ignore non http-https input",
            source: "s3://bucket/something.txt",
            assert: assert.NoError,
        },
    }

    // 遍历测试用例
    for _, test := range testCases {
        // 运行子测试
        t.Run(test.name, func(t *testing.T) {
            // 调用验证 HTTP 源的函数，并进行断言
            test.assert(t, validateHTTPSource(test.source))
        })
    }
}

func TestGetter_GetToDir_CertConcerns(t *testing.T) {
    // 定义测试用例
    testCases := []struct {
        name          string
        prepareClient func(*http.Client)
        assert        assert.ErrorAssertionFunc
    }{

        {
            name:   "client trusts server's CA",
            assert: assert.NoError,
        },
        {
            name:          "client doesn't trust server's CA",
            prepareClient: removeTrustedCAs,
            assert:        assertUnknownAuthorityError,
        },
    }
}
    # 遍历测试用例切片，每个测试用例包含名称和函数
    for _, tc := range testCases {
        # 使用测试用例的名称创建子测试，运行子测试函数
        t.Run(tc.name, func(t *testing.T) {
            # 设置请求路径
            requestPath := "/foo.tar"
            # 创建名为"foo"的 tar 文件，内容为 testFileContent
            tarball := createTarball("foo", testFileContent)

            # 创建测试服务器，并设置请求路径的响应为 tar 文件
            server := newTestServer(t, withResponseForPath(t, requestPath, tarball))
            # 在测试结束时关闭服务器
            t.Cleanup(server.Close)

            # 获取 HTTP 客户端
            httpClient := getClient(t, server)
            # 如果测试用例中有准备客户端的函数，则执行
            if tc.prepareClient != nil {
                tc.prepareClient(httpClient)
            }

            # 创建 Getter 对象
            getter := NewGetter(httpClient)
            # 创建请求 URL
            requestURL := createRequestURL(t, server, requestPath)

            # 创建临时目录
            tempDir := t.TempDir()

            # 使用 Getter 对象从请求 URL 下载文件到临时目录
            err := getter.GetToDir(tempDir, requestURL)
            # 断言测试结果
            tc.assert(t, err)
        })
    }
}

# 断言未知授权错误
func assertUnknownAuthorityError(t assert.TestingT, err error, _ ...interface{}) bool {
    return assert.ErrorAs(t, err, &x509.UnknownAuthorityError{})
}

# 断言非存档源错误
func assertErrNonArchiveSource(t assert.TestingT, err error, _ ...interface{}) bool {
    return assert.ErrorIs(t, err, ErrNonArchiveSource)
}

# 移除信任的 CA
func removeTrustedCAs(client *http.Client) {
    client.Transport.(*http.Transport).TLSClientConfig.RootCAs = x509.NewCertPool()
}

# 创建 tarball，将其作为字节切片返回
func createTarball(filename string, content []byte) []byte {
    tarBuffer := new(bytes.Buffer)
    tarWriter := tar.NewWriter(tarBuffer)
    tarWriter.WriteHeader(&tar.Header{
        Name: filename,
        Size: int64(len(content)),
        Mode: 0600,
    })
    tarWriter.Write(content)
    tarWriter.Close()

    return tarBuffer.Bytes()
}

# mux 选项
type muxOption func(mux *http.ServeMux)

# 为路径创建响应
func withResponseForPath(t *testing.T, path string, response []byte) muxOption {
    t.Helper()

    return func(mux *http.ServeMux) {
        mux.HandleFunc(path, func(w http.ResponseWriter, req *http.Request) {
            t.Logf("server handling request: %s %s", req.Method, req.URL)

            _, err := w.Write(response)
            if err != nil {
                t.Fatal(err)
            }
        })
    }
}

# 创建测试服务器
func newTestServer(t *testing.T, muxOptions ...muxOption) *httptest.Server {
    t.Helper()

    mux := http.NewServeMux()
    for _, option := range muxOptions {
        option(mux)
    }

    server := httptest.NewTLSServer(mux)
    t.Logf("new TLS server listening at %s", getHost(t, server))

    return server
}

# 创建请求 URL
func createRequestURL(t *testing.T, server *httptest.Server, path string) string {
    t.Helper()

    # TODO: Figure out how to get this value from the server without hardcoding it here
    const testServerCertificateName = "example.com"

    serverURL, err := url.Parse(server.URL)
    if err != nil {
        t.Fatal(err)
    }
}
    // 将服务器URL的主机名设置为TLS证书中的值
    serverURL.Host = fmt.Sprintf("%s:%s", testServerCertificateName, serverURL.Port())

    // 设置服务器URL的路径
    serverURL.Path = path

    // 返回完整的服务器URL字符串
    return serverURL.String()
// getClient 返回一个可以用来联系测试 TLS 服务器的 http.Client
func getClient(t *testing.T, server *httptest.Server) *http.Client {
    t.Helper()

    // 从服务器的 http.Client 中获取 transport
    httpClient := server.Client()
    transport := httpClient.Transport.(*http.Transport)

    // 获取服务器的主机地址
    serverHost := getHost(t, server)

    // 重写 transport 的 DialContext 方法，确保客户端拨号到我们的测试服务器
    transport.DialContext = func(_ context.Context, _, addr string) (net.Conn, error) {
        t.Logf("client dialing %q for host %q", serverHost, addr)

        // 确保客户端拨号到我们的测试服务器
        return net.Dial("tcp", serverHost)
    }

    return httpClient
}

// getHost 从服务器 URL 字符串中提取主机值
// 例如，给定 URL 为 "http://1.2.3.4:5000/foo" 的服务器，getHost 返回 "1.2.3.4:5000"
func getHost(t *testing.T, server *httptest.Server) string {
    t.Helper()

    // 解析服务器的 URL，获取主机名和端口
    u, err := url.Parse(server.URL)
    if err != nil {
        t.Fatal(err)
    }

    return u.Hostname() + ":" + u.Port()
}

// testFileContent 是一个测试文件的内容的字节数组
var testFileContent = []byte("This is the content of a test file!\n")
```