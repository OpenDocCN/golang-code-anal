# `v2ray-core\app\router\command\command_grpc.pb.go`

```go
// 由 protoc-gen-go-grpc 生成的代码。请勿编辑。

package command

import (
    context "context"  // 导入 context 包
    grpc "google.golang.org/grpc"  // 导入 grpc 包
    codes "google.golang.org/grpc/codes"  // 导入 codes 包
    status "google.golang.org/grpc/status"  // 导入 status 包
)

// 这是一个编译时断言，用于确保生成的文件与正在编译的 grpc 包兼容。
const _ = grpc.SupportPackageIsVersion7

// RoutingServiceClient 是 RoutingService 服务的客户端 API。
//
// 有关 ctx 使用和关闭/结束流式 RPC 的语义，请参阅 https://pkg.go.dev/google.golang.org/grpc/?tab=doc#ClientConn.NewStream。
type RoutingServiceClient interface {
    SubscribeRoutingStats(ctx context.Context, in *SubscribeRoutingStatsRequest, opts ...grpc.CallOption) (RoutingService_SubscribeRoutingStatsClient, error)  // 订阅路由统计信息
    TestRoute(ctx context.Context, in *TestRouteRequest, opts ...grpc.CallOption) (*RoutingContext, error)  // 测试路由
}

type routingServiceClient struct {
    cc grpc.ClientConnInterface
}

func NewRoutingServiceClient(cc grpc.ClientConnInterface) RoutingServiceClient {
    return &routingServiceClient{cc}  // 创建并返回 RoutingServiceClient 对象
}

func (c *routingServiceClient) SubscribeRoutingStats(ctx context.Context, in *SubscribeRoutingStatsRequest, opts ...grpc.CallOption) (RoutingService_SubscribeRoutingStatsClient, error) {
    stream, err := c.cc.NewStream(ctx, &_RoutingService_serviceDesc.Streams[0], "/v2ray.core.app.router.command.RoutingService/SubscribeRoutingStats", opts...)  // 使用给定的上下文、流描述符和选项创建新的流
    if err != nil {
        return nil, err
    }
    x := &routingServiceSubscribeRoutingStatsClient{stream}  // 创建并返回 RoutingService_SubscribeRoutingStatsClient 对象
    if err := x.ClientStream.SendMsg(in); err != nil {
        return nil, err
    }
    if err := x.ClientStream.CloseSend(); err != nil {
        return nil, err
    }
    return x, nil
}

type RoutingService_SubscribeRoutingStatsClient interface {
    Recv() (*RoutingContext, error)  // 接收路由上下文
    grpc.ClientStream
}

type routingServiceSubscribeRoutingStatsClient struct {
    grpc.ClientStream
}
// 接收路由统计客户端的Recv方法，返回RoutingContext和error
func (x *routingServiceSubscribeRoutingStatsClient) Recv() (*RoutingContext, error) {
    // 创建一个新的RoutingContext对象
    m := new(RoutingContext)
    // 从ClientStream接收消息并存储到m中，如果出现错误则返回nil和错误
    if err := x.ClientStream.RecvMsg(m); err != nil {
        return nil, err
    }
    // 返回RoutingContext对象和nil
    return m, nil
}

// 发起TestRoute请求，返回RoutingContext和error
func (c *routingServiceClient) TestRoute(ctx context.Context, in *TestRouteRequest, opts ...grpc.CallOption) (*RoutingContext, error) {
    // 创建一个新的RoutingContext对象
    out := new(RoutingContext)
    // 调用cc的Invoke方法，传入上下文ctx、请求in、响应out和选项opts，如果出现错误则返回nil和错误
    err := c.cc.Invoke(ctx, "/v2ray.core.app.router.command.RoutingService/TestRoute", in, out, opts...)
    if err != nil {
        return nil, err
    }
    // 返回RoutingContext对象和nil
    return out, nil
}

// RoutingServiceServer是RoutingService服务的服务器API
// 所有实现都必须嵌入UnimplementedRoutingServiceServer以实现向前兼容
type RoutingServiceServer interface {
    // 订阅路由统计，接收SubscribeRoutingStatsRequest和RoutingService_SubscribeRoutingStatsServer，返回error
    SubscribeRoutingStats(*SubscribeRoutingStatsRequest, RoutingService_SubscribeRoutingStatsServer) error
    // 测试路由，接收上下文ctx和TestRouteRequest，返回RoutingContext和error
    TestRoute(context.Context, *TestRouteRequest) (*RoutingContext, error)
    // 必须嵌入UnimplementedRoutingServiceServer
    mustEmbedUnimplementedRoutingServiceServer()
}

// UnimplementedRoutingServiceServer必须被嵌入以实现向前兼容
type UnimplementedRoutingServiceServer struct {
}

// SubscribeRoutingStats方法，接收SubscribeRoutingStatsRequest和RoutingService_SubscribeRoutingStatsServer，返回未实现错误
func (UnimplementedRoutingServiceServer) SubscribeRoutingStats(*SubscribeRoutingStatsRequest, RoutingService_SubscribeRoutingStatsServer) error {
    return status.Errorf(codes.Unimplemented, "method SubscribeRoutingStats not implemented")
}

// TestRoute方法，接收上下文ctx和TestRouteRequest，返回nil和未实现错误
func (UnimplementedRoutingServiceServer) TestRoute(context.Context, *TestRouteRequest) (*RoutingContext, error) {
    return nil, status.Errorf(codes.Unimplemented, "method TestRoute not implemented")
}

// 必须嵌入UnimplementedRoutingServiceServer
func (UnimplementedRoutingServiceServer) mustEmbedUnimplementedRoutingServiceServer() {}

// UnsafeRoutingServiceServer可以被嵌入以选择不向前兼容这个服务
// 不建议使用此接口，因为向RoutingServiceServer添加方法将导致编译错误
type UnsafeRoutingServiceServer interface {
    # 调用必须实现的未实现的路由服务服务器
    mustEmbedUnimplementedRoutingServiceServer()
}

// 注册路由服务的 gRPC 服务器
func RegisterRoutingServiceServer(s *grpc.Server, srv RoutingServiceServer) {
    s.RegisterService(&_RoutingService_serviceDesc, srv)
}

// 处理订阅路由统计信息的 gRPC 请求
func _RoutingService_SubscribeRoutingStats_Handler(srv interface{}, stream grpc.ServerStream) error {
    // 创建一个订阅路由统计信息的请求对象
    m := new(SubscribeRoutingStatsRequest)
    // 接收流中的消息并解码到请求对象中
    if err := stream.RecvMsg(m); err != nil {
        return err
    }
    // 调用服务端的 SubscribeRoutingStats 方法处理请求
    return srv.(RoutingServiceServer).SubscribeRoutingStats(m, &routingServiceSubscribeRoutingStatsServer{stream})
}

// 订阅路由统计信息的 gRPC 服务器流接口
type RoutingService_SubscribeRoutingStatsServer interface {
    Send(*RoutingContext) error
    grpc.ServerStream
}

// 订阅路由统计信息的 gRPC 服务器流实现
type routingServiceSubscribeRoutingStatsServer struct {
    grpc.ServerStream
}

// 发送路由上下文信息到客户端
func (x *routingServiceSubscribeRoutingStatsServer) Send(m *RoutingContext) error {
    return x.ServerStream.SendMsg(m)
}

// 处理测试路由的 gRPC 请求
func _RoutingService_TestRoute_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    // 创建一个测试路由的请求对象
    in := new(TestRouteRequest)
    // 解码客户端请求
    if err := dec(in); err != nil {
        return nil, err
    }
    // 如果没有拦截器，则直接调用服务端的 TestRoute 方法处理请求
    if interceptor == nil {
        return srv.(RoutingServiceServer).TestRoute(ctx, in)
    }
    // 创建 gRPC 一元拦截器信息
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/v2ray.core.app.router.command.RoutingService/TestRoute",
    }
    // 创建处理器函数
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(RoutingServiceServer).TestRoute(ctx, req.(*TestRouteRequest))
    }
    // 调用拦截器处理请求
    return interceptor(ctx, in, info, handler)
}

// 路由服务的 gRPC 服务描述
var _RoutingService_serviceDesc = grpc.ServiceDesc{
    ServiceName: "v2ray.core.app.router.command.RoutingService",
    HandlerType: (*RoutingServiceServer)(nil),
    Methods: []grpc.MethodDesc{
        {
            MethodName: "TestRoute",
            Handler:    _RoutingService_TestRoute_Handler,
        },
    },
    # 定义一个流描述符数组，包含一个流描述符对象
    Streams: []grpc.StreamDesc{
        {
            # 设置流的名称为"SubscribeRoutingStats"
            StreamName:    "SubscribeRoutingStats",
            # 设置处理程序为_RoutingService_SubscribeRoutingStats_Handler
            Handler:       _RoutingService_SubscribeRoutingStats_Handler,
            # 设置服务器流为true
            ServerStreams: true,
        },
    },
    # 设置元数据为"app/router/command/command.proto"
    Metadata: "app/router/command/command.proto",
# 闭合前面的函数定义
```