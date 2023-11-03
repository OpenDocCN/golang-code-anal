# v2ray-core源码解析 71

# `transport/internet/tcp/sockopt_freebsd.go`

这段代码是一个 Go 语言编写的文本，定义了一个名为 "tcp" 的包。

首先，这个包使用了 FreeBSD 构建系统，会在编译时生成一些可执行文件。接着，它定义了一个名为 "+build" 的指令，这个指令可能是用于构建其他 Go 语言项目。

然后，这个包导入了一些来自 "v2ray.com/core/common/net" 和 "v2ray.com/core/transport/internet" 的包。

接着，这个包中定义了一个名为 "GetOriginalDestination" 的函数，这个函数接受一个互联网连接对象（可能来自 "v2ray.com/core/transport/internet" 包），并返回原始目的地的 IP 地址和端口号（如果计算正确）。

最后，这个包定义了一个名为 "tcp" 的包，可能用于定义其他 Go 语言项目所需的功能。


```go
// +build freebsd
// +build !confonly

package tcp

import (
	"v2ray.com/core/common/net"
	"v2ray.com/core/transport/internet"
)

// GetOriginalDestination from tcp conn
func GetOriginalDestination(conn internet.Connection) (net.Destination, error) {
	la := conn.LocalAddr()
	ra := conn.RemoteAddr()
	ip, port, err := internet.OriginalDst(la, ra)
	if err != nil {
		return net.Destination{}, newError("failed to get destination").Base(err)
	}
	dest := net.TCPDestination(net.IPAddress(ip), net.Port(port))
	if !dest.IsValid() {
		return net.Destination{}, newError("failed to parse destination.")
	}
	return dest, nil
}

```

# `transport/internet/tcp/sockopt_linux.go`

这段代码是一个 Go 语言编写的工具函数，它的作用是获取 TCP 连接的原始目的服务器。

首先，该函数使用 `+build` 标志编译为 linux 平台。然后，使用 `confonly` 标志编译为只读模式，这样编译出的代码只能被读取不能被修改。

接下来，该函数导入了一些必要的库，包括 `syscall` 和 `v2ray.com/core/transport/internet`，用于获取原始目的服务器信息和发送控制信息。

函数内部，首先通过 `syscall.Conn` 获取到当前连接的原始数据链路层（即 TCP）连接。如果获取失败，则返回一个错误并输出错误信息。如果获取成功，则执行以下操作：

1. 获取原始目的服务器 IP 地址和端口号。这通过调用 `syscall.GetsockoptIPv6Mreq` 函数获取，该函数将获取一个 IPv6 数据链路层的原始数据链路层地址。
2. 通过调用 `syscall.Getsockopt` 函数获取原始目的服务器端口号。
3. 创建一个网络接口，并将获取到的 IP 地址和端口号映射到该接口上，然后使用该接口创建一个 TCP 连接。
4. 将创建好的 TCP 连接发送控制信息，设置其原始目的服务器。
5. 如果过程中出现错误，则返回一个错误并输出错误信息。

最后，如果原始目的服务器设置成功，则返回原始目的服务器，否则返回一个错误并输出错误信息。


```go
// +build linux
// +build !confonly

package tcp

import (
	"syscall"

	"v2ray.com/core/common/net"
	"v2ray.com/core/transport/internet"
)

const SO_ORIGINAL_DST = 80

func GetOriginalDestination(conn internet.Connection) (net.Destination, error) {
	sysrawconn, f := conn.(syscall.Conn)
	if !f {
		return net.Destination{}, newError("unable to get syscall.Conn")
	}
	rawConn, err := sysrawconn.SyscallConn()
	if err != nil {
		return net.Destination{}, newError("failed to get sys fd").Base(err)
	}
	var dest net.Destination
	err = rawConn.Control(func(fd uintptr) {
		addr, err := syscall.GetsockoptIPv6Mreq(int(fd), syscall.IPPROTO_IP, SO_ORIGINAL_DST)
		if err != nil {
			newError("failed to call getsockopt").Base(err).WriteToLog()
			return
		}
		ip := net.IPAddress(addr.Multiaddr[4:8])
		port := uint16(addr.Multiaddr[2])<<8 + uint16(addr.Multiaddr[3])
		dest = net.TCPDestination(ip, net.Port(port))
	})
	if err != nil {
		return net.Destination{}, newError("failed to control connection").Base(err)
	}
	if !dest.IsValid() {
		return net.Destination{}, newError("failed to call getsockopt")
	}
	return dest, nil
}

```

# `transport/internet/tcp/sockopt_linux_test.go`

这段代码是一个 Go 语言编写的测试框架，用于测试 TCP 协议的客户端和服务器端。它包括一个名为 "tcp_test" 的包，以及一个名为 "test_tcp" 的测试文件。

具体来说，这段代码的作用是以下几个方面：

1. 定义了一个名为 "tcp_test" 的包，这个包中定义了一些 "testing.T" 类型的测试函数，用于测试 TCP 协议的客户端和服务器端。
2. 引入了 "v2ray.com/core/common"、"v2ray.com/core/testing/servers/tcp" 和 "v2ray.com/core/transport/internet" 包，这些包提供了用于测试 TCP 协议的客户端和服务器端的函数和接口。
3. 通过 "strings" 包中的 "format" 函数，将 "linux" 编译为 "linux" 操作系统对应的名称。这个字符串就是 "+build linux" 的意思，用于告诉编译器在编译时使用哪个操作系统编译模式。
4. 在 "test_tcp" 测试文件中，可能进行了对 TCP 协议的客户端和服务器端的测试。由于没有具体测试的内容，所以无法给出这些测试的具体实现。


```go
// +build linux

package tcp_test

import (
	"context"
	"strings"
	"testing"

	"v2ray.com/core/common"
	"v2ray.com/core/testing/servers/tcp"
	"v2ray.com/core/transport/internet"
	. "v2ray.com/core/transport/internet/tcp"
)

```

这段代码在测试中使用tcp.Server()启动了一个tcp服务器，并使用Dial()方法尝试连接到该服务器，从而建立了一个tcp对。然后，它使用GetOriginalDestination()函数来获取服务器启动时设置的原始目标地址。如果获取到的原始目标地址与服务器启动时设置的目标地址不匹配，或者在尝试获取服务器选项时出现错误，那么代码将输出"unexpected state"并退出测试。


```go
func TestGetOriginalDestination(t *testing.T) {
	tcpServer := tcp.Server{}
	dest, err := tcpServer.Start()
	common.Must(err)
	defer tcpServer.Close()

	config, err := internet.ToMemoryStreamConfig(nil)
	common.Must(err)
	conn, err := Dial(context.Background(), dest, config)
	common.Must(err)
	defer conn.Close()

	originalDest, err := GetOriginalDestination(conn)
	if !(dest == originalDest || strings.Contains(err.Error(), "failed to call getsockopt")) {
		t.Error("unexpected state")
	}
}

```

# `transport/internet/tcp/sockopt_other.go`

这段代码是一个用于TCP协议的网络工具类，通过以下几行代码实现了TCP连接的构建和解析。

1. `// +build !linux,!freebsd`
这一行代码是编译构建选项，表示编译为不依赖于具体操作系统的工具链。这里使用了`!linux`和`!freebsd`，分别表示不支持Linux和FreeBSD操作系统。

2. `// +build !confonly`
这一行代码也是编译构建选项，表示编译为仅输出引用了被编译源代码的函数和变量。

3. `package tcp`
这一行代码定义了输出包名称，表明这个工具类属于"tcp"包。

4. `import ( "v2ray.com/core/common/net" "v2ray.com/core/transport/internet" )`
这一行代码引入了两个标准库：`v2ray.com/core/common/net`和`v2ray.com/core/transport/internet`。

5. `func GetOriginalDestination(conn internet.Connection) (net.Destination, error)`
这一行代码定义了一个名为`GetOriginalDestination`的函数，接收一个`internet.Connection`类型的连接对象，并返回原始目标IP地址和错误。

6. `return net.Destination{}, nil`
这一行代码返回一个`net.Destination`类型，其中`net.Destination`表示目标IP地址，这里使用了`!linux`构建的`internet.Connection`不依赖于操作系统，因此IP地址不会被限制。同时，这一行代码还返回一个`nil`类型的错误，表示操作成功。


```go
// +build !linux,!freebsd
// +build !confonly

package tcp

import (
	"v2ray.com/core/common/net"
	"v2ray.com/core/transport/internet"
)

func GetOriginalDestination(conn internet.Connection) (net.Destination, error) {
	return net.Destination{}, nil
}

```

# `transport/internet/tcp/tcp.go`

这段代码定义了一个名为 "tcp" 的包，其中包含了一些通用的 TCP 函数和错误代码。

第一行 "go:generate go run v2ray.com/core/common/errors/errorgen" 是 Go 语言的编译器命令，用于生成一个名为 "v2ray.com/core/common/errors/errorgen" 的文件，该文件包含了GO文档中定义的所有GO类型。

下面是该代码中包含的一些通用的 TCP 函数和错误代码：


package tcp

import (
	"net"
	"fmt"
)

const (
	连接池 DefaultConnections = 10
	心跳 Interval = 10 * time.Second
	Max SendChunkSize = 1024
	MaxAmbientClocktime = 30 * time.Second
)

type upgradeMap struct {
	M map[string]int
}

func (u *upgradeMap) Add(v int) {
	u.M[v] = 1
}

func (u *upgradeMap) Get(v int) int {
	return u.M[v]
}

func (u *upgradeMap) Clear(v int) {
	u.M[v] = 0
}

func (u *upgradeMap) Update(ch chan<-[]byte, w net.Conn) {
	// 更新
}

func (u *upgradeMap) Retrieve(path string, read bool) ([]byte, error) {
	// 读取
}


这些函数和错误代码都可以在GO应用程序中被用来处理TCP连接、心跳、发送数据、设置连接池等任务。通过定义这些通用的函数和错误代码，可以简化TCP编程的难度，同时减少出错的可能性。


```go
package tcp

//go:generate go run v2ray.com/core/common/errors/errorgen

```

# `transport/internet/tls/config.go`

这段代码是一个Go语言编写的命令行工具，它用于将一个TLS证书打包为二进制格式，并将其保存到指定的目录中。

首先，它使用`build`构建工具编译源代码，这将生成一个名为`tls_cert_packaged`的可执行文件。接下来，它使用`confonly`选项来创建一个只读的文件，这样我们就可以在保存二进制文件的同时，确保文件不会被修改。

接下来，它导入了自定义的`tls`包，这个包可能是从其他依赖包中导入的。然后，它使用`crypto/tls`包中的`TLS`类型来与TLS证书相关操作，使用`crypto/x509`包中的`X509`类型来加载证书的私钥。

接着，它使用Go语言的`strings`函数截取证书颁发者名称，并将其存储在一个名为`cert_ca_name`的变量中。然后，它构造一个证书包装函数，该函数将证书、私钥和颁发者名称作为参数，返回一个包含这些参数的接口。

接下来，它使用Go语言的`sync`包中的`Chan`类型来等待证书包装函数返回的值，以确保在所有证书都被包装完成后，包装程序可以继续执行。最后，它使用Go语言的`time`包中的`Duration`类型来在包装完成后等待一段时间，以确保所有证书都被处理。

综上所述，这段代码的作用是将一个TLS证书打包为二进制格式，并将其保存到指定的目录中。


```go
// +build !confonly

package tls

import (
	"crypto/tls"
	"crypto/x509"
	"strings"
	"sync"
	"time"

	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol/tls/cert"
	"v2ray.com/core/transport/internet"
)

```

这段代码定义了一个名为“exp8357”的变量，并将其初始化为“experiment:8357”。接下来，它创建了一个名为“globalSessionCache”的TCP连接缓存实例，并将其设置为128。

然后，它定义了一个名为“ParseCertificate”的函数，该函数接收一个名为“c”的证书对象作为参数，并将其转换为Certificate类型。在函数内部，它使用了“c.ToPEM()”方法将证书对象转换为PEM格式的字节切片，并从其中提取了证书和私钥。最后，它返回了一个Certificate类型的变量，将证书的PEM字节片和私钥字节片都存储在同一个内存中。


```go
var (
	globalSessionCache = tls.NewLRUClientSessionCache(128)
)

const exp8357 = "experiment:8357"

// ParseCertificate converts a cert.Certificate to Certificate.
func ParseCertificate(c *cert.Certificate) *Certificate {
	certPEM, keyPEM := c.ToPEM()
	return &Certificate{
		Certificate: certPEM,
		Key:         keyPEM,
	}
}

```

这两段代码是针对一个名为“Config”的结构体进行操作的函数。

第一段代码名为“loadSelfCertPool”，它接收一个名为“c”的指针参数，然后返回一个名为“root”和一个名为“error”的变量。函数的作用是加载自签名证书池并将其存储在“root”中，然后在循环中从给定的“c”结构体中加载证书并将其存储在“root”中。如果函数在加载证书时遇到任何错误，它将返回一个错误并输出警告信息。

第二段代码名为“BuildCertificates”，它接收一个名为“c”的指针参数，然后返回一个包含由“BuildCertificates”函数生成的TLS证书的数组。函数的作用是生成TLS证书并将其存储在数组中。它遍历给定的“c”结构体，检查每个证书的用途是否为“Certificate\_ENCIPHERMENT”。如果是，那么函数将生成一个TLS证书并将其添加到数组中。最后，函数返回生成的TLS证书数组。


```go
func (c *Config) loadSelfCertPool() (*x509.CertPool, error) {
	root := x509.NewCertPool()
	for _, cert := range c.Certificate {
		if !root.AppendCertsFromPEM(cert.Certificate) {
			return nil, newError("failed to append cert").AtWarning()
		}
	}
	return root, nil
}

// BuildCertificates builds a list of TLS certificates from proto definition.
func (c *Config) BuildCertificates() []tls.Certificate {
	certs := make([]tls.Certificate, 0, len(c.Certificate))
	for _, entry := range c.Certificate {
		if entry.Usage != Certificate_ENCIPHERMENT {
			continue
		}
		keyPair, err := tls.X509KeyPair(entry.Certificate, entry.Key)
		if err != nil {
			newError("ignoring invalid X509 key pair").Base(err).AtWarning().WriteToLog()
			continue
		}
		certs = append(certs, keyPair)
	}
	return certs
}

```

这段代码定义了两个函数：isCertificateExpired 和 issueCertificate。

isCertificateExpired 的作用是检查给定的 tls.Certificate 是否过期。具体来说，它首先检查给定的证书是否为空（如果证书为空，那么它肯定还没有过期），然后检查给定的证书是否有效（即检查证书是否过期或者是否被撤销）。

issueCertificate 的作用是颁发一个新的证书。它接收一个 Certificate 对象（由 tls.Certificate 类型表示），一个域名和一个证书颁发者（通常是 CA）。函数首先尝试从给定的 Certificate 对象中解析出证书，如果失败则返回一个错误。然后，函数使用证书颁发者证书颁发新的证书。如果新的证书还没有过期，函数将新证书返回。


```go
func isCertificateExpired(c *tls.Certificate) bool {
	if c.Leaf == nil && len(c.Certificate) > 0 {
		if pc, err := x509.ParseCertificate(c.Certificate[0]); err == nil {
			c.Leaf = pc
		}
	}

	// If leaf is not there, the certificate is probably not used yet. We trust user to provide a valid certificate.
	return c.Leaf != nil && c.Leaf.NotAfter.Before(time.Now().Add(-time.Minute))
}

func issueCertificate(rawCA *Certificate, domain string) (*tls.Certificate, error) {
	parent, err := cert.ParseCertificate(rawCA.Certificate, rawCA.Key)
	if err != nil {
		return nil, newError("failed to parse raw certificate").Base(err)
	}
	newCert, err := cert.Generate(parent, cert.CommonName(domain), cert.DNSNames(domain))
	if err != nil {
		return nil, newError("failed to generate new certificate for ", domain).Base(err)
	}
	newCertPEM, newKeyPEM := newCert.ToPEM()
	cert, err := tls.X509KeyPair(newCertPEM, newKeyPEM)
	return &cert, err
}

```

This is a function called `createCertificate` that creates a certificate for a domain by using a certificate authority (CA) if one is available, or generating a new certificate if the request is made within an hour of the start of the day. The certificate is then added to the `access.Certificates` slice, which is a slice of certificates that are stored in memory for high availability purposes. If a certificate becomes expired, it is added to a `certExpired` variable, and if no new certificate is issued within the next hour, the function returns the expired certificate.


```go
func (c *Config) getCustomCA() []*Certificate {
	certs := make([]*Certificate, 0, len(c.Certificate))
	for _, certificate := range c.Certificate {
		if certificate.Usage == Certificate_AUTHORITY_ISSUE {
			certs = append(certs, certificate)
		}
	}
	return certs
}

func getGetCertificateFunc(c *tls.Config, ca []*Certificate) func(hello *tls.ClientHelloInfo) (*tls.Certificate, error) {
	var access sync.RWMutex

	return func(hello *tls.ClientHelloInfo) (*tls.Certificate, error) {
		domain := hello.ServerName
		certExpired := false

		access.RLock()
		certificate, found := c.NameToCertificate[domain]
		access.RUnlock()

		if found {
			if !isCertificateExpired(certificate) {
				return certificate, nil
			}
			certExpired = true
		}

		if certExpired {
			newCerts := make([]tls.Certificate, 0, len(c.Certificates))

			access.Lock()
			for _, certificate := range c.Certificates {
				if !isCertificateExpired(&certificate) {
					newCerts = append(newCerts, certificate)
				}
			}

			c.Certificates = newCerts
			access.Unlock()
		}

		var issuedCertificate *tls.Certificate

		// Create a new certificate from existing CA if possible
		for _, rawCert := range ca {
			if rawCert.Usage == Certificate_AUTHORITY_ISSUE {
				newCert, err := issueCertificate(rawCert, domain)
				if err != nil {
					newError("failed to issue new certificate for ", domain).Base(err).WriteToLog()
					continue
				}

				access.Lock()
				c.Certificates = append(c.Certificates, *newCert)
				issuedCertificate = &c.Certificates[len(c.Certificates)-1]
				access.Unlock()
				break
			}
		}

		if issuedCertificate == nil {
			return nil, newError("failed to create a new certificate for ", domain)
		}

		access.Lock()
		c.BuildNameToCertificate()
		access.Unlock()

		return issuedCertificate, nil
	}
}

```

这段代码是一个 Go 语言编写的函数，接收一个名为 Config 的 Config 结构体参数。

func (c *Config) IsExperiment8357() bool {
	return strings.HasPrefix(c.ServerName, "exp8357")
}

func (c *Config) parseServerName() string {
	if c.IsExperiment8357() {
		return c.ServerName[len("exp8357"):]
	}

	return c.ServerName
}

// GetTLSConfig converts this Config into tls.Config.
func (c *Config) GetTLSConfig(opts ...Option) *tls.Config {
	root, err := c.getCertPool()
	if err != nil {
		newError("failed to load system root certificate").AtError().Base(err).WriteToLog()
	}

	config := &tls.Config{
		ClientSessionCache:     globalSessionCache,
		RootCAs:                root,
		InsecureSkipVerify:     c.AllowInsecure,
		NextProtos:             c.NextProtocol,
		SessionTicketsDisabled: c.DisableSessionResumption,
	}
	if c == nil {
		return config
	}

	for _, opt := range opts {
		opt(config)
	}

	config.Certificates = c.BuildCertificates()
	config.BuildNameToCertificate()

	caCerts := c.getCustomCA()
	if len(caCerts) > 0 {
		config.GetCertificate = getGetCertificateFunc(config, caCerts)
	}

	if sn := c.parseServerName(); len(sn) > 0 {
		config.ServerName = sn
	}

	if len(config.NextProtos) == 0 {
		config.NextProtos = []string{"h2", "http/1.1"}
	}

	return config
}

func (c *Config) getCertPool() (caCerts ...[]byte) error {
	// load root certificate
	tr, err := c.getCertificate([]byte("system root certificate"))
	if err != nil {
		return err
	}

	// get server certificate(s)
	sc, err := c.getServerCertificates(tr)
	if err != nil {
		return err
	}

	// get server name
	sn := c.parseServerName()
	if err := validateServerCerts(sc, sn); err != nil {
		return err
	}

	caCerts = append(caCerts, sc...)

	return nil
}

func (c *Config) getCustomCA() ([]byte, error) {
	// get system root certificate
	tr, err := c.getCertificate([]byte("system root certificate"))
	if err != nil {
		return nil, err
	}

	// get server issue certificate
	si, err := c.getIssuedCertificate(tr)
	if err != nil {
		return nil, err
	}

	return si.Certificate, nil
}

func (c *Config) validateServerCerts(sc []byte, sn string) error {
	// validate server identity
	for i, cert := range sc {
		sa, ok := validateServerCertificate(cert, sn)
		if !ok {
			return err("failed to validate server identity")
		}

		if !sa.Valid {
			return err("server identity is not valid")
		}

		sa.Certificate = cert
	}

	return nil
}

func (c *Config) getIssuedCertificate(cert []byte) (tls.Certificate, error) {
	// get server issued certificate
	return &tls.Certificate{
		Certificate: cert,
		Expires:    c.GetIssuedCertificateExpiration(),
	}, nil
}

func (c *Config) GetIssuedCertificateExpiration() (expiration time.Time, err := nil) {
	// get server issued certificate expiration
	tr, err := c.getCertificate([]byte("system root certificate"))
	if err != nil {
		return nil, err
	}

	return time.Now().Unix(int(tr.Certificate.Certificate.X509().NotAfter))+1, nil
}

func (c *Config) getServerCertificates(tr []byte) ([]tls.Certificate, error) {
	// get server certificates
	sc := []tls.Certificate{}
	for i, cert := range tr {
		sa, ok := validateServerCertificate(cert, "")
		if !ok {
			return nil, err("failed to validate server identity")
		}

		if !sa.Valid {
			return nil, err("server identity is not valid")
		}

		sa.Certificate = cert
		sc = append(sc, sa)
	}

	return sc, nil
}

func (c *Config) validateServerCertificate(cert []byte, serverName string) (tls.Certificate, error) {
	// validate server identity
	tr, err := c.getCertificate(cert)
	if err != nil {
		return nil, err
	}

	var tlsCert tls.Certificate
	tlsCert.Certificate = cert
	tlsCert.ServerName = serverName

	return tlsCert, nil
}

func (c *Config) getCertificate(cert []byte) (caCerts ...[]byte) error {
	// load root certificate
	tr, err := c.getCertificate(cert...)
	if err != nil {
		return err
	}

	// get server identity
	si, err := c.getIssuedCertificate(tr)
	if err != nil {
		return err
	}

	// get server certificate(s)
	sc, err := c.getServerCertificates(tr)
	if err != nil {
		return err
	}

	caCerts = append(caCerts, tr...)

	return nil



```go
func (c *Config) IsExperiment8357() bool {
	return strings.HasPrefix(c.ServerName, exp8357)
}

func (c *Config) parseServerName() string {
	if c.IsExperiment8357() {
		return c.ServerName[len(exp8357):]
	}

	return c.ServerName
}

// GetTLSConfig converts this Config into tls.Config.
func (c *Config) GetTLSConfig(opts ...Option) *tls.Config {
	root, err := c.getCertPool()
	if err != nil {
		newError("failed to load system root certificate").AtError().Base(err).WriteToLog()
	}

	config := &tls.Config{
		ClientSessionCache:     globalSessionCache,
		RootCAs:                root,
		InsecureSkipVerify:     c.AllowInsecure,
		NextProtos:             c.NextProtocol,
		SessionTicketsDisabled: c.DisableSessionResumption,
	}
	if c == nil {
		return config
	}

	for _, opt := range opts {
		opt(config)
	}

	config.Certificates = c.BuildCertificates()
	config.BuildNameToCertificate()

	caCerts := c.getCustomCA()
	if len(caCerts) > 0 {
		config.GetCertificate = getGetCertificateFunc(config, caCerts)
	}

	if sn := c.parseServerName(); len(sn) > 0 {
		config.ServerName = sn
	}

	if len(config.NextProtos) == 0 {
		config.NextProtos = []string{"h2", "http/1.1"}
	}

	return config
}

```

这段代码定义了两个函数，分别名为`WithOption`和`WithTLS`，用于设置TLS配置中的选项。

`WithOption`函数接受一个函数指针`Option`作为参数，这个函数指针可以用来设置TLS配置中的选项。具体来说，这个函数指针被传递给`*tls.Config`类型的变量`config`，然后通过不断地使用`*`运算符获取该变量的引用，最终实现设置TLS配置中选项的功能。

`WithTLS`函数也接受一个函数指针`Option`作为参数，这个函数指针可以用来设置TLS配置中的选项。具体来说，这个函数指针被传递给`tls.Config`类型的变量`config`，然后通过不断地使用`*`运算符获取该变量的引用，最终实现设置TLS配置中选项的功能。不过，`WithTLS`函数的实现比较复杂，因为它需要遍历所有的`NextProtos`字段，并且需要确保所有的NextProtos都是有效的。

`WithDestination`函数接受一个`net.Destination`类型的参数`dest`，用于设置TLS配置中的服务器名称。具体来说，这个函数通过不断地使用`*`运算符获取`dest`变量所表示的服务器地址，然后设置TLS配置中的服务器名称，使得所有的TLS连接都会使用这个服务器名称。

`WithNextProtio`函数接受一个或多个`string`类型的参数`protocol`，用于设置TLS配置中的ALPN选项。具体来说，这个函数通过不断地使用`*`运算符获取`protocol`中的每个元素，然后设置TLS配置中的NextProtos字段，使得所有的TLS连接都会使用指定的ALPN选项。不过，为了确保安全性和兼容性，所有的ALPN选项都必须使用`HTTP/1.1`作为前缀。


```go
// Option for building TLS config.
type Option func(*tls.Config)

// WithDestination sets the server name in TLS config.
func WithDestination(dest net.Destination) Option {
	return func(config *tls.Config) {
		if dest.Address.Family().IsDomain() && config.ServerName == "" {
			config.ServerName = dest.Address.Domain()
		}
	}
}

// WithNextProto sets the ALPN values in TLS config.
func WithNextProto(protocol ...string) Option {
	return func(config *tls.Config) {
		if len(config.NextProtos) == 0 {
			config.NextProtos = protocol
		}
	}
}

```

这段代码定义了一个名为`ConfigFromStreamSettings`的函数，它接受一个`internet.MemoryStreamConfig`类型的参数`settings`。

如果`settings`参数为空，函数返回一个`nil`值。否则，函数尝试从`settings`中获取一个`Config`类型的参数，并返回它。如果从`settings`中无法获取任何`Config`对象，函数将返回`nil`。

函数的作用是获取一个`internet.MemoryStreamConfig`类型的参数`settings`，并返回其中的`Config`。如果`settings`参数为空，函数将返回`nil`；否则，函数将返回`Settings`对象。


```go
// ConfigFromStreamSettings fetches Config from stream settings. Nil if not found.
func ConfigFromStreamSettings(settings *internet.MemoryStreamConfig) *Config {
	if settings == nil {
		return nil
	}
	config, ok := settings.SecuritySettings.(*Config)
	if !ok {
		return nil
	}
	return config
}

```

# `transport/internet/tls/config.pb.go`

这段代码定义了一个名为“tls”的包，该包用于定义与TLS（Transport Layer Security）相关的配置和信息。

具体来说，这段代码实现了以下功能：

1. 导入了多个Go标准库中的相关包，包括“protoc-gen-go”、“protoc”和“proto”。
2. 通过使用“import .proto”语句，将定义的.proto文件导入到函数内部，以便于使用里面的定义。
3. 定义了一个名为“tls”的包，该包的定义在“protoc-gen-go/example/tls/config.proto”文件中。
4. 在定义“tls”包的过程中，使用了Go标准库中“reflect”和“sync”的包，以便于实现TLS的配置和信息。


```go
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
// 	protoc-gen-go v1.25.0
// 	protoc        v3.13.0
// source: transport/internet/tls/config.proto

package tls

import (
	proto "github.com/golang/protobuf/proto"
	protoreflect "google.golang.org/protobuf/reflect/protoreflect"
	protoimpl "google.golang.org/protobuf/runtime/protoimpl"
	reflect "reflect"
	sync "sync"
)

```

这段代码是一个编译时检查，用于确保所生成的代码足够最新。它检查了两个依赖项（protoimpl.EnforceVersion和protoimpl.MaxVersion）的版本是否与当前兼容。

首先，它使用protoimpl.EnforceVersion（20 - protoimpl.MinVersion）来确保生成的代码在protoimpl包的20版本中足够更新。然后，它使用protoimpl.EnforceVersion（protoimpl.MaxVersion - 20）来确保生成的代码在protoimpl包的4版本中足够更新。

接下来，它定义了一个名为Certificate_Usage的枚举类型，用于指定在何种情况下使用特定类型的证书。

最后，它使用一个名为_的常量来存储 Certificate_PackageIsVersion4 的返回值，这是一个编译时检查，用于确保当前使用的证书包版本4足够更新。


```go
const (
	// Verify that this generated code is sufficiently up-to-date.
	_ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
	// Verify that runtime/protoimpl is sufficiently up-to-date.
	_ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// This is a compile-time assertion that a sufficiently up-to-date version
// of the legacy proto package is being used.
const _ = proto.ProtoPackageIsVersion4

type Certificate_Usage int32

const (
	Certificate_ENCIPHERMENT     Certificate_Usage = 0
	Certificate_AUTHORITY_VERIFY Certificate_Usage = 1
	Certificate_AUTHORITY_ISSUE  Certificate_Usage = 2
)

```

这段代码定义了一个名为 "Certificate_Usage" 的枚举类型，包含了三个枚举值，分别对应着 "ENCIPHERMENT"、"AUTHORITY_VERIFY" 和 "AUTHORITY_ISSUE" 这三个用途。每个枚举值都对应了一个名为 "Certificate_Usage_value" 的整数类型，分别对应于 "0"、"1" 和 "2" 这三个数字。

在函数 "Enum" 中，使用了一个名为 "Certificate_Usage" 的变量 x，将其赋值为该枚举类型中对应的枚举值，创建了一个 "Certificate_Usage" 的实例变量 *p。最后，函数返回 *p，即取得了 "Certificate_Usage" 类型的引用。


```go
// Enum value maps for Certificate_Usage.
var (
	Certificate_Usage_name = map[int32]string{
		0: "ENCIPHERMENT",
		1: "AUTHORITY_VERIFY",
		2: "AUTHORITY_ISSUE",
	}
	Certificate_Usage_value = map[string]int32{
		"ENCIPHERMENT":     0,
		"AUTHORITY_VERIFY": 1,
		"AUTHORITY_ISSUE":  2,
	}
)

func (x Certificate_Usage) Enum() *Certificate_Usage {
	p := new(Certificate_Usage)
	*p = x
	return p
}

```

这段代码定义了一个名为 "func" 的函数，它接受一个名为 "x" 的参数，并返回一个字符串类型的函数返回值。

函数的第一个参数 "x" 被传递给 "func" 函数，然后通过调用 "protoimpl.X.EnumStringOf" 函数，将 "x" 的描述符转换为一个枚举类型，再通过调用 "protoreflect.EnumNumber" 函数将枚举类型转换为相应的数字类型，最后将得到的结果返回。

函数的第二个参数 "Certificate_Usage" 被传递给 "func" 函数，然后通过调用 "file_transport_internet_tls_config_proto_enumTypes[0].Descriptor" 函数，获取到一个名为 "Descriptor" 的函数返回值，该函数返回一个名为 "protoreflect.EnumDescriptor" 的接口类型，该接口类型定义了枚举类型的描述符。最后，将获取到的函数返回值返回。

函数的第三个参数 "Certificate_Usage" 被传递给 "func" 函数，然后通过调用自身 "Number" 函数，返回一个名为 "protoreflect.EnumNumber" 的函数返回值，该函数将 "Certificate_Usage" 的值转换为相应的枚举类型。


```go
func (x Certificate_Usage) String() string {
	return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (Certificate_Usage) Descriptor() protoreflect.EnumDescriptor {
	return file_transport_internet_tls_config_proto_enumTypes[0].Descriptor()
}

func (Certificate_Usage) Type() protoreflect.EnumType {
	return &file_transport_internet_tls_config_proto_enumTypes[0]
}

func (x Certificate_Usage) Number() protoreflect.EnumNumber {
	return protoreflect.EnumNumber(x)
}

```

这段代码定义了一个名为 Certificate_Usage 的枚举类型，它用于表示 TLS 证书的使用情况。

该函数返回一个由两个字节组成的 Byte 数组，以及一个包含两个整数的列表，用于表示 TLS 证书的使用情况。第一个字节表示 TLS 证书的状态（使用或未使用），第二个字节表示 TLS 证书是否已配置好（true 或 false）。

函数的实现依赖于一个名为 file_transport_internet_tls_config_proto_rawDescGZIP 的协议描述。但该描述已经被 deprecated，建议使用 Certificate_Usage.Descriptor 来代替。


```go
// Deprecated: Use Certificate_Usage.Descriptor instead.
func (Certificate_Usage) EnumDescriptor() ([]byte, []int) {
	return file_transport_internet_tls_config_proto_rawDescGZIP(), []int{0, 0}
}

type Certificate struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	// TLS certificate in x509 format.
	Certificate []byte `protobuf:"bytes,1,opt,name=Certificate,proto3" json:"Certificate,omitempty"`
	// TLS key in x509 format.
	Key   []byte            `protobuf:"bytes,2,opt,name=Key,proto3" json:"Key,omitempty"`
	Usage Certificate_Usage `protobuf:"varint,3,opt,name=usage,proto3,enum=v2ray.core.transport.internet.tls.Certificate_Usage" json:"usage,omitempty"`
}

```

此代码定义了两个函数：

1. `func (x *Certificate) Reset()` 函数的作用是重置传入的 `x` 对象，将其设置为一个新的 `Certificate` 对象。

2. `func (x *Certificate) String()` 函数的作用是将 `x` 对象转换为字符串，并返回该对象的 `String` 字段。

3. `func (x *Certificate) ProtoMessage()` 函数的作用是返回一个 `Certificate` 对象的 `Message` 类型，以便在消息传递中进行传输。

具体来说，这两个函数都是通过 `Certificate` 类型实现了 `msg.Pointer` 接口，这个接口定义了 `Message` 类型，其中包含了 `Certificate` 类型的相关信息。通过实现这个接口，可以将 `Certificate` 对象作为 `Message` 类型的参数进行传递，从而实现与其他组件的交互。


```go
func (x *Certificate) Reset() {
	*x = Certificate{}
	if protoimpl.UnsafeEnabled {
		mi := &file_transport_internet_tls_config_proto_msgTypes[0]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *Certificate) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*Certificate) ProtoMessage() {}

```

这段代码定义了两个函数，一个是 `func (x *Certificate) ProtoReflect() protoreflect.Message`，另一个是 `func (x *Certificate) Descriptor() ([]byte, []int)`。

第一个函数 `func (x *Certificate) ProtoReflect() protoreflect.Message` 接收一个 `*Certificate` 类型的参数 `x`，并返回一个来自 `file_transport_internet_tls_config_proto_msgTypes` 数组的 `protoreflect.Message` 类型。这个函数的作用是将传入的 `*Certificate` 类型的变量 `x` 进行反向反射，然后将其转换为一个 `protoreflect.Message` 类型，以便在需要时进行调用。

第二个函数 `func (x *Certificate) Descriptor() ([]byte, []int)` 接收一个 `*Certificate` 类型的参数 `x`，并返回其 `descriptor` 字段中的字节切片和编码类型数组。这个函数的作用是将 `*Certificate` 类型的变量 `x` 进行反向反射，然后返回其 `descriptor` 字段中的字节切片和编码类型数组，以便在需要时进行调用。不过，这个函数是过时的，因为已经有更加安全和易于使用的替代函数了，比如 `Certificate.Descriptor`。


```go
func (x *Certificate) ProtoReflect() protoreflect.Message {
	mi := &file_transport_internet_tls_config_proto_msgTypes[0]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use Certificate.ProtoReflect.Descriptor instead.
func (*Certificate) Descriptor() ([]byte, []int) {
	return file_transport_internet_tls_config_proto_rawDescGZIP(), []int{0}
}

```

这段代码定义了三个函数，分别接收一个名为x的Certificate类型的参数。这些函数的作用如下：

1. func(x *Certificate) GetCertificate() []byte {
	如果x不等于 nil，那么它所指的Certificate对象已经被封装为一个字节切片（[]byte），因此直接返回该切片。
2. func(x *Certificate) GetKey() []byte {
	如果x不等于 nil，那么它所指的Certificate对象已经被封装了一个字节切片（[]byte），因此直接返回该切片。
3. func(x *Certificate) GetUsage() Certificate_Usage {
	如果x不等于 nil，那么它所指的Certificate对象已经被封装为一个Certificate_Usage类型的值，因此直接返回该值。


```go
func (x *Certificate) GetCertificate() []byte {
	if x != nil {
		return x.Certificate
	}
	return nil
}

func (x *Certificate) GetKey() []byte {
	if x != nil {
		return x.Key
	}
	return nil
}

func (x *Certificate) GetUsage() Certificate_Usage {
	if x != nil {
		return x.Usage
	}
	return Certificate_ENCIPHERMENT
}

```

这段代码定义了一个名为 Config 的结构体类型，用于配置服务器的安全选项。该 Config 结构体包含了以下字段：

* state：表示服务器当前处于的安全状态，使用 protoimpl.MessageState 类型表示。
* sizeCache：表示服务器的证书缓存大小，使用 protoimpl.SizeCache 类型表示。
* unknownFields：表示服务器未知的选项，使用 protoimpl.UnknownFields 类型表示。

该 Config 结构体定义了一些选项，用于配置服务器的安全选项，包括是否允许自我签名证书、是否允许不安全的加密套接字、允许的证书列表、是否允许服务器的名称等等。通过使用 protobuf，可以确保这些选项在服务器启动时以正确的方式传递给服务器，并且可以使得服务器在启动时检查这些选项，以确保服务器的安全性。


```go
type Config struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	// Whether or not to allow self-signed certificates.
	AllowInsecure bool `protobuf:"varint,1,opt,name=allow_insecure,json=allowInsecure,proto3" json:"allow_insecure,omitempty"`
	// Whether or not to allow insecure cipher suites.
	AllowInsecureCiphers bool `protobuf:"varint,5,opt,name=allow_insecure_ciphers,json=allowInsecureCiphers,proto3" json:"allow_insecure_ciphers,omitempty"`
	// List of certificates to be served on server.
	Certificate []*Certificate `protobuf:"bytes,2,rep,name=certificate,proto3" json:"certificate,omitempty"`
	// Override server name.
	ServerName string `protobuf:"bytes,3,opt,name=server_name,json=serverName,proto3" json:"server_name,omitempty"`
	// Lists of string as ALPN values.
	NextProtocol []string `protobuf:"bytes,4,rep,name=next_protocol,json=nextProtocol,proto3" json:"next_protocol,omitempty"`
	// Whether or not to disable session (ticket) resumption.
	DisableSessionResumption bool `protobuf:"varint,6,opt,name=disable_session_resumption,json=disableSessionResumption,proto3" json:"disable_session_resumption,omitempty"`
	// If true, root certificates on the system will not be loaded for
	// verification.
	DisableSystemRoot bool `protobuf:"varint,7,opt,name=disable_system_root,json=disableSystemRoot,proto3" json:"disable_system_root,omitempty"`
}

```

这段代码定义了两个函数：

1. `Reset()`，该函数接受一个指向 `Config` 类型对象的 `x` 参数。函数的作用是将 `x` 对象中的值设为一个新的 `Config` 对象，并检查 `protoimpl.UnsafeEnabled` 是否为 `true`。如果是，则执行以下操作：

- 检查 `x` 对象是否指向 `file_transport_internet_tls_config_proto_msgTypes[1]` 类型对象。
- 如果 `x` 对象是指定的 `Config` 类型对象的别名，则将其存储为 `mi` 变量，并将其设置为 `x` 对象所属的 `MessageInfo` 类型的存储源。

2. `String()`，该函数接受一个指向 `Config` 类型对象的 `x` 参数。函数的作用是返回 `x` 对象的字符串表示形式，与 `protoimpl.X.MessageStringOf()` 函数配合使用。

3. `ProtoMessage()` 函数接受一个指向 `Config` 类型对象的 `x` 参数，但没有具体的实现。这个函数的作用是在 `x` 对象上执行 `protoimpl.X.MessageProxy()` 函数，但没有指定具体的 `Message` 类型。


```go
func (x *Config) Reset() {
	*x = Config{}
	if protoimpl.UnsafeEnabled {
		mi := &file_transport_internet_tls_config_proto_msgTypes[1]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *Config) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*Config) ProtoMessage() {}

```

这段代码定义了两个函数，分别接收一个*Config类型的参数x，并返回相应的函数值。

第一个函数是protoreflect.Message类型的函数，其作用是将传入的*Config类型的参数x转换为相应的protoreflect.Message类型，然后返回该Message类型。实现方式如下：

1. 判断是否启用 protoimpl.UnsafeEnabled 选项，如果启用，则执行以下操作：

	1. 获取一个名为 mi 的指针，该指针指向一个名为 file_transport_internet_tls_config_proto_msgTypes 的常量，该常量的值为 1。

	2. 如果 x 不为空，执行以下操作：

		1. 创建一个名为 ms 的新 Message 实例。

		2. 如果已经配置了 protoimpl.UnsafeEnabled，则执行以下操作：

			1. 从 x 的 MessageStateOf 函数中获取消息信息。

			2. 如果消息信息为空，则执行以下操作：

				1. 将消息信息设置为 mi。

			2. 返回 ms。

		3. 如果 x 不为空，执行以下操作：

				1. 从 x 的 MessageOf 函数中获取消息信息。

				2. 如果已经配置了 protoimpl.UnsafeEnabled，则执行以下操作：

					1. 从 x 的 MessageStateOf 函数中获取消息信息。

					2. 如果消息信息为空，则执行以下操作：

						1. 将消息信息设置为 mi。

						2. 返回 mi。

					3. 返回 x。

2. 如果 x 为空，返回一个名为 file_transport_internet_tls_config_proto_rawDescGZIP 的字节切片和一个名为 []int 的整数切片，其中 file_transport_internet_tls_config_proto_rawDescGZIP 的值为 file_transport_internet_tls_config_proto_rawDescGZIP，[]int 中的第一个值为 1。

第二个函数是一个空函数，其作用是返回 Config 类型的参数 x 的描述符，该描述符用于在代码中使用，通常不会直接输出。该函数的实现方式如下：

1. 从 Config 类型中获取描述符，并将其存储在 Config.Descriptor 中。

2. 如果 Config.Descriptor 为空，返回一个名为 file_transport_internet_tls_config_proto_rawDescGZIP 的字节切片和一个名为 []int 的整数切片，其中 file_transport_internet_tls_config_proto_rawDescGZIP 的值为 file_transport_internet_tls_config_proto_rawDescGZIP，[]int 中的第一个值为 1。

如果需要使用 Config.Descriptor 中定义的值，只需要在代码中直接使用 Config.Descriptor[0] 即可。


```go
func (x *Config) ProtoReflect() protoreflect.Message {
	mi := &file_transport_internet_tls_config_proto_msgTypes[1]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use Config.ProtoReflect.Descriptor instead.
func (*Config) Descriptor() ([]byte, []int) {
	return file_transport_internet_tls_config_proto_rawDescGZIP(), []int{1}
}

```

这段代码定义了三个函数，分别接收一个`Config`类型的参数`x`，并返回一个布尔值。

第一个函数是`func (x *Config) GetAllowInsecure() bool`，这个函数的作用是获取`x`对象允许加密安全性的方式是否为`true`。函数的实现使用了`x`对象自身来获取其安全性设置，如果`x`对象不为`nil`，则返回`true`，否则返回`false`。

第二个函数是`func (x *Config) GetAllowInsecureCiphers() bool`，与第一个函数类似，但是这个函数的作用是获取`x`对象允许的加密算法是否为`true`。函数的实现也是使用了`x`对象自身来获取其安全性设置，如果`x`对象不为`nil`，则返回`true`，否则返回`false`。

第三个函数是`func (x *Config) GetCertificate() []*Certificate`，这个函数的作用是获取`x`对象返回的证书链。函数的实现使用了`x`对象自身来获取其证书链，如果`x`对象不为`nil`，则返回一个包含所有`Certificate`类型的切片，否则返回一个`nil`表示没有证书链。


```go
func (x *Config) GetAllowInsecure() bool {
	if x != nil {
		return x.AllowInsecure
	}
	return false
}

func (x *Config) GetAllowInsecureCiphers() bool {
	if x != nil {
		return x.AllowInsecureCiphers
	}
	return false
}

func (x *Config) GetCertificate() []*Certificate {
	if x != nil {
		return x.Certificate
	}
	return nil
}

```

这是一段 Go 语言中的函数指针变量（function pointer）类型。它定义了三个名为 "func" 的函数，它们的接收者分别为 Config、Config 和 Config，但它们的参数都是一个指向 Config 类型结构的指针变量 x。

具体来说，这段代码实现了一个简单的数据结构和函数，用于管理网络会话的配置信息。这些函数会根据传入的参数 x 的值来执行相应的操作，包括获取服务器名称、尝试获取下一个协议、以及检测是否啟用会话保留等。通过这种方式，可以将一些数据冗余的代码变得更为简洁且易于维护。


```go
func (x *Config) GetServerName() string {
	if x != nil {
		return x.ServerName
	}
	return ""
}

func (x *Config) GetNextProtocol() []string {
	if x != nil {
		return x.NextProtocol
	}
	return nil
}

func (x *Config) GetDisableSessionResumption() bool {
	if x != nil {
		return x.DisableSessionResumption
	}
	return false
}

```

It looks like a吐蕃文字衔。 这是一段来源于网络的文字，你可能感兴趣。

然而，我并不能提供关于它的详细信息，因为我不支持或查阅相关信息。


```go
func (x *Config) GetDisableSystemRoot() bool {
	if x != nil {
		return x.DisableSystemRoot
	}
	return false
}

var File_transport_internet_tls_config_proto protoreflect.FileDescriptor

var file_transport_internet_tls_config_proto_rawDesc = []byte{
	0x0a, 0x23, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65,
	0x72, 0x6e, 0x65, 0x74, 0x2f, 0x74, 0x6c, 0x73, 0x2f, 0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e,
	0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x21, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72,
	0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65,
	0x72, 0x6e, 0x65, 0x74, 0x2e, 0x74, 0x6c, 0x73, 0x22, 0xd3, 0x01, 0x0a, 0x0b, 0x43, 0x65, 0x72,
	0x74, 0x69, 0x66, 0x69, 0x63, 0x61, 0x74, 0x65, 0x12, 0x20, 0x0a, 0x0b, 0x43, 0x65, 0x72, 0x74,
	0x69, 0x66, 0x69, 0x63, 0x61, 0x74, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0c, 0x52, 0x0b, 0x43,
	0x65, 0x72, 0x74, 0x69, 0x66, 0x69, 0x63, 0x61, 0x74, 0x65, 0x12, 0x10, 0x0a, 0x03, 0x4b, 0x65,
	0x79, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0c, 0x52, 0x03, 0x4b, 0x65, 0x79, 0x12, 0x4a, 0x0a, 0x05,
	0x75, 0x73, 0x61, 0x67, 0x65, 0x18, 0x03, 0x20, 0x01, 0x28, 0x0e, 0x32, 0x34, 0x2e, 0x76, 0x32,
	0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f,
	0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x74, 0x6c, 0x73, 0x2e,
	0x43, 0x65, 0x72, 0x74, 0x69, 0x66, 0x69, 0x63, 0x61, 0x74, 0x65, 0x2e, 0x55, 0x73, 0x61, 0x67,
	0x65, 0x52, 0x05, 0x75, 0x73, 0x61, 0x67, 0x65, 0x22, 0x44, 0x0a, 0x05, 0x55, 0x73, 0x61, 0x67,
	0x65, 0x12, 0x10, 0x0a, 0x0c, 0x45, 0x4e, 0x43, 0x49, 0x50, 0x48, 0x45, 0x52, 0x4d, 0x45, 0x4e,
	0x54, 0x10, 0x00, 0x12, 0x14, 0x0a, 0x10, 0x41, 0x55, 0x54, 0x48, 0x4f, 0x52, 0x49, 0x54, 0x59,
	0x5f, 0x56, 0x45, 0x52, 0x49, 0x46, 0x59, 0x10, 0x01, 0x12, 0x13, 0x0a, 0x0f, 0x41, 0x55, 0x54,
	0x48, 0x4f, 0x52, 0x49, 0x54, 0x59, 0x5f, 0x49, 0x53, 0x53, 0x55, 0x45, 0x10, 0x02, 0x22, 0xeb,
	0x02, 0x0a, 0x06, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x25, 0x0a, 0x0e, 0x61, 0x6c, 0x6c,
	0x6f, 0x77, 0x5f, 0x69, 0x6e, 0x73, 0x65, 0x63, 0x75, 0x72, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28,
	0x08, 0x52, 0x0d, 0x61, 0x6c, 0x6c, 0x6f, 0x77, 0x49, 0x6e, 0x73, 0x65, 0x63, 0x75, 0x72, 0x65,
	0x12, 0x34, 0x0a, 0x16, 0x61, 0x6c, 0x6c, 0x6f, 0x77, 0x5f, 0x69, 0x6e, 0x73, 0x65, 0x63, 0x75,
	0x72, 0x65, 0x5f, 0x63, 0x69, 0x70, 0x68, 0x65, 0x72, 0x73, 0x18, 0x05, 0x20, 0x01, 0x28, 0x08,
	0x52, 0x14, 0x61, 0x6c, 0x6c, 0x6f, 0x77, 0x49, 0x6e, 0x73, 0x65, 0x63, 0x75, 0x72, 0x65, 0x43,
	0x69, 0x70, 0x68, 0x65, 0x72, 0x73, 0x12, 0x50, 0x0a, 0x0b, 0x63, 0x65, 0x72, 0x74, 0x69, 0x66,
	0x69, 0x63, 0x61, 0x74, 0x65, 0x18, 0x02, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x2e, 0x2e, 0x76, 0x32,
	0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f,
	0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x74, 0x6c, 0x73, 0x2e,
	0x43, 0x65, 0x72, 0x74, 0x69, 0x66, 0x69, 0x63, 0x61, 0x74, 0x65, 0x52, 0x0b, 0x63, 0x65, 0x72,
	0x74, 0x69, 0x66, 0x69, 0x63, 0x61, 0x74, 0x65, 0x12, 0x1f, 0x0a, 0x0b, 0x73, 0x65, 0x72, 0x76,
	0x65, 0x72, 0x5f, 0x6e, 0x61, 0x6d, 0x65, 0x18, 0x03, 0x20, 0x01, 0x28, 0x09, 0x52, 0x0a, 0x73,
	0x65, 0x72, 0x76, 0x65, 0x72, 0x4e, 0x61, 0x6d, 0x65, 0x12, 0x23, 0x0a, 0x0d, 0x6e, 0x65, 0x78,
	0x74, 0x5f, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x18, 0x04, 0x20, 0x03, 0x28, 0x09,
	0x52, 0x0c, 0x6e, 0x65, 0x78, 0x74, 0x50, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x12, 0x3c,
	0x0a, 0x1a, 0x64, 0x69, 0x73, 0x61, 0x62, 0x6c, 0x65, 0x5f, 0x73, 0x65, 0x73, 0x73, 0x69, 0x6f,
	0x6e, 0x5f, 0x72, 0x65, 0x73, 0x75, 0x6d, 0x70, 0x74, 0x69, 0x6f, 0x6e, 0x18, 0x06, 0x20, 0x01,
	0x28, 0x08, 0x52, 0x18, 0x64, 0x69, 0x73, 0x61, 0x62, 0x6c, 0x65, 0x53, 0x65, 0x73, 0x73, 0x69,
	0x6f, 0x6e, 0x52, 0x65, 0x73, 0x75, 0x6d, 0x70, 0x74, 0x69, 0x6f, 0x6e, 0x12, 0x2e, 0x0a, 0x13,
	0x64, 0x69, 0x73, 0x61, 0x62, 0x6c, 0x65, 0x5f, 0x73, 0x79, 0x73, 0x74, 0x65, 0x6d, 0x5f, 0x72,
	0x6f, 0x6f, 0x74, 0x18, 0x07, 0x20, 0x01, 0x28, 0x08, 0x52, 0x11, 0x64, 0x69, 0x73, 0x61, 0x62,
	0x6c, 0x65, 0x53, 0x79, 0x73, 0x74, 0x65, 0x6d, 0x52, 0x6f, 0x6f, 0x74, 0x42, 0x74, 0x0a, 0x25,
	0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74,
	0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65,
	0x74, 0x2e, 0x74, 0x6c, 0x73, 0x50, 0x01, 0x5a, 0x25, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
	0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72,
	0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2f, 0x74, 0x6c, 0x73, 0xaa, 0x02,
	0x21, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x54, 0x72, 0x61, 0x6e,
	0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x49, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x54,
	0x6c, 0x73, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
}

```

此代码定义了一个名为file_transport_internet_tls_config_proto_rawDescOnce的变量，其作用是确保一个名为file_transport_internet_tls_config_proto_rawDesc的实体的可见性，该实体的类型为v2ray.core.transport.internet.tls.ConfigurationProtocolOnce。

函数file_transport_internet_tls_config_proto_rawDescGZIP返回一个字节切片，其中包含经过GZIP压缩的file_transport_internet_tls_config_proto_rawDescData。


```go
var (
	file_transport_internet_tls_config_proto_rawDescOnce sync.Once
	file_transport_internet_tls_config_proto_rawDescData = file_transport_internet_tls_config_proto_rawDesc
)

func file_transport_internet_tls_config_proto_rawDescGZIP() []byte {
	file_transport_internet_tls_config_proto_rawDescOnce.Do(func() {
		file_transport_internet_tls_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_tls_config_proto_rawDescData)
	})
	return file_transport_internet_tls_config_proto_rawDescData
}

var file_transport_internet_tls_config_proto_enumTypes = make([]protoimpl.EnumInfo, 1)
var file_transport_internet_tls_config_proto_msgTypes = make([]protoimpl.MessageInfo, 2)
var file_transport_internet_tls_config_proto_goTypes = []interface{}{
	(Certificate_Usage)(0), // 0: v2ray.core.transport.internet.tls.Certificate.Usage
	(*Certificate)(nil),    // 1: v2ray.core.transport.internet.tls.Certificate
	(*Config)(nil),         // 2: v2ray.core.transport.internet.tls.Config
}
```

This is a Go file that exports some fenc阳气质量体验几个v公主想要学习了解下火通过火眼术提升的实现火眼精神看更清晰脱敏实现少女或少年宫廷之间的事情分享给想学习了解下火通过火眼术提升的团队领导人火眼术需要团队合作才能维持全局的高度了解相关职业在未来的一段时间内，我将帮助团队领导人火眼术需要团队合作才能维持全局的高度了解相关职业在未来的一段时间内，我将帮助团队领导人提升火的技能和经验，让他们可以更好地带领团队走向胜利。提升火焰的技能和经验，让他们的团队更加强大。


```go
var file_transport_internet_tls_config_proto_depIdxs = []int32{
	0, // 0: v2ray.core.transport.internet.tls.Certificate.usage:type_name -> v2ray.core.transport.internet.tls.Certificate.Usage
	1, // 1: v2ray.core.transport.internet.tls.Config.certificate:type_name -> v2ray.core.transport.internet.tls.Certificate
	2, // [2:2] is the sub-list for method output_type
	2, // [2:2] is the sub-list for method input_type
	2, // [2:2] is the sub-list for extension type_name
	2, // [2:2] is the sub-list for extension extendee
	0, // [0:2] is the sub-list for field type_name
}

func init() { file_transport_internet_tls_config_proto_init() }
func file_transport_internet_tls_config_proto_init() {
	if File_transport_internet_tls_config_proto != nil {
		return
	}
	if !protoimpl.UnsafeEnabled {
		file_transport_internet_tls_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*Certificate); i {
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
		file_transport_internet_tls_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*Config); i {
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
			RawDescriptor: file_transport_internet_tls_config_proto_rawDesc,
			NumEnums:      1,
			NumMessages:   2,
			NumExtensions: 0,
			NumServices:   0,
		},
		GoTypes:           file_transport_internet_tls_config_proto_goTypes,
		DependencyIndexes: file_transport_internet_tls_config_proto_depIdxs,
		EnumInfos:         file_transport_internet_tls_config_proto_enumTypes,
		MessageInfos:      file_transport_internet_tls_config_proto_msgTypes,
	}.Build()
	File_transport_internet_tls_config_proto = out.File
	file_transport_internet_tls_config_proto_rawDesc = nil
	file_transport_internet_tls_config_proto_goTypes = nil
	file_transport_internet_tls_config_proto_depIdxs = nil
}

```

# `transport/internet/tls/config_other.go`

这段代码是一个 Go 语言编写的 package，名为 "tls"。它定义了一个名为 "rootCertsCache" 的结构体，该结构体实现了一个用于存储 TLS 根证书的并发池。

具体来说，该代码以下两个函数为重点：

1. `build` 函数：用于创建一个根证书池，并将其存储在名为 "rootCertsCache" 的同步锁中。
2. `confonly` 函数：在 `build` 函数的基础上，创建一个名为 "confonly" 的文件，并将根证书池的定义存储到该文件中。这样做是为了在 `build` 函数定义的根证书池中创建根证书，但不会在根证书池中创建或写入任何证书。

通过调用这两个函数，你可以使用 Go 语言编写 TLS 根证书的池，从而轻松地在应用程序中使用 TLS 根证书。


```go
// +build !windows
// +build !confonly

package tls

import (
	"crypto/x509"
	"sync"
)

type rootCertsCache struct {
	sync.Mutex
	pool *x509.CertPool
}

```

此函数的作用是加载一个根证书缓存链中的证书。它首先对缓存链进行加锁操作，以确保在尝试获取或修改证书时不会被其他操作干扰。然后，它检查缓存链中是否有证书，如果没有，则代表函数无法返回任何有效的证书，并返回一个错误。如果缓存链中包含证书，则返回该证书池，如果过程中出现错误，则返回一个错误。

具体来说，该函数首先创建一个名为c的指针，并使用该指针调用一个名为load的函数。load函数的作用是在根证书缓存链中查找证书。如果找到了证书，则将其返回，否则返回一个错误。在函数中，首先检查缓存链中是否有证书，如果没有，则创建一个名为x509的CertPool对象并返回，其中x509表示使用的是系统根证书缓存。如果找到了证书，则将该证书池返回，并返回该根证书池。


```go
func (c *rootCertsCache) load() (*x509.CertPool, error) {
	c.Lock()
	defer c.Unlock()

	if c.pool != nil {
		return c.pool, nil
	}

	pool, err := x509.SystemCertPool()
	if err != nil {
		return nil, err
	}
	c.pool = pool
	return pool, nil
}

```

这段代码的作用是获取Kubernetes客户端配置中的证书颁发池。

首先，它检查是否启用了系统根，如果是，则使用从根目录加载自签名证书的方式获取证书池。如果不是，那么就从根目录加载证书。

然后，它会检查证书池是否为空，如果是，那么就从根目录加载证书。

接下来，它会使用系统证书池加载证书。如果在这个过程中出现错误，它将从警告中获取错误并返回。

最后，它循环遍历证书并将它们添加到根证书池中。这样做，即使已经有证书被添加到根证书池中，仍然可以添加新的证书。


```go
var rootCerts rootCertsCache

func (c *Config) getCertPool() (*x509.CertPool, error) {
	if c.DisableSystemRoot {
		return c.loadSelfCertPool()
	}

	if len(c.Certificate) == 0 {
		return rootCerts.load()
	}

	pool, err := x509.SystemCertPool()
	if err != nil {
		return nil, newError("system root").AtWarning().Base(err)
	}
	for _, cert := range c.Certificate {
		if !pool.AppendCertsFromPEM(cert.Certificate) {
			return nil, newError("append cert to root").AtWarning().Base(err)
		}
	}
	return pool, err
}

```

# `transport/internet/tls/config_test.go`

这段代码是一个名为 "tls_test" 的包，用于测试 TLS(Transport Layer Security) 证书颁发功能。

该代码首先导入了 "crypto/tls" 和 "crypto/x509" 包，分别用于生成 TLS 证书和验证证书的有效性。

接着导入了 "testing" 和 "v2ray.com/core/common" 包，用于测试和导入关于 TLS 证书的一些常见组件。

然后定义了一个名为 "TestCertificateIssuing" 的函数，该函数使用 "ParseCertificate" 函数从给定的本地证书(使用 "cert.Authority(true)")中提取证书，然后将其设置为 "CertificateIssuing" 的使用方式，接着定义了一个 "c" 变量，用于在配置中指定证书，并使用 "GetTLSConfig" 函数获取 TLS 配置，然后使用 "GetCertificate" 函数从 TLS 配置获取证书。

最后，使用 "tlsConfig.GetCertificate" 函数获取 TLS 配置中的证书，并使用 "x509.ParseCertificate" 函数验证证书的有效性，如果证书在当前时间之后没有被签发，则会输出错误。


```go
package tls_test

import (
	gotls "crypto/tls"
	"crypto/x509"
	"testing"
	"time"

	"v2ray.com/core/common"
	"v2ray.com/core/common/protocol/tls/cert"
	. "v2ray.com/core/transport/internet/tls"
)

func TestCertificateIssuing(t *testing.T) {
	certificate := ParseCertificate(cert.MustGenerate(nil, cert.Authority(true), cert.KeyUsage(x509.KeyUsageCertSign)))
	certificate.Usage = Certificate_AUTHORITY_ISSUE

	c := &Config{
		Certificate: []*Certificate{
			certificate,
		},
	}

	tlsConfig := c.GetTLSConfig()
	v2rayCert, err := tlsConfig.GetCertificate(&gotls.ClientHelloInfo{
		ServerName: "www.v2ray.com",
	})
	common.Must(err)

	x509Cert, err := x509.ParseCertificate(v2rayCert.Certificate[0])
	common.Must(err)
	if !x509Cert.NotAfter.After(time.Now()) {
		t.Error("NotAfter: ", x509Cert.NotAfter)
	}
}

```

该代码是一个名为 "TestExpiredCertificate" 的测试函数，用于测试证书过期的情况。它的作用是生成一个过期的证书并将其用于测试。

具体来说，代码的作用如下：

1. 生成一个证书并将其用于测试，使用 cert.MustGenerate() 函数生成一个自签名证书，使用 cert.Authority() 函数设置证书的签名用途为自签名，使用 cert.KeyUsage() 函数设置证书用于签名。然后使用 cert.MustGenerate() 函数再次生成同一个证书，设置其过期的时间为 2 分钟后的现在时间，并将其用于测试。

2. 解析测试中使用的证书，并检查其是否过期。使用 ParseCertificate() 函数加载证书，并使用 certificate.Usage 字段设置其用途为测试，然后使用 certificate.GetTLSConfig() 函数获取其 TLS 配置，最后使用 tlsConfig.GetCertificate() 函数获取测试证书的 TLS 证书。然后使用 x509.ParseCertificate() 函数解析证书，并检查其是否过期。

3. 如果测试证书过期，则输出错误信息。

因此，该代码的主要目的是测试证书过期的情况，并确保在测试中生成过期的证书时，可以正确地获取其 TLS 证书并解析其安全性。


```go
func TestExpiredCertificate(t *testing.T) {
	caCert := cert.MustGenerate(nil, cert.Authority(true), cert.KeyUsage(x509.KeyUsageCertSign))
	expiredCert := cert.MustGenerate(caCert, cert.NotAfter(time.Now().Add(time.Minute*-2)), cert.CommonName("www.v2ray.com"), cert.DNSNames("www.v2ray.com"))

	certificate := ParseCertificate(caCert)
	certificate.Usage = Certificate_AUTHORITY_ISSUE

	certificate2 := ParseCertificate(expiredCert)

	c := &Config{
		Certificate: []*Certificate{
			certificate,
			certificate2,
		},
	}

	tlsConfig := c.GetTLSConfig()
	v2rayCert, err := tlsConfig.GetCertificate(&gotls.ClientHelloInfo{
		ServerName: "www.v2ray.com",
	})
	common.Must(err)

	x509Cert, err := x509.ParseCertificate(v2rayCert.Certificate[0])
	common.Must(err)
	if !x509Cert.NotAfter.After(time.Now()) {
		t.Error("NotAfter: ", x509Cert.NotAfter)
	}
}

```

该代码的作用是测试客户端证书颁发功能是否正常。

具体来说，代码首先创建了一个名为`Config`的上下文对象，该上下文对象包含一个名为`AllowInsecureCiphers`的布尔字段，其值为`true`，允许使用不安全的加密套件。然后，代码创建了一个名为`tlsConfig`的`Config`上下文对象的副本，该上下文对象包含了允许使用不安全加密套件的配置。

接下来，代码使用`c.GetTLSConfig()`方法获取了`tlsConfig`中的`CipherSuites`字段，如果`CipherSuites`字段为空，则说明存在配置错误，会输出错误信息。否则，代码会尝试使用`tlsConfig.CipherSuites`数组中的第一个证书，如果该证书的`CipherSuites`数组包含多个证书，则说明存在配置错误，会输出错误信息。

接下来，代码定义了一个名为`BenchmarkCertificateIssuing`的测试函数，该函数使用了`testing.B`和`ParseCertificate`和`Certificate_AUTHORITY_ISSUE`函数。`BenchmarkCertificateIssuing`函数的作用是测试客户端证书签发功能是否正常，具体来说，它会创建一个名为`certificate`的临时证书，并使用该证书的签发权限设置上下文对象的`Certificate`字段，然后使用`c.GetTLSConfig()`方法获取了配置对象中的`Certificate`字段，设置其`CipherSuites`字段为空，并使用该配置对象的`GetCertificate()`方法获取了客户端证书，并设置其使用该证书的签发权限。最后，函数使用循环语句，对配置对象中的证书使用`GetCertificate()`方法获取，并使用`tlsConfig.GetCertificate(&gotls.ClientHelloInfo{ServerName: "www.v2ray.com"})`方法获取，循环设置服务器名称，直到所有证书都被设置完成，并输出测试结果。

最后，代码创建了一个名为`func TestInsecureCertificates(t *testing.T)`的测试函数，该函数的作用是测试客户端证书签发功能是否正常，与上面定义的`BenchmarkCertificateIssuing`函数类似，但会输出配置错误信息，以便进行调试。


```go
func TestInsecureCertificates(t *testing.T) {
	c := &Config{
		AllowInsecureCiphers: true,
	}

	tlsConfig := c.GetTLSConfig()
	if len(tlsConfig.CipherSuites) > 0 {
		t.Fatal("Unexpected tls cipher suites list: ", tlsConfig.CipherSuites)
	}
}

func BenchmarkCertificateIssuing(b *testing.B) {
	certificate := ParseCertificate(cert.MustGenerate(nil, cert.Authority(true), cert.KeyUsage(x509.KeyUsageCertSign)))
	certificate.Usage = Certificate_AUTHORITY_ISSUE

	c := &Config{
		Certificate: []*Certificate{
			certificate,
		},
	}

	tlsConfig := c.GetTLSConfig()
	lenCerts := len(tlsConfig.Certificates)

	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		_, _ = tlsConfig.GetCertificate(&gotls.ClientHelloInfo{
			ServerName: "www.v2ray.com",
		})
		delete(tlsConfig.NameToCertificate, "www.v2ray.com")
		tlsConfig.Certificates = tlsConfig.Certificates[:lenCerts]
	}
}

```