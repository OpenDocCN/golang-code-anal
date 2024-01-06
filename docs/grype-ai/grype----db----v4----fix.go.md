# `grype\grype\db\v4\fix.go`

```
// 定义 FixState 类型为字符串
type FixState string

// 定义 FixState 常量
const (
	UnknownFixState FixState = "unknown" // 未知状态
	FixedState      FixState = "fixed" // 已修复状态
	NotFixedState   FixState = "not-fixed" // 未修复状态
	WontFixState    FixState = "wont-fix" // 不会修复状态
)

// Fix 结构体表示已知漏洞修复的所有信息
type Fix struct {
	Versions []string `json:"versions"` // 特定漏洞修复的版本
	State    FixState `json:"state"` // 修复状态
}
```