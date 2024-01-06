# `grype\internal\file\getter_test.go`

```
package file

import (
	"archive/tar" // 导入tar包，用于处理tar文件
	"bytes" // 导入bytes包，用于操作字节
	"context" // 导入context包，用于控制goroutine的执行
	"crypto/x509" // 导入x509包，用于操作证书
	"fmt" // 导入fmt包，用于格式化输出
	"net" // 导入net包，用于网络编程
	"net/http" // 导入http包，用于处理HTTP请求
	"net/http/httptest" // 导入httptest包，用于模拟HTTP请求
	"net/url" // 导入url包，用于解析URL
	"path" // 导入path包，用于处理文件路径
	"testing" // 导入testing包，用于编写测试用例

	"github.com/stretchr/testify/assert" // 导入assert包，用于编写断言
)

func TestGetter_GetFile(t *testing.T) {
	testCases := []struct {
```

注释：
这段代码是一个Go语言的测试函数，用于测试Getter的GetFile方法。其中包含了一系列的导入包和测试用例的定义。
# 定义一个结构体，包含三个字段：name（字符串类型）、prepareClient（函数类型，接受一个指向http.Client的指针作为参数）、assert（assert.ErrorAssertionFunc类型）
{
    # 第一个测试用例
    {
        name:   "client trusts server's CA",  # 测试用例的名称
        assert: assert.NoError,  # 断言函数，用于判断测试结果是否为无错误
    },
    # 第二个测试用例
    {
        name:          "client doesn't trust server's CA",  # 测试用例的名称
        prepareClient: removeTrustedCAs,  # 准备客户端的函数，用于移除信任的CA
        assert:        assertUnknownAuthorityError,  # 断言函数，用于判断测试结果是否为未知的授权错误
    },
}

# 遍历测试用例
for _, tc := range testCases {
    # 对每个测试用例运行子测试
    t.Run(tc.name, func(t *testing.T) {
        requestPath := "/foo"  # 请求路径

        # 创建一个测试服务器，并为指定路径设置响应内容
        server := newTestServer(t, withResponseForPath(t, requestPath, testFileContent))
# 在测试结束时关闭服务器
t.Cleanup(server.Close)

# 获取 HTTP 客户端
httpClient := getClient(t, server)

# 如果有准备客户端的函数，则调用该函数
if tc.prepareClient != nil:
    tc.prepareClient(httpClient)

# 创建 Getter 对象
getter := NewGetter(httpClient)

# 创建请求的 URL
requestURL := createRequestURL(t, server, requestPath)

# 创建临时目录
tempDir := t.TempDir()

# 创建临时文件路径
tempFile := path.Join(tempDir, "some-destination-file")

# 获取文件并保存到临时文件中
err := getter.GetFile(tempFile, requestURL)

# 断言获取文件的结果
tc.assert(t, err)
# 定义测试用例的结构体，包括名称、来源和断言函数
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
}

# 遍历测试用例，对每个测试用例运行测试
for _, test := range testCases {
    # 使用测试名称创建子测试，并运行断言函数
    t.Run(test.name, func(t *testing.T) {
        test.assert(t, NewGetter(nil).GetToDir(t.TempDir(), test.source))
    })
}

# 定义测试函数，用于验证 HTTP 来源
func TestGetter_validateHttpSource(t *testing.T) {
// 定义测试用例的结构体，包括名称、源数据和断言函数
testCases := []struct {
    name   string
    source string
    assert assert.ErrorAssertionFunc
}{
    // 第一个测试用例：在非存档源上出错
    {
        name:   "error out on non-archive sources",
        source: "http://localhost/something.txt",
        assert: assertErrNonArchiveSource,
    },
    // 第二个测试用例：使用 GET 参数过滤非存档源
    {
        name:   "filter out non-archive sources with get param",
        source: "https://localhost/vulnerability-db_v3_2021-11-21T08:15:44Z.txt?checksum=sha256%3Ac402d01fa909a3fa85a5c6733ef27a3a51a9105b6c62b9152adbd24c08358911",
        assert: assertErrNonArchiveSource,
    },
    // 第三个测试用例：忽略非 http-https 输入
    {
        name:   "ignore non http-https input",
        source: "s3://bucket/something.txt",
        assert: assert.NoError,
    },
}
	}

	for _, test := range testCases {
		t.Run(test.name, func(t *testing.T) {
			test.assert(t, validateHTTPSource(test.source))
		})
	}
}

func TestGetter_GetToDir_CertConcerns(t *testing.T) {
	testCases := []struct {
		name          string  // 测试用例名称
		prepareClient func(*http.Client)  // 准备客户端函数
		assert        assert.ErrorAssertionFunc  // 断言函数
	}{

		{
			name:   "client trusts server's CA",  // 测试用例名称
			assert: assert.NoError,  // 断言函数为无错误
		},
```

# 定义测试用例的数组，每个测试用例包括名称、客户端准备函数和断言函数
testCases := []struct {
    name          string           // 测试用例名称
    prepareClient func(client *http.Client) // 客户端准备函数
    assert        func(t *testing.T, err error) // 断言函数
}{
    {
        name:          "client doesn't trust server's CA", // 客户端不信任服务器的 CA
        prepareClient: removeTrustedCAs, // 准备客户端函数，移除信任的 CA
        assert:        assertUnknownAuthorityError, // 断言函数，断言未知的授权错误
    },
}

# 遍历测试用例数组
for _, tc := range testCases {
    # 使用测试用例的名称创建子测试
    t.Run(tc.name, func(t *testing.T) {
        # 设置请求路径和创建 tar 文件
        requestPath := "/foo.tar"
        tarball := createTarball("foo", testFileContent)

        # 创建测试服务器，并在路径上设置响应
        server := newTestServer(t, withResponseForPath(t, requestPath, tarball))
        t.Cleanup(server.Close) // 在测试结束时关闭服务器

        # 获取 HTTP 客户端，并根据测试用例准备客户端
        httpClient := getClient(t, server)
        if tc.prepareClient != nil {
            tc.prepareClient(httpClient)
        }
// 创建一个新的Getter对象，使用给定的httpClient
getter := NewGetter(httpClient)

// 创建请求的URL
requestURL := createRequestURL(t, server, requestPath)

// 创建临时目录
tempDir := t.TempDir()

// 使用Getter对象从请求URL获取数据到临时目录
err := getter.GetToDir(tempDir, requestURL)
// 断言检查错误
tc.assert(t, err)
```

```
// 断言检查错误是否为UnknownAuthorityError类型
func assertUnknownAuthorityError(t assert.TestingT, err error, _ ...interface{}) bool {
	return assert.ErrorAs(t, err, &x509.UnknownAuthorityError{})
}

// 断言检查错误是否为ErrNonArchiveSource类型
func assertErrNonArchiveSource(t assert.TestingT, err error, _ ...interface{}) bool {
	return assert.ErrorIs(t, err, ErrNonArchiveSource)
}

// 移除http.Client的TrustedCAs
func removeTrustedCAs(client *http.Client) {
// 设置客户端传输的根证书列表
client.Transport.(*http.Transport).TLSClientConfig.RootCAs = x509.NewCertPool()
}

// createTarball 创建一个单文件的tarball，并将其作为字节片返回
func createTarball(filename string, content []byte) []byte {
	// 创建一个新的字节缓冲区
	tarBuffer := new(bytes.Buffer)
	// 创建一个tar.Writer对象
	tarWriter := tar.NewWriter(tarBuffer)
	// 写入tar文件头信息
	tarWriter.WriteHeader(&tar.Header{
		Name: filename,
		Size: int64(len(content)),
		Mode: 0600,
	})
	// 写入文件内容
	tarWriter.Write(content)
	// 关闭tar.Writer
	tarWriter.Close()

	// 返回tar文件的字节片
	return tarBuffer.Bytes()
}

// muxOption 是一个函数类型，用于配置http.ServeMux
type muxOption func(mux *http.ServeMux)
// 为指定路径和响应数据创建一个 muxOption 函数
func withResponseForPath(t *testing.T, path string, response []byte) muxOption {
	// 标记该函数是测试辅助函数
	t.Helper()

	// 返回一个匿名函数，该函数会在 ServeMux 中注册指定路径的处理函数
	return func(mux *http.ServeMux) {
		// 注册处理函数，当请求路径匹配时，返回指定的响应数据
		mux.HandleFunc(path, func(w http.ResponseWriter, req *http.Request) {
			// 记录服务器处理请求的日志
			t.Logf("server handling request: %s %s", req.Method, req.URL)

			// 将响应数据写入到响应流中
			_, err := w.Write(response)
			// 如果写入过程中出现错误，则测试失败
			if err != nil {
				t.Fatal(err)
			}
		})
	}
}

// 创建一个新的测试服务器
func newTestServer(t *testing.T, muxOptions ...muxOption) *httptest.Server {
	// 标记该函数是测试辅助函数
	t.Helper()

	// 创建一个新的 ServeMux
	mux := http.NewServeMux()
	// 遍历传入的 muxOption 函数列表
	for _, option := range muxOptions {
		option(mux)
	}
```
这是一个函数的结尾，表示函数定义结束。

```
	server := httptest.NewTLSServer(mux)
	t.Logf("new TLS server listening at %s", getHost(t, server))

	return server
```
创建一个新的基于TLS的HTTP测试服务器，并打印服务器的监听地址，然后返回该服务器。

```
func createRequestURL(t *testing.T, server *httptest.Server, path string) string {
	t.Helper()

	// TODO: Figure out how to get this value from the server without hardcoding it here
	const testServerCertificateName = "example.com"
```
创建一个名为createRequestURL的函数，该函数接受一个testing.T类型的参数t，一个*httptest.Server类型的参数server和一个string类型的参数path，并返回一个string类型的值。t.Helper()用于标记该函数是测试辅助函数。testServerCertificateName是一个常量，表示测试服务器的证书名称。

```
serverURL, err := url.Parse(server.URL)
if err != nil {
	t.Fatal(err)
}
```
解析服务器的URL，并在出现错误时终止测试并打印错误信息。

// 将 URL 的主机名设置为 TLS 证书中的值
serverURL.Host = fmt.Sprintf("%s:%s", testServerCertificateName, serverURL.Port())

// 设置 URL 的路径
serverURL.Path = path

// 返回 URL 的字符串表示形式
return serverURL.String()
}

// getClient 返回一个可以用于联系测试 TLS 服务器的 http.Client
func getClient(t *testing.T, server *httptest.Server) *http.Client {
    t.Helper()

    // 获取测试服务器的 http.Client
    httpClient := server.Client()
    // 获取 http.Client 的传输层
    transport := httpClient.Transport.(*http.Transport)

    // 获取测试服务器的主机名
    serverHost := getHost(t, server)

    // 设置传输层的拨号上下文
    transport.DialContext = func(_ context.Context, _, addr string) (net.Conn, error) {
        t.Logf("client dialing %q for host %q", serverHost, addr)
// 确保客户端拨号到我们的测试服务器
return net.Dial("tcp", serverHost)
}

return httpClient
}

// getHost从服务器URL字符串中提取主机值。
// 例如，给定URL为"http://1.2.3.4:5000/foo"的服务器，getHost返回"1.2.3.4:5000"
func getHost(t *testing.T, server *httptest.Server) string {
t.Helper()

u, err := url.Parse(server.URL)
if err != nil {
t.Fatal(err)
}

return u.Hostname() + ":" + u.Port()
}
# 创建一个包含测试文件内容的字节切片
```