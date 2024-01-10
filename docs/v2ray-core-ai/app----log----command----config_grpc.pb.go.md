# `v2ray-core\app\log\command\config_grpc.pb.go`

```
// 由 protoc-gen-go-grpc 生成的代码。请勿编辑。

package command

import (
    context "context"  // 导入 context 包并重命名为 context
    grpc "google.golang.org/grpc"  // 导入 grpc 包并重命名为 grpc
    codes "google.golang.org/grpc/codes"  // 导入 codes 包并重命名为 codes
    status "google.golang.org/grpc/status"  // 导入 status 包并重命名为 status
)

// 这是一个编译时断言，用于确保生成的文件与正在编译的 grpc 包兼容。
const _ = grpc.SupportPackageIsVersion7

// LoggerServiceClient 是 LoggerService 服务的客户端 API。
//
// 有关 ctx 使用和关闭/结束流式 RPC 的语义，请参阅 https://pkg.go.dev/google.golang.org/grpc/?tab=doc#ClientConn.NewStream。
type LoggerServiceClient interface {
    RestartLogger(ctx context.Context, in *RestartLoggerRequest, opts ...grpc.CallOption) (*RestartLoggerResponse, error)
}

type loggerServiceClient struct {
    cc grpc.ClientConnInterface
}

func NewLoggerServiceClient(cc grpc.ClientConnInterface) LoggerServiceClient {
    return &loggerServiceClient{cc}
}

func (c *loggerServiceClient) RestartLogger(ctx context.Context, in *RestartLoggerRequest, opts ...grpc.CallOption) (*RestartLoggerResponse, error) {
    out := new(RestartLoggerResponse)
    err := c.cc.Invoke(ctx, "/v2ray.core.app.log.command.LoggerService/RestartLogger", in, out, opts...)
    if err != nil {
        return nil, err
    }
    return out, nil
}

// LoggerServiceServer 是 LoggerService 服务的服务器 API。
// 所有实现都必须嵌入 UnimplementedLoggerServiceServer 以实现向前兼容
type LoggerServiceServer interface {
    RestartLogger(context.Context, *RestartLoggerRequest) (*RestartLoggerResponse, error)
    mustEmbedUnimplementedLoggerServiceServer()
}

// UnimplementedLoggerServiceServer 必须嵌入以实现向前兼容的实现。
type UnimplementedLoggerServiceServer struct {
}

func (UnimplementedLoggerServiceServer) RestartLogger(context.Context, *RestartLoggerRequest) (*RestartLoggerResponse, error) {
    # 返回空值和未实现的错误信息
    return nil, status.Errorf(codes.Unimplemented, "method RestartLogger not implemented")
// 定义 UnimplementedLoggerServiceServer 结构体，实现 mustEmbedUnimplementedLoggerServiceServer 方法
func (UnimplementedLoggerServiceServer) mustEmbedUnimplementedLoggerServiceServer() {}

// UnsafeLoggerServiceServer 可以被嵌入以放弃对该服务的向前兼容性。
// 不建议使用此接口，因为向 LoggerServiceServer 添加方法将导致编译错误。
type UnsafeLoggerServiceServer interface {
    mustEmbedUnimplementedLoggerServiceServer()
}

// RegisterLoggerServiceServer 用于注册 LoggerServiceServer 到 gRPC 服务器
func RegisterLoggerServiceServer(s *grpc.Server, srv LoggerServiceServer) {
    s.RegisterService(&_LoggerService_serviceDesc, srv)
}

// _LoggerService_RestartLogger_Handler 是用于处理 RestartLogger 方法的 gRPC 服务器端处理程序
func _LoggerService_RestartLogger_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    in := new(RestartLoggerRequest)
    if err := dec(in); err != nil {
        return nil, err
    }
    if interceptor == nil {
        return srv.(LoggerServiceServer).RestartLogger(ctx, in)
    }
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/v2ray.core.app.log.command.LoggerService/RestartLogger",
    }
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(LoggerServiceServer).RestartLogger(ctx, req.(*RestartLoggerRequest))
    }
    return interceptor(ctx, in, info, handler)
}

// _LoggerService_serviceDesc 是 LoggerServiceServer 的 gRPC 服务描述
var _LoggerService_serviceDesc = grpc.ServiceDesc{
    ServiceName: "v2ray.core.app.log.command.LoggerService",
    HandlerType: (*LoggerServiceServer)(nil),
    Methods: []grpc.MethodDesc{
        {
            MethodName: "RestartLogger",
            Handler:    _LoggerService_RestartLogger_Handler,
        },
    },
    Streams:  []grpc.StreamDesc{},
    Metadata: "app/log/command/config.proto",
}
```