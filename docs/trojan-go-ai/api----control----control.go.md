# `trojan-go\api\control\control.go`

```go
package control

import (
    "context"  // 上下文包，用于控制请求的取消、超时等
    "encoding/json"  // JSON 编解码包
    "flag"  // 命令行参数解析包
    "fmt"  // 格式化包，用于打印输出
    "io"  // I/O 操作包

    "google.golang.org/grpc"  // gRPC 客户端包

    "github.com/p4gefau1t/trojan-go/api/service"  // 引入自定义的服务包
    "github.com/p4gefau1t/trojan-go/common"  // 引入自定义的通用包
    "github.com/p4gefau1t/trojan-go/log"  // 引入自定义的日志包
    "github.com/p4gefau1t/trojan-go/option"  // 引入自定义的选项包
)

type apiController struct {
    address *string  // 服务器地址指针
    key     *string  // 密钥指针
    hash    *string  // 哈希指针
    cert    *string  // 证书指针

    cmd                *string  // 命令指针
    password           *string  // 密码指针
    add                *bool    // 添加标志指针
    delete             *bool    // 删除标志指针
    modify             *bool    // 修改标志指针
    list               *bool    // 列表标志指针
    uploadSpeedLimit   *int     // 上传速度限制指针
    downloadSpeedLimit *int     // 下载速度限制指针
    ipLimit            *int     // IP 限制指针
    ctx                context.Context  // 上下文对象
}

func (apiController) Name() string {
    return "api"  // 返回控制器名称
}

func (o *apiController) listUsers(apiClient service.TrojanServerServiceClient) error {
    stream, err := apiClient.ListUsers(o.ctx, &service.ListUsersRequest{})  // 调用 gRPC 客户端的 ListUsers 方法
    if err != nil {
        return err  // 返回错误
    }
    defer stream.CloseSend()  // 延迟关闭发送流
    result := []*service.ListUsersResponse{}  // 初始化结果数组
    for {
        resp, err := stream.Recv()  // 接收流数据
        if err != nil {
            if err == io.EOF {  // 如果是文件结束错误
                break  // 跳出循环
            }
            return err  // 返回错误
        }
        result = append(result, resp)  // 将接收到的数据追加到结果数组
    }
    data, err := json.Marshal(result)  // 将结果数组转换为 JSON 格式
    common.Must(err)  // 检查错误
    fmt.Println(string(data))  // 打印 JSON 数据
    return nil  // 返回空
}

func (o *apiController) getUsers(apiClient service.TrojanServerServiceClient) error {
    stream, err := apiClient.GetUsers(o.ctx)  // 调用 gRPC 客户端的 GetUsers 方法
    if err != nil {
        return err  // 返回错误
    }
    defer stream.CloseSend()  // 延迟关闭发送流
    err = stream.Send(&service.GetUsersRequest{  // 发送请求
        User: &service.User{  // 用户对象
            Password: *o.password,  // 设置密码
            Hash:     *o.hash,  // 设置哈希
        },
    })
    if err != nil {
        return err  // 返回错误
    }
    resp, err := stream.Recv()  // 接收流数据
    if err != nil {
        return err  // 返回错误
    }
    data, err := json.Marshal(resp)  // 将接收到的数据转换为 JSON 格式
    common.Must(err)  // 检查错误
    fmt.Print(string(data))  // 打印 JSON 数据
    return nil  // 返回空
}
func (o *apiController) setUsers(apiClient service.TrojanServerServiceClient) error {
    // 调用 API 客户端的 SetUsers 方法，获取流和错误信息
    stream, err := apiClient.SetUsers(o.ctx)
    if err != nil {
        return err
    }
    // 延迟关闭发送流
    defer stream.CloseSend()

    // 创建设置用户请求
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

    // 根据命令类型设置操作类型
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

    // 发送设置用户请求
    err = stream.Send(req)
    if err != nil {
        return err
    }
    // 接收响应
    resp, err := stream.Recv()
    if err != nil {
        return err
    }
    // 根据响应结果输出信息
    if resp.Success {
        fmt.Println("Done")
    } else {
        fmt.Println("Failed: " + resp.Info)
    }
    return nil
}

func (o *apiController) Handle() error {
    // 如果命令为空，则返回错误
    if *o.cmd == "" {
        return common.NewError("")
    }
    // 连接到 gRPC 服务器
    conn, err := grpc.Dial(*o.address, grpc.WithInsecure())
    if err != nil {
        log.Error(err)
        return nil
    }
    // 延迟关闭连接
    defer conn.Close()
    // 创建 API 客户端
    apiClient := service.NewTrojanServerServiceClient(conn)
    // 根据命令类型执行相应操作
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
# 返回 API 控制器的优先级
func (o *apiController) Priority() int {
    return 50
}

# 初始化 API 控制器
func init() {
    # 注册 API 控制器的处理程序
    option.RegisterHandler(&apiController{
        # 设置命令行参数，用于连接到 Trojan-Go API 服务
        cmd:                flag.String("api", "", "Connect to a Trojan-Go API service. \"-api add/get/list\""),
        # 设置 Trojan-Go API 服务的地址
        address:            flag.String("api-addr", "127.0.0.1:10000", "Address of Trojan-Go API service"),
        # 设置目标用户的密码
        password:           flag.String("target-password", "", "Password of the target user"),
        # 设置目标用户的哈希值
        hash:               flag.String("target-hash", "", "Hash of the target user"),
        # 设置是否添加新配置文件的标志
        add:                flag.Bool("add-profile", false, "Add a new profile with API"),
        # 设置是否删除现有配置文件的标志
        delete:             flag.Bool("delete-profile", false, "Delete an existing profile with API"),
        # 设置是否修改现有配置文件的标志
        modify:             flag.Bool("modify-profile", false, "Modify an existing profile with API"),
        # 设置上传速度限制
        uploadSpeedLimit:   flag.Int("upload-speed-limit", 0, "Limit the upload speed with API"),
        # 设置下载速度限制
        downloadSpeedLimit: flag.Int("download-speed-limit", 0, "Limit the download speed with API"),
        # 设置 IP 数量限制
        ipLimit:            flag.Int("ip-limit", 0, "Limit the number of IP with API"),
        # 创建一个空的上下文
        ctx:                context.Background(),
    })
}
```