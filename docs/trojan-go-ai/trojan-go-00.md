# trojan-go源码解析 0

# `main.go`

这段代码是一个 Trojan Go 框架的主程序。它使用 Go 标准库中的 "flag" 包来解析命令行选项。

程序的主要功能是接收用户输入的选项，并调用一个名为 "h" 的选项处理函数。如果选项处理函数出现错误，程序会打印错误并退出。如果选项处理函数正常退出，那么程序就会继续等待下一个选项。

具体来说，这段代码可以分为以下几个步骤：

1. 解析命令行选项，使用 "flag.Parse()" 函数从命令行输入中获取选项；
2. 如果解析选项失败，使用 "log.Fatal()" 函数打印错误并退出；
3. 如果解析选项成功，使用 "h.Handle()" 函数调用选项处理函数；
4. 如果选项处理函数出现错误，使用 "log.Fatal()" 函数打印错误并退出；
5. 如果选项处理函数正常退出，那么程序就会继续等待下一个选项。

选项处理函数的实现可能会因具体需求而异，但是这道题目没有提供具体的函数实现，所以无法提供更具体的解释。


```go
package main

import (
	"flag"

	_ "github.com/p4gefau1t/trojan-go/component"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/option"
)

func main() {
	flag.Parse()
	for {
		h, err := option.PopOptionHandler()
		if err != nil {
			log.Fatal("invalid options")
		}
		err = h.Handle()
		if err == nil {
			break
		}
	}
}

```

# Trojan-Go [![Go Report Card](https://goreportcard.com/badge/github.com/p4gefau1t/trojan-go)](https://goreportcard.com/report/github.com/p4gefau1t/trojan-go) [![Downloads](https://img.shields.io/github/downloads/p4gefau1t/trojan-go/total?label=downloads&logo=github&style=flat-square)](https://img.shields.io/github/downloads/p4gefau1t/trojan-go/total?label=downloads&logo=github&style=flat-square)

使用 Go 实现的完整 Trojan 代理，兼容原版 Trojan 协议及配置文件格式。安全、高效、轻巧、易用。

Trojan-Go 支持[多路复用](#多路复用)提升并发性能；使用[路由模块](#路由模块)实现国内外分流；支持 [CDN 流量中转](#Websocket)(基于 WebSocket over TLS)；支持使用 AEAD 对 Trojan 流量进行[二次加密](#aead-加密)(基于 Shadowsocks AEAD)；支持可插拔的[传输层插件](#传输层插件)，允许替换 TLS，使用其他加密隧道传输 Trojan 协议流量。

预编译二进制可执行文件可在 [Release 页面](https://github.com/p4gefau1t/trojan-go/releases)下载。解压后即可直接运行，无其他组件依赖。

如遇到配置和使用问题、发现 bug，或是有更好的想法，欢迎加入 [Telegram 交流反馈群](https://t.me/trojan_go_chat)。

## 简介

**完整介绍和配置教程，参见 [Trojan-Go 文档](https://p4gefau1t.github.io/trojan-go)。**

Trojan-Go 兼容原版 Trojan 的绝大多数功能，包括但不限于：

- TLS 隧道传输
- UDP 代理
- 透明代理 (NAT 模式，iptables 设置参考[这里](https://github.com/shadowsocks/shadowsocks-libev/tree/v3.3.1#transparent-proxy))
- 对抗 GFW 被动检测 / 主动检测的机制
- MySQL 数据持久化方案
- MySQL 用户权限认证
- 用户流量统计和配额限制

同时，Trojan-Go 还扩展实现了更多高效易用的功能特性：

- 便于快速部署的「简易模式」
- Socks5 / HTTP 代理自动适配
- 基于 TProxy 的透明代理（TCP / UDP）
- 全平台支持，无特殊依赖
- 基于多路复用（smux）降低延迟，提升并发性能
- 自定义路由模块，可实现国内外分流 / 广告屏蔽等功能
- Websocket 传输支持，以实现 CDN 流量中转（基于 WebSocket over TLS）和对抗 GFW 中间人攻击
- TLS 指纹伪造，以对抗 GFW 针对 TLS Client Hello 的特征识别
- 基于 gRPC 的 API 支持，以实现用户管理和速度限制等
- 可插拔传输层，可将 TLS 替换为其他协议或明文传输，同时有完整的 Shadowsocks 混淆插件支持
- 支持对用户更友好的 YAML 配置文件格式

## 图形界面客户端

Trojan-Go 服务端兼容所有原 Trojan 客户端，如 Igniter、ShadowRocket 等。以下是支持 Trojan-Go 扩展特性（Websocket / Mux 等）的客户端：

- [Qv2ray](https://github.com/Qv2ray/Qv2ray)：跨平台客户端，支持 Windows / macOS / Linux，使用 Trojan-Go 核心，支持所有 Trojan-Go 扩展特性。
- [Igniter-Go](https://github.com/p4gefau1t/trojan-go-android)：Android 客户端，Fork 自 Igniter，将 Igniter 核心替换为 Trojan-Go 并做了一定修改，支持所有 Trojan-Go 扩展特性。

## 使用方法

1. 快速启动服务端和客户端（简易模式）

    - 服务端

        ```goshell
        sudo ./trojan-go -server -remote 127.0.0.1:80 -local 0.0.0.0:443 -key ./your_key.key -cert ./your_cert.crt -password your_password
        ```

    - 客户端

        ```goshell
        ./trojan-go -client -remote example.com:443 -local 127.0.0.1:1080 -password your_password
        ```

2. 使用配置文件启动客户端 / 服务端 / 透明代理 / 中继（一般模式）

    ```goshell
    ./trojan-go -config config.json
    ```

3. 使用 URL 启动客户端（格式参见文档）

    ```goshell
    ./trojan-go -url 'trojan-go://password@cloudflare.com/?type=ws&path=%2Fpath&host=your-site.com'
    ```

4. 使用 Docker 部署

    ```goshell
    docker run \
        --name trojan-go \
        -d \
        -v /etc/trojan-go/:/etc/trojan-go \
        --network host \
        p4gefau1t/trojan-go
    ```

   或者

    ```goshell
    docker run \
        --name trojan-go \
        -d \
        -v /path/to/host/config:/path/in/container \
        --network host \
        p4gefau1t/trojan-go \
        /path/in/container/config.json
    ```

## 特性

一般情况下，Trojan-Go 和 Trojan 是互相兼容的，但一旦使用下面介绍的扩展特性（如多路复用、Websocket 等），则无法兼容。

### 移植性

编译得到的 Trojan-Go 单个可执行文件不依赖其他组件。同时，你可以很方便地编译（或交叉编译） Trojan-Go，然后在你的服务器、PC、树莓派，甚至路由器上部署；可以方便地使用 build tag 删减模块，以缩小可执行文件体积。

例如，交叉编译一个可在 mips 处理器、Linux 操作系统上运行的、只有客户端功能的 Trojan-Go，只需执行下面的命令，得到的可执行文件可以直接在目标平台运行：

```goshell
CGO_ENABLED=0 GOOS=linux GOARCH=mips go build -tags "client" -trimpath -ldflags "-s -w -buildid="
```

完整的 tag 说明参见 [Trojan-Go 文档](https://p4gefau1t.github.io/trojan-go)。

### 易用

配置文件格式与原版 Trojan 兼容，但做了大幅简化，未指定的字段会被赋予默认值，由此可以更方便地部署服务端和客户端。以下是一个简单例子，完整的配置文件可以参见[这里](https://p4gefau1t.github.io/trojan-go)。

服务端配置文件 `server.json`：

```gojson
{
  "run_type": "server",
  "local_addr": "0.0.0.0",
  "local_port": 443,
  "remote_addr": "127.0.0.1",
  "remote_port": 80,
  "password": ["your_awesome_password"],
  "ssl": {
    "cert": "your_cert.crt",
    "key": "your_key.key",
    "sni": "www.your-awesome-domain-name.com"
  }
}
```

客户端配置文件 `client.json`：

```gojson
{
  "run_type": "client",
  "local_addr": "127.0.0.1",
  "local_port": 1080,
  "remote_addr": "www.your-awesome-domain-name.com",
  "remote_port": 443,
  "password": ["your_awesome_password"]
}
```

可以使用更简明易读的 YAML 语法进行配置。以下是一个客户端的例子，与上面的 `client.json` 等价：

客户端配置文件 `client.yaml`：

```goyaml
run-type: client
local-addr: 127.0.0.1
local-port: 1080
remote-addr: www.your-awesome-domain_name.com
remote-port: 443
password:
  - your_awesome_password
```

### WebSocket

Trojan-Go 支持使用 TLS + Websocket 承载 Trojan 协议，使得利用 CDN 进行流量中转成为可能。

服务端和客户端配置文件中同时添加 `websocket` 选项即可启用 Websocket 支持，例如

```gojson
"websocket": {
    "enabled": true,
    "path": "/your-websocket-path",
    "hostname": "www.your-awesome-domain-name.com"
}
```

完整的选项说明参见 [Trojan-Go 文档](https://p4gefau1t.github.io/trojan-go)。

可以省略 `hostname`, 但服务端和客户端的 `path` 必须一致。服务端开启 Websocket 支持后，可以同时支持 Websocket 和一般 Trojan 流量。未配置 Websocket 选项的客户端依然可以正常使用。

由于 Trojan 并不支持 Websocket，因此，虽然开启了 Websocket 支持的 Trojan-Go 服务端可以兼容所有客户端，但如果要使用 Websocket 承载流量，请确保双方都使用 Trojan-Go。

### 多路复用

在很差的网络条件下，一次 TLS 握手可能会花费很多时间。Trojan-Go 支持多路复用（基于 [smux](https://github.com/xtaci/smux)），通过一条 TLS 隧道连接承载多条 TCP 连接的方式，减少 TCP 和 TLS 握手带来的延迟，以期提升高并发情景下的性能。

> 启用多路复用并不能提高测速得到的链路速度，但能降低延迟、提升大量并发请求时的网络体验，例如浏览含有大量图片的网页等。

你可以通过设置客户端的 `mux` 选项 `enabled` 字段启用它：

```gojson
"mux": {
    "enabled": true
}
```

只需开启客户端 mux 配置即可，服务端会自动检测是否启用多路复用并提供支持。完整的选项说明参见 [Trojan-Go 文档](https://p4gefau1t.github.io/trojan-go)。

### 路由模块

Trojan-Go 客户端内建一个简单实用的路由模块，以方便实现国内直连、海外代理等自定义路由功能。

路由策略有三种：

- `Proxy` 代理：将请求通过 TLS 隧道进行代理，由 Trojan 服务端与目的地址进行连接。
- `Bypass` 绕过：直接使用本地设备与目的地址进行连接。
- `Block` 封锁：不发送请求，直接关闭连接。

要激活路由模块，请在配置文件中添加 `router` 选项，并设置 `enabled` 字段为 `true`：

```gojson
"router": {
    "enabled": true,
    "bypass": [
        "geoip:cn",
        "geoip:private",
        "full:localhost"
    ],
    "block": [
        "cidr:192.168.1.1/24",
    ],
    "proxy": [
        "domain:google.com",
    ],
    "default_policy": "proxy"
}
```

完整的选项说明参见 [Trojan-Go 文档](https://p4gefau1t.github.io/trojan-go)。

### AEAD 加密

Trojan-Go 支持基于 Shadowsocks AEAD 对 Trojan 协议流量进行二次加密，以保证 Websocket 传输流量无法被不可信的 CDN 识别和审查：

```gojson
"shadowsocks": {
    "enabled": true,
    "password": "my-password"
}
```

如需开启，服务端和客户端必须同时开启并保证密码一致。

### 传输层插件

Trojan-Go 支持可插拔的传输层插件，并支持 Shadowsocks [SIP003](https://shadowsocks.org/en/wiki/Plugin.html) 标准的混淆插件。下面是使用 `v2ray-plugin` 的一个例子：

> **此配置并不安全，仅作为演示**

服务端配置：

```gojson
"transport_plugin": {
    "enabled": true,
    "type": "shadowsocks",
    "command": "./v2ray-plugin",
    "arg": ["-server", "-host", "www.baidu.com"]
}
```

客户端配置：

```gojson
"transport_plugin": {
    "enabled": true,
    "type": "shadowsocks",
    "command": "./v2ray-plugin",
    "arg": ["-host", "www.baidu.com"]
}
```

完整的选项说明参见 [Trojan-Go 文档](https://p4gefau1t.github.io/trojan-go)。

## 构建

> 请确保 Go 版本 >= 1.14

使用 `make` 进行编译：

```goshell
git clone https://github.com/p4gefau1t/trojan-go.git
cd trojan-go
make
make install #安装systemd服务等，可选
```

或者使用 Go 自行编译：

```goshell
go build -tags "full"
```

Go 支持通过设置环境变量进行交叉编译，例如：

编译适用于 64 位 Windows 操作系统的可执行文件：

```goshell
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build -tags "full"
```

编译适用于 Apple Silicon 的可执行文件：

```goshell
CGO_ENABLED=0 GOOS=macos GOARCH=arm64 go build -tags "full"
```

编译适用于 64 位 Linux 操作系统的可执行文件：

```goshell
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -tags "full"
```

## 致谢

- [Trojan](https://github.com/trojan-gfw/trojan)
- [V2Fly](https://github.com/v2fly)
- [utls](https://github.com/refraction-networking/utls)
- [smux](https://github.com/xtaci/smux)
- [go-tproxy](https://github.com/LiamHaworth/go-tproxy)

## Stargazers over time

[![Stargazers over time](https://starchart.cc/p4gefau1t/trojan-go.svg)](https://starchart.cc/p4gefau1t/trojan-go)


---
name: Bug 报告
about: 提交项目中存在的漏洞和问题
title: "[BUG]"
labels: ''
assignees: ''

---

- [ ] **我确定我已经尝试多次复现此次问题，并且将会提供涉及此问题的系统和网络环境，软件及其版本。**

我们建议您按照下方模板填写 Bug Report，以便我们收集更多的有效信息

## 简单描述这个 Bug

## 如何复现这个 Bug

在此描述复现这个Bug所需要的操作步骤

## 服务器和客户端环境信息

在此描述你的服务器和客户端所处的网络环境，系统架构，以及其他信息

## 服务端和客户端日志

粘贴**故障发生时**，服务端和客户端日志

## 服务端和客户端配置文件

可以复现该问题的客户端和服务端的完整配置（请隐去域名和IP等隐私信息）

## 服务端和客户端版本信息

请执行./trojan-go -version并将输出完整粘贴在此处

## 其他信息

你认为对我们修复bug有帮助的任何信息都可以在这里写出来


# `api/api.go`

这段代码定义了一个名为"api"的包，其中包含一个名为"Handler"的类型，以及一个名为"registerHandler"的函数。

"Handler"是一个函数类型，它接收一个上下文上下文和一個驗證器 Statistic.Authenticator，然后返回一个错误。

"registerHandler"函数接收一个名字和一個驗證器 Statistic.Authenticator，然后将驗證器注册到 Handlers map 上，使得以后所有使用 RegisterHandler 注册的 Handler 都会被使用这个验證器进行验证。

如果同时没有传入上下文上下文和驗證器，那么会自动将上下文上下文设置为当前上下文上下文，这样就可以将上下文相关的一些信息作为参数传递给 Handler。


```go
package api

import (
	"context"

	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/statistic"
)

type Handler func(ctx context.Context, auth statistic.Authenticator) error

var handlers = make(map[string]Handler)

func RegisterHandler(name string, handler Handler) {
	handlers[name] = handler
}

```

这段代码定义了一个名为 RunService 的函数，它接受一个上下文 context、一个名称 string 和一个身份验证器统计器Authenticator。它的作用是判断是否有一个预定义的 API 处理程序，如果找到了，就返回该程序对上下文质量和授权的统计结果。否则，返回一个表示失败错误的反向通道。

具体来说，这段代码实现了一个简单的逻辑：首先，它遍历一个名为 handlers 的映射，查找预定义的 API 处理程序。如果找到了，就调用该函数处理输入的上下文质量和授权；否则，输出一个表示失败错误的反向通道。在这个过程中，函数使用了开源的一些标准库函数，如 "context"、"diagnose" 和 "log"，以便更轻松地处理错误和返回信息。


```go
func RunService(ctx context.Context, name string, auth statistic.Authenticator) error {
	if h, ok := handlers[name]; ok {
		log.Debug("api handler found", name)
		return h(ctx, auth)
	}
	log.Debug("api handler not found", name)
	return nil
}

```

# `api/control/control.go`

这是一个 Go 语言编写的包控制工具，用于管理命令行工具 "trojan-go" 在集群中的操作。

具体来说，这个包控制工具实现了以下功能：

1. 通过 gRPC 实现了与 trojan-go 服务器的通信。
2. 解析 JSON 格式的配置文件，包括选项（option）的设置。
3. 支持命令行工具的选项，例如设置选项 "--ssl-verify" 的值，是否启用 SSL 验证。
4. 将服务器和客户端的通信结果输出到控制台。
5. 通过 JSON 配置文件来设置选项，例如设置服务器和客户端的地址、端口、认证信息等。


```go
package control

import (
	"context"
	"encoding/json"
	"flag"
	"fmt"
	"io"

	"google.golang.org/grpc"

	"github.com/p4gefau1t/trojan-go/api/service"
	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/option"
)

```

这是一个定义了一个名为`apiController`的结构体的代码。`apiController`是一个` struct`类型，其中包含以下字段：

- `address`字段，类型为`*string`(字符串类型)，表示API的地址，比如API的服务器地址。
- `key`字段，类型为`*string`(字符串类型)，表示API的密钥，用于身份验证。
- `hash`字段，类型为`*string`(字符串类型)，表示API的哈希值，通常用于缓存。
- `cert`字段，类型为`*string`(字符串类型)，表示API的证书，用于安全验证。
- `cmd`字段，类型为`*string`(字符串类型)，表示API的命令，可能的命令包括创建、删除、修改等。
- `password`字段，类型为`*string`(字符串类型)，表示API的密码，用于登录或验证。
- `add`字段，类型为`*bool`(布尔类型)，表示API的添加功能，如果为`true`，则允许添加新的资源或组件。
- `delete`字段，类型为`*bool`(布尔类型)，表示API的删除功能，如果为`true`，则允许删除资源或组件。
- `modify`字段，类型为`*bool`(布尔类型)，表示API的修改功能，如果为`true`，则允许修改资源或组件。
- `list`字段，类型为`*bool`(布尔类型)，表示API的列表功能，如果为`true`，则允许列出资源或组件。
- `uploadSpeedLimit`字段，类型为`*int`(整数类型)，表示API的上传速度限制。
- `downloadSpeedLimit`字段，类型为`*int`(整数类型)，表示API的下载速度限制。
- `ipLimit`字段，类型为`*int`(整数类型)，表示API的IP限制，用于防止分布式攻击。
- `ctx`字段，类型为`context.Context`，用于跨上下文。


```go
type apiController struct {
	address *string
	key     *string
	hash    *string
	cert    *string

	cmd                *string
	password           *string
	add                *bool
	delete             *bool
	modify             *bool
	list               *bool
	uploadSpeedLimit   *int
	downloadSpeedLimit *int
	ipLimit            *int
	ctx                context.Context
}

```

这段代码定义了一个名为 "apiController" 的函数接收者函数，返回值为 "api"。这个函数接收一个名为 "apiController" 的参数，并返回一个字符串类型的变量。

接下来的函数 "listUsers" 接收一个名为 "apiController" 的参数，以及一个名为 "apiClient" 的 service.TrojanServerServiceClient 类型的参数。这个函数使用 apiClient.ListUsers 方法来获取用户列表，并将返回的结果存储在名为 "result" 的字符数组中。

函数 "listUsers" 的实现比较简单，直接将返回结果逐个打印出来。这里需要注意的是，函数使用了 "stream.Recv()" 和 "stream.Send()" 方法来获取和发送请求。其中 "stream.Recv()" 方法从请求中读取数据，而 "stream.Send()" 方法将数据发送回服务器。


```go
func (apiController) Name() string {
	return "api"
}

func (o *apiController) listUsers(apiClient service.TrojanServerServiceClient) error {
	stream, err := apiClient.ListUsers(o.ctx, &service.ListUsersRequest{})
	if err != nil {
		return err
	}
	defer stream.CloseSend()
	result := []*service.ListUsersResponse{}
	for {
		resp, err := stream.Recv()
		if err != nil {
			if err == io.EOF {
				break
			}
			return err
		}
		result = append(result, resp)
	}
	data, err := json.Marshal(result)
	common.Must(err)
	fmt.Println(string(data))
	return nil
}

```

该函数名为 `getUsers`，它接收一个指向 `apiController` 类型和一个指向 `TrojanServerServiceClient` 类型的指针变量 `o`，并返回一个错误。

函数体内部，首先使用 `apiClient` 创建一个请求并获取用户数据，如果请求成功，则返回。然后将获取到的数据通过 `stream.Send` 方法发送给一个 `GetUsersRequest` 类型的参数，其中包含用户名和密码哈希。

如果发送数据失败，函数会返回一个错误。如果函数正常返回，它将返回一个空字符串。


```go
func (o *apiController) getUsers(apiClient service.TrojanServerServiceClient) error {
	stream, err := apiClient.GetUsers(o.ctx)
	if err != nil {
		return err
	}
	defer stream.CloseSend()
	err = stream.Send(&service.GetUsersRequest{
		User: &service.User{
			Password: *o.password,
			Hash:     *o.hash,
		},
	})
	if err != nil {
		return err
	}
	resp, err := stream.Recv()
	if err != nil {
		return err
	}
	data, err := json.Marshal(resp)
	common.Must(err)
	fmt.Print(string(data))
	return nil
}

```

这段代码是一个 Go 语言中的函数，名为 `setUsers`。它接收一个指向 `apiController` 类型对象的 `o` 参数，并使用它来设置用户。

函数首先通过调用 `apiClient.SetUsers` 函数来设置用户。如果设置用户时出现错误，函数将返回该错误。设置用户后，函数关闭了传入的 `stream` 并发送了新的 `SetUsersRequest` 函数。

`SetUsersRequest` 是一个包含设置用户信息的 `service.TrojanServerServiceClient` 类型的结构体。其中的 `Status` 字段指定了要设置的用户的状态，包括密码、哈希和限制（如上传和下载速度）。

如果 `o` 参数中的任何操作不是 `*o.add`、`*o.modify` 或 `*o.delete` 之一，函数将通过 `common.NewError` 函数返回一个错误。如果设置用户成功，函数将打印 "Done"。否则，函数将打印 "Failed: " 并返回错误信息。


```go
func (o *apiController) setUsers(apiClient service.TrojanServerServiceClient) error {
	stream, err := apiClient.SetUsers(o.ctx)
	if err != nil {
		return err
	}
	defer stream.CloseSend()

	req := &service.SetUsersRequest{
		Status: &service.UserStatus{
			User: &service.User{
				Password: *o.password,
				Hash:     *o.hash,
			},
			IpLimit: int32(*o.ipLimit),
			SpeedLimit: &service.Speed{
				UploadSpeed:   uint64(*o.uploadSpeedLimit),
				DownloadSpeed: uint64(*o.downloadSpeedLimit),
			},
		},
	}

	switch {
	case *o.add:
		req.Operation = service.SetUsersRequest_Add
	case *o.modify:
		req.Operation = service.SetUsersRequest_Modify
	case *o.delete:
		req.Operation = service.SetUsersRequest_Delete
	default:
		return common.NewError("Invalid operation")
	}

	err = stream.Send(req)
	if err != nil {
		return err
	}
	resp, err := stream.Recv()
	if err != nil {
		return err
	}
	if resp.Success {
		fmt.Println("Done")
	} else {
		fmt.Println("Failed: " + resp.Info)
	}
	return nil
}

```

这段代码定义了一个名为 `Handle` 的函数，属于一个名为 `apiController` 的类。

该函数接收一个名为 `*o` 的整数指针，代表一个命令对象。函数的作用是在调用命令对象中的 `handle` 方法时，处理命令对象中的错误。

函数首先检查 `*o.cmd` 是否为空字符串，如果是，函数将返回一个名为 ` common.NewError` 的函数并传入字符串 `""`。否则，函数使用 `grpc.Dial` 函数建立一个安全连接并关闭该连接，然后使用 `service.NewTrojanServerServiceClient` 函数创建一个客户端并调用 `apiClient.handle` 方法。如果命令对象中的 `cmd` 不是 "list" 或 "get"，函数将调用 `apiClient.handle` 方法并处理返回的错误。否则，函数将返回 `nil`。

函数的逻辑总结为：

1. 如果 `*o.cmd` 为空字符串，则返回一个名为 `common.NewError` 的函数并传入字符串 ""。

2. 否则，使用 `grpc.Dial` 函数建立一个安全连接并关闭该连接，然后使用 `service.NewTrojanServerServiceClient` 函数创建一个客户端并调用 `apiClient.handle` 方法。

3. 如果 `*o.cmd` 是 "list" 或 "get"，则函数将调用 `apiClient.handle` 方法并处理返回的错误。


```go
func (o *apiController) Handle() error {
	if *o.cmd == "" {
		return common.NewError("")
	}
	conn, err := grpc.Dial(*o.address, grpc.WithInsecure())
	if err != nil {
		log.Error(err)
		return nil
	}
	defer conn.Close()
	apiClient := service.NewTrojanServerServiceClient(conn)
	switch *o.cmd {
	case "list":
		err := o.listUsers(apiClient)
		if err != nil {
			log.Error(err)
		}
	case "get":
		err := o.getUsers(apiClient)
		if err != nil {
			log.Error(err)
		}
	case "set":
		err := o.setUsers(apiClient)
		if err != nil {
			log.Error(err)
		}
	default:
		log.Error("unknown command " + *o.cmd)
	}
	return nil
}

```

该代码定义了一个名为“Priority”的函数，接受一个名为“apiController”的参数。函数返回一个整数类型的值，代表优先级。

该函数使用了名为“option”的变量，用于存储API服务器的配置信息。在函数初始化时，option变量注册了一个名为“apiController”的函数，并将其作为参数传递给该函数。根据函数定义，apiController函数需要返回一个字符串类型的参数，其中包含连接到Trojan-Go API服务器的命令和API服务器的地址等信息。这些信息都是通过option变量传递给apiController函数的。


```go
func (o *apiController) Priority() int {
	return 50
}

func init() {
	option.RegisterHandler(&apiController{
		cmd:                flag.String("api", "", "Connect to a Trojan-Go API service. \"-api add/get/list\""),
		address:            flag.String("api-addr", "127.0.0.1:10000", "Address of Trojan-Go API service"),
		password:           flag.String("target-password", "", "Password of the target user"),
		hash:               flag.String("target-hash", "", "Hash of the target user"),
		add:                flag.Bool("add-profile", false, "Add a new profile with API"),
		delete:             flag.Bool("delete-profile", false, "Delete an existing profile with API"),
		modify:             flag.Bool("modify-profile", false, "Modify an existing profile with API"),
		uploadSpeedLimit:   flag.Int("upload-speed-limit", 0, "Limit the upload speed with API"),
		downloadSpeedLimit: flag.Int("download-speed-limit", 0, "Limit the download speed with API"),
		ipLimit:            flag.Int("ip-limit", 0, "Limit the number of IP with API"),
		ctx:                context.Background(),
	})
}

```

# `api/service/api.pb.go`

这段代码是一个 Go 语言的服务层代码，定义了一个名为 "service" 的包。它主要使用了 Google 的 protoc-gen-go 工具，根据 API 协议生成了一些 Go 语言的接口和依赖。以下是这段代码的作用和涉及的主要组件：

1. 引入了来自 protoc-gen-go 的版本信息，表明此代码是使用 Go 1.27.1 版本构建的。

2. 引入了 protoc 的版本信息，表明此代码是使用 Go 3.17.3 版本构建的。

3. 通过 import 语句引入了需要用到的反射、接口和 sync 包。

4. 在包的定义中，定义了 "api" 包的接口类型以及该接口的实现，这通常是生成接口类型的依赖关系。

5. 通过反射 package 的 reflect.mu_姓氏成员，实现了从 "api.proto" 文件中读取实体的功能。

6. 通过同步 package 的 sync.Once，实现了一个全局的同步原语，确保同一时间只有一个实例的创建。

7. 通过 protoreflect package 的 protoreflect.Timestamp，实现了从 "api.proto" 文件中读取时间戳的功能。

8. 通过 protected 字段实现了接口中可观测到的部分，也就是实现了 "api.proto" 中定义的接口。


```go
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
// 	protoc-gen-go v1.27.1
// 	protoc        v3.17.3
// source: api.proto

package service

import (
	protoreflect "google.golang.org/protobuf/reflect/protoreflect"
	protoimpl "google.golang.org/protobuf/runtime/protoimpl"
	reflect "reflect"
	sync "sync"
)

```

这段代码定义了一个名为 `SetUsersRequest_Operation` 的枚举类型，用于定义操作类型。枚举类型使用了一个名为 `_` 的保留字作为下划线，这表示它是一个自定义类型。

接着，代码定义了一个名为 `SetUsersRequest_Operation` 的常量类型，它使用了 `int32` 作为其整型表示。

最后，代码使用 `const` 关键字定义了两个表达式，它们通过 `protoimpl.EnforceVersion` 函数进行了版本检查。这些表达式的值确保了运行时和 protobuf 实现的版本与接口的版本兼容。具体来说，第一个表达式的值为 `20 - protoimpl.MinVersion`，第二个表达式的值为 `protoimpl.MaxVersion - 20`。这些表达式的含义是，如果运行时的版本过低，或者 protobuf 实现的版本过低，代码会抛出警告。


```go
const (
	// Verify that this generated code is sufficiently up-to-date.
	_ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
	// Verify that runtime/protoimpl is sufficiently up-to-date.
	_ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

type SetUsersRequest_Operation int32

const (
	SetUsersRequest_Add    SetUsersRequest_Operation = 0
	SetUsersRequest_Delete SetUsersRequest_Operation = 1
	SetUsersRequest_Modify SetUsersRequest_Operation = 2
)

```

这段代码定义了一个枚举类型变量 `SetUsersRequest_Operation`，包含了 `Add`, `Delete`, `Modify` 三种操作。

同时，还定义了一个包含这些操作名称和对应枚举值的字典类型变量 `SetUsersRequest_Operation_name`，以及一个包含这些枚举值和对应操作名称的字典类型变量 `SetUsersRequest_Operation_value`。

最后，定义了一个名为 `x` 的变量，其类型为 `SetUsersRequest_Operation` 类型，并且通过 `x.Encode()` 方法将 `x` 的值转换为对应的枚举值，返回一个指向 `SetUsersRequest_Operation` 类型的引用，即 `x` 的对象。


```go
// Enum value maps for SetUsersRequest_Operation.
var (
	SetUsersRequest_Operation_name = map[int32]string{
		0: "Add",
		1: "Delete",
		2: "Modify",
	}
	SetUsersRequest_Operation_value = map[string]int32{
		"Add":    0,
		"Delete": 1,
		"Modify": 2,
	}
)

func (x SetUsersRequest_Operation) Enum() *SetUsersRequest_Operation {
	p := new(SetUsersRequest_Operation)
	*p = x
	return p
}

```

这段代码定义了一个名为func的函数，它接受一个名为x的参数并返回一个字符串类型的函数值。

函数的实现包括以下步骤：

1. 首先定义了函数的一个字符串类型变量，它将从函数参数x中获取该参数的值。

2. 然后定义了一个名为descriptor的函数，它将从函数参数x中获取的值对应的字符串类型，并将其存储到一个名为descriptor的函数指针中，函数指针使用protoreflect.EnumNumber类型存储。

3. 接着定义了一个名为type的函数，它将从函数参数x中获取的值对应的字符串类型，并将其存储到一个名为type的函数指针中，函数指针使用protoreflect.EnumType类型存储。

4. 然后定义了一个名为number的函数，它将从函数参数x中获取的值对应的字符串类型，并将其存储到一个名为number的函数指针中，函数指针使用protoreflect.EnumNumber类型存储。

这段代码的作用是定义了一个字符串类型的函数，该函数可以接收一个名为x的参数，并返回一个字符串类型的函数值。函数的值将来自一个名为file_api_proto_enumTypes的接口，该接口定义了多个枚举类型。


```go
func (x SetUsersRequest_Operation) String() string {
	return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (SetUsersRequest_Operation) Descriptor() protoreflect.EnumDescriptor {
	return file_api_proto_enumTypes[0].Descriptor()
}

func (SetUsersRequest_Operation) Type() protoreflect.EnumType {
	return &file_api_proto_enumTypes[0]
}

func (x SetUsersRequest_Operation) Number() protoreflect.EnumNumber {
	return protoreflect.EnumNumber(x)
}

```

这段代码定义了一个名为Traffic的结构体类型，用于表示数据传输过程中的流量信息。

Traffic结构体包含以下字段：

* upload_traffic：上传流量，是一个uint64类型的字段。
* download_traffic：下载流量，是一个uint64类型的字段。

在Reset()函数中，所有字段都初始化为Traffic{}，即它们的值都为0。此外，如果启用了负号重置，那么会强制重置所有字段。

同时，代码中还包含一个使用SetUsersRequest_Operation.Descriptor作为De空函数的依赖，这个依赖在代码开头已经定义了，作用是返回De空函数的接口类型，即文件_api_proto_rawDescGZIP()和[]int类型的函数指针。


```go
// Deprecated: Use SetUsersRequest_Operation.Descriptor instead.
func (SetUsersRequest_Operation) EnumDescriptor() ([]byte, []int) {
	return file_api_proto_rawDescGZIP(), []int{10, 0}
}

type Traffic struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	UploadTraffic   uint64 `protobuf:"varint,1,opt,name=upload_traffic,json=uploadTraffic,proto3" json:"upload_traffic,omitempty"`
	DownloadTraffic uint64 `protobuf:"varint,2,opt,name=download_traffic,json=downloadTraffic,proto3" json:"download_traffic,omitempty"`
}

func (x *Traffic) Reset() {
	*x = Traffic{}
	if protoimpl.UnsafeEnabled {
		mi := &file_api_proto_msgTypes[0]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

```

这段代码定义了一个名为Traffic的结构的体，该结构体包含一个指向Traffic类型的指针变量x。

该代码定义了一个名为func的函数，该函数接收一个Traffic类型的参数x，并返回一个字符串类型的值，该值表示一个 X 类型（未知的数据类型）的内部实现的消息的JSON字符串表示。

该代码定义了一个名为func的函数，该函数接收一个Traffic类型的参数x，并返回一个空的空的结构体类型的指针，该指针指向一个 X 类型（未知的数据类型）的内部实现的消息的JSON字符串表示。

该代码定义了一个名为func的函数，该函数接收一个Traffic类型的参数x，并返回一个空的空的结构体类型的指针，该指针指向一个 X 类型（未知的数据类型）的内部实现的消息的JSON字符串表示。

该代码定义了一个名为Traffic的函数，该函数接收一个Traffic类型的参数x，并返回一个空的空的结构体类型的指针，该指针指向一个 X 类型（未知的数据类型）的内部实现的消息的JSON字符串表示。


```go
func (x *Traffic) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*Traffic) ProtoMessage() {}

func (x *Traffic) ProtoReflect() protoreflect.Message {
	mi := &file_api_proto_msgTypes[0]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

```

此代码是一个名为Traffic的接口，其实现了Go语言中的一个名为traffic的类型。下面是对接口方法的实现：

1. Descriptor()函数返回Traffic的描述符，其中包括了该接口实现的两个方法：GetUploadTraffic和GetDownloadTraffic的函数指针。函数返回的内容是一个字节切片和两个整数，第一个字节切片的内容是该接口实现的文件描述符，第二个整数的内容是该接口实现的文件描述符的类型。

2. GetUploadTraffic函数接收一个Traffic类型的变量，并且不检查该变量是否为空。如果该变量不为空，则调用该接口的GetUploadTraffic方法，并且返回该方法的返回值(uint64类型的变量)。

3. GetDownloadTraffic函数与GetUploadTraffic类似，只是将GetUploadTraffic方法的返回值取反(即uint64类型的变量转换为uint64类型)。

由于此接口已经过时，建议使用Go语言中提供的Traffic.proto文件进行使用，该文件中包含了更现代和安全的接口实现。


```go
// Deprecated: Use Traffic.ProtoReflect.Descriptor instead.
func (*Traffic) Descriptor() ([]byte, []int) {
	return file_api_proto_rawDescGZIP(), []int{0}
}

func (x *Traffic) GetUploadTraffic() uint64 {
	if x != nil {
		return x.UploadTraffic
	}
	return 0
}

func (x *Traffic) GetDownloadTraffic() uint64 {
	if x != nil {
		return x.DownloadTraffic
	}
	return 0
}

```

这段代码定义了一个名为 "Speed" 的结构体，它包含三个字段：upload_speed、download_speed 和 unknown_fields。

upload_speed 和 download_speed 字段是速度类型的实例化字段，它们分别表示上传和下载的速度。unknown_fields 字段是一个保留字段，它是一个 struct 类型的 unknown_fields 字段的包装，可以用来将 unknown_fields 字段映射到 json 字段上。

函数 Reset() 是这个结构体的一个方法，它重置了 speed 对象的内部状态，然后可能抛出了一个前驱对象（可能是一个已知的 struct 类型）。函数 Reset() 的实现包括两个步骤：

1. 通过传入一个空 Speed 对象，重置了 speed 对象的内部状态。
2. 如果使用了 unsafe_Enabled 环境，那么将未知字段（可能是已经定义好的 struct 类型）的值设置为 0。


```go
type Speed struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	UploadSpeed   uint64 `protobuf:"varint,1,opt,name=upload_speed,json=uploadSpeed,proto3" json:"upload_speed,omitempty"`
	DownloadSpeed uint64 `protobuf:"varint,2,opt,name=download_speed,json=downloadSpeed,proto3" json:"download_speed,omitempty"`
}

func (x *Speed) Reset() {
	*x = Speed{}
	if protoimpl.UnsafeEnabled {
		mi := &file_api_proto_msgTypes[1]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

```

这段代码定义了一个名为"Speed"的类型，该类型有一个名为"x"的 pointer字段。

该函数"func (x *Speed) String() string"返回一个字符串，该字符串表示"Speed"类型的实例"x"。

该函数"func (*Speed) ProtoMessage() {}"返回一个指向"Speed"类型实例的指针，该指针将调用"protoimpl.X.MessageStringOf"函数的实现。

该函数"func (x *Speed) ProtoReflect() protoreflect.Message"返回一个指向"Speed"类型实例的指针，该指针将调用"protoimpl.X.MessageStateOf"函数的实现。

函数"func (x *Speed) ProtoReflect() protoreflect.Message"的实现将在需要时返回一个指向"Speed"类型实例的指针，该指针将调用"file_api_proto_msgTypes[1]"的函数，该函数返回一个与"Speed"类型同名的类型，如果"Speed"类型同名，则返回该类型实例的"Message"类型实例。


```go
func (x *Speed) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*Speed) ProtoMessage() {}

func (x *Speed) ProtoReflect() protoreflect.Message {
	mi := &file_api_proto_msgTypes[1]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

```

这段代码定义了一个名为Speed的结构的体，该结构包含上传和下载网络速度。

首先，定义了一个指向名为Speed.Descriptor的函数指针，该函数返回一个字节切片和一个整数数组，表示文件的描述信息。函数指针使用file_api_proto_rawDescGZIP和[]int{1}作为其输入参数，并返回file_api_proto_rawDescGZIP和[]int{1}作为其输出参数，这表明函数指针Speed.Descriptor()函数将文件描述信息返回给函数调用者。

其次，定义了两个名为x.GetUploadSpeed和x.GetDownloadSpeed的函数，它们的实现都为uint64类型的函数，分别返回x的 upload 和 download 网络速度。函数实现中，首先判断x是否为空，如果是，则默认为0。否则，使用x.UploadSpeed和x.DownloadSpeed来获取上传和下载网络速度。

最后，定义了两个函数，一个名为file_api_proto_rawDescGZIP，该函数接受一个字节切片作为输入参数，并返回一个描述文件API协议的函数指针。


```go
// Deprecated: Use Speed.ProtoReflect.Descriptor instead.
func (*Speed) Descriptor() ([]byte, []int) {
	return file_api_proto_rawDescGZIP(), []int{1}
}

func (x *Speed) GetUploadSpeed() uint64 {
	if x != nil {
		return x.UploadSpeed
	}
	return 0
}

func (x *Speed) GetDownloadSpeed() uint64 {
	if x != nil {
		return x.DownloadSpeed
	}
	return 0
}

```

这段代码定义了一个名为 "User" 的结构体类型，该类型使用了 Protocol Buffers 的语法。

具体来说，这个结构体类型包含以下字段：

* "state"：一个名为 "MessageState" 的字段，使用了 Protocol Buffers 的 "messageState" 协议类型。
* "sizeCache"：一个名为 "SizeCache" 的字段，使用了 Protocol Buffers 的 "sizecache" 协议类型。
* "unknownFields"：一个名为 "UnknownFields" 的字段，使用了 Protocol Buffers 的 "unknownFields" 协议类型。
* "Password"：一个名为 "Password" 的字段，使用了 Protocol Buffers 的 "bytes" 协议类型，该字段存储了一个字符串类型的变量。
* "Hash"：一个名为 "Hash" 的字段，使用了 Protocol Buffers 的 "bytes" 协议类型，该字段存储了一个字符串类型的变量。

此外，还有一段代码 "Reset" 函数，该函数重置了 "User" 结构体的所有字段，使其回到了其原始值。如果使用了 "protoimpl.UnsafeEnabled" 选项，则会启用该函数。


```go
type User struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Password string `protobuf:"bytes,1,opt,name=password,proto3" json:"password,omitempty"`
	Hash     string `protobuf:"bytes,2,opt,name=hash,proto3" json:"hash,omitempty"`
}

func (x *User) Reset() {
	*x = User{}
	if protoimpl.UnsafeEnabled {
		mi := &file_api_proto_msgTypes[2]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

```

这段代码定义了两个函数，一个是将一个 `*User` 类型的对象作为参数，返回其 `String` 类型的字符串表示，另一个是在不输出 `*User` 的指针的情况下，返回其 `*User` 类型对应的 `FileAPI_proto.X` 接口的 `Message` 类型的 `*User` 类型指针。

第一个函数 `func (x *User) String() string` 接收一个 `*User` 类型的参数 `x`，并返回它类型的 `String` 类型。这个函数的作用是将 `x` 转换为字符串表示，然后返回这个字符串。

第二个函数 `func (*User) ProtoMessage() {}` 接收一个 `*User` 类型的参数 `*x`，并返回一个空的 `FileAPI_proto.X` 接口类型的指针。这个函数的作用是在不输出 `*User` 指针的情况下，返回 `*User` 类型对象的 `Message` 类型指针。

第三个函数 `func (x *User) ProtoReflect() protoreflect.Message` 接收一个 `*User` 类型的参数 `x`，并返回一个指向 `FileAPI_proto.X` 接口类型的 `protoreflect.Message` 类型的指针。这个函数的作用是在不输出 `*User` 指针的情况下，返回 `*User` 类型对象的 `Message` 类型指针，然后将这个指针转换为 `protoreflect.Message` 类型，返回这个类型。


```go
func (x *User) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*User) ProtoMessage() {}

func (x *User) ProtoReflect() protoreflect.Message {
	mi := &file_api_proto_msgTypes[2]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

```

这段代码定义了一个名为User的类型，其中包含两个方法：Descriptor和GetPassword。函数 Descriptor() 会返回一个字符串和两个整数，该类型是使用User.proto文件中定义的协议的描述符。GetPassword() 和 GetHash() 函数则在函数内部进行了实现，但是由于使用了 deprecated 的标签，因此建议使用 User.protoReflect.Descriptor 函数来获取该接口的描述符。


```go
// Deprecated: Use User.ProtoReflect.Descriptor instead.
func (*User) Descriptor() ([]byte, []int) {
	return file_api_proto_rawDescGZIP(), []int{2}
}

func (x *User) GetPassword() string {
	if x != nil {
		return x.Password
	}
	return ""
}

func (x *User) GetHash() string {
	if x != nil {
		return x.Hash
	}
	return ""
}

```

这段代码定义了一个名为 `UserStatus` 的结构体，它包含了一些 `protoimpl` 类型的成员变量和成员函数。下面是每个成员变量的简要解释：


- `state`：一个 `protoimpl.MessageState` 类型的变量，它用于表示该结构体的状态。
- `sizeCache`：一个 `protoimpl.SizeCache` 类型的变量，它用于缓存 `UserStatus` 实例的大小。
- `unknownFields`：一个 `protoimpl.UnknownFields` 类型的变量，它用于存储在该结构体中未定义的包分隔的额外字段。
- `User`：一个 `User` 类型的指针变量，它用于表示该结构体中的 `User` 成员。
- `TrafficTotal`：一个 `Traffic` 类型的指针变量，它用于表示该结构体中的 `TrafficTotal` 成员。
- `SpeedCurrent`：一个 `Speed` 类型的指针变量，它用于表示该结构体中的 `SpeedCurrent` 成员。
- `SpeedLimit`：一个 `Speed` 类型的指针变量，它用于表示该结构体中的 `SpeedLimit` 成员。
- `IpCurrent`：一个 `int32` 类型的成员变量，它用于表示该结构体中的 `IpCurrent` 成员。
- `IpLimit`：一个 `int32` 类型的成员变量，它用于表示该结构体中的 `IpLimit` 成员。

该 `UserStatus` 结构体的定义是为了在 `protobuf` 编码中保留一些未知字段，以便在将来的 `protobuf` 编码活动中为这些字段提供默认值。


```go
type UserStatus struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	User         *User    `protobuf:"bytes,1,opt,name=user,proto3" json:"user,omitempty"`
	TrafficTotal *Traffic `protobuf:"bytes,2,opt,name=traffic_total,json=trafficTotal,proto3" json:"traffic_total,omitempty"`
	SpeedCurrent *Speed   `protobuf:"bytes,3,opt,name=speed_current,json=speedCurrent,proto3" json:"speed_current,omitempty"`
	SpeedLimit   *Speed   `protobuf:"bytes,4,opt,name=speed_limit,json=speedLimit,proto3" json:"speed_limit,omitempty"`
	IpCurrent    int32    `protobuf:"varint,5,opt,name=ip_current,json=ipCurrent,proto3" json:"ip_current,omitempty"`
	IpLimit      int32    `protobuf:"varint,6,opt,name=ip_limit,json=ipLimit,proto3" json:"ip_limit,omitempty"`
}

func (x *UserStatus) Reset() {
	*x = UserStatus{}
	if protoimpl.UnsafeEnabled {
		mi := &file_api_proto_msgTypes[3]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

```

这段代码定义了两个函数，一个是将一个`UserStatus`类型的参数`x`转换为字符串，另一个是在定义和使用`UserStatus`类型时使用的`UserStatus`接口的`Message`字段。

具体来说，这段代码实现了一个`Message`类型的函数`func (x *UserStatus) String() string`，该函数将`x`的值作为参数并返回一个字符串，使用了`protoimpl.X.MessageStringOf`函数将`UserStatus`的值转换为`Message`类型，然后将`Message`类型转换为字符串。

另一个函数`func (*UserStatus) ProtoMessage() []byte`，返回一个`UserStatus`类型到`Message`类型映射的`Message`切片，使用了`protoimpl.UnsafeEnabled`注释启用`Message`类型字段。

第三个函数`func (x *UserStatus) ProtoReflect() protoreflect.Message`，返回一个`Message`类型，将`UserStatus`类型的`x`作为参数，返回`UserStatus`类型原始值`x`的`Message`类型。


```go
func (x *UserStatus) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*UserStatus) ProtoMessage() {}

func (x *UserStatus) ProtoReflect() protoreflect.Message {
	mi := &file_api_proto_msgTypes[3]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

```

此代码定义了一个名为UserStatus的接口，其包含了两个方法：Descriptor()和GetUser()。这两个方法都在尝试输出 deprecated 警告，建议使用UserStatus.proto中定义的Descriptor代替该接口。

在Descriptor()方法中，使用file_api_proto_rawDescGZIP()函数将一个名为UserStatus.proto的描述符字节切片返回，并将其作为第一个参数传递给第二个参数，第二个参数返回一个包含两个整数的切片，分别表示该UserStatus对象的静态字段数量和静态字段名称。

在GetUser()方法中，首先检查给定的引用是否为nil，如果是，则假设该引用指向的UserStatus对象仍然存在，然后返回该引用。否则，返回nil。

在GetTrafficTotal()方法中，与上面类似，首先检查给定的引用是否为nil，如果是，则假设该引用指向的UserStatus对象仍然存在，然后返回该引用。否则，返回nil。


```go
// Deprecated: Use UserStatus.ProtoReflect.Descriptor instead.
func (*UserStatus) Descriptor() ([]byte, []int) {
	return file_api_proto_rawDescGZIP(), []int{3}
}

func (x *UserStatus) GetUser() *User {
	if x != nil {
		return x.User
	}
	return nil
}

func (x *UserStatus) GetTrafficTotal() *Traffic {
	if x != nil {
		return x.TrafficTotal
	}
	return nil
}

```

这是一段使用指针（*UserStatus）访问UserStatus结构体中三个函数的代码。UserStatus结构体中包含三个成员变量：SpeedCurrent、SpeedLimit和IpCurrent。

1. func (x *UserStatus) GetSpeedCurrent() *Speed {
这段代码定义了一个名为"GetSpeedCurrent"的函数，接收一个名为"x"的指针参数，然后判断x是否为nil。如果是，就返回x的SpeedCurrent成员变量。否则，返回nil。

2. func (x *UserStatus) GetSpeedLimit() *Speed {
与上面函数类似，定义一个名为"GetSpeedLimit"的函数，接收一个名为"x"的指针参数，然后判断x是否为nil。如果是，就返回x的SpeedLimit成员变量。否则，返回nil。

3. func (x *UserStatus) GetIpCurrent() int32 {
这段代码定义了一个名为"GetIpCurrent"的函数，接收一个名为"x"的指针参数，然后判断x是否为nil。如果是，就返回x的IpCurrent成员变量。否则，返回0。


```go
func (x *UserStatus) GetSpeedCurrent() *Speed {
	if x != nil {
		return x.SpeedCurrent
	}
	return nil
}

func (x *UserStatus) GetSpeedLimit() *Speed {
	if x != nil {
		return x.SpeedLimit
	}
	return nil
}

func (x *UserStatus) GetIpCurrent() int32 {
	if x != nil {
		return x.IpCurrent
	}
	return 0
}

```

这段代码定义了一个名为GetTrafficRequest的结构体，用于表示请求数据。

该结构体有一个名为User的类型字段，该字段是一个UserStatus类型的指针。UserStatus是一个未定义的类型，但根据该名称可以推测其可能是一个与网络访问相关的状态。

此外，该结构体还有一个名为IpLimit的类型字段，其类型为int32。根据该名称可以推测该字段表示请求数据中的IP地址限制。

最后，该结构体还包含一个名为UnknownFields的类型字段，未定义其具体类型，但可以推测可能是一个包含未知量的字段。

整个函数名为GetTrafficRequest，该函数接收一个名为x的int32类型的参数，并返回一个int32类型的值。函数的条件判断中使用了if x != nil { return x.IpLimit; }，如果x不等于 nil，则返回x.IpLimit，否则返回0。

如果需要输出该函数的定义，可以将其展开为以下内容：

func (x *UserStatus) GetIpLimit() int32 {
   if x != nil {
       return x.IpLimit
   }
   return 0
}

type GetTrafficRequest struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	User *User `protobuf:"bytes,1,opt,name=user,proto3" json:"user,omitempty"`
}



```go
func (x *UserStatus) GetIpLimit() int32 {
	if x != nil {
		return x.IpLimit
	}
	return 0
}

type GetTrafficRequest struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	User *User `protobuf:"bytes,1,opt,name=user,proto3" json:"user,omitempty"`
}

```

这段代码定义了一个名为 GetTrafficRequest 的函数类型，并实现了两个函数方法：Reset 和 String。

Reset 函数的主要作用是重置 GetTrafficRequest 类型的实例，即将其恢复为一个新的空实例。具体实现包括以下几个步骤：

1. 将 x 类型设置为 GetTrafficRequest{} 类型，这会创建一个空 instance，相当于调用 x 的构造函数。
2. 如果定义了 protoimpl.UnsafeEnabled 标识，那么会执行以下操作：
  1. 获取文件_api_proto_msgTypes 类型为 4 的一个实例。
  2. 获取 x 的指向对象，并将其赋值给 mi。
  3. 将 mi 存储的消息信息存储为 4 字段的 GetTrafficRequest 类型实例。

如果 protoimpl.UnsafeEnabled 为 false，那么不会执行上述操作，但是仍然会创建一个空 instance。

String 函数的主要作用是将 GetTrafficRequest 类型实例转换为字符串字面值，并返回一个字符串。具体实现包括以下几个步骤：

1. 如果定义了 protoimpl.UnsafeEnabled 标识，那么会执行以下操作：
  1. 获取 x 的指向对象。
  2. 创建一个字符串字面值，并将其存储到 x。

如果 protoimpl.UnsafeEnabled 为 false，那么不会创建字符串字面值，也不会输出变量 x。


```go
func (x *GetTrafficRequest) Reset() {
	*x = GetTrafficRequest{}
	if protoimpl.UnsafeEnabled {
		mi := &file_api_proto_msgTypes[4]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *GetTrafficRequest) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*GetTrafficRequest) ProtoMessage() {}

```

这段代码定义了一个名为"func"的函数，接收一个名为"x"的整数类型的参数，并返回一个名为"GetTrafficRequest"的实参类型。

函数的作用是通过使用"file_api_proto_msgTypes"和"protoimpl.UnsafeEnabled"条件，检查传入的整数类型参数"x"是否为空指针。如果是，则执行以下操作：

1. 从"file_api_proto_msgTypes"数组中获取类型为"file_api_proto_msgTypes"的第二个元素，并将其赋值给整数类型变量"mi"。
2. 如果"protoimpl.UnsafeEnabled"为true，则执行以下操作：

  1. 从"x"整数类型的指针中获取消息类型的信息，并将其存储在整数类型变量"ms"中。
  2. 如果消息类型的信息存储在"file_api_proto_rawDescGZIP"数组中，则将整数类型变量"ms"存储的消息类型信息存储为"file_api_proto_rawDescGZIP"数组中的第一个元素。
  3. 返回整数类型变量"ms"。

如果"protoimpl.UnsafeEnabled"为false，则直接返回整数类型变量"mi"。

此外，函数还定义了一个名为"Descriptor"的函数，它接收一个整数类型的参数，并返回一个字节切片和一个整数类型的参数。函数的作用是在不使用"GetTrafficRequest.ProtoReflect.Descriptor"函数的情况下获取接口的描述符。


```go
func (x *GetTrafficRequest) ProtoReflect() protoreflect.Message {
	mi := &file_api_proto_msgTypes[4]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use GetTrafficRequest.ProtoReflect.Descriptor instead.
func (*GetTrafficRequest) Descriptor() ([]byte, []int) {
	return file_api_proto_rawDescGZIP(), []int{4}
}

```

此代码定义了一个名为 GetTrafficRequest 的接口，该接口包含一个名为 x 的 GetTrafficRequest 类型的变量。函数 GetUser() 通过该接口返回一个 User 类型的变量，如果 x 不为空则返回，否则返回 nil。

定义了一个名为 GetTrafficResponse 的结构体，该结构体包含一个名为 state 的 protoimpl.MessageState 类型的变量，一个名为 sizeCache 的 protoimpl.SizeCache 类型的变量，一个名为 unknownFields 的 protoimpl.UnknownFields 类型的变量。该结构体还包含一个名为 Success 的 bool 类型的变量，一个名为 info 的 string 类型的变量，一个名为 trafficTotal 的 Traffic 类型的变量，一个名为 speedCurrent 的 Speed 类型的变量。

根据 GetTrafficRequest 中的 x，创建并返回 GetTrafficResponse 中的 state，sizeCache，unknownFields 和 TrafficTotal 成员变量。如果 x 为空，返回 nil，否则返回 GetUser() 返回的 User 类型的变量。


```go
func (x *GetTrafficRequest) GetUser() *User {
	if x != nil {
		return x.User
	}
	return nil
}

type GetTrafficResponse struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Success      bool     `protobuf:"varint,1,opt,name=success,proto3" json:"success,omitempty"`
	Info         string   `protobuf:"bytes,2,opt,name=info,proto3" json:"info,omitempty"`
	TrafficTotal *Traffic `protobuf:"bytes,3,opt,name=traffic_total,json=trafficTotal,proto3" json:"traffic_total,omitempty"`
	SpeedCurrent *Speed   `protobuf:"bytes,4,opt,name=speed_current,json=speedCurrent,proto3" json:"speed_current,omitempty"`
}

```

这段代码定义了一个名为GetTrafficResponse的函数接收者类型，以及两个函数方法：Reset和String。

Reset函数的作用是重置接收者x的值，将其设置为GetTrafficResponse{}。如果定义时使用了`protoimpl.UnsafeEnabled`，那么在函数内部会禁止从函数外部直接访问接收者的`*x`字段。否则，直接将接收者的值赋给x。

String函数的作用是将接收者x转换为字符串，并返回。同样，如果定义时使用了`protoimpl.UnsafeEnabled`，那么该函数不会接受外部对x的直接访问。否则，使用`protoimpl.X.MessageStringOf`函数将接收者x转换为字符串，并将其存储在内部代理变量`ms`中。


```go
func (x *GetTrafficResponse) Reset() {
	*x = GetTrafficResponse{}
	if protoimpl.UnsafeEnabled {
		mi := &file_api_proto_msgTypes[5]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *GetTrafficResponse) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*GetTrafficResponse) ProtoMessage() {}

```

这段代码定义了一个名为func的函数，它接收一个名为x的整数类型的指针参数，并返回一个名为GetTrafficResponse的protoreflect.Message类型的值。函数的作用是接收一个GetTrafficResponse类型的对象，并将其转换为protoreflect.Message类型，以便使用该类型进行安全地与其他Go语言中的结构体进行交互。

具体来说，这段代码实现以下几个步骤：

1. 检查传入的x参数是否为空，如果是，则表示无法获取到该结构体的实例，直接返回file_api_proto_rawDescGZIP()和[]int{}，表示该函数不需要返回任何值。
2. 如果x参数不为空，则执行以下操作：
a. 获取一个名为mi的整数类型的实例，使用file_api_proto_msgTypes[5]作为key，因为该结构体对应的protoreflect.Message类型中包含一个名为mi的成员。
b. 如果使用了unsafe=true的选项，则表示启用unsafe模式，允许将x指向的内存中的值传递给Message，如果x指向的内存中没有Message类型的实例，则会创建一个新的Message实例，并将其存储为mi。
c. 如果使用了MessageStateOf函数，则直接返回x所指向对象的MessageStateOf函数的返回值，因为该函数返回的是一份Message类型，而不是一个新的Message实例。
3. 如果以上步骤2的任意一个都没有执行，则直接返回file_api_proto_rawDescGZIP()和[]int{}，表示该函数不需要返回任何值。

另外，该函数还实现了一个名为descriptor的函数，该函数接收一个GetTrafficResponse类型的指针参数，并返回一份包含GetTrafficResponse类型实例的JSON字节数组和该结构体类型的索引数组，以便在代码中更方便地使用该结构体类型的实例。


```go
func (x *GetTrafficResponse) ProtoReflect() protoreflect.Message {
	mi := &file_api_proto_msgTypes[5]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use GetTrafficResponse.ProtoReflect.Descriptor instead.
func (*GetTrafficResponse) Descriptor() ([]byte, []int) {
	return file_api_proto_rawDescGZIP(), []int{5}
}

```

这是一个接受一个名为GetTrafficResponse的整型指针变量x的函数，并返回一个布尔值，表示GetTrafficResponse对象中Success字段是否为真。

如果x不等于 nil，则函数将返回x.Success的值，否则返回false。

函数还有两个函数，一个是GetInfo，一个是GetTrafficTotal。

GetInfo函数接受一个名为GetTrafficResponse的整型指针变量x，并返回x.Info字段的值，如果x不等于 nil，则返回其.Info字段的值，否则返回"0"。

GetTrafficTotal函数接受一个名为GetTrafficResponse的整型指针变量x，并返回x.TrafficTotal字段的值，如果x不等于 nil，则返回其.TrafficTotal字段的值，否则返回 nil。


```go
func (x *GetTrafficResponse) GetSuccess() bool {
	if x != nil {
		return x.Success
	}
	return false
}

func (x *GetTrafficResponse) GetInfo() string {
	if x != nil {
		return x.Info
	}
	return ""
}

func (x *GetTrafficResponse) GetTrafficTotal() *Traffic {
	if x != nil {
		return x.TrafficTotal
	}
	return nil
}

```

这段代码定义了一个名为func的函数，接受一个名为x的指针参数，并返回一个名为Speed的类型为SpeedCurrent的类型。函数首先检查x是否为 nil，如果是，则返回x.SpeedCurrent，否则返回 nil。

接下来定义了一个名为ListUsersRequest的结构体，其中包含一个名为Reset的函数，该函数重置了ListUsersRequest的结构体。

最后，在代码的最后，定义了一个名为file_api_proto的类型，用于存储与该代码相关的proto消息类型。


```go
func (x *GetTrafficResponse) GetSpeedCurrent() *Speed {
	if x != nil {
		return x.SpeedCurrent
	}
	return nil
}

type ListUsersRequest struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields
}

func (x *ListUsersRequest) Reset() {
	*x = ListUsersRequest{}
	if protoimpl.UnsafeEnabled {
		mi := &file_api_proto_msgTypes[6]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

```

这段代码定义了一个名为ListUsersRequest的接口类型，并实现了两个方法：

1. String()，该方法返回一个字符串表示该接口类型的实例。
2. ProtoMessage()，该方法返回一个指向该接口类型实例的 protoreflect.Message 类型的指针。
3. ProtoReflect()，该方法返回一个指向该接口类型实例的 protoreflect.Message 的指针。

具体来说，这段代码实现了一个文件系统（fs）API中的ListUsersRequest接口类型的实例。

在这段代码中，作者首先定义了一个名为x的指针变量，该变量存储了一个ListUsersRequest类型的实例。

然后，作者定义了两个方法：

1. String()方法，该方法接收一个ListUsersRequest类型的参数x，并将其转换为字符串类型。具体来说，作者通过调用protoimpl.X.MessageStringOf()函数，将x的值转换为一个字符串，然后将其返回。
2. ProtoMessage()方法，该方法返回一个指向ListUsersRequest类型实例的protoreflect.Message类型的指针。这个指针类型在后面的使用中，被用来在代码中调用MessageStringOf()函数和MessageStateOf()函数，来获取和设置该接口类型实例的消息信息。
3. ProtoReflect()方法，该方法返回一个指向ListUsersRequest类型实例的protoreflect.Message类型的指针。这个指针类型在后面的使用中，被用来在代码中调用MessageOf()函数，来获取和设置该接口类型实例的消息信息。


```go
func (x *ListUsersRequest) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*ListUsersRequest) ProtoMessage() {}

func (x *ListUsersRequest) ProtoReflect() protoreflect.Message {
	mi := &file_api_proto_msgTypes[6]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

```

这段代码定义了一个名为ListUsersRequest的接口，该接口没有实现具体的方法来处理ListUsersRequest和ListUsersResponse之间的交互，也没有实现将ListUsersRequest和ListUsersResponse映射到同一接口的功能。

更具体地说，这段代码定义了一个名为Descriptor的函数，它返回了接口类型ListUsersRequest的元数据（即JSON编码的接口定义）的字节切片和该接口定义中所有字段名称的整数切片。这个函数在函数内部使用了file_api_proto_rawDescGZIP()函数来获取接口定义的JSON编码，然后将其转换为字节切片并返回。

另外，还定义了一个名为ListUsersResponse的 struct类型，该类型包含了一个名为状态的int类型，一个名为sizeCache的int类型和一个名为unknownFields的Pointer类型。然后，在函数内部创建了一个名为x的 pointer变量，并使用Reset()函数将其重置为ListUsersResponse的初始值。最后，使用了protoimpl.X.MessageStateOf()函数将ListUsersRequest的指针变量x的内部状态重置为initial歌坛。


```go
// Deprecated: Use ListUsersRequest.ProtoReflect.Descriptor instead.
func (*ListUsersRequest) Descriptor() ([]byte, []int) {
	return file_api_proto_rawDescGZIP(), []int{6}
}

type ListUsersResponse struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Status *UserStatus `protobuf:"bytes,1,opt,name=status,proto3" json:"status,omitempty"`
}

func (x *ListUsersResponse) Reset() {
	*x = ListUsersResponse{}
	if protoimpl.UnsafeEnabled {
		mi := &file_api_proto_msgTypes[7]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

```

这段代码定义了一个名为ListUsersResponse的接口类型，并实现了一些方法来转换为该接口类型的字节切片。

首先，该函数*ListUsersResponse) String()将接口类型的x转换为字符串表示。

其次，该函数(*ListUsersResponse) ProtoMessage()返回一个指向该接口类型的指针变量，但没有具体的实现。

接着，该函数(x *ListUsersResponse) ProtoReflect()返回一个指向该接口类型的 reflect 类型。

最后，该函数(x *ListUsersResponse) ProtoMessage()返回一个指向该接口类型的指针变量，实现了将该接口类型的x转换为相应长度的字节切片。


```go
func (x *ListUsersResponse) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*ListUsersResponse) ProtoMessage() {}

func (x *ListUsersResponse) ProtoReflect() protoreflect.Message {
	mi := &file_api_proto_msgTypes[7]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

```

这段代码定义了一个名为ListUsersResponse的接口，该接口具有两个方法：Descriptor和GetStatus。函数内部使用了Deprecated的标签，建议使用ListUsersResponse.protoReflect.Descriptor代替。

具体来说，这段代码实现了一个GetUsersRequest类型的实参，其中包含了一个User对象，该User对象在接口的Descriptor中对应了一个名为user的未知字段。在函数内部，还实现了一个名为Status的接口，该接口在UserStatus中对应了一个名为status的未知字段。

ListUsersResponse的作用是封装并实现了一个GetUsersRequest的请求，该请求包含了一个User对象，并返回了UserStatus和User对象的引用。


```go
// Deprecated: Use ListUsersResponse.ProtoReflect.Descriptor instead.
func (*ListUsersResponse) Descriptor() ([]byte, []int) {
	return file_api_proto_rawDescGZIP(), []int{7}
}

func (x *ListUsersResponse) GetStatus() *UserStatus {
	if x != nil {
		return x.Status
	}
	return nil
}

type GetUsersRequest struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	User *User `protobuf:"bytes,1,opt,name=user,proto3" json:"user,omitempty"`
}

```

这段代码定义了一个名为 GetUsersRequest 的结构体，并实现了两个函数：Reset 和 String。

Reset 函数接收一个指向 GetUsersRequest 类型的参数 x，将其赋值为一个空结构体，然后检查是否启用了 Unsafe 选项。如果启用了 Unsafe 选项，那么会创建一个名为 file_api_proto_msgTypes 的常量，并将 x 传递给它。最后，将 x 的内存地址存储在 file_api_proto_msgTypes 中，以便在将来的函数中访问。

String 函数接收一个指向 GetUsersRequest 类型的参数 x，并返回一个字符串，类似于其他 Go 语言中的 JSON 类型，将 GetUsersRequest 转换为 JSON 字节序列，然后将其转换为字符串并返回。

GetUsersRequest 是 GetUsersRequest 类型的别名，它似乎是一个非常具体的名字，但我不确定它的实际作用。这个名称可能是在代码中使用的，而不是在程序中使用的。


```go
func (x *GetUsersRequest) Reset() {
	*x = GetUsersRequest{}
	if protoimpl.UnsafeEnabled {
		mi := &file_api_proto_msgTypes[8]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *GetUsersRequest) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*GetUsersRequest) ProtoMessage() {}

```

这段代码定义了一个名为func的函数，它接收一个名为x的*GetUsersRequest类型的参数，并返回一个指向GetUsersRequest类型对象的指针，该对象携带了与该接口类型定义相对应的元数据。

首先，函数的实现部分检查传入的x参数是否为空，如果是，则执行以下操作：

1. 从file_api_proto_proto中查找与x指向的接口类型定义相关的元数据，如果没有找到，则创建一个新的元数据并将其存储为ms；
2. 如果已经找到了与x指向的接口类型定义相关的元数据，则返回指向该元数据对象的ms；
3. 如果上述步骤成功，则返回原始的x参数，该参数将被作为GetUsersRequest类型对象的指针返回。

另外，函数还定义了一个名为Descriptor的函数，该函数返回GetUsersRequest类型对象的一个描述，不包括该对象的内部实现，仅作为字符串返回。


```go
func (x *GetUsersRequest) ProtoReflect() protoreflect.Message {
	mi := &file_api_proto_msgTypes[8]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use GetUsersRequest.ProtoReflect.Descriptor instead.
func (*GetUsersRequest) Descriptor() ([]byte, []int) {
	return file_api_proto_rawDescGZIP(), []int{8}
}

```

此代码定义了一个名为 func 的函数，接收一个名为 x 的 *GetUsersRequest 参数，并返回一个 *User 类型的数据结构。

函数首先检查 x 是否为 nil，如果是，则直接返回 x.User 作为结果。否则，函数返回 nil，表示没有可用的数据结构。

接下来定义了一个名为 GetUsersResponse 的结构体，它包含一个名为 state 的 protoimpl.MessageState 字段，一个名为 sizeCache 的 protoimpl.SizeCache 字段，一个名为 unknownFields 的 protoimpl.UnknownFields 字段。

接着定义了一个名为 Success 的布尔字段，它表示是否成功，如果成功，则字段值为 1，否则为 0。

接着定义了一个名为 Info 的字符串字段，它表示操作结果的详细信息，如果信息可用，则字段值为操作结果的字符串，否则为空字符串。

最后定义了一个名为 Status 的 *UserStatus 类型的字段，它表示操作结果的状态，如果可访问，则字段值为 UserStatus.ACTIVE，否则为 UserStatus.INACTIVE。

最后在函数中返回了接收到的 *GetUsersRequest 参数，如果该参数为 nil，则返回 nil。


```go
func (x *GetUsersRequest) GetUser() *User {
	if x != nil {
		return x.User
	}
	return nil
}

type GetUsersResponse struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Success bool        `protobuf:"varint,1,opt,name=success,proto3" json:"success,omitempty"`
	Info    string      `protobuf:"bytes,2,opt,name=info,proto3" json:"info,omitempty"`
	Status  *UserStatus `protobuf:"bytes,3,opt,name=status,proto3" json:"status,omitempty"`
}

```

这段代码定义了一个名为 GetUsersResponse 的接口类型，以及两个函数：Reset 和 String。

Reset 函数的作用是重置 GetUsersResponse 接口类型的实例 x，即将 x 设为一个空 GetUsersResponse 实例。

String 函数的作用是将 GetUsersResponse 接口类型的实例 x 转换为字符串表示，即将 x 的字符串表示输出。

ProtoMessage 函数的作用是定义 GetUsersResponse 接口类型的消息类型，以便在消息接收方从客户端获取消息时能够正确地解析和构建消息。这个函数没有提供实现，因为它只是一个空函数，不会产生任何行为。


```go
func (x *GetUsersResponse) Reset() {
	*x = GetUsersResponse{}
	if protoimpl.UnsafeEnabled {
		mi := &file_api_proto_msgTypes[9]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *GetUsersResponse) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*GetUsersResponse) ProtoMessage() {}

```

这段代码定义了一个名为"func"的函数，接收一个名为"x"的参数，并返回一个名为"GetUsersResponse"的接口类型。

函数的作用是通过使用"file_api_proto_msgTypes"和"protoimpl.UnsafeEnabled"的条件，检查传入的"x"是否为空指针。如果是，则执行以下操作：

1. 从"file_api_proto_msgTypes"数组中获取与"file_api_proto_msgTypes"数组长度为9的元素。
2. 如果"protoimpl.UnsafeEnabled"为true，则执行以下操作：
  1. 从"x"指针所指向的"file_api_proto_msgTypes"数组中获取与"file_api_proto_msgTypes"数组长度为9的元素。
  2. 如果从"file_api_proto_msgTypes"数组中获取元素的"LoadMessageInfo"返回nil，则执行以下操作：
      1. 创建一个名为"ms"的新"file_api_proto_msgTypes"数组，并将其设置为从"file_api_proto_msgTypes"数组长度为9的元素的"MessageOf"函数的返回值。
      2. 将从"file_api_proto_msgTypes"数组中获取的元素的"MessageOf"函数的返回值存储为"ms"数组的第9个元素。
      3. 返回"ms"数组和从"file_api_proto_msgTypes"数组长度为9的元素的索引。

如果"protoimpl.UnsafeEnabled"为false，则直接返回"file_api_proto_msgTypes"数组和从"file_api_proto_msgTypes"数组长度为9的元素的索引。

另外，函数还定义了一个名为"Descriptor"的函数，该函数返回GetUsersResponse的接口类型的描述信息。


```go
func (x *GetUsersResponse) ProtoReflect() protoreflect.Message {
	mi := &file_api_proto_msgTypes[9]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use GetUsersResponse.ProtoReflect.Descriptor instead.
func (*GetUsersResponse) Descriptor() ([]byte, []int) {
	return file_api_proto_rawDescGZIP(), []int{9}
}

```

这段代码定义了一个名为GetUsersResponse的结构体，该结构体可能代表了与用户相关的信息。

接下来，该函数分别对GetUsersResponse结构的三个字段进行访问，并返回相应的值。

具体来说：

1. func (x *GetUsersResponse) GetSuccess() bool {
  如果x不等于 nil，那么x.Success的值就是真的，所以该函数返回的是一份成功信息。

2. func (x *GetUsersResponse) GetInfo() string {
  如果x不等于 nil，那么x.Info的值就是该结构体中的Info字段，所以该函数返回的字符串是该结构体的Info字段的值。

3. func (x *GetUsersResponse) GetStatus() *UserStatus {
  如果x不等于 nil，那么x.Status的值就是该结构体中的Status字段的值，所以该函数返回的UserStatus类型的指针是一个包含Status字段值的内存分配。


```go
func (x *GetUsersResponse) GetSuccess() bool {
	if x != nil {
		return x.Success
	}
	return false
}

func (x *GetUsersResponse) GetInfo() string {
	if x != nil {
		return x.Info
	}
	return ""
}

func (x *GetUsersResponse) GetStatus() *UserStatus {
	if x != nil {
		return x.Status
	}
	return nil
}

```

这段代码定义了一个名为 "SetUsersRequest" 的结构体，用于表示一个设置用户列表的请求。

这个结构体包含以下字段：

* "status"：一个 "UserStatus" 类型的字段，用于表示请求的状态信息，例如请求是否成功、是否有错误等等。这个字段被同步到 "SetUsersRequestImpl" 结构体中，同时也被持久化到磁盘上，以便以后的重置。
* "operation"：一个 "SetUsersRequest_Operation" 类型的字段，用于表示请求的操作类型，例如添加用户、删除用户等等。这个字段被同步到 "SetUsersRequestImpl" 结构体中，同时也被持久化到磁盘上，以便以后的重置。

另外，还包含一个 "unknownFields" 字段，这个字段是一个 "protoimpl.UnknownFields" 的指针，用于表示一些可能没有被定义的字段。


```go
type SetUsersRequest struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Status    *UserStatus               `protobuf:"bytes,1,opt,name=status,proto3" json:"status,omitempty"`
	Operation SetUsersRequest_Operation `protobuf:"varint,2,opt,name=operation,proto3,enum=trojan.api.SetUsersRequest_Operation" json:"operation,omitempty"`
}

func (x *SetUsersRequest) Reset() {
	*x = SetUsersRequest{}
	if protoimpl.UnsafeEnabled {
		mi := &file_api_proto_msgTypes[10]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

```

这段代码定义了一个名为 func 的函数，它接收一个名为 x 的 *SetUsersRequest 类型的参数，并返回一个字符串类型的结果。

首先，定义了一个名为 func 的函数，它接收一个名为 x 的 *SetUsersRequest 类型的参数，并将其作为参数传递给 protoimpl.X.MessageStringOf() 函数，然后将返回结果存储在 x 的 SetUsersRequest 类型的字段中，最后返回该字段。

接着，定义了一个名为 func 的函数，它接收一个名为 x 的 *SetUsersRequest 类型的参数，然后将其作为参数传递给 protoimpl.X.MessageStringOf() 函数，并返回一个空字符串。

最后，定义了一个名为 func 的函数，它接收一个名为 x 的 *SetUsersRequest 类型的参数，然后将其作为参数传递给 mi.MessageOf() 函数，并返回一个名为 file_api_proto_msgTypes 的名为 10 的类型，或者是一个名为 *SetUsersRequest 的名为 *10 的类型，具体取决于是否启用了 protoimpl.UnsafeEnabled。


```go
func (x *SetUsersRequest) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*SetUsersRequest) ProtoMessage() {}

func (x *SetUsersRequest) ProtoReflect() protoreflect.Message {
	mi := &file_api_proto_msgTypes[10]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

```

该代码定义了一个名为 `SetUsersRequest` 的接口，用于在用户授权下批量创建或更新用户。

该接口的实现包括以下方法：

1. `Descriptor()` 方法返回了该接口的 `Descriptor` 字段类型以及该接口的 `field_网络诈骗_容错_parse_description` 标记数。其中 `Descriptor` 字段类型表示接口的版本信息，`field_网络诈骗_容错_parse_description` 标记数表示该接口支持的最大版本号。

2. `GetStatus()` 方法返回了该接口的 `Status` 字段类型的实例，该字段类型表示用户的状态，可以是一个 `UserStatus` 类型的实例。

3. `GetOperation()` 方法返回了该接口的 `Operation` 字段类型的实例，该字段类型表示操作类型，可以是 `SetUsersRequest_Add`、`SetUsersRequest_UpdateUser` 或 `SetUsersRequest_DeleteUser` 中的一个。

该代码的目的是定义一个用于设置或获取用户信息的接口，通过该接口，用户可以方便地进行用户信息的设置、获取和更新等操作。


```go
// Deprecated: Use SetUsersRequest.ProtoReflect.Descriptor instead.
func (*SetUsersRequest) Descriptor() ([]byte, []int) {
	return file_api_proto_rawDescGZIP(), []int{10}
}

func (x *SetUsersRequest) GetStatus() *UserStatus {
	if x != nil {
		return x.Status
	}
	return nil
}

func (x *SetUsersRequest) GetOperation() SetUsersRequest_Operation {
	if x != nil {
		return x.Operation
	}
	return SetUsersRequest_Add
}

```

这段代码定义了一个名为 "SetUsersResponse" 的 struct 类型，它用于表示在文件或网络用户授权时接收到的响应。

这个 struct 类型包含三个字段：

- "state"（类型为 protoimpl.MessageState，表示一个消息的狀態，用于定义该消息的支持哪些属性和方法。这个字段通常不会在 serialized 版本中出现，因为它仅在构建或解码消息时使用。）
- "sizeCache"（类型为 protoimpl.SizeCache，表示一个缓存大小，用于在后续请求中缓存已经下载的数据。这个字段通常不会在 serialized 版本中出现，因为它仅在构建或解码消息时使用。）
- "unknownFields"（类型为 protoimpl.UnknownFields，表示一个未知字段的集合。这个字段包含多个未知字段，在序列化时被忽略。）

此外，还有一个 "Reset" 方法和一个 "unknownUsers" 字段，但是它没有被使用。


```go
type SetUsersResponse struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Success bool   `protobuf:"varint,1,opt,name=success,proto3" json:"success,omitempty"`
	Info    string `protobuf:"bytes,2,opt,name=info,proto3" json:"info,omitempty"`
}

func (x *SetUsersResponse) Reset() {
	*x = SetUsersResponse{}
	if protoimpl.UnsafeEnabled {
		mi := &file_api_proto_msgTypes[11]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

```

这段代码定义了两个函数，分别用于将 `SetUsersResponse` 类型转换为字符串和将字符串类型转换为 `SetUsersResponse` 类型。

第一个函数 `func (x *SetUsersResponse) String() string` 将 `SetUsersResponse` 类型的 `x` 值转换为字符串，并返回 `x` 指向的 `SetUsersResponse` 类型的实例的 `MessageStringOf` 函数的返回值。这个函数的作用是将 `SetUsersResponse` 类型的数据转换为字符串，然后返回该数据。

第二个函数 `func (*SetUsersResponse) ProtoMessage()` 将 `SetUsersResponse` 类型转换为 `FileAPI_Response` 类型的 `Message` 类型，并返回一个空指针。这个函数的作用是将 `SetUsersResponse` 类型的数据转换为 `FileAPI_Response` 类型的 `Message` 类型，然后返回该类型的指针。

第三个函数 `func (x *SetUsersResponse) ProtoReflect() protoreflect.Message` 将 `SetUsersResponse` 类型转换为 `Message` 指针，并返回一个空指针。这个函数的作用是将 `SetUsersResponse` 类型的数据转换为 `Message` 指针类型，然后返回该类型的指针。


```go
func (x *SetUsersResponse) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*SetUsersResponse) ProtoMessage() {}

func (x *SetUsersResponse) ProtoReflect() protoreflect.Message {
	mi := &file_api_proto_msgTypes[11]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

```

该代码定义了一个名为 `SetUsersResponse` 的结构体类型。这个结构体类型有两个方法：

1. `Descriptor()` 方法返回 `file_api_proto_rawDescGZIP()` 和 `[]int` 两个字节切片。其中，`file_api_proto_rawDescGZIP()` 方法从 `file_api_proto.proto` 文件中生成描述符，`[]int` 表示该结构体类型的字段数量。

2. `GetSuccess()` 方法返回 `x` 结构体中的 `Success` 字段值，如果 `x` 为 `nil` 则返回 `false`。

3. `GetInfo()` 方法返回 `x` 结构体中的 `Info` 字段值，如果 `x` 为 `nil` 则返回 `"`。

这个 `SetUsersResponse` 结构体类型可能是从另一个结构体类型派生而来的，而且它使用了另一个名为 `file_api_proto.proto` 的文件中的类型描述符。由于在代码中使用了 `Deprecated: Use SetUsersResponse.ProtoReflect.Descriptor instead.` 的注释，因此推荐使用 `SetUsersResponse.ProtoReflect.Descriptor()` 函数来获取该结构体类型的描述符。


```go
// Deprecated: Use SetUsersResponse.ProtoReflect.Descriptor instead.
func (*SetUsersResponse) Descriptor() ([]byte, []int) {
	return file_api_proto_rawDescGZIP(), []int{11}
}

func (x *SetUsersResponse) GetSuccess() bool {
	if x != nil {
		return x.Success
	}
	return false
}

func (x *SetUsersResponse) GetInfo() string {
	if x != nil {
		return x.Info
	}
	return ""
}

```

It appears that the output data is a series of hexadecimal values. Each value is separated by a delimiter, which appears to be a space character. It is difficult to determine the meaning of each value without more information.



```go
var File_api_proto protoreflect.FileDescriptor

var file_api_proto_rawDesc = []byte{
	0x0a, 0x09, 0x61, 0x70, 0x69, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x0a, 0x74, 0x72, 0x6f,
	0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x22, 0x5b, 0x0a, 0x07, 0x54, 0x72, 0x61, 0x66, 0x66,
	0x69, 0x63, 0x12, 0x25, 0x0a, 0x0e, 0x75, 0x70, 0x6c, 0x6f, 0x61, 0x64, 0x5f, 0x74, 0x72, 0x61,
	0x66, 0x66, 0x69, 0x63, 0x18, 0x01, 0x20, 0x01, 0x28, 0x04, 0x52, 0x0d, 0x75, 0x70, 0x6c, 0x6f,
	0x61, 0x64, 0x54, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63, 0x12, 0x29, 0x0a, 0x10, 0x64, 0x6f, 0x77,
	0x6e, 0x6c, 0x6f, 0x61, 0x64, 0x5f, 0x74, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63, 0x18, 0x02, 0x20,
	0x01, 0x28, 0x04, 0x52, 0x0f, 0x64, 0x6f, 0x77, 0x6e, 0x6c, 0x6f, 0x61, 0x64, 0x54, 0x72, 0x61,
	0x66, 0x66, 0x69, 0x63, 0x22, 0x51, 0x0a, 0x05, 0x53, 0x70, 0x65, 0x65, 0x64, 0x12, 0x21, 0x0a,
	0x0c, 0x75, 0x70, 0x6c, 0x6f, 0x61, 0x64, 0x5f, 0x73, 0x70, 0x65, 0x65, 0x64, 0x18, 0x01, 0x20,
	0x01, 0x28, 0x04, 0x52, 0x0b, 0x75, 0x70, 0x6c, 0x6f, 0x61, 0x64, 0x53, 0x70, 0x65, 0x65, 0x64,
	0x12, 0x25, 0x0a, 0x0e, 0x64, 0x6f, 0x77, 0x6e, 0x6c, 0x6f, 0x61, 0x64, 0x5f, 0x73, 0x70, 0x65,
	0x65, 0x64, 0x18, 0x02, 0x20, 0x01, 0x28, 0x04, 0x52, 0x0d, 0x64, 0x6f, 0x77, 0x6e, 0x6c, 0x6f,
	0x61, 0x64, 0x53, 0x70, 0x65, 0x65, 0x64, 0x22, 0x36, 0x0a, 0x04, 0x55, 0x73, 0x65, 0x72, 0x12,
	0x1a, 0x0a, 0x08, 0x70, 0x61, 0x73, 0x73, 0x77, 0x6f, 0x72, 0x64, 0x18, 0x01, 0x20, 0x01, 0x28,
	0x09, 0x52, 0x08, 0x70, 0x61, 0x73, 0x73, 0x77, 0x6f, 0x72, 0x64, 0x12, 0x12, 0x0a, 0x04, 0x68,
	0x61, 0x73, 0x68, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52, 0x04, 0x68, 0x61, 0x73, 0x68, 0x22,
	0x92, 0x02, 0x0a, 0x0a, 0x55, 0x73, 0x65, 0x72, 0x53, 0x74, 0x61, 0x74, 0x75, 0x73, 0x12, 0x24,
	0x0a, 0x04, 0x75, 0x73, 0x65, 0x72, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x10, 0x2e, 0x74,
	0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x55, 0x73, 0x65, 0x72, 0x52, 0x04,
	0x75, 0x73, 0x65, 0x72, 0x12, 0x38, 0x0a, 0x0d, 0x74, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63, 0x5f,
	0x74, 0x6f, 0x74, 0x61, 0x6c, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x13, 0x2e, 0x74, 0x72,
	0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x54, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63,
	0x52, 0x0c, 0x74, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63, 0x54, 0x6f, 0x74, 0x61, 0x6c, 0x12, 0x36,
	0x0a, 0x0d, 0x73, 0x70, 0x65, 0x65, 0x64, 0x5f, 0x63, 0x75, 0x72, 0x72, 0x65, 0x6e, 0x74, 0x18,
	0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x11, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61,
	0x70, 0x69, 0x2e, 0x53, 0x70, 0x65, 0x65, 0x64, 0x52, 0x0c, 0x73, 0x70, 0x65, 0x65, 0x64, 0x43,
	0x75, 0x72, 0x72, 0x65, 0x6e, 0x74, 0x12, 0x32, 0x0a, 0x0b, 0x73, 0x70, 0x65, 0x65, 0x64, 0x5f,
	0x6c, 0x69, 0x6d, 0x69, 0x74, 0x18, 0x04, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x11, 0x2e, 0x74, 0x72,
	0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x53, 0x70, 0x65, 0x65, 0x64, 0x52, 0x0a,
	0x73, 0x70, 0x65, 0x65, 0x64, 0x4c, 0x69, 0x6d, 0x69, 0x74, 0x12, 0x1d, 0x0a, 0x0a, 0x69, 0x70,
	0x5f, 0x63, 0x75, 0x72, 0x72, 0x65, 0x6e, 0x74, 0x18, 0x05, 0x20, 0x01, 0x28, 0x05, 0x52, 0x09,
	0x69, 0x70, 0x43, 0x75, 0x72, 0x72, 0x65, 0x6e, 0x74, 0x12, 0x19, 0x0a, 0x08, 0x69, 0x70, 0x5f,
	0x6c, 0x69, 0x6d, 0x69, 0x74, 0x18, 0x06, 0x20, 0x01, 0x28, 0x05, 0x52, 0x07, 0x69, 0x70, 0x4c,
	0x69, 0x6d, 0x69, 0x74, 0x22, 0x39, 0x0a, 0x11, 0x47, 0x65, 0x74, 0x54, 0x72, 0x61, 0x66, 0x66,
	0x69, 0x63, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x12, 0x24, 0x0a, 0x04, 0x75, 0x73, 0x65,
	0x72, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x10, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e,
	0x2e, 0x61, 0x70, 0x69, 0x2e, 0x55, 0x73, 0x65, 0x72, 0x52, 0x04, 0x75, 0x73, 0x65, 0x72, 0x22,
	0xb4, 0x01, 0x0a, 0x12, 0x47, 0x65, 0x74, 0x54, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63, 0x52, 0x65,
	0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x12, 0x18, 0x0a, 0x07, 0x73, 0x75, 0x63, 0x63, 0x65, 0x73,
	0x73, 0x18, 0x01, 0x20, 0x01, 0x28, 0x08, 0x52, 0x07, 0x73, 0x75, 0x63, 0x63, 0x65, 0x73, 0x73,
	0x12, 0x12, 0x0a, 0x04, 0x69, 0x6e, 0x66, 0x6f, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52, 0x04,
	0x69, 0x6e, 0x66, 0x6f, 0x12, 0x38, 0x0a, 0x0d, 0x74, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63, 0x5f,
	0x74, 0x6f, 0x74, 0x61, 0x6c, 0x18, 0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x13, 0x2e, 0x74, 0x72,
	0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x54, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63,
	0x52, 0x0c, 0x74, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63, 0x54, 0x6f, 0x74, 0x61, 0x6c, 0x12, 0x36,
	0x0a, 0x0d, 0x73, 0x70, 0x65, 0x65, 0x64, 0x5f, 0x63, 0x75, 0x72, 0x72, 0x65, 0x6e, 0x74, 0x18,
	0x04, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x11, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61,
	0x70, 0x69, 0x2e, 0x53, 0x70, 0x65, 0x65, 0x64, 0x52, 0x0c, 0x73, 0x70, 0x65, 0x65, 0x64, 0x43,
	0x75, 0x72, 0x72, 0x65, 0x6e, 0x74, 0x22, 0x12, 0x0a, 0x10, 0x4c, 0x69, 0x73, 0x74, 0x55, 0x73,
	0x65, 0x72, 0x73, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x22, 0x43, 0x0a, 0x11, 0x4c, 0x69,
	0x73, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x12,
	0x2e, 0x0a, 0x06, 0x73, 0x74, 0x61, 0x74, 0x75, 0x73, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b, 0x32,
	0x16, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x55, 0x73, 0x65,
	0x72, 0x53, 0x74, 0x61, 0x74, 0x75, 0x73, 0x52, 0x06, 0x73, 0x74, 0x61, 0x74, 0x75, 0x73, 0x22,
	0x37, 0x0a, 0x0f, 0x47, 0x65, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x52, 0x65, 0x71, 0x75, 0x65,
	0x73, 0x74, 0x12, 0x24, 0x0a, 0x04, 0x75, 0x73, 0x65, 0x72, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b,
	0x32, 0x10, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x55, 0x73,
	0x65, 0x72, 0x52, 0x04, 0x75, 0x73, 0x65, 0x72, 0x22, 0x70, 0x0a, 0x10, 0x47, 0x65, 0x74, 0x55,
	0x73, 0x65, 0x72, 0x73, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x12, 0x18, 0x0a, 0x07,
	0x73, 0x75, 0x63, 0x63, 0x65, 0x73, 0x73, 0x18, 0x01, 0x20, 0x01, 0x28, 0x08, 0x52, 0x07, 0x73,
	0x75, 0x63, 0x63, 0x65, 0x73, 0x73, 0x12, 0x12, 0x0a, 0x04, 0x69, 0x6e, 0x66, 0x6f, 0x18, 0x02,
	0x20, 0x01, 0x28, 0x09, 0x52, 0x04, 0x69, 0x6e, 0x66, 0x6f, 0x12, 0x2e, 0x0a, 0x06, 0x73, 0x74,
	0x61, 0x74, 0x75, 0x73, 0x18, 0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x16, 0x2e, 0x74, 0x72, 0x6f,
	0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x55, 0x73, 0x65, 0x72, 0x53, 0x74, 0x61, 0x74,
	0x75, 0x73, 0x52, 0x06, 0x73, 0x74, 0x61, 0x74, 0x75, 0x73, 0x22, 0xb4, 0x01, 0x0a, 0x0f, 0x53,
	0x65, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x12, 0x2e,
	0x0a, 0x06, 0x73, 0x74, 0x61, 0x74, 0x75, 0x73, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x16,
	0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x55, 0x73, 0x65, 0x72,
	0x53, 0x74, 0x61, 0x74, 0x75, 0x73, 0x52, 0x06, 0x73, 0x74, 0x61, 0x74, 0x75, 0x73, 0x12, 0x43,
	0x0a, 0x09, 0x6f, 0x70, 0x65, 0x72, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x18, 0x02, 0x20, 0x01, 0x28,
	0x0e, 0x32, 0x25, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x53,
	0x65, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x2e, 0x4f,
	0x70, 0x65, 0x72, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x52, 0x09, 0x6f, 0x70, 0x65, 0x72, 0x61, 0x74,
	0x69, 0x6f, 0x6e, 0x22, 0x2c, 0x0a, 0x09, 0x4f, 0x70, 0x65, 0x72, 0x61, 0x74, 0x69, 0x6f, 0x6e,
	0x12, 0x07, 0x0a, 0x03, 0x41, 0x64, 0x64, 0x10, 0x00, 0x12, 0x0a, 0x0a, 0x06, 0x44, 0x65, 0x6c,
	0x65, 0x74, 0x65, 0x10, 0x01, 0x12, 0x0a, 0x0a, 0x06, 0x4d, 0x6f, 0x64, 0x69, 0x66, 0x79, 0x10,
	0x02, 0x22, 0x40, 0x0a, 0x10, 0x53, 0x65, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x52, 0x65, 0x73,
	0x70, 0x6f, 0x6e, 0x73, 0x65, 0x12, 0x18, 0x0a, 0x07, 0x73, 0x75, 0x63, 0x63, 0x65, 0x73, 0x73,
	0x18, 0x01, 0x20, 0x01, 0x28, 0x08, 0x52, 0x07, 0x73, 0x75, 0x63, 0x63, 0x65, 0x73, 0x73, 0x12,
	0x12, 0x0a, 0x04, 0x69, 0x6e, 0x66, 0x6f, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52, 0x04, 0x69,
	0x6e, 0x66, 0x6f, 0x32, 0x64, 0x0a, 0x13, 0x54, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x43, 0x6c, 0x69,
	0x65, 0x6e, 0x74, 0x53, 0x65, 0x72, 0x76, 0x69, 0x63, 0x65, 0x12, 0x4d, 0x0a, 0x0a, 0x47, 0x65,
	0x74, 0x54, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63, 0x12, 0x1d, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61,
	0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x47, 0x65, 0x74, 0x54, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63,
	0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x1a, 0x1e, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e,
	0x2e, 0x61, 0x70, 0x69, 0x2e, 0x47, 0x65, 0x74, 0x54, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63, 0x52,
	0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x22, 0x00, 0x32, 0xfd, 0x01, 0x0a, 0x13, 0x54, 0x72,
	0x6f, 0x6a, 0x61, 0x6e, 0x53, 0x65, 0x72, 0x76, 0x65, 0x72, 0x53, 0x65, 0x72, 0x76, 0x69, 0x63,
	0x65, 0x12, 0x4c, 0x0a, 0x09, 0x4c, 0x69, 0x73, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x12, 0x1c,
	0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x4c, 0x69, 0x73, 0x74,
	0x55, 0x73, 0x65, 0x72, 0x73, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x1a, 0x1d, 0x2e, 0x74,
	0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x4c, 0x69, 0x73, 0x74, 0x55, 0x73,
	0x65, 0x72, 0x73, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x22, 0x00, 0x30, 0x01, 0x12,
	0x4b, 0x0a, 0x08, 0x47, 0x65, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x12, 0x1b, 0x2e, 0x74, 0x72,
	0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x47, 0x65, 0x74, 0x55, 0x73, 0x65, 0x72,
	0x73, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x1a, 0x1c, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61,
	0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x47, 0x65, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x52, 0x65,
	0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x22, 0x00, 0x28, 0x01, 0x30, 0x01, 0x12, 0x4b, 0x0a, 0x08,
	0x53, 0x65, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x12, 0x1b, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61,
	0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x53, 0x65, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x52, 0x65,
	0x71, 0x75, 0x65, 0x73, 0x74, 0x1a, 0x1c, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61,
	0x70, 0x69, 0x2e, 0x53, 0x65, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x52, 0x65, 0x73, 0x70, 0x6f,
	0x6e, 0x73, 0x65, 0x22, 0x00, 0x28, 0x01, 0x30, 0x01, 0x42, 0x2c, 0x5a, 0x2a, 0x67, 0x69, 0x74,
	0x68, 0x75, 0x62, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x70, 0x34, 0x67, 0x65, 0x66, 0x61, 0x75, 0x31,
	0x74, 0x2f, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2d, 0x67, 0x6f, 0x2f, 0x61, 0x70, 0x69, 0x2f,
	0x73, 0x65, 0x72, 0x76, 0x69, 0x63, 0x65, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
}

```

该代码定义了一个名为file_api_proto_rawDescOnce的变量，其类型为sync.Once，用于保证该变量的值仅在函数内部有效，即一次函数调用后自动失效。

该代码还定义了一个名为file_api_proto_rawDescData的变量，其类型为[protoimpl.X:1]byte，用于保存file_api_proto_rawDesc类型数据结构的原始摘要。

该代码实现了一个名为file_api_proto_rawDescGZIP的函数，其功能是将file_api_proto_rawDescData编码为GZIP压缩后的字节切片，并返回file_api_proto_rawDescData。

该函数内部使用了一个名为file_api_proto_rawDescOnce的Once类型变量，用于确保file_api_proto_rawDescOnce只被设置一次，并在函数内部正确地设置该Once类型变量。

该函数内部还定义了一个名为file_api_proto_enumTypes的变量，其类型为[protoimpl.X:1]protoimpl.EnumInfo，用于定义file_api_proto_enum类型的结构体变量。

该函数内部还定义了一个名为file_api_proto_msgTypes的变量，其类型为[protoimpl.X:1]protoimpl.MessageInfo，用于定义file_api_proto_msg类型的结构体变量。

该函数内部还定义了一个名为file_api_proto_goTypes的变量，其类型为[]interface{}{}，用于定义go类型变量。

该函数内部定义了一个名为file_api_proto_setUsersRequest的函数，其接收一个SetUsersRequest类型的参数，并返回一个SetUsersResponse类型的结果。其函数内部的参数个数为1，即只接收一个参数，因此该函数的参数类型为nil。

该函数内部定义了一个名为file_api_proto_getTrafficRequest的函数，其接收一个GetTrafficRequest类型的参数，并返回一个GetTrafficResponse类型的结果。其函数内部的参数个数为1，即只接收一个参数，因此该函数的参数类型为nil。

该函数内部定义了一个名为file_api_proto_listUsersRequest的函数，其接收一个ListUsersRequest类型的参数，并返回一个ListUsersResponse类型的结果。其函数内部的参数个数为1，即只接收一个参数，因此该函数的参数类型为nil。

该函数内部定义了一个名为file_api_proto_getUsersResponse的函数，其接收一个GetUsersResponse类型的参数，并返回一个User类型的结果。其函数内部的参数个数为1，即只接收一个参数，因此该函数的参数类型为nil。

该函数内部定义了一个名为file_api_proto_setUsersRequest的函数，其接收一个SetUsersRequest类型的参数，并返回一个SetUsersResponse类型的结果。其函数内部的参数个数为1，即只接收一个参数，因此该函数的参数类型为nil。

该函数内部定义了一个名为file_api_proto_getUsersResponse的函数，其接收一个GetUsersResponse类型的参数，并返回一个User类型的结果。其函数内部的参数个数为1，即只接收一个参数，因此该函数的参数类型为nil。


```go
var (
	file_api_proto_rawDescOnce sync.Once
	file_api_proto_rawDescData = file_api_proto_rawDesc
)

func file_api_proto_rawDescGZIP() []byte {
	file_api_proto_rawDescOnce.Do(func() {
		file_api_proto_rawDescData = protoimpl.X.CompressGZIP(file_api_proto_rawDescData)
	})
	return file_api_proto_rawDescData
}

var file_api_proto_enumTypes = make([]protoimpl.EnumInfo, 1)
var file_api_proto_msgTypes = make([]protoimpl.MessageInfo, 12)
var file_api_proto_goTypes = []interface{}{
	(SetUsersRequest_Operation)(0), // 0: trojan.api.SetUsersRequest.Operation
	(*Traffic)(nil),                // 1: trojan.api.Traffic
	(*Speed)(nil),                  // 2: trojan.api.Speed
	(*User)(nil),                   // 3: trojan.api.User
	(*UserStatus)(nil),             // 4: trojan.api.UserStatus
	(*GetTrafficRequest)(nil),      // 5: trojan.api.GetTrafficRequest
	(*GetTrafficResponse)(nil),     // 6: trojan.api.GetTrafficResponse
	(*ListUsersRequest)(nil),       // 7: trojan.api.ListUsersRequest
	(*ListUsersResponse)(nil),      // 8: trojan.api.ListUsersResponse
	(*GetUsersRequest)(nil),        // 9: trojan.api.GetUsersRequest
	(*GetUsersResponse)(nil),       // 10: trojan.api.GetUsersResponse
	(*SetUsersRequest)(nil),        // 11: trojan.api.SetUsersRequest
	(*SetUsersResponse)(nil),       // 12: trojan.api.SetUsersResponse
}
```

This is a list of extensions for the trojan.api.h interface, which provides methods for various tasks in the Trojan server and client services, such as getting and setting users, getting traffic, and lists users. The extensions include the return type, input/output types, and the available methods for each task.

Methods:

- GetUsers: This method returns a list of users in the current project.
- ListUsers: This method returns a list of users in the current project.
- SetUsers: This method updates the settings for a user in the current project.
- GetTraffic: This method retrieves the traffic logs from the server.
- SetTraffic: This method sends the traffic logs from the server to the client.

Input/Output Types:

- GetUsers: This method requires an empty list as input, and returns a list of users in the current project.
- ListUsers: This method requires an empty list as input, and returns a list of users in the current project.
- SetUsers: This method requires a user to be added or updated, and returns a boolean indicating whether the operation was successful or not.
- GetTraffic: This method requires an empty list as input, and returns a list of traffic objects in the current project.
- SetTraffic: This method requires a traffic object to be added or updated, and returns a boolean indicating whether the operation was successful or not.

Extension Type:

- trojan.api.UserStatus
- trojan.api.Status

Extension Extendee:

- trojan.api.User
- trojan.api.Group
- trojan.api.SecurityContext
- trojan.api.System

Extension Type:

- ext



```go
var file_api_proto_depIdxs = []int32{
	3,  // 0: trojan.api.UserStatus.user:type_name -> trojan.api.User
	1,  // 1: trojan.api.UserStatus.traffic_total:type_name -> trojan.api.Traffic
	2,  // 2: trojan.api.UserStatus.speed_current:type_name -> trojan.api.Speed
	2,  // 3: trojan.api.UserStatus.speed_limit:type_name -> trojan.api.Speed
	3,  // 4: trojan.api.GetTrafficRequest.user:type_name -> trojan.api.User
	1,  // 5: trojan.api.GetTrafficResponse.traffic_total:type_name -> trojan.api.Traffic
	2,  // 6: trojan.api.GetTrafficResponse.speed_current:type_name -> trojan.api.Speed
	4,  // 7: trojan.api.ListUsersResponse.status:type_name -> trojan.api.UserStatus
	3,  // 8: trojan.api.GetUsersRequest.user:type_name -> trojan.api.User
	4,  // 9: trojan.api.GetUsersResponse.status:type_name -> trojan.api.UserStatus
	4,  // 10: trojan.api.SetUsersRequest.status:type_name -> trojan.api.UserStatus
	0,  // 11: trojan.api.SetUsersRequest.operation:type_name -> trojan.api.SetUsersRequest.Operation
	5,  // 12: trojan.api.TrojanClientService.GetTraffic:input_type -> trojan.api.GetTrafficRequest
	7,  // 13: trojan.api.TrojanServerService.ListUsers:input_type -> trojan.api.ListUsersRequest
	9,  // 14: trojan.api.TrojanServerService.GetUsers:input_type -> trojan.api.GetUsersRequest
	11, // 15: trojan.api.TrojanServerService.SetUsers:input_type -> trojan.api.SetUsersRequest
	6,  // 16: trojan.api.TrojanClientService.GetTraffic:output_type -> trojan.api.GetTrafficResponse
	8,  // 17: trojan.api.TrojanServerService.ListUsers:output_type -> trojan.api.ListUsersResponse
	10, // 18: trojan.api.TrojanServerService.GetUsers:output_type -> trojan.api.GetUsersResponse
	12, // 19: trojan.api.TrojanServerService.SetUsers:output_type -> trojan.api.SetUsersResponse
	16, // [16:20] is the sub-list for method output_type
	12, // [12:16] is the sub-list for method input_type
	12, // [12:12] is the sub-list for extension type_name
	12, // [12:12] is the sub-list for extension extendee
	0,  // [0:12] is the sub-list for field type_name
}

```

`file_api_proto` 是用于定义 API 接口的编程语言实现。在 Go 语言中，我们可以通过使用 `go. bytecode.在看向文件时，我们需要用到文件 API 的接口类型。


```go
func init() { file_api_proto_init() }
func file_api_proto_init() {
	if File_api_proto != nil {
		return
	}
	if !protoimpl.UnsafeEnabled {
		file_api_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*Traffic); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_api_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*Speed); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_api_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*User); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_api_proto_msgTypes[3].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*UserStatus); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_api_proto_msgTypes[4].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*GetTrafficRequest); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_api_proto_msgTypes[5].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*GetTrafficResponse); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_api_proto_msgTypes[6].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*ListUsersRequest); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_api_proto_msgTypes[7].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*ListUsersResponse); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_api_proto_msgTypes[8].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*GetUsersRequest); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_api_proto_msgTypes[9].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*GetUsersResponse); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_api_proto_msgTypes[10].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*SetUsersRequest); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_api_proto_msgTypes[11].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*SetUsersResponse); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
	}
	type x struct{}
	out := protoimpl.TypeBuilder{
		File: protoimpl.DescBuilder{
			GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
			RawDescriptor: file_api_proto_rawDesc,
			NumEnums:      1,
			NumMessages:   12,
			NumExtensions: 0,
			NumServices:   2,
		},
		GoTypes:           file_api_proto_goTypes,
		DependencyIndexes: file_api_proto_depIdxs,
		EnumInfos:         file_api_proto_enumTypes,
		MessageInfos:      file_api_proto_msgTypes,
	}.Build()
	File_api_proto = out.File
	file_api_proto_rawDesc = nil
	file_api_proto_goTypes = nil
	file_api_proto_depIdxs = nil
}

```