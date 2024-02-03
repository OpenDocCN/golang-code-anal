# `trojan-go\api\service\api_grpc.pb.go`

```go
// 由 protoc-gen-go-grpc 生成的代码。请勿编辑。

package service

import (
    context "context"  // 导入 context 包
    grpc "google.golang.org/grpc"  // 导入 grpc 包
    codes "google.golang.org/grpc/codes"  // 导入 codes 包
    status "google.golang.org/grpc/status"  // 导入 status 包
)

// 这是一个编译时断言，用于确保生成的文件与正在编译的 grpc 包兼容。
// 需要 gRPC-Go v1.32.0 或更高版本。
const _ = grpc.SupportPackageIsVersion7

// TrojanClientServiceClient 是 TrojanClientService 服务的客户端 API。
//
// 有关 ctx 使用和关闭/结束流式 RPC 的语义，请参阅 https://pkg.go.dev/google.golang.org/grpc/?tab=doc#ClientConn.NewStream。
type TrojanClientServiceClient interface {
    GetTraffic(ctx context.Context, in *GetTrafficRequest, opts ...grpc.CallOption) (*GetTrafficResponse, error)
}

type trojanClientServiceClient struct {
    cc grpc.ClientConnInterface
}

func NewTrojanClientServiceClient(cc grpc.ClientConnInterface) TrojanClientServiceClient {
    return &trojanClientServiceClient{cc}
}

func (c *trojanClientServiceClient) GetTraffic(ctx context.Context, in *GetTrafficRequest, opts ...grpc.CallOption) (*GetTrafficResponse, error) {
    out := new(GetTrafficResponse)
    err := c.cc.Invoke(ctx, "/trojan.api.TrojanClientService/GetTraffic", in, out, opts...)
    if err != nil {
        return nil, err
    }
    return out, nil
}

// TrojanClientServiceServer 是 TrojanClientService 服务的服务器 API。
// 所有实现都必须嵌入 UnimplementedTrojanClientServiceServer 以实现向前兼容
type TrojanClientServiceServer interface {
    GetTraffic(context.Context, *GetTrafficRequest) (*GetTrafficResponse, error)
    mustEmbedUnimplementedTrojanClientServiceServer()
}

// UnimplementedTrojanClientServiceServer 必须嵌入以实现向前兼容的实现。
type UnimplementedTrojanClientServiceServer struct {
}
// GetTraffic 方法用于获取流量信息，但当前未实现，返回未实现错误
func (UnimplementedTrojanClientServiceServer) GetTraffic(context.Context, *GetTrafficRequest) (*GetTrafficResponse, error) {
    return nil, status.Errorf(codes.Unimplemented, "method GetTraffic not implemented")
}

// mustEmbedUnimplementedTrojanClientServiceServer 方法用于强制实现未实现的 TrojanClientServiceServer 接口
func (UnimplementedTrojanClientServiceServer) mustEmbedUnimplementedTrojanClientServiceServer() {}

// UnsafeTrojanClientServiceServer 接口用于禁止对 TrojanClientServiceServer 接口进行向前兼容
// 不建议使用此接口，因为对 TrojanClientServiceServer 添加方法将导致编译错误
type UnsafeTrojanClientServiceServer interface {
    mustEmbedUnimplementedTrojanClientServiceServer()
}

// RegisterTrojanClientServiceServer 用于注册 TrojanClientServiceServer
func RegisterTrojanClientServiceServer(s grpc.ServiceRegistrar, srv TrojanClientServiceServer) {
    s.RegisterService(&TrojanClientService_ServiceDesc, srv)
}

// _TrojanClientService_GetTraffic_Handler 用于处理 GetTraffic 方法的请求
func _TrojanClientService_GetTraffic_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    in := new(GetTrafficRequest)
    if err := dec(in); err != nil {
        return nil, err
    }
    if interceptor == nil {
        return srv.(TrojanClientServiceServer).GetTraffic(ctx, in)
    }
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/trojan.api.TrojanClientService/GetTraffic",
    }
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(TrojanClientServiceServer).GetTraffic(ctx, req.(*GetTrafficRequest))
    }
    return interceptor(ctx, in, info, handler)
}

// TrojanClientService_ServiceDesc 是 TrojanClientService 服务的 grpc.ServiceDesc
// 仅用于直接与 grpc.RegisterService 一起使用，不可内省或修改（即使是副本）
var TrojanClientService_ServiceDesc = grpc.ServiceDesc{
    ServiceName: "trojan.api.TrojanClientService",
    HandlerType: (*TrojanClientServiceServer)(nil),
}
    # 定义一个空的 Methods 数组，用于存储 gRPC 方法描述
    Methods: []grpc.MethodDesc{
        # 定义一个 gRPC 方法描述对象，包括方法名和处理函数
        {
            MethodName: "GetTraffic",  # 方法名为 GetTraffic
            Handler:    _TrojanClientService_GetTraffic_Handler,  # 处理函数为 _TrojanClientService_GetTraffic_Handler
        },
    },
    # 定义一个空的 Streams 数组，用于存储 gRPC 流描述
    Streams:  []grpc.StreamDesc{},
    # 指定 API 的元数据文件为 "api.proto"
    Metadata: "api.proto",
// TrojanServerServiceClient 是 TrojanServerService 服务的客户端 API。
//
// 有关 ctx 使用和关闭/结束流式 RPC 的语义，请参考 https://pkg.go.dev/google.golang.org/grpc/?tab=doc#ClientConn.NewStream。
type TrojanServerServiceClient interface {
    // 列出所有用户
    ListUsers(ctx context.Context, in *ListUsersRequest, opts ...grpc.CallOption) (TrojanServerService_ListUsersClient, error)
    // 获取指定用户的信息
    GetUsers(ctx context.Context, opts ...grpc.CallOption) (TrojanServerService_GetUsersClient, error)
    // 设置现有用户的配置
    SetUsers(ctx context.Context, opts ...grpc.CallOption) (TrojanServerService_SetUsersClient, error)
}

type trojanServerServiceClient struct {
    cc grpc.ClientConnInterface
}

func NewTrojanServerServiceClient(cc grpc.ClientConnInterface) TrojanServerServiceClient {
    return &trojanServerServiceClient{cc}
}

func (c *trojanServerServiceClient) ListUsers(ctx context.Context, in *ListUsersRequest, opts ...grpc.CallOption) (TrojanServerService_ListUsersClient, error) {
    // 创建新的流式 RPC
    stream, err := c.cc.NewStream(ctx, &TrojanServerService_ServiceDesc.Streams[0], "/trojan.api.TrojanServerService/ListUsers", opts...)
    if err != nil {
        return nil, err
    }
    x := &trojanServerServiceListUsersClient{stream}
    // 发送消息到服务器
    if err := x.ClientStream.SendMsg(in); err != nil {
        return nil, err
    }
    // 关闭发送端
    if err := x.ClientStream.CloseSend(); err != nil {
        return nil, err
    }
    return x, nil
}

type TrojanServerService_ListUsersClient interface {
    // 接收服务器返回的消息
    Recv() (*ListUsersResponse, error)
    grpc.ClientStream
}

type trojanServerServiceListUsersClient struct {
    grpc.ClientStream
}

func (x *trojanServerServiceListUsersClient) Recv() (*ListUsersResponse, error) {
    m := new(ListUsersResponse)
    // 接收服务器返回的消息
    if err := x.ClientStream.RecvMsg(m); err != nil {
        return nil, err
    }
    return m, nil
}
// GetUsers 方法用于获取 TrojanServerService_GetUsersClient 接口实例
func (c *trojanServerServiceClient) GetUsers(ctx context.Context, opts ...grpc.CallOption) (TrojanServerService_GetUsersClient, error) {
    // 创建新的流对象，用于与服务端进行通信
    stream, err := c.cc.NewStream(ctx, &TrojanServerService_ServiceDesc.Streams[1], "/trojan.api.TrojanServerService/GetUsers", opts...)
    if err != nil {
        return nil, err
    }
    // 创建 TrojanServerService_GetUsersClient 实例并返回
    x := &trojanServerServiceGetUsersClient{stream}
    return x, nil
}

// TrojanServerService_GetUsersClient 接口定义了与服务端通信的方法
type TrojanServerService_GetUsersClient interface {
    Send(*GetUsersRequest) error
    Recv() (*GetUsersResponse, error)
    grpc.ClientStream
}

// trojanServerServiceGetUsersClient 结构体实现了 TrojanServerService_GetUsersClient 接口
type trojanServerServiceGetUsersClient struct {
    grpc.ClientStream
}

// Send 方法用于向服务端发送 GetUsersRequest
func (x *trojanServerServiceGetUsersClient) Send(m *GetUsersRequest) error {
    return x.ClientStream.SendMsg(m)
}

// Recv 方法用于从服务端接收 GetUsersResponse
func (x *trojanServerServiceGetUsersClient) Recv() (*GetUsersResponse, error) {
    m := new(GetUsersResponse)
    if err := x.ClientStream.RecvMsg(m); err != nil {
        return nil, err
    }
    return m, nil
}

// SetUsers 方法用于获取 TrojanServerService_SetUsersClient 接口实例
func (c *trojanServerServiceClient) SetUsers(ctx context.Context, opts ...grpc.CallOption) (TrojanServerService_SetUsersClient, error) {
    // 创建新的流对象，用于与服务端进行通信
    stream, err := c.cc.NewStream(ctx, &TrojanServerService_ServiceDesc.Streams[2], "/trojan.api.TrojanServerService/SetUsers", opts...)
    if err != nil {
        return nil, err
    }
    // 创建 TrojanServerService_SetUsersClient 实例并返回
    x := &trojanServerServiceSetUsersClient{stream}
    return x, nil
}

// TrojanServerService_SetUsersClient 接口定义了与服务端通信的方法
type TrojanServerService_SetUsersClient interface {
    Send(*SetUsersRequest) error
    Recv() (*SetUsersResponse, error)
    grpc.ClientStream
}

// trojanServerServiceSetUsersClient 结构体实现了 TrojanServerService_SetUsersClient 接口
type trojanServerServiceSetUsersClient struct {
    grpc.ClientStream
}

// Send 方法用于向服务端发送 SetUsersRequest
func (x *trojanServerServiceSetUsersClient) Send(m *SetUsersRequest) error {
    return x.ClientStream.SendMsg(m)
}

// Recv 方法用于从服务端接收 SetUsersResponse
func (x *trojanServerServiceSetUsersClient) Recv() (*SetUsersResponse, error) {
    m := new(SetUsersResponse)
    if err := x.ClientStream.RecvMsg(m); err != nil {
        return nil, err
    }
    return m, nil
}

// TrojanServerServiceServer 是 TrojanServerService 服务的服务器 API
// 所有实现都必须嵌入UnimplementedTrojanServerServiceServer以实现向前兼容
type TrojanServerServiceServer interface {
    // 列出所有用户
    ListUsers(*ListUsersRequest, TrojanServerService_ListUsersServer) error
    // 获取指定用户的信息
    GetUsers(TrojanServerService_GetUsersServer) error
    // 设置现有用户的配置
    SetUsers(TrojanServerService_SetUsersServer) error
    mustEmbedUnimplementedTrojanServerServiceServer()
}

// UnimplementedTrojanServerServiceServer必须被嵌入以实现向前兼容
type UnimplementedTrojanServerServiceServer struct {
}

func (UnimplementedTrojanServerServiceServer) ListUsers(*ListUsersRequest, TrojanServerService_ListUsersServer) error {
    return status.Errorf(codes.Unimplemented, "method ListUsers not implemented")
}
func (UnimplementedTrojanServerServiceServer) GetUsers(TrojanServerService_GetUsersServer) error {
    return status.Errorf(codes.Unimplemented, "method GetUsers not implemented")
}
func (UnimplementedTrojanServerServiceServer) SetUsers(TrojanServerService_SetUsersServer) error {
    return status.Errorf(codes.Unimplemented, "method SetUsers not implemented")
}
func (UnimplementedTrojanServerServiceServer) mustEmbedUnimplementedTrojanServerServiceServer() {}

// UnsafeTrojanServerServiceServer可以被嵌入以选择退出此服务的向前兼容性
// 不建议使用此接口，因为向TrojanServerServiceServer添加方法将导致编译错误
type UnsafeTrojanServerServiceServer interface {
    mustEmbedUnimplementedTrojanServerServiceServer()
}

func RegisterTrojanServerServiceServer(s grpc.ServiceRegistrar, srv TrojanServerServiceServer) {
    s.RegisterService(&TrojanServerService_ServiceDesc, srv)
}

func _TrojanServerService_ListUsers_Handler(srv interface{}, stream grpc.ServerStream) error {
    m := new(ListUsersRequest)
    # 如果接收消息时发生错误，返回错误
    if err := stream.RecvMsg(m); err != nil:
        return err
    # 调用 TrojanServerServiceServer 接口的 ListUsers 方法，并返回结果
    return srv.(TrojanServerServiceServer).ListUsers(m, &trojanServerServiceListUsersServer{stream})
// TrojanServerService_ListUsersServer 是 TrojanServerService 服务的 ListUsersServer 接口
type TrojanServerService_ListUsersServer interface {
    // 发送 ListUsersResponse 到客户端
    Send(*ListUsersResponse) error
    // grpc.ServerStream 接口
    grpc.ServerStream
}

// trojanServerServiceListUsersServer 是 TrojanServerService_ListUsersServer 的实现
type trojanServerServiceListUsersServer struct {
    // grpc.ServerStream 接口
    grpc.ServerStream
}

// Send 方法实现了 TrojanServerService_ListUsersServer 接口的 Send 方法
func (x *trojanServerServiceListUsersServer) Send(m *ListUsersResponse) error {
    return x.ServerStream.SendMsg(m)
}

// _TrojanServerService_GetUsers_Handler 是 GetUsers 方法的处理函数
func _TrojanServerService_GetUsers_Handler(srv interface{}, stream grpc.ServerStream) error {
    return srv.(TrojanServerServiceServer).GetUsers(&trojanServerServiceGetUsersServer{stream})
}

// TrojanServerService_GetUsersServer 是 TrojanServerService 服务的 GetUsersServer 接口
type TrojanServerService_GetUsersServer interface {
    // 发送 GetUsersResponse 到客户端
    Send(*GetUsersResponse) error
    // 接收 GetUsersRequest 从客户端
    Recv() (*GetUsersRequest, error)
    // grpc.ServerStream 接口
    grpc.ServerStream
}

// trojanServerServiceGetUsersServer 是 TrojanServerService_GetUsersServer 的实现
type trojanServerServiceGetUsersServer struct {
    // grpc.ServerStream 接口
    grpc.ServerStream
}

// Send 方法实现了 TrojanServerService_GetUsersServer 接口的 Send 方法
func (x *trojanServerServiceGetUsersServer) Send(m *GetUsersResponse) error {
    return x.ServerStream.SendMsg(m)
}

// Recv 方法实现了 TrojanServerService_GetUsersServer 接口的 Recv 方法
func (x *trojanServerServiceGetUsersServer) Recv() (*GetUsersRequest, error) {
    m := new(GetUsersRequest)
    if err := x.ServerStream.RecvMsg(m); err != nil {
        return nil, err
    }
    return m, nil
}

// _TrojanServerService_SetUsers_Handler 是 SetUsers 方法的处理函数
func _TrojanServerService_SetUsers_Handler(srv interface{}, stream grpc.ServerStream) error {
    return srv.(TrojanServerServiceServer).SetUsers(&trojanServerServiceSetUsersServer{stream})
}

// TrojanServerService_SetUsersServer 是 TrojanServerService 服务的 SetUsersServer 接口
type TrojanServerService_SetUsersServer interface {
    // 发送 SetUsersResponse 到客户端
    Send(*SetUsersResponse) error
    // 接收 SetUsersRequest 从客户端
    Recv() (*SetUsersRequest, error)
    // grpc.ServerStream 接口
    grpc.ServerStream
}

// trojanServerServiceSetUsersServer 是 TrojanServerService_SetUsersServer 的实现
type trojanServerServiceSetUsersServer struct {
    // grpc.ServerStream 接口
    grpc.ServerStream
}

// Send 方法实现了 TrojanServerService_SetUsersServer 接口的 Send 方法
func (x *trojanServerServiceSetUsersServer) Send(m *SetUsersResponse) error {
    return x.ServerStream.SendMsg(m)
}

// Recv 方法实现了 TrojanServerService_SetUsersServer 接口的 Recv 方法
func (x *trojanServerServiceSetUsersServer) Recv() (*SetUsersRequest, error) {
    m := new(SetUsersRequest)
    if err := x.ServerStream.RecvMsg(m); err != nil {
        return nil, err
    }
    return m, nil
}

// TrojanServerService_ServiceDesc 是 TrojanServerService 服务的 grpc.ServiceDesc
// 仅用于直接与 grpc.RegisterService 使用
// 定义一个不可被内省或修改的 gRPC 服务描述
var TrojanServerService_ServiceDesc = grpc.ServiceDesc{
    ServiceName: "trojan.api.TrojanServerService",  // 服务名称为 "trojan.api.TrojanServerService"
    HandlerType: (*TrojanServerServiceServer)(nil),  // 处理程序类型为 TrojanServerServiceServer
    Methods:     []grpc.MethodDesc{},  // 没有定义方法
    Streams: []grpc.StreamDesc{  // 定义流
        {
            StreamName:    "ListUsers",  // 流名称为 "ListUsers"
            Handler:       _TrojanServerService_ListUsers_Handler,  // 处理程序为 _TrojanServerService_ListUsers_Handler
            ServerStreams: true,  // 服务器端流为 true
        },
        {
            StreamName:    "GetUsers",  // 流名称为 "GetUsers"
            Handler:       _TrojanServerService_GetUsers_Handler,  // 处理程序为 _TrojanServerService_GetUsers_Handler
            ServerStreams: true,  // 服务器端流为 true
            ClientStreams: true,  // 客户端流为 true
        },
        {
            StreamName:    "SetUsers",  // 流名称为 "SetUsers"
            Handler:       _TrojanServerService_SetUsers_Handler,  // 处理程序为 _TrojanServerService_SetUsers_Handler
            ServerStreams: true,  // 服务器端流为 true
            ClientStreams: true,  // 客户端流为 true
        },
    },
    Metadata: "api.proto",  // 元数据为 "api.proto"
}
```