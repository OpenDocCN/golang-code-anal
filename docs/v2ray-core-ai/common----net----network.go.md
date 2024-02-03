# `v2ray-core\common\net\network.go`

```go
// SystemString 方法返回网络类型的字符串表示
func (n Network) SystemString() string {
    // 使用 switch 语句根据不同的网络类型返回对应的字符串
    switch n {
    case Network_TCP:
        return "tcp"
    case Network_UDP:
        return "udp"
    default:
        return "unknown"
    }
}

// HasNetwork 方法检查网络列表中是否存在特定的网络类型
func HasNetwork(list []Network, network Network) bool {
    // 使用 for 循环遍历网络列表，如果找到指定的网络类型则返回 true
    for _, value := range list {
        if value == network {
            return true
        }
    }
    // 如果循环结束仍未找到指定的网络类型，则返回 false
    return false
}
```