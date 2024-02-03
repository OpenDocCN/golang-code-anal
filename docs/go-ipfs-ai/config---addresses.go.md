# `kubo\config\addresses.go`

```go
// Addresses 结构体存储节点的（字符串）多地址地址。
type Addresses struct {
    Swarm          []string // 用于监听的 swarm 地址
    Announce       []string // 要向网络宣布的 swarm 地址，如果长度 > 0，则替换自动检测到的地址
    AppendAnnounce []string // 类似于 Announce，但不会覆盖自动检测到的地址，它们只是被追加
    NoAnnounce     []string // 不要向网络宣布的 swarm 地址
    API            Strings  // 本地 API（RPC）的地址
    Gateway        Strings  // 用于 IPFS HTTP 对象网关的地址
}
```