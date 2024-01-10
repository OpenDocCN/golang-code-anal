# `v2ray-core\app\commander\service.go`

```
// +build !confonly
// 设置构建标记，表示该代码块不仅仅是配置
package commander

import (
    "google.golang.org/grpc"
)

// Service is a Commander service.
// 定义一个接口类型 Service，表示一个 Commander 服务
type Service interface {
    // Register registers the service itself to a gRPC server.
    // 定义接口方法 Register，用于将服务注册到 gRPC 服务器
    Register(*grpc.Server)
}
```