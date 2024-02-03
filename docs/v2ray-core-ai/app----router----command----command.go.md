# `v2ray-core\app\router\command\command.go`

```go
// +build !confonly

package command

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
    "context"
    "time"

    "google.golang.org/grpc"

    "v2ray.com/core"
    "v2ray.com/core/common"
    "v2ray.com/core/features/routing"
    "v2ray.com/core/features/stats"
)

// routingServer is an implementation of RoutingService.
type routingServer struct {
    router       routing.Router
    routingStats stats.Channel
}

// NewRoutingServer creates a statistics service with statistics manager.
func NewRoutingServer(router routing.Router, routingStats stats.Channel) RoutingServiceServer {
    return &routingServer{
        router:       router,
        routingStats: routingStats,
    }
}

func (s *routingServer) TestRoute(ctx context.Context, request *TestRouteRequest) (*RoutingContext, error) {
    // 检查请求的路由上下文是否为空
    if request.RoutingContext == nil {
        return nil, newError("Invalid routing request.")
    }
    // 从路由器中选择路由
    route, err := s.router.PickRoute(AsRoutingContext(request.RoutingContext))
    if err != nil {
        return nil, err
    }
    // 如果请求发布结果并且路由统计不为空，则发布路由统计
    if request.PublishResult && s.routingStats != nil {
        ctx, _ := context.WithTimeout(context.Background(), 4*time.Second) // nolint: govet
        s.routingStats.Publish(ctx, route)
    }
    // 将路由转换为 Protobuf 消息并返回
    return AsProtobufMessage(request.FieldSelectors)(route), nil
}

func (s *routingServer) SubscribeRoutingStats(request *SubscribeRoutingStatsRequest, stream RoutingService_SubscribeRoutingStatsServer) error {
    // 如果路由统计为空，则返回错误
    if s.routingStats == nil {
        return newError("Routing statistics not enabled.")
    }
    // 将请求的字段选择器转换为消息
    genMessage := AsProtobufMessage(request.FieldSelectors)
    // 订阅可运行的路由统计通道
    subscriber, err := stats.SubscribeRunnableChannel(s.routingStats)
    if err != nil {
        return err
    }
    // 延迟取消订阅可关闭的路由统计通道
    defer stats.UnsubscribeClosableChannel(s.routingStats, subscriber) // nolint: errcheck
}
    # 无限循环，等待从subscriber通道接收数据
    for {
        # 从subscriber通道接收数据，并判断通道是否已关闭
        select {
        case value, ok := <-subscriber:
            # 如果通道已关闭，则返回错误信息
            if !ok {
                return newError("Upstream closed the subscriber channel.")
            }
            # 将接收到的数据转换为routing.Route类型，并判断是否转换成功
            route, ok := value.(routing.Route)
            # 如果转换失败，则返回错误信息
            if !ok {
                return newError("Upstream sent malformed statistics.")
            }
            # 将route转换为消息，并发送到stream中
            err := stream.Send(genMessage(route))
            # 如果发送过程中出现错误，则返回错误信息
            if err != nil {
                return err
            }
        # 监听stream的Context是否已经关闭
        case <-stream.Context().Done():
            # 如果Context已关闭，则返回Context的错误信息
            return stream.Context().Err()
        }
    }
# 定义一个方法，用于确保未实现的路由服务服务器被嵌入
func (s *routingServer) mustEmbedUnimplementedRoutingServiceServer() {}

# 定义一个结构体，包含一个核心实例指针
type service struct {
    v *core.Instance
}

# 定义一个方法，用于在 gRPC 服务器上注册服务
func (s *service) Register(server *grpc.Server) {
    # 调用核心实例的 RequireFeatures 方法，确保路由和统计特性已经准备就绪
    common.Must(s.v.RequireFeatures(func(router routing.Router, stats stats.Manager) {
        # 在 gRPC 服务器上注册路由服务
        RegisterRoutingServiceServer(server, NewRoutingServer(router, nil))
    }))
}

# 在包初始化时执行的方法
func init() {
    # 注册配置并返回服务实例
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, cfg interface{}) (interface{}, error) {
        # 从上下文中获取核心实例
        s := core.MustFromContext(ctx)
        # 返回包含核心实例指针的服务实例
        return &service{v: s}, nil
    }))
}
```