# `grype\grype\event\monitor\matching.go`

```
# 定义一个名为 monitor 的包
package monitor

# 导入 go-progress 包，用于显示进度条
import (
    "github.com/wagoodman/go-progress"

    # 导入 vulnerability 包，用于处理漏洞信息
    "github.com/anchore/grype/grype/vulnerability"
)

# 定义一个名为 Matching 的结构体，用于存储匹配信息
type Matching struct {
    # 已处理的包的进度
    PackagesProcessed progress.Progressable
    # 已发现的匹配进度
    MatchesDiscovered progress.Monitorable
    # 已修复的进度
    Fixed             progress.Monitorable
    # 已忽略的进度
    Ignored           progress.Monitorable
    # 已丢弃的进度
    Dropped           progress.Monitorable
    # 按严重程度分类的进度
    BySeverity        map[vulnerability.Severity]progress.Monitorable
}
```