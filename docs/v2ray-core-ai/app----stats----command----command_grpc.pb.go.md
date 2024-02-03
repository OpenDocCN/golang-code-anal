# `v2ray-core\app\stats\command\command_grpc.pb.go`

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

// StatsServiceClient 是 StatsService 服务的客户端 API。
//
// 有关 ctx 使用和关闭/结束流式 RPC 的语义，请参阅 https://pkg.go.dev/google.golang.org/grpc/?tab=doc#ClientConn.NewStream。
type StatsServiceClient interface {
    GetStats(ctx context.Context, in *GetStatsRequest, opts ...grpc.CallOption) (*GetStatsResponse, error)  // 获取统计信息
    QueryStats(ctx context.Context, in *QueryStatsRequest, opts ...grpc.CallOption) (*QueryStatsResponse, error)  // 查询统计信息
    GetSysStats(ctx context.Context, in *SysStatsRequest, opts ...grpc.CallOption) (*SysStatsResponse, error)  // 获取系统统计信息
}

type statsServiceClient struct {
    cc grpc.ClientConnInterface
}

func NewStatsServiceClient(cc grpc.ClientConnInterface) StatsServiceClient {
    return &statsServiceClient{cc}  // 创建 StatsServiceClient 实例
}

func (c *statsServiceClient) GetStats(ctx context.Context, in *GetStatsRequest, opts ...grpc.CallOption) (*GetStatsResponse, error) {
    out := new(GetStatsResponse)  // 创建 GetStatsResponse 实例
    err := c.cc.Invoke(ctx, "/v2ray.core.app.stats.command.StatsService/GetStats", in, out, opts...)  // 调用远程服务
    if err != nil {
        return nil, err  // 如果出错，返回 nil 和错误信息
    }
    return out, nil  // 返回结果和 nil
}

func (c *statsServiceClient) QueryStats(ctx context.Context, in *QueryStatsRequest, opts ...grpc.CallOption) (*QueryStatsResponse, error) {
    out := new(QueryStatsResponse)  // 创建 QueryStatsResponse 实例
    err := c.cc.Invoke(ctx, "/v2ray.core.app.stats.command.StatsService/QueryStats", in, out, opts...)  // 调用远程服务
    if err != nil {
        return nil, err  // 如果出错，返回 nil 和错误信息
    }
    return out, nil  // 返回结果和 nil
}
// GetSysStats 是 statsServiceClient 结构体的方法，用于获取系统统计信息
func (c *statsServiceClient) GetSysStats(ctx context.Context, in *SysStatsRequest, opts ...grpc.CallOption) (*SysStatsResponse, error) {
    // 创建一个 SysStatsResponse 对象
    out := new(SysStatsResponse)
    // 调用 gRPC 客户端的 Invoke 方法，发送 GetSysStats 请求，并将结果存储在 out 中
    err := c.cc.Invoke(ctx, "/v2ray.core.app.stats.command.StatsService/GetSysStats", in, out, opts...)
    // 如果出现错误，则返回 nil 和错误信息
    if err != nil {
        return nil, err
    }
    // 返回获取的系统统计信息
    return out, nil
}

// StatsServiceServer 是 StatsService 服务的服务器 API 接口
// 所有实现都必须嵌入 UnimplementedStatsServiceServer 以实现向前兼容
type StatsServiceServer interface {
    // GetStats 方法用于获取统计信息
    GetStats(context.Context, *GetStatsRequest) (*GetStatsResponse, error)
    // QueryStats 方法用于查询统计信息
    QueryStats(context.Context, *QueryStatsRequest) (*QueryStatsResponse, error)
    // GetSysStats 方法用于获取系统统计信息
    GetSysStats(context.Context, *SysStatsRequest) (*SysStatsResponse, error)
    // mustEmbedUnimplementedStatsServiceServer 方法用于嵌入未实现的 StatsServiceServer 接口
    mustEmbedUnimplementedStatsServiceServer()
}

// UnimplementedStatsServiceServer 必须被嵌入以实现向前兼容的未实现的 StatsServiceServer
type UnimplementedStatsServiceServer struct {
}

// GetStats 方法返回未实现的错误信息
func (UnimplementedStatsServiceServer) GetStats(context.Context, *GetStatsRequest) (*GetStatsResponse, error) {
    return nil, status.Errorf(codes.Unimplemented, "method GetStats not implemented")
}

// QueryStats 方法返回未实现的错误信息
func (UnimplementedStatsServiceServer) QueryStats(context.Context, *QueryStatsRequest) (*QueryStatsResponse, error) {
    return nil, status.Errorf(codes.Unimplemented, "method QueryStats not implemented")
}

// GetSysStats 方法返回未实现的错误信息
func (UnimplementedStatsServiceServer) GetSysStats(context.Context, *SysStatsRequest) (*SysStatsResponse, error) {
    return nil, status.Errorf(codes.Unimplemented, "method GetSysStats not implemented")
}

// mustEmbedUnimplementedStatsServiceServer 方法用于嵌入未实现的 StatsServiceServer 接口
func (UnimplementedStatsServiceServer) mustEmbedUnimplementedStatsServiceServer() {}

// UnsafeStatsServiceServer 可以被嵌入以选择不进行服务的向前兼容
// 不建议使用此接口，因为向 StatsServiceServer 添加方法将导致编译错误
type UnsafeStatsServiceServer interface {
    // mustEmbedUnimplementedStatsServiceServer 方法用于嵌入未实现的 StatsServiceServer 接口
    mustEmbedUnimplementedStatsServiceServer()
}
# 注册统计服务的 gRPC 服务器
func RegisterStatsServiceServer(s *grpc.Server, srv StatsServiceServer) {
    # 注册服务描述和对应的服务实现
    s.RegisterService(&_StatsService_serviceDesc, srv)
}

# 处理 GetStats 请求的函数
func _StatsService_GetStats_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    # 创建 GetStatsRequest 对象
    in := new(GetStatsRequest)
    # 解码请求数据
    if err := dec(in); err != nil {
        return nil, err
    }
    # 如果没有拦截器，则直接调用服务实现的 GetStats 方法
    if interceptor == nil {
        return srv.(StatsServiceServer).GetStats(ctx, in)
    }
    # 创建 gRPC 一元服务器信息对象
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/v2ray.core.app.stats.command.StatsService/GetStats",
    }
    # 创建处理函数，调用服务实现的 GetStats 方法
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(StatsServiceServer).GetStats(ctx, req.(*GetStatsRequest))
    }
    # 调用拦截器处理请求
    return interceptor(ctx, in, info, handler)
}

# 处理 QueryStats 请求的函数
func _StatsService_QueryStats_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    # 创建 QueryStatsRequest 对象
    in := new(QueryStatsRequest)
    # 解码请求数据
    if err := dec(in); err != nil {
        return nil, err
    }
    # 如果没有拦截器，则直接调用服务实现的 QueryStats 方法
    if interceptor == nil {
        return srv.(StatsServiceServer).QueryStats(ctx, in)
    }
    # 创建 gRPC 一元服务器信息对象
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/v2ray.core.app.stats.command.StatsService/QueryStats",
    }
    # 创建处理函数，调用服务实现的 QueryStats 方法
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(StatsServiceServer).QueryStats(ctx, req.(*QueryStatsRequest))
    }
    # 调用拦截器处理请求
    return interceptor(ctx, in, info, handler)
}

# 处理 GetSysStats 请求的函数
func _StatsService_GetSysStats_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    # 创建 SysStatsRequest 对象
    in := new(SysStatsRequest)
    # 解码请求数据
    if err := dec(in); err != nil {
        return nil, err
    }
    # 如果没有拦截器，则直接调用服务实现的 GetSysStats 方法
    if interceptor == nil {
        return srv.(StatsServiceServer).GetSysStats(ctx, in)
    }
    # 创建一个包含服务器信息的结构体指针
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/v2ray.core.app.stats.command.StatsService/GetSysStats",
    }
    # 创建一个处理函数，接收上下文和请求参数，返回结果和错误
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(StatsServiceServer).GetSysStats(ctx, req.(*SysStatsRequest))
    }
    # 调用拦截器函数，传入上下文、输入参数、服务器信息和处理函数
    return interceptor(ctx, in, info, handler)
# 定义 gRPC 服务的描述信息
var _StatsService_serviceDesc = grpc.ServiceDesc{
    # 设置服务名称
    ServiceName: "v2ray.core.app.stats.command.StatsService",
    # 设置处理程序类型
    HandlerType: (*StatsServiceServer)(nil),
    # 设置方法描述列表
    Methods: []grpc.MethodDesc{
        {
            # 设置方法名
            MethodName: "GetStats",
            # 设置处理程序
            Handler:    _StatsService_GetStats_Handler,
        },
        {
            # 设置方法名
            MethodName: "QueryStats",
            # 设置处理程序
            Handler:    _StatsService_QueryStats_Handler,
        },
        {
            # 设置方法名
            MethodName: "GetSysStats",
            # 设置处理程序
            Handler:    _StatsService_GetSysStats_Handler,
        },
    },
    # 设置流描述列表
    Streams:  []grpc.StreamDesc{},
    # 设置元数据
    Metadata: "app/stats/command/command.proto",
}
```