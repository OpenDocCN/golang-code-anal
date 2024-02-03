# `v2ray-core\infra\control\tlsping.go`

```go
package control

import (
    "crypto/tls"  // 导入加密传输层协议包
    "crypto/x509"  // 导入加密证书包
    "flag"  // 导入命令行参数解析包
    "fmt"  // 导入格式化包
    "net"  // 导入网络包

    "v2ray.com/core/common"  // 导入自定义包
)

type TlsPingCommand struct{}  // 定义 TlsPingCommand 结构体

func (c *TlsPingCommand) Name() string {  // 定义 Name 方法
    return "tlsping"  // 返回字符串 "tlsping"
}

func (c *TlsPingCommand) Description() Description {  // 定义 Description 方法
    return Description{  // 返回 Description 结构体
        Short: "Ping the domain with TLS handshake",  // 设置 Short 字段
        Usage: []string{"v2ctl tlsping <domain> --ip <ip>"},  // 设置 Usage 字段
    }
}

func printCertificates(certs []*x509.Certificate) {  // 定义 printCertificates 函数，参数为证书数组
    for _, cert := range certs {  // 遍历证书数组
        if len(cert.DNSNames) == 0 {  // 如果证书中的 DNSNames 字段为空
            continue  // 继续下一次循环
        }
        fmt.Println("Allowed domains: ", cert.DNSNames)  // 打印允许的域名
    }
}

func (c *TlsPingCommand) Execute(args []string) error {  // 定义 Execute 方法，参数为字符串数组
    fs := flag.NewFlagSet(c.Name(), flag.ContinueOnError)  // 创建命令行参数解析对象
    ipStr := fs.String("ip", "", "IP address of the domain")  // 定义命令行参数 -ip，返回字符串指针

    if err := fs.Parse(args); err != nil {  // 解析命令行参数
        return newError("flag parsing").Base(err)  // 返回解析错误
    }

    if fs.NArg() < 1 {  // 如果参数个数小于 1
        return newError("domain not specified")  // 返回域名未指定错误
    }

    domain := fs.Arg(0)  // 获取第一个参数作为域名
    fmt.Println("Tls ping: ", domain)  // 打印 Tls ping: 域名

    var ip net.IP  // 定义 IP 地址变量
    if len(*ipStr) > 0 {  // 如果 IP 地址字符串长度大于 0
        v := net.ParseIP(*ipStr)  // 解析 IP 地址字符串
        if v == nil {  // 如果解析失败
            return newError("invalid IP: ", *ipStr)  // 返回无效 IP 地址错误
        }
        ip = v  // 将解析后的 IP 地址赋值给 ip
    } else {  // 如果 IP 地址字符串长度为 0
        v, err := net.ResolveIPAddr("ip", domain)  // 解析域名对应的 IP 地址
        if err != nil {  // 如果解析失败
            return newError("resolve IP").Base(err)  // 返回解析 IP 地址错误
        }
        ip = v.IP  // 将解析后的 IP 地址赋值给 ip
    }
    fmt.Println("Using IP: ", ip.String())  // 打印使用的 IP 地址

    fmt.Println("-------------------")  // 打印分隔线
    fmt.Println("Pinging without SNI")  // 打印提示信息
}
    {
        // 使用TCP协议连接指定的IP和端口
        tcpConn, err := net.DialTCP("tcp", nil, &net.TCPAddr{IP: ip, Port: 443})
        // 如果连接出现错误，返回一个新的错误对象
        if err != nil {
            return newError("dial tcp").Base(err)
        }
        // 使用TLS协议包装TCP连接
        tlsConn := tls.Client(tcpConn, &tls.Config{
            InsecureSkipVerify: true, // 跳过证书验证
            NextProtos:         []string{"http/1.1"}, // 指定下一个协议
            MaxVersion:         tls.VersionTLS12, // 指定最大TLS版本
            MinVersion:         tls.VersionTLS12, // 指定最小TLS版本
        })
        // 进行TLS握手
        err = tlsConn.Handshake()
        // 如果握手出现错误，打印错误信息
        if err != nil {
            fmt.Println("Handshake failure: ", err)
        } else {
            // 握手成功时打印成功信息，并打印对等证书信息
            fmt.Println("Handshake succeeded")
            printCertificates(tlsConn.ConnectionState().PeerCertificates)
        }
        // 关闭TLS连接
        tlsConn.Close()
    }

    // 打印分隔线
    fmt.Println("-------------------")
    // 打印SNI的ping信息
    fmt.Println("Pinging with SNI")
    {
        // 再次使用TCP协议连接指定的IP和端口
        tcpConn, err := net.DialTCP("tcp", nil, &net.TCPAddr{IP: ip, Port: 443})
        // 如果连接出现错误，返回一个新的错误对象
        if err != nil {
            return newError("dial tcp").Base(err)
        }
        // 使用TLS协议包装TCP连接，同时指定服务器名称
        tlsConn := tls.Client(tcpConn, &tls.Config{
            ServerName: domain, // 指定服务器名称
            NextProtos: []string{"http/1.1"}, // 指定下一个协议
            MaxVersion: tls.VersionTLS12, // 指定最大TLS版本
            MinVersion: tls.VersionTLS12, // 指定最小TLS版本
        })
        // 进行TLS握手
        err = tlsConn.Handshake()
        // 如果握手出现错误，打印错误信息
        if err != nil {
            fmt.Println("handshake failure: ", err)
        } else {
            // 握手成功时打印成功信息，并打印对等证书信息
            fmt.Println("handshake succeeded")
            printCertificates(tlsConn.ConnectionState().PeerCertificates)
        }
        // 关闭TLS连接
        tlsConn.Close()
    }

    // 打印TLS ping完成信息
    fmt.Println("Tls ping finished")

    // 返回空值
    return nil
# 初始化函数，用于注册 TlsPingCommand 命令
func init() {
    # 调用 common 包的 RegisterCommand 函数注册 TlsPingCommand 命令，并使用 common 包的 Must 函数确保注册成功
    common.Must(RegisterCommand(&TlsPingCommand{}))
}
```