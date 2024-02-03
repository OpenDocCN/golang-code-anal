# `grype\grype\event\event.go`

```go
package event

import (
    "github.com/wagoodman/go-partybus"
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

    // CLIReport is a partybus event that occurs when an analysis result is ready for final presentation to stdout
    CLIReport partybus.EventType = cliTypePrefix + "-report"  // 定义事件类型 CLIReport 为 cliTypePrefix 加上 "-report"

    // CLINotification is a partybus event that occurs when auxiliary information is ready for presentation to stderr
    CLINotification partybus.EventType = cliTypePrefix + "-notification"  // 定义事件类型 CLINotification 为 cliTypePrefix 加上 "-notification"
)
```