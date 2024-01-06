# `grype\grype\event\monitor\matching.go`

```
// 导入 monitor 包
package monitor

// 导入 go-progress 包中的进度条和监控器
import (
	"github.com/wagoodman/go-progress"

	"github.com/anchore/grype/grype/vulnerability"
)

// 定义 Matching 结构体，用于存储匹配信息
type Matching struct {
	// PackagesProcessed 用于跟踪已处理的软件包数量
	PackagesProcessed progress.Progressable
	// MatchesDiscovered 用于监控已发现的匹配数量
	MatchesDiscovered progress.Monitorable
	// Fixed 用于监控已修复的数量
	Fixed progress.Monitorable
	// Ignored 用于监控已忽略的数量
	Ignored progress.Monitorable
	// Dropped 用于监控已丢弃的数量
	Dropped progress.Monitorable
	// BySeverity 用于存储按严重程度分类的监控器
	BySeverity map[vulnerability.Severity]progress.Monitorable
}
```