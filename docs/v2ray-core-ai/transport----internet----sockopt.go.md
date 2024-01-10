# `v2ray-core\transport\internet\sockopt.go`

```
# 判断给定的网络类型是否为 TCP 类型的套接字
func isTCPSocket(network string) bool:
    # 使用 switch 语句判断网络类型
    switch network:
        # 如果是 "tcp", "tcp4", "tcp6" 中的一种，则返回 true
        case "tcp", "tcp4", "tcp6":
            return true
        # 如果不是上述类型，则返回 false
        default:
            return false

# 判断给定的网络类型是否为 UDP 类型的套接字
func isUDPSocket(network string) bool:
    # 使用 switch 语句判断网络类型
    switch network:
        # 如果是 "udp", "udp4", "udp6" 中的一种，则返回 true
        case "udp", "udp4", "udp6":
            return true
        # 如果不是上述类型，则返回 false
        default:
            return false
```