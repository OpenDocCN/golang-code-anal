# `v2ray-core\mocks.go`

```go
// 生成 mock 文件，用于模拟测试，指定包名为 mocks，目标文件为 io.go，模拟 Reader 和 Writer 接口
// 使用 go:generate 指令，可以在代码中直接运行命令
// 通过 mockgen 工具生成模拟文件，模拟 v2ray.com/core/common/log 包中的 Handler 接口
// 生成模拟文件，模拟 v2ray.com/core/common/mux 包中的 ClientWorkerFactory 接口
// 生成模拟文件，模拟 v2ray.com/core/features/dns 包中的 Client 接口
// 生成模拟文件，模拟 v2ray.com/core/features/outbound 包中的 Manager 和 HandlerSelector 接口
// 生成模拟文件，模拟 v2ray.com/core/proxy 包中的 Inbound 和 Outbound 接口
```