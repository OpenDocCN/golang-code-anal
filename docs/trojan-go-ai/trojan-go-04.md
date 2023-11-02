# trojan-go源码解析 4

---
title: "Trojan基本原理"
draft: false
weight: 21
---

这个页面将会简单讲述Trojan协议的基本工作原理。如果你对于GFW和Trojan的工作方式不感兴趣，可以跳过这一小节。但为了更好地保护你的通讯安全性和节点的隐蔽性，我还是建议你阅读。

## 为什么（使用流密码的）Shadowsocks容易遭到封锁

防火墙在早期仅仅只是对出境流量进行截获和审查，也即**被动检测**。Shadowsocks的加密协议设计使得传输的数据包本身几乎没有任何特征，看起来类似于完全随机的比特流，这在早期的确能有效绕过GFW。

目前的GFW已经开始采用**主动探测**的方式。具体来说，当GFW发现一个可疑的无法识别的连接时（大流量，随机字节流，高位端口等特征），将会**主动连接**这个服务器端口，重放之前捕获到的流量（或者经过一些精心修改后重放）。Shadowsocks服务器检测到不正常的连接，将连接断开。这种不正常的流量和断开连接的行为被视作可疑的Shadowsocks服务器的特征，于是该服务器被加入GFW的可疑名单中。这个名单不一定立即生效，而是在某些特殊的敏感时期，可疑名单中的服务器会遭到暂时或者永久的封锁。该可疑名单是否封锁，可能由人为因素决定。

如果你想了解更多，可以参考[这篇文章](https://gfw.report/blog/gfw_shadowsocks/)。

## Trojan如何绕过GFW

与Shadowsocks相反，Trojan不使用自定义的加密协议来隐藏自身。相反，使用特征明显的TLS协议(TLS/SSL)，使得流量看起来与正常的HTTPS网站相同。TLS是一个成熟的加密体系，HTTPS即使用TLS承载HTTP流量。使用**正确配置**的加密TLS隧道，可以保证传输的

- 保密性（GFW无法得知传输的内容）

- 完整性（一旦GFW试图篡改传输的密文，通讯双方都会发现）

- 不可抵赖（GFW无法伪造身份冒充服务端或者客户端）

- 前向安全（即使密钥泄露，GFW也无法解密先前的加密流量）

对于被动检测，Trojan协议的流量与HTTPS流量的特征和行为完全一致。而HTTPS流量占据了目前互联网流量的一半以上，且TLS握手成功后流量均为密文，几乎不存在可行方法从其中分辨出Trojan协议流量。

对于主动检测，当防火墙主动连接Trojan服务器进行检测时，Trojan可以正确识别非Trojan协议的流量。与Shadowsocks等代理不同的是，此时Trojan不会断开连接，而是将这个连接代理到一个正常的Web服务器。在GFW看来，该服务器的行为和一个普通的HTTPS网站行为完全相同，无法判断是否是一个Trojan代理节点。这也是Trojan推荐使用合法的域名、使用权威CA签名的HTTPS证书的原因: 这让你的服务器完全无法被GFW使用主动检测判定是一个Trojan服务器。

因此，就目前的情况来看，若要识别并阻断Trojan的连接，只能使用无差别封锁（封锁某个IP段，某一类证书，某一类域名，甚至阻断全国所有出境HTTPS连接）或发动大规模的中间人攻击（劫持所有TLS流量并劫持证书，审查内容）。对于中间人攻击，可以使用Websocket的双重TLS应对，高级配置中有详细讲解。


---
title: "基本配置"
draft: false
weight: 20
---

这一部分内容将介绍如何配置基本的Trojan-Go代理服务器和客户端。


---
title: "API开发"
draft: false
weight: 100
---

Trojan-Go基于gRPC实现了API，使用protobuf交换数据。客户端可获取流量和速度信息；服务端可获取各用户流量，速度，在线情况，并动态增删用户和限制速度。可以通过在配置文件中添加```goapi```选项激活API模块。下面是一个例子，各字段含义参见“完整的配置文件”一节。

```gojson
...
"api": {
    "enabled": true,
    "api_addr": "0.0.0.0",
    "api_port": 10000,
    "ssl": {
      "enabled": true,
      "cert": "api_cert.crt",
      "key": "api_key.key",
      "verify_client": true,
      "client_cert": [
          "api_client_cert1.crt",
          "api_client_cert2.crt"
      ]
    },
}
```

如果需要实现API客户端进行对接，请参考api/service/api.proto文件。


---
title: "编译和自定义Trojan-Go"
draft: false
weight: 10
---

编译需要Go版本号高于1.14.x，请在编译前确认你的编译器版本。推荐使用snap安装和更新go。

编译方式非常简单，可以使用Makefile预设步骤进行编译：

```goshell
make
make install #安装systemd服务等，可选
```

或者直接使用Go进行编译：

```goshell
go build -tags "full" #编译完整版本
```

可以通过指定GOOS和GOARCH环境变量，指定交叉编译的目标操作系统和架构，例如

```goshell
GOOS=windows GOARCH=386 go build -tags "full" #windows x86
GOOS=linux GOARCH=arm64 go build -tags "full" #linux arm64
```

你可以使用release.sh进行批量的多个平台的交叉编译，release版本使用了这个脚本进行构建。

Trojan-Go的大多数模块是可插拔的。在build文件夹下可以找到各个模块的导入声明。如果你不需要其中某些功能，或者需要缩小可执行文件的体积，可以使用构建标签(tags)进行模块的自定义，例如

```goshell
go build -tags "full" #编译所有模块
go build -tags "client" -trimpath -ldflags="-s -w -buildid=" #只有客户端功能，且去除符号表缩小体积
go build -tags "server mysql" #只有服务端和mysql支持
```

使用full标签等价于

```goshell
go build -tags "api client server forward nat other"
```


---
title: "多路复用"
draft: false
weight: 30
---

Trojan-Go使用[smux](https://github.com/xtaci/smux)实现多路复用。同时实现了simplesocks协议用于进行代理传输。

当启用多路复用时，客户端首先发起TLS连接，使用正常trojan协议格式，但协议Command部分填入0x7f(protocol.Mux)，标识此连接为复用连接（类似于http的upgrade），之后连接交由smux客户端管理。服务器收到请求头部后，交由smux服务器解析该连接的所有流量。在每条分离出的smux连接上，使用simplesocks协议（去除认证部分的trojan协议)标明代理目的地。自顶向下的协议栈如下：

| 协议        | 备注     |
| ----------- | -------- |
| 真实流量    |
| SimpleSocks |
| smux        |
| Trojan      | 用于鉴权 |
| 底层协议    |          |


---
title: "基本介绍"
draft: false
weight: 1
---

Trojan-Go的核心部分有

- tunnel 各个协议具体实现

- proxy 代理核心

- config 配置注册和解析模块

- redirector 主动检测欺骗模块

- statistics 用户认证和统计模块

可以在对应文件夹中找到相关源代码。

## tunnel.Tunnel隧道

Trojan-Go将所有协议（包括路由功能等）抽象为隧道(tunnel.Tunnel接口)，每个隧道可开启服务端（tunnel.Server接口）和客户端（tunnel.Client）。每个服务端可以从其底层隧道中，剥离并接受流（tunnel.Conn）和包（tunnel.PacketConn)。客户端可以向底层隧道，创建流和包。

每个隧道并不关心其下方的隧道是什么，但是每个隧道清楚知道这个它上方的其他隧道的相关信息。

所有隧道需要下层提供流或包传输支持，或两者都要求提供。所有隧道必须向上层隧道提供流传输支持，但不一定提供包传输。

隧道可能只有服务端，也可能只有客户端，也可能两者皆有。两者皆有的隧道，可被用于作为Trojan-Go客户端和服务端间的传输隧道。

注意，请区分Trojan-Go的服务端/客户端，和隧道的服务端/客户端的区别。下面是一个方便理解的图例。

```gotext

  入站                              GFW                                  出站
-------->隧道A服务端->隧道B客户端 ----------------> 隧道B服务端->隧道C客户端----------->
           (Trojan-Go客户端)                          (Trojan-Go服务端)

```

最底层的隧道为传输层，即不从其他隧道获取或者创建流和包的隧道，充当上图中隧道A或者C的角色。

- transport，可插拔传输层

- socks，socks5代理，仅隧道服务端
  
- tproxy，透明代理，仅隧道服务端

- dokodemo，反向代理，仅隧道服务端

- freedom，自由出站，仅隧道客户端

这几个隧道直接从TCP/UDP Socket创建流和包，不接受为其底层添加的任何隧道。

其他隧道，只要下层能满足上层对包和流传输的需求，则原则上可以任何方式，任何数量进行组合和堆叠。这些隧道在上图中充当隧道B的角色，他们有

- trojan

- websocket

- mux

- simplesocks

- tls

- router，路由功能，仅隧道客户端

他们都不关心其下层隧道实现。但可以根据到来的流和包，将其分发给上层隧道。

例如，在这张图中，是一个典型的Trojan-Go客户端和服务端，各个隧道自下往上堆叠的顺序是：

- 隧道A: transport->socks

- 隧道B: transport->tls->trojan

- 隧道C: freedom

实际上的隧道堆叠的情况会比这个更复杂一些。通常的入站的隧道是一棵多叉树的形式，而非一条链。具体解释参考下文。

## proxy.Proxy代理核心

代理核心的作用，是监听上述隧道进行组合堆叠并形成的协议栈，将所有的入站协议栈（多个隧道Server的终端节点，见下）中抽取的流和包，以及对应元信息，转送给出站协议栈（一个隧道Client）。

注意，这里的入站协议栈可以有多个，如客户端可以同时从Socks5和HTTP协议栈中抽取流和包，服务端可以同时从Websocket承载的Trojan协议，和TLS承载的Trojan协议中抽取流和包等。但是出站协议栈只能有一个，如只使用TLS承载的Trojan协议出站。

为了描述入站协议栈（隧道服务端）的组合和堆叠方式，使用一棵多叉树对所有协议栈进行描述。你可以在proxy文件夹中各组件，看到构建树的过程。

而出站协议栈则比较简单，使用一个简单列表即可描述。

所以实际上，对于一个典型的开启了Websocket和Mux的客户端/服务端，上图的隧道堆叠模型为：

客户端

- 入站（树）
  - transport (根)
    - adapter 能够识别HTTP和Socks流量并分发给上层协议
      - http （终端节点）
      - socks（终端节点）

- 出站(链)
  - transport (根)
  - tls
  - websocket
  - trojan
  - mux
  - simplesocks

服务端

- 入站（树）
  - transport (根)
    - tls 能够识别HTTP和非HTTP流量并分发
      - websocket
        - trojan（终端节点）
          - mux
            - simplesocks （终端节点）
      - trojan 能够识别mux和普通trojan流量并分发（终端节点）
        - mux
          - simplesocks （终端节点）

- 出站（链）
  - freedom

注意，代理核心只从隧道构成的树的终端节点抽取流和包，并转送到唯一的出站上。多个终端节点的设计的目的，是使Trojan-Go同时兼容Websocket和Trojan协议入站连接，开启/未开启Mux的入站连接，以及HTTP/Socks5自动识别的功能。每个拥有多个儿子的树上节点，具有精确识别和分发流和包给不同的儿子节点的能力。这符合我们假定每个协议了解其上层承载协议的假设。


---
title: "可插拔传输层插件开发"
draft: false
weight: 150
---

Trojan-Go鼓励开发传输层插件，以丰富协议类型，增加与GFW对抗的战略纵深。

传输层插件的作用，是替代tansport隧道的TLS进行传输加密和混淆。

插件与Trojan-Go基于TCP Socket通讯，与Trojan-Go本身不存在任何耦合关系，你可以使用任何你喜欢的语言和设计模式进行开发。我们建议的参照[SIP003](https://shadowsocks.org/en/spec/Plugin.html)标准进行开发。如此开发的插件可以同时用于Trojan-Go和Shadowsocks。

Trojan-Go开启插件功能后，仅使用TCP进行传输（明文）。你的插件只需要处理入站的TCP请求即可。你可以将这些TCP流量转换成任何你喜欢的流量格式，如QUIC，HTTP，甚至是ICMP。

Trojan-Go插件设计原则，与Shadowsocks略有不同：

1. 插件本身可以对传输内容进行加密，混淆和完整性校验，以及可以抵抗重放攻击。

2. 插件应该伪造一种已有的、常见的服务（记做X服务），及其流量，在此基础上嵌入自己的加密内容。

3. 服务端的插件，在检验到内容被篡改/遭到重放时，**必须将此连接交由Trojan-Go处理**。具体步骤是，将已读入和未读入的内容一并发送给Trojan-Go，并建立双向连接，而不是直接断开。Trojan-Go将与一个真实的X服务器建立连接，使攻击者直接与真实的X服务器进行交互。

解释如下：

第一点原则，是由于Trojan协议本身并不加密。将TLS替换为传输层插件后，将**完全信任插件的安全性**。

第二点原则，是继承Trojan的精神。最适合隐藏一棵树的地方，是森林。

第三点原则，是为了充分利用Trojan-Go的抗主动探测特性。即使GFW对你的服务器进行主动探测，你的服务器也可以表现得与X服务一致，而不存在其他特征。

为了方便理解，举一个例子。

1. 假设你的插件伪装的是MySQL流量。防火墙通过流量嗅探，发现你的MySQL流量大得异常，决定主动连接你的服务器进行主动探测。

2. 防火墙连接到你的服务器并发送探测载荷，你的Trojan-Go服务端插件，经过校验，发现这个异常连接不是代理流量，于是将这个连接交由Trojan-Go处理。

3. Trojan-Go发现这个连接异常，将这个连接重定向到一个真正的MySQL服务器上。于是，防火墙开始与一个真正的MySQL服务器进行交互，发现其行为与真实MySQL服务器无异，无法对服务器进行封禁。

另外，即使你的协议协议和插件不能满足原则2和原则3，甚至不能很好满足原则1，我们同样鼓励开发。因为GFW仅仅针对流行的协议进行审计和封锁，此类协议（土制密码学/土制协议）只要不公开发表，同样能保持非常强健的生命力。


---
title: "SimpleSocks协议"
draft: false
weight: 50
---

SimpleSocks协议是无鉴权机制的简单代理协议，本质上是去除了sha224的Trojan协议。使用该协议的目的是减少多路复用时的overhead。

只有启用多路复用之后，被复用的连接才会使用这个协议。也即SimpleSocks总是被SMux承载。

SimpleSocks甚至比Socks5更简单，下面是头部结构。

```gotext
+-----+------+----------+----------+-----------+
| CMD | ATYP | DST.ADDR | DST.PORT |  Payload  |
+-----+------+----------+----------+-----------+
|  1  |  1   | Variable |    2     |  Variable |
+-----+------+----------+----------+-----------+
```

各字段定义与Trojan协议相同，不再赘述。


---
title: "Trojan协议"
draft: false
weight: 20
---

Trojan-Go遵循原始的trojan协议，具体格式可以参考[Trojan文档](https://trojan-gfw.github.io/trojan/protocol)，这里不再赘述。

默认情况下，trojan协议使用TLS来承载，协议栈如下：

| 协议     |
| -------- |
| 真实流量 |
| Trojan   |
| TLS      |
| TCP      |


---
title: "URL方案（草案）"
draft: false
weight: 200
---

## Changelog

- encryption 格式修改为 ss;method:password

## 概述

感谢 @DuckSoft @StudentMain @phlinhng 对 Trojan-Go URL 方案的讨论和贡献。**目前 URL 方案为草案，需要更多的实践和讨论。**

Trojan-Go**客户端**可以接受URL，以定位服务器资源。原则如下:

- 遵守 URL 格式规范

- 保证人类可读，对机器友好

- URL 的作用，是定位 Trojan-Go 节点资源，方便资源分享

需要注意，基于人类可读性的考虑，禁止将 base64 等编码数据嵌入 URL 中。首先， base64 编码不能保证传输安全，其意义在于在 ASCII 信道传输非 ASCII 数据。其次，如果需要保证分享 URL 时的传输安全，请对明文 URL 进行加密，而不是修改 URL 格式。

## 格式

基本格式如下，`$()` 代表此处需要 `encodeURIComponent`。

```gotext
trojan-go://
    $(trojan-password)
    @
    trojan-host
    :
    port
/?
    sni=$(tls-sni.com)&
    type=$(original|ws|h2|h2+ws)&
        host=$(websocket-host.com)&
        path=$(/websocket/path)&
    encryption=$(ss;aes-256-gcm;ss-password)&
    plugin=$(...)
#$(descriptive-text)
```

例如

```gotext
trojan-go://password1234@google.com/?sni=microsoft.com&type=ws&host=youtube.com&path=%2Fgo&encryption=ss%3Baes-256-gcm%3Afuckgfw
```

由于 Trojan-Go 兼容 Trojan，所以对于 Trojan 的 URL 方案

```gotext
trojan://password@remote_host:remote_port
```

可以兼容接受。它等价于

```gotext
trojan-go://password@remote_host:remote_port
```

需要注意的是，一旦服务器使用了非Trojan兼容的功能，必须使用```gotrojan-go://```定位服务器。这样设计的目的是使得 Trojan-Go 的 URL 不会被 Trojan 错误接受，避免污染 Trojan 用户的 URL 分享。同时，Trojan-Go 确保可以兼容接受 Trojan 的 URL。

## 详述

注意：所有参数名和常数字符串均区分大小写。

### `trojan-password`

Trojan 的密码。
不可省略，不能为空字符串，不建议含有非 ASCII 可打印字符。
必须使用 `encodeURIComponent` 编码。

### `trojan-host`

节点 IP / 域名。
不可省略，不能为空字符串。
IPv6 地址必须扩方括号。
IDN 域名（如“百度.cn”）必须使用 `xn--xxxxxx` 格式。

### `port`

节点端口。
省略时默认为 `443`。
必须取 `[1,65535]` 中的整数。

### `tls`或`allowInsecure`

没有这个字段。
TLS 默认一直启用，除非有传输插件禁用它。
TLS 认证必须开启。无法使用根CA校验服务器身份的节点，不适合分享。

### `sni`

自定义 TLS 的 SNI。
省略时默认与 `trojan-host` 同值。不得为空字符串。

必须使用 `encodeURIComponent` 编码。

### `type`

传输类型。
省略时默认为 `original`，但不可为空字符串。
目前可选值只有 `original` 和 `ws`，未来可能会有 `h2`、`h2+ws` 等取值。

当取值为 `original` 时，使用原始 Trojan 传输方式，无法方便通过 CDN。
当取值为 `ws` 时，使用 Websocket over TLS 传输。

### `host`

自定义 HTTP `Host` 头。
可以省略，省略时值同 `trojan-host`。
可以为空字符串，但可能带来非预期情形。

警告：若你的端口非标准端口（不是 80 / 443），RFC 标准规定 `Host` 应在主机名后附上端口号，例如 `example.com:44333`。至于是否遵守，请自行斟酌。

必须使用 `encodeURIComponent` 编码。

### `path`

当传输类型 `type` 取 `ws`、`h2`、`h2+ws` 时，此项有效。
不可省略，不可为空。
必须以 `/` 开头。
可以使用 URL 中的 `&` `#` `?` 等字符，但应当是合法的 URL 路径。

必须使用 `encodeURIComponent` 编码。

### `mux`

没有这个字段。
当前服务器默认一直支持 `mux`。
启用 `mux` 与否各有利弊，应由客户端决定自己是否启用。URL的作用，是定位服务器资源，而不是规定用户使用偏好。

### `encryption`

用于保证 Trojan 流量密码学安全的加密层。
可省略，默认为 `none`，即不使用加密。
不可以为空字符串。

必须使用 encodeURIComponent 编码。

使用 Shadowsocks 算法进行流量加密时，其格式为：

```gotext
ss;method:password
```

其中 ss 是固定内容，method 是加密方法，必须为下列之一：

- `aes-128-gcm`
- `aes-256-gcm`
- `chacha20-ietf-poly1305`

其中的 `password` 是 Shadowsocks 的密码，不得为空字符串。
`password` 中若包含分号，不需要进行转义。
`password` 应为英文可打印 ASCII 字符。

其他加密方案待定。

### `plugin`

额外的插件选项。本字段保留。
可省略，但不可以为空字符串。

### URL Fragment (# 后内容)

节点说明。
不建议省略，不建议为空字符串。

必须使用 `encodeURIComponent` 编码。


---
title: "Websocket"
draft: false
weight: 40
---

由于使用CDN中转时，HTTPS对CDN透明，CDN可以审查Websocket传输内容。而Trojan协议本身是明文传输，因此为保证安全性，可添加一层Shadowsocks AEAD加密层以混淆流量特征并保证安全性。

**如果你使用的是中国境内运营商提供的CDN，请务必开启AEAD加密**

开启AEAD加密后，Websocket承载的流量将被Shadowsocks AEAD加密，头部具体格式参见Shadowsocks白皮书。

开启Websocket支持后，协议栈如下：

| 协议        | 备注             |
| ----------- | ---------------- |
| 真实流量    |                  |
| SimpleSocks | 如果开启多路复用 |
| smux        | 如果开启多路复用 |
| Trojan      |                  |
| Shadowsocks | 如果开启加密     |
| Websocket   |                  |
| 传输层协议  |                  |


---
title: "实现细节和开发指南"
draft: false
weight: 40
---

这一部分介绍Trojan-Go底层实现的细节，主要面向开发者。


# `easy/easy.go`

这是一个用Go编程语言编写的简单Easy客户端命令行工具。它主要用于在远程服务器上验证身份，并在客户端和服务器之间建立安全通信。

具体来说，这个工具包含以下功能：

1. 服务器设置：设置服务器是否运行（true表示开启，false表示关闭），设置客户端是否运行（true表示开启，false表示关闭），设置密码（json字符串），设置本地用户名（string类型），设置远程服务器地址（string类型），设置服务器证书（string类型），设置服务器私钥（string类型）。

2. 客户端连接：连接服务器，设置连接超时时间（int类型），用于在连接成功后进行心跳检测。

3. 发送消息：将接收到的消息发送给服务器。

4. 验证消息：检查消息是否包含有效的令牌（token）。

5. 设置日志：设置是否在输出的消息中包含日志信息（true表示开启，false表示关闭）。

6. 设置选项：设置是否使用代理服务器（true表示开启，false表示关闭），设置代理服务器的URL（string类型）。

7. 错误处理：在连接失败或接收到无效消息时，打印错误并退出。

8. 帮助：设置是否有帮助信息（true表示开启，false表示关闭）。

这个工具主要用于在本地开发一个简单的身份验证系统，并在实际应用中进行远程服务器验证。


```go
package easy

import (
	"encoding/json"
	"flag"
	"net"
	"strconv"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/option"
	"github.com/p4gefau1t/trojan-go/proxy"
)

type easy struct {
	server   *bool
	client   *bool
	password *string
	local    *string
	remote   *string
	cert     *string
	key      *string
}

```

该代码定义了一个名为 ClientConfig 的结构体类型，它用于配置客户端程序的运行类型、本地地址、本地端口、远程地址和远程端口，以及用户名密码。

接着定义了一个名为 TLS 的结构体类型，它用于配置 TLS 证书的 SNI、证书和私钥。

最后，该代码定义了一个名为 Main 的函数，它接收一个名为 ClientConfig 的参数，然后将该参数的值存储到一个 ClientConfig 类型的变量中，并将该变量输出到 json。然后，它又接收一个名为 TLS 的参数，然后将该参数的值存储到一个 TLS 类型的变量中，并将该变量输出到 json。


```go
type ClientConfig struct {
	RunType    string   `json:"run_type"`
	LocalAddr  string   `json:"local_addr"`
	LocalPort  int      `json:"local_port"`
	RemoteAddr string   `json:"remote_addr"`
	RemotePort int      `json:"remote_port"`
	Password   []string `json:"password"`
}

type TLS struct {
	SNI  string `json:"sni"`
	Cert string `json:"cert"`
	Key  string `json:"key"`
}

```

This is a Go program that creates a Kubernetes proxy server that enables users to access a backend service as if they were directly connected to it. The program is using the `net/http` and `net/url` packages to handle the actual HTTP/HTTPS connections and the `encoding/json` package to handle the JSON data structures used in the program.

The program has three arguments:

* `*o.local`: The local IP address and port of the backend service that the client is trying to connect to.
* `*o.remote`: The remote IP address and port of the backend service that the client is trying to connect to.
* `*o.password`: A list of valid passwords for authentication.

The program first checks if the arguments are valid and then splits the local and remote addresses and ports into their respective variables. It then sets up a `ServerConfig` object with the type of server and the connection information, and then converts the `ServerConfig` object to a JSON string and marshals it.

Finally, it creates a `Proxy` object from the `encoding/json` package and runs the `Run` method of the `Proxy` object, passing in the JSON-formatted `ServerConfig` object. If any errors occur during this process, they are logged and the program exits with a non-zero exit code.


```go
type ServerConfig struct {
	RunType    string   `json:"run_type"`
	LocalAddr  string   `json:"local_addr"`
	LocalPort  int      `json:"local_port"`
	RemoteAddr string   `json:"remote_addr"`
	RemotePort int      `json:"remote_port"`
	Password   []string `json:"password"`
	TLS        `json:"ssl"`
}

func (o *easy) Name() string {
	return "easy"
}

func (o *easy) Handle() error {
	if !*o.server && !*o.client {
		return common.NewError("empty")
	}
	if *o.password == "" {
		log.Fatal("empty password is not allowed")
	}
	log.Info("easy mode enabled, trojan-go will NOT use the config file")
	if *o.client {
		if *o.local == "" {
			log.Warn("client local addr is unspecified, using 127.0.0.1:1080")
			*o.local = "127.0.0.1:1080"
		}
		localHost, localPortStr, err := net.SplitHostPort(*o.local)
		if err != nil {
			log.Fatal(common.NewError("invalid local addr format:" + *o.local).Base(err))
		}
		remoteHost, remotePortStr, err := net.SplitHostPort(*o.remote)
		if err != nil {
			log.Fatal(common.NewError("invalid remote addr format:" + *o.remote).Base(err))
		}
		localPort, err := strconv.Atoi(localPortStr)
		if err != nil {
			log.Fatal(err)
		}
		remotePort, err := strconv.Atoi(remotePortStr)
		if err != nil {
			log.Fatal(err)
		}
		clientConfig := ClientConfig{
			RunType:    "client",
			LocalAddr:  localHost,
			LocalPort:  localPort,
			RemoteAddr: remoteHost,
			RemotePort: remotePort,
			Password: []string{
				*o.password,
			},
		}
		clientConfigJSON, err := json.Marshal(&clientConfig)
		common.Must(err)
		log.Info("generated config:")
		log.Info(string(clientConfigJSON))
		proxy, err := proxy.NewProxyFromConfigData(clientConfigJSON, true)
		if err != nil {
			log.Fatal(err)
		}
		if err := proxy.Run(); err != nil {
			log.Fatal(err)
		}
	} else if *o.server {
		if *o.remote == "" {
			log.Warn("server remote addr is unspecified, using 127.0.0.1:80")
			*o.remote = "127.0.0.1:80"
		}
		if *o.local == "" {
			log.Warn("server local addr is unspecified, using 0.0.0.0:443")
			*o.local = "0.0.0.0:443"
		}
		localHost, localPortStr, err := net.SplitHostPort(*o.local)
		if err != nil {
			log.Fatal(common.NewError("invalid local addr format:" + *o.local).Base(err))
		}
		remoteHost, remotePortStr, err := net.SplitHostPort(*o.remote)
		if err != nil {
			log.Fatal(common.NewError("invalid remote addr format:" + *o.remote).Base(err))
		}
		localPort, err := strconv.Atoi(localPortStr)
		if err != nil {
			log.Fatal(err)
		}
		remotePort, err := strconv.Atoi(remotePortStr)
		if err != nil {
			log.Fatal(err)
		}
		serverConfig := ServerConfig{
			RunType:    "server",
			LocalAddr:  localHost,
			LocalPort:  localPort,
			RemoteAddr: remoteHost,
			RemotePort: remotePort,
			Password: []string{
				*o.password,
			},
			TLS: TLS{
				Cert: *o.cert,
				Key:  *o.key,
			},
		}
		serverConfigJSON, err := json.Marshal(&serverConfig)
		common.Must(err)
		log.Info("generated json config:")
		log.Info(string(serverConfigJSON))
		proxy, err := proxy.NewProxyFromConfigData(serverConfigJSON, true)
		if err != nil {
			log.Fatal(err)
		}
		if err := proxy.Run(); err != nil {
			log.Fatal(err)
		}
	}
	return nil
}

```

这段代码定义了一个名为`func`的函数，它接收一个名为`easy`的参数，并返回一个整数类型的值。

接下来，定义了一个名为`init`的函数，它执行以下操作：

1. 注册一个名为`easy`的匿名函数，它接收一个包含各个参数的字符串，并将其赋值给一个名为`option`的变量。

2. 在`option`的函数体中，使用`RegisterHandler`方法将`easy`函数注册为当前选项中的一个处理程序。

3. 在`easy`函数中，定义了以下常量：

	- `server`：服务器选项，默认为false。
	- `client`：客户端选项，默认为false。
	- `password`：密码选项，默认为空字符串。
	- `remote`：远程选项，默认为空字符串。
	- `local`：本地选项，默认为空字符串。
	- `key`：服务器私钥选项，默认为空字符串。
	- `cert`：服务器证书选项，默认为空字符串。

4. 在`func`函数中，返回了`50`。


```go
func (o *easy) Priority() int {
	return 50
}

func init() {
	option.RegisterHandler(&easy{
		server:   flag.Bool("server", false, "Run a trojan-go server"),
		client:   flag.Bool("client", false, "Run a trojan-go client"),
		password: flag.String("password", "", "Password for authentication"),
		remote:   flag.String("remote", "", "Remote address, e.g. 127.0.0.1:12345"),
		local:    flag.String("local", "", "Local address, e.g. 127.0.0.1:12345"),
		key:      flag.String("key", "server.key", "Key of the server"),
		cert:     flag.String("cert", "server.crt", "Certificates of the server"),
	})
}

```

# `log/log.go`

这段代码定义了一个名为“log”的包，其中定义了一个名为“LogLevel”的枚举类型，表示日志的级别，包括输出额度从0到5的各种级别，以及一个名为“AllLevel”的枚举类型，表示将所有日志输出。

该代码还导入了两个标准库中的“io”和“os”包，以及定义了一个名为“信息”的函数，其作用是输出指定的日志级别，函数的参数为“LogLevel”类型和“输出方式”参数。

该代码的最后两行定义了一个名为“Level”的变量，使用类型注解将其声明为整数类型，并定义了一个包含所有日志级别的常量“LogLevel”。


```go
package log

import (
	"io"
	"os"
)

// LogLevel how much log to dump
// 0: ALL; 1: INFO; 2: WARN; 3: ERROR; 4: FATAL; 5: OFF
type LogLevel int

const (
	AllLevel   LogLevel = 0
	InfoLevel  LogLevel = 1
	WarnLevel  LogLevel = 2
	ErrorLevel LogLevel = 3
	FatalLevel LogLevel = 4
	OffLevel   LogLevel = 5
)

```

这是一段定义了一个名为Logger的接口，定义了多个函数，包括Fatal、Fatalf、Error、Errorf、Warn、Warnf、Info、Infof、Debug、Debugf、Trace、Tracef和SetLogLevel、SetOutput。

具体来说，这段代码定义了一个Logger接口，其中Fatal、Fatalf、Error、Errorf、Warn、Warnf、Info、Infof、Debug、Debugf、Trace、Tracef和SetLogLevel、SetOutput都是该接口的成员函数。这些函数可以用来输出不同级别的错误或警告信息，以及设置日志记录器的级别和输出目标。

例如，如果在日志记录器设置为Fatalf级别时，使用了Fatal函数，则会对调用该函数的开发者或系统进行严重警告，并输出详细信息。而如果设置了日志记录器级别为Warn级别，则Warn函数将会输出警告信息，但不会对系统进行任何操作。

通过调用这些函数，可以实现对日志记录的管理，以便在程序运行过程中更好地诊断和解决问题。


```go
type Logger interface {
	Fatal(v ...interface{})
	Fatalf(format string, v ...interface{})
	Error(v ...interface{})
	Errorf(format string, v ...interface{})
	Warn(v ...interface{})
	Warnf(format string, v ...interface{})
	Info(v ...interface{})
	Infof(format string, v ...interface{})
	Debug(v ...interface{})
	Debugf(format string, v ...interface{})
	Trace(v ...interface{})
	Tracef(format string, v ...interface{})
	SetLogLevel(level LogLevel)
	SetOutput(io.Writer)
}

```

这段代码定义了一个名为Logger的类，该类继承自名为EmptyLogger的内部类。Logger类包含了一些方法，用于设置日志级别、输出日志信息、捕获异常并记录错误信息。

具体来说，以下代码详细解释了每个方法的含义：

javascript
var logger Logger = &EmptyLogger{}


这段代码定义了一个名为logger的变量，并将其初始化为一个名为EmptyLogger的内部类实例。这个内部类可能是一个实现了Logger接口的类，它包含了一些通用的方法，如SetLogLevel、Fatal、Error、Warn等。
csharp
type EmptyLogger struct{}


这段代码定义了一个名为EmptyLogger的内部类，该类继承自匿名类型。这个类的成员变量为空，因此它的类型无法被编译器识别。但是，在实际应用中，EmptyLogger可能是一个具体的日志类，它包含了一些通用的方法，用于在日志中记录和输出信息。
java
func (l *EmptyLogger) SetLogLevel(LogLevel) {}


这段代码定义了一个名为l的指针变量，该变量被赋值为指向一个名为EmptyLogger的内部类的引用。这个方法接收一个名为LogLevel的整数参数，表示日志级别。设置日志级别可以用于控制日志的输出策略，例如，将LogLevel设置为LogLevel.Fatal时，如果日志中的任何错误信息导致程序崩溃，那么控制程序的继续执行。
java
func (l *EmptyLogger) Fatal(v ...interface{}) { os.Exit(1) }


这段代码定义了一个名为Fatal的静态方法，该方法接收多个参数，包括一个整数和一个可传递给os.Exit()的整数。这个方法的参数v...interface{}表示，这些参数将被传递给os.Exit()函数，如果这个函数在当前日志级别下无法继续执行，那么程序将崩溃。这个方法在没有日志级别设置时，不会输出任何信息，仅仅是抛出一个不可恢复的异常。
scss
func (l *EmptyLogger) Fatalf(format string, v ...interface{}) { os.Exit(1) }


这段代码定义了一个名为Fatalf的静态方法，该方法接收一个格式字符串和一个可传递给os.Exit()的整数和一个可传递给os.Exit()的整数。这个方法的参数v...interface{}表示，这些参数将被传递给os.Exit()函数，如果这个函数在当前日志级别下无法继续执行，那么程序将崩溃。这个方法接收的格式字符串是字符串，而不是字符串Lang，因此可以包含任意数量的参数。
scss
func (l *EmptyLogger) Error(v ...interface{}) { }


这段代码定义了一个名为Error的静态方法，该方法接收一个可传递给os.Exit()的整数和一个可传递给os.Exit()的整数。这个方法的参数v...interface{}表示，这些参数将被传递给os.Exit()函数，如果这个函数在当前日志级别下无法继续执行，那么程序将崩溃。
scss
func (l *EmptyLogger) Errorf(format string, v ...interface{}) { os.Exit(1) }


这段代码定义了一个名为Errorf的静态方法，该方法接收一个格式字符串和一个可传递给os.Exit()的整数和一个可传递给os.Exit()的整数。这个方法的参数v...interface{}表示，这些参数将被传递给os.Exit()函数，如果这个函数在当前日志级别下无法继续执行，那么程序将崩溃。这个方法的格式字符串是字符串，并且其中的%v表示一个可变参数。这个方法的参数v...interface{}表示，这些参数将被传递给os.Exit()函数，如果这个函数在当前日志级别下无法继续执行，那么程序将崩溃。


```go
var logger Logger = &EmptyLogger{}

type EmptyLogger struct{}

func (l *EmptyLogger) SetLogLevel(LogLevel) {}

func (l *EmptyLogger) Fatal(v ...interface{}) { os.Exit(1) }

func (l *EmptyLogger) Fatalf(format string, v ...interface{}) { os.Exit(1) }

func (l *EmptyLogger) Error(v ...interface{}) {}

func (l *EmptyLogger) Errorf(format string, v ...interface{}) {}

func (l *EmptyLogger) Warn(v ...interface{}) {}

```

这是一段 Go 语言中的函数指针类型，表示了一个名为 `func` 的函数，接受一个名为 `l` 的参数，并返回一个指向 `EmptyLogger` 类型的变量 `*l`。

函数指针 `*l` 代表了一个空 `EmptyLogger` 类型的变量，这个 `EmptyLogger` 类型可能用来记录日志，但是没有任何实际的日志记录。

函数 `func` 接收两个参数 `format` 和 `v...interface{}`。其中 `format` 是字符串，表示日志格式，`v...interface{}` 是 `v` 和其他 `interface{}` 类型的参数，可能是日志的其他状态信息，但具体是什么信息不确定。

函数体中，对于不同的日志级别 `Info`, `Warnf`, `Info`, `Infof`, `Debug`, `Debugf`, `Trace`, `Tracef`, `SetOutput` 函数，会调用 `l` 中的 `Info`、 `Warnf`、 `Info`、 `Infof`、 `Debug`、 `Debugf`、 `Trace` 和 `Tracef` 函数，分别传递 `v...interface{}` 参数，并将它们转化为日志格式 string 和日志级别、状态信息等，然后输出到指定的输出设备 `w`。


```go
func (l *EmptyLogger) Warnf(format string, v ...interface{}) {}

func (l *EmptyLogger) Info(v ...interface{}) {}

func (l *EmptyLogger) Infof(format string, v ...interface{}) {}

func (l *EmptyLogger) Debug(v ...interface{}) {}

func (l *EmptyLogger) Debugf(format string, v ...interface{}) {}

func (l *EmptyLogger) Trace(v ...interface{}) {}

func (l *EmptyLogger) Tracef(format string, v ...interface{}) {}

func (l *EmptyLogger) SetOutput(w io.Writer) {}

```

这段代码定义了三个函数，分别是Error、Errorf和Warn，它们都使用了名为logger的标头。

函数Error接收一个可变参数v...interface{}，表示将多个interface{}类型的变量作为参数传递给logger，然后将它们都转换为错误信息并打印到控制台。

函数Errorf接收一个格式字符串和可变参数v...interface{}，表示将多个interface{}类型的变量作为参数传递给logger，然后使用格式字符串将它们都格式化，再将格式化后的信息打印到控制台。

函数Warn接收一个可变参数v...interface{}，表示将多个interface{}类型的变量作为参数传递给logger，然后将它们都打印到控制台，并在信息中加入"Warning"。

函数Warnf接收一个格式字符串和一个可变参数v...interface{}，表示将多个interface{}类型的变量作为参数传递给logger，然后使用格式字符串将格式化后的信息打印到控制台。


```go
func Error(v ...interface{}) {
	logger.Error(v...)
}

func Errorf(format string, v ...interface{}) {
	logger.Errorf(format, v...)
}

func Warn(v ...interface{}) {
	logger.Warn(v...)
}

func Warnf(format string, v ...interface{}) {
	logger.Warnf(format, v...)
}

```

这段代码定义了三个函数，分别是Info、Infof和Debugf，以及一个参数为整数类型的函数Info。

Info函数接收一个或多个接口类型的参数，将它们传递给logger.Info方法，用于向logger输出一条简短的信息。

Infof函数接收一个格式化字符串和一个或多个接口类型的参数，将它们传递给logger.Infof方法，用于向logger输出一个更详细的错误信息。

Dbg函数接收一个或多个接口类型的参数，将它们传递给logger.Debug方法，用于向logger输出一条简短的信息。

Info函数还有一个参数为整数的函数Info，用于将整数类型的参数传递给logger.Info方法，以获取更多的信息，如调试信息。

总结一下，这些函数用于在程序中记录日志信息，包括简短的信息、详细的错误信息和简短的信息以及更多的调试信息。通过这些函数，我们可以方便地将日志信息记录下来，然后在程序中的调试和错误处理过程中使用它们。


```go
func Info(v ...interface{}) {
	logger.Info(v...)
}

func Infof(format string, v ...interface{}) {
	logger.Infof(format, v...)
}

func Debug(v ...interface{}) {
	logger.Debug(v...)
}

func Debugf(format string, v ...interface{}) {
	logger.Debugf(format, v...)
}

```

这三段代码都是使用了Go标准库中的`logger`包，用于输出日志信息。

`Trace`函数接受一个或多个`interface{}`类型的参数，将其传递给`logger.Trace`函数，用于输出日志信息。

`Tracef`函数与`Trace`函数类似，但其接受了一个格式字符串和一个或多个`interface{}`类型的参数，然后使用`logger.Tracef`函数来输出日志信息，其中格式字符串指定要输出的信息。

`Fatal`函数与`Fatal`函数不同，其接受了两个或多个`interface{}`类型的参数，将其传递给`logger.Fatal`函数，用于输出错误信息。

`Fatalf`函数与`Fatal`函数类似，但其接受了一个格式字符串和一个或多个`interface{}`类型的参数，然后使用`logger.Fatalf`函数来输出错误信息，其中格式字符串指定要输出的信息。


```go
func Trace(v ...interface{}) {
	logger.Trace(v...)
}

func Tracef(format string, v ...interface{}) {
	logger.Tracef(format, v...)
}

func Fatal(v ...interface{}) {
	logger.Fatal(v...)
}

func Fatalf(format string, v ...interface{}) {
	logger.Fatalf(format, v...)
}

```

这段代码定义了三个函数，用于在日志记录功能中设置日志级别、输出内容和注册一个logger实例。

第一个函数 `SetLogLevel` 接受一个日志级别参数，并将其设置为当前的日志级别。它使用 `logger.SetLogLevel` 方法将设置好的日志级别存储在当前的logger实例中，然后返回。

第二个函数 `SetOutput` 接受一个输出流参数，并将其设置为当前的logger实例的输出流。它使用 `logger.SetOutput` 方法将设置好的输出流存储在当前的logger实例中，然后返回。

第三个函数 `RegisterLogger` 接受一个logger实例作为参数，将其存储在当前的logger实例中，并返回该实例。这个函数将作为上下文传递给其他函数使用，例如 `SetLogLevel` 和 `SetOutput`，以便正确地设置日志记录功能。


```go
func SetLogLevel(level LogLevel) {
	logger.SetLogLevel(level)
}

func SetOutput(w io.Writer) {
	logger.SetOutput(w)
}

func RegisterLogger(l Logger) {
	logger = l
}

```

# `log/golog/golog.go`

该代码是一个名为 "golog" 的日志库，用于输出彩色、简单且格式化的日志信息。它使用了 F珍珠开源库进行输出，同时使用了 Go 标准库中的 "fmt" 函数进行格式化输出。

具体来说，该代码的作用是提供了一个方便、快速的输出日志信息的方法，使得我们可以轻松地在应用程序中记录和输出日志信息。通过使用 colorful 标识，我们可以设置输出格式为 "绿色" 或 "红色"，并通过 "time" 函数来记录当前时间戳。

此外，该代码还提供了一些有用的功能。例如，可以使用 "io" 包中的 "Erl" 函数来输出错误信息，使用 "path/filepath" 包中的 "PlatForm" 函数来输出文件路径。另外，该代码还提供了一个 "runtime" 包中的 "Sync" 函数来保证多个 golog 实例之间的一致性。


```go
// The colorful and simple logging library
// Copyright (c) 2017 Fadhli Dzil Ikram

package golog

import (
	"fmt"
	"io"
	"os"
	"path/filepath"
	"runtime"
	"sync"
	"sync/atomic"
	"time"

	terminal "golang.org/x/term"

	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/log/golog/colorful"
)

```

该代码定义了一个名为`Logger`的结构体类型，用于表示单个logger。该`Logger`类型实现了`FdWriter`接口，提供了对日志输出和新建文件归档的支持。

具体来说，该代码主要实现了以下功能：

1. 初始化函数`init()`，设置日志输出为标准输出（os.Stdout），并输出调试信息。
2. 定义了一个`Logger`类型，其中包含了对`FdWriter`接口的实现，以及一些用于内部存储的变量。
3. 定义了一个`Logger`结构体类型，其中包含了一个`mu`成员变量，用于实现并发写入，同时包含了一个`color`成员变量，用于设置日志输出的颜色，`out`成员变量，用于输出日志文件，`debug`成员变量，用于控制是否输出调试信息，`timestamp`成员变量，用于记录日志的创建时间，`quiet`成员变量，用于控制是否输出安静模式，`buf`成员变量，用于存储日志缓冲区，`logLevel`成员变量，用于记录当前日志输出级别。
4. 实现了`FdWriter`接口的`Logger`类型，其中包含了一个`io.Writer`类型的`out`成员变量，用于输出日志文件；同时，还包含了一个`Fd`成员变量，用于获取文件描述符，从而实现了文件归档的功能。
5. 通过`registerLogger`函数，将`Logger`实例注册到`log.RegisterLogger`函数中，从而实现对日志输出实现在运行时应用程序的启动过程中。


```go
func init() {
	log.RegisterLogger(New(os.Stdout))
}

// FdWriter interface extends existing io.Writer with file descriptor function
// support
type FdWriter interface {
	io.Writer
	Fd() uintptr
}

// Logger struct define the underlying storage for single logger
type Logger struct {
	mu        sync.RWMutex
	color     bool
	out       io.Writer
	debug     bool
	timestamp bool
	quiet     bool
	buf       colorful.ColorBuffer
	logLevel  int32
}

```

这段代码定义了一个名为`Prefix`的结构体，它包含以下字段：

- `Plain`：字符串数组，定义为`[]byte`类型。
- `Color`：字符串数组，定义为`[]byte`类型。
- `File`：布尔值，表示是否在文件中输出信息。

定义了一系列预定义的`Fatal`, `Error`, `Warn`, `Info`, `Debug`, `Trace`前缀，以及它们的`Color`。这些前缀都有不同的颜色，用于在代码中显示不同的错误、警告和信息。

最后，还定义了一个`FatalPrefix`结构体，它继承自`Prefix`，包含了`Plain`、`Color`和`File`字段，并设置了它们的值为：

- `Plain`：显示为`"[FATAL] "`。
- `Color`：显示为`"[ERROR] "`。
- `File`：输出为`true`。

这样，就可以在程序中使用预定义的错误、警告和信息格式，无需自己定义它们。


```go
// Prefix struct define plain and color byte
type Prefix struct {
	Plain []byte
	Color []byte
	File  bool
}

var (
	// Plain prefix template
	plainFatal = []byte("[FATAL] ")
	plainError = []byte("[ERROR] ")
	plainWarn  = []byte("[WARN]  ")
	plainInfo  = []byte("[INFO]  ")
	plainDebug = []byte("[DEBUG] ")
	plainTrace = []byte("[TRACE] ")

	// FatalPrefix show fatal prefix
	FatalPrefix = Prefix{
		Plain: plainFatal,
		Color: colorful.Red(plainFatal),
		File:  true,
	}

	// ErrorPrefix show error prefix
	ErrorPrefix = Prefix{
		Plain: plainError,
		Color: colorful.Red(plainError),
		File:  true,
	}

	// WarnPrefix show warn prefix
	WarnPrefix = Prefix{
		Plain: plainWarn,
		Color: colorful.Orange(plainWarn),
	}

	// InfoPrefix show info prefix
	InfoPrefix = Prefix{
		Plain: plainInfo,
		Color: colorful.Green(plainInfo),
	}

	// DebugPrefix show info prefix
	DebugPrefix = Prefix{
		Plain: plainDebug,
		Color: colorful.Purple(plainDebug),
		File:  true,
	}

	// TracePrefix show info prefix
	TracePrefix = Prefix{
		Plain: plainTrace,
		Color: colorful.Cyan(plainTrace),
	}
)

```

这段代码定义了一个名为Logger的类，以及其构造函数、SetLogLevel方法和一个新函数New。具体解释如下：

1. New函数返回一个Logger实例，其中color属性指定输出是否为终端颜色，out属性指定写入的文件描述符，timestamp属性指定是否记录时间戳。

2. SetLogLevel方法使用mu锁机制确保同一时间只有一个实例同时访问l.mu.Lock()和l.mu.Unlock()，然后 atomic.StoreInt32(&l.logLevel, int32(level)) 将l.logLevel设置为传入的level。

3. New函数中的color属性使用了终端颜色支持，如果当前进程的终端颜色支持，则会创建一个Logger实例，否则不会创建。

4. SetLogLevel方法会将l.logLevel设置为传入的level，这个level可以是一个log.LogLevel类型的值，如log.LogLevel(log.Debug)、log.LogLevel(log.Warning)、log.LogLevel(log.Error)等等。


```go
// New returns new Logger instance with predefined writer output and
// automatically detect terminal coloring support
func New(out FdWriter) *Logger {
	return &Logger{
		color:     terminal.IsTerminal(int(out.Fd())),
		out:       out,
		timestamp: true,
	}
}

func (l *Logger) SetLogLevel(level log.LogLevel) {
	l.mu.Lock()
	defer l.mu.Unlock()
	atomic.StoreInt32(&l.logLevel, int32(level))
}

```

这段代码定义了两个函数，一个是`func (l *Logger) SetOutput(w io.Writer)`，另一个是`func (l *Logger) WithColor() *Logger`。

这两个函数的作用如下：

1. `SetOutput`函数：

a. 调用自`Logger`类型的`l`实例。

b. 使用`io.Writer`类型的`w`作为输出目标。

c. 先获取`FdWriter`类型的`w`，如果`w`是有效的，说明`l`的输出颜色设置为真，否则设置为假。

d. 设置`l.color`为`true`。

e. 设置`l.out`为`w`。

2. `WithColor`函数：

a. 调用自`Logger`类型的`l`实例。

b. 使用`io.Writer`类型的`w`作为输出目标。

c. 设置`l.color`为`true`。

d. 返回`l`，这样`l`将具有与`SetOutput`函数相同的输出特性，但不会自动应用颜色主题。

请注意，这些函数可能会修改全局日志配置，因此在修改日志配置之前，请仔细阅读代码，理解其可能的影响。


```go
func (l *Logger) SetOutput(w io.Writer) {
	l.mu.Lock()
	defer l.mu.Unlock()
	l.color = false
	if fdw, ok := w.(FdWriter); ok {
		l.color = terminal.IsTerminal(int(fdw.Fd()))
	}
	l.out = w
}

// WithColor explicitly turn on colorful features on the log
func (l *Logger) WithColor() *Logger {
	l.mu.Lock()
	defer l.mu.Unlock()
	l.color = true
	return l
}

```

这两段代码是LOF(Log-Oriented Format)Logger类的两个方法，主要作用是设置Logger的输出风格，以使输出更加友好和易于理解。

`WithoutColor()`方法的作用是关闭Logger的彩色功能，即不再输出 `color` 字段的数据，从而使得输出更加简洁和易于阅读。这个方法使用了多线程安全锁(Mutex)来确保在多线程环境中可以安全地访问该方法。

`WithDebug()`方法的作用是开启Logger的调试输出，即输出 `debug` 字段的数据，以提供更多的调试信息。同样使用了多线程安全锁来确保在多线程环境中可以安全地访问该方法。

这两段代码的实现比较简单，主要通过设置Logger的`color`字段来控制其输出风格的开启和关闭，以及通过设置`debug`字段来开启或关闭调试输出。


```go
// WithoutColor explicitly turn off colorful features on the log
func (l *Logger) WithoutColor() *Logger {
	l.mu.Lock()
	defer l.mu.Unlock()
	l.color = false
	return l
}

// WithDebug turn on debugging output on the log to reveal debug and trace level
func (l *Logger) WithDebug() *Logger {
	l.mu.Lock()
	defer l.mu.Unlock()
	l.debug = true
	return l
}

```

这两位作者都定义了 `Logger` 类型的变量 `l`，并在其中使用了 `mu` 类型的锁来同步输出，同时使用了 `l.debug` 和 `l.mu.Lock()` / `l.mu.Unlock()` 来控制 `l.debug` 的值，最后通过 `l.mu.RLock()` / `l.mu.RUnlock()` 检查当前的 `l.debug` 值是否为 `false`。

具体来说，这两位作者创建了一个 `Logger` 类型，可以使用 `WithoutDebug()` 函数将其设置为不输出调试信息，即不会在输出中打印 `DEBUG` 级别的信息。然后，作者定义了两个函数 `IsDebug()` 和 `WithoutDebug()`。 `IsDebug()` 函数使用 `l.mu.RLock()` 获取当前的 `l.debug` 值，并使用 `l.mu.RUnlock()` 释放锁，最终返回一个布尔值表示当前的 `l.debug` 值。 `WithoutDebug()` 函数使用 `l.mu.Lock()` 获取当前的 `l.debug` 值，并在函数内部将 `l.debug` 的值设置为 `false`，返回原始的 `Logger` 实例。


```go
// WithoutDebug turn off debugging output on the log
func (l *Logger) WithoutDebug() *Logger {
	l.mu.Lock()
	defer l.mu.Unlock()
	l.debug = false
	return l
}

// IsDebug check the state of debugging output
func (l *Logger) IsDebug() bool {
	l.mu.RLock()
	defer l.mu.RUnlock()
	return l.debug
}

```

这段代码定义了两个与Logger接口相同的函数，分别名为WithTimestamp和WithoutTimestamp。这两个函数的作用是，当日志输出时，设置或取消一个名为"timestamp"的设置，以便在日志中记录当前时间戳。

具体来说，当WithTimestamp函数被调用时，会尝试获取一个Logger实例，并尝试获取一个名为"timestamp"的设置。如果当前设置为真，则执行完操作后返回已修改的Logger实例。如果当前设置为假，则返回原始Logger实例。

类似地，当WithoutTimestamp函数被调用时，会尝试获取一个Logger实例，并尝试获取一个名为"timestamp"的设置。如果当前设置为真，则执行完操作后返回原始Logger实例。如果当前设置为假，则返回已修改的Logger实例。

这两个函数都在同一侧加锁和解锁，以保证在同一时刻只有一个实例被创建或修改。


```go
// WithTimestamp turn on timestamp output on the log
func (l *Logger) WithTimestamp() *Logger {
	l.mu.Lock()
	defer l.mu.Unlock()
	l.timestamp = true
	return l
}

// WithoutTimestamp turn off timestamp output on the log
func (l *Logger) WithoutTimestamp() *Logger {
	l.mu.Lock()
	defer l.mu.Unlock()
	l.timestamp = false
	return l
}

```

这是一个使用Go的Logger类，用于记录函数调用过程中的日志输出开关。

具体来说，以下是代码的实现方式：


// Quiet turn off all log output
func (l *Logger) Quiet() *Logger {
	l.mu.Lock()
	defer l.mu.Unlock()
	l.quiet = true
	return l
}

// NoQuiet turn on all log output
func (l *Logger) NoQuiet() *Logger {
	l.mu.Lock()
	defer l.mu.Unlock()
	l.quiet = false
	return l
}


其中，`l.mu.Lock()` 是获取锁，确保在多线程环境下对Logger对象进行操作时，其他线程无法访问并修改Logger对象的状态。

`l.mu.Unlock()` 是释放锁，确保在多线程环境下对Logger对象进行操作时，其他线程可以访问并修改Logger对象的状态。

`l.quiet = true` 将Logger对象的`quiet`字段设置为`true`，这意味着将停止所有日志输出，即不会在日志中记录函数调用过程中的信息。

`l.quiet = false` 将Logger对象的`quiet`字段设置为`false`，这意味着将开启所有日志输出，即会在日志中记录函数调用过程中的信息。

`l` 返回一个指向Logger对象的引用，以便其他函数可以调用`Quiet`和`NoQuiet`方法。


```go
// Quiet turn off all log output
func (l *Logger) Quiet() *Logger {
	l.mu.Lock()
	defer l.mu.Unlock()
	l.quiet = true
	return l
}

// NoQuiet turn on all log output
func (l *Logger) NoQuiet() *Logger {
	l.mu.Lock()
	defer l.mu.Unlock()
	l.quiet = false
	return l
}

```

This appears to be a JavaScript function that generates human-readable HTTP headers based on the current date, time, and potentially other parameters. The function takes in the current date and time as arguments and returns the generated headers as a ByteArray.

The headers generated by the function are written to the ByteArray output by the `http.out` module, which is part of the Node.js HTTP server module. The headers are written in the order that they should appear in a standard HTTP request.

The function also generates a reset color header if the `l.color` parameter is `true`. This can be used to reset the color scheme used by the headers if needed.

Note that the generated headers are written in UTF-8 encoding to ensure that they are compatible with the majority of web servers.


```go
// IsQuiet check for quiet state
func (l *Logger) IsQuiet() bool {
	l.mu.RLock()
	defer l.mu.RUnlock()
	return l.quiet
}

// Output print the actual value
func (l *Logger) Output(depth int, prefix Prefix, data string) error {
	// Check if quiet is requested, and try to return no error and be quiet
	if l.IsQuiet() {
		return nil
	}
	// Get current time
	now := time.Now()
	// Temporary storage for file and line tracing
	var file string
	var line int
	var fn string
	// Check if the specified prefix needs to be included with file logging
	if prefix.File {
		var ok bool
		var pc uintptr

		// Get the caller filename and line
		if pc, file, line, ok = runtime.Caller(depth + 2); !ok {
			file = "<unknown file>"
			fn = "<unknown function>"
			line = 0
		} else {
			file = filepath.Base(file)
			fn = runtime.FuncForPC(pc).Name()
		}
	}
	// Acquire exclusive access to the shared buffer
	l.mu.Lock()
	defer l.mu.Unlock()
	// Reset buffer so it start from the beginning
	l.buf.Reset()
	// Write prefix to the buffer
	if l.color {
		l.buf.Append(prefix.Color)
	} else {
		l.buf.Append(prefix.Plain)
	}
	// Check if the log require timestamping
	if l.timestamp {
		// Print timestamp color if color enabled
		if l.color {
			l.buf.Blue()
		}
		// Print date and time
		year, month, day := now.Date()
		l.buf.AppendInt(year, 4)
		l.buf.AppendByte('/')
		l.buf.AppendInt(int(month), 2)
		l.buf.AppendByte('/')
		l.buf.AppendInt(day, 2)
		l.buf.AppendByte(' ')
		hour, min, sec := now.Clock()
		l.buf.AppendInt(hour, 2)
		l.buf.AppendByte(':')
		l.buf.AppendInt(min, 2)
		l.buf.AppendByte(':')
		l.buf.AppendInt(sec, 2)
		l.buf.AppendByte(' ')
		// Print reset color if color enabled
		if l.color {
			l.buf.Off()
		}
	}
	// Add caller filename and line if enabled
	if prefix.File {
		// Print color start if enabled
		if l.color {
			l.buf.Orange()
		}
		// Print filename and line
		l.buf.Append([]byte(fn))
		l.buf.AppendByte(':')
		l.buf.Append([]byte(file))
		l.buf.AppendByte(':')
		l.buf.AppendInt(line, 0)
		l.buf.AppendByte(' ')
		// Print color stop
		if l.color {
			l.buf.Off()
		}
	}
	// Print the actual string data from caller
	l.buf.Append([]byte(data))
	if len(data) == 0 || data[len(data)-1] != '\n' {
		l.buf.AppendByte('\n')
	}
	// Flush buffer to output
	_, err := l.out.Write(l.buf.Buffer)
	return err
}

```

这两函数函数是 `Logger` 类的成员函数，作用是输出 Fatal 错误消息并退出应用程序，其中第一个函数 `Fatal` 输出带有 `FatalPrefix` 的 Fatal 错误消息，第二个函数 `Fatalf` 输出带有 `FatalPrefix` 的 Fatal 错误消息。

在这里，我们创建了一个名为 `l` 的 `Logger` 类型的变量，并定义了这两个函数。第一个函数 `Fatal` 接收一个或多个 `interface{}` 类型的参数 `v`，在条件 `atomic.LoadInt32(&l.logLevel) <= 4` 满足时执行以下操作：

1. 获取 `l.logLevel` 的原子值，并将其加载为 `int32` 类型。
2. 如果原子值小于等于 4，就执行以下操作：
  1. 获取 `v` 切片中的第一个元素，并将其输出。
  2. 将 `FatalPrefix` 设置为 `format`。
  3. 尝试使用 `fmt.Sprintln` 格式化 `v` 切片中的元素，如果格式化失败，则输出 Fatal 错误消息并退出。
3. 执行 `os.Exit` 并返回状态码 1。

第二个函数 `Fatalf` 同样接收一个或多个 `interface{}` 类型的参数 `v`，在条件 `atomic.LoadInt32(&l.logLevel) <= 4` 满足时执行以下操作：

1. 获取 `l.logLevel` 的原子值，并将其加载为 `int32` 类型。
2. 如果原子值小于等于 4，就执行以下操作：
  1. 获取 `v` 切片中的第一个元素，并将其输出。
  2. 将 `FatalPrefix` 设置为 `format`。
  3. 尝试使用 `fmt.Sprintf` 格式化 `v` 切片中的元素，如果格式化失败，则输出 Fatal 错误消息并退出。
4. 执行 `os.Exit` 并返回状态码 1。


```go
// Fatal print fatal message to output and quit the application with status 1
func (l *Logger) Fatal(v ...interface{}) {
	if atomic.LoadInt32(&l.logLevel) <= 4 {
		l.Output(1, FatalPrefix, fmt.Sprintln(v...))
	}
	os.Exit(1)
}

// Fatalf print formatted fatal message to output and quit the application
// with status 1
func (l *Logger) Fatalf(format string, v ...interface{}) {
	if atomic.LoadInt32(&l.logLevel) <= 4 {
		l.Output(1, FatalPrefix, fmt.Sprintf(format, v...))
	}
	os.Exit(1)
}

```

这两函数主要作用于在调试信息中输出错误、警告信息。

第一个函数 `Error` 接收一个或多个 `interface{}` 类型的参数 `v...`, 并检查原子计数器 `l.logLevel` 的值是否小于或等于 3。如果是，则使用 `fmt.Sprintln` 函数输出整数 `1` 和 `v...` 作为参数的错误消息。这个函数的输出信息将只传递给调试信息的消费者，如开发者的控制台或系统日志。

第二个函数 `Errorf` 接收一个格式化的错误消息 `format` 和一个或多个 `interface{}` 类型的参数 `v...`。如果原子计数器 `l.logLevel` 的值小于或等于 3，则使用 `fmt.Sprintf` 函数输出整数 `1` 和 `v...` 作为参数的错误消息，格式化后的错误消息将按照 `format` 中的字符串模板显示。这个函数的输出信息将传递给调试信息的消费者和开发者的控制台，例如开发者的调试工具和日志输出。


```go
// Error print error message to output
func (l *Logger) Error(v ...interface{}) {
	if atomic.LoadInt32(&l.logLevel) <= 3 {
		l.Output(1, ErrorPrefix, fmt.Sprintln(v...))
	}
}

// Errorf print formatted error message to output
func (l *Logger) Errorf(format string, v ...interface{}) {
	if atomic.LoadInt32(&l.logLevel) <= 3 {
		l.Output(1, ErrorPrefix, fmt.Sprintf(format, v...))
	}
}

// Warn print warning message to output
```

这是一个 Go 语言中的包 level-log 中的三个函数，它们用于向 logger 实例输出不同级别的警告或信息。

在这些函数中，首先定义了三个整数类型的变量 l，分别命名为 logLevel、warnLevel 和 infoLevel，这些变量用于存储日志级别的值，初始值均为 0。然后定义了三个整数类型的函数 Warn、Warnf 和 Info，它们分别接收一个或多个整数类型的参数 v，并根据 logLevel 变量的值来决定是否输出警告或信息，然后使用Output函数来输出相应的信息。

在这些函数中，使用了 Go 语言中的原子类型来保证同一时间只有一个进程能够访问 l.logLevel 变量，即使在并发调用的情况下，也不会存在竞争条件。另外，为了避免潜在的死锁，这些函数中的原子操作都使用了原子类型。


```go
func (l *Logger) Warn(v ...interface{}) {
	if atomic.LoadInt32(&l.logLevel) <= 2 {
		l.Output(1, WarnPrefix, fmt.Sprintln(v...))
	}
}

// Warnf print formatted warning message to output
func (l *Logger) Warnf(format string, v ...interface{}) {
	if atomic.LoadInt32(&l.logLevel) <= 2 {
		l.Output(1, WarnPrefix, fmt.Sprintf(format, v...))
	}
}

// Info print informational message to output
func (l *Logger) Info(v ...interface{}) {
	if atomic.LoadInt32(&l.logLevel) <= 1 {
		l.Output(1, InfoPrefix, fmt.Sprintln(v...))
	}
}

```

这两函数一起作用于信息输出和调试输出。

Info输出函数 `Infof` 的作用是打印信息级别的消息。它接收一个格式字符串和多个参数。

当原子 `logLevel` 变量加载的值小于或等于 1 时，它将调用 `l.Output` 并传入一个 `InfoPrefix`、一个格式字符串和一个或多个参数 `v`。`InfoPrefix` 是输出信息的预兆，而 `fmt.Sprintf` 将格式化 `v` 并将结果填充到 `InfoPrefix` 和 `format` 之间。

当原子 `logLevel` 变量加载的值大于 1 时，这个函数不会被调用。

Debug 输出函数 `Debug` 的作用是打印调试级别的消息。它接收一个或多个参数 `v`。当原子 `logLevel` 变量加载的值小于或等于 0 时，它将调用 `l.Output` 并传入一个 `DebugPrefix` 和多个参数 `v`。`DebugPrefix` 是调试信息的预兆，而 `fmt.Sprintln` 将格式化 `v` 并将结果打印到 `DebugPrefix` 和 `format` 之间。

当原子 `logLevel` 变量加载的值大于 1 时，这个函数不会被调用。


```go
// Infof print formatted informational message to output
func (l *Logger) Infof(format string, v ...interface{}) {
	if atomic.LoadInt32(&l.logLevel) <= 1 {
		l.Output(1, InfoPrefix, fmt.Sprintf(format, v...))
	}
}

// Debug print debug message to output if debug output enabled
func (l *Logger) Debug(v ...interface{}) {
	if atomic.LoadInt32(&l.logLevel) == 0 {
		l.Output(1, DebugPrefix, fmt.Sprintln(v...))
	}
}

// Debugf print formatted debug message to output if debug output enabled
```

这段代码定义了三个名为 `func`, `Trace`, 和 `Tracef` 的函数，它们都接受一个名为 `Logger` 的指针参数。

`func` 函数接收一个 `Logger` 类型的参数 `l` 和一个字符串格式化字符串 `format` 和一个或多个 `interface{}` 类型的参数 `v...`。它首先检查本地是否有指定的日志级别，如果没有，就执行 `l.Output` 函数，其中 `1` 表示输出为 `10` 级别的信息，`DebugPrefix` 和 `fmt.Sprintf` 函数用于输出调试前缀和格式化后的信息。如果本地有指定的日志级别，就先将指定的日志级别设置为本地日志级别的值，然后调用 `l.Output` 函数。

`Trace` 函数与 `func` 函数的作用类似，但它的参数中传递的是一个或多个 `interface{}` 类型的参数 `v...`。`Trace` 函数的输出风格更加友好，它将输出类似于 JUnit 测试中使用的 `@Test` 方法的 `TRACE` 方法的输出，同时还能够提供更多的信息，例如输出前缀、格式化字符串和参数列表。

`Tracef` 函数与 `func` 和 `Trace` 函数的作用类似，但它的参数中传递的是一个字符串格式化字符串 `format` 和一个或多个 `interface{}` 类型的参数 `v...`。`Tracef` 函数的输出风格更加类似于格式化字符串，它将输出类似于 `printf` 函数的输出，同时还能够提供更多的信息，例如输出前缀、格式化字符串和参数列表。


```go
func (l *Logger) Debugf(format string, v ...interface{}) {
	if atomic.LoadInt32(&l.logLevel) == 0 {
		l.Output(1, DebugPrefix, fmt.Sprintf(format, v...))
	}
}

// Trace print trace message to output if debug output enabled
func (l *Logger) Trace(v ...interface{}) {
	if atomic.LoadInt32(&l.logLevel) == 0 {
		l.Output(1, TracePrefix, fmt.Sprintln(v...))
	}
}

// Tracef print formatted trace message to output if debug output enabled
func (l *Logger) Tracef(format string, v ...interface{}) {
	if atomic.LoadInt32(&l.logLevel) == 0 {
		l.Output(1, TracePrefix, fmt.Sprintf(format, v...))
	}
}

```