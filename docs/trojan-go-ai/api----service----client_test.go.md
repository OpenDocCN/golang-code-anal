# `trojan-go\api\service\client_test.go`

```go
package service

import (
    "context"  // 导入上下文包
    "fmt"  // 导入格式化包
    "testing"  // 导入测试包
    "time"  // 导入时间包

    "google.golang.org/grpc"  // 导入 gRPC 包

    "github.com/p4gefau1t/trojan-go/common"  // 导入 common 包
    "github.com/p4gefau1t/trojan-go/config"  // 导入 config 包
    "github.com/p4gefau1t/trojan-go/statistic/memory"  // 导入 memory 包
)

func TestClientAPI(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())  // 创建一个带有取消函数的上下文
    ctx = config.WithConfig(ctx, memory.Name,  // 将 memory 配置添加到上下文中
        &memory.Config{
            Passwords: []string{"useless"},  // 设置密码
        })
    port := common.PickPort("tcp", "127.0.0.1")  // 选择一个可用端口
    ctx = config.WithConfig(ctx, Name, &Config{  // 将配置添加到上下文中
        APIConfig{
            Enabled: true,  // 启用 API
            APIHost: "127.0.0.1",  // 设置 API 主机
            APIPort: port,  // 设置 API 端口
        },
    })
    auth, err := memory.NewAuthenticator(ctx)  // 创建一个新的认证对象
    common.Must(err)  // 检查错误
    go RunClientAPI(ctx, auth)  // 启动客户端 API

    time.Sleep(time.Second * 3)  // 休眠 3 秒
    common.Must(auth.AddUser("hash1234"))  // 添加用户
    valid, user := auth.AuthUser("hash1234")  // 验证用户
    if !valid {  // 如果用户无效
        t.Fail()  // 测试失败
    }
    user.AddTraffic(1234, 5678)  // 添加流量
    time.Sleep(time.Second)  // 休眠 1 秒
    conn, err := grpc.Dial(fmt.Sprintf("127.0.0.1:%d", port), grpc.WithInsecure())  // 建立 gRPC 连接
    common.Must(err)  // 检查错误
    client := NewTrojanClientServiceClient(conn)  // 创建 Trojan 客户端服务客户端
    resp, err := client.GetTraffic(ctx, &GetTrafficRequest{User: &User{  // 获取流量信息
        Hash: "hash1234",  // 用户哈希
    }})
    common.Must(err)  // 检查错误
    if resp.TrafficTotal.DownloadTraffic != 5678 || resp.TrafficTotal.UploadTraffic != 1234 {  // 如果下载流量不等于 5678 或上传流量不等于 1234
        t.Fail()  // 测试失败
    }
    _, err = client.GetTraffic(ctx, &GetTrafficRequest{})  // 获取流量信息
    if err == nil {  // 如果没有错误
        t.Fail()  // 测试失败
    }
    cancel()  // 取消上下文
}
```