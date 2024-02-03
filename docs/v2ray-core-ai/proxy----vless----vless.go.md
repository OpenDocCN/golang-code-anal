# `v2ray-core\proxy\vless\vless.go`

```go
// Package vless 包含 VLess 协议和传输的实现。
//
// VLess 包含入站和出站连接。VLess 入站通常与 'freedom' 一起在服务器上使用，用于与最终目的地通信，而 VLess 出站通常与 'socks' 一起在客户端上使用进行代理。
package vless

//go:generate go run v2ray.com/core/common/errors/errorgen

// 定义常量
const (
    XRO = "xtls-rprx-origin"  // XRO 常量表示 xtls-rprx-origin
    XRD = "xtls-rprx-direct"  // XRD 常量表示 xtls-rprx-direct
)
```