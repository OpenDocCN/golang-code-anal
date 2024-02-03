# `grype\internal\stringutil\string_helpers.go`

```go
// 导入 strings 包，用于处理字符串相关操作
import "strings"

// HasAnyOfSuffixes 函数用于判断给定的字符串是否以给定的后缀之一结尾
func HasAnyOfSuffixes(input string, suffixes ...string) bool {
    // 遍历所有的后缀
    for _, suffix := range suffixes {
        // 如果字符串以某个后缀结尾，则返回 true
        if strings.HasSuffix(input, suffix) {
            return true
        }
    }
    // 如果没有任何后缀匹配，则返回 false
    return false
}

// HasAnyOfPrefixes 函数用于判断给定的字符串是否以给定的前缀之一开头
func HasAnyOfPrefixes(input string, prefixes ...string) bool {
    // 遍历所有的前缀
    for _, prefix := range prefixes {
        // 如果字符串以某个前缀开头，则返回 true
        if strings.HasPrefix(input, prefix) {
            return true
        }
    }
    // 如果没有任何前缀匹配，则返回 false
    return false
}

// SplitCommaSeparatedString 函数用于将逗号分隔的字符串分割成字符串切片
func SplitCommaSeparatedString(input string) []string {
    // 初始化一个空的字符串切片
    output := make([]string, 0)
    // 遍历逗号分隔的字符串
    for _, inputItem := range strings.Split(input, ",") {
        // 如果分割后的字符串长度大于 0，则添加到输出切片中
        if len(inputItem) > 0 {
            output = append(output, inputItem)
        }
    }
    // 返回分割后的字符串切片
    return output
}
```