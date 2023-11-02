# go-ipfs 源码解析 26

# `/opt/kubo/core/corehttp/commands.go`

这段代码是一个 Go 语言package 中的代码，它实现了通过 IPFS(InterPlanetary File System) 下载文件并执行一些操作的功能。下面是简要解释：

1. `import (`：导入了一个名为 "errors" 的错误类型和一个名为 "fmt" 的字符串格式化类型。

2. `import (net "net.http" net/http "os" strconv.Istrings" strings"`：导入了一系列来自 net、http 和 os 包的函数和类型。

3. `import ipms "github.com/ipfs/go-ipfs-cmds" ipmshttp "github.com/ipfs/go-ipfs-cmds/http"`：导入了一个名为 ipms 和 ipmshttp 的函数类型，它们都是从 ipfs-cmds 包中导入的。

4. `import version "github.com/ipfs/kubo"`：导入了一个名为 version 的函数类型，它来自 ipfs-kubo 包。

5. `import oldcmds "github.com/ipfs/kubo/commands"`：导入了一个名为 oldcmds 的函数类型，它来自 ipfs-kubo 包。

6. `import commands "github.com/ipfs/kubo/core/commands"`：导入了一个名为 commands 的函数类型，它来自 ipfs-kubo 和 core 包。

7. `import ipcmds "github.com/ipfs/go-ipfs-cmds"`：导入了一个名为 ipcmds 的函数类型，它来自 ipfs-go 包。

8. `import othellHTTP "github.com/ipfs/go-ipfs-cmds/http"`：导入了一个名为 othellHTTP 的函数类型，它来自 ipfs-go 包。

9. `ipms.SetNetHTTP "ipmshttp"`：设置 ipms 包的 HTTP 函数为 ipmshttp 包。

10. `ipms.SetIPFS "ipfs"`：设置 ipms 包的文件系统为 ipfs 包。

11. `ipms.SetVerbosity "quiet"`：设置 ipms 包的输出策略为 "quiet"。

12. `ipms.SetCommands "ipfs-quickstart"`：设置 ipms 包的命令为 "ipfs-quickstart"。

13. `ipms.SetPromptPrefix "ipfs-// "`：设置 ipms 包的命令提示的前缀为 "ipfs-// "。

14. `ipms.SetTrustK静 "user"`：设置 ipms 包的信任级别为 "user"。

15. `ipms.SetTrustW静 "core"`：设置 ipms 包的信任级别为 "core"。

16. `ipms.SetBlockExit "112"`：设置 ipms 包的块退出为 112。

17. `ipms.SetHashControl "mean"`：设置 ipms 包的哈希控制为 "mean"。

18. `ipms.SetNanos "0"`：设置 ipms 包的纳米秒为 0。

19. `ipms.SetChecksum "none"`：设置 ipms 包的校验和为 "none"。

20. `ipms.SetCipherSuite "ES256"`：设置 ipms 包的加密套件为 "ES256"。

21. `ipms.SetHash "sha256"`：设置 ipms 包的哈希算法为 "sha256 "。

22. `ipms.SetDaemon "0"`：设置 ipms 包的后台为 0。

23. `ipms.SetTrustedRecipient "file"`：设置 ipms 包的信任客户端为 file。

24. `ipms.SetTrustedPeer "ipfs"`：设置 ipms 包的信任端点为 ipfs。

25. `ipms.SetTrustedttl "1h"`：设置 ipms 包的信任超时为 1 小时。

26. `ipms.SetCommands "ipfs-rescan"`：设置 ipms 包的命令为 "ipfs-rescan"。

27. `ipms.SetCommands "ipfs-setup-multi"`：设置 ipms 包的命令为 "ipfs-setup-multi"。

28. `ipms.SetCommands "ipfs-setup-ZERO"`：设置 ipms 包的命令为 "ipfs-setup-ZERO"。


```
package corehttp

import (
	"errors"
	"fmt"
	"net"
	"net/http"
	"os"
	"strconv"
	"strings"

	cmds "github.com/ipfs/go-ipfs-cmds"
	cmdsHttp "github.com/ipfs/go-ipfs-cmds/http"
	version "github.com/ipfs/kubo"
	oldcmds "github.com/ipfs/kubo/commands"
	config "github.com/ipfs/kubo/config"
	"github.com/ipfs/kubo/core"
	corecommands "github.com/ipfs/kubo/core/commands"
	"go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp"
)

```

这段代码的作用是创建一个名为 "errAPIVersionMismatch" 的错误对象，该对象使用 "errors.New()" 方法从 errors 包中创建一个新错误并设置其类型为 "api version mismatch"。

该错误对象中包含一个名为 "errAPIVersionMismatch" 的错误信息，该信息类似于一个自定义的错误消息，但会在错误发生时被保存在当前作用域的 "err" 字段中。

该代码还定义了一个名为 "originEnvKeyDeprecate" 的常量，它会提示在使用 "originEnvKey" 时，此功能已被弃用，因为在未来的版本中不再支持它。

最后，该代码还提供了一些示例用法，以通过配置文件或命令行参数使用 "ipfs config API.HTTPHeaders" 函数将请求头设置为 JSON 字符串形式，其中请求头中的 "Access-Control-Allow-Origin" 键设置为 "*"。


```
var errAPIVersionMismatch = errors.New("api version mismatch")

const (
	originEnvKey          = "API_ORIGIN"
	originEnvKeyDeprecate = `You are using the ` + originEnvKey + `ENV Variable.
This functionality is deprecated, and will be removed in future versions.
Instead, try either adding headers to the config, or passing them via
cli arguments:

	ipfs config API.HTTPHeaders --json '{"Access-Control-Allow-Origin": ["*"]}'
	ipfs daemon
`
)

// APIPath is the path at which the API is mounted.
```

这段代码定义了两个变量 `defaultLocalhostOrigins` 和 `companionBrowserExtensionOrigins`，并分别存储了两个列表，前者包含了一些默认的 Localhost 起源，后者包含了两个 extensions 的浏览器扩展程序 origin。这些起源都是通过 `<port>` 参数指定端口号。

defaultLocalhostOrigins 的作用是提供一个默认的 Localhost 网络接口，方便在代码中进行相关操作。例如，在与其他服务进行通信时，可能会使用 Localhost 作为中转服务器，而这段代码中的 `defaultLocalhostOrigins` 就是用来定义这些中转服务器上的地址。

companionBrowserExtensionOrigins 中的内容则是两个已知的浏览器扩展程序的 origin，这些扩展程序据说是通过 IPFS (InterPlanetary File System) 技术进行分布式存储的。这里的 IPFS-Companion 和 ipfs-companion-beta 分别是两个已知的 IPFS 扩展程序。


```
const APIPath = "/api/v0"

var defaultLocalhostOrigins = []string{
	"http://127.0.0.1:<port>",
	"https://127.0.0.1:<port>",
	"http://[::1]:<port>",
	"https://[::1]:<port>",
	"http://localhost:<port>",
	"https://localhost:<port>",
}

var companionBrowserExtensionOrigins = []string{
	"chrome-extension://nibjojkomfdiaoajekhjakgkdhaomnch", // ipfs-companion
	"chrome-extension://hjoieblefckbooibpepigmacodalfndh", // ipfs-companion-beta
}

```

这段代码定义了两个函数：addCORSFromEnv()和addHeadersFromConfig()。

addCORSFromEnv()函数的作用是获取请求服务器所请求的CORS来源，如果从环境变量中获取不到，则使用默认的"https://"。函数检查如果从环境变量中获取到了CORS来源，则将该来源添加到请求服务器允许的CORS列表中。

addHeadersFromConfig()函数的作用是读取配置中定义的HTTP头信息，然后设置请求服务器允许的CORS头信息。它检查当前的请求头中是否包含ACA、ACAMetrics和ACACredentials头，如果是，则执行相应的操作并将结果存储到请求头中。接下来，函数会将这些头信息以及一个额外的"Server"头信息添加到请求头中。最后，它还会将请求头信息的存储键名更改为"Server"。

这两个函数一起作用，以确定允许的CORS来源和头信息，通过组合它们可以允许服务器在响应中使用CORS头信息，从而提高安全性。


```
func addCORSFromEnv(c *cmdsHttp.ServerConfig) {
	origin := os.Getenv(originEnvKey)
	if origin != "" {
		log.Warn(originEnvKeyDeprecate)
		c.AppendAllowedOrigins(origin)
	}
}

func addHeadersFromConfig(c *cmdsHttp.ServerConfig, nc *config.Config) {
	log.Info("Using API.HTTPHeaders:", nc.API.HTTPHeaders)

	if acao := nc.API.HTTPHeaders[cmdsHttp.ACAOrigin]; acao != nil {
		c.SetAllowedOrigins(acao...)
	}
	if acam := nc.API.HTTPHeaders[cmdsHttp.ACAMethods]; acam != nil {
		c.SetAllowedMethods(acam...)
	}
	for _, v := range nc.API.HTTPHeaders[cmdsHttp.ACACredentials] {
		c.SetAllowCredentials(strings.ToLower(v) == "true")
	}

	c.Headers = make(map[string][]string, len(nc.API.HTTPHeaders)+1)

	// Copy these because the config is shared and this function is called
	// in multiple places concurrently. Updating these in-place *is* racy.
	for h, v := range nc.API.HTTPHeaders {
		h = http.CanonicalHeaderKey(h)
		switch h {
		case cmdsHttp.ACAOrigin, cmdsHttp.ACAMethods, cmdsHttp.ACACredentials:
			// these are handled by the CORs library.
		default:
			c.Headers[h] = v
		}
	}
	c.Headers["Server"] = []string{"kubo/" + version.CurrentVersionNumber}
}

```

这段代码定义了两个名为 addCORSDefaults 和 patchCORSVars 的函数，用于设置 CORS（跨源资源共享）defaults 和 CORS Vars。

addCORSDefaults 函数接收一个 CmdsHttp.ServerConfig 类型的参数，并添加一些默认允许的起源。这些起源包括默认的 GET、POST 和 PUT 方法。函数通过使用 AppendAllowedOrigins 和 AppendAllowedOriginsTo 函数来设置这些允许的起源。AppendAllowedOriginsTo 函数将允许的起源列表传递给 AppendAllowedOrigins 函数进行设置。

patchCORSVars 函数同样接收一个 CmdsHttp.ServerConfig 类型的参数，并接收一个 net.Addr 类型的参数 addr。函数通过先生成一个 port 变量来获取目标服务器监听的端口，然后设置 CORS Vars。如果目标服务器监听的端口是 IPv4 或 IPv6 类型的地址，则使用该地址的端口号作为 port 变量。否则，如果目标服务器监听的端口是 UDP 类型的地址，则使用 default 的 128 和 161 端口。最后，函数使用 make 和 AppendAllowedOrigins 和 AppendAllowedOriginsTo 函数来设置允许的起源列表和 CORS Vars。

这两个函数一起工作，确保在添加 CORS Vars 时，正确设置服务器监听的端口以及允许的起源列表。


```
func addCORSDefaults(c *cmdsHttp.ServerConfig) {
	// always safelist certain origins
	c.AppendAllowedOrigins(defaultLocalhostOrigins...)
	c.AppendAllowedOrigins(companionBrowserExtensionOrigins...)

	// by default, use GET, PUT, POST
	if len(c.AllowedMethods()) == 0 {
		c.SetAllowedMethods(http.MethodGet, http.MethodPost, http.MethodPut)
	}
}

func patchCORSVars(c *cmdsHttp.ServerConfig, addr net.Addr) {
	// we have to grab the port from an addr, which may be an ip6 addr.
	// TODO: this should take multiaddrs and derive port from there.
	port := ""
	if tcpaddr, ok := addr.(*net.TCPAddr); ok {
		port = strconv.Itoa(tcpaddr.Port)
	} else if udpaddr, ok := addr.(*net.UDPAddr); ok {
		port = strconv.Itoa(udpaddr.Port)
	}

	// we're listening on tcp/udp with ports. ("udp!?" you say? yeah... it happens...)
	oldOrigins := c.AllowedOrigins()
	newOrigins := make([]string, len(oldOrigins))
	for i, o := range oldOrigins {
		// TODO: allow replacing <host>. tricky, ip4 and ip6 and hostnames...
		if port != "" {
			o = strings.Replace(o, "<port>", port, -1)
		}
		newOrigins[i] = o
	}
	c.SetAllowedOrigins(newOrigins...)
}

```

该函数`func commandsOption(cctx oldcmds.Context, command *cmds.Command, allowGet bool) ServeOption`的作用是创建一个HTTP选项函数，用于处理使用该函数的请求，并将请求转发到函数体内的命令处理函数。

具体来说，该函数以下是这样的实现：

1. 返回一个函数，该函数接收三个参数：一个上下文`cctx`，一个命令对象`command`，以及一个布尔值`allowGet`，这些参数在函数体中进行使用。
2. 在函数体中，首先创建一个HTTP配置对象`cfg`，该对象的`allowGet`字段根据`allowGet`的值设置为`true`或`false`。
3. 定义一个字符串数组`corsAllowedMethods`，该数组包含允许的HTTP方法，该数组在`allowGet`为`true`时包含`http.MethodPost`，在`allowGet`为`false`时包含`http.MethodGet`。
4. 如果`allowGet`为`true`，则将`corsAllowedMethods`中的所有方法添加到`cfg`对象的`allowedMethods`字段中，并将`APIPath`设置为`cfg.APIPath`。
5. 如果`allowGet`为`false`，则将`corsAllowedMethods`中的所有方法添加到`cfg`对象的`allowedMethods`字段中，并将`corsAllowedMethods`设置为`corsAllowedMethods`的初始值。
6. 使用`n.Repo.Config()`函数将`cfg`对象中的配置信息写入`cctx`的`config`字段中，如果配置文件存在，否则返回一个`nil`的错误。
7. 使用`addHeadersFromConfig()`函数从`cfg`对象中的配置信息中添加HTTP头部，并使用`addCORSFromEnv()`函数将`corsAllowedMethods`中的HTTP方法添加到`cfg`对象的`corsAllowedMethods`字段中。
8. 使用`addCORSDefaults()`函数将`corsAllowedMethods`中的所有方法添加到`cfg`对象的`corsAllowedMethods`字段中。
9. 使用`patchCORSVars()`函数在`cfg`对象的`corsAllowedMethods`字段中应用操作系统级别的CORS变量。
10. 使用`cmdHandler`函数创建一个HTTP命令处理函数，该函数接收一个上下文`cctx`，一个命令对象`command`，以及一个HTTP选项函数`cfg`，然后使用这些参数返回一个HTTP命令处理函数的`http.Handler`对象，该对象使用`cmdsHttp.NewHandler()`创建的函数。
11. 使用`mux.Handle()`函数和`nil`作为返回值创建一个HTTP选项函数，该函数将请求转发到函数体内的命令处理函数，并返回一个HTTP选项函数的`http.ServeMux`对象，该对象使用`oldcmds.ServeOption()`创建的函数。


```
func commandsOption(cctx oldcmds.Context, command *cmds.Command, allowGet bool) ServeOption {
	return func(n *core.IpfsNode, l net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
		cfg := cmdsHttp.NewServerConfig()
		cfg.AllowGet = allowGet
		corsAllowedMethods := []string{http.MethodPost}
		if allowGet {
			corsAllowedMethods = append(corsAllowedMethods, http.MethodGet)
		}

		cfg.SetAllowedMethods(corsAllowedMethods...)
		cfg.APIPath = APIPath
		rcfg, err := n.Repo.Config()
		if err != nil {
			return nil, err
		}

		addHeadersFromConfig(cfg, rcfg)
		addCORSFromEnv(cfg)
		addCORSDefaults(cfg)
		patchCORSVars(cfg, l.Addr())

		cmdHandler := cmdsHttp.NewHandler(&cctx, command, cfg)
		cmdHandler = otelhttp.NewHandler(cmdHandler, "corehttp.cmdsHandler")
		mux.Handle(APIPath+"/", cmdHandler)
		return mux, nil
	}
}

```

This is a Go language implementation of the `CommandsROOption` command.

The `CommandsROOption` command is used to hook read-only commands into the HTTP server, allowing GET requests.

The implementation checks the client IPFS version and compares it to the required version `/go-ipfs/` or `/kubo/`. If the versions are not equal, the HTTP server returns an error with a message indicating the version is not supported.

The `CheckVersionOption` function returns a serve option that checks the client IPFS version. It does nothing when the user agent string does not contain `/kubo/` or `/go-ipfs/`.


```
// CommandsOption constructs a ServerOption for hooking the commands into the
// HTTP server. It will NOT allow GET requests.
func CommandsOption(cctx oldcmds.Context) ServeOption {
	return commandsOption(cctx, corecommands.Root, false)
}

// CommandsROOption constructs a ServerOption for hooking the read-only commands
// into the HTTP server. It will allow GET requests.
func CommandsROOption(cctx oldcmds.Context) ServeOption {
	return commandsOption(cctx, corecommands.RootRO, true)
}

// CheckVersionOption returns a ServeOption that checks whether the client ipfs version matches. Does nothing when the user agent string does not contain `/kubo/` or `/go-ipfs/`
func CheckVersionOption() ServeOption {
	daemonVersion := version.ApiVersion

	return func(n *core.IpfsNode, l net.Listener, parent *http.ServeMux) (*http.ServeMux, error) {
		mux := http.NewServeMux()
		parent.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
			if strings.HasPrefix(r.URL.Path, APIPath) {
				cmdqry := r.URL.Path[len(APIPath):]
				pth := strings.Split(cmdqry, "/")

				// backwards compatibility to previous version check
				if len(pth) >= 2 && pth[1] != "version" {
					clientVersion := r.UserAgent()
					// skips check if client is not kubo (go-ipfs)
					if (strings.Contains(clientVersion, "/go-ipfs/") || strings.Contains(clientVersion, "/kubo/")) && daemonVersion != clientVersion {
						http.Error(w, fmt.Sprintf("%s (%s != %s)", errAPIVersionMismatch, daemonVersion, clientVersion), http.StatusBadRequest)
						return
					}
				}
			}

			mux.ServeHTTP(w, r)
		})

		return mux, nil
	}
}

```

# `/opt/kubo/core/corehttp/corehttp.go`

这段代码定义了一个名为"corehttp"的包，它提供了对HTTP包、网络包和其他高级HTTP接口的支持，以便于开发人员构建 web 界面、网关和其他基于 HTTP 的应用程序。

具体来说，该包包含了一些通用的工具函数，例如获取 HTTP 请求和响应的上下文、处理 HTTP 连接、处理 HTTP 请求头和请求体、处理 HTTP 状态码、处理 HTTP 错误等等。同时，该包还提供了一些与 IPFS(InterPlanetary File System)相关的工具函数，以帮助开发人员使用 IPFS 构建分布式文件系统。

例如，该包中的"/net/http"导入了一个名为"net"的包，其中包含了一些 HTTP 相关的函数和数据结构，例如 HTTP 请求和响应的上下文、设置 HTTP 头部和状态码、处理 HTTP 错误等等。另外，该包中的"/github.com/ipfs/go-log"导入了一个名为"logging"的包，其中包含了一些 logging 相关的函数和数据结构，例如记录 HTTP 请求和响应的日志、设置 HTTP 请求和响应的上下文等等。

该包还定义了一些函数，例如"/core/http/request上下文、请求头、请求体、响应、错误"，这些函数用于处理 HTTP 请求的上下文、请求头、请求体、响应以及错误。另外，该包中的"/core/http/package上下文"定义了一些通用的函数和数据结构，例如获取当前 HTTP 上下文、设置 HTTP 上下文等等。


```
/*
Package corehttp provides utilities for the webui, gateways, and other
high-level HTTP interfaces to IPFS.
*/
package corehttp

import (
	"context"
	"fmt"
	"net"
	"net/http"
	"time"

	logging "github.com/ipfs/go-log"
	core "github.com/ipfs/kubo/core"
	"github.com/jbenet/goprocess"
	periodicproc "github.com/jbenet/goprocess/periodic"
	ma "github.com/multiformats/go-multiaddr"
	manet "github.com/multiformats/go-multiaddr/net"
)

```

这段代码定义了一个名为 "core/server" 的 Logger，用于输出在服务器上发生的信息。

在代码中，首先通过 var 关键字定义了一个名为 log 的日志输出上下文，这个上下文将在应用程序中注册一些输出，如错误、警告和信息。

接下来，定义了一个名为 shutdownTimeout 的常量，表示在服务器停止等待挂起的时间内。如果关闭服务器的超时时间到期后，仍然有未完成的命令在运行，那么就会停止服务器并等待它们完成。

然后定义了一个名为 ServeOption 的类型，用于注册服务器在 Shutdown 过程中提供的 HTTP 处理程序。通过这个类型，可以注册不同的 HTTP 处理程序，以实现服务器提供的 HTTP 服务，如果服务器对提供的 HTTP 服务感兴趣，则可以使用 ServeOption注册一个新的 HTTP 处理程序，否则就可以使用原来的 HTTP 处理程序。

接着定义了一个名为 MakeHandler 的函数，该函数将一个包含多个 ServeOption 的选项对象转换为一个 HTTP 处理程序，这个 HTTP 处理程序会按照注册的顺序依次执行注册的 ServeOption。如果转换过程中出现错误，就会返回一个错误，否则就会返回新的 HTTP 处理程序。

最后，在函数内部，定义了一个 HTTP 处理程序，用于在请求到达时返回 HTTP 状态码 200，表示请求成功。这个处理程序使用了一个 net/http 包中的 Write 函数来写入响应，因为 net/http 包不支持 CONNECT 方法，所以在这种情况下需要使用 ServeHTTP 函数来完成 CONNECT 请求的转发。


```
var log = logging.Logger("core/server")

// shutdownTimeout is the timeout after which we'll stop waiting for hung
// commands to return on shutdown.
const shutdownTimeout = 30 * time.Second

// ServeOption registers any HTTP handlers it provides on the given mux.
// It returns the mux to expose to future options, which may be a new mux if it
// is interested in mediating requests to future options, or the same mux
// initially passed in if not.
type ServeOption func(*core.IpfsNode, net.Listener, *http.ServeMux) (*http.ServeMux, error)

// MakeHandler turns a list of ServeOptions into a http.Handler that implements
// all of the given options, in order.
func MakeHandler(n *core.IpfsNode, l net.Listener, options ...ServeOption) (http.Handler, error) {
	topMux := http.NewServeMux()
	mux := topMux
	for _, option := range options {
		var err error
		mux, err = option(n, l, mux)
		if err != nil {
			return nil, err
		}
	}
	handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// ServeMux does not support requests with CONNECT method,
		// so we need to handle them separately
		// https://golang.org/src/net/http/request.go#L111
		if r.Method == http.MethodConnect {
			w.WriteHeader(http.StatusOK)
			return
		}
		topMux.ServeHTTP(w, r)
	})
	return handler, nil
}

```

这段代码定义了一个名为 ListenAndServe 的函数，它接受一个 HTTP 服务器监听选项（如 IPFS 节点地址、监听的多个地址等）和一个或多个选项参数。函数的作用是创建并运行一个 HTTP 服务器，监听在指定的多个地址上，并返回一个用于输出 HTTP 请求的错误。

具体来说，这段代码实现了一个多路复用（multiplexing）的功能。它通过将传入的多个地址合并成一个多路复用地址，使得服务器可以监听多个地址。在函数中，首先尝试使用 manet.Listen 函数监听一个地址，如果失败，则尝试使用 ma.NewMultiaddr 函数将多个地址合并成一个多路复用地址，并使用 manet.NetListener 函数监听该地址。这样，就可以通过一个地址监听多个地址，从而实现多个 IPFS 节点之间的通信。

函数的参数部分，包括一个表示 HTTP 服务器监听选项的参数清单（ServeOption）和一个或多个输入参数，这些参数可能会用于配置 HTTP 服务器（如设置请求头、设置代理等）。


```
// ListenAndServe runs an HTTP server listening at |listeningMultiAddr| with
// the given serve options. The address must be provided in multiaddr format.
//
// TODO intelligently parse address strings in other formats so long as they
// unambiguously map to a valid multiaddr. e.g. for convenience, ":8080" should
// map to "/ip4/0.0.0.0/tcp/8080".
func ListenAndServe(n *core.IpfsNode, listeningMultiAddr string, options ...ServeOption) error {
	addr, err := ma.NewMultiaddr(listeningMultiAddr)
	if err != nil {
		return err
	}

	list, err := manet.Listen(addr)
	if err != nil {
		return err
	}

	// we might have listened to /tcp/0 - let's see what we are listing on
	addr = list.Multiaddr()
	fmt.Printf("RPC API server listening on %s\n", addr)

	return Serve(n, manet.NetListener(list), options...)
}

```

This is a Go function that creates and starts an IPFS (InterPlanetary File System) server using the IPFS-P威震服务器。IPFS is a decentralized, distributed file system that allows nodes to store and share data using a network of computers, rather than a centralized server.

The function takes in several options and returns an error or a non-empty `IpfsNode` object if the server starts successfully, or an error and an empty `IpfsNode` object if it fails to start.

The `IpfsNode` type represents an IPFS node, which can be used to store and serve files using the IPFS protocol. It has several fields, including `Addr` (the IP address of the node), `Handler` (the handler for incoming file requests), and `Ipfs` (a reference to the IPFS instance).

The `MakeHandler` function is used to create a handler for incoming file requests. This handler is passed to the `IpfsNode.GoCreate` method, which creates a new `IpfsNode` instance and returns it.

The `manet.FromNetAddr` function is used to create a `net.Listener` instance that listens for incoming network connections. This listener is passed to the `IpfsNode.GoCreate` method as an argument to the `ListenAndServe` method, which creates a new `IpfsNode` instance and starts serving files from the IPFS node.

The `select` statement is used to wait for the server to complete before continuing. If the server is closed before it finishes, the function returns an error. If the server completes successfully, the function returns an empty `IpfsNode` object. If the server fails to start, the function returns an error.


```
// Serve accepts incoming HTTP connections on the listener and pass them
// to ServeOption handlers.
func Serve(node *core.IpfsNode, lis net.Listener, options ...ServeOption) error {
	// make sure we close this no matter what.
	defer lis.Close()

	handler, err := MakeHandler(node, lis, options...)
	if err != nil {
		return err
	}

	addr, err := manet.FromNetAddr(lis.Addr())
	if err != nil {
		return err
	}

	select {
	case <-node.Process.Closing():
		return fmt.Errorf("failed to start server, process closing")
	default:
	}

	server := &http.Server{
		Handler: handler,
	}

	var serverError error
	serverProc := node.Process.Go(func(p goprocess.Process) {
		serverError = server.Serve(lis)
	})

	// wait for server to exit.
	select {
	case <-serverProc.Closed():
	// if node being closed before server exits, close server
	case <-node.Process.Closing():
		log.Infof("server at %s terminating...", addr)

		warnProc := periodicproc.Tick(5*time.Second, func(_ goprocess.Process) {
			log.Infof("waiting for server at %s to terminate...", addr)
		})

		// This timeout shouldn't be necessary if all of our commands
		// are obeying their contexts but we should have *some* timeout.
		ctx, cancel := context.WithTimeout(context.Background(), shutdownTimeout)
		defer cancel()
		err := server.Shutdown(ctx)

		// Should have already closed but we still need to wait for it
		// to set the error.
		<-serverProc.Closed()
		serverError = err

		warnProc.Close()
	}

	log.Infof("server at %s terminated", addr)
	return serverError
}

```

# `/opt/kubo/core/corehttp/gateway.go`

该代码包是旨在实现一个名为“corehttp”的库，它提供了在Go标准库中的HTTP客户端功能。它包含了来自以下外部的依赖项：
- “github.com/ipfs/boxo”的“blockservice”包
- “github.com/ipfs/boxo”的“coreiface”包
- “github.com/ipfs/boxo”的“exchange/offline”包
- “github.com/ipfs/boxo”的“files”包
- “github.com/ipfs/boxo”的“gateway”包
- “github.com/ipfs/boxo”的“namesys”包
- “github.com/ipfs/boxo”的“path”包
- “github.com/ipfs/boxo/routing/offline”包
- “github.com/ipfs/boxo/routing”包
- “github.com/ipfs/boxo/service”包
- “github.com/ipfs/boxo/token”包

这个库的作用是提供一个HTTP客户端，可以在Go应用程序中使用，用于从IPFS（InterPlanetary File System）和其他IPFS- compatible的系统中读取和写入文件。它提供了以下功能：

- 通过 namesys 包，提供对 namesys 系统的大多数功能。
- 通过 blockservice 包，提供对 IPFS 块存储的访问。
- 通过 exchange/offline 包，提供对IPFS的 off-line 交换服务。
- 通过 gateway 包， 提供IPFS作为后端网络服务。
- 通过 files 包， 提供对本地文件系统的访问。
- 通过 gateway 包， 提供IPFS作为后端网络服务。
- 通过 namesys 包， 提供对 namesys 系统的大多数功能。
- 通过 routing 包， 提供IPFS路由服务。
- 通过 offlineroute 包， 提供IPFS作为后端网络服务。
- 通过 gateway 包， 提供IPFS作为后端网络服务。
- 通过 namesys 包， 提供对 namesys 系统的大多数功能。
- 通过 core 包， 提供对IPFS核心网络服务。
- 通过 node 包， 提供对IPFS节点网络服务。
- 通过 libp2p-routing 包， 提供libp2p路由服务。
- 通过 libp2p-core 包， 提供libp2p核心网络服务。
- 通过 opentel移民计数器 库


```
package corehttp

import (
	"context"
	"errors"
	"fmt"
	"io"
	"net"
	"net/http"
	"time"

	"github.com/ipfs/boxo/blockservice"
	iface "github.com/ipfs/boxo/coreiface"
	"github.com/ipfs/boxo/exchange/offline"
	"github.com/ipfs/boxo/files"
	"github.com/ipfs/boxo/gateway"
	"github.com/ipfs/boxo/namesys"
	"github.com/ipfs/boxo/path"
	offlineroute "github.com/ipfs/boxo/routing/offline"
	"github.com/ipfs/go-cid"
	version "github.com/ipfs/kubo"
	"github.com/ipfs/kubo/config"
	"github.com/ipfs/kubo/core"
	"github.com/ipfs/kubo/core/node"
	"github.com/libp2p/go-libp2p/core/routing"
	"go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp"
)

```

该函数名为`GatewayOption`，它接收一个字符串数组`paths`作为参数。它的作用是返回一个`ServeOption`，用于设置在处理请求时使用的选项。

函数内部，首先通过调用`getGatewayConfig`函数获取配置文件中的配置信息，如果出错则返回。然后通过调用`newGatewayBackend`函数创建一个后端实例，如果出错则返回。

接下来，定义了一个名为`handler`的函数，它使用`gateway.NewHandler`创建一个处理路由的函数。然后使用`otelhttp.NewHandler`将其包装到一个`Handler`对象中，并将路由参数设置为`"Gateway"`。

最后，遍历`paths`字符串数组，将其对应的路径路由添加到`Handler`的`path`参数中，最终返回一个`ServeMux`实例，用于处理请求。如果出错，返回一个非`nil`的`error`。


```
func GatewayOption(paths ...string) ServeOption {
	return func(n *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
		config, err := getGatewayConfig(n)
		if err != nil {
			return nil, err
		}

		backend, err := newGatewayBackend(n)
		if err != nil {
			return nil, err
		}

		handler := gateway.NewHandler(config, backend)
		handler = otelhttp.NewHandler(handler, "Gateway")

		for _, p := range paths {
			mux.Handle(p+"/", handler)
		}

		return mux, nil
	}
}

```

这段代码定义了一个名为 "HostnameOption" 的函数，它返回一个名为 "ServeOption" 的函数。

函数的实现包括以下步骤：

1. 获取基于 IPAddle 存储的配置文件中的网络配置。
2. 如果配置文件不存在或出现错误，返回一个空字符串和错误。
3. 创建一个后端服务器，并将其作为参数传递给 "getGatewayConfig" 函数。
4. 创建一个名为 "childMux" 的 HTTP 服务器映射。
5. 将 "backend" 作为参数传递给 "newGatewayBackend" 函数，并将它作为 "childMux" 的后端。
6. 在 "childMux" 中处理 HTTP 请求，使用 "gateway.NewHostnameHandler" 函数在处理请求时使用获取的配置文件和后端服务器。
7. 返回 "childMux"，没有错误。


```
func HostnameOption() ServeOption {
	return func(n *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
		config, err := getGatewayConfig(n)
		if err != nil {
			return nil, err
		}

		backend, err := newGatewayBackend(n)
		if err != nil {
			return nil, err
		}

		childMux := http.NewServeMux()
		mux.Handle("/", gateway.NewHostnameHandler(config, backend, childMux))
		return childMux, nil
	}
}

```

这段代码定义了两个名为`Func`的函数，它们都返回一个名为`ServeOption`的函数，代表一个HTTP服务器的选项。这两个函数都使用`http.ServeMux`和`core.IpfsNode`作为参数，并在函数内部创建一个HTTP服务器。

具体来说，第一个函数`Func VersionOption() ServeOption`返回一个HTTP服务器，该服务器使用`blockservice.New`和`gateway.NewBlocksBackend`创建一个 blockservice 块存储器和一个 gateway，然后使用 `http.HandleFunc`和`fmt.Fprintf`函数将当前版本信息和客户端版本信息写入到HTTP响应中。

第二个函数`Func Libp2pGatewayOption() ServeOption`返回一个HTTP服务器，该服务器使用`blockservice.New`和`gateway.NewBlocksBackend`创建一个 blockservice 块存储器和一个 gateway，然后使用 `http.HandleFunc`和`gateway.WithResolver`组合函数来设置HTTP响应，并使用 `offlineGatewayErrWrapper`将错误信息包含在 response 中。


```
func VersionOption() ServeOption {
	return func(_ *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
		mux.HandleFunc("/version", func(w http.ResponseWriter, r *http.Request) {
			fmt.Fprintf(w, "Commit: %s\n", version.CurrentCommit)
			fmt.Fprintf(w, "Client Version: %s\n", version.GetUserAgentVersion())
		})
		return mux, nil
	}
}

func Libp2pGatewayOption() ServeOption {
	return func(n *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
		bserv := blockservice.New(n.Blocks.Blockstore(), offline.Exchange(n.Blocks.Blockstore()))

		backend, err := gateway.NewBlocksBackend(bserv,
			// GatewayOverLibp2p only returns things that are in local blockstore
			// (same as Gateway.NoFetch=true), we have to pass offline path resolver
			gateway.WithResolver(n.OfflineUnixFSPathResolver),
		)
		if err != nil {
			return nil, err
		}

		gwConfig := gateway.Config{
			DeserializedResponses: false,
			NoDNSLink:             true,
			PublicGateways:        nil,
			Menu:                  nil,
		}

		handler := gateway.NewHandler(gwConfig, &offlineGatewayErrWrapper{gwimpl: backend})
		handler = otelhttp.NewHandler(handler, "Libp2p-Gateway")

		mux.Handle("/ipfs/", handler)

		return mux, nil
	}
}

```

This is a Go-like language construct that initializes an OfflineGateway instance. It takes a configuration file and可能的 errors related to it.

The configuration file, specified by the `-f` or `--config` flag, is parsed and passed to the `n.Repo.Config()` method, which initializes the Blockstore, Blockstream, and other components of the OfflineGateway. If there are any errors related to the configuration, they are added to the `offlineGatewayErrWrapper` struct, which will be returned along with the initialized `backend` instance.

The `-c` or `--证书` flag is used to specify the root certificate used by the OfflineGateway's Blockchain. It should be passed to the `n.DNSResolver.RootCertificate()` method, which sets the root certificate for the DNS resolver.

The `-ns` or `--服务` flag is used to specify the number of namespaces used by the OfflineGateway. It should be passed to the `n.Namesys` constructor, which sets the number of namespaces used by the OfflineGateway.

The `-nd` or `--数据库` flag is used to specify the name of the database used by the OfflineGateway's Blockstore. It should be passed to the `n.Repo.SetDatabase()` method, which initializes the Blockstore.

The `-nf` or `--下载权限` flag is used to specify the minimum number of blocks that can be downloaded by the OfflineGateway. It should be passed to the `n.Blocks` constructor, which sets the minimum number of blocks that can be downloaded by the OfflineGateway. If this number is set to zero or negative, it means that the OfflineGateway won't download any blocks and can be used only for unregistering blocks from the Blockstore.


```
func newGatewayBackend(n *core.IpfsNode) (gateway.IPFSBackend, error) {
	cfg, err := n.Repo.Config()
	if err != nil {
		return nil, err
	}

	bserv := n.Blocks
	var vsRouting routing.ValueStore = n.Routing
	nsys := n.Namesys
	pathResolver := n.UnixFSPathResolver

	if cfg.Gateway.NoFetch {
		bserv = blockservice.New(bserv.Blockstore(), offline.Exchange(bserv.Blockstore()))

		cs := cfg.Ipns.ResolveCacheSize
		if cs == 0 {
			cs = node.DefaultIpnsCacheSize
		}
		if cs < 0 {
			return nil, fmt.Errorf("cannot specify negative resolve cache size")
		}

		vsRouting = offlineroute.NewOfflineRouter(n.Repo.Datastore(), n.RecordValidator)
		nsys, err = namesys.NewNameSystem(vsRouting,
			namesys.WithDatastore(n.Repo.Datastore()),
			namesys.WithDNSResolver(n.DNSResolver),
			namesys.WithCache(cs))
		if err != nil {
			return nil, fmt.Errorf("error constructing namesys: %w", err)
		}

		// Gateway.NoFetch=true requires offline path resolver
		// to avoid fetching missing blocks during path traversal
		pathResolver = n.OfflineUnixFSPathResolver
	}

	backend, err := gateway.NewBlocksBackend(bserv,
		gateway.WithValueStore(vsRouting),
		gateway.WithNameSystem(nsys),
		gateway.WithResolver(pathResolver),
	)
	if err != nil {
		return nil, err
	}
	return &offlineGatewayErrWrapper{gwimpl: backend}, nil
}

```

该代码定义了一个名为offlineGatewayErrWrapper的结构体，其中包含一个IPFSBackend类型的变量gwimpl，以及一个名为offlineErrWrap的函数，该函数接收一个错误参数err，并返回一个修改后的错误值err。

函数中的if语句检查给定的错误是否为iface.ErrOffline，如果是，则返回一个错误字符串，其中包含错误细节，例如"无法连接到IPFS服务器"。否则，函数继续执行，将错误作为返回值返回。

函数中的另一个if语句检查给定的错误是否是一个IPFSBackend实现的错误。如果是，则返回错误并使用错误代码iface.ErrServiceUnavailable。否则，函数继续执行，将错误作为返回值返回。

函数中的Get函数使用IPFSBackend的Get方法来获取指定路径的元数据，包括范围。函数的第一个参数是一个path.ImmutablePath类型的变量，用于指定要获取的路径。第二个参数是一个可选的多个gateway.ByteRange类型的参数，用于指定搜索范围。该函数使用offlineErrWrap函数来处理返回的错误，以便在返回错误的同时能够正确地传递错误信息。

最后，该代码创建了一个offlineGatewayErrWrapper类型的变量o，并将其设置为IPFSBackend类型的gwimpl，以便在需要时使用该变量来获取IPFS元数据。


```
type offlineGatewayErrWrapper struct {
	gwimpl gateway.IPFSBackend
}

func offlineErrWrap(err error) error {
	if errors.Is(err, iface.ErrOffline) {
		return fmt.Errorf("%s : %w", err.Error(), gateway.ErrServiceUnavailable)
	}
	return err
}

func (o *offlineGatewayErrWrapper) Get(ctx context.Context, path path.ImmutablePath, ranges ...gateway.ByteRange) (gateway.ContentPathMetadata, *gateway.GetResponse, error) {
	md, n, err := o.gwimpl.Get(ctx, path, ranges...)
	err = offlineErrWrap(err)
	return md, n, err
}

```

这三个人工智能函数函数包装了一个名为offlineGatewayErrWrapper的传输上下文中的错误对象。它们接收一个名为ctx的上下文，一个名为path的路径变量和一个名为path的 immutablePath参数。

这些函数的作用是通过调用offlineGatewayErrWrapper中的三个函数来获取指定路径的文件信息。这些函数包括：

1. GetAll：函数接收一个contentPathMetadata类型的参数，一个Node类型的参数和一个error类型的参数。它使用offlineGatewayErrWrapper中的gwimpl.GetAll函数来获取指定路径的所有文件，将错误对象offlineErrWrap传递给GetAll函数，如果错误发生，则返回0。GetAll函数返回的内容包括contentPathMetadata,Node和error三个值。

2. GetBlock：函数与GetAll类似，只是使用了offlineGatewayErrWrapper中的gwimpl.GetBlock函数。它同样接收一个contentPathMetadata类型的参数，一个Node类型的参数和一个error类型的参数。使用offlineErrWrap来处理错误，并返回0。

3. Head：函数与GetAll和GetBlock类似，只是使用了offlineGatewayErrWrapper中的gwimpl.Head函数。它同样接收一个contentPathMetadata类型的参数，一个Node类型的参数和一个error类型的参数。使用offlineErrWrap来处理错误，并返回0。

这些函数可以确保在函数调用成功时，获取指定路径的文件信息。如果函数调用失败，则使用offlineGatewayErrWrapper中的错误处理函数来捕获和处理错误。


```
func (o *offlineGatewayErrWrapper) GetAll(ctx context.Context, path path.ImmutablePath) (gateway.ContentPathMetadata, files.Node, error) {
	md, n, err := o.gwimpl.GetAll(ctx, path)
	err = offlineErrWrap(err)
	return md, n, err
}

func (o *offlineGatewayErrWrapper) GetBlock(ctx context.Context, path path.ImmutablePath) (gateway.ContentPathMetadata, files.File, error) {
	md, n, err := o.gwimpl.GetBlock(ctx, path)
	err = offlineErrWrap(err)
	return md, n, err
}

func (o *offlineGatewayErrWrapper) Head(ctx context.Context, path path.ImmutablePath) (gateway.ContentPathMetadata, *gateway.HeadResponse, error) {
	md, n, err := o.gwimpl.Head(ctx, path)
	err = offlineErrWrap(err)
	return md, n, err
}

```

这段代码定义了三个函数，分别接收一个offlineGatewayErrWrapper类型的参数，并返回一个gateway.ContentPathMetadata类型的结果和一个error类型的结果。

第一个函数是ResolvePath函数，它接收一个context.Context和一个ImmutablePath类型的参数，并返回一个gateway.ContentPathMetadata类型的结果和一个error类型的结果。函数首先调用offlineGatewayErrWrapper中提供的ResolvePath函数，如果该函数成功，则使用其返回的结果作为第一个返回值，否则使用其返回的错误作为第二个返回值。

第二个函数是GetCAR函数，它接收一个context.Context和一个ImmutablePath类型的参数，并返回一个gateway.ContentPathMetadata类型的结果一个io.ReadCloser类型的结果和一个error类型的结果。函数首先调用offlineGatewayErrWrapper中提供的GetCAR函数，如果该函数成功，则使用其返回的结果作为第一个返回值，否则使用其返回的错误作为第二个返回值。

第三个函数是IsCached函数，它接收一个context.Context和一个路径类型的参数，并返回一个bool类型的结果。函数直接调用offlineGatewayErrWrapper中提供的IsCached函数，并返回其返回的结果。


```
func (o *offlineGatewayErrWrapper) ResolvePath(ctx context.Context, path path.ImmutablePath) (gateway.ContentPathMetadata, error) {
	md, err := o.gwimpl.ResolvePath(ctx, path)
	err = offlineErrWrap(err)
	return md, err
}

func (o *offlineGatewayErrWrapper) GetCAR(ctx context.Context, path path.ImmutablePath, params gateway.CarParams) (gateway.ContentPathMetadata, io.ReadCloser, error) {
	md, data, err := o.gwimpl.GetCAR(ctx, path, params)
	err = offlineErrWrap(err)
	return md, data, err
}

func (o *offlineGatewayErrWrapper) IsCached(ctx context.Context, path path.Path) bool {
	return o.gwimpl.IsCached(ctx, path)
}

```

这段代码定义了三个函数，分别是 `func (o *offlineGatewayErrWrapper) GetIPNSRecord(ctx context.Context, c cid.Cid) ([]byte, error)`，`func (o *offlineGatewayErrWrapper) ResolveMutable(ctx context.Context, path path.Path) (path.ImmutablePath, time.Duration, time.Time, error)` 和 `func (o *offlineGatewayErrWrapper) GetDNSLinkRecord(ctx context.Context, s string) (path.Path, error)`。

这些函数的作用如下：

1. `GetIPNSRecord` 函数接收一个 `offlineGatewayErrWrapper` 类型的参数 `o` 和一个 `cid` 类型的参数 `c`。它使用 `o.gwimpl.GetIPNSRecord(ctx, c)` 方法获取一个 IPNS 记录，并将其包装在一个 `[]byte` 类型的数组中。如果返回值为 `nil`，则表示出错，并返回一个非 `nil` 的错误对象。
2. `ResolveMutable` 函数接收一个 `offlineGatewayErrWrapper` 类型的参数 `o` 和一个 `path` 类型的参数 `path`。它使用 `o.gwimpl.ResolveMutable(ctx, path)` 方法将 `path` 解析为一个可变长度的 `path.ImmutablePath` 类型，设置一个保留时差（`ttl`）和一个最后的修改时间（`lastMod`）。如果返回值为 `nil`，则表示出错，并返回一个非 `nil` 的错误对象。
3. `GetDNSLinkRecord` 函数接收一个 `offlineGatewayErrWrapper` 类型的参数 `o` 和一个 `s` 类型的参数 `s`。它使用 `o.gwimpl.GetDNSLinkRecord(ctx, s)` 方法获取一个 DNS 链接记录，并将其包装成一个 `path.Path` 类型的路径。如果返回值为 `nil`，则表示出错，并返回一个非 `nil` 的错误对象。

由于这些函数使用了 `offlineGatewayErrWrapper` 类型的对象，它们都有 `GetIPNSRecord()`，`ResolveMutable()` 和 `GetDNSLinkRecord()` 方法，这些方法的具体实现可能因具体的网络服务提供商（ISP）而有所不同。


```
func (o *offlineGatewayErrWrapper) GetIPNSRecord(ctx context.Context, c cid.Cid) ([]byte, error) {
	rec, err := o.gwimpl.GetIPNSRecord(ctx, c)
	err = offlineErrWrap(err)
	return rec, err
}

func (o *offlineGatewayErrWrapper) ResolveMutable(ctx context.Context, path path.Path) (path.ImmutablePath, time.Duration, time.Time, error) {
	imPath, ttl, lastMod, err := o.gwimpl.ResolveMutable(ctx, path)
	err = offlineErrWrap(err)
	return imPath, ttl, lastMod, err
}

func (o *offlineGatewayErrWrapper) GetDNSLinkRecord(ctx context.Context, s string) (path.Path, error) {
	p, err := o.gwimpl.GetDNSLinkRecord(ctx, s)
	err = offlineErrWrap(err)
	return p, err
}

```

This is a function that initializes the gateway configuration of the incoming request. It does this by setting up the headers and the default deserialized responses, and then populating the public gateways of the gateway configuration.

The function takes in a set of default known gateways as a parameter, and also accepts an optional parameter for the configuration of the public gateways.

It then initializes the gateway configuration by setting the headers and the default deserialized responses, and also disabling HTTP errors if they are enabled in the gateway configuration.

Finally, it applies the settings from the public gateways configuration, if they exist, and also applies any default settings for the public gateways, such as the use of subdomains or inline DNS linking.

It returns the gateway configuration and a nil error.


```
var _ gateway.IPFSBackend = (*offlineGatewayErrWrapper)(nil)

var defaultPaths = []string{"/ipfs/", "/ipns/", "/api/", "/p2p/"}

var subdomainGatewaySpec = &gateway.PublicGateway{
	Paths:         defaultPaths,
	UseSubdomains: true,
}

var defaultKnownGateways = map[string]*gateway.PublicGateway{
	"localhost": subdomainGatewaySpec,
}

func getGatewayConfig(n *core.IpfsNode) (gateway.Config, error) {
	cfg, err := n.Repo.Config()
	if err != nil {
		return gateway.Config{}, err
	}

	// Parse configuration headers and add the default Access Control Headers.
	headers := make(map[string][]string, len(cfg.Gateway.HTTPHeaders))
	for h, v := range cfg.Gateway.HTTPHeaders {
		headers[http.CanonicalHeaderKey(h)] = v
	}
	gateway.AddAccessControlHeaders(headers)

	// Initialize gateway configuration, with empty PublicGateways, handled after.
	gwCfg := gateway.Config{
		Headers:               headers,
		DeserializedResponses: cfg.Gateway.DeserializedResponses.WithDefault(config.DefaultDeserializedResponses),
		DisableHTMLErrors:     cfg.Gateway.DisableHTMLErrors.WithDefault(config.DefaultDisableHTMLErrors),
		NoDNSLink:             cfg.Gateway.NoDNSLink,
		PublicGateways:        map[string]*gateway.PublicGateway{},
	}

	// Add default implicit known gateways, such as subdomain gateway on localhost.
	for hostname, gw := range defaultKnownGateways {
		gwCfg.PublicGateways[hostname] = gw
	}

	// Apply values from cfg.Gateway.PublicGateways if they exist.
	for hostname, gw := range cfg.Gateway.PublicGateways {
		if gw == nil {
			// Remove any implicit defaults, if present. This is useful when one
			// wants to disable subdomain gateway on localhost, etc.
			delete(gwCfg.PublicGateways, hostname)
			continue
		}

		gwCfg.PublicGateways[hostname] = &gateway.PublicGateway{
			Paths:                 gw.Paths,
			NoDNSLink:             gw.NoDNSLink,
			UseSubdomains:         gw.UseSubdomains,
			InlineDNSLink:         gw.InlineDNSLink.WithDefault(config.DefaultInlineDNSLink),
			DeserializedResponses: gw.DeserializedResponses.WithDefault(gwCfg.DeserializedResponses),
		}
	}

	return gwCfg, nil
}

```

# `/opt/kubo/core/corehttp/gateway_test.go`

该代码包是用于测试 Core API 的工具包，它包含了与测试相关的函数和变量。

具体来说，该代码包的作用是用于测试 Core API 的功能是否符合预期。它包含了一些函数和变量，用于测试 Core API 是否能够正确地执行各种操作，例如创建、获取、更新和删除资源等。

例如，该代码包包含了一个名为 "testCreateHTTPContext" 的函数，用于创建一个 HTTP 上下文，用于与 Core API 进行交互。在这个函数中，通过调用 "net/http/httptest" 包中的 "http.DefaultContext" 函数，创建了一个 HTTP 上下文对象。

另外，该代码包还包含了一个名为 "testCreateGatewayService" 的函数，用于创建一个名为 "gateway-service" 的路由器服务。在这个函数中，通过调用 "net/http/contrib/gateway/ctx/config.GatewayServiceConfig" 函数，创建了一个路由器服务配置对象。

总之，该代码包的作用是用于测试 Core API 的功能，通过编写测试函数来验证它是否能够正确地执行各种操作，确保它能够正常工作。


```
package corehttp

import (
	"context"
	"errors"
	"io"
	"net/http"
	"net/http/httptest"
	"strings"
	"testing"

	"github.com/ipfs/boxo/namesys"
	version "github.com/ipfs/kubo"
	"github.com/ipfs/kubo/core"
	"github.com/ipfs/kubo/core/coreapi"
	"github.com/ipfs/kubo/repo"
	"github.com/stretchr/testify/assert"

	iface "github.com/ipfs/boxo/coreiface"
	"github.com/ipfs/boxo/path"
	"github.com/ipfs/go-datastore"
	syncds "github.com/ipfs/go-datastore/sync"
	"github.com/ipfs/kubo/config"
	ci "github.com/libp2p/go-libp2p/core/crypto"
)

```

该代码定义了一个名为 `mockNamesys` 的类型，该类型实现了一个 `Resolve` 函数，用于在给定路径 `p` 和可选的 `opts` 的情况下，返回一个 `namesys.Result` 类型的结果，或者一个 `namesys.ErrResolveRecursion` 类型的错误。

该函数的实现采用了 Recursive 风格，其中使用了 `namesys.DefaultResolveOptions` 来设置默认的解析选项，包括最大深度为无限大。函数在 `Resolve` 函数中，遍历了给定的 `opts` 中的解析选项，并对每个解析选项进行了设置。然后，函数根据给定的路径 `p`，递归地查找给定路径下标为 0 的子路径，如果子路径存在于给定深度下的某个路径中，则返回一个 `namesys.Result` 类型的结果，否则返回一个 `namesys.ErrResolveFailed` 类型的错误。在查找子路径时，函数使用了路径 `path.SegmentsToString` 将路径分割为元素，并遍历每个元素。如果元素是 "ipns/" 的前缀，则设置深度为 0，并返回一个 `namesys.Result` 类型的结果。如果元素不是以 "/ipns/" 开头的元素，则设置深度为 depth - 1，并尝试从 `mockNamesys` 映射中获取该元素的值，如果映射成功，则返回该元素的值，否则返回一个 `namesys.ErrResolveFailed` 类型的错误。最后，函数返回结果，并不会输出源代码。


```
type mockNamesys map[string]path.Path

func (m mockNamesys) Resolve(ctx context.Context, p path.Path, opts ...namesys.ResolveOption) (namesys.Result, error) {
	cfg := namesys.DefaultResolveOptions()
	for _, o := range opts {
		o(&cfg)
	}
	depth := cfg.Depth
	if depth == namesys.UnlimitedDepth {
		// max uint
		depth = ^uint(0)
	}
	var (
		value path.Path
	)
	name := path.SegmentsToString(p.Segments()[:2]...)
	for strings.HasPrefix(name, "/ipns/") {
		if depth == 0 {
			return namesys.Result{Path: value}, namesys.ErrResolveRecursion
		}
		depth--

		v, ok := m[name]
		if !ok {
			return namesys.Result{}, namesys.ErrResolveFailed
		}
		value = v
		name = value.String()
	}

	value, err := path.Join(value, p.Segments()[2:]...)
	return namesys.Result{Path: value}, err
}

```

该代码定义了三个函数：

1. `func (m mockNamesys) ResolveAsync(ctx context.Context, p path.Path, opts ...namesys.ResolveOption) <-chan namesys.AsyncResult {
	out := make(chan namesys.AsyncResult, 1)
	res, err := m.Resolve(ctx, p, opts...)
	out <- namesys.AsyncResult{Path: res.Path, TTL: res.TTL, LastMod: res.LastMod, Err: err}
	close(out)
	return out
}`
该函数接收一个名为 `p` 的路径参数，一个或多个名为 `opts` 的参数，以及一个或多个可选的 `namesys.ResolveOption` 类型参数。它使用 `m.Resolve` 函数解决给定路径和参数 `opts` 并返回一个解析器。然后，它创建一个名为 `out` 的通道，将解析器生成的 async 结果发送到该通道中。最后，它关闭输出通道并返回它。

2. `func (m mockNamesys) Publish(ctx context.Context, name ci.PrivKey, value path.Path, opts ...namesys.PublishOption) error {
	return errors.New("not implemented for mockNamesys")
}`
该函数没有实现，只是声明了一个名为 `Publish` 的函数。由于它没有实现，因此无法提供任何具体的错误信息。

3. `func (m mockNamesys) GetResolver(subs string) (namesys.Resolver, bool)`
该函数接收一个名为 `subs` 的字符串参数，并返回一个解析器和一个布尔值，表示是否成功获取解析器。由于它没有实现，因此无法提供任何具体的信息。


```
func (m mockNamesys) ResolveAsync(ctx context.Context, p path.Path, opts ...namesys.ResolveOption) <-chan namesys.AsyncResult {
	out := make(chan namesys.AsyncResult, 1)
	res, err := m.Resolve(ctx, p, opts...)
	out <- namesys.AsyncResult{Path: res.Path, TTL: res.TTL, LastMod: res.LastMod, Err: err}
	close(out)
	return out
}

func (m mockNamesys) Publish(ctx context.Context, name ci.PrivKey, value path.Path, opts ...namesys.PublishOption) error {
	return errors.New("not implemented for mockNamesys")
}

func (m mockNamesys) GetResolver(subs string) (namesys.Resolver, bool) {
	return nil, false
}

```

这段代码定义了一个名为 `newNodeWithMockNamesys` 的函数，它接收一个名为 `ns` 的 `core.IpfsNode` 类型的参数。函数的作用是在 `core.IpfsNode` 类型的 `ns` 中创建一个新的节点，并在节点上设置 `namesys` 字段为 `ns` 本身。

具体来说，这段代码在以下步骤中实现了 `newNodeWithMockNamesys` 函数：

1. 创建一个配置对象 `c`，该对象包含一个 `core.Identity` 字段，其中 `PeerID` 字段指定了一个需要使用的虚拟节点 ID。

2. 创建一个 `repo.Mock` 类型的对象 `r`，该对象包含一个 `core.IpfsNode` 类型的字段 `c` 和一个 `syncds.MutexWrap` 类型的字段 `D`。`D` 类型表示一个数据存储库，用于管理 `r` 对象的 `datastore` 字段。

3. 使用 `core.NewNode` 函数创建一个新的节点 `n`，该节点使用上面创建的 `r` 对象中的数据存储库。

4. 将 `n` 的 `namesys` 字段设置为 `ns` 本身。

5. 如果创建节点过程中出现错误，函数返回一个 `error` 类型的值。

6. 返回 `n` 和 `n` 创建失败时可能返回的 `error` 类型。


```
func newNodeWithMockNamesys(ns mockNamesys) (*core.IpfsNode, error) {
	c := config.Config{
		Identity: config.Identity{
			PeerID: "QmTFauExutTsy4XP6JbMFcw2Wa9645HJt2bTqL6qYDCKfe", // required by offline node
		},
	}
	r := &repo.Mock{
		C: c,
		D: syncds.MutexWrap(datastore.NewMapDatastore()),
	}
	n, err := core.NewNode(context.Background(), &core.BuildCfg{Repo: r})
	if err != nil {
		return nil, err
	}
	n.Namesys = ns
	return n, nil
}

```

这段代码定义了一个名为delegatedHandler的结构体，它是一个delegatedHandler类型的处理程序。

该处理程序包含一个HTTP处理程序，因此可以像一般的HTTP处理程序一样处理HTTP请求。

该处理程序包含一个名为doWithoutRedirect的函数，它接受一个HTTP请求并返回一个HTTP响应。这个函数使用了一个HTTP客户端，并且使用了一个custom的redirect handler，当出现redirect时，会直接返回一个带标签的http错误，这个标签为"without-redirect"。

该函数的实现中，首先对请求进行分析，如果请求带有标签，则执行该标签对应的处理程序，否则创建一个HTTP错误并返回。

最后，该函数使用一个名为c的HTTP客户端，执行请求并返回响应。


```
type delegatedHandler struct {
	http.Handler
}

func (dh *delegatedHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	dh.Handler.ServeHTTP(w, r)
}

func doWithoutRedirect(req *http.Request) (*http.Response, error) {
	tag := "without-redirect"
	c := &http.Client{
		CheckRedirect: func(req *http.Request, via []*http.Request) error {
			return errors.New(tag)
		},
	}
	res, err := c.Do(req)
	if err != nil && !strings.Contains(err.Error(), tag) {
		return nil, err
	}
	return res, nil
}

```

该函数 `newTestServerAndNode` 接受两个参数：

1. `t`： testing.T 类型的变量，用于测试；
2. `ns`：一个名为 `namesys` 的模拟命名空间。

函数内部执行以下步骤：

1. 尝试调用 `newNodeWithMockNamesys` 函数，并将结果存储在 `ns` 变量中。如果出错，函数结束；如果成功，继续执行下面步骤。
2. 构造一个代表客户端的 `httptest.Server` 实例，以及一个代表服务器的 `http.Handler` 实例。构造过程包括设置监听器、服务器端处理程序以及一些选项，如使用 "/ipfs" 和 "/ipns" 协议头，以及指定版本选项。
3. 设置代表服务器的代理处理程序，并使用上面构造的 `Handler` 实例。
4. 设置代表客户端的代理处理程序，并使用上面构造的 `Handler` 实例。
5. 返回上面构造的服务器实例、代理处理程序实例和上下文。

该函数的作用是创建一个测试服务器，用于测试 `coreapi` 和 `httptest` 包的功能。它通过创建一个代表客户端和服务器的 `httptest.Server` 实例，以及一个代表服务器的 `http.Handler` 实例，来构建一个可以进行客户端和服务器通信的测试环境。通过这种方式，可以方便地进行测试，而无需编写实际的应用程序。


```
func newTestServerAndNode(t *testing.T, ns mockNamesys) (*httptest.Server, iface.CoreAPI, context.Context) {
	n, err := newNodeWithMockNamesys(ns)
	if err != nil {
		t.Fatal(err)
	}

	// need this variable here since we need to construct handler with
	// listener, and server with handler. yay cycles.
	dh := &delegatedHandler{}
	ts := httptest.NewServer(dh)
	t.Cleanup(func() { ts.Close() })

	dh.Handler, err = MakeHandler(n,
		ts.Listener,
		HostnameOption(),
		GatewayOption("/ipfs", "/ipns"),
		VersionOption(),
	)
	if err != nil {
		t.Fatal(err)
	}

	api, err := coreapi.NewCoreAPI(n)
	if err != nil {
		t.Fatal(err)
	}

	return ts, api, n.Context()
}

```

该代码段是一个名为 "TestVersion" 的函数，它是 testing 包中的一个测试函数。函数接收一个 testing.T 类型的参数，然后执行以下操作：

1. 将版本号设置为 "theshortcommithash"。
2. 创建一个 mockNamesys 对象。
3. 创建一个新的测试服务器和节点，并将其存储在 ns 变量中。
4. 将测试服务器 URL 存储在 t.logf 函数中。
5. 创建一个 HTTP GET 请求，将请求的 URL 设置为测试服务器 URL 和版本号，然后将请求发送给服务器。
6. 获取服务器返回的响应，并将其存储在 body 变量中。
7. 遍历响应体，查找其中包含 "Commit: theshortcommithash" 和 "Client Version: "+version.GetUserAgentVersion() 的行。
8. 如果响应体中包含 "Commit: theshortcommithash"，则执行下列操作：
	* 检查 "Client Version: "+version.GetUserAgentVersion() 是否与 "Commit: theshortcommithash" 一起出现。
	* 不是。
	* 输出一条消息并指出响应体中错误的内容。
9. 如果响应体中包含 "Client Version: "+version.GetUserAgentVersion()"，则执行下列操作：
	* 检查版本号是否包含 "Commit: theshortcommithash"。
	* 是。
	* 输出一条消息并指出响应体中错误的内容。

函数的作用是验证版本号是否正确，并且检查客户端版本是否与服务器版本兼容。


```
func TestVersion(t *testing.T) {
	version.CurrentCommit = "theshortcommithash"

	ns := mockNamesys{}
	ts, _, _ := newTestServerAndNode(t, ns)
	t.Logf("test server url: %s", ts.URL)

	req, err := http.NewRequest(http.MethodGet, ts.URL+"/version", nil)
	if err != nil {
		t.Fatal(err)
	}

	res, err := doWithoutRedirect(req)
	if err != nil {
		t.Fatal(err)
	}
	body, err := io.ReadAll(res.Body)
	if err != nil {
		t.Fatalf("error reading response: %s", err)
	}
	s := string(body)

	if !strings.Contains(s, "Commit: theshortcommithash") {
		t.Fatalf("response doesn't contain commit:\n%s", s)
	}

	if !strings.Contains(s, "Client Version: "+version.GetUserAgentVersion()) {
		t.Fatalf("response doesn't contain client version:\n%s", s)
	}
}

```

This is a Go test function that tests the `TestDeserializedResponsesInheritance` function. It takes a testing package (t.) and a test case (testCase) as input.

The function iterates over a list of test cases, each of which defines a configuration setting for the identity (IDentity) and gateway (Gateway) settings. The `globalSetting` and `gatewaySetting` settings are optional.

For each test case, the function creates a `config.Config` object and sets the identity and gateway settings. It then calls the `core.NewNode` method to create a new offline node, and returns the node.

The function then calls the `getGatewayConfig` function to retrieve the gateway configuration for the offline node. It then asserts that the configuration contains a public gateway for the "example.com" domain, and asserts that the gateway has the expected `DeserializedResponses`.

Overall, this function tests that the `TestDeserializedResponsesInheritance` function works correctly and handles various combinations of global setting and gateway setting values.


```
func TestDeserializedResponsesInheritance(t *testing.T) {
	for _, testCase := range []struct {
		globalSetting          config.Flag
		gatewaySetting         config.Flag
		expectedGatewaySetting bool
	}{
		{config.True, config.Default, true},
		{config.False, config.Default, false},
		{config.False, config.True, true},
		{config.True, config.False, false},
	} {
		c := config.Config{
			Identity: config.Identity{
				PeerID: "QmTFauExutTsy4XP6JbMFcw2Wa9645HJt2bTqL6qYDCKfe", // required by offline node
			},
			Gateway: config.Gateway{
				DeserializedResponses: testCase.globalSetting,
				PublicGateways: map[string]*config.GatewaySpec{
					"example.com": {
						DeserializedResponses: testCase.gatewaySetting,
					},
				},
			},
		}
		r := &repo.Mock{
			C: c,
			D: syncds.MutexWrap(datastore.NewMapDatastore()),
		}
		n, err := core.NewNode(context.Background(), &core.BuildCfg{Repo: r})
		assert.NoError(t, err)

		gwCfg, err := getGatewayConfig(n)
		assert.NoError(t, err)

		assert.Contains(t, gwCfg.PublicGateways, "example.com")
		assert.Equal(t, testCase.expectedGatewaySetting, gwCfg.PublicGateways["example.com"].DeserializedResponses)
	}
}

```

# `/opt/kubo/core/corehttp/logs.go`

这段代码定义了一个名为`writeErrNotifier`的结构体，它包含一个`Writer`类型的字段`w`和一个`ErrorChan`类型的字段`errs`。

接下来，定义了一个名为`writeErrNotifier`的函数，它接受一个`WriteError`类型的参数。

该函数使用`net/http`库创建一个HTTP请求，并将请求的所有相关参数作为参数传递给`net.http.Client.Do`函数。如果 HTTP 请求成功，它将在`errs`字段中传递一个`WriteError`类型的错误，否则将在`errs`字段中传递一个未定义的错误类型。

函数返回一个`WriteError`类型的结果，表示尝试写入到外部网络中的任何错误。


```
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

```

此代码定义了一个名为 `newWriteErrNotifier` 的函数，它接收一个名为 `w` 的 `io.Writer` 类型的参数。

该函数返回一个名为 `writeErrNotifier` 的类型，该类型包含一个 `io.WriteCloser` 和一个通道 `<-chan error>`。函数创建了一个名为 `ch` 的通道，用于存储所有错误或异常信息。

函数的 `Write` 方法接收一个字节切片 `b`，并返回写入到 `w` 的字节数和可能的错误。如果错误发生，函数将 `errs` 通道中的当前错误添加到 `errs`  channel 中，并将函数返回值设置为零。如果 `w` 是一个 `http.Flooter` 类型的值，则调用 `Flush` 方法将所有错误或异常信息发送到 `w` 缓冲区中。

函数返回的 `n` 表示写入到 `w` 的字节数，`err` 表示可能的错误。如果错误发生，函数将返回一个非零值，否则返回写入到 `w` 的字节数。


```
func newWriteErrNotifier(w io.Writer) (io.WriteCloser, <-chan error) {
	ch := make(chan error, 1)
	return &writeErrNotifier{
		w:    w,
		errs: ch,
	}, ch
}

func (w *writeErrNotifier) Write(b []byte) (int, error) {
	n, err := w.w.Write(b)
	if err != nil {
		select {
		case w.errs <- err:
		default:
		}
	}
	if f, ok := w.w.(http.Flusher); ok {
		f.Flush()
	}
	return n, err
}

```

这两段代码都是Go语言中的函数，它们的作用是不同的。

第一段代码是一个函数，它接收一个WriteErrNotifier类型的变量w，然后执行一个select语句。在select语句中，它等待w.errs <- io.EOF这条语句，如果w.errs是<> nil，那么它就会执行w.errs <- io.EOF这条语句，关闭所有错误并返回一个 nil 值。否则，它会执行default语句，没有做任何特殊处理。函数返回一个 nil 值。

第二段代码定义了一个ServeOption类型的函数，它接收一个IpfsNode类型的参数n，一个网络听众类型的参数_net.Listener，和一个http.ServeMux类型的参数mux。函数返回一个http.ServeMux类型的参数，如果没有错误，它会在mux上添加一个以"/logs"为路径的HTTP请求处理函数。这个函数会接收一个http.ResponseWriter类型的参数w和一个http.Request类型的参数r，然后它会将w.WriteHeader(200)并把wnf, errs这两个变量添加到mux.HandleFunc中的参数中。最后，函数返回一个http.ServeMux类型的参数，如果没有错误，它会在mux上返回一个 nil 值。


```
func (w *writeErrNotifier) Close() error {
	select {
	case w.errs <- io.EOF:
	default:
	}
	return nil
}

func LogOption() ServeOption {
	return func(n *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
		mux.HandleFunc("/logs", func(w http.ResponseWriter, r *http.Request) {
			w.WriteHeader(200)
			wnf, errs := newWriteErrNotifier(w)
			lwriter.WriterGroup.AddWriter(wnf)
			log.Event(n.Context(), "log API client connected") //nolint deprecated
			<-errs
		})
		return mux, nil
	}
}

```

# `/opt/kubo/core/corehttp/metrics.go`

这段代码是一个 Go 语言中的 package，它导入了 CoreHTTP、Core、net、time 和 statistics 等几个库。

首先，它导入了 CoreHTTP 和 Core库，这些库可能与网络 HTTP 请求和响应有关。

然后，它导入了 net 和 time 库，这些库与网络通信有关。

接着，它导入了 statistics 库，这个库与统计数据收集和分析有关。

最后，它导入了 ocprom 和 prometheus 库，这两个库与 Prometheus 统计系统有关。

总结起来，这段代码可能是一个用于从网络请求和响应中收集统计数据并将其存储在 Prometheus 中的库。


```
package corehttp

import (
	"net"
	"net/http"
	"time"

	core "github.com/ipfs/kubo/core"
	"go.opencensus.io/stats/view"
	"go.opencensus.io/zpages"

	ocprom "contrib.go.opencensus.io/exporter/prometheus"
	prometheus "github.com/prometheus/client_golang/prometheus"
	promhttp "github.com/prometheus/client_golang/prometheus/promhttp"
)

```

这段代码定义了两个函数，一个是`MetricsScrapingOption`，另一个是`MetricsOpenCensorationCollectionOption`。它们都接受一个参数`path`，用于指定要 scrap的 metrics 文件或 Endpoint。

`MetricsScrapingOption`函数，当接收到一个 `MetricsScrapingOption` 请求时，创建一个 HTTP  serveMux 函数，并在该函数中添加一个到 metrics 文件的 scraping Endpoint。这个 endpoint 使用的 Prometheus 的 Endpoint 是用 `metrics` 存储桶来提供的。

`MetricsOpenCensorationCollectionOption`函数，当接收到一个 `MetricsOpenCensorationCollectionOption` 请求时，创建一个 HTTP serveMux 函数，并在该函数中添加一个到 metrics 文件的 scraping Endpoint。这个 endpoint 使用的 OpenCounters 存储桶来提供 metrics。函数内部还会注册 OpenCounters 并用 `debugz` 存储所有 metrics。最后，函数还会处理 `/debug/metrics/oc/debugz`。


```
// MetricsScrapingOption adds the scraping endpoint which Prometheus uses to fetch metrics.
func MetricsScrapingOption(path string) ServeOption {
	return func(n *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
		mux.Handle(path, promhttp.HandlerFor(prometheus.DefaultGatherer, promhttp.HandlerOpts{}))
		return mux, nil
	}
}

// This adds collection of OpenCensus metrics
func MetricsOpenCensusCollectionOption() ServeOption {
	return func(_ *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
		log.Info("Init OpenCensus")

		promRegistry := prometheus.NewRegistry()
		pe, err := ocprom.NewExporter(ocprom.Options{
			Namespace: "ipfs_oc",
			Registry:  promRegistry,
			OnError: func(err error) {
				log.Errorw("OC ERROR", "error", err)
			},
		})
		if err != nil {
			return nil, err
		}

		// register prometheus with opencensus
		view.RegisterExporter(pe)
		view.SetReportingPeriod(2 * time.Second)

		// Construct the mux
		zpages.Handle(mux, "/debug/metrics/oc/debugz")
		mux.Handle("/debug/metrics/oc", pe)

		return mux, nil
	}
}

```

这段代码定义了一个名为 "MetricsOpenCitnessDefaultPrometheusRegistry" 的函数，该函数用于注册 MetricsOpenCillery 中 default Prometheus 注册表，将其注册为 OpenC天道指标的 exporter。

具体来说，这段代码实现以下几个步骤：

1. 注册 MetricsOpenC烟火棒（registry）作为 exporter，将其注册为 OpenC天道指标的 exporter。

2. 使用 ocprom.NewExporter 函数创建一个新的 exporter 实例，并使用该实例注册 Prometheus 作为 OpenC天道指标的 exporter。

3. 使用 view.RegisterExporter 函数将注册的 exporter 注册到 OpenC天道指标中。

4. 返回 MetricsOpenC烟火棒 和注册成功后可能产生的错误。


```
// MetricsOpenCensusDefaultPrometheusRegistry registers the default prometheus
// registry as an exporter to OpenCensus metrics. This means that OpenCensus
// metrics will show up in the prometheus metrics endpoint
func MetricsOpenCensusDefaultPrometheusRegistry() ServeOption {
	return func(_ *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
		log.Info("Init OpenCensus with default prometheus registry")

		pe, err := ocprom.NewExporter(ocprom.Options{
			Registry: prometheus.DefaultRegisterer.(*prometheus.Registry),
			OnError: func(err error) {
				log.Errorw("OC default registry ERROR", "error", err)
			},
		})
		if err != nil {
			return nil, err
		}

		// register prometheus with opencensus
		view.RegisterExporter(pe)

		return mux, nil
	}
}

```

This appears to be a function that creates a Prometheus Summer肌理。SummaryVec is a type of Prometheus vector that stores the sum of a set of Prometheus metrics.

The function takes an Options struct as its input and returns a tuple of two values: the PrometheusSummaryVec and an error. If an error is returned, it is wrapped in a already registered error and the resulting vector is set to be a default vector.

The function first sets the name of the Summer肌理 to "request\_size\_bytes" and a brief description. It then creates a new PrometheusSummaryVec by calling the NewSummaryVec function on the Options struct and storing the result in a variable called reqSz.

If an error is encountered, it is likely that the Summer肌理 has already been registered, in which case the existing Vector is used, otherwise an error will be thrown.

Finally, the function constructs the mux (i.e. thePrometheusSummaryVec) by setting the Handler to the newPrometheusSummaryVec and instrumenting it with the appropriate metrics. This is done using PrometheusInstrumentedHandler, which instrument the response and counter metrics, and instrumenting the request metrics using PrometheusInstrumentedHandlerRequestSize. It will return the handle and a error.


```
// MetricsCollectionOption adds collection of net/http-related metrics.
func MetricsCollectionOption(handlerName string) ServeOption {
	return func(_ *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
		// Adapted from github.com/prometheus/client_golang/prometheus/http.go
		// Work around https://github.com/prometheus/client_golang/pull/311
		opts := prometheus.SummaryOpts{
			Namespace:   "ipfs",
			Subsystem:   "http",
			ConstLabels: prometheus.Labels{"handler": handlerName},
			Objectives:  map[float64]float64{0.5: 0.05, 0.9: 0.01, 0.99: 0.001},
		}

		reqCnt := prometheus.NewCounterVec(
			prometheus.CounterOpts{
				Namespace:   opts.Namespace,
				Subsystem:   opts.Subsystem,
				Name:        "requests_total",
				Help:        "Total number of HTTP requests made.",
				ConstLabels: opts.ConstLabels,
			},
			[]string{"method", "code"},
		)
		if err := prometheus.Register(reqCnt); err != nil {
			if are, ok := err.(prometheus.AlreadyRegisteredError); ok {
				reqCnt = are.ExistingCollector.(*prometheus.CounterVec)
			} else {
				return nil, err
			}
		}

		opts.Name = "request_duration_seconds"
		opts.Help = "The HTTP request latencies in seconds."
		reqDur := prometheus.NewSummaryVec(opts, nil)
		if err := prometheus.Register(reqDur); err != nil {
			if are, ok := err.(prometheus.AlreadyRegisteredError); ok {
				reqDur = are.ExistingCollector.(*prometheus.SummaryVec)
			} else {
				return nil, err
			}
		}

		opts.Name = "request_size_bytes"
		opts.Help = "The HTTP request sizes in bytes."
		reqSz := prometheus.NewSummaryVec(opts, nil)
		if err := prometheus.Register(reqSz); err != nil {
			if are, ok := err.(prometheus.AlreadyRegisteredError); ok {
				reqSz = are.ExistingCollector.(*prometheus.SummaryVec)
			} else {
				return nil, err
			}
		}

		opts.Name = "response_size_bytes"
		opts.Help = "The HTTP response sizes in bytes."
		resSz := prometheus.NewSummaryVec(opts, nil)
		if err := prometheus.Register(resSz); err != nil {
			if are, ok := err.(prometheus.AlreadyRegisteredError); ok {
				resSz = are.ExistingCollector.(*prometheus.SummaryVec)
			} else {
				return nil, err
			}
		}

		// Construct the mux
		childMux := http.NewServeMux()
		var promMux http.Handler = childMux
		promMux = promhttp.InstrumentHandlerResponseSize(resSz, promMux)
		promMux = promhttp.InstrumentHandlerRequestSize(reqSz, promMux)
		promMux = promhttp.InstrumentHandlerDuration(reqDur, promMux)
		promMux = promhttp.InstrumentHandlerCounter(reqCnt, promMux)
		mux.Handle("/", promMux)

		return childMux, nil
	}
}

```

该代码定义了一个名为`peersTotalMetric`的`Desc`指标，用于描述IPFS网络中连接的端点的数量。指标的计算基于一个名为`transport`的transport类型，意味着计算连接的TCP或UDP套接字。指标定义中包括指标名称和指标族，以及指标的标签。最后，将`nil`作为指标的默认值，以便在将来添加指标时可以将其设置为默认值。

该代码还定义了一个名为`IpfsNodeCollector`的结构体，该结构体代表一个IPFS节点收集器。该结构体包含一个名为`Node`的`IpfsNode`类型的成员变量，该成员变量代表一个已经解析并连接到IPFS网络的节点。

该结构体中的`Describe`方法与指标的`Desc`方法类似，因此可以将`peersTotalMetric`作为该方法的指标，并将`ch`作为方法接受者的通道。该方法将`peersTotalMetric`发送到接受者`ch`中，以便将其添加到`peersTotalMetric`的统计数据中。


```
var peersTotalMetric = prometheus.NewDesc(
	prometheus.BuildFQName("ipfs", "p2p", "peers_total"),
	"Number of connected peers",
	[]string{"transport"},
	nil,
)

type IpfsNodeCollector struct {
	Node *core.IpfsNode
}

func (IpfsNodeCollector) Describe(ch chan<- *prometheus.Desc) {
	ch <- peersTotalMetric
}

```

该函数实现了IpfsNodeCollector的Collect方法，该函数接收一个channel类型的参数c，该参数用于输出经过的peersTotalMetric指标。

函数遍历了c中的所有peersTotalValues()方法返回的值，并为每个值创建了一个prometheus.Metric实例，并将其添加到channel中。

函数PeersTotalValues()返回一个包含所有经过的peersTotalMetric指标的map，其中每个键都是一个string类型的metric名称，其值为float64类型的指标值。这个map是通过对c中的peersTotalValues()方法返回的值进行迭代获得的。

函数的作用是输出经过的peersTotalMetric指标，并将其存储在一个map中，以便将其返回。


```
func (c IpfsNodeCollector) Collect(ch chan<- prometheus.Metric) {
	for tr, val := range c.PeersTotalValues() {
		ch <- prometheus.MustNewConstMetric(
			peersTotalMetric,
			prometheus.GaugeValue,
			val,
			tr,
		)
	}
}

func (c IpfsNodeCollector) PeersTotalValues() map[string]float64 {
	vals := make(map[string]float64)
	if c.Node.PeerHost == nil {
		return vals
	}
	for _, peerID := range c.Node.PeerHost.Network().Peers() {
		// Each peer may have more than one connection (see for an explanation
		// https://github.com/libp2p/go-libp2p-swarm/commit/0538806), so we grab
		// only one, the first (an arbitrary and non-deterministic choice), which
		// according to ConnsToPeer is the oldest connection in the list
		// (https://github.com/libp2p/go-libp2p-swarm/blob/v0.2.6/swarm.go#L362-L364).
		conns := c.Node.PeerHost.Network().ConnsToPeer(peerID)
		if len(conns) == 0 {
			continue
		}
		tr := ""
		for _, proto := range conns[0].RemoteMultiaddr().Protocols() {
			tr = tr + "/" + proto.Name
		}
		vals[tr] = vals[tr] + 1
	}
	return vals
}

```