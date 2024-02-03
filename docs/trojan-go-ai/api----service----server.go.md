# `trojan-go\api\service\server.go`

```go
package service

import (
    "context"
    "crypto/tls"
    "crypto/x509"
    "io"
    "io/ioutil"
    "net"

    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials"

    "github.com/p4gefau1t/trojan-go/api"
    "github.com/p4gefau1t/trojan-go/common"
    "github.com/p4gefau1t/trojan-go/config"
    "github.com/p4gefau1t/trojan-go/log"
    "github.com/p4gefau1t/trojan-go/statistic"
    "github.com/p4gefau1t/trojan-go/tunnel/trojan"
)

type ServerAPI struct {
    TrojanServerServiceServer
    auth statistic.Authenticator
}

func (s *ServerAPI) GetUsers(stream TrojanServerService_GetUsersServer) error {
    log.Debug("API: GetUsers")
    // 在日志中记录 API 调用信息
    // 无限循环，接收客户端流的请求
    for {
        // 接收客户端流的请求
        req, err := stream.Recv()
        // 如果到达流的末尾，返回空值
        if err == io.EOF {
            return nil
        }
        // 如果出现错误，返回错误信息
        if err != nil {
            return err
        }
        // 如果请求中的用户信息为空，返回用户未指定错误
        if req.User == nil {
            return common.NewError("user is unspecified")
        }
        // 如果请求中的用户哈希值为空，使用密码生成哈希值
        if req.User.Hash == "" {
            req.User.Hash = common.SHA224String(req.User.Password)
        }
        // 验证用户哈希值，获取用户信息
        valid, user := s.auth.AuthUser(req.User.Hash)
        // 如果用户验证不通过，发送无效用户信息并继续下一次循环
        if !valid {
            stream.Send(&GetUsersResponse{
                Success: false,
                Info:    "invalid user: " + req.User.Hash,
            })
            continue
        }
        // 获取用户的下载流量和上传流量
        downloadTraffic, uploadTraffic := user.GetTraffic()
        // 获取用户的下载速度和上传速度
        downloadSpeed, uploadSpeed := user.GetSpeed()
        // 获取用户的下载速度限制和上传速度限制
        downloadSpeedLimit, uploadSpeedLimit := user.GetSpeedLimit()
        // 获取用户的 IP 限制
        ipLimit := user.GetIPLimit()
        // 获取用户当前的 IP 地址
        ipCurrent := user.GetIP()
        // 发送用户信息响应，包括用户状态、流量、速度、IP 信息
        err = stream.Send(&GetUsersResponse{
            Success: true,
            Status: &UserStatus{
                User: req.User,
                TrafficTotal: &Traffic{
                    UploadTraffic:   uploadTraffic,
                    DownloadTraffic: downloadTraffic,
                },
                SpeedCurrent: &Speed{
                    DownloadSpeed: downloadSpeed,
                    UploadSpeed:   uploadSpeed,
                },
                SpeedLimit: &Speed{
                    DownloadSpeed: uint64(downloadSpeedLimit),
                    UploadSpeed:   uint64(uploadSpeedLimit),
                },
                IpCurrent: int32(ipCurrent),
                IpLimit:   int32(ipLimit),
            },
        })
        // 如果发送响应时出现错误，返回错误信息
        if err != nil {
            return err
        }
    }
}



func (s *ServerAPI) SetUsers(stream TrojanServerService_SetUsersServer) error {
    // 记录调试信息
    log.Debug("API: SetUsers")
    // 返回空错误
    return nil
}



}

func (s *ServerAPI) ListUsers(req *ListUsersRequest, stream TrojanServerService_ListUsersServer) error {
    // 记录调试信息
    log.Debug("API: ListUsers")
    // 获取所有用户信息
    users := s.auth.ListUsers()
    // 遍历所有用户
    for _, user := range users {
        // 获取用户的流量信息
        downloadTraffic, uploadTraffic := user.GetTraffic()
        // 获取用户的速度信息
        downloadSpeed, uploadSpeed := user.GetSpeed()
        // 获取用户的速度限制信息
        downloadSpeedLimit, uploadSpeedLimit := user.GetSpeedLimit()
        // 获取用户的 IP 限制信息
        ipLimit := user.GetIPLimit()
        // 获取用户当前的 IP 信息
        ipCurrent := user.GetIP()
        // 发送用户信息到流
        err := stream.Send(&ListUsersResponse{
            Status: &UserStatus{
                User: &User{
                    Hash: user.Hash(),
                },
                TrafficTotal: &Traffic{
                    DownloadTraffic: downloadTraffic,
                    UploadTraffic:   uploadTraffic,
                },
                SpeedCurrent: &Speed{
                    DownloadSpeed: downloadSpeed,
                    UploadSpeed:   uploadSpeed,
                },
                SpeedLimit: &Speed{
                    DownloadSpeed: uint64(downloadSpeedLimit),
                    UploadSpeed:   uint64(uploadSpeedLimit),
                },
                IpLimit:   int32(ipLimit),
                IpCurrent: int32(ipCurrent),
            },
        })
        // 如果发送出错，则返回错误
        if err != nil {
            return err
        }
    }
    // 返回空错误
    return nil
}



func newAPIServer(cfg *Config) (*grpc.Server, error) {
    var server *grpc.Server
    # 如果 API 的 SSL 功能被启用
    if cfg.API.SSL.Enabled {
        # 记录日志，表示 API 的 TLS 功能已启用
        log.Info("api tls enabled")
        # 加载证书和私钥，生成 keyPair 对象
        keyPair, err := tls.LoadX509KeyPair(cfg.API.SSL.CertPath, cfg.API.SSL.KeyPath)
        # 如果加载失败，返回错误
        if err != nil {
            return nil, common.NewError("failed to load key pair").Base(err)
        }
        # 创建 TLS 配置对象，包含加载的证书和私钥
        tlsConfig := &tls.Config{
            Certificates: []tls.Certificate{keyPair},
        }
        # 如果需要验证客户端证书
        if cfg.API.SSL.VerifyClient {
            # 设置客户端认证方式为需要验证客户端证书
            tlsConfig.ClientAuth = tls.RequireAndVerifyClientCert
            # 创建用于存储客户端证书的证书池
            tlsConfig.ClientCAs = x509.NewCertPool()
            # 遍历客户端证书路径列表
            for _, path := range cfg.API.SSL.ClientCertPath {
                # 记录日志，表示正在加载客户端证书
                log.Debug("loading client cert: " + path)
                # 读取客户端证书文件内容
                certBytes, err := ioutil.ReadFile(path)
                # 如果读取失败，返回错误
                if err != nil {
                    return nil, common.NewError("failed to load cert file").Base(err)
                }
                # 将客户端证书内容添加到证书池中
                ok := tlsConfig.ClientCAs.AppendCertsFromPEM(certBytes)
                # 如果添加失败，返回错误
                if !ok {
                    return nil, common.NewError("invalid client cert")
                }
            }
        }
        # 创建基于 TLS 配置的凭证对象
        creds := credentials.NewTLS(tlsConfig)
        # 创建一个带有 TLS 凭证的 gRPC 服务器
        server = grpc.NewServer(grpc.Creds(creds))
    } else {
        # 如果 API 的 SSL 功能未启用，创建一个普通的 gRPC 服务器
        server = grpc.NewServer()
    }
    # 返回创建的 gRPC 服务器对象和空错误
    return server, nil
}


func RunServerAPI(ctx context.Context, auth statistic.Authenticator) error {
    // 从上下文中获取配置信息
    cfg := config.FromContext(ctx, Name).(*Config)
    // 如果 API 未启用，则返回空
    if !cfg.API.Enabled {
        return nil
    }
    // 创建 ServerAPI 实例
    service := &ServerAPI{
        auth: auth,
    }
    // 创建新的 API 服务器
    server, err := newAPIServer(cfg)
    if err != nil {
        return err
    }
    // 延迟关闭服务器
    defer server.Stop()
    // 注册 Trojan 服务器服务
    RegisterTrojanServerServiceServer(server, service)
    // 解析 API 主机地址
    addr, err := net.ResolveIPAddr("ip", cfg.API.APIHost)
    if err != nil {
        return common.NewError("api found invalid addr").Base(err)
    }
    // 监听 TCP 连接
    listener, err := net.Listen("tcp", (&net.TCPAddr{
        IP:   addr.IP,
        Port: cfg.API.APIPort,
        Zone: addr.Zone,
    }).String())
    if err != nil {
        return common.NewError("server api failed to listen").Base(err)
    }
    // 延迟关闭监听器
    defer listener.Close()
    // 打印日志，显示服务器端 API 服务正在监听的地址
    log.Info("server-side api service is listening on", listener.Addr().String())
    // 创建错误通道
    errChan := make(chan error, 1)
    // 启动服务器
    go func() {
        errChan <- server.Serve(listener)
    }()
    // 选择处理通道中的错误
    select {
    case err := <-errChan:
        return err
    case <-ctx.Done():
        log.Debug("closed")
        return nil
    }
}

func init() {
    // 注册 Trojan 服务器处理程序
    api.RegisterHandler(trojan.Name+"_SERVER", RunServerAPI)
}
```