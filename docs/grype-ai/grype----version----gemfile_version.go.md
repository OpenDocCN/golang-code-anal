# `grype\grype\version\gemfile_version.go`

```
package version

import (
	"strings"
)

// Gemfile.lock doesn't follow a spec, the best documentation comes
// from `gem help platform`. Gemfile.lock versions may have "{cpu}-{os}"
// or "{cpu}-{os}-{version}"
// after the semvVer, for example, 12.2.1-alpha-x86_64-darwin-8, where `2.2.1-alpha`
// is a valid and comparable semVer, and `x86_64-darwin-8` is not a semVer due to
// the underscore. Also, we can't sort based on arch and OS in a way that make sense
// for versions. SemVer is a characteristic of the code, not which arch OS it runs on.
//
//
// Spec for pre-release info: https://github.com/rubygems/rubygems/blob/2070231bf0c7c4654bbc2e4c08882bf414840360/bundler/spec/install/gemfile/path_spec.rb#L186
//
// CPU/arch is the most structured value present in gemfile.lock versions, we use it
// to split the version info in half, the first half has semVer, and
// the second half has arch and OS which we ignore.
```

注释：
- 该部分代码是一个包的声明，用于声明该文件属于version包。
- 导入strings包，用于处理字符串相关操作。
- 以下是对Gemfile.lock文件版本号的一些说明和规范，包括版本号可能包含的CPU和OS信息，以及版本号的结构和排序规则。
- 提供了有关预发布信息的规范链接。
- 解释了CPU/架构是gemfile.lock版本中最有结构的值，用于将版本信息分成两部分，第一部分是semVer，第二部分是架构和操作系统，我们忽略后半部分。
// 从版本字符串中提取语义版本号，当没有架构信息时，使用{java, delvik, mswin}进行分割
func extractSemVer(raw string) string {
    // 定义可能的平台架构
    platforms := []string{"x86", "universal", "arm", "java", "dalvik", "x64", "powerpc", "sparc", "mswin"}
    // 定义分隔符
    dash := "-"
    // 遍历可能的平台架构
    for _, p := range platforms {
        // 使用当前平台架构进行分割
        vals := strings.SplitN(raw, dash+p, 2)
        // 如果成功分割出两部分，则返回第一部分
        if len(vals) == 2 {
            return vals[0]
        }
    }
    // 如果没有成功分割，则返回原始字符串
    return raw
}

// 根据原始字符串创建新的语义版本对象
func newGemfileVersion(raw string) (*semanticVersion, error) {
    // 提取语义版本号
    cleaned := extractSemVer(raw)
    // 创建新的语义版本对象
    return newSemanticVersion(cleaned)
}
```