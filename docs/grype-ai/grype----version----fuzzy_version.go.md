# `grype\grype\version\fuzzy_version.go`

```go
package version

import (
    "fmt"
)

type fuzzyVersion struct {
    semVer *semanticVersion  // 定义模糊版本结构体，包含指向语义版本的指针和原始字符串
    raw    string  // 原始字符串
}

//nolint:unparam
func newFuzzyVersion(raw string) (fuzzyVersion, error) {
    var semVer *semanticVersion  // 定义语义版本指针变量

    candidate, err := newSemanticVersion(raw)  // 尝试解析原始字符串为语义版本
    if err == nil {
        semVer = candidate  // 如果解析成功，将解析结果赋值给语义版本指针
    }

    return fuzzyVersion{
        semVer: semVer,  // 返回模糊版本结构体
        raw:    raw,
    }, nil
}

func (v *fuzzyVersion) Compare(other *Version) (int, error) {
    // check if both versions can be compared as semvers...
    if other.Format == SemanticFormat && v.semVer != nil {  // 检查是否两个版本都可以作为语义版本进行比较
        if other.rich.semVer == nil {  // 如果另一个版本的语义版本为空
            return -1, fmt.Errorf("given empty semver object (fuzzy)")  // 返回错误信息
        }
        return other.rich.semVer.verObj.Compare(v.semVer.verObj), nil  // 比较两个语义版本
    }

    // one or both are no semver compliant, use fuzzy comparison
    return fuzzyVersionComparison(other.Raw, v.raw), nil  // 使用模糊比较
}
```