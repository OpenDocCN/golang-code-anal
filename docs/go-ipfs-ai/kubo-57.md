# go-ipfs 源码解析 57

# `/opt/kubo/test/cli/harness/http_client.go`

这段代码是一个名为“harness”的包，它定义了一个HTTP客户端和一个URL解析器。它的作用是帮助开发者测试HTTP客户端应用程序。

具体来说，这段代码实现了一个HTTP客户端（URL解析器和HTTP客户端）和一个通用的字符串模板引擎。HTTP客户端用于向远程服务器发送请求并获取响应，URL解析器用于解析和构建HTTP请求和响应的URL。模板引擎则提供了将字符串模板转换为字符串的语法，这对于将动态数据（如JSON或XML）与静态模板结合使用非常有用。

总的来说，这段代码主要作用是为开发者提供一个简单而强大的工具，以便在测试中构建HTTP客户端应用程序，并使应用程序在遇到内部错误时能够正常运行而不必关心错误细节。


```
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
```

该代码定义了一个名为 HTTPClient 的结构体，它用于表示 HTTP客户端。该结构体包含一个名为 Client 的指针，它指向一个 HTTPClient 类型的字段，以及一个名为 BaseURL 的字符串字段，它表示 HTTP 客户端的默认 base URL。此外，还包括一个名为 Timeout 的字段，它表示 HTTP 客户端超时的时间，以及一个名为 TemplateData 的字段，它表示 HTTP 请求的模板数据。

接着，定义了一个名为 HTTPResponse 的结构体，它用于表示 HTTP 响应。该结构体包含一个名为 Body 的字段，它表示 HTTP 响应的原始内容，以及一个名为 StatusCode 的字段，它表示 HTTP 响应的状态码。此外，还包括一个名为 Headers 的字段，它表示 HTTP 响应的 headers。最后，还包括一个名为 Resp 的字段，它表示一个 HTTPResponse 类型的字段，用于存储具体的 HTTP 响应对象。

整段代码的作用是定义了一个 HTTPClient 类型的 struct，它用于表示 HTTP 客户端，该 struct 中包含了一些与 HTTP 客户端相关的字段和类型，以及一个 HTTPResponse 类型的字段，用于存储 HTTP 响应的对象。该 struct 可以用于创建 HTTP 客户端对象，并通过一些字段设置 HTTP 客户端的一些参数，如超时时间、模板数据等。然后，定义了一个 HTTPResponse 类型，用于存储 HTTP 响应的对象，该类型包含了一些与 HTTP 响应相关的字段和类型，如响应的原始内容、状态码和 headers 等。最后，通过创建 HTTPClient 类型的实例，并设置其中的字段，可以创建一个 HTTP 客户端对象，并通过一些字段来设置该对象的参数，如超时时间、模板数据等，然后，该对象可以用来发送 HTTP 请求，并获取 HTTP 响应，其中 HTTPResponse 类型的字段用于存储 HTTP 响应的对象。


```
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

```

这段代码定义了两个函数，函数1接收一个HTTP客户端对象（c）和一个HTTP请求（h），返回一个带头信息（k和v）的函数，这个函数将添加k和v到请求的头部。函数2是一个disableRedirects函数，它将关闭客户端的redirection。

函数1中的代码将创建一个带有头部信息的HTTP客户端（c.client）后，将其头部设置为使用func函数作为依赖，将传入的（h）请求的头部添加到请求的头部。

函数2中的代码，首先，客户端检查redirection，然后设置为func函数作为依赖，使用func函数传递一个错误的（req， via）的函数作为处理因redirection错误。

总的来说，这两个函数主要作用于设置HTTP请求头和客户端的行为。


```
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

```

这段代码定义了一个名为`BuildURL`的函数，该函数接收一个URL路径作为参数，并将其构建为一个HTTP请求的URL。函数创建了一个字符串构建器`sb`，并使用`template.Must`调用了一个名为`test`的模板，然后传递给其中的`Parse`函数，该函数将URL路径作为参数并返回一个`test`字符串。如果模板解析失败，函数将抛出异常。

接下来，函数定义了一个名为`Get`的函数，该函数接收一个URL路径和一个或多个`http.Request`选项参数。函数创建一个HTTP请求并将其构建为上面定义的请求，然后使用`c.Do`方法发送请求。该请求的URL是上面定义的`BuildURL`函数的结果。

该代码片段的具体作用是，在向客户端发送请求之前，将请求的URL构建为将要发送的请求的URL。这样，客户端在构建URL之后，就可以将`url`参数传递给`Get`函数，请求将会根据传递的URL来发送请求。


```
// BuildURL constructs a request URL from the given path by interpolating the string and then appending it to the base URL.
func (c *HTTPClient) BuildURL(urlPath string) string {
	sb := &strings.Builder{}
	err := template.Must(template.New("test").Parse(urlPath)).Execute(sb, c.TemplateData)
	if err != nil {
		panic(err)
	}
	renderedPath := sb.String()
	return c.BaseURL + renderedPath
}

func (c *HTTPClient) Get(urlPath string, opts ...func(*http.Request)) *HTTPResponse {
	req, err := http.NewRequest(http.MethodGet, c.BuildURL(urlPath), nil)
	if err != nil {
		panic(err)
	}
	for _, o := range opts {
		o(req)
	}
	return c.Do(req)
}

```

这段代码定义了两个名为"func"的函数，分别接收一个HTTP客户端对象(c)和传递给请求选项的数组作为参数。

这两个函数都执行HTTP请求并返回HTTP响应。第一个函数名为"func(c *HTTPClient) Post"，其中c是一个HTTP客户端对象。这个函数接收一个URL路径字符串、一个字节输入流和一个选项的数组作为参数。如果传递的任何一个参数出现错误，函数将 panic。

第二个函数名为"func(c *HTTPClient) PostStr"，其中c是一个HTTP客户端对象。这个函数接收一个URL路径字符串和一个字节输入流和一个选项的数组作为参数。这个函数创建一个字符串输入流来读取传入的body，然后将请求发送到给定的URL路径，将输入流作为请求的正文。

在这两个函数中，都可以看到一个名为"opts..."的参数数组，但并没有定义该数组的下标或者类型，因此无法确定该数组的内容是什么。


```
func (c *HTTPClient) Post(urlPath string, body io.Reader, opts ...func(*http.Request)) *HTTPResponse {
	req, err := http.NewRequest(http.MethodPost, c.BuildURL(urlPath), body)
	if err != nil {
		panic(err)
	}
	for _, o := range opts {
		o(req)
	}
	return c.Do(req)
}

func (c *HTTPClient) PostStr(urlpath, body string, opts ...func(*http.Request)) *HTTPResponse {
	r := strings.NewReader(body)
	return c.Post(urlpath, r, opts...)
}

```

该函数`func`接收一个`*HTTPClient`类型的参数`c`，并接受一个`urlPath`字符串参数以及一个可选的`opts`参数列表。

该函数的作用是创建一个HTTP请求并返回一个`*HTTPResponse`类型的数据结构。具体来说，它执行以下步骤：

1. 创建一个名为`req`的HTTP请求对象，使用`http.MethodHead`设置请求方法为“GET”，并使用`c.BuildURL(urlPath)`构建请求的URL路径。

2. 如果步骤1中创建请求对象出现错误，它将抛出`err`异常并崩溃。

3. 遍历可选的`opts`参数列表，对每个函数`opts`中的函数应用到请求对象`req`上，以实现在函数`opts`中的所有请求选项。

4. 调用`c.Do`函数发送请求。

5. 如果请求发送成功，它将返回一个`*HTTPResponse`类型的数据结构。

因此，该函数的作用是创建一个HTTP请求并返回一个HTTP响应。


```
func (c *HTTPClient) Head(urlPath string, opts ...func(*http.Request)) *HTTPResponse {
	req, err := http.NewRequest(http.MethodHead, c.BuildURL(urlPath), nil)
	if err != nil {
		panic(err)
	}
	for _, o := range opts {
		o(req)
	}
	return c.Do(req)
}

```

# `/opt/kubo/test/cli/harness/ipfs.go`

这段代码是一个 Go 语言编写的测试 harness，用于在测试过程中运行 IPFS 命令行工具。具体来说，它实现了以下功能：

1. 导入了一个 " harness" 包，这个包可能包含其他依赖库或者工具。
2. 导入了 "encoding/json"、"fmt"、"io" 和 "reflect"，这些库可能用于 parsing 和格式化 JSON 数据。
3. 实现了 "Node" 类型的 IPFSNode 接口的 "IPFSCommands" 方法。该方法使用 IPFS "commands" 目录下的 JSON 数据，返回其中所有的命令。
4. 通过 "splitLines" 函数将 IPFS 输出结果字符串分割成多个行，然后遍历每个行，如果输出结果以 "ipfs" 开头，则跳过该行，否则将该行添加到 "cmds" 数组中。
5. 返回 "cmds" 数组，其中包含了所有在 IPFS 输出结果中出现的命令。


```
package harness

import (
	"encoding/json"
	"fmt"
	"io"
	"reflect"
	"strings"

	. "github.com/ipfs/kubo/test/cli/testutils"
)

func (n *Node) IPFSCommands() []string {
	res := n.IPFS("commands").Stdout.String()
	res = strings.TrimSpace(res)
	split := SplitLines(res)
	var cmds []string
	for _, line := range split {
		trimmed := strings.TrimSpace(line)
		if trimmed == "ipfs" {
			continue
		}
		cmds = append(cmds, trimmed)
	}
	return cmds
}

```

此函数的作用是设置IPFS（Interactive的追求无意义反馈的系统）的配置参数。它接受一个整数类型的参数n，以及两个接口类型的参数key和val，这些参数将作为设置的键和值传递给IPFS的配置函数。此函数首先将参数val编码为字节序列，然后将此值字符串化。接下来，它构建了一个字符串数组，其中包含用于设置IPFS配置的参数，然后将该数组传递给IPFS的config函数。

此函数的目的是在给定的键上设置IPFS的配置参数，然后在另一个函数中检查是否已经设置了正确的配置值。如果设置的值与获取的新的配置值不同，则函数将记录错误并输出。


```
func (n *Node) SetIPFSConfig(key string, val interface{}, flags ...string) {
	valBytes, err := json.Marshal(val)
	if err != nil {
		log.Panicf("marshling config for key '%s': %s", key, err)
	}
	valStr := string(valBytes)

	args := []string{"config", "--json"}
	args = append(args, flags...)
	args = append(args, key, valStr)
	n.IPFS(args...)

	// validate the config was set correctly
	var newVal string
	n.GetIPFSConfig(key, &newVal)
	if val != newVal {
		log.Panicf("key '%s' did not retain value '%s' after it was set, got '%s'", key, val, newVal)
	}
}

```

此函数的作用是获取IPFS Config中的一个指定键的值，并将结果返回。它接受两个参数：一个指向Node类型的n变量和一个字符串类型的参数val。

具体来说，函数首先通过n的IPFS方法调用，传递一个名为"config"的键和一个名为val的参数。然后，它获取配置响应的stdout字符串，并使用strings.TrimSpace函数将其字符串截去不必要的空格。

接下来，函数检查返回的结果是否为字符串类型，如果是，就执行以下操作：将结果字符串中的所有字符串截去双引号，并将"/"加入其中。这样，当返回的结果是一个字符串类型时，它就可以作为一个JSON对象使用。

如果返回的结果不是字符串类型，那么函数会尝试将结果解包为JSON对象，并检查它是否预期是一个字符串。如果是，函数会执行以下操作：将结果字符串中的所有字符串截去双引号，并将"/"加入其中。这样，当返回的结果是一个字符串类型时，它就可以作为一个JSON对象使用。

最后，函数使用json.Unmarshal函数将结果JSON对象解包为原始类型，并返回结果。如果解包过程中出现错误，函数会打印错误消息并返回一个错误。


```
func (n *Node) GetIPFSConfig(key string, val interface{}) {
	res := n.IPFS("config", key)
	valStr := strings.TrimSpace(res.Stdout.String())
	// only when the result is a string is the result not well-formed JSON,
	// so check the value type and add quotes if it's expected to be a string
	reflectVal := reflect.ValueOf(val)
	if reflectVal.Kind() == reflect.Ptr && reflectVal.Elem().Kind() == reflect.String {
		valStr = fmt.Sprintf(`"%s"`, valStr)
	}
	err := json.Unmarshal([]byte(valStr), val)
	if err != nil {
		log.Fatalf("unmarshaling config for key '%s', value '%s': %s", key, valStr, err)
	}
}

```

这两函数的作用是协同工作，其中第一函数`func (n *Node) IPFSAddStr(content string, args ...string) string`接收一个内容字符串参数`content`以及多个传递给第二函数的参数`args...string`，并将其添加到IPFS文件系统中。第二函数`func (n *Node) IPFSAdd(content io.Reader, args ...string) string`接收一个IPFS文件系统的内容`content`和一个或多个传递给它的参数`args...string`，并返回添加内容的元数据。

具体来说，这两函数的作用是将接收到的内容字符串`content`作为参数传递给第二函数，第二函数在这个基础上执行了IPFS文件系统的`add`命令，以添加所选内容到IPFS文件系统中。然后，第二函数将返回所选内容的元数据，第一函数在这里获取了这个元数据并输出它。


```
func (n *Node) IPFSAddStr(content string, args ...string) string {
	log.Debugf("node %d adding content '%s' with args: %v", n.ID, PreviewStr(content), args)
	return n.IPFSAdd(strings.NewReader(content), args...)
}

func (n *Node) IPFSAdd(content io.Reader, args ...string) string {
	log.Debugf("node %d adding with args: %v", n.ID, args)
	fullArgs := []string{"add", "-q"}
	fullArgs = append(fullArgs, args...)
	res := n.Runner.MustRun(RunRequest{
		Path:    n.IPFSBin,
		Args:    fullArgs,
		CmdOpts: []CmdOpt{RunWithStdin(content)},
	})
	out := strings.TrimSpace(res.Stdout.String())
	log.Debugf("add result: %q", out)
	return out
}

```

此代码是一个名为 `func` 的函数，接受一个整数参数 `n` 和一个字符串参数 `cid`，以及一个或多个参数 `args`。函数的作用是导入 IPFSDag 数据树，将数据树导出为 `content` 类型的 io.Reader，并将其保存在名为 `cid` 的文件中。

函数的具体实现包括以下几个步骤：

1. 记录导入数据的日志信息，包括节点 ID 和参数列表。
2. 将参数列表转换成字符串数组，并追加 `dag` 和 `import` 参数。
3. 使用 `n.Runner` 的 `MustRun` 函数发起一个命令行操作，将 `fullArgs` 参数的每个参数传递给 `RunWithStdin` 函数，其中 `content` 参数使用 stdin输入。
4. 使用 `n.Runner` 的 `MustRun` 函数发起另一个命令行操作，将参数 `cid` 传递给 `RunWithStdin` 函数，其中 `--offline` 参数使用 `cid` 文件名。
5. 如果两个步骤都成功，返回 `n.Runner` 的 `MustRun` 函数返回的错误值。

函数的实现基于 `io.Reader`、`io.Error` 和 `os.Run` 包，以及 `github.com/ethereum/go-ipfs/v3/log` 包中的 `log.Debugf` 函数。


```
func (n *Node) IPFSDagImport(content io.Reader, cid string, args ...string) error {
	log.Debugf("node %d dag import with args: %v", n.ID, args)
	fullArgs := []string{"dag", "import", "--pin-roots=false"}
	fullArgs = append(fullArgs, args...)
	res := n.Runner.MustRun(RunRequest{
		Path:    n.IPFSBin,
		Args:    fullArgs,
		CmdOpts: []CmdOpt{RunWithStdin(content)},
	})
	if res.Err != nil {
		return res.Err
	}
	res = n.Runner.MustRun(RunRequest{
		Path: n.IPFSBin,
		Args: []string{"block", "stat", "--offline", cid},
	})
	return res.Err
}

```

# `/opt/kubo/test/cli/harness/log.go`

该代码包名为“harness”，其中包含了以下功能：

1. 导入了一个名为“event”的结构体类型，定义了timestamp和msg两个字段。
2. 导入了一个名为“fmt”的函数类型，用于输出信息。
3. 导入了一个名为“path/filepath”的包，用于文件路径操作。
4. 导入了一个名为“runtime”的包，用于与操作系统交互，以执行时间相关的操作。
5. 导入了一个名为“sort”的包，用于对一个名为“strings”的包中的字符串进行排序。
6. 导入了一个名为“sync”的包，用于与多个进程或机器交互。
7. 导入了一个名为“testing”的包，用于对一个名为“time”的包中的时间进行测试。
8. 在自己的包目录下创建了一个名为“test”的目录。
9. 在“test”目录中创建了一个名为“test_utils.go”的文件，并在其中定义了一个名为“Event”的结构体类型，该类型包含了一个名为“timestamp”和一个名为“msg”的字段。
10. 在“test_utils.go”文件中，定义了一个名为“main”的函数，该函数创建了一个名为“event_data”的数组，该数组长度为10，然后将数组中的每个元素打印出来，以便于测试。
11. 在“main”函数中，首先创建了一个名为“event_data”的数组，并将数组长度设置为10。
12. 接着，定义了一个名为“timestamp_data”的变量，并将其设置为当前时间的时间戳。
13. 然后，定义了一个名为“msg_data”的变量，并将其设置为一个字符串“Harness”。
14. 将打印出的内容作为参数，调用了一个名为“fmt.Println”的函数，并打印了数组中所有的元素。
15. 最后，在“main”函数中，定义了一个名为“test_確保”的函数，该函数创建了一个名为“test_case”的数组，该数组长度为10，然后将数组中的每个元素打印出来，以确认是否可以正确地打印出包含两个字段的事件数组。
16. 在“test_ensure”函数中，创建了一个名为“test_case”的数组，该数组长度为10，然后将数组中的每个元素打印出来，并使用一个名为“sort.Strings”的函数对字符串进行排序。
17. 最后，通过调用一个名为“runtime.Wait”的函数，来等待当前操作系统的命令行窗口加载完毕，然后返回一个布尔值，表明所有的测试都已完成，函数不会继续执行。


```
package harness

import (
	"fmt"
	"path/filepath"
	"runtime"
	"sort"
	"strings"
	"sync"
	"testing"
	"time"
)

type event struct {
	timestamp time.Time
	msg       string
}

```

这段代码定义了一个名为`events`的类型，该类型有一个名为`event`的*指针。

该代码实现了两个函数：

1. `Len()`函数，返回`events`中`event`类型元素的个数。
2. `Less()`函数，比较两个`event`类型元素的索引，返回它们的`timestamp`值是否在当前时间之前。
3. `Swap()`函数，交换两个`event`类型元素的索引。

此外，还实现了一个名为`TestLogger`的测试日志器，用于在测试中缓冲输出，只有在测试失败或日志打开时才输出。

该日志器的作用是让Go测试运行时能够使用`--verbose`标志，而不会打印日志。

该日志器将输出消息作为一个字符串，并在消息前面添加了`prefix`。

该日志器将消息发送到其父进程，并在传递给子进程之前将其传递给子进程。

该代码的目的是实现一个简单的日志记录器，用于在Go测试中记录测试过程中发生的事件。


```
type events []*event

func (e events) Len() int           { return len(e) }
func (e events) Less(i, j int) bool { return e[i].timestamp.Before(e[j].timestamp) }
func (e events) Swap(i, j int)      { e[i], e[j] = e[j], e[i] }

// TestLogger is a logger for tests.
// It buffers output and only writes the output if the test fails or output is explicitly turned on.
// The purpose of this logger is to allow Go test to run with the verbose flag without printing logs.
// The verbose flag is useful since it streams test progress, but also printing logs makes the output too verbose.
//
// You can also add prefixes that are prepended to each log message, for extra logging context.
//
// This is implemented as a hierarchy of loggers, with children flushing log entries back to parents.
// This works because t.Cleanup() processes entries in LIFO order, so children always flush first.
```

这是一个用Go语言编写的测试logger类型，它的作用是协调和记录测试中的日志输出。

该logger使用了一个TestLogger结构体，其中包含以下字段：

- parent：指向其父logger的引用，如果这个logger是父logger的最后一个实例，则可以设置为nil。
- children：记录日志输出的子logger结构体，通过set方法将子logger添加到这个结构体中。
- prefixes：一个长度为len(prefixes)的切片，用于在输出日志前添加前缀。
- prefixesIface：一个代表prefixes切片的类型，接受任何类型的切片作为参数。
- t：一个指向testing.T的引用，用于记录日志。
- buf：一个长度为16的缓冲区，用于保存日志。
- logsEnabled：一个布尔值，表示是否启用日志记录。

该logger还重写了两个方法：

- new：创建一个新的logger实例，并初始化其字段。
- cleanup：清除内存，将日志缓冲区清空。

该logger的主要作用是在测试过程中记录日志，并提供了方法来设置和清空日志缓冲区。通过使用这些方法，可以方便地在测试过程中记录和查看日志，以便更好地调试和解决问题。


```
//
// Obviously this logger should never be used in production systems.
type TestLogger struct {
	parent        *TestLogger
	children      []*TestLogger
	prefixes      []string
	prefixesIface []any
	t             *testing.T
	buf           events
	m             sync.Mutex
	logsEnabled   bool
}

func NewTestLogger(t *testing.T) *TestLogger {
	l := &TestLogger{t: t, buf: make(events, 0)}
	t.Cleanup(l.flush)
	return l
}

```

这段代码定义了一个名为`func`的函数，接收一个名为`t`的`TestLogger`类型的参数。函数的作用是创建一个输出字符串，表示给定的`timestamp`和一系列`args`。

具体来说，函数的实现包括以下步骤：

1. 将`timestamp`的`2006-01-02T15:04:05.999999`格式化日期时间字符串，得到一个`d`字符串。
2. 根据步骤1生成的`d`字符串，创建一个表示调用者文件的路径。
3. 根据步骤2的生成的文件路径，创建一个表示行号`lineno`的路径。
4. 根据`filepath.Base`函数获取步骤3的文件路径，并将其作为参数传递给`fmt.Sprintf`函数，得到一个包含`file`和`lineno`的字符串。
5. 根据步骤1中`len`函数的返回值，如果`t.prefixes`为空，则返回`d`和`caller`之间的`:`字符串。
6. 根据步骤1中`fmt.Sprint`函数的返回值，将`t.prefixes`中的所有字符串连接起来，并返回一个包含`d`、`caller`和连接好的字符串的字符串。
7. 根据步骤1中`fmt.Sprintf`函数的第二个参数，将`d`、`caller`和`t.prefixes`中的字符串格式化，得到一个字符串。
8. 根据步骤4的`file`路径，调用`t.log`函数，并将`d`、`caller`和`e`作为参数传递给`t.log`函数。


```
func (t *TestLogger) buildPrefix(timestamp time.Time) string {
	d := timestamp.Format("2006-01-02T15:04:05.999999")
	_, file, lineno, _ := runtime.Caller(2)
	file = filepath.Base(file)
	caller := fmt.Sprintf("%s:%d", file, lineno)

	if len(t.prefixes) == 0 {
		return fmt.Sprintf("%s\t%s\t", d, caller)
	}

	prefixes := strings.Join(t.prefixes, ":")
	return fmt.Sprintf("%s\t%s\t%s: ", d, caller, prefixes)
}

func (t *TestLogger) Log(args ...any) {
	timestamp := time.Now()
	e := t.buildPrefix(timestamp) + fmt.Sprint(args...)
	t.add(&event{timestamp: timestamp, msg: e})
}

```

这是一个通用的日志函数，接受一个由 `any` 类型组成的参数 `args` 和一个格式字符串 `format`。这个函数的作用是创建一个 `TestLogger` 实例的 `Logf` 函数。

具体来说，这个函数的主要作用是记录一条日志，其中包括以下信息：

1. 获取当前时间戳 `timestamp`；
2. 使用 `t.buildPrefix` 函数将时间戳格式化，加上 `format` 中指定的格式字符串，以及 `args` 中的参数；
3. 将格式化后的字符串和参数添加到 `event` 结构体中，用于记录日志；
4. 将 `event` 结构体添加到 `t.t.Logger` 实例的 `events` 数组中，以便进行 `Fatal` 和 ` Fatalf` 函数的调用；
5. 如果 `t.t.Logger` 实例没有设置 `output` 选项为 `null`，则输出创建的 `event` 结构体；
6. 调用 `t.t.Logger.Log` 函数，传递创建的 `event` 结构体，以便输出更多的日志信息。

在 `Fatal` 和 `Fatalf` 函数中，输出的信息包含：

1. 获取当前时间戳 `timestamp`；
2. 使用 `t.buildPrefix` 函数将时间戳格式化，加上 `format` 中指定的格式字符串，以及 `args` 中的参数；
3. 将格式化后的字符串和参数添加到 `event` 结构体中，用于记录日志；
4. 将 `event` 结构体添加到 `t.t.Logger` 实例的 `events` 数组中，以便进行 `Fatal` 函数的调用；
5. 如果 `t.t.Logger` 实例没有设置 `output` 选项为 `null`，则输出创建的 `event` 结构体；
6. 调用 `t.t.Logger.Fatal` 函数，传递创建的 `event` 结构体，以便输出更多的日志信息；
7. 如果 `t.t.Logger` 实例已经设置 `output` 选项为 `null`，则不输出任何信息。


```
func (t *TestLogger) Logf(format string, args ...any) {
	timestamp := time.Now()
	e := t.buildPrefix(timestamp) + fmt.Sprintf(format, args...)
	t.add(&event{timestamp: timestamp, msg: e})
}

func (t *TestLogger) Fatal(args ...any) {
	timestamp := time.Now()
	e := t.buildPrefix(timestamp) + fmt.Sprint(append([]any{"fatal: "}, args...)...)
	t.add(&event{timestamp: timestamp, msg: e})
	t.t.FailNow()
}

func (t *TestLogger) Fatalf(format string, args ...any) {
	timestamp := time.Now()
	e := t.buildPrefix(timestamp) + fmt.Sprintf(fmt.Sprintf("fatal: %s", format), args...)
	t.add(&event{timestamp: timestamp, msg: e})
	t.t.FailNow()
}

```

这两段代码都是定义了一个名为 TestLogger 的类，用于记录测试过程中的日志信息。

第一段代码 `func (t *TestLogger) add(e *event)` 接收一个 `event` 类型的参数 `e`，并使用 `t.m.Lock()` 函数获取线程安全锁，确保在添加日志信息的时候只有一个线程在进行操作。接着，使用 `t.buf` 数组将 `e` 添加到 `t.buf` 中，最后使用 `defer t.m.Unlock()` 来确保在函数内部释放锁。

第二段代码 `func (t *TestLogger) AddPrefix(prefix string) *TestLogger` 接收一个前缀字符串 `prefix`，并使用 `t.m.Lock()` 函数获取线程安全锁。在这个函数内部，使用 `t.prefixes` 和 `t.prefixesIface` 数组将前缀字符串存储到 `t.prefixes` 和 `t.prefixesIface` 中，并将它们与当前的日志信息合并。接着，创建一个新的 `TestLogger` 对象 `l`，并将 `l` 存储到 `t.children` 数组中，`t.t.Cleanup(l.flush)` 函数确保在日志信息中不会保留父日志信息。最后，返回新创建的 `l` 对象。


```
func (t *TestLogger) add(e *event) {
	t.m.Lock()
	defer t.m.Unlock()
	t.buf = append(t.buf, e)
}

func (t *TestLogger) AddPrefix(prefix string) *TestLogger {
	l := &TestLogger{
		prefixes:      append(t.prefixes, prefix),
		prefixesIface: append(t.prefixesIface, prefix),
		t:             t.t,
		parent:        t,
		logsEnabled:   t.logsEnabled,
	}
	t.m.Lock()
	defer t.m.Unlock()

	t.children = append(t.children, l)
	t.t.Cleanup(l.flush)

	return l
}

```

这段代码定义了一个名为`func`的函数，接收一个名为`t`的指针变量作为参数，代表一个`TestLogger`类型的对象。

函数的作用是打开或关闭`t`指向对象的`logsEnabled`字段，如果`t`的`parent`字段存在并且`parent`对象的`logsEnabled`字段已启用，则调用`parent.EnableLogs()`函数。

函数还打印出`t`对象当前拥有的`children`数量，然后遍历`t`的`children`链，检查每个子对象是否启用了`logsEnabled`字段，如果子对象未启用，则调用子对象的`EnableLogs()`函数。


```
func (t *TestLogger) EnableLogs() {
	t.m.Lock()
	defer t.m.Unlock()
	t.logsEnabled = true
	if t.parent != nil {
		if t.parent.logsEnabled {
			t.parent.EnableLogs()
		}
	}
	fmt.Printf("enabling %d children\n", len(t.children))
	for _, c := range t.children {
		if !c.logsEnabled {
			c.EnableLogs()
		}
	}
}

```

该函数 `func (t *TestLogger) flush()` 作用是输出所有与 `t` 相关的日志信息。具体解释如下：

1. 首先判断 `t` 是否已经设置好了日志输出开关 `t.logsEnabled`，如果没有设置，则继续执行。
2. 如果 `t.t.Failed()` 或 `t.logsEnabled` 为真，则执行以下操作：
a. 获取 `t` 的父日志输出 `t.parent`。
b. 如果 `t.parent` 不为空，则将 `t.buf` 中的所有事件添加到 `t.parent` 上。
c. 如果 `t.parent` 为空或者 `t.buf` 为空，则执行以下操作：
   - 如果 `t.t.Failed()` 为真，则输出所有事件并调用 `t.parent` 中的 `add()` 方法将所有事件添加到 `t.parent` 上。
   - 如果 `t.logsEnabled` 为真，则先使用 `sort.Sort(t.buf)` 对 `t.buf` 中的事件进行排序，然后打印排序后的所有事件。接着，打印链式调用 `t.parent` 中的 `add()` 方法添加的事件，并调用 `fmt.Println()` 输出事件信息。最后，输出结束后将 `t.buf` 中的所有事件清空。


```
func (t *TestLogger) flush() {
	if t.t.Failed() || t.logsEnabled {
		t.m.Lock()
		defer t.m.Unlock()
		// if this is a child, send the events to the parent
		// the root parent will print all the events in sorted order
		if t.parent != nil {
			for _, e := range t.buf {
				t.parent.add(e)
			}
		} else {
			// we're the root, sort all the events and then print them
			sort.Sort(t.buf)
			fmt.Println()
			fmt.Printf("Logs for test %q:\n\n", t.t.Name())
			for _, e := range t.buf {
				fmt.Println(e.msg)
			}
			fmt.Println()
		}
		t.buf = nil
	}
}

```

# `/opt/kubo/test/cli/harness/node.go`

该代码包是一个用于管理命令行工具的工具，名为" harness"。它主要实现了以下功能：

1. 导入了一些必要的库，包括 bytes、encoding/json、errors、fmt、io、io/fs、net/http、os、os/exec、path/filepath、runtime、strconv、strings、syscall、time、以及 logging、config、serialize 等库。

2. 实现了一个 HTTP 命令行工具的接口，可以执行下载、上传、查询等操作。

3. 通过封装实现了 libp2p 库的 peer 组件，使得 libp2p 库能够更方便地在其他 libp2p 库中使用。

4. 实现了文件系统的操作，包括创建、打开、关闭、读取、写入、删除等操作。

5. 通过封装实现了 HTTP 请求的封装，包括设置请求头、请求参数、发送请求等操作。

6. 通过封装实现了命令行工具的运行时间，包括设置工具名、版本、选项、执行命令等操作。

7. 通过封装实现了 logging 包的输出，可以将日志信息输出到指定的目标，例如输出到屏幕、输出到文件、输出到网络等。

8. 通过封装实现了 config 包的序列化和反序列化，可以将配置信息序列化为 JSON 字符串，也可以将 JSON 字符串反序列化为配置信息。

9. 通过封装实现了 serialize 包的序列化和反序列化，可以将 JSON 数据序列化为其他数据类型，也可以将其他数据类型反序列化为 JSON 数据。

10. 通过封装实现了 time 包的获取当前时间，可以获取当前时间的年、月、日、小时、分钟和秒。

11. 通过封装实现了 libp2p 库的 peers 管理器，可以管理libp2p的peer组件，实现注册、发现、连接、断开等操作。

12. 通过封装实现了 HTTP命令行工具的内部处理，包括解析命令行参数、执行下载、上传、查询等操作。


```
package harness

import (
	"bytes"
	"encoding/json"
	"errors"
	"fmt"
	"io"
	"io/fs"
	"net/http"
	"os"
	"os/exec"
	"path/filepath"
	"runtime"
	"strconv"
	"strings"
	"syscall"
	"time"

	logging "github.com/ipfs/go-log/v2"
	"github.com/ipfs/kubo/config"
	serial "github.com/ipfs/kubo/config/serialize"
	"github.com/libp2p/go-libp2p/core/peer"
	rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager"
	"github.com/multiformats/go-multiaddr"
	manet "github.com/multiformats/go-multiaddr/net"
)

```

这段代码创建了一个名为 "testharness" 的日志输出器，并将其注册到该输出器中。这个输出器将记录 Node 结构的每个字段，包括 ID、目录和一些系统信息。此外，还记录了每个 Node 实例的配置和状态，例如 API、Gateway 和 Swarm 服务器的地址，以及是否启用了 MDNS 注册。最后，创建了一个 "Runner" 类型的实例，用于运行在 Node 上的测试。


```
var log = logging.Logger("testharness")

// Node is a single Kubo node.
// Each node has its own config and can run its own Kubo daemon.
type Node struct {
	ID  int
	Dir string

	APIListenAddr     multiaddr.Multiaddr
	GatewayListenAddr multiaddr.Multiaddr
	SwarmAddr         multiaddr.Multiaddr
	EnableMDNS        bool

	IPFSBin string
	Runner  *Runner

	Daemon *RunResult
}

```

该代码定义了一个名为 "BuildNode" 的函数，接受三个参数：ipfsBin、baseDir 和 id。函数的作用是在指定的 baseDir 目录下创建一个名为 "id" 的目录，并在其中创建一个名为 "ipfsBin" 的文件夹。

函数首先检查 os.MkdirAll 函数是否出错，如果出错则输出错误并退出。然后将 IPFS_PATH 环境变量设置为生成的目录，通过 os.Environ 函数获取。

最后，函数创建了一个名为 "Runner" 的匿名函数作为参数，并传递给该匿名函数的 Env 和 Dir 字段。通过传递的 runner.Runner 构造了一个 Runner 对象，该对象实现了 os.Run 函数。通过 Runner.Start 方法启动 runner，通过 runner.Once 方法获取 runner 对象 once，然后函数返回一个指向 Node 对象的引用。


```
func BuildNode(ipfsBin, baseDir string, id int) *Node {
	dir := filepath.Join(baseDir, strconv.Itoa(id))
	if err := os.MkdirAll(dir, 0o755); err != nil {
		panic(err)
	}

	env := environToMap(os.Environ())
	env["IPFS_PATH"] = dir

	return &Node{
		ID:      id,
		Dir:     dir,
		IPFSBin: ipfsBin,
		Runner: &Runner{
			Env: env,
			Dir: dir,
		},
	}
}

```

这两函数的作用如下：

1. `func (n *Node) WriteBytes(filename string, b []byte)` 函数的作用是向文件 `filename` 写入字节数组 `b`。它使用 `os.Create` 函数创建一个新文件，并使用 `io.Copy` 函数将字节数组 `b` 写入到文件中。如果创建或写入文件时出现错误，函数会输出相应的错误信息并终止执行。

2. `func (n *Node) ReadFile(filename string)` 函数的作用是从文件 `filename` 中读取字节数组。如果文件名是相对路径，它会从节点根目录开始查找，而不是从当前工作目录开始。函数使用 `os.ReadFile` 函数读取文件中的字节并返回它，然后使用 `string` 函数将字节转换为字符串。如果读取文件时出现错误，函数会输出相应的错误信息并终止执行。

这两个函数都在尝试从文件系统中读取或写入文件。为了确保文件操作的可靠性，它们使用了错误处理和异常处理。如果出现错误，函数会输出相应的错误信息并终止执行，以帮助开发人员更好地理解问题。


```
func (n *Node) WriteBytes(filename string, b []byte) {
	f, err := os.Create(filepath.Join(n.Dir, filename))
	if err != nil {
		panic(err)
	}
	defer f.Close()
	_, err = io.Copy(f, bytes.NewReader(b))
	if err != nil {
		panic(err)
	}
}

// ReadFile reads the specific file. If it is relative, it is relative the node's root dir.
func (n *Node) ReadFile(filename string) string {
	f := filename
	if !filepath.IsAbs(filename) {
		f = filepath.Join(n.Dir, filename)
	}
	b, err := os.ReadFile(f)
	if err != nil {
		panic(err)
	}
	return string(b)
}

```

这段代码定义了三个函数，分别接收一个整型指针变量 `n`，并返回一个字符串类型的文件路径，表示当前目录下的一个名为 "config" 的文件。

第一个函数 `ConfigFile` 将 `n` 传递给一个名为 `filepath.Join` 的函数，该函数将两个参数 `n` 和路径连接起来，返回一个字符串类型的文件路径。这个文件路径是相对于当前目录（也就是 `n` 的 `Dir`）的，因此，如果 `n` 的 `Dir` 目录不存在，函数将返回一个空字符串。

第二个函数 `ReadConfig` 接收一个整型指针变量 `n`，并返回一个指向 `config.Config` 类型对象的引用。这个函数首先尝试读取一个名为 "config" 的文件，如果文件不存在或者读取过程中出现错误，函数将引发一个异常并返回一个空指针。如果文件读取成功，函数返回文件中的配置信息。

第三个函数 `WriteConfig` 接收一个整型指针变量 `c` 和一个字符串路径 `path`，并返回一个整型指针变量，表示文件是否成功写入。这个函数将 `c` 的值作为参数，并使用 `filepath.Join` 函数将文件路径和文件名连接起来，然后使用 `serial.WriteConfigFile` 函数将配置信息写入到文件中。如果写入过程中出现错误，函数将引发一个异常并返回一个空指针。如果写入成功，函数返回 `true`。


```
func (n *Node) ConfigFile() string {
	return filepath.Join(n.Dir, "config")
}

func (n *Node) ReadConfig() *config.Config {
	cfg, err := serial.Load(filepath.Join(n.Dir, "config"))
	if err != nil {
		panic(err)
	}
	return cfg
}

func (n *Node) WriteConfig(c *config.Config) {
	err := serial.WriteConfigFile(filepath.Join(n.Dir, "config"), c)
	if err != nil {
		panic(err)
	}
}

```

这两段代码都是指向一个名为Node的 struct类型的函数，函数接收两个参数，一个是n *Node类型的指针，另一个是一个f函数，它是函数指针类型。

第一段代码定义了一个名为func的函数，函数接收一个n *Node类型的指针和一个f函数作为参数，函数的作用是读取并更新n的配置文件，然后将更新后的配置文件写回n中。具体来说，函数首先读取n的配置文件，然后调用f函数中的第一个参数，将读取到的配置文件作为参数传入，接着函数调用f函数中的第二个参数，将更新后的配置文件作为参数传入，最后将更新后的配置文件写回n中。

第二段代码定义了一个名为ReadUserResourceOverrides的函数，函数接收一个rcmgr.PartialLimitConfig类型的指针，函数的作用是从用户资源限制配置文件中读取用户资源限制，并返回一个rcmgr.PartialLimitConfig类型的指针。函数首先从用户资源限制配置文件中读取相关配置，然后返回一个rcmgr.PartialLimitConfig类型的指针，表示用户资源限制的配置。


```
func (n *Node) UpdateConfig(f func(cfg *config.Config)) {
	cfg := n.ReadConfig()
	f(cfg)
	n.WriteConfig(cfg)
}

func (n *Node) ReadUserResourceOverrides() *rcmgr.PartialLimitConfig {
	var r rcmgr.PartialLimitConfig
	err := serial.ReadConfigFile(filepath.Join(n.Dir, "libp2p-resource-limit-overrides.json"), &r)
	switch err {
	case nil, serial.ErrNotInitialized:
		return &r
	default:
		panic(err)
	}
}

```

这段代码定义了三个函数，分别作用于三个不同的场景。下面分别对这三个函数进行解释：

1. `func (n *Node) WriteUserSuppliedResourceOverrides(c *rcmgr.PartialLimitConfig)`：
这个函数的作用是将用户提供的资源超过限制的配置信息写入到 `rcmgr.PartialLimitConfig` 类型的指针变量 `c` 中。具体来说，这个函数首先尝试将配置文件 `libp2p-resource-limit-overrides.json` 写入到 `n.Dir` 目录下的 `libp2p-resource-limit-overrides.json` 文件中，如果写入失败，则抛出异常并返回。然后，这个函数将用户提供的资源超过限制的配置信息传递给 `f` 函数进行处理，最后将处理后的结果传回给 `n.WriteUserSuppliedResourceOverrides` 函数。

2. `func (n *Node) UpdateUserSuppliedResourceManagerOverrides(f func(overrides *rcmgr.PartialLimitConfig))`：
这个函数的作用是更新 `Node` 实例的 `userSuppliedResourceManagerOverrides` 函数，这个函数接受一个 `rcmgr.PartialLimitConfig` 类型的参数 `f`，表示如何处理用户提供的资源超过限制的配置信息。具体来说，这个函数首先尝试从 `n.UserSuppliedResourceOverrides` 函数中读取用户提供的资源超过限制的配置信息，然后传递给 `f` 函数进行处理，最后将处理后的结果存储回 `n.UserSuppliedResourceManagerOverrides` 函数中。

3. `func (n *Node) IPFS(args ...string) *RunResult`：
这个函数的作用是执行 `IPFS` 操作并返回结果。具体来说，这个函数接受多个参数 `args`，表示 `IPFS` 操作需要使用的命令行参数，然后执行 `n.RunIPFS` 函数，并将 `args` 中的参数传递给 `n.RunIPFS` 函数进行处理，最后将处理后的结果返回给 `*RunResult` 类型的变量 `res`。


```
func (n *Node) WriteUserSuppliedResourceOverrides(c *rcmgr.PartialLimitConfig) {
	err := serial.WriteConfigFile(filepath.Join(n.Dir, "libp2p-resource-limit-overrides.json"), c)
	if err != nil {
		panic(err)
	}
}

func (n *Node) UpdateUserSuppliedResourceManagerOverrides(f func(overrides *rcmgr.PartialLimitConfig)) {
	overrides := n.ReadUserResourceOverrides()
	f(overrides)
	n.WriteUserSuppliedResourceOverrides(overrides)
}

func (n *Node) IPFS(args ...string) *RunResult {
	res := n.RunIPFS(args...)
	n.Runner.AssertNoError(res)
	return res
}

```

这是一个 Go 语言中的函数，它实现了将字符串 "PipeStrToIPFS" 传递给 IPFS 库并将结果返回给调用者。

具体来说，代码实现了一个名为 "PipeStrToIPFS" 的函数，它接收一个字符串参数 "s"，以及一个或多个参数 "args"，并将这些参数传递给 IPFS 库。然后，它从 IPFS 库的 "PipeToIPFS" 函数中返回结果，如果调用过程中出现错误，函数将返回一个 "RunResult" 类型的数据。

接下来，代码实现了一个名为 "PipeToIPFS" 的函数，它接收一个 "Reader" 类型的参数 "reader"，以及一个或多个参数 "args"，并将这些参数传递给 IPFS 库的 "RunPipeToIPFS" 函数。如果调用过程中出现错误，函数将返回一个 "RunResult" 类型的数据。

最后，代码实现了一个名为 "RunPipeToIPFS" 的函数，它接收一个 "Reader" 类型的参数 "reader"，以及一个或多个参数 "args"，并将这些参数传递给 "RunWithStdin" 函数，然后调用 IPFS 库的 "Run" 函数，并将结果返回给调用者。


```
func (n *Node) PipeStrToIPFS(s string, args ...string) *RunResult {
	return n.PipeToIPFS(strings.NewReader(s), args...)
}

func (n *Node) PipeToIPFS(reader io.Reader, args ...string) *RunResult {
	res := n.RunPipeToIPFS(reader, args...)
	n.Runner.AssertNoError(res)
	return res
}

func (n *Node) RunPipeToIPFS(reader io.Reader, args ...string) *RunResult {
	return n.Runner.Run(RunRequest{
		Path:    n.IPFSBin,
		Args:    args,
		CmdOpts: []CmdOpt{RunWithStdin(reader)},
	})
}

```

This code looks like it initializes and configures an IPFS node. The IPFS node has a specific IP address, a mechanism for connecting to it, and a mechanism for reaching it from outside the node. It also initializes a swarm of IPFS nodes and a gateway for connecting to the outside world. The node can also be configured through a `config.Config` struct, which specifies various details of how the node should behave.


```
func (n *Node) RunIPFS(args ...string) *RunResult {
	return n.Runner.Run(RunRequest{
		Path: n.IPFSBin,
		Args: args,
	})
}

// Init initializes and configures the IPFS node, after which it is ready to run.
func (n *Node) Init(ipfsArgs ...string) *Node {
	n.Runner.MustRun(RunRequest{
		Path: n.IPFSBin,
		Args: append([]string{"init"}, ipfsArgs...),
	})

	if n.SwarmAddr == nil {
		swarmAddr, err := multiaddr.NewMultiaddr("/ip4/127.0.0.1/tcp/0")
		if err != nil {
			panic(err)
		}
		n.SwarmAddr = swarmAddr
	}

	if n.APIListenAddr == nil {
		apiAddr, err := multiaddr.NewMultiaddr("/ip4/127.0.0.1/tcp/0")
		if err != nil {
			panic(err)
		}
		n.APIListenAddr = apiAddr
	}

	if n.GatewayListenAddr == nil {
		gatewayAddr, err := multiaddr.NewMultiaddr("/ip4/127.0.0.1/tcp/0")
		if err != nil {
			panic(err)
		}
		n.GatewayListenAddr = gatewayAddr
	}

	n.UpdateConfig(func(cfg *config.Config) {
		cfg.Bootstrap = []string{}
		cfg.Addresses.Swarm = []string{n.SwarmAddr.String()}
		cfg.Addresses.API = []string{n.APIListenAddr.String()}
		cfg.Addresses.Gateway = []string{n.GatewayListenAddr.String()}
		cfg.Swarm.DisableNatPortMap = true
		cfg.Discovery.MDNS.Enabled = n.EnableMDNS
	})
	return n
}

```

这段代码的作用是创建并运行一个Kubernetes（daemon）daemon，该daemon以指定的请求启动。具体来说，它通过调用`node.StartDaemonWithReq`函数来创建一个`RunRequest`对象，该对象包含运行Kubernetes daemon所需的参数、CMD选项以及运行函数。然后，它将该请求设置为给定的请求，并使用`n.Runner`字段所提供的`MustRun`方法来运行该请求。在函数内部，它使用`log.Debugf`和`log.Panicf`函数来输出调试信息。


```
// StartDaemonWithReq runs a Kubo daemon with the given request.
// This overwrites the request Path with the Kubo bin path.
//
// For example, if you want to run the daemon and see stderr and stdout to debug:
//
//	 node.StartDaemonWithReq(harness.RunRequest{
//	 	 CmdOpts: []harness.CmdOpt{
//			harness.RunWithStderr(os.Stdout),
//			harness.RunWithStdout(os.Stdout),
//		 },
//	 })
func (n *Node) StartDaemonWithReq(req RunRequest) *Node {
	alive := n.IsAlive()
	if alive {
		log.Panicf("node %d is already running", n.ID)
	}
	newReq := req
	newReq.Path = n.IPFSBin
	newReq.Args = append([]string{"daemon"}, req.Args...)
	newReq.RunFunc = (*exec.Cmd).Start

	log.Debugf("starting node %d", n.ID)
	res := n.Runner.MustRun(newReq)

	n.Daemon = res

	log.Debugf("node %d started, checking API", n.ID)
	n.WaitOnAPI()
	return n
}

```

这两段代码定义了两个函数，分别是 `func` 和 `func`。这两个函数都是星号 `func` 函数，它们接收一个 `n` 类型的指针参数，并返回一个 `*Node` 类型的指针。

第一个函数是 `StartDaemon` 函数，它的接收者是一个 `args` 类型的结构体，包含了一些参数，用于启动一个分布式时序网络（DAO）的节点。这个函数的作用是在给定的参数中启动一个DAO节点并返回这个节点，以便用户使用StartDaemon函数来启动自己的DAO节点。

第二个函数是 `signalAndWait` 函数，它接收一个 `watch` 类型的结构体，包含一个监视器通道，用于等待给定的信号。它还接收一个 `signal` 类型的结构体，用于发送给定信号。最后，它接收一个 `timeout` 类型的结构体，用于设置超时时间。

这两个函数的具体实现没有给出，但从函数的名称来看，`StartDaemon` 函数用于启动一个DAO节点并返回，而 `signalAndWait` 函数用于等待给定信号并超时。


```
func (n *Node) StartDaemon(ipfsArgs ...string) *Node {
	return n.StartDaemonWithReq(RunRequest{
		Args: ipfsArgs,
	})
}

func (n *Node) signalAndWait(watch <-chan struct{}, signal os.Signal, t time.Duration) bool {
	err := n.Daemon.Cmd.Process.Signal(signal)
	if err != nil {
		if errors.Is(err, os.ErrProcessDone) {
			log.Debugf("process for node %d has already finished", n.ID)
			return true
		}
		log.Panicf("error killing daemon for node %d with peer ID %s: %s", n.ID, n.PeerID(), err.Error())
	}
	timer := time.NewTimer(t)
	defer timer.Stop()
	select {
	case <-watch:
		return true
	case <-timer.C:
		return false
	}
}

```

This is a Go program that stops a Node.js server. The program takes a peer ID as an argument and uses the `stop-node` command, which is a custom command that stops the Node.js server.

The program first checks if the Node.js server has a daemon running. If it does not, it logs a message and returns the Node.js server.

If the Node.js server has a daemon running, the program uses the `signalAndWait` function to stop the server by SIGTERM or SIGKILL. The `signalAndWait` function sends a signal to the specified channel and waits for a signal. The `watch` channel receives signals from the Node.js server and returns a signal after 5 seconds if the server stops.

If the server stops, the program logs a message and returns the Node.js server. If the server does not stop or if it takes longer than 5 seconds, the program panics and returns.


```
func (n *Node) StopDaemon() *Node {
	log.Debugf("stopping node %d", n.ID)
	if n.Daemon == nil {
		log.Debugf("didn't stop node %d since no daemon present", n.ID)
		return n
	}
	watch := make(chan struct{}, 1)
	go func() {
		_, _ = n.Daemon.Cmd.Process.Wait()
		watch <- struct{}{}
	}()

	// os.Interrupt does not support interrupts on Windows https://github.com/golang/go/issues/46345
	if runtime.GOOS == "windows" {
		if n.signalAndWait(watch, syscall.SIGKILL, 5*time.Second) {
			return n
		}
		log.Panicf("timed out stopping node %d with peer ID %s", n.ID, n.PeerID())
	}

	log.Debugf("signaling node %d with SIGTERM", n.ID)
	if n.signalAndWait(watch, syscall.SIGTERM, 1*time.Second) {
		return n
	}
	log.Debugf("signaling node %d with SIGTERM", n.ID)
	if n.signalAndWait(watch, syscall.SIGTERM, 2*time.Second) {
		return n
	}
	log.Debugf("signaling node %d with SIGQUIT", n.ID)
	if n.signalAndWait(watch, syscall.SIGQUIT, 5*time.Second) {
		return n
	}
	log.Debugf("signaling node %d with SIGKILL", n.ID)
	if n.signalAndWait(watch, syscall.SIGKILL, 5*time.Second) {
		return n
	}
	log.Panicf("timed out stopping node %d with peer ID %s", n.ID, n.PeerID())
	return n
}

```

这段代码定义了两个函数，分别接收一个名为 `n` 的 `Node` 类型的参数，并返回一个 `multiaddr.Multiaddr` 类型的值。

第一个函数名为 `func (n *Node) APIAddr() multiaddr.Multiaddr`，函数接收一个 `*Node` 类型的参数 `n`，并尝试通过调用 `n.TryAPIAddr()` 函数获取到 `n` 的 `APIAddr()` 值。如果这个尝试失败，函数将会崩溃并输出一个错误。

第二个函数名为 `func (n *Node) APIURL() string`，函数接收一个 `*Node` 类型的参数 `n`，并尝试通过调用 `n.APIAddr()` 函数获取到 `n` 的 `APIAddr()` 值。如果这个尝试失败，函数将会崩溃并输出一个错误。然后，函数使用 `manet.ToNetAddr()` 函数将 `n.APIAddr()` 返回的值转换为一个 `netaddr.NetAddress` 类型，并返回一个字符串，该字符串表示 API 服务的 URL。


```
func (n *Node) APIAddr() multiaddr.Multiaddr {
	ma, err := n.TryAPIAddr()
	if err != nil {
		panic(err)
	}
	return ma
}

func (n *Node) APIURL() string {
	apiAddr := n.APIAddr()
	netAddr, err := manet.ToNetAddr(apiAddr)
	if err != nil {
		panic(err)
	}
	return "http://" + netAddr.String()
}

```

This function appears to be part of a networked NodeP风华实，用于向聚合栈中注册自己，并检查聚合栈中的PeerID是否与该注册的PeerID相同。

具体来说，它首先尝试向预设的API地址注册自己，如果注册失败，则向该地址发送请求并检查返回的响应结果。如果注册成功，则向该地址发送请求并检查返回的响应结果，以确定是否包含一个PeerID。如果返回的响应结果包含一个PeerID，则将其存储在函数的响应中，并返回True表示注册成功。

函数使用了`log`函数来输出一些调试信息，如果注册失败，则会输出错误信息。


```
func (n *Node) TryAPIAddr() (multiaddr.Multiaddr, error) {
	b, err := os.ReadFile(filepath.Join(n.Dir, "api"))
	if err != nil {
		return nil, err
	}
	ma, err := multiaddr.NewMultiaddr(string(b))
	if err != nil {
		return nil, err
	}
	return ma, nil
}

func (n *Node) checkAPI() bool {
	apiAddr, err := n.TryAPIAddr()
	if err != nil {
		log.Debugf("node %d API addr not available yet: %s", n.ID, err.Error())
		return false
	}
	ip, err := apiAddr.ValueForProtocol(multiaddr.P_IP4)
	if err != nil {
		panic(err)
	}
	port, err := apiAddr.ValueForProtocol(multiaddr.P_TCP)
	if err != nil {
		panic(err)
	}
	url := fmt.Sprintf("http://%s:%s/api/v0/id", ip, port)
	log.Debugf("checking API for node %d at %s", n.ID, url)
	httpResp, err := http.Post(url, "", nil)
	if err != nil {
		log.Debugf("node %d API check error: %s", err.Error())
		return false
	}
	defer httpResp.Body.Close()
	resp := struct {
		ID string
	}{}

	respBytes, err := io.ReadAll(httpResp.Body)
	if err != nil {
		log.Debugf("error reading API check response for node %d: %s", n.ID, err.Error())
		return false
	}
	log.Debugf("got API check response for node %d: %s", n.ID, string(respBytes))

	err = json.Unmarshal(respBytes, &resp)
	if err != nil {
		log.Debugf("error decoding API check response for node %d: %s", n.ID, err.Error())
		return false
	}
	if resp.ID == "" {
		log.Debugf("API check response for node %d did not contain a Peer ID", n.ID)
		return false
	}
	respPeerID, err := peer.Decode(resp.ID)
	if err != nil {
		panic(err)
	}

	peerID := n.PeerID()
	if respPeerID != peerID {
		log.Panicf("expected peer ID %s but got %s", peerID, resp.ID)
	}

	log.Debugf("API check for node %d successful", n.ID)
	return true
}

```

这段代码定义了两个函数，分别是 `func (n *Node) PeerID() peer.ID` 和 `func (n *Node) WaitOnAPI() *Node`。

1. `func (n *Node) PeerID() peer.ID` 函数接收一个 `Node` 类型的参数 `n`，并尝试从 `n` 的配置中读取出与当前节点 `n` 相关的 `PeerID`。如果没有找到，则会尝试使用 `peer.Decode` 函数将 `PeerID` 编码为字符串，并返回结果。函数的作用是获取与当前节点相关的 `PeerID`。

2. `func (n *Node) WaitOnAPI() *Node` 函数接收一个 `Node` 类型的参数 `n`，并使用一个循环来等待服务器响应。每次循环检查 `n` 是否已经连接到服务器。如果 `n` 已经连接到服务器，则返回 `n`。如果 `n` 没有连接到服务器，则会尝试使用 `peer.Decode` 函数将 `PeerID` 编码为字符串，并尝试使用 `io.ioutil.Subsystem` 函数来读取服务器返回的 `stdout` 和 `stderr` 内容。如果 `n` 正确地连接到服务器，则返回 `n`。如果 `n` 无法连接到服务器，则会输出一条错误信息并返回 `n`。


```
func (n *Node) PeerID() peer.ID {
	cfg := n.ReadConfig()
	id, err := peer.Decode(cfg.Identity.PeerID)
	if err != nil {
		panic(err)
	}
	return id
}

func (n *Node) WaitOnAPI() *Node {
	log.Debugf("waiting on API for node %d", n.ID)
	for i := 0; i < 50; i++ {
		if n.checkAPI() {
			log.Debugf("daemon API found, daemon stdout: %s", n.Daemon.Stdout.String())
			return n
		}
		time.Sleep(400 * time.Millisecond)
	}
	log.Panicf("node %d with peer ID %s failed to come online: \n%s\n\n%s", n.ID, n.PeerID(), n.Daemon.Stderr.String(), n.Daemon.Stdout.String())
	return n
}

```

这两段代码分别定义了两个函数：

1. `func (n *Node) IsAlive() bool`：该函数接收一个 `Node` 类型的参数 `n`，并检查 `n` 的 `Daemon` 字段是否为 `nil`，以及 `n.Daemon.Cmd` 和 `n.Daemon.Cmd.Process` 字段是否为 `nil`。如果是，函数返回 `false`；否则，函数执行以下操作：
	* 调用 `n.Daemon.Cmd.Process.Signal`，并检查调用是否成功。如果调用成功，函数输出 `node` `ID` 并返回 `true`；否则，函数输出 `node` `ID` 并返回 `false`。

2. `func (n *Node) SwarmAddrs() []multiaddr.Multiaddr`：该函数接收一个 `Node` 类型的参数 `n`，并执行以下操作：
	* 调用 `n.Runner.MustRun`，并传递一个 `RunRequest` 参数，其中包含以下参数：
		+ `Path`：节点存储器所在路径，即 `n.IPFSBin`。
		+ `Args`：用于指定 `swarm` 子命令需要传递给 `swarm` 子系统的参数，这里是 `local`。
		+ `swarm` 子命令用于在本地网络中传播节点信息，以便其他节点发现并连接它们。
		+ `Node`：被要求传播的节点。
		+ `Cmd`：需要传播给 `swarm` 子系统的命令，这里是 `n.Daemon.Cmd`。
		+ `Process`：需要传播给 `swarm` 子系统的节点进程，这里是 `n.Daemon.Cmd.Process`。
		+ `Signal`：发送 `SIG_WAKEup` 信号给指定的进程，用于唤醒节点。
		+ `Cmd`：用于发送 `SIG_WAKEup` 信号给指定的进程的 `cmd` 字段。
		+ `Process.Signal`：发送 `SIG_WAKEup` 信号给指定的进程。
		+ `node.ID`：需要传播给 `swarm` 子系统的节点 ID。
		+ `stdout`： `swarm` 子系统返回的 `stdout` 字段，其中包含节点信息。
		+ `stderr`： `swarm` 子系统返回的 `stderr` 字段，其中包含错误信息。
		+ `os.Stderr.ReadAll`：读取 `stdout` 和 `stderr` 字段的并行内容。
		+ `strconv`：对 `os.Stderr.ReadAll` 返回的字符串进行字符串转换。
		+ `sprint`：打印 `strconv.Sprint` 的值。
		+ `fSafeCmd`：用于执行 `fork` 和 `exec` 并发的 `cmd` 字段。
		+ `os.Exit`：执行 `os.Exit` 并返回 0，以便 `swarm` 子系统退出。
		+ `errChan`：用于接收从 `swarm` 子系统传递的错误信息。
		+ `signalChan`：用于接收 `Node` 发送的 `SIG_WAKEup` 信号。
		+ `os.Signal`：用于操作系统信号，由 `errChan` 和 `signalChan` 返回。
		+ `syscall.Signal`：用于操作系统信号，由 `errChan` 和 `signalChan` 返回。
		+ `signal(<0)`：向指定的 <0> 信号发送，并返回。


```
func (n *Node) IsAlive() bool {
	if n.Daemon == nil || n.Daemon.Cmd == nil || n.Daemon.Cmd.Process == nil {
		return false
	}
	log.Debugf("signaling node %d daemon process for liveness check", n.ID)
	err := n.Daemon.Cmd.Process.Signal(syscall.Signal(0))
	if err == nil {
		log.Debugf("node %d daemon is alive", n.ID)
		return true
	}
	log.Debugf("node %d daemon not alive: %s", err.Error())
	return false
}

func (n *Node) SwarmAddrs() []multiaddr.Multiaddr {
	res := n.Runner.MustRun(RunRequest{
		Path: n.IPFSBin,
		Args: []string{"swarm", "addrs", "local"},
	})
	out := strings.TrimSpace(res.Stdout.String())
	outLines := strings.Split(out, "\n")
	var addrs []multiaddr.Multiaddr
	for _, addrStr := range outLines {
		ma, err := multiaddr.NewMultiaddr(addrStr)
		if err != nil {
			panic(err)
		}
		addrs = append(addrs, ma)
	}
	return addrs
}

```

该函数的作用是获取一个节点`n`的`SwarmAddrs()`方法返回的地址列表，并将其与该节点的`PeerID`一起存储在`multiaddr.Multiaddr`类型的数组中。

具体来说，该函数首先定义了一个名为`SwarmAddrsWithPeerIDs()`的函数，它接收一个`n`指针和`multiaddr.ProtocolWithCode(multiaddr.P_IPFS).Name`参数。接着，它定义了一个名为`ipfsProtocol`的变量，并将其设置为`multiaddr.ProtocolWithCode(multiaddr.P_IPFS)`的返回值。接下来，它定义了一个名为`peerID`的变量，并将其设置为`n`的`PeerID()`方法的返回值。

函数内部，该函数使用一个循环遍历`n`的`SwarmAddrs()`方法返回的地址列表，对于每个地址，函数首先尝试将其`PeerID()`字段存储在`multiaddr.Multiaddr`类型的变量中。如果该尝试失败（可能是由于该地址不支持`multiaddr.P_IPFS`协议）且错误为`multiaddr.ErrProtocolNotFound`，函数将捕获并处理错误。否则，函数将该地址转换为`multiaddr.Components()`函数的`multiaddr.Multiaddr`类型，并将其添加到结果数组中。

最后，函数返回结果数组。


```
func (n *Node) SwarmAddrsWithPeerIDs() []multiaddr.Multiaddr {
	ipfsProtocol := multiaddr.ProtocolWithCode(multiaddr.P_IPFS).Name
	peerID := n.PeerID()
	var addrs []multiaddr.Multiaddr
	for _, ma := range n.SwarmAddrs() {
		// add the peer ID to the multiaddr if it doesn't have it
		_, err := ma.ValueForProtocol(multiaddr.P_IPFS)
		if errors.Is(err, multiaddr.ErrProtocolNotFound) {
			comp, err := multiaddr.NewComponent(ipfsProtocol, peerID.String())
			if err != nil {
				panic(err)
			}
			ma = ma.Encapsulate(comp)
		}
		addrs = append(addrs, ma)
	}
	return addrs
}

```

该函数接收一个整型参数 `n` 表示一个 `Node` 类型的实例，并返回一个 `[]multiaddr.Multiaddr` 类型的数组，其中包含添加到 `swarmAddrs()` 函数中的地址。

该函数遍历 `n.SwarmAddrs()` 返回的列表，对于每个地址 `ma`，通过循环遍历地址的组件并检查其是否为 IPFS 协议的组件，如果是，则返回 `true`。否则，将组件添加到 `components` 数组中，并将 `ma` 添加到 `addrs` 数组中。

最后，函数将 `addrs` 数组中的地址连接成一个 `multiaddr.Multiaddr` 类型的实例并返回。


```
func (n *Node) SwarmAddrsWithoutPeerIDs() []multiaddr.Multiaddr {
	var addrs []multiaddr.Multiaddr
	for _, ma := range n.SwarmAddrs() {
		var components []multiaddr.Multiaddr
		multiaddr.ForEach(ma, func(c multiaddr.Component) bool {
			if c.Protocol().Code == multiaddr.P_IPFS {
				return true
			}
			components = append(components, &c)
			return true
		})
		ma = multiaddr.Join(components...)
		addrs = append(addrs, ma)
	}
	return addrs
}

```

这两函数的作用是实现一个网络中的节点通信和获取节点列表的功能。

第一个函数 `Connect` 接收两个参数：一个 `Node` 类型的变量 `n` 和一个 `Node` 类型的变量 `other`。函数的作用是将 `other` 节点的地址存储在一个 `Node` 类型的变量 `n` 的 `Runner` 字段中，然后运行一个 `RunRequest` 类型，其中 `Path` 字段指定 `n` 的 IPFSBin 文件，`Args` 字段是一个字符串数组，包含一个指向其他节点地址的列表。

第二个函数 `Peers` 同样接收两个参数：一个 `Node` 类型的变量 `n` 和一个 `multiaddr.Multiaddr` 类型的变量 `res`。函数的作用是运行一个 `RunRequest` 类型，其中 `Path` 字段指定 `n` 的 IPFSBin 文件，`Args` 字段包含一个字符串数组，包含一个名为 "peers" 的参数。然后从 `res.Stdout.Lines()` 方法中读取所有内容，并将其转换为 `multiaddr.Multiaddr` 类型，然后将其添加到 `addrs` 数组中。最后，函数返回 `addrs` 数组。


```
func (n *Node) Connect(other *Node) *Node {
	n.Runner.MustRun(RunRequest{
		Path: n.IPFSBin,
		Args: []string{"swarm", "connect", other.SwarmAddrsWithPeerIDs()[0].String()},
	})
	return n
}

func (n *Node) Peers() []multiaddr.Multiaddr {
	res := n.Runner.MustRun(RunRequest{
		Path: n.IPFSBin,
		Args: []string{"swarm", "peers"},
	})
	var addrs []multiaddr.Multiaddr
	for _, line := range res.Stdout.Lines() {
		ma, err := multiaddr.NewMultiaddr(line)
		if err != nil {
			panic(err)
		}
		addrs = append(addrs, ma)
	}
	return addrs
}

```

该函数接收两个`Node`类型的参数，分别为`n`和`other`。

首先，函数内部创建一个`UpdateConfig`函数，该函数接收一个`cfg`参数，并执行以下操作：

1. 创建一个空字符串切片，名为`addrs`。
2. 遍历`other`的`ReadConfig().Addresses.Swarm`方法返回的地址字符串。
3. 创建一个`multiaddr.Multiaddr`对象，并将其添加到`addrs`切片中。如果创建过程中出现错误，函数将输出错误并终止执行。
4. 将`cfg.Peering.Peers`字段添加到`cfg.Peering.Peers`字段的值中。

函数输出的结果为：`cfg.Peering.Peers`字段中的一个`peer.AddrInfo`对象，包含以下字段：

ID：`other.PeerID()`函数返回的ID。
Addrs：`cfg.Peering.Peers`字段中`addrs`切片包含的所有`multiaddr.Multiaddr`对象的值。

函数的作用是允许两个`Node`之间建立连接，并允许连接的节点将自己的地址添加到连接中。


```
func (n *Node) PeerWith(other *Node) {
	n.UpdateConfig(func(cfg *config.Config) {
		var addrs []multiaddr.Multiaddr
		for _, addrStr := range other.ReadConfig().Addresses.Swarm {
			ma, err := multiaddr.NewMultiaddr(addrStr)
			if err != nil {
				panic(err)
			}
			addrs = append(addrs, ma)
		}

		cfg.Peering.Peers = append(cfg.Peering.Peers, peer.AddrInfo{
			ID:    other.PeerID(),
			Addrs: addrs,
		})
	})
}

```

这两段代码是Go语言中的函数，分别作用如下：

1. disconnect函数：
* n：指受影响的服务节点（任一个接入网络的节点）
* other：受到 disconnect 函数调用影响的节点（调用方为该函数的节点）

这个函数的作用是让被其他节点连接的节点与其它节点断开连接，并将其状态设置为"disconnected"。

1. GatewayURL函数：
* n：指当前节点

这个函数的作用是获取当前节点的GatewayURL，如果当前节点不存在Gateway，函数会尝试从其父节点或根节点获取Gateway。如果从父节点或根节点获取成功，函数返回当前节点的GatewayURL；如果从父节点或根节点获取失败，函数会超时并输出"timeout waiting for gateway file"。


```
func (n *Node) Disconnect(other *Node) {
	n.IPFS("swarm", "disconnect", "/p2p/"+other.PeerID().String())
}

// GatewayURL waits for the gateway file and then returns its contents or times out.
func (n *Node) GatewayURL() string {
	timer := time.NewTimer(1 * time.Second)
	defer timer.Stop()
	for {
		select {
		case <-timer.C:
			panic("timeout waiting for gateway file")
		default:
			b, err := os.ReadFile(filepath.Join(n.Dir, "gateway"))
			if err == nil {
				return strings.TrimSpace(string(b))
			}
			if !errors.Is(err, fs.ErrNotExist) {
				panic(err)
			}
			time.Sleep(1 * time.Millisecond)
		}
	}
}

```

这两行代码定义了两个函数，分别名为`func (n *Node) GatewayClient() *HTTPClient`和`func (n *Node) APIClient() *HTTPClient`。这两个函数都接受一个参数`n`，并返回一个指向`HTTPClient`类型的`*HTTPClient`变量。

这两行代码实现了一个简单的HTTP客户端，根据传入的`n`值来设置客户端的URL。在函数内部，客户端的URL设置为`n.GatewayURL()`和`n.APIURL()`，分别从`n`的`Gateway`和`API`字段中获取对应的URL。

总的来说，这两行代码定义了两个HTTP客户端函数，用于在Node对象中创建HTTP客户端，可以方便地使用默认的HTTP客户端。


```
func (n *Node) GatewayClient() *HTTPClient {
	return &HTTPClient{
		Client:  http.DefaultClient,
		BaseURL: n.GatewayURL(),
	}
}

func (n *Node) APIClient() *HTTPClient {
	return &HTTPClient{
		Client:  http.DefaultClient,
		BaseURL: n.APIURL(),
	}
}

```