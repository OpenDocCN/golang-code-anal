# `grype\grype\version\format_test.go`

```go
package version

import (
    "fmt"
    "testing"

    "github.com/anchore/syft/syft/pkg"
)

func TestParseFormat(t *testing.T) {
    tests := []struct {
        input  string
        format Format
    }{
        {
            input:  "dpkg",  // 设置输入字符串
            format: DebFormat,  // 设置期望的格式
        },
        {
            input:  "maven",  // 设置输入字符串
            format: MavenFormat,  // 设置期望的格式
        },
        {
            input:  "gem",  // 设置输入字符串
            format: GemFormat,  // 设置期望的格式
        },
        {
            input:  "deb",  // 设置输入字符串
            format: DebFormat,  // 设置期望的格式
        },
        {
            input:  "semantic",  // 设置输入字符串
            format: SemanticFormat,  // 设置期望的格式
        },
        {
            input:  "semver",  // 设置输入字符串
            format: SemanticFormat,  // 设置期望的格式
        },
    }

    for _, test := range tests {
        name := fmt.Sprintf("'%s'->format[%s]", test.input, test.format)  // 根据输入字符串和期望的格式创建测试名称
        t.Run(name, func(t *testing.T) {
            actual := ParseFormat(test.input)  // 调用函数获取实际的格式
            if actual != test.format {  // 检查实际的格式是否与期望的格式一致
                t.Errorf("mismatched user string -> format mapping, pkgType='%s': '%s'!='%s'", test.input, test.format, actual)  // 输出错误信息
            }
        })
    }
}

func TestFormatFromPkgType(t *testing.T) {
    tests := []struct {
        pkgType pkg.Type
        format  Format
    }{
        {
            pkgType: pkg.DebPkg,  // 设置包类型
            format:  DebFormat,  // 设置期望的格式
        },
        {
            pkgType: pkg.JavaPkg,  // 设置包类型
            format:  MavenFormat,  // 设置期望的格式
        },
        {
            pkgType: pkg.GemPkg,  // 设置包类型
            format:  GemFormat,  // 设置期望的格式
        },
    }

    for _, test := range tests {
        name := fmt.Sprintf("pkgType[%s]->format[%s]", test.pkgType, test.format)  // 根据包类型和期望的格式创建测试名称
        t.Run(name, func(t *testing.T) {
            actual := FormatFromPkgType(test.pkgType)  // 调用函数获取实际的格式
            if actual != test.format {  // 检查实际的格式是否与期望的格式一致
                t.Errorf("mismatched pkgType->format mapping, pkgType='%s': '%s'!='%s'", test.pkgType, test.format, actual)  // 输出错误信息
            }
        })
    }
}
```