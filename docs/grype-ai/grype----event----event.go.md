# `grype\grype\event\event.go`

```
package event

import (
	"github.com/wagoodman/go-partybus"  // 导入 go-partybus 包
)

const (
	typePrefix    = "grype"  // 定义常量 typePrefix 为 "grype"
	cliTypePrefix = typePrefix + "-cli"  // 定义常量 cliTypePrefix 为 typePrefix 加上 "-cli"

	// Events from the grype library

	UpdateVulnerabilityDatabase  partybus.EventType = typePrefix + "-update-vulnerability-database"  // 定义事件类型 UpdateVulnerabilityDatabase 为 typePrefix 加上 "-update-vulnerability-database"
	VulnerabilityScanningStarted partybus.EventType = typePrefix + "-vulnerability-scanning-started"  // 定义事件类型 VulnerabilityScanningStarted 为 typePrefix 加上 "-vulnerability-scanning-started"
	DatabaseDiffingStarted       partybus.EventType = typePrefix + "-database-diffing-started"  // 定义事件类型 DatabaseDiffingStarted 为 typePrefix 加上 "-database-diffing-started"

	// Events exclusively for the CLI

	// CLIAppUpdateAvailable is a partybus event that occurs when an application update is available
	CLIAppUpdateAvailable partybus.EventType = cliTypePrefix + "-app-update-available"  // 定义事件类型 CLIAppUpdateAvailable 为 cliTypePrefix 加上 "-app-update-available"
```
```go
// 定义CLIReport为partybus事件，表示分析结果准备好最终呈现到标准输出
CLIReport partybus.EventType = cliTypePrefix + "-report"

// 定义CLINotification为partybus事件，表示辅助信息准备好呈现到标准错误输出
CLINotification partybus.EventType = cliTypePrefix + "-notification"
```