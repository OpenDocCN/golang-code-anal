# `v2ray-core\transport\internet\kcp\kcp.go`

```
// 包 kcp - 一个快速可靠的 ARQ 协议
//
// 致谢：
//    skywind3000@github 发明了 KCP 协议
//    xtaci@github 将其翻译成了 Golang
package kcp

// 使用 go:generate 命令来运行 errorgen 工具生成错误处理代码
//go:generate go run v2ray.com/core/common/errors/errorgen
```