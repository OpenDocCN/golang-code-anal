# `v2ray-core\app\log\command\command.go`

```go
// +build !confonly  // 标记此文件不仅仅是配置文件

package command  // 声明包名为 command

//go:generate go run v2ray.com/core/common/errors/errorgen  // 使用go:generate命令生成错误处理代码

import (  // 导入所需的包
    "context"  // 上下文包

    grpc "google.golang.org/grpc"  // gRPC包

    "v2ray.com/core"  // V2Ray核心包
    "v2ray.com/core/app/log"  // 日志包
    "v2ray.com/core/common"  // 公共包
)

type LoggerServer struct {  // 定义LoggerServer结构体
    V *core.Instance  // 包含V字段，类型为*core.Instance
}

// RestartLogger implements LoggerService.  // 实现LoggerService的RestartLogger方法
func (s *LoggerServer) RestartLogger(ctx context.Context, request *RestartLoggerRequest) (*RestartLoggerResponse, error) {
    logger := s.V.GetFeature((*log.Instance)(nil))  // 获取日志实例
    if logger == nil {  // 如果日志实例为空
        return nil, newError("unable to get logger instance")  // 返回错误信息
    }
    if err := logger.Close(); err != nil {  // 如果关闭日志实例出错
        return nil, newError("failed to close logger").Base(err)  // 返回错误信息
    }
    if err := logger.Start(); err != nil {  // 如果启动日志实例出错
        return nil, newError("failed to start logger").Base(err)  // 返回错误信息
    }
    return &RestartLoggerResponse{}, nil  // 返回重启日志响应
}

func (s *LoggerServer) mustEmbedUnimplementedLoggerServiceServer() {}  // 实现未实现的LoggerServiceServer方法

type service struct {  // 定义service结构体
    v *core.Instance  // 包含v字段，类型为*core.Instance
}

func (s *service) Register(server *grpc.Server) {  // 注册方法
    RegisterLoggerServiceServer(server, &LoggerServer{  // 注册LoggerServiceServer
        V: s.v,  // 设置V字段为s.v
    })
}

func init() {  // 初始化方法
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, cfg interface{}) (interface{}, error) {  // 注册配置
        s := core.MustFromContext(ctx)  // 从上下文中获取核心实例
        return &service{v: s}, nil  // 返回service实例
    }))
}
```