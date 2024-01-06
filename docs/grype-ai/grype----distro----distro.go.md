# `grype\grype\distro\distro.go`

```
// 导入必要的包
package distro

import (
	"fmt"  // 导入格式化包
	"strings"  // 导入字符串处理包

	hashiVer "github.com/hashicorp/go-version"  // 导入版本控制包

	"github.com/anchore/syft/syft/linux"  // 导入Linux相关包
)

// Distro表示Linux发行版
type Distro struct {
	Type       Type  // 发行版类型
	Version    *hashiVer.Version  // 发行版版本
	RawVersion string  // 原始版本字符串
	IDLike     []string  // 类似ID的字符串数组
}

// New创建一个新的Distro对象，并用给定的值填充
# 创建一个新的Distro对象，根据给定的类型、版本和IDLikes参数
func New(t Type, version string, idLikes ...string) (*Distro, error) {
    var verObj *hashiVer.Version
    var err error

    # 如果版本不为空，则尝试解析版本字符串为hashiVer.Version对象
    if version != "" {
        verObj, err = hashiVer.NewVersion(version)
        # 如果解析出错，则返回错误信息
        if err != nil {
            return nil, fmt.Errorf("unable to parse version: %w", err)
        }
    }

    # 返回一个新的Distro对象，包括类型、版本对象、原始版本字符串和IDLikes参数
    return &Distro{
        Type:       t,
        Version:    verObj,
        RawVersion: version,
        IDLike:     idLikes,
    }, nil
}

// NewFromRelease creates a new Distro object derived from a syft linux.Release object.
# 从给定的 Linux 发行版信息创建一个 Distro 对象
func NewFromRelease(release linux.Release) (*Distro, error) {
    # 从发行版信息中获取类型
    t := TypeFromRelease(release)
    # 如果类型为空，返回错误
    if t == "" {
        return nil, fmt.Errorf("unable to determine distro type")
    }

    # 选择版本号
    var selectedVersion string

    # 遍历版本号列表，选择第一个非空的版本号
    for _, version := range []string{release.VersionID, release.Version} {
        if version == "" {
            continue
        }

        # 如果版本号符合语义化版本规范，则选择该版本号
        if _, err := hashiVer.NewVersion(version); err == nil {
            selectedVersion = version
            break
        }
    }

    # 如果类型为 Debian，且版本号为空，且发行版名称包含 "sid"
    if t == Debian && release.VersionID == "" && release.Version == "" && strings.Contains(release.PrettyName, "sid") {
// 返回一个Distro结构体，包含类型、原始版本和IDLike字段
return &Distro{
    Type:       t,
    RawVersion: "unstable",
    IDLike:     release.IDLike,
}, nil
```

```
// 返回一个新的Distro结构体，包含类型、选定版本和release.IDLike字段
return New(t, selectedVersion, release.IDLike...)
```

```
// 返回Distro结构体的名称
func (d Distro) Name() string {
    return string(d.Type)
}
```

```
// 从伪语义版本的发行版版本值中返回主要版本值
func (d Distro) MajorVersion() string {
    if d.Version == nil {
        return strings.Split(d.RawVersion, ".")[0]
    }
    return fmt.Sprintf("%d", d.Version.Segments()[0])
}
// FullVersion函数返回原始用户版本值。
func (d Distro) FullVersion() string {
	return d.RawVersion
}

// String函数返回Linux发行版的人类友好表示。
func (d Distro) String() string {
	versionStr := "(version unknown)"
	if d.RawVersion != "" {
		versionStr = d.RawVersion
	}
	return fmt.Sprintf("%s %s", d.Type, versionStr)
}

// IsRolling函数检查发行版是否为滚动更新类型。
func (d Distro) IsRolling() bool {
	return d.Type == Wolfi || d.Type == Chainguard || d.Type == ArchLinux || d.Type == Gentoo
}
// 检查当前发行版是否被禁用
func (d Distro) Disabled() bool {
    // 使用 switch 语句检查发行版类型
    switch {
    // 如果发行版类型为 ArchLinux，则返回 true
    case d.Type == ArchLinux:
        return true
    // 如果发行版类型不是 ArchLinux，则返回 false
    default:
        return false
    }
}
```