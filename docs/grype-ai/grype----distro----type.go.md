# `grype\grype\distro\type.go`

```
package distro

import (
    "github.com/anchore/syft/syft/linux"
)

// Type represents the different Linux distribution options
type Type string

const (
    // represents the set of supported Linux Distributions

    Debian       Type = "debian"  // 声明并初始化 Debian 变量
    Ubuntu       Type = "ubuntu"  // 声明并初始化 Ubuntu 变量
    RedHat       Type = "redhat"  // 声明并初始化 RedHat 变量
    CentOS       Type = "centos"  // 声明并初始化 CentOS 变量
    Fedora       Type = "fedora"  // 声明并初始化 Fedora 变量
    Alpine       Type = "alpine"  // 声明并初始化 Alpine 变量
    Busybox      Type = "busybox" // 声明并初始化 Busybox 变量
    AmazonLinux  Type = "amazonlinux"  // 声明并初始化 AmazonLinux 变量
    OracleLinux  Type = "oraclelinux"  // 声明并初始化 OracleLinux 变量
    ArchLinux    Type = "archlinux"  // 声明并初始化 ArchLinux 变量
    OpenSuseLeap Type = "opensuseleap"  // 声明并初始化 OpenSuseLeap 变量
    SLES         Type = "sles"  // 声明并初始化 SLES 变量
    Photon       Type = "photon"  // 声明并初始化 Photon 变量
    Windows      Type = "windows"  // 声明并初始化 Windows 变量
    Mariner      Type = "mariner"  // 声明并初始化 Mariner 变量
    RockyLinux   Type = "rockylinux"  // 声明并初始化 RockyLinux 变量
    AlmaLinux    Type = "almalinux"  // 声明并初始化 AlmaLinux 变量
    Gentoo       Type = "gentoo"  // 声明并初始化 Gentoo 变量
    Wolfi        Type = "wolfi"  // 声明并初始化 Wolfi 变量
    Chainguard   Type = "chainguard"  // 声明并初始化 Chainguard 变量
)

// All contains all Linux distribution options
var All = []Type{
    Debian,
    Ubuntu,
    RedHat,
    CentOS,
    Fedora,
    Alpine,
    Busybox,
    AmazonLinux,
    OracleLinux,
    ArchLinux,
    OpenSuseLeap,
    SLES,
    Photon,
    Windows,
    Mariner,
    RockyLinux,
    AlmaLinux,
    Gentoo,
    Wolfi,
    Chainguard,
}

// IDMapping connects a distro ID like "ubuntu" to a Distro type
var IDMapping = map[string]Type{
    "debian":        Debian,  // 将 "debian" 映射到 Debian 变量
    "ubuntu":        Ubuntu,  // 将 "ubuntu" 映射到 Ubuntu 变量
    "rhel":          RedHat,  // 将 "rhel" 映射到 RedHat 变量
    "centos":        CentOS,  // 将 "centos" 映射到 CentOS 变量
    "fedora":        Fedora,  // 将 "fedora" 映射到 Fedora 变量
    "alpine":        Alpine,  // 将 "alpine" 映射到 Alpine 变量
    "busybox":       Busybox,  // 将 "busybox" 映射到 Busybox 变量
    "amzn":          AmazonLinux,  // 将 "amzn" 映射到 AmazonLinux 变量
    "ol":            OracleLinux,  // 将 "ol" 映射到 OracleLinux 变量
    "arch":          ArchLinux,  // 将 "arch" 映射到 ArchLinux 变量
    "opensuse-leap": OpenSuseLeap,  // 将 "opensuse-leap" 映射到 OpenSuseLeap 变量
    "sles":          SLES,  // 将 "sles" 映射到 SLES 变量
    "photon":        Photon,  // 将 "photon" 映射到 Photon 变量
    "windows":       Windows,  // 将 "windows" 映射到 Windows 变量
    "mariner":       Mariner,  // 将 "mariner" 映射到 Mariner 变量
    "rocky":         RockyLinux,  // 将 "rocky" 映射到 RockyLinux 变量
    "almalinux":     AlmaLinux,  // 将 "almalinux" 映射到 AlmaLinux 变量
    "gentoo":        Gentoo,  // 将 "gentoo" 映射到 Gentoo 变量
    "wolfi":         Wolfi,  // 将 "wolfi" 映射到 Wolfi 变量
    "chainguard":    Chainguard,  // 将 "chainguard" 映射到 Chainguard 变量
}
# 根据给定的 Linux 发行版信息返回对应的 Type 类型
func TypeFromRelease(release linux.Release) Type {
    # 首先尝试使用发行版的 ID 来获取对应的 Type
    t, ok := IDMapping[release.ID]
    if ok {
        return t
    }

    # 如果 ID 没有对应的 Type，则尝试使用 ID_LIKE 作为备用
    for _, l := range release.IDLike {
        if t, ok := IDMapping[l]; ok {
            return t
        }
    }

    # 如果 ID_LIKE 也没有对应的 Type，则尝试使用发行版的名称作为备用
    t, ok = IDMapping[release.Name]
    if ok {
        return t
    }

    # 如果都没有找到对应的 Type，则返回空字符串
    return ""
}

# 返回给定 Linux 发行版的字符串表示
func (t Type) String() string {
    return string(t)
}
```