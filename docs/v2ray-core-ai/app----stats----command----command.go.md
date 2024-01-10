# `v2ray-core\app\stats\command\command.go`

```
// +build !confonly
// 定义包名为 command

package command

//go:generate go run v2ray.com/core/common/errors/errorgen
// 使用 errorgen 工具生成错误处理代码

import (
    "context"
    "runtime"
    "time"

    grpc "google.golang.org/grpc"

    "v2ray.com/core"
    "v2ray.com/core/app/stats"
    "v2ray.com/core/common"
    "v2ray.com/core/common/strmatcher"
    feature_stats "v2ray.com/core/features/stats"
)

// statsServer is an implementation of StatsService.
// statsServer 是 StatsService 的实现
type statsServer struct {
    stats     feature_stats.Manager
    startTime time.Time
}

// NewStatsServer creates a new StatsServer with the given feature_stats.Manager.
// 使用给定的 feature_stats.Manager 创建一个新的 StatsServer
func NewStatsServer(manager feature_stats.Manager) StatsServiceServer {
    return &statsServer{
        stats:     manager,
        startTime: time.Now(),
    }
}

// GetStats retrieves the stats with the given name from the server.
// 从服务器中检索具有给定名称的统计信息
func (s *statsServer) GetStats(ctx context.Context, request *GetStatsRequest) (*GetStatsResponse, error) {
    c := s.stats.GetCounter(request.Name)
    if c == nil {
        return nil, newError(request.Name, " not found.")
    }
    var value int64
    if request.Reset_ {
        value = c.Set(0)
    } else {
        value = c.Value()
    }
    return &GetStatsResponse{
        Stat: &Stat{
            Name:  request.Name,
            Value: value,
        },
    }, nil
}

// QueryStats retrieves the stats that match the given pattern from the server.
// 从服务器中检索与给定模式匹配的统计信息
func (s *statsServer) QueryStats(ctx context.Context, request *QueryStatsRequest) (*QueryStatsResponse, error) {
    matcher, err := strmatcher.Substr.New(request.Pattern)
    if err != nil {
        return nil, err
    }

    response := &QueryStatsResponse{}

    manager, ok := s.stats.(*stats.Manager)
    if !ok {
        return nil, newError("QueryStats only works its own stats.Manager.")
    }

    manager.VisitCounters(func(name string, c feature_stats.Counter) bool {
        if matcher.Match(name) {
            var value int64
            if request.Reset_ {
                value = c.Set(0)
            } else {
                value = c.Value()
            }
            response.Stat = append(response.Stat, &Stat{
                Name:  name,
                Value: value,
            })
        }
        return true
    })
}
    # 返回 response 和 nil
    return response, nil
// GetSysStats 方法用于获取系统统计信息，接收上下文和 SysStatsRequest 对象作为参数，返回 SysStatsResponse 和错误信息
func (s *statsServer) GetSysStats(ctx context.Context, request *SysStatsRequest) (*SysStatsResponse, error) {
    // 声明一个 runtime.MemStats 变量
    var rtm runtime.MemStats
    // 读取内存统计信息并存储到 rtm 变量中
    runtime.ReadMemStats(&rtm)

    // 计算系统启动时间
    uptime := time.Since(s.startTime)

    // 创建 SysStatsResponse 对象并填充统计信息
    response := &SysStatsResponse{
        Uptime:       uint32(uptime.Seconds()),
        NumGoroutine: uint32(runtime.NumGoroutine()),
        Alloc:        rtm.Alloc,
        TotalAlloc:   rtm.TotalAlloc,
        Sys:          rtm.Sys,
        Mallocs:      rtm.Mallocs,
        Frees:        rtm.Frees,
        LiveObjects:  rtm.Mallocs - rtm.Frees,
        NumGC:        rtm.NumGC,
        PauseTotalNs: rtm.PauseTotalNs,
    }

    // 返回 SysStatsResponse 对象和空的错误信息
    return response, nil
}

// mustEmbedUnimplementedStatsServiceServer 方法用于嵌入未实现的 StatsServiceServer 接口
func (s *statsServer) mustEmbedUnimplementedStatsServiceServer() {}

// service 结构体包含 statsManager 属性
type service struct {
    statsManager feature_stats.Manager
}

// Register 方法用于注册服务
func (s *service) Register(server *grpc.Server) {
    // 在 gRPC 服务器上注册 StatsServiceServer
    RegisterStatsServiceServer(server, NewStatsServer(s.statsManager))
}

// init 函数用于初始化服务
func init() {
    // 注册配置并返回服务对象
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, cfg interface{}) (interface{}, error) {
        s := new(service)

        // 获取特性管理器并赋值给 statsManager 属性
        core.RequireFeatures(ctx, func(sm feature_stats.Manager) {
            s.statsManager = sm
        })

        // 返回服务对象和空的错误信息
        return s, nil
    }))
}
```