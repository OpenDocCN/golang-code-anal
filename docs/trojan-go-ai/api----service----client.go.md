# `trojan-go\api\service\client.go`

```go
package service

import (
    "context"
    "net"

    "github.com/p4gefau1t/trojan-go/api"
    "github.com/p4gefau1t/trojan-go/common"
    "github.com/p4gefau1t/trojan-go/config"
    "github.com/p4gefau1t/trojan-go/log"
    "github.com/p4gefau1t/trojan-go/statistic"
    "github.com/p4gefau1t/trojan-go/tunnel/trojan"
)

type ClientAPI struct {
    TrojanClientServiceServer

    auth          statistic.Authenticator  // 统计认证信息
    ctx           context.Context          // 上下文
    uploadSpeed   uint64                   // 上传速度
    downloadSpeed uint64                   // 下载速度
    lastSent      uint64                   // 上次发送
    lastRecv      uint64                   // 上次接收
}

func (s *ClientAPI) GetTraffic(ctx context.Context, req *GetTrafficRequest) (*GetTrafficResponse, error) {
    log.Debug("API: GetTraffic")  // 调试日志
    if req.User == nil {
        return nil, common.NewError("User is unspecified")  // 返回用户未指定错误
    }
    if req.User.Hash == "" {
        req.User.Hash = common.SHA224String(req.User.Password)  // 使用密码生成哈希值
    }
    valid, user := s.auth.AuthUser(req.User.Hash)  // 验证用户
    if !valid {
        return nil, common.NewError("User " + req.User.Hash + " not found")  // 返回用户未找到错误
    }
    sent, recv := user.GetTraffic()  // 获取用户的流量
    sentSpeed, recvSpeed := user.GetSpeed()  // 获取用户的速度
    resp := &GetTrafficResponse{
        Success: true,
        TrafficTotal: &Traffic{
            UploadTraffic:   sent,  // 上传流量
            DownloadTraffic: recv,  // 下载流量
        },
        SpeedCurrent: &Speed{
            UploadSpeed:   sentSpeed,  // 上传速度
            DownloadSpeed: recvSpeed,  // 下载速度
        },
    }
    return resp, nil  // 返回结果和空错误
}

func RunClientAPI(ctx context.Context, auth statistic.Authenticator) error {
    cfg := config.FromContext(ctx, Name).(*Config)  // 从上下文中获取配置
    if !cfg.API.Enabled {
        return nil  // 如果 API 未启用，则返回空错误
    }
    server, err := newAPIServer(cfg)  // 创建 API 服务器
    if err != nil {
        return err  // 如果创建失败，则返回错误
    }
    defer server.Stop()  // 延迟关闭服务器
    service := &ClientAPI{
        ctx:  ctx,
        auth: auth,
    }
    RegisterTrojanClientServiceServer(server, service)  // 注册 Trojan 客户端服务
    addr, err := net.ResolveIPAddr("ip", cfg.API.APIHost)  // 解析 API 主机地址
    if err != nil {
        return common.NewError("api found invalid addr").Base(err)  // 返回无效地址错误
    }
}
    // 使用 net.Listen() 方法创建一个监听器，监听指定的 TCP 地址
    listener, err := net.Listen("tcp", (&net.TCPAddr{
        IP:   addr.IP,
        Port: cfg.API.APIPort,
        Zone: addr.Zone,
    }).String())
    // 如果创建监听器时发生错误，则返回一个包含错误信息的新错误
    if err != nil {
        return common.NewError("client api failed to listen").Base(err)
    }
    // 延迟关闭监听器
    defer listener.Close()
    // 打印日志，提示客户端 API 服务正在监听的地址
    log.Info("client-side api service is listening on", listener.Addr().String())
    // 创建一个用于接收错误信息的通道
    errChan := make(chan error, 1)
    // 启动一个 goroutine，用于处理服务端的请求
    go func() {
        errChan <- server.Serve(listener)
    }()
    // 使用 select 语句监听多个通道的操作，哪个通道先准备好就执行哪个操作
    select {
    // 如果 errChan 通道有数据，则返回该错误
    case err := <-errChan:
        return err
    // 如果 ctx.Done() 通道有数据，则返回 nil，表示正常关闭
    case <-ctx.Done():
        log.Debug("closed")
        return nil
    }
# 初始化函数，用于注册 Trojan 客户端的 API 处理程序
func init() {
    # 注册 Trojan 客户端的 API 处理程序，使用 Trojan 名称加上"_CLIENT"作为标识
    api.RegisterHandler(trojan.Name+"_CLIENT", RunClientAPI)
}
```