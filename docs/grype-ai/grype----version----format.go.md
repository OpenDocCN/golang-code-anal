# `grype\grype\version\format.go`

```
package version

import (
    "strings"

    "github.com/anchore/syft/syft/pkg"
)

const (
    UnknownFormat Format = iota  // 未知格式的常量值
    SemanticFormat  // 语义格式的常量值
    ApkFormat  // APK 格式的常量值
    DebFormat  // DEB 格式的常量值
    MavenFormat  // Maven 格式的常量值
    RpmFormat  // RPM 格式的常量值
    PythonFormat  // Python 格式的常量值
    KBFormat  // KB 格式的常量值
    GemFormat  // Gem 格式的常量值
    PortageFormat  // Portage 格式的常量值
    GolangFormat  // Golang 格式的常量值
)

type Format int  // 格式类型的定义

var formatStr = []string{  // 格式字符串的数组
    "UnknownFormat",
    "Semantic",
    "Apk",
    "Deb",
    "Maven",
    "RPM",
    "Python",
    "KB",
    "Gem",
    "Portage",
    "Go",
}

var Formats = []Format{  // 格式类型的数组
    SemanticFormat,
    ApkFormat,
    DebFormat,
    MavenFormat,
    RpmFormat,
    PythonFormat,
    KBFormat,
    GemFormat,
    PortageFormat,
}

func ParseFormat(userStr string) Format {  // 解析用户输入的格式字符串
    switch strings.ToLower(userStr) {  // 将用户输入的字符串转换为小写后进行匹配
    case strings.ToLower(SemanticFormat.String()), "semver":  // 如果匹配到语义格式或者 "semver"
        return SemanticFormat  // 返回语义格式
    case strings.ToLower(ApkFormat.String()), "apk":  // 如果匹配到 APK 格式或者 "apk"
        return ApkFormat  // 返回 APK 格式
    case strings.ToLower(DebFormat.String()), "dpkg":  // 如果匹配到 DEB 格式或者 "dpkg"
        return DebFormat  // 返回 DEB 格式
    case strings.ToLower(GolangFormat.String()), "go":  // 如果匹配到 Golang 格式或者 "go"
        return GolangFormat  // 返回 Golang 格式
    case strings.ToLower(MavenFormat.String()), "maven":  // 如果匹配到 Maven 格式或者 "maven"
        return MavenFormat  // 返回 Maven 格式
    case strings.ToLower(RpmFormat.String()), "rpm":  // 如果匹配到 RPM 格式或者 "rpm"
        return RpmFormat  // 返回 RPM 格式
    case strings.ToLower(PythonFormat.String()), "python":  // 如果匹配到 Python 格式或者 "python"
        return PythonFormat  // 返回 Python 格式
    case strings.ToLower(KBFormat.String()), "kb":  // 如果匹配到 KB 格式或者 "kb"
        return KBFormat  // 返回 KB 格式
    case strings.ToLower(GemFormat.String()), "gem":  // 如果匹配到 Gem 格式或者 "gem"
        return GemFormat  // 返回 Gem 格式
    case strings.ToLower(PortageFormat.String()), "portage":  // 如果匹配到 Portage 格式或者 "portage"
        return PortageFormat  // 返回 Portage 格式
    }
    return UnknownFormat  // 如果都没有匹配到，则返回未知格式
}

func FormatFromPkgType(t pkg.Type) Format {  // 根据包类型获取格式
    var format Format  // 定义格式变量
    switch t {  // 根据包类型进行匹配
    case pkg.ApkPkg:  // 如果是 APK 包
        format = ApkFormat  // 设置为 APK 格式
    case pkg.DebPkg:  // 如果是 DEB 包
        format = DebFormat  // 设置为 DEB 格式
    case pkg.JavaPkg:  // 如果是 Java 包
        format = MavenFormat  // 设置为 Maven 格式
    case pkg.RpmPkg:  // 如果是 RPM 包
        format = RpmFormat  // 设置为 RPM 格式
    case pkg.GemPkg:  // 如果是 Gem 包
        format = GemFormat  // 设置为 Gem 格式
    case pkg.PythonPkg:  // 如果是 Python 包
        format = PythonFormat  // 设置为 Python 格式
    # 根据不同的包类型设置不同的格式
    case pkg.KbPkg:  # 如果是 KbPkg 类型的包
        format = KBFormat  # 设置格式为 KBFormat
    case pkg.PortagePkg:  # 如果是 PortagePkg 类型的包
        format = PortageFormat  # 设置格式为 PortageFormat
    case pkg.GoModulePkg:  # 如果是 GoModulePkg 类型的包
        format = GolangFormat  # 设置格式为 GolangFormat
    default:  # 如果是其他类型的包
        format = UnknownFormat  # 设置格式为 UnknownFormat
    }  # 结束 switch 语句
    return format  # 返回设置好的格式
# 定义 Format 结构体的 String 方法，用于将 Format 类型转换为字符串
func (f Format) String() string:
    # 如果 f 的值大于等于 formatStr 的长度，或者小于 0，则返回 formatStr 的第一个元素
    if int(f) >= len(formatStr) || f < 0:
        return formatStr[0]
    # 否则返回 formatStr 中索引为 f 的元素
    return formatStr[f]
```