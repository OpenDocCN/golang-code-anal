# `grype\grype\db\v5\id.go`

```
// 定义包名为 v5
package v5

// 导入 time 包
import (
    "time"
)

// ID 表示数据库及其包含数据的标识信息
type ID struct {
    // BuildTimestamp 是用于定义数据库年龄的时间戳，理想情况下包括数据的年龄，而不仅仅是数据库文件创建的时间
    BuildTimestamp time.Time `json:"build_timestamp"`
    SchemaVersion  int       `json:"schema_version"`
}

// IDReader 接口定义了获取 ID 的方法
type IDReader interface {
    GetID() (*ID, error)
}

// IDWriter 接口定义了设置 ID 的方法
type IDWriter interface {
    SetID(ID) error
}

// NewID 函数用于创建一个新的 ID 对象
func NewID(age time.Time) ID {
    return ID{
        BuildTimestamp: age.UTC(),
        SchemaVersion:  SchemaVersion,
    }
}
```