# `v2ray-core\proxy\vmess\vmess.go`

```
// Package vmess 包含了 VMess 协议和传输的实现。
//
// VMess 包含了入站和出站连接。VMess 入站通常与 'freedom' 一起在服务器上使用，用于与最终目的地通信，而 VMess 出站通常与 'socks' 一起在客户端上使用，用于代理。
package vmess

//go:generate go run v2ray.com/core/common/errors/errorgen
```