# `grype\grype\db\status.go`

```
package db
// 导入 time 包

import "time"

// 定义 Status 结构体
type Status struct {
	// 内建时间类型，表示构建时间
	Built         time.Time `json:"built"`
	// 整型，表示数据库模式版本
	SchemaVersion int       `json:"schemaVersion"`
	// 字符串，表示数据库位置
	Location      string    `json:"location"`
	// 字符串，表示数据库校验和
	Checksum      string    `json:"checksum"`
	// 错误类型，表示数据库操作错误
	Err           error     `json:"error"`
}
```