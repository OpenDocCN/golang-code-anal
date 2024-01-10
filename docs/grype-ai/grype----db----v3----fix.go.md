# `grype\grype\db\v3\fix.go`

```
// 定义一个名为 FixState 的自定义类型
type FixState string

// 定义 FixState 类型的常量
const (
    UnknownFixState FixState = "unknown" // 未知状态
    FixedState      FixState = "fixed"   // 已修复状态
    NotFixedState   FixState = "not-fixed" // 未修复状态
    WontFixState    FixState = "wont-fix" // 不会修复状态
)

// Fix 结构体表示已知漏洞修复的所有信息
type Fix struct {
    Versions []string // 该漏洞修复的版本
    State    FixState // 修复状态
}
```