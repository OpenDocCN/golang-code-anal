# `grype\grype\db\status.go`

```go
# 导入时间包
import "time"

# 定义 Status 结构体
type Status struct {
    # 表示构建时间的字段
    Built         time.Time `json:"built"`
    # 表示数据库模式版本的字段
    SchemaVersion int       `json:"schemaVersion"`
    # 表示数据库位置的字段
    Location      string    `json:"location"`
    # 表示数据库校验和的字段
    Checksum      string    `json:"checksum"`
    # 表示错误信息的字段
    Err           error     `json:"error"`
}
```