# `grype\grype\event\monitor\db_diff.go`

```go
# 导入名为 "github.com/wagoodman/go-progress" 的包
import "github.com/wagoodman/go-progress"

# 定义名为 "DBDiff" 的结构体
type DBDiff struct {
    # 定义名为 "Stager" 的字段，类型为 progress.Stager
    Stager                progress.Stager
    # 定义名为 "StageProgress" 的字段，类型为 progress.Progressable
    StageProgress         progress.Progressable
    # 定义名为 "DifferencesDiscovered" 的字段，类型为 progress.Monitorable
    DifferencesDiscovered progress.Monitorable
}
```