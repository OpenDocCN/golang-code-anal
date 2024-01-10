# `trojan-go\api\service\server_test.go`

```
package service

import (
    "context"
    "crypto/tls"
    "crypto/x509"
    "fmt"
    "os"
    "testing"
    "time"

    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials"

    "github.com/p4gefau1t/trojan-go/common"
    "github.com/p4gefau1t/trojan-go/config"
    "github.com/p4gefau1t/trojan-go/statistic/memory"
)

func TestServerAPI(t *testing.T) {
    // 创建一个可以取消的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 在上下文中设置内存统计配置
    ctx = config.WithConfig(ctx, memory.Name,
        &memory.Config{
            Passwords: []string{},
        })
    // 选择一个可用的端口
    port := common.PickPort("tcp", "127.0.0.1")
    // 在上下文中设置 API 配置
    ctx = config.WithConfig(ctx, Name, &Config{
        APIConfig{
            Enabled: true,
            APIHost: "127.0.0.1",
            APIPort: port,
        },
    })
    // 创建内存认证器
    auth, err := memory.NewAuthenticator(ctx)
    common.Must(err)
    // 启动 API 服务器
    go RunServerAPI(ctx, auth)
    // 等待3秒
    time.Sleep(time.Second * 3)
    // 添加用户
    common.Must(auth.AddUser("hash1234"))
    // 验证用户
    _, user := auth.AuthUser("hash1234")
    // 连接到 gRPC 服务器
    conn, err := grpc.Dial(fmt.Sprintf("127.0.0.1:%d", port), grpc.WithInsecure())
    common.Must(err)
    // 创建 TrojanServerServiceClient
    server := NewTrojanServerServiceClient(conn)
    // 获取用户列表
    stream1, err := server.ListUsers(ctx, &ListUsersRequest{})
    common.Must(err)
    // 循环处理用户列表
    for {
        resp, err := stream1.Recv()
        if err != nil {
            break
        }
        fmt.Println(resp.Status.User.Hash)
        // 检查用户哈希值
        if resp.Status.User.Hash != "hash1234" {
            t.Fail()
        }
        fmt.Println(resp.Status.SpeedCurrent)
        fmt.Println(resp.Status.SpeedLimit)
    }
    // 关闭发送流
    stream1.CloseSend()
    // 增加用户流量
    user.AddTraffic(1234, 5678)
    // 等待1秒
    time.Sleep(time.Second * 1)
    // 获取用户信息
    stream2, err := server.GetUsers(ctx)
    common.Must(err)
    // 发送获取用户请求
    stream2.Send(&GetUsersRequest{
        User: &User{
            Hash: "hash1234",
        },
    })
    // 接收用户信息
    resp2, err := stream2.Recv()
    common.Must(err)
    // 检查用户流量
    if resp2.Status.TrafficTotal.DownloadTraffic != 1234 || resp2.Status.TrafficTotal.UploadTraffic != 5678 {
        t.Fatal("wrong traffic")
    }
}
    // 调用 server.SetUsers 方法，返回一个流和可能的错误
    stream3, err := server.SetUsers(ctx)
    // 检查错误，如果有错误则触发 panic
    common.Must(err)
    // 向流中发送 SetUsersRequest 消息
    stream3.Send(&SetUsersRequest{
        Status: &UserStatus{
            User: &User{
                Hash: "hash1234",
            },
        },
        Operation: SetUsersRequest_Delete,
    })
    // 从流中接收响应消息
    resp3, err := stream3.Recv()
    // 如果有错误或者响应中的 Success 字段为 false，则触发测试失败
    if err != nil || !resp3.Success {
        t.Fatal("user not exists")
    }
    // 调用 auth.AuthUser 方法验证用户，获取验证结果和用户信息
    valid, _ := auth.AuthUser("hash1234")
    // 如果验证结果为 true，则触发测试失败
    if valid {
        t.Fatal("failed to auth")
    }
    // 向流中发送 SetUsersRequest 消息
    stream3.Send(&SetUsersRequest{
        Status: &UserStatus{
            User: &User{
                Hash: "newhash",
            },
        },
        Operation: SetUsersRequest_Add,
    })
    // 从流中接收响应消息
    resp3, err = stream3.Recv()
    // 如果有错误或者响应中的 Success 字段为 false，则触发测试失败
    if err != nil || !resp3.Success {
        t.Fatal("failed to read")
    }
    // 调用 auth.AuthUser 方法验证用户，获取验证结果和用户信息
    valid, user = auth.AuthUser("newhash")
    // 如果验证结果为 false，则触发测试失败
    if !valid {
        t.Fatal("failed to auth 2")
    }
    // 向流中发送 SetUsersRequest 消息，包含用户状态的修改
    stream3.Send(&SetUsersRequest{
        Status: &UserStatus{
            User: &User{
                Hash: "newhash",
            },
            SpeedLimit: &Speed{
                DownloadSpeed: 5000,
                UploadSpeed:   3000,
            },
            TrafficTotal: &Traffic{
                DownloadTraffic: 1,
                UploadTraffic:   1,
            },
        },
        Operation: SetUsersRequest_Modify,
    })
    // 启动一个 goroutine，模拟用户流量的增加
    go func() {
        for {
            select {
            case <-ctx.Done():
                return
            default:
            }
            user.AddTraffic(200, 0)
        }
    }()
    // 启动一个 goroutine，模拟用户流量的增加
    go func() {
        for {
            select {
            case <-ctx.Done():
                return
            default:
            }
            user.AddTraffic(0, 300)
        }
    }()
    // 等待 3 秒
    time.Sleep(time.Second * 3)
    # 循环3次
    for i := 0; i < 3; i++ {
        # 向流中发送获取用户请求
        stream2.Send(&GetUsersRequest{
            User: &User{
                Hash: "newhash",
            },
        })
        # 接收响应并检查错误
        resp2, err = stream2.Recv()
        common.Must(err)
        # 打印当前速度
        fmt.Println(resp2.Status.SpeedCurrent)
        # 打印速度限制
        fmt.Println(resp2.Status.SpeedLimit)
        # 休眠1秒
        time.Sleep(time.Second)
    }
    # 关闭发送流
    stream2.CloseSend()
    # 取消操作
    cancel()
# 测试 TLS RSA 连接
func TestTLSRSA(t *testing.T):
    # 选择一个可用的端口
    port := common.PickPort("tcp", "127.0.0.1")
    # 配置 TLS RSA 连接
    cfg := &Config{
        API: APIConfig{
            Enabled: true,
            APIHost: "127.0.0.1",
            APIPort: port,
            SSL: SSLConfig{
                Enabled:        true,
                CertPath:       "server-rsa2048.crt",
                KeyPath:        "server-rsa2048.key",
                VerifyClient:   false,
                ClientCertPath: []string{"client-rsa2048.crt"},
            },
        },
    }
    # 将配置信息添加到上下文中
    ctx := config.WithConfig(context.Background(), Name, cfg)
    ctx = config.WithConfig(ctx, memory.Name,
        &memory.Config{
            Passwords: []string{},
        })
    # 创建认证对象
    auth, err := memory.NewAuthenticator(ctx)
    common.Must(err)
    # 启动服务器 API
    go func():
        common.Must(RunServerAPI(ctx, auth))
    }()
    # 等待一秒钟
    time.Sleep(time.Second)
    # 创建证书池
    pool := x509.NewCertPool()
    # 读取服务器证书
    certBytes, err := os.ReadFile("server-rsa2048.crt")
    common.Must(err)
    # 将服务器证书添加到证书池
    pool.AppendCertsFromPEM(certBytes)
    # 加载客户端证书和私钥
    certificate, err := tls.LoadX509KeyPair("client-rsa2048.crt", "client-rsa2048.key")
    common.Must(err)
    # 创建 TLS 凭证
    creds := credentials.NewTLS(&tls.Config{
        ServerName:   "localhost",
        RootCAs:      pool,
        Certificates: []tls.Certificate{certificate},
    })
    # 建立带有 TLS 传输凭据的 gRPC 连接
    conn, err := grpc.Dial(fmt.Sprintf("127.0.0.1:%d", port), grpc.WithTransportCredentials(creds))
    common.Must(err)
    # 创建 TrojanServerServiceClient
    server := NewTrojanServerServiceClient(conn)
    # 调用 ListUsers 方法
    stream, err := server.ListUsers(ctx, &ListUsersRequest{})
    common.Must(err)
    # 关闭流
    stream.CloseSend()
    # 关闭连接
    conn.Close()

# 测试 TLS ECC 连接
func TestTLSECC(t *testing.T):
    # 选择一个可用的端口
    port := common.PickPort("tcp", "127.0.0.1")
    // 创建一个 Config 结构体的指针，并初始化其中的 APIConfig 和 SSLConfig 字段
    cfg := &Config{
        API: APIConfig{
            Enabled: true,
            APIHost: "127.0.0.1",
            APIPort: port,
            SSL: SSLConfig{
                Enabled:        true,
                CertPath:       "server-ecc.crt",
                KeyPath:        "server-ecc.key",
                VerifyClient:   false,
                ClientCertPath: []string{"client-ecc.crt"},
            },
        },
    }

    // 使用配置信息创建一个带有配置信息的上下文
    ctx := config.WithConfig(context.Background(), Name, cfg)
    // 在上下文中添加内存配置信息
    ctx = config.WithConfig(ctx, memory.Name,
        &memory.Config{
            Passwords: []string{},
        })

    // 使用上下文创建一个内存认证器
    auth, err := memory.NewAuthenticator(ctx)
    common.Must(err)
    // 启动一个 goroutine，运行服务器 API
    go func() {
        common.Must(RunServerAPI(ctx, auth))
    }()
    // 等待一秒钟
    time.Sleep(time.Second)
    // 创建一个新的 x509 证书池
    pool := x509.NewCertPool()
    // 读取服务器证书的内容，并添加到证书池中
    certBytes, err := os.ReadFile("server-ecc.crt")
    common.Must(err)
    pool.AppendCertsFromPEM(certBytes)

    // 加载客户端证书和私钥
    certificate, err := tls.LoadX509KeyPair("client-ecc.crt", "client-ecc.key")
    common.Must(err)
    // 创建一个包含 TLS 配置的凭证
    creds := credentials.NewTLS(&tls.Config{
        ServerName:   "localhost",
        RootCAs:      pool,
        Certificates: []tls.Certificate{certificate},
    })
    // 使用凭证建立与 gRPC 服务器的安全连接
    conn, err := grpc.Dial(fmt.Sprintf("127.0.0.1:%d", port), grpc.WithTransportCredentials(creds))
    common.Must(err)
    // 创建一个 TrojanServerServiceClient 对象
    server := NewTrojanServerServiceClient(conn)
    // 调用服务器的 ListUsers 方法，获取用户列表的流
    stream, err := server.ListUsers(ctx, &ListUsersRequest{})
    common.Must(err)
    // 关闭流
    stream.CloseSend()
    // 关闭连接
    conn.Close()
# 服务器的 RSA 2048 位证书
var serverRSA2048Cert = `
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
`

# 服务器的 RSA 2048 位私钥
var serverRSA2048Key = `
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCu27qSfLZhzN7I
UP+ze99I86UbF3TYgcavyccfI/rwdwHMbj+W410GakumEeVub63y5XoLatRP5pmi
Nx7dEXyJc3r9OA7mvT1piECQ/NNZxD5J234Ai1w4fqWK9rH4PJlIdy4xra8ch/p5
m041oO4lkiqIZmfJ1A3F/eC5aORcYmzAM/DrAMDplLngh4z917MGWBxMvjjj9ytG
RIictH+U+mV5PTlmLjzeFUCicdaq3PSs2bUjx8KlXZBlImrHvbTjwOGjLrc7dG7Y
Fu4e1K7xugtXP0a9QcXYmKtREPssqZwJck6FUEjW6Ou888BkJOBnq7iJDoCTi3aK
Dfx44WxzAgMBAAECggEBAKYhib/H0ZhWB4yWuHqUxG4RXtrAjHlvw5Acy5zgmHiC
+Sh7ztrTJf0EXN9pvWwRm1ldgXj7hMBtPaaLbD1pccM9/qo66p17Sq/LjlyyeTOe
affOHIbz4Sij2zCOdkR9fr0EztTQScF3yBhl4Aa/4cO8fcCeWxm86WEldq9x4xWJ
s5WMR4CnrOJhDINLNPQPKX92KyxEQ/RfuBWovx3M0nl3fcUWfESY134t5g/UBFId
In19tZ+pGIpCkxP0U1AZWrlZRA8Q/3sO2orUpoAOdCrGk/DcCTMh0c1pMzbYZ1/i
cYXn38MpUo8QeG4FElUhAv6kzeBIl2tRBMVzIigo+AECgYEA3No1rHdFu6Ox9vC8
E93PTZevYVcL5J5yx6x7khCaOLKKuRXpjOX/h3Ll+hlN2DVAg5Jli/JVGCco4GeK
// 客户端RSA私钥
var clientRSA2048Key = `
-----BEGIN PRIVATE KEY-----
# 定义一个包含私钥的字符串变量
var serverECCKey = `
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCXg7ut9JaLrn9l
eLMN/enBogJ8Zk7HlUTBvptz52fN9abgLAKZtyFkhHDqiFDJOrNaYEi8rIOhsUWY
V+C/5YxK4zDuPsvK4mY6qkdMHdBp4bjcbl2klLBJinQ2W3CAql5GXmUZ+I8P+y9M
YHQHTNsNy10emqugsL3hWN/yZ3BGRUBx+Zp6K2XaW4Xkeq4byC+gACNGBGPbs7kA
Uq8zppd3+w/seV5ZdeB6DDdJAsrFrfI0j/wJ+cj/KS+UTqZ+ZDQXPAcfJcpLldbk
vKIJ3EjOyc5qT9UngFYjqvAql5SbVG1KlT3JDJnXzDLWyKiDu0ef/Ipq3YSzItuX
wDp35g83AgMBAAECggEASOv4CjMrubKUUgwTcWqBdNY6iBDdXaVz4COSweffx/qx
BDdqUP0Yrz4m8loFN7Ru2dJ5b4VAHTQqoLW6z+D08p4B0MicYNsyBI4rnnDC/BLN
XBoqK6n8Zoiigf7kWKimkwufcS51/GUSUJojfdf5ndwAx1f9vmsSGEEkF5C9MrQo
Sa4eyySuXuDS9swrXuTn9FcVcbUoIblgL8GIlmX4S1Xl9NaVS/VAVt42FgpNSYy7
2Qf9Medg3ApimjwkLiDSh0RElirlUwzSg9dx2U+hHwWQWimb2AhA85uK3bpFB/x9
b2agS1uxTar4mk4LFppQqVUuXlpj2hW7HxTdcxGHYQKBgQDGA+Wv3b8FMeszYqqx
BbI5+FeXmQ5AoYNdHyPfCH8f2LX1FTnnQbUvFMJ3UQZl/GGHocgyFNvfDo6YyYsg
2XgcNO/JWMbKEw0HkfMgkaIa3Jfq/PTB386NhqBq5FHiBUrvSHzhaIzpuaokBrRk
jFlcqONK+uj77iRgci4wR59POQKBgQDD4fDZzOGQCy/AWrMX/adk8ymoxhWQj6PN
zhy2GZo9jGwyskQr5neIDQtxRbgokMspxpyPdQG2SxbG/zygyFEaCsp/Xb3iB3aA
2dcktkV9agx+g1WrflTpGG9quW5vQJgQv9FtlVXJdiihN9CQvXAcYRjUA/zkadIL
GsrVF9rh7wKBgFoMLaiDU7neEJKGnQ7xgzI/kD29ebDEgkOXxK1JZN4ro9t3MqTK
ycVGUIUIELvSQNv4I107BR3ztb8fcCiZHLjfDehnecctULCPm5vE/o3uoRtYu0lr
KLhNb6gMenwpYgFc2oV7ERG8v/WwItrSxFSR7QMNBWSD0IEXi4+jEnxpAoGAMJTS
5VG5B76egzh7fpG8eH8Ob/tg0c+uMpbR7CABbw5qr1AjNDgeoTGLCvbdq8HtgVju
7213lTyeU5Bt+vpzkt/mRRx8wZhUPbTJdSN3rJkmrCHql3Pnn0AeMfv3dcQxcsYA
LQuCkUqq3QE4yw0QxxkVzU+H4yaTn4lvkNYvxSUCgYEAgD85MM3DaiNcrevQv6Wb
vSo7jBhd8Q/NawH53V32eIlF9Kkm2mqTmPIQ2OxOtdZ3xm+JmPd20jEVg7NcPL58
t6BoSH10pLaI0CP2NdcYfJTIiHAhuQe7PxPCA0nlHTXgSv97QR7y4qMhbf2j1umc
hKym0tdk2QjR3qqglaD572A=
-----END PRIVATE KEY-----
`

# 定义一个包含证书的字符串变量
var serverECCCert = `
-----BEGIN CERTIFICATE-----
MIICTDCCAfKgAwIBAgIQDtCrO8cNST2eY2tA/AGrsDAKBggqhkjOPQQDAjBeMQsw
CQYDVQQGEwJDTjEOMAwGA1UEChMFTXlTU0wxKzApBgNVBAsTIk15U1NMIFRlc3Qg
RUNDIC0gRm9yIHRlc3QgdXNlIG9ubHkxEjAQBgNVBAMTCU15U1NMLmNvbTAeFw0y
MTA5MTQwNjQ1MzNaFw0yNjA5MTMwNjQ1MzNaMCExCzAJBgNVBAYTAkNOMRIwEAYD
# 定义变量 serverECCCert，存储服务器的 ECC 证书
var serverECCCert = `
-----BEGIN CERTIFICATE-----
MIICTDCCAfKgAwIBAgIQb5FLuCggTiWFtt/dPh+2bTAKBggqhkjOPQQDAjBeMQsw
CQYDVQQGEwJDTjEOMAwGA1UEChMFTXlTU0wxKzApBgNVBAsTIk15U1NMIFRlc3Qg
RUNDIC0gRm9yIHRlc3QgdXNlIG9ubHkxEjAQBgNVBAMTCU15U1NMLmNvbTAeFw0y
MTA5MTQwNjQ2MTFaFw0yNjA5MTMwNjQ2MTFaMCExCzAJBgNVBAYTAkNOMRIwEAYD
VQQDEwlsb2NhbGhvc3QwWTATBgcqhkjOPQIBBggqhkjOPQMBBwNCAAQME+DnYtcK
lbcZmc33oEtoeRWH61DYdl4ei/bM+vkv01MkBB+YTZl0yofJIFJYsfU5pMFK+uyw
D4qdcklKPGIKo4HOMIHLMA4GA1UdDwEB/wQEAwIFoDAdBgNVHSUEFjAUBggrBgEF
BQcDAQYIKwYBBQUHAwIwHwYDVR0jBBgwFoAUWxGyVxD0fBhTy3tH4eKznRFXFCYw
YwYIKwYBBQUHAQEEVzBVMCEGCCsGAQUFBzABhhVodHRwOi8vb2NzcC5teXNzbC5j
b20wMAYIKwYBBQUHMAKGJGh0dHA6Ly9jYS5teXNzbC5jb20vbXlzc2x0ZXN0ZWNj
LmNydDAUBgNVHREEDTALgglsb2NhbGhvc3QwCgYIKoZIzj0EAwIDSAAwRQIgfjaU
sCfgklPsjHzs3fUtSRGfWoRLFsRBO66RtHJSzrYCIQDAxgx0FB0mAXbflj4mXJVA
9/SjaCI40D6MhMnJhQS7Zg==
-----END CERTIFICATE-----
`

# 定义变量 serverECCKey，存储服务器的 ECC 私钥
var serverECCKey = `
-----BEGIN EC PRIVATE KEY-----
MHcCAQEEIB8G2suYKuBLoodNIwRMp3JPN1fcZxCt3kcOYIx4nbcPoAoGCCqGSM49
AwEHoUQDQgAEr2Mv6+10dWN/ZQtiaUSYefNs0dgkzcp27qJSvWtY7G12mb/oaQZR
5IMzB7Fruy2PT52/w26eItZUQEjnXQ/6yw==
-----END EC PRIVATE KEY-----
`

# 定义变量 clientECCCert，存储客户端的 ECC 证书
var clientECCCert = `
-----BEGIN CERTIFICATE-----
MIICTDCCAfKgAwIBAgIQb5FLuCggTiWFtt/dPh+2bTAKBggqhkjOPQQDAjBeMQsw
CQYDVQQGEwJDTjEOMAwGA1UEChMFTXlTU0wxKzApBgNVBAsTIk15U1NMIFRlc3Qg
RUNDIC0gRm9yIHRlc3QgdXNlIG9ubHkxEjAQBgNVBAMTCU15U1NMLmNvbTAeFw0y
MTA5MTQwNjQ2MTFaFw0yNjA5MTMwNjQ2MTFaMCExCzAJBgNVBAYTAkNOMRIwEAYD
VQQDEwlsb2NhbGhvc3QwWTATBgcqhkjOPQIBBggqhkjOPQMBBwNCAAQME+DnYtcK
lbcZmc33oEtoeRWH61DYdl4ei/bM+vkv01MkBB+YTZl0yofJIFJYsfU5pMFK+uyw
D4qdcklKPGIKo4HOMIHLMA4GA1UdDwEB/wQEAwIFoDAdBgNVHSUEFjAUBggrBgEF
BQcDAQYIKwYBBQUHAwIwHwYDVR0jBBgwFoAUWxGyVxD0fBhTy3tH4eKznRFXFCYw
YwYIKwYBBQUHAQEEVzBVMCEGCCsGAQUFBzABhhVodHRwOi8vb2NzcC5teXNzbC5j
b20wMAYIKwYBBQUHMAKGJGh0dHA6Ly9jYS5teXNzbC5jb20vbXlzc2x0ZXN0ZWNj
LmNydDAUBgNVHREEDTALgglsb2NhbGhvc3QwCgYIKoZIzj0EAwIDSAAwRQIgfjaU
sCfgklPsjHzs3fUtSRGfWoRLFsRBO66RtHJSzrYCIQDAxgx0FB0mAXbflj4mXJVA
9/SjaCI40D6MhMnJhQS7Zg==
-----END CERTIFICATE-----
`

# 定义变量 clientECCKey，存储客户端的 ECC 私钥
var clientECCKey = `
-----BEGIN EC PRIVATE KEY-----
MHcCAQEEINgbap6WOGXm8V+ghIyporGcGWnggbjjP9xNsxhn+0sqoAoGCCqGSM49
AwEHoUQDQgAEDBPg52LXCpW3GZnN96BLaHkVh+tQ2HZeHov2zPr5L9NTJAQfmE2Z
dMqHySBSWLH1OaTBSvrssA+KnXJJSjxiCg==
-----END EC PRIVATE KEY-----
`

# 初始化函数
func init() {
    # 将 serverRSA2048Cert 的内容写入 server-rsa2048.crt 文件中，权限设置为 777
    os.WriteFile("server-rsa2048.crt", []byte(serverRSA2048Cert), 0o777)
    # 将 serverRSA2048Key 的内容写入 server-rsa2048.key 文件中，权限设置为 777
    os.WriteFile("server-rsa2048.key", []byte(serverRSA2048Key), 0o777)
    # 将 clientRSA2048Cert 的内容写入 client-rsa2048.crt 文件中，权限设置为 777
    os.WriteFile("client-rsa2048.crt", []byte(clientRSA2048Cert), 0o777)
    # 将 clientRSA2048Key 的内容写入 client-rsa2048.key 文件中，权限设置为 777
    os.WriteFile("client-rsa2048.key", []byte(clientRSA2048Key), 0o777)
    
    # 将 serverECCCert 的内容写入 server-ecc.crt 文件中，权限设置为 777
    os.WriteFile("server-ecc.crt", []byte(serverECCCert), 0o777)
    # 将 serverECCKey 的内容写入 server-ecc.key 文件中，权限设置为 777
    os.WriteFile("server-ecc.key", []byte(serverECCKey), 0o777)
    # 将 clientECCCert 的内容写入 client-ecc.crt 文件中，权限设置为 777
    os.WriteFile("client-ecc.crt", []byte(clientECCCert), 0o777)
    # 将 clientECCKey 的内容写入 client-ecc.key 文件中，权限设置为 777
    os.WriteFile("client-ecc.key", []byte(clientECCKey), 0o777)
# 闭合前面的函数定义
```