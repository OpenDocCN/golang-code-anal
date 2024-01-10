# `grype\grype\version\version.go`

```
package version

import (
    "fmt"

    "github.com/anchore/grype/grype/pkg"
    "github.com/anchore/syft/syft/cpe"
)

type Version struct {
    Raw    string    // 版本的原始字符串
    Format Format    // 版本的格式
    rich   rich      // 版本的丰富信息
}

type rich struct {
    cpeVers       []cpe.CPE           // CPE 版本信息
    semVer        *semanticVersion    // 语义化版本信息
    apkVer        *apkVersion         // APK 版本信息
    debVer        *debVersion         // DEB 包版本信息
    golangVersion *golangVersion      // Golang 版本信息
    mavenVer      *mavenVersion       // Maven 版本信息
    rpmVer        *rpmVersion         // RPM 包版本信息
    kbVer         *kbVersion          // KB 包版本信息
    portVer       *portageVersion     // Portage 版本信息
    pep440version *pep440Version      // PEP440 版本信息
}

func NewVersion(raw string, format Format) (*Version, error) {
    version := &Version{
        Raw:    raw,
        Format: format,
    }

    err := version.populate()    // 填充版本的丰富信息
    if err != nil {
        return nil, err
    }

    return version, nil
}

func NewVersionFromPkg(p pkg.Package) (*Version, error) {
    ver, err := NewVersion(p.Version, FormatFromPkgType(p.Type))    // 从包信息创建版本对象
    if err != nil {
        return nil, err
    }

    ver.rich.cpeVers = p.CPEs    // 设置版本的 CPE 信息
    return ver, nil
}

func (v *Version) populate() error {
    switch v.Format {
    case SemanticFormat:
        ver, err := newSemanticVersion(v.Raw)    // 创建语义化版本对象
        v.rich.semVer = ver    // 设置版本的语义化版本信息
        return err
    case ApkFormat:
        ver, err := newApkVersion(v.Raw)    // 创建 APK 版本对象
        v.rich.apkVer = ver    // 设置版本的 APK 版本信息
        return err
    case DebFormat:
        ver, err := newDebVersion(v.Raw)    // 创建 DEB 包版本对象
        v.rich.debVer = ver    // 设置版本的 DEB 包版本信息
        return err
    case GolangFormat:
        ver, err := newGolangVersion(v.Raw)    // 创建 Golang 版本对象
        v.rich.golangVersion = ver    // 设置版本的 Golang 版本信息
        return err
    case MavenFormat:
        ver, err := newMavenVersion(v.Raw)    // 创建 Maven 版本对象
        v.rich.mavenVer = ver    // 设置版本的 Maven 版本信息
        return err
    case RpmFormat:
        ver, err := newRpmVersion(v.Raw)    // 创建 RPM 包版本对象
        v.rich.rpmVer = &ver    // 设置版本的 RPM 包版本信息
        return err
    case PythonFormat:
        ver, err := newPep440Version(v.Raw)    // 创建 PEP440 版本对象
        v.rich.pep440version = &ver    // 设置版本的 PEP440 版本信息
        return err
    case KBFormat:
        ver := newKBVersion(v.Raw)    // 创建 KB 包版本对象
        v.rich.kbVer = &ver    // 设置版本的 KB 包版本信息
        return nil
    # 根据不同的格式进行不同的处理
    case GemFormat:
        # 根据原始字符串创建 Gemfile 版本对象，并将其赋值给 rich.semVer
        ver, err := newGemfileVersion(v.Raw)
        v.rich.semVer = ver
        # 返回可能的错误
        return err
    case PortageFormat:
        # 根据原始字符串创建 Portage 版本对象，并将其赋值给 rich.portVer
        ver := newPortageVersion(v.Raw)
        v.rich.portVer = &ver
        # 返回空错误
        return nil
    case UnknownFormat:
        # 使用原始字符串 + 模糊约束
        return nil
    }

    # 如果没有为 rich 版本赋值，则返回错误信息
    return fmt.Errorf("no rich version populated (format=%s)", v.Format)
# 返回版本对象中的CPEs（Common Platform Enumeration，通用平台枚举）列表
func (v Version) CPEs() []cpe.CPE {
    return v.rich.cpeVers
}

# 返回版本对象的字符串表示，包括原始版本和格式
func (v Version) String() string {
    return fmt.Sprintf("%s (%s)", v.Raw, v.Format)
}
```