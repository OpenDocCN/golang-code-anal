# `trojan-go\tunnel\tls\tls_test.go`

```go
package tls

import (
    "context"  // 上下文包，用于跟踪请求的上下文
    "net"  // 网络包，提供了基本的网络 I/O 接口
    "os"  // 操作系统包，提供了与操作系统交互的功能
    "sync"  // 同步包，提供了基本的同步原语
    "testing"  // 测试包，提供了对 Go 程序进行单元测试的支持

    "github.com/p4gefau1t/trojan-go/common"  // 引入 trojan-go 项目的 common 包
    "github.com/p4gefau1t/trojan-go/config"  // 引入 trojan-go 项目的 config 包
    "github.com/p4gefau1t/trojan-go/test/util"  // 引入 trojan-go 项目的测试工具包
    "github.com/p4gefau1t/trojan-go/tunnel/freedom"  // 引入 trojan-go 项目的自由隧道包
    "github.com/p4gefau1t/trojan-go/tunnel/transport"  // 引入 trojan-go 项目的传输隧道包
)

var rsa2048Cert = `
-----BEGIN CERTIFICATE-----
MIIC5TCCAc2gAwIBAgIJAJqNVe6g/10vMA0GCSqGSIb3DQEBCwUAMBQxEjAQBgNV
BAMMCWxvY2FsaG9zdDAeFw0yMTA5MTQwNjE1MTFaFw0yNjA5MTMwNjE1MTFaMBQx
EjAQBgNVBAMMCWxvY2FsaG9zdDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoC
ggEBAK7bupJ8tmHM3shQ/7N730jzpRsXdNiBxq/Jxx8j+vB3AcxuP5bjXQZqS6YR
5W5vrfLlegtq1E/mmaI3Ht0RfIlzev04Dua9PWmIQJD801nEPknbfgCLXDh+pYr2
sfg8mUh3LjGtrxyH+nmbTjWg7iWSKohmZ8nUDcX94Llo5FxibMAz8OsAwOmUueCH
jP3XswZYHEy+OOP3K0ZEiJy0f5T6ZXk9OWYuPN4VQKJx1qrc9KzZtSPHwqVdkGUi
ase9tOPA4aMutzt0btgW7h7UrvG6C1c/Rr1BxdiYq1EQ+yypnAlyToVQSNbo67zz
wGQk4GeruIkOgJOLdooN/HjhbHMCAwEAAaM6MDgwFAYDVR0RBA0wC4IJbG9jYWxo
b3N0MAsGA1UdDwQEAwIHgDATBgNVHSUEDDAKBggrBgEFBQcDATANBgkqhkiG9w0B
AQsFAAOCAQEASsBzHHYiWDDiBVWUEwVZAduTrslTLNOxG0QHBKsHWIlz/3QlhQil
ywb3OhfMTUR1dMGY5Iq5432QiCHO4IMCOv7tDIkgb4Bc3v/3CRlBlnurtAmUfNJ6
pTRSlK4AjWpGHAEEd/8aCaOE86hMP8WDht8MkJTRrQqpJ1HeDISoKt9nepHOIsj+
I2zLZZtw0pg7FuR4MzWuqOt071iRS46Pupryb3ZEGIWNz5iLrDQod5Iz2ZGSRGqE
rB8idX0mlj5AHRRanVR3PAes+eApsW9JvYG/ImuCOs+ZsukY614zQZdR+SyFm85G
4NICyeQsmiypNHHgw+xZmGqZg65bXNGoyg==
-----END CERTIFICATE-----
`  // RSA 2048 位证书

var rsa2048Key = `
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCu27qSfLZhzN7I
UP+ze99I86UbF3TYgcavyccfI/rwdwHMbj+W410GakumEeVub63y5XoLatRP5pmi
Nx7dEXyJc3r9OA7mvT1piECQ/NNZxD5J234Ai1w4fqWK9rH4PJlIdy4xra8ch/p5
m041oO4lkiqIZmfJ1A3F/eC5aORcYmzAM/DrAMDplLngh4z917MGWBxMvjjj9ytG
RIictH+U+mV5PTlmLjzeFUCicdaq3PSs2bUjx8KlXZBlImrHvbTjwOGjLrc7dG7Y
Fu4e1K7xugtXP0a9QcXYmKtREPssqZwJck6FUEjW6Ou888BkJOBnq7iJDoCTi3aK
Dfx44WxzAgMBAAECggEBAKYhib/H0ZhWB4yWuHqUxG4RXtrAjHlvw5Acy5zgmHiC
+Sh7ztrTJf0EXN9pvWwRm1ldgXj7hMBtPaaLbD1pccM9/qo66p17Sq/LjlyyeTOe
```  // RSA 2048 位私钥
# 定义一个名为 eccKey 的变量，存储了一个私钥
var eccKey = `
-----BEGIN PRIVATE KEY-----
affOHIbz4Sij2zCOdkR9fr0EztTQScF3yBhl4Aa/4cO8fcCeWxm86WEldq9x4xWJ
s5WMR4CnrOJhDINLNPQPKX92KyxEQ/RfuBWovx3M0nl3fcUWfESY134t5g/UBFId
In19tZ+pGIpCkxP0U1AZWrlZRA8Q/3sO2orUpoAOdCrGk/DcCTMh0c1pMzbYZ1/i
cYXn38MpUo8QeG4FElUhAv6kzeBIl2tRBMVzIigo+AECgYEA3No1rHdFu6Ox9vC8
E93PTZevYVcL5J5yx6x7khCaOLKKuRXpjOX/h3Ll+hlN2DVAg5Jli/JVGCco4GeK
kbFLSyxG1+E63JbgsVpaEOgvFT3bHHSPSRJDnIU+WkcNQ2u4Ky5ahZzbNdV+4fj2
NO2iMgkm7hoJANrm3IqqW8epenMCgYEAyq+qdNj5DiDzBcDvLwY+4/QmMOOgDqeh
/TzhbDRyr+m4xNT7LLS4s/3wcbkQC33zhMUI3YvOHnYq5Ze/iL/TSloj0QCp1I7L
J7sZeM1XimMBQIpCfOC7lf4tU76Fz0DTHAL+CmX1DgmRJdYO09843VsKkscC968R
4cwL5oGxxgECgYAM4TTsH/CTJtLEIfn19qOWVNhHhvoMlSkAeBCkzg8Qa2knrh12
uBsU3SCIW11s1H40rh758GICDJaXr7InGP3ZHnXrNRlnr+zeqvRBtCi6xma23B1X
F5eV0zd1sFsXqXqOGh/xVtp54z+JEinZoForLNl2XVJVGG8KQZP50kUR/QKBgH4O
8zzpFT0sUPlrHVdp0wODfZ06dPmoWJ9flfPuSsYN3tTMgcs0Owv3C+wu5UPAegxB
X1oq8W8Qn21cC8vJQmgj19LNTtLcXI3BV/5B+Aghu02gr+lq/EA1bYuAG0jjUGlD
kyx0bQzl9lhJ4b70PjGtxc2z6KyTPdPpTB143FABAoGAQDoIUdc77/IWcjzcaXeJ
8abak5rAZA7cu2g2NVfs+Km+njsB0pbTwMnV1zGoFABdaHLdqbthLWtX7WOb1PDD
MQ+kbiLw5uj8IY2HEqJhDGGEdXBqxbW7kyuIAN9Mw+mwKzkikNcFQdxgchWH1d1o
lVkr92iEX+IhIeYb4DN1vQw=
-----END PRIVATE KEY-----
`

# 定义一个名为 eccCert 的变量，存储了一个证书
var eccCert = `
-----BEGIN CERTIFICATE-----
MIICTDCCAfKgAwIBAgIQDtCrO8cNST2eY2tA/AGrsDAKBggqhkjOPQQDAjBeMQsw
CQYDVQQGEwJDTjEOMAwGA1UEChMFTXlTU0wxKzApBgNVBAsTIk15U1NMIFRlc3Qg
RUNDIC0gRm9yIHRlc3QgdXNlIG9ubHkxEjAQBgNVBAMTCU15U1NM...
// 定义一个包含 RSA 2048 证书的字符串
var rsa2048Cert = `
-----BEGIN CERTIFICATE-----
MIIC5DCCAcwCCQDq7JL8p4z3wTANBgkqhkiG9w0BAQsFADCBiDELMAkGA1UEBhMC
Q04xETAPBgNVBAgMCE1pY2hpZ2FuMQ8wDQYDVQQHDAZBbm4gQWlyYTEPMA0GA1UE
CgwGQW5uIEluYzEPMA0GA1UECwwGQW5uIEluYzEPMA0GA1UEAwwGQW5uIEluYzAe
Fw0yMDA4MjcxNzEwMzlaFw0zMDA4MjUxNzEwMzlaMIGIMQswCQYDVQQGEwJDTjER
MA8GA1UECAwITWljaGlnYW4xDzANBgNVBAcMBkFubiBBaXJhMQ8wDQYDVQQKDAZB
bm4gSW5jMQ8wDQYDVQQLDAZBbm4gSW5jMQ8wDQYDVQQDDAZBbm4gSW5jMFkwEwYH
KoZIzj0CAQYIKoZIzj0DAQcDQgAEr2Mv6+10dWN/ZQtiaUSYefNs0dgkzcp27qJS
vWtY7G12mb/oaQZR5IMzB7Fruy2PT52/w26eItZUQEjnXQ/6yw==`
// 定义一个包含 RSA 2048 私钥的字符串
var rsa2048Key = `
-----BEGIN EC PRIVATE KEY-----
MHcCAQEEIB8G2suYKuBLoodNIwRMp3JPN1fcZxCt3kcOYIx4nbcPoAoGCCqGSM49
AwEHoUQDQgAEr2Mv6+10dWN/ZQtiaUSYefNs0dgkzcp27qJSvWtY7G12mb/oaQZR
5IMzB7Fruy2PT52/w26eItZUQEjnXQ/6yw==`

// 定义一个测试函数 TestDefaultTLSRSA2048
func TestDefaultTLSRSA2048(t *testing.T) {
    // 写入 RSA 2048 证书文件
    os.WriteFile("server-rsa2048.crt", []byte(rsa2048Cert), 0o777)
    // 写入 RSA 2048 私钥文件
    os.WriteFile("server-rsa2048.key", []byte(rsa2048Key), 0o777)
    // 定义服务器配置
    serverCfg := &Config{
        TLS: TLSConfig{
            VerifyHostName: true,
            CertCheckRate:  1,
            KeyPath:        "server-rsa2048.key",
            CertPath:       "server-rsa2048.crt",
        },
    }
    // 定义客户端配置
    clientCfg := &Config{
        TLS: TLSConfig{
            Verify:      false,
            SNI:         "localhost",
            Fingerprint: "",
        },
    }
    // 将服务器配置添加到上下文中
    sctx := config.WithConfig(context.Background(), Name, serverCfg)
    // 将客户端配置添加到上下文中
    cctx := config.WithConfig(context.Background(), Name, clientCfg)
    // 选择一个端口
    port := common.PickPort("tcp", "127.0.0.1")
    // 定义传输配置
    transportConfig := &transport.Config{
        LocalHost:  "127.0.0.1",
        LocalPort:  port,
        RemoteHost: "127.0.0.1",
        RemotePort: port,
    }
    // 将传输配置添加到上下文中
    ctx := config.WithConfig(context.Background(), transport.Name, transportConfig)
    // 将 freedom 配置添加到上下文中
    ctx = config.WithConfig(ctx, freedom.Name, &freedom.Config{})
    // 创建传输客户端
    tcpClient, err := transport.NewClient(ctx, nil)
    common.Must(err)
    // 创建传输服务器
    tcpServer, err := transport.NewServer(ctx, nil)
    common.Must(err)
    common.Must(err)
    // 创建服务器
    s, err := NewServer(sctx, tcpServer)
    common.Must(err)
    // 创建客户端
    c, err := NewClient(cctx, tcpClient)
    common.Must(err)

    // 创建一个等待组
    wg := sync.WaitGroup{}
    wg.Add(1)
    var conn1, conn2 net.Conn
    // 启动一个协程，接受连接
    go func() {
        conn2, err = s.AcceptConn(nil)
        common.Must(err)
        wg.Done()
    }()
    // 客户端发起连接
    conn1, err = c.DialConn(nil, nil)
    common.Must(err)

    // 写入数据到连接
    common.Must2(conn1.Write([]byte("12345678\r\n")))
    // 等待协程结束
    wg.Wait()
    buf := [10]byte{}
}
    # 从 conn2 读取数据到 buf 中
    conn2.Read(buf[:])
    # 如果 conn1 和 conn2 的连接状态不一致
    if !util.CheckConn(conn1, conn2) {
        # 标记测试失败
        t.Fail()
    }
    # 关闭 conn1 的连接
    conn1.Close()
    # 关闭 conn2 的连接
    conn2.Close()
}
// 测试默认的 TLS ECC 配置
func TestDefaultTLSECC(t *testing.T) {
    // 写入 ECC 证书文件
    os.WriteFile("server-ecc.crt", []byte(eccCert), 0o777)
    // 写入 ECC 私钥文件
    os.WriteFile("server-ecc.key", []byte(eccKey), 0o777)
    // 服务器端配置
    serverCfg := &Config{
        TLS: TLSConfig{
            VerifyHostName: true,
            CertCheckRate:  1,
            KeyPath:        "server-ecc.key",
            CertPath:       "server-ecc.crt",
        },
    }
    // 客户端配置
    clientCfg := &Config{
        TLS: TLSConfig{
            Verify:      false,
            SNI:         "localhost",
            Fingerprint: "",
        },
    }
    // 服务器端上下文配置
    sctx := config.WithConfig(context.Background(), Name, serverCfg)
    // 客户端上下文配置
    cctx := config.WithConfig(context.Background(), Name, clientCfg)

    // 选择端口
    port := common.PickPort("tcp", "127.0.0.1")
    // 传输配置
    transportConfig := &transport.Config{
        LocalHost:  "127.0.0.1",
        LocalPort:  port,
        RemoteHost: "127.0.0.1",
        RemotePort: port,
    }
    // 传输上下文配置
    ctx := config.WithConfig(context.Background(), transport.Name, transportConfig)
    // 自由上下文配置
    ctx = config.WithConfig(ctx, freedom.Name, &freedom.Config{})
    // 新建 TCP 客户端
    tcpClient, err := transport.NewClient(ctx, nil)
    common.Must(err)
    // 新建 TCP 服务器
    tcpServer, err := transport.NewServer(ctx, nil)
    common.Must(err)
    common.Must(err)
    // 新建服务器
    s, err := NewServer(sctx, tcpServer)
    common.Must(err)
    // 新建客户端
    c, err := NewClient(cctx, tcpClient)
    common.Must(err)

    wg := sync.WaitGroup{}
    wg.Add(1)
    var conn1, conn2 net.Conn
    // 启动协程，接受连接
    go func() {
        conn2, err = s.AcceptConn(nil)
        common.Must(err)
        wg.Done()
    }()
    // 客户端连接
    conn1, err = c.DialConn(nil, nil)
    common.Must(err)

    // 写入数据
    common.Must2(conn1.Write([]byte("12345678\r\n")))
    wg.Wait()
    buf := [10]byte{}
    conn2.Read(buf[:])
    // 检查连接
    if !util.CheckConn(conn1, conn2) {
        t.Fail()
    }
    conn1.Close()
    conn2.Close()
}

// 测试 UTLS RSA2048 配置
func TestUTLSRSA2048(t *testing.T) {
    // 写入 RSA2048 证书文件
    os.WriteFile("server-rsa2048.crt", []byte(rsa2048Cert), 0o777)
    // 写入 RSA2048 私钥文件
    os.WriteFile("server-rsa2048.key", []byte(rsa2048Key), 0o777)
    # 定义指纹列表
    fingerprints := []string{
        "chrome",
        "firefox",
        "ios",
    }
    # 遍历指纹列表
    for _, s := range fingerprints {
        # 为服务器配置 TLS
        serverCfg := &Config{
            TLS: TLSConfig{
                CertCheckRate: 1,
                KeyPath:       "server-rsa2048.key",
                CertPath:      "server-rsa2048.crt",
            },
        }
        # 为客户端配置 TLS
        clientCfg := &Config{
            TLS: TLSConfig{
                Verify:      false,
                SNI:         "localhost",
                Fingerprint: s,
            },
        }
        # 为服务器上下文添加配置
        sctx := config.WithConfig(context.Background(), Name, serverCfg)
        # 为客户端上下文添加配置
        cctx := config.WithConfig(context.Background(), Name, clientCfg)

        # 选择端口
        port := common.PickPort("tcp", "127.0.0.1")
        # 配置传输参数
        transportConfig := &transport.Config{
            LocalHost:  "127.0.0.1",
            LocalPort:  port,
            RemoteHost: "127.0.0.1",
            RemotePort: port,
        }
        # 为上下文添加传输配置
        ctx := config.WithConfig(context.Background(), transport.Name, transportConfig)
        # 为上下文添加自由配置
        ctx = config.WithConfig(ctx, freedom.Name, &freedom.Config{})
        # 创建 TCP 客户端
        tcpClient, err := transport.NewClient(ctx, nil)
        common.Must(err)
        # 创建 TCP 服务器
        tcpServer, err := transport.NewServer(ctx, nil)
        common.Must(err)

        # 创建服务器
        s, err := NewServer(sctx, tcpServer)
        common.Must(err)
        # 创建客户端
        c, err := NewClient(cctx, tcpClient)
        common.Must(err)

        # 创建等待组
        wg := sync.WaitGroup{}
        wg.Add(1)
        var conn1, conn2 net.Conn
        # 启动协程，接受连接
        go func() {
            conn2, err = s.AcceptConn(nil)
            common.Must(err)
            wg.Done()
        }()
        # 客户端连接
        conn1, err = c.DialConn(nil, nil)
        common.Must(err)

        # 写入数据到连接1
        common.Must2(conn1.Write([]byte("12345678\r\n")))
        # 等待协程完成
        wg.Wait()
        buf := [10]byte{}
        conn2.Read(buf[:])
        # 检查连接是否正常
        if !util.CheckConn(conn1, conn2) {
            t.Fail()
        }
        # 关闭连接
        conn1.Close()
        conn2.Close()
        s.Close()
        c.Close()
    }
# 定义一个名为 TestUTLSECC 的单元测试函数
func TestUTLSECC(t *testing.T) {
    # 将 eccCert 的内容写入 server-ecc.crt 文件，权限设置为 777
    os.WriteFile("server-ecc.crt", []byte(eccCert), 0o777)
    # 将 eccKey 的内容写入 server-ecc.key 文件，权限设置为 777
    os.WriteFile("server-ecc.key", []byte(eccKey), 0o777)
    # 创建一个名为 fingerprints 的字符串数组
    fingerprints := []string{
        "chrome",
        "firefox",
        "ios",
    }
    # 遍历指纹列表
    for _, s := range fingerprints:
        # 创建服务器配置对象
        serverCfg := &Config{
            TLS: TLSConfig{
                CertCheckRate: 1,  # 设置证书检查频率
                KeyPath:       "server-ecc.key",  # 设置服务器私钥路径
                CertPath:      "server-ecc.crt",  # 设置服务器证书路径
            },
        }
        # 创建客户端配置对象
        clientCfg := &Config{
            TLS: TLSConfig{
                Verify:      false,  # 设置是否验证服务器证书
                SNI:         "localhost",  # 设置服务器名称指示
                Fingerprint: s,  # 设置指纹
            },
        }
        # 使用服务器配置对象创建服务器上下文
        sctx := config.WithConfig(context.Background(), Name, serverCfg)
        # 使用客户端配置对象创建客户端上下文
        cctx := config.WithConfig(context.Background(), Name, clientCfg)

        # 选择端口
        port := common.PickPort("tcp", "127.0.0.1")
        # 创建传输配置对象
        transportConfig := &transport.Config{
            LocalHost:  "127.0.0.1",  # 设置本地主机
            LocalPort:  port,  # 设置本地端口
            RemoteHost: "127.0.0.1",  # 设置远程主机
            RemotePort: port,  # 设置远程端口
        }
        # 使用传输配置对象创建上下文
        ctx := config.WithConfig(context.Background(), transport.Name, transportConfig)
        # 向上下文添加自由配置对象
        ctx = config.WithConfig(ctx, freedom.Name, &freedom.Config{})
        # 创建 TCP 客户端
        tcpClient, err := transport.NewClient(ctx, nil)
        common.Must(err)
        # 创建 TCP 服务器
        tcpServer, err := transport.NewServer(ctx, nil)
        common.Must(err)

        # 创建服务器
        s, err := NewServer(sctx, tcpServer)
        common.Must(err)
        # 创建客户端
        c, err := NewClient(cctx, tcpClient)
        common.Must(err)

        # 创建等待组
        wg := sync.WaitGroup{}
        wg.Add(1)
        var conn1, conn2 net.Conn
        # 启动协程，接受连接
        go func() {
            conn2, err = s.AcceptConn(nil)
            common.Must(err)
            wg.Done()
        }()
        # 客户端发起连接
        conn1, err = c.DialConn(nil, nil)
        common.Must(err)

        # 写入数据到连接1
        common.Must2(conn1.Write([]byte("12345678\r\n")))
        # 等待协程完成
        wg.Wait()
        buf := [10]byte{}
        conn2.Read(buf[:])
        # 检查连接是否正常
        if !util.CheckConn(conn1, conn2) {
            t.Fail()
        }
        # 关闭连接
        conn1.Close()
        conn2.Close()
        s.Close()
        c.Close()
    }
# 定义一个测试函数，用于测试 isDomainNameMatched 函数
func TestMatch(t *testing.T) {
    # 如果 *.google.com 匹配 www.google.com，则测试通过
    if !isDomainNameMatched("*.google.com", "www.google.com") {
        t.Fail()
    }

    # 如果 *.google.com 不匹配 google.com，则测试通过
    if isDomainNameMatched("*.google.com", "google.com") {
        t.Fail()
    }

    # 如果 localhost 匹配 localhost，则测试通过
    if !isDomainNameMatched("localhost", "localhost") {
        t.Fail()
    }
}
```