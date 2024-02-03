# `grype\grype\match\type.go`

```go
package match

import (
    "github.com/anchore/grype/grype/pkg"
)

const (
    // 定义精确直接匹配类型常量
    ExactDirectMatch   Type = "exact-direct-match"
    // 定义精确间接匹配类型常量
    ExactIndirectMatch Type = "exact-indirect-match"
    // 定义CPE匹配类型常量
    CPEMatch           Type = "cpe-match"
)

type Type string

func ConvertToIndirectMatches(matches []Match, p pkg.Package) {
    // 遍历匹配结果数组
    for idx := range matches {
        // 遍历每个匹配结果的详细信息
        for dIdx := range matches[idx].Details {
            // 只有当匹配结果的详细信息明确指示为"直接"匹配时，才将匹配类型覆盖为"间接"匹配
            if matches[idx].Details[dIdx].Type == ExactDirectMatch {
                matches[idx].Details[dIdx].Type = ExactIndirectMatch
            }
        }
        // 总是将匹配结果的包信息覆盖为指定的包信息
        matches[idx].Package = p
    }
}
```