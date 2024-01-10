# `v2ray-core\infra\control\api.go`

```
package control

import (
    "context" // 上下文包，用于控制请求的取消、超时等
    "errors" // 错误处理包，用于创建和处理错误
    "flag" // 命令行参数解析包，用于解析命令行参数
    "fmt" // 格式化包，用于格式化输出
    "strings" // 字符串处理包，用于处理字符串
    "time" // 时间包，用于处理时间

    "github.com/golang/protobuf/proto" // Google Protocol Buffers 包，用于处理协议缓冲区
    "google.golang.org/grpc" // gRPC 包，用于构建和处理 gRPC 服务

    logService "v2ray.com/core/app/log/command" // V2Ray 日志服务包
    statsService "v2ray.com/core/app/stats/command" // V2Ray 统计服务包
    "v2ray.com/core/common" // V2Ray 公共包
)

type ApiCommand struct{} // 定义 ApiCommand 结构体

func (c *ApiCommand) Name() string { // 定义 Name 方法
    return "api" // 返回字符串 "api"
}

func (c *ApiCommand) Description() Description { // 定义 Description 方法
    return Description{ // 返回 Description 结构体
        Short: "Call V2Ray API", // 简短描述
        Usage: []string{ // 用法说明
            "v2ctl api [--server=127.0.0.1:8080] Service.Method Request",
            "Call an API in an V2Ray process.",
            "The following methods are currently supported:",
            "\tLoggerService.RestartLogger",
            "\tStatsService.GetStats",
            "\tStatsService.QueryStats",
            "API calls in this command have a timeout to the server of 3 seconds.",
            "Examples:",
            "v2ctl api --server=127.0.0.1:8080 LoggerService.RestartLogger '' ",
            "v2ctl api --server=127.0.0.1:8080 StatsService.QueryStats 'pattern: \"\" reset: false'",
            "v2ctl api --server=127.0.0.1:8080 StatsService.GetStats 'name: \"inbound>>>statin>>>traffic>>>downlink\" reset: false'",
            "v2ctl api --server=127.0.0.1:8080 StatsService.GetSysStats ''",
        },
    }
}

func (c *ApiCommand) Execute(args []string) error { // 定义 Execute 方法
    fs := flag.NewFlagSet(c.Name(), flag.ContinueOnError) // 创建命令行参数解析对象

    serverAddrPtr := fs.String("server", "127.0.0.1:8080", "Server address") // 定义 serverAddrPtr 指针变量，用于存储服务器地址

    if err := fs.Parse(args); err != nil { // 解析命令行参数
        return err // 返回错误
    }

    unnamedArgs := fs.Args() // 获取未命名参数
    if len(unnamedArgs) < 2 { // 如果未命名参数少于 2 个
        return newError("service name or request not specified.") // 返回新错误
    }

    service, method := getServiceMethod(unnamedArgs[0]) // 获取服务和方法
    handler, found := serivceHandlerMap[strings.ToLower(service)] // 获取处理程序
    if !found { // 如果未找到
        return newError("unknown service: ", service) // 返回新错误
    }
}
    # 使用 context 包创建一个带有超时时间的上下文
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    # 在函数返回时调用 cancel 函数，释放资源
    defer cancel()

    # 使用 grpc 包基于上下文创建一个连接
    conn, err := grpc.DialContext(ctx, *serverAddrPtr, grpc.WithInsecure(), grpc.WithBlock())
    # 如果连接失败，返回一个包含错误信息的新错误
    if err != nil {
        return newError("failed to dial ", *serverAddrPtr).Base(err)
    }
    # 在函数返回时关闭连接
    defer conn.Close()

    # 调用 handler 函数处理请求，并获取响应
    response, err := handler(ctx, conn, method, unnamedArgs[1])
    # 如果处理请求出错，返回一个包含错误信息的新错误
    if err != nil {
        return newError("failed to call service ", unnamedArgs[0]).Base(err)
    }

    # 打印响应内容
    fmt.Println(response)
    # 返回空值
    return nil
# 定义一个函数，根据输入的字符串s，返回其中的服务名和方法名
func getServiceMethod(s string) (string, string) {
    # 使用"."分割字符串s，得到服务名和方法名的数组
    ss := strings.Split(s, ".")
    # 从数组中获取服务名
    service := ss[0]
    # 定义方法名变量
    var method string
    # 如果数组长度大于1，说明有方法名，进行赋值
    if len(ss) > 1 {
        method = ss[1]
    }
    # 返回服务名和方法名
    return service, method
}

# 定义一个类型为serviceHandler的函数类型，接受上下文、客户端连接、方法名和请求字符串，返回响应字符串和错误
type serviceHandler func(ctx context.Context, conn *grpc.ClientConn, method string, request string) (string, error)

# 定义一个map，将服务名和对应的处理函数关联起来
var serivceHandlerMap = map[string]serviceHandler{
    "statsservice":  callStatsService,  # 将"statsservice"服务名和callStatsService函数关联
    "loggerservice": callLogService,     # 将"loggerservice"服务名和callLogService函数关联
}

# 定义一个处理log服务的函数，接受上下文、客户端连接、方法名和请求字符串，返回响应字符串和错误
func callLogService(ctx context.Context, conn *grpc.ClientConn, method string, request string) (string, error) {
    # 创建log服务的客户端
    client := logService.NewLoggerServiceClient(conn)

    # 根据方法名进行不同的处理
    switch strings.ToLower(method) {
    case "restartlogger":  # 如果方法名为"restartlogger"
        # 创建一个RestartLoggerRequest对象，并将请求字符串解析为对象
        r := &logService.RestartLoggerRequest{}
        if err := proto.UnmarshalText(request, r); err != nil {
            return "", err
        }
        # 调用客户端的RestartLogger方法，得到响应和错误
        resp, err := client.RestartLogger(ctx, r)
        if err != nil {
            return "", err
        }
        # 将响应对象转换为文本格式并返回
        return proto.MarshalTextString(resp), nil
    default:  # 如果方法名为其他值
        return "", errors.New("Unknown method: " + method)  # 返回错误信息
    }
}

# 定义一个处理stats服务的函数，接受上下文、客户端连接、方法名和请求字符串，返回响应字符串和错误
func callStatsService(ctx context.Context, conn *grpc.ClientConn, method string, request string) (string, error) {
    # 创建stats服务的客户端
    client := statsService.NewStatsServiceClient(conn)

    # 根据方法名进行不同的处理
    switch strings.ToLower(method) {
    case "getstats":  # 如果方法名为"getstats"
        # 创建一个GetStatsRequest对象，并将请求字符串解析为对象
        r := &statsService.GetStatsRequest{}
        if err := proto.UnmarshalText(request, r); err != nil {
            return "", err
        }
        # 调用客户端的GetStats方法，得到响应和错误
        resp, err := client.GetStats(ctx, r)
        if err != nil {
            return "", err
        }
        # 将响应对象转换为文本格式并返回
        return proto.MarshalTextString(resp), nil
    case "querystats":  # 如果方法名为"querystats"
        # 创建一个QueryStatsRequest对象，并将请求字符串解析为对象
        r := &statsService.QueryStatsRequest{}
        if err := proto.UnmarshalText(request, r); err != nil {
            return "", err
        }
        # 调用客户端的QueryStats方法，得到响应和错误
        resp, err := client.QueryStats(ctx, r)
        if err != nil {
            return "", err
        }
        # 将响应对象转换为文本格式并返回
        return proto.MarshalTextString(resp), nil
    case "getsysstats":
        // 如果方法名为"getsysstats"，则执行以下操作
        // 创建一个空的 SysStatsRequest 消息
        r := &statsService.SysStatsRequest{}
        // 使用客户端调用 GetSysStats 方法，传入上下文和空消息
        resp, err := client.GetSysStats(ctx, r)
        // 如果出现错误，返回空字符串和错误信息
        if err != nil {
            return "", err
        }
        // 将响应消息转换为文本格式并返回
        return proto.MarshalTextString(resp), nil
    default:
        // 如果方法名不是"getsysstats"，返回错误信息
        return "", errors.New("Unknown method: " + method)
    }
# 初始化函数，用于注册ApiCommand命令
func init() {
    # 调用common包中的Must函数，用于注册ApiCommand命令
    common.Must(RegisterCommand(&ApiCommand{}))
}
```