# `kubo\config\mounts.go`

```go
// 定义一个名为 Mounts 的结构体，用于存储字符串类型的挂载点信息
type Mounts struct {
    IPFS           string        // IPFS 挂载点
    IPNS           string        // IPNS 挂载点
    FuseAllowOther bool          // 是否允许其他用户挂载
}
```