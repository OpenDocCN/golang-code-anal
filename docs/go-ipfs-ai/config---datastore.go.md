# `kubo\config\datastore.go`

```
package config

import (
    "encoding/json"
)

// DefaultDataStoreDirectory is the directory to store all the local IPFS data.
const DefaultDataStoreDirectory = "datastore"

// Datastore tracks the configuration of the datastore.
type Datastore struct {
    StorageMax         string // in B, kB, kiB, MB, ...
    StorageGCWatermark int64  // in percentage to multiply on StorageMax
    GCPeriod           string // in ns, us, ms, s, m, h

    // deprecated fields, use Spec
    Type   string           `json:",omitempty"` // 数据存储类型，如果为空则忽略
    Path   string           `json:",omitempty"` // 数据存储路径，如果为空则忽略
    NoSync bool             `json:",omitempty"` // 是否同步写入，如果为空则忽略
    Params *json.RawMessage `json:",omitempty"` // 参数，如果为空则忽略

    Spec map[string]interface{} // 数据存储的特定配置

    HashOnRead      bool // 读取时是否进行哈希
    BloomFilterSize int  // 布隆过滤器的大小
}

// DataStorePath returns the default data store path given a configuration root
// (set an empty string to have the default configuration root).
func DataStorePath(configroot string) (string, error) {
    return Path(configroot, DefaultDataStoreDirectory) // 返回默认数据存储路径
}
```