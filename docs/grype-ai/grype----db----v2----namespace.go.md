# `grype\grype\db\v2\namespace.go`

```go
package v2

import (
    "fmt"
)

const (
    NVDNamespace = "nvd"  # 定义常量 NVDNamespace 为 "nvd"
)

func RecordSource(feed, group string) string {
    switch feed {
    case "github", "nvdv2":  # 如果 feed 为 "github" 或 "nvdv2"
        return group  # 返回 group
    default:
        return fmt.Sprintf("%s:%s", feed, group)  # 返回格式化后的字符串
    }
}

func NamespaceForFeedGroup(feed, group string) (string, error) {
    switch {
    case feed == "vulnerabilities":  # 如果 feed 为 "vulnerabilities"
        return group, nil  # 返回 group 和空错误
    case feed == "github":  # 如果 feed 为 "github"
        return group, nil  # 返回 group 和空错误
    case feed == "nvdv2" && group == "nvdv2:cves":  # 如果 feed 为 "nvdv2" 并且 group 为 "nvdv2:cves"
        return NVDNamespace, nil  # 返回 NVDNamespace 和空错误
    }
    return "", fmt.Errorf("feed=%q group=%q has no namespace mappings", feed, group)  # 返回空字符串和格式化后的错误信息
}
```