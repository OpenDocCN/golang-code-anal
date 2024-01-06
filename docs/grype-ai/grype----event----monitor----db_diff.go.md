# `grype\grype\event\monitor\db_diff.go`

```
package monitor
// 导入名为 "github.com/wagoodman/go-progress" 的包

import "github.com/wagoodman/go-progress"
// 导入名为 "github.com/wagoodman/go-progress" 的包

type DBDiff struct {
	Stager                progress.Stager
	// 定义一个名为 Stager 的属性，类型为 progress.Stager
	StageProgress         progress.Progressable
	// 定义一个名为 StageProgress 的属性，类型为 progress.Progressable
	DifferencesDiscovered progress.Monitorable
	// 定义一个名为 DifferencesDiscovered 的属性，类型为 progress.Monitorable
}
```