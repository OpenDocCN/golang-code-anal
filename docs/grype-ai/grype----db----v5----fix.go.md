# `grype\grype\db\v5\fix.go`

```
// 定义 FixState 类型为字符串
type FixState string

// 定义 FixState 常量，分别表示未知、已修复、未修复、不会修复状态
const (
    UnknownFixState FixState = "unknown"
    FixedState      FixState = "fixed"
    NotFixedState   FixState = "not-fixed"
    WontFixState    FixState = "wont-fix"
)

// Fix 结构体表示已知漏洞修复的所有信息
type Fix struct {
    Versions []string `json:"versions"` // 特定漏洞修复的版本
    State    FixState `json:"state"` // 修复状态
}
```