# `grype\grype\version\gemfile_version.go`

```
// 定义一个名为version的包

import (
    "strings"  // 导入strings包，用于字符串操作
)

// 从Gemfile.lock中提取语义化版本号
func extractSemVer(raw string) string {
    // 定义可能的平台值
    platforms := []string{"x86", "universal", "arm", "java", "dalvik", "x64", "powerpc", "sparc", "mswin"}
    dash := "-"  // 定义连接符
    // 遍历可能的平台值
    for _, p := range platforms {
        // 使用平台值对原始字符串进行分割
        vals := strings.SplitN(raw, dash+p, 2)
        // 如果成功分割，返回分割后的第一部分作为语义化版本号
        if len(vals) == 2 {
            return vals[0]
        }
    }
    // 如果无法分割，返回原始字符串
    return raw
}

// 创建一个新的Gemfile版本
func newGemfileVersion(raw string) (*semanticVersion, error) {
    // 提取语义化版本号
    cleaned := extractSemVer(raw)
    // 创建新的语义化版本对象
    return newSemanticVersion(cleaned)
}
```