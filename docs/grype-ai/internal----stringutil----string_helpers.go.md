# `grype\internal\stringutil\string_helpers.go`

```
package stringutil

import "strings"

// HasAnyOfSuffixes函数返回给定字符串是否具有给定后缀的指示。
func HasAnyOfSuffixes(input string, suffixes ...string) bool {
	// 遍历后缀数组
	for _, suffix := range suffixes {
		// 检查字符串是否以指定后缀结尾
		if strings.HasSuffix(input, suffix) {
			// 如果是，则返回true
			return true
		}
	}

	// 如果没有匹配的后缀，则返回false
	return false
}

// HasAnyOfPrefixes函数返回给定字符串是否具有给定前缀的指示。
func HasAnyOfPrefixes(input string, prefixes ...string) bool {
	// 遍历前缀数组
	for _, prefix := range prefixes {
		// 检查字符串是否以指定前缀开头
		if strings.HasPrefix(input, prefix) {
			// 如果是，则返回true
			return true
		}
	}

	// 如果没有匹配的前缀，则返回false
// 结束当前函数的执行，并返回 false
		}
	}

	// 如果以上条件都不满足，则返回 false
	return false
}

// SplitCommaSeparatedString 根据逗号分隔的输入字符串返回一个字符串切片
func SplitCommaSeparatedString(input string) []string {
	// 创建一个空的字符串切片
	output := make([]string, 0)
	// 遍历使用逗号分隔的输入字符串的每个部分
	for _, inputItem := range strings.Split(input, ",") {
		// 如果部分的长度大于 0，则将其添加到输出切片中
		if len(inputItem) > 0 {
			output = append(output, inputItem)
		}
	}
	// 返回输出切片
	return output
}
```