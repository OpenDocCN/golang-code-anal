# `grype\grype\distro\distro.go`

```go
// 导入必要的包
package distro

import (
    "fmt"
    "strings"

    hashiVer "github.com/hashicorp/go-version"

    "github.com/anchore/syft/syft/linux"
)

// Distro 表示一个 Linux 发行版
type Distro struct {
    Type       Type
    Version    *hashiVer.Version
    RawVersion string
    IDLike     []string
}

// New 根据给定的数值创建一个新的 Distro 对象
func New(t Type, version string, idLikes ...string) (*Distro, error) {
    var verObj *hashiVer.Version
    var err error

    // 如果版本号不为空，则尝试创建一个 hashiVer.Version 对象
    if version != "" {
        verObj, err = hashiVer.NewVersion(version)
        if err != nil {
            return nil, fmt.Errorf("unable to parse version: %w", err)
        }
    }

    // 返回一个新的 Distro 对象
    return &Distro{
        Type:       t,
        Version:    verObj,
        RawVersion: version,
        IDLike:     idLikes,
    }, nil
}

// NewFromRelease 从 syft linux.Release 对象创建一个新的 Distro 对象
func NewFromRelease(release linux.Release) (*Distro, error) {
    t := TypeFromRelease(release)
    if t == "" {
        return nil, fmt.Errorf("unable to determine distro type")
    }

    var selectedVersion string

    // 遍历版本号列表，选择第一个能成功创建 hashiVer.Version 对象的版本号
    for _, version := range []string{release.VersionID, release.Version} {
        if version == "" {
            continue
        }

        if _, err := hashiVer.NewVersion(version); err == nil {
            selectedVersion = version
            break
        }
    }

    // 如果是 Debian 并且版本号为空，并且 PrettyName 包含 "sid"，则返回一个特定的 Distro 对象
    if t == Debian && release.VersionID == "" && release.Version == "" && strings.Contains(release.PrettyName, "sid") {
        return &Distro{
            Type:       t,
            RawVersion: "unstable",
            IDLike:     release.IDLike,
        }, nil
    }

    // 否则，根据选定的版本号和 IDLike 创建一个新的 Distro 对象
    return New(t, selectedVersion, release.IDLike...)
}

// 返回 Distro 的名称
func (d Distro) Name() string {
    return string(d.Type)
}

// MajorVersion 从伪语义版本的发行版版本值中返回主要版本值
func (d Distro) MajorVersion() string {
    # 如果版本号为空，则返回原始版本号的第一个点之前的部分
    if d.Version == nil:
        return strings.Split(d.RawVersion, ".")[0]
    # 否则，返回版本号的第一个段
    return fmt.Sprintf("%d", d.Version.Segments()[0])
// FullVersion 返回原始用户版本值。
func (d Distro) FullVersion() string {
    return d.RawVersion
}

// String 返回 Linux 发行版的人类友好表示。
func (d Distro) String() string {
    versionStr := "(version unknown)"
    if d.RawVersion != "" {
        versionStr = d.RawVersion
    }
    return fmt.Sprintf("%s %s", d.Type, versionStr)
}

func (d Distro) IsRolling() bool {
    return d.Type == Wolfi || d.Type == Chainguard || d.Type == ArchLinux || d.Type == Gentoo
}

// 不支持的 Linux 发行版
func (d Distro) Disabled() bool {
    switch {
    case d.Type == ArchLinux:
        return true
    default:
        return false
    }
}
```