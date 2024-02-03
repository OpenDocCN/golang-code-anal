# `v2ray-core\proto.go`

```go
// 使用go:generate指令来运行命令，安装google.golang.org/protobuf/cmd/protoc-gen-go
// 生成protobuf的go插件
// 使用go:generate指令来运行命令，安装google.golang.org/grpc/cmd/protoc-gen-go-grpc
// 生成grpc的go插件
// 使用go:generate指令来运行命令，安装github.com/gogo/protobuf/protoc-gen-gofast
// 生成gofast的protobuf插件
// 使用go:generate指令来运行命令，运行infra/vprotogen/main.go文件
// 生成protobuf文件

import "path/filepath"

// ProtoFilesUsingProtocGenGoFast是使用`protoc-gen-gofast`生成pb.go文件的Proto文件的映射
var ProtoFilesUsingProtocGenGoFast = map[string]bool{"proxy/vless/encoding/addons.proto": true}

// ProtocMap是特定平台的`protoc`二进制可执行文件路径的映射
var ProtocMap = map[string]string{
    "windows": filepath.Join(".dev", "protoc", "windows", "protoc.exe"),
    "darwin":  filepath.Join(".dev", "protoc", "macos", "protoc"),
    "linux":   filepath.Join(".dev", "protoc", "linux", "protoc"),
}
```