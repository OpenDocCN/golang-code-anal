# `grype\internal\stringutil\parse.go`

```
// 导入 regexp 包，用于正则表达式操作
import "regexp"

// MatchCaptureGroups 函数接受一个正则表达式和一个字符串，返回所有命名捕获组的结果映射
func MatchCaptureGroups(regEx *regexp.Regexp, str string) map[string]string {
    // 使用正则表达式在字符串中查找所有匹配项
    match := regEx.FindStringSubmatch(str)
    // 创建一个空的结果映射
    results := make(map[string]string)
    // 遍历所有捕获组的索引和名称
    for i, name := range regEx.SubexpNames() {
        // 如果索引大于 0 并且小于等于匹配项的数量，则将捕获组的名称和值添加到结果映射中
        if i > 0 && i <= len(match) {
            results[name] = match[i]
        }
    }
    // 返回结果映射
    return results
}
```