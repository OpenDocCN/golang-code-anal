# `kubo\config\migration.go`

```go
package config

// 默认迁移保留方式为"cache"
const DefaultMigrationKeep = "cache"

// 默认迁移下载来源为HTTPS和IPFS
var DefaultMigrationDownloadSources = []string{"HTTPS", "IPFS"}

// 迁移配置了如何下载迁移以及下载后是否将其添加到本地IPFS
type Migration struct {
    // 下载来源的优先顺序，"IPFS"表示使用IPFS，"HTTPS"表示使用默认网关。任何其他值都被解释为自定义网关的主机名。空列表表示"使用默认来源"。
    DownloadSources []string
    // 是否在下载后保留迁移
    // 选项有"discard"、"cache"、"pin"。空字符串表示默认值。
    Keep string
}
```