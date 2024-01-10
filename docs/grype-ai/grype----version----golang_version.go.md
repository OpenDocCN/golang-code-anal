# `grype\grype\version\golang_version.go`

```
package version

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "strings"  // 导入 strings 包，用于字符串操作

    hashiVer "github.com/anchore/go-version"  // 导入第三方库 github.com/anchore/go-version，并重命名为 hashiVer
)

var _ Comparator = (*golangVersion)(nil)  // 定义一个接口类型的变量，用于比较版本号

type golangVersion struct {
    raw    string  // 定义原始版本号字符串
    semVer *hashiVer.Version  // 定义语义化版本号对象
}

func (g golangVersion) Compare(version *Version) (int, error) {
    if version.Format != GolangFormat {  // 如果版本号格式不是 GolangFormat
        return -1, fmt.Errorf("cannot compare %v to golang version", version.Format)  // 返回错误信息
    }
    if version.rich.golangVersion == nil {  // 如果版本号中的 golangVersion 为 nil
        return -1, fmt.Errorf("cannot compare version with nil golang version to golang version")  // 返回错误信息
    }
    if version.rich.golangVersion.raw == g.raw {  // 如果版本号中的原始版本号与当前 golangVersion 的原始版本号相同
        return 0, nil  // 返回相同
    }
    if version.rich.golangVersion.raw == "(devel)" {  // 如果版本号中的原始版本号为 "(devel)"
        return -1, fmt.Errorf("cannot compare %s with %s", g.raw, version.rich.golangVersion.raw)  // 返回错误信息
    }

    return version.rich.golangVersion.compare(g), nil  // 调用 golangVersion 的 compare 方法进行比较
}

func (g golangVersion) compare(o golangVersion) int {
    switch {
    case g.semVer != nil && o.semVer != nil:  // 如果当前 golangVersion 和目标 golangVersion 的语义化版本号都不为 nil
        return g.semVer.Compare(o.semVer)  // 调用语义化版本号对象的 Compare 方法进行比较
    case g.semVer != nil && o.semVer == nil:  // 如果当前 golangVersion 的语义化版本号不为 nil，目标 golangVersion 的语义化版本号为 nil
        return 1  // 返回 1
    case g.semVer == nil && o.semVer != nil:  // 如果当前 golangVersion 的语义化版本号为 nil，目标 golangVersion 的语义化版本号不为 nil
        return -1  // 返回 -1
    default:  // 其他情况
        return strings.Compare(g.raw, o.raw)  // 使用 strings 包中的 Compare 方法比较原始版本号字符串
    }
}

func newGolangVersion(v string) (*golangVersion, error) {
    // go stdlib is reported by syft as a go package with version like "go1.24.1"
    // other versions have "v" as a prefix, which the semver lib handles automatically
    semver, err := hashiVer.NewSemver(strings.TrimPrefix(v, "go"))  // 使用第三方库中的 NewSemver 方法创建语义化版本号对象
    if err != nil {  // 如果有错误发生
        return nil, err  // 返回 nil 和错误信息
    }
    return &golangVersion{
        raw:    v,  // 设置原始版本号字符串
        semVer: semver,  // 设置语义化版本号对象
    }, nil  // 返回 golangVersion 对象和 nil
}
```