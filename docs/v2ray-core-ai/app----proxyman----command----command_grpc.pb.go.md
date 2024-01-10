# `v2ray-core\app\proxyman\command\command_grpc.pb.go`

```
// 由 protoc-gen-go-grpc 生成的代码。请勿编辑。

// 导入必要的包
package command

import (
    context "context"  // 导入 context 包
    grpc "google.golang.org/grpc"  // 导入 grpc 包
    codes "google.golang.org/grpc/codes"  // 导入 codes 包
    status "google.golang.org/grpc/status"  // 导入 status 包
)

// 这是一个编译时断言，用于确保生成的文件与正在编译的 grpc 包兼容。
const _ = grpc.SupportPackageIsVersion7

// HandlerServiceClient 是 HandlerService 服务的客户端 API。
//
// 有关 ctx 使用和关闭/结束流式 RPC 的语义，请参阅 https://pkg.go.dev/google.golang.org/grpc/?tab=doc#ClientConn.NewStream。
type HandlerServiceClient interface {
    AddInbound(ctx context.Context, in *AddInboundRequest, opts ...grpc.CallOption) (*AddInboundResponse, error)
    RemoveInbound(ctx context.Context, in *RemoveInboundRequest, opts ...grpc.CallOption) (*RemoveInboundResponse, error)
    AlterInbound(ctx context.Context, in *AlterInboundRequest, opts ...grpc.CallOption) (*AlterInboundResponse, error)
    AddOutbound(ctx context.Context, in *AddOutboundRequest, opts ...grpc.CallOption) (*AddOutboundResponse, error)
    RemoveOutbound(ctx context.Context, in *RemoveOutboundRequest, opts ...grpc.CallOption) (*RemoveOutboundResponse, error)
    AlterOutbound(ctx context.Context, in *AlterOutboundRequest, opts ...grpc.CallOption) (*AlterOutboundResponse, error)
}

type handlerServiceClient struct {
    cc grpc.ClientConnInterface
}

// 创建新的 HandlerServiceClient
func NewHandlerServiceClient(cc grpc.ClientConnInterface) HandlerServiceClient {
    return &handlerServiceClient{cc}
}

// 调用 AddInbound RPC
func (c *handlerServiceClient) AddInbound(ctx context.Context, in *AddInboundRequest, opts ...grpc.CallOption) (*AddInboundResponse, error) {
    out := new(AddInboundResponse)  // 创建 AddInboundResponse 对象
    err := c.cc.Invoke(ctx, "/v2ray.core.app.proxyman.command.HandlerService/AddInbound", in, out, opts...)  // 调用 Invoke 方法执行 RPC
    if err != nil {
        return nil, err
    }
    return out, nil  // 返回结果
}
# 从handlerServiceClient中移除入站代理
func (c *handlerServiceClient) RemoveInbound(ctx context.Context, in *RemoveInboundRequest, opts ...grpc.CallOption) (*RemoveInboundResponse, error) {
    # 创建一个新的RemoveInboundResponse对象
    out := new(RemoveInboundResponse)
    # 调用cc的Invoke方法，传入上下文、方法路径、输入参数、输出参数和选项，返回错误
    err := c.cc.Invoke(ctx, "/v2ray.core.app.proxyman.command.HandlerService/RemoveInbound", in, out, opts...)
    # 如果有错误发生，则返回nil和错误
    if err != nil {
        return nil, err
    }
    # 返回输出参数和nil
    return out, nil
}

# 修改入站代理
func (c *handlerServiceClient) AlterInbound(ctx context.Context, in *AlterInboundRequest, opts ...grpc.CallOption) (*AlterInboundResponse, error) {
    # 创建一个新的AlterInboundResponse对象
    out := new(AlterInboundResponse)
    # 调用cc的Invoke方法，传入上下文、方法路径、输入参数、输出参数和选项，返回错误
    err := c.cc.Invoke(ctx, "/v2ray.core.app.proxyman.command.HandlerService/AlterInbound", in, out, opts...)
    # 如果有错误发生，则返回nil和错误
    if err != nil {
        return nil, err
    }
    # 返回输出参数和nil
    return out, nil
}

# 添加出站代理
func (c *handlerServiceClient) AddOutbound(ctx context.Context, in *AddOutboundRequest, opts ...grpc.CallOption) (*AddOutboundResponse, error) {
    # 创建一个新的AddOutboundResponse对象
    out := new(AddOutboundResponse)
    # 调用cc的Invoke方法，传入上下文、方法路径、输入参数、输出参数和选项，返回错误
    err := c.cc.Invoke(ctx, "/v2ray.core.app.proxyman.command.HandlerService/AddOutbound", in, out, opts...)
    # 如果有错误发生，则返回nil和错误
    if err != nil {
        return nil, err
    }
    # 返回输出参数和nil
    return out, nil
}

# 从handlerServiceClient中移除出站代理
func (c *handlerServiceClient) RemoveOutbound(ctx context.Context, in *RemoveOutboundRequest, opts ...grpc.CallOption) (*RemoveOutboundResponse, error) {
    # 创建一个新的RemoveOutboundResponse对象
    out := new(RemoveOutboundResponse)
    # 调用cc的Invoke方法，传入上下文、方法路径、输入参数、输出参数和选项，返回错误
    err := c.cc.Invoke(ctx, "/v2ray.core.app.proxyman.command.HandlerService/RemoveOutbound", in, out, opts...)
    # 如果有错误发生，则返回nil和错误
    if err != nil {
        return nil, err
    }
    # 返回输出参数和nil
    return out, nil
}

# 修改出站代理
func (c *handlerServiceClient) AlterOutbound(ctx context.Context, in *AlterOutboundRequest, opts ...grpc.CallOption) (*AlterOutboundResponse, error) {
    # 创建一个新的AlterOutboundResponse对象
    out := new(AlterOutboundResponse)
    # 调用cc的Invoke方法，传入上下文、方法路径、输入参数、输出参数和选项，返回错误
    err := c.cc.Invoke(ctx, "/v2ray.core.app.proxyman.command.HandlerService/AlterOutbound", in, out, opts...)
    # 如果有错误发生，则返回nil和错误
    if err != nil {
        return nil, err
    }
    # 返回输出参数和nil
    return out, nil
}

# HandlerServiceServer是HandlerService服务的服务器API
# 所有实现都必须嵌入UnimplementedHandlerServiceServer
// 为了向前兼容而定义的接口，包含了处理服务的各种方法
type HandlerServiceServer interface {
    AddInbound(context.Context, *AddInboundRequest) (*AddInboundResponse, error)  // 添加入站请求的方法
    RemoveInbound(context.Context, *RemoveInboundRequest) (*RemoveInboundResponse, error)  // 移除入站请求的方法
    AlterInbound(context.Context, *AlterInboundRequest) (*AlterInboundResponse, error)  // 修改入站请求的方法
    AddOutbound(context.Context, *AddOutboundRequest) (*AddOutboundResponse, error)  // 添加出站请求的方法
    RemoveOutbound(context.Context, *RemoveOutboundRequest) (*RemoveOutboundResponse, error)  // 移除出站请求的方法
    AlterOutbound(context.Context, *AlterOutboundRequest) (*AlterOutboundResponse, error)  // 修改出站请求的方法
    mustEmbedUnimplementedHandlerServiceServer()  // 必须嵌入未实现的处理服务服务器
}

// 未实现的处理服务服务器必须被嵌入以具有向前兼容的实现
type UnimplementedHandlerServiceServer struct {
}

func (UnimplementedHandlerServiceServer) AddInbound(context.Context, *AddInboundRequest) (*AddInboundResponse, error) {
    return nil, status.Errorf(codes.Unimplemented, "method AddInbound not implemented")  // 返回未实现的错误信息
}
func (UnimplementedHandlerServiceServer) RemoveInbound(context.Context, *RemoveInboundRequest) (*RemoveInboundResponse, error) {
    return nil, status.Errorf(codes.Unimplemented, "method RemoveInbound not implemented")  // 返回未实现的错误信息
}
func (UnimplementedHandlerServiceServer) AlterInbound(context.Context, *AlterInboundRequest) (*AlterInboundResponse, error) {
    return nil, status.Errorf(codes.Unimplemented, "method AlterInbound not implemented")  // 返回未实现的错误信息
}
func (UnimplementedHandlerServiceServer) AddOutbound(context.Context, *AddOutboundRequest) (*AddOutboundResponse, error) {
    return nil, status.Errorf(codes.Unimplemented, "method AddOutbound not implemented")  // 返回未实现的错误信息
}
func (UnimplementedHandlerServiceServer) RemoveOutbound(context.Context, *RemoveOutboundRequest) (*RemoveOutboundResponse, error) {
    return nil, status.Errorf(codes.Unimplemented, "method RemoveOutbound not implemented")  // 返回未实现的错误信息
}
// 实现 AlterOutbound 方法，返回未实现错误
func (UnimplementedHandlerServiceServer) AlterOutbound(context.Context, *AlterOutboundRequest) (*AlterOutboundResponse, error) {
    return nil, status.Errorf(codes.Unimplemented, "method AlterOutbound not implemented")
}
// 实现 mustEmbedUnimplementedHandlerServiceServer 方法
func (UnimplementedHandlerServiceServer) mustEmbedUnimplementedHandlerServiceServer() {}

// UnsafeHandlerServiceServer 可以嵌入以退出此服务的向前兼容性。
// 不建议使用此接口，因为向 HandlerServiceServer 添加方法将导致编译错误。
type UnsafeHandlerServiceServer interface {
    mustEmbedUnimplementedHandlerServiceServer()
}

// 注册 HandlerServiceServer 到 gRPC 服务器
func RegisterHandlerServiceServer(s *grpc.Server, srv HandlerServiceServer) {
    s.RegisterService(&_HandlerService_serviceDesc, srv)
}

// 处理 AddInbound 方法的 gRPC 服务器处理程序
func _HandlerService_AddInbound_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    in := new(AddInboundRequest)
    if err := dec(in); err != nil {
        return nil, err
    }
    if interceptor == nil {
        return srv.(HandlerServiceServer).AddInbound(ctx, in)
    }
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/v2ray.core.app.proxyman.command.HandlerService/AddInbound",
    }
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(HandlerServiceServer).AddInbound(ctx, req.(*AddInboundRequest))
    }
    return interceptor(ctx, in, info, handler)
}

// 处理 RemoveInbound 方法的 gRPC 服务器处理程序
func _HandlerService_RemoveInbound_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    in := new(RemoveInboundRequest)
    if err := dec(in); err != nil {
        return nil, err
    }
    if interceptor == nil {
        return srv.(HandlerServiceServer).RemoveInbound(ctx, in)
    }
    # 创建一个包含服务器信息的结构体指针
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/v2ray.core.app.proxyman.command.HandlerService/RemoveInbound",
    }
    # 创建一个处理函数，接收上下文和请求参数，返回结果和错误
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(HandlerServiceServer).RemoveInbound(ctx, req.(*RemoveInboundRequest))
    }
    # 调用拦截器函数，传入上下文、输入参数、服务器信息和处理函数
    return interceptor(ctx, in, info, handler)
# 处理 AlterInbound 请求的处理函数
func _HandlerService_AlterInbound_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    # 创建一个新的 AlterInboundRequest 对象
    in := new(AlterInboundRequest)
    # 解码请求数据到 AlterInboundRequest 对象
    if err := dec(in); err != nil {
        return nil, err
    }
    # 如果拦截器为空，则直接调用 AlterInbound 方法处理请求
    if interceptor == nil {
        return srv.(HandlerServiceServer).AlterInbound(ctx, in)
    }
    # 创建一个包含服务信息的对象
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/v2ray.core.app.proxyman.command.HandlerService/AlterInbound",
    }
    # 创建一个处理函数，用于调用 AlterInbound 方法处理请求
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(HandlerServiceServer).AlterInbound(ctx, req.(*AlterInboundRequest))
    }
    # 调用拦截器处理请求
    return interceptor(ctx, in, info, handler)
}

# 处理 AddOutbound 请求的处理函数
func _HandlerService_AddOutbound_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    # 创建一个新的 AddOutboundRequest 对象
    in := new(AddOutboundRequest)
    # 解码请求数据到 AddOutboundRequest 对象
    if err := dec(in); err != nil {
        return nil, err
    }
    # 如果拦截器为空，则直接调用 AddOutbound 方法处理请求
    if interceptor == nil {
        return srv.(HandlerServiceServer).AddOutbound(ctx, in)
    }
    # 创建一个包含服务信息的对象
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/v2ray.core.app.proxyman.command.HandlerService/AddOutbound",
    }
    # 创建一个处理函数，用于调用 AddOutbound 方法处理请求
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(HandlerServiceServer).AddOutbound(ctx, req.(*AddOutboundRequest))
    }
    # 调用拦截器处理请求
    return interceptor(ctx, in, info, handler)
}

# 处理 RemoveOutbound 请求的处理函数
func _HandlerService_RemoveOutbound_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    # 创建一个新的 RemoveOutboundRequest 对象
    in := new(RemoveOutboundRequest)
    # 解码请求数据到 RemoveOutboundRequest 对象
    if err := dec(in); err != nil {
        return nil, err
    }
    # 如果拦截器为空，则直接调用 RemoveOutbound 方法处理请求
    if interceptor == nil {
        return srv.(HandlerServiceServer).RemoveOutbound(ctx, in)
    }
    # 创建一个包含服务器信息的结构体指针
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/v2ray.core.app.proxyman.command.HandlerService/RemoveOutbound",
    }
    # 创建一个处理函数，接收上下文和请求参数，返回结果和错误
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(HandlerServiceServer).RemoveOutbound(ctx, req.(*RemoveOutboundRequest))
    }
    # 调用拦截器函数，传入上下文、输入参数、服务器信息和处理函数
    return interceptor(ctx, in, info, handler)
func _HandlerService_AlterOutbound_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    // 创建一个新的 AlterOutboundRequest 对象
    in := new(AlterOutboundRequest)
    // 解码输入的数据到 AlterOutboundRequest 对象中
    if err := dec(in); err != nil {
        return nil, err
    }
    // 如果拦截器为空，则直接调用 AlterOutbound 方法处理请求
    if interceptor == nil {
        return srv.(HandlerServiceServer).AlterOutbound(ctx, in)
    }
    // 创建一个包含服务和方法信息的对象
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/v2ray.core.app.proxyman.command.HandlerService/AlterOutbound",
    }
    // 创建一个处理函数，用于调用 AlterOutbound 方法
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(HandlerServiceServer).AlterOutbound(ctx, req.(*AlterOutboundRequest))
    }
    // 调用拦截器处理请求
    return interceptor(ctx, in, info, handler)
}

// 定义 HandlerService 的服务描述
var _HandlerService_serviceDesc = grpc.ServiceDesc{
    ServiceName: "v2ray.core.app.proxyman.command.HandlerService",
    HandlerType: (*HandlerServiceServer)(nil),
    Methods: []grpc.MethodDesc{
        {
            MethodName: "AddInbound",
            Handler:    _HandlerService_AddInbound_Handler,
        },
        {
            MethodName: "RemoveInbound",
            Handler:    _HandlerService_RemoveInbound_Handler,
        },
        {
            MethodName: "AlterInbound",
            Handler:    _HandlerService_AlterInbound_Handler,
        },
        {
            MethodName: "AddOutbound",
            Handler:    _HandlerService_AddOutbound_Handler,
        },
        {
            MethodName: "RemoveOutbound",
            Handler:    _HandlerService_RemoveOutbound_Handler,
        },
        {
            MethodName: "AlterOutbound",
            Handler:    _HandlerService_AlterOutbound_Handler,
        },
    },
    Streams:  []grpc.StreamDesc{},
    Metadata: "app/proxyman/command/command.proto",
}
```