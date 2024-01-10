# `v2ray-core\transport\internet\sockopt_other.go`

```
// +build js dragonfly netbsd openbsd solaris
// 标记这个文件只在特定的操作系统上构建

package internet
// 定义 internet 包

func applyOutboundSocketOptions(network string, address string, fd uintptr, config *SocketConfig) error {
    // 应用出站套接字选项
    return nil
}

func applyInboundSocketOptions(network string, fd uintptr, config *SocketConfig) error {
    // 应用入站套接字选项
    return nil
}

func bindAddr(fd uintptr, ip []byte, port uint32) error {
    // 绑定地址
    return nil
}

func setReuseAddr(fd uintptr) error {
    // 设置地址重用
    return nil
}

func setReusePort(fd uintptr) error {
    // 设置端口重用
    return nil
}
```