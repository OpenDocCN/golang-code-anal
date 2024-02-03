# `grype\internal\stringutil\parse.go`

```go
// 导入 regexp 包
import "regexp"

// MatchCaptureGroups 函数接受一个正则表达式和一个字符串，返回一个包含所有命名捕获组结果的映射
func MatchCaptureGroups(regEx *regexp.Regexp, str string) map[string]string {
    // 使用正则表达式在字符串中查找匹配项
    match := regEx.FindStringSubmatch(str)
    // 创建一个空的映射用于存储结果
    results := make(map[string]string)
    // 遍历捕获组名称和匹配结果，将结果存储在映射中
    for i, name := range regEx.SubexpNames() {
        if i > 0 && i <= len(match) {
            results[name] = match[i]
        }
    }
    // 返回结果映射
    return results
}
```