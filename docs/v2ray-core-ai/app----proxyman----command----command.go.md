# `v2ray-core\app\proxyman\command\command.go`

```
// +build !confonly

package command

import (
    "context"

    grpc "google.golang.org/grpc"

    "v2ray.com/core"
    "v2ray.com/core/common"
    "v2ray.com/core/features/inbound"
    "v2ray.com/core/features/outbound"
    "v2ray.com/core/proxy"
)

// InboundOperation is the interface for operations that applies to inbound handlers.
type InboundOperation interface {
    // ApplyInbound applies this operation to the given inbound handler.
    ApplyInbound(context.Context, inbound.Handler) error
}

// OutboundOperation is the interface for operations that applies to outbound handlers.
type OutboundOperation interface {
    // ApplyOutbound applies this operation to the given outbound handler.
    ApplyOutbound(context.Context, outbound.Handler) error
}

func getInbound(handler inbound.Handler) (proxy.Inbound, error) {
    gi, ok := handler.(proxy.GetInbound)
    if !ok {
        return nil, newError("can't get inbound proxy from handler.")
    }
    return gi.GetInbound(), nil
}

// ApplyInbound implements InboundOperation.
func (op *AddUserOperation) ApplyInbound(ctx context.Context, handler inbound.Handler) error {
    p, err := getInbound(handler)
    if err != nil {
        return err
    }
    um, ok := p.(proxy.UserManager)
    if !ok {
        return newError("proxy is not a UserManager")
    }
    mUser, err := op.User.ToMemoryUser()
    if err != nil {
        return newError("failed to parse user").Base(err)
    }
    return um.AddUser(ctx, mUser)
}

// ApplyInbound implements InboundOperation.
func (op *RemoveUserOperation) ApplyInbound(ctx context.Context, handler inbound.Handler) error {
    p, err := getInbound(handler)
    if err != nil {
        return err
    }
    um, ok := p.(proxy.UserManager)
    if !ok {
        return newError("proxy is not a UserManager")
    }
    return um.RemoveUser(ctx, op.Email)
}

type handlerServer struct {
    s   *core.Instance
    ihm inbound.Manager
    ohm outbound.Manager
}
# 添加入站处理程序的方法，接收上下文和添加入站请求作为参数，返回添加入站响应和错误
func (s *handlerServer) AddInbound(ctx context.Context, request *AddInboundRequest) (*AddInboundResponse, error) {
    # 调用核心包的添加入站处理程序方法，如果出现错误则返回空和错误
    if err := core.AddInboundHandler(s.s, request.Inbound); err != nil {
        return nil, err
    }
    # 返回添加入站响应
    return &AddInboundResponse{}, nil
}

# 移除入站处理程序的方法，接收上下文和移除入站请求作为参数，返回移除入站响应和错误
func (s *handlerServer) RemoveInbound(ctx context.Context, request *RemoveInboundRequest) (*RemoveInboundResponse, error) {
    # 返回移除入站响应和调用入站处理程序管理器的移除处理程序方法的结果
    return &RemoveInboundResponse{}, s.ihm.RemoveHandler(ctx, request.Tag)
}

# 修改入站处理程序的方法，接收上下文和修改入站请求作为参数，返回修改入站响应和错误
func (s *handlerServer) AlterInbound(ctx context.Context, request *AlterInboundRequest) (*AlterInboundResponse, error) {
    # 获取请求中的操作实例
    rawOperation, err := request.Operation.GetInstance()
    # 如果出现错误则返回空和包含错误信息的新错误
    if err != nil {
        return nil, newError("unknown operation").Base(err)
    }
    # 将操作实例转换为入站操作类型
    operation, ok := rawOperation.(InboundOperation)
    # 如果无法转换则返回空和包含错误信息的新错误
    if !ok {
        return nil, newError("not an inbound operation")
    }
    # 获取指定标签的处理程序
    handler, err := s.ihm.GetHandler(ctx, request.Tag)
    # 如果出现错误则返回空和包含错误信息的新错误
    if err != nil {
        return nil, newError("failed to get handler: ", request.Tag).Base(err)
    }
    # 返回修改入站响应和应用入站操作的结果
    return &AlterInboundResponse{}, operation.ApplyInbound(ctx, handler)
}

# 添加出站处理程序的方法，接收上下文和添加出站请求作为参数，返回添加出站响应和错误
func (s *handlerServer) AddOutbound(ctx context.Context, request *AddOutboundRequest) (*AddOutboundResponse, error) {
    # 调用核心包的添加出站处理程序方法，如果出现错误则返回空和错误
    if err := core.AddOutboundHandler(s.s, request.Outbound); err != nil {
        return nil, err
    }
    # 返回添加出站响应
    return &AddOutboundResponse{}, nil
}

# 移除出站处理程序的方法，接收上下文和移除出站请求作为参数，返回移除出站响应和错误
func (s *handlerServer) RemoveOutbound(ctx context.Context, request *RemoveOutboundRequest) (*RemoveOutboundResponse, error) {
    # 返回移除出站响应和调用出站处理程序管理器的移除处理程序方法的结果
    return &RemoveOutboundResponse{}, s.ohm.RemoveHandler(ctx, request.Tag)
}

# 修改出站处理程序的方法，接收上下文和修改出站请求作为参数，返回修改出站响应和错误
func (s *handlerServer) AlterOutbound(ctx context.Context, request *AlterOutboundRequest) (*AlterOutboundResponse, error) {
    # 获取请求中的操作实例
    rawOperation, err := request.Operation.GetInstance()
    # 如果出现错误则返回空和包含错误信息的新错误
    if err != nil {
        return nil, newError("unknown operation").Base(err)
    }
    # 将操作实例转换为出站操作类型
    operation, ok := rawOperation.(OutboundOperation)
    # 如果无法转换则返回空和包含错误信息的新错误
    if !ok {
        return nil, newError("not an outbound operation")
    }
    # 根据请求的标签获取对应的处理程序
    handler := s.ohm.GetHandler(request.Tag)
    # 返回一个空的AlterOutboundResponse指针，并将处理程序应用于出站操作
    return &AlterOutboundResponse{}, operation.ApplyOutbound(ctx, handler)
# 定义一个方法，用于实现未实现的HandlerServiceServer接口
func (s *handlerServer) mustEmbedUnimplementedHandlerServiceServer() {}

# 定义一个service结构体，包含一个core.Instance类型的指针
type service struct {
    v *core.Instance
}

# 注册方法，将handlerServer注册到grpc.Server上
func (s *service) Register(server *grpc.Server) {
    # 创建handlerServer对象，并将core.Instance的值赋给s.v
    hs := &handlerServer{
        s: s.v,
    }
    # 调用common.Must方法，确保s.v满足要求的特性
    common.Must(s.v.RequireFeatures(func(im inbound.Manager, om outbound.Manager) {
        # 将inbound.Manager和outbound.Manager赋值给handlerServer对象的ihm和ohm属性
        hs.ihm = im
        hs.ohm = om
    }))
    # 将handlerServer注册到grpc.Server上
    RegisterHandlerServiceServer(server, hs)
}

# 初始化方法，注册Config并返回service对象
func init() {
    # 调用common.RegisterConfig方法，注册Config并返回service对象
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, cfg interface{}) (interface{}, error) {
        # 从上下文中获取core.Instance对象
        s := core.MustFromContext(ctx)
        # 返回包含core.Instance对象的service对象
        return &service{v: s}, nil
    }))
}
```