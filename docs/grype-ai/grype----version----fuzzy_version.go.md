# `grype\grype\version\fuzzy_version.go`

```
// 导入 fmt 包，用于格式化输出
import (
	"fmt"
)

// 定义模糊版本结构体，包含语义化版本和原始字符串
type fuzzyVersion struct {
	semVer *semanticVersion
	raw    string
}

// 创建新的模糊版本对象，接受原始字符串作为参数，返回模糊版本对象和可能的错误
// nolint:unparam 表示忽略未使用的参数警告
func newFuzzyVersion(raw string) (fuzzyVersion, error) {
	var semVer *semanticVersion

	// 尝试将原始字符串转换为语义化版本对象
	candidate, err := newSemanticVersion(raw)
	if err == nil {
		// 如果转换成功，将语义化版本对象赋值给 semVer
		semVer = candidate
	}
// 返回一个模糊版本对象，包括语义版本和原始版本字符串
return fuzzyVersion{
    semVer: semVer,
    raw:    raw,
}, nil
}

// 比较当前模糊版本对象和另一个版本对象
func (v *fuzzyVersion) Compare(other *Version) (int, error) {
    // 检查两个版本是否都可以作为语义版本进行比较...
    if other.Format == SemanticFormat && v.semVer != nil {
        if other.rich.semVer == nil {
            return -1, fmt.Errorf("given empty semver object (fuzzy)")
        }
        return other.rich.semVer.verObj.Compare(v.semVer.verObj), nil
    }

    // 一个或两个版本都不符合语义版本规范，使用模糊比较
    return fuzzyVersionComparison(other.Raw, v.raw), nil
}
```