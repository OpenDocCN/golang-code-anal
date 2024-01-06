# `grype\grype\match\type.go`

```
package match

import (
	"github.com/anchore/grype/grype/pkg"
)

// 定义常量，表示匹配类型
const (
	ExactDirectMatch   Type = "exact-direct-match"
	ExactIndirectMatch Type = "exact-indirect-match"
	CPEMatch           Type = "cpe-match"
)

// 定义匹配类型的字符串类型
type Type string

// 将匹配结果转换为间接匹配
func ConvertToIndirectMatches(matches []Match, p pkg.Package) {
	// 遍历匹配结果
	for idx := range matches {
		// 遍历每个匹配结果的详细信息
		for dIdx := range matches[idx].Details {
			// 只有当匹配详细信息明确指示为“直接”匹配时，才将匹配类型改为“间接”匹配
			if matches[idx].Details[dIdx].Type == ExactDirectMatch {
				matches[idx].Details[dIdx].Type = ExactIndirectMatch
		}
	}
	// 我们总是将包覆盖为直接包
	matches[idx].Package = p
}
```