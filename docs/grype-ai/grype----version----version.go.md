# `grype\grype\version\version.go`

```
# 导入所需的包
package version

import (
	"fmt"

	"github.com/anchore/grype/grype/pkg"
	"github.com/anchore/syft/syft/cpe"
)

# 定义 Version 结构体
type Version struct {
	# 原始版本字符串
	Raw    string
	# 版本格式
	Format Format
	# 富版本信息
	rich   rich
}

# 富版本信息结构体
type rich struct {
	# CPE 版本信息数组
	cpeVers       []cpe.CPE
	# 语义版本
	semVer        *semanticVersion
	# APK 包管理器版本
	apkVer        *apkVersion
	# DEB 包管理器版本
	debVer        *debVersion
}
# 定义变量 golangVersion，类型为 golangVersion 结构体指针
# 定义变量 mavenVer，类型为 mavenVersion 结构体指针
# 定义变量 rpmVer，类型为 rpmVersion 结构体指针
# 定义变量 kbVer，类型为 kbVersion 结构体指针
# 定义变量 portVer，类型为 portageVersion 结构体指针
# 定义变量 pep440version，类型为 pep440Version 结构体指针

# 定义函数 NewVersion，接受原始版本字符串和格式作为参数，返回 Version 结构体指针和错误信息
func NewVersion(raw string, format Format) (*Version, error) {
    # 创建 Version 结构体对象，并赋值原始版本字符串和格式
    version := &Version{
        Raw:    raw,
        Format: format,
    }

    # 调用 populate 方法填充 Version 结构体对象的其他字段
    err := version.populate()
    # 如果填充过程中出现错误，返回 nil 和错误信息
    if err != nil {
        return nil, err
    }

    # 返回填充后的 Version 结构体对象和 nil 错误信息
    return version, nil
}
}

// 从包对象创建新版本对象
func NewVersionFromPkg(p pkg.Package) (*Version, error) {
	// 根据包的版本和类型创建新版本对象
	ver, err := NewVersion(p.Version, FormatFromPkgType(p.Type))
	if err != nil {
		return nil, err
	}

	// 将包的CPEs赋值给新版本对象的CPEs
	ver.rich.cpeVers = p.CPEs
	return ver, nil
}

// 填充版本对象的数据
func (v *Version) populate() error {
	// 根据不同的格式填充版本对象的数据
	switch v.Format {
	case SemanticFormat:
		// 根据语义版本格式填充数据
		ver, err := newSemanticVersion(v.Raw)
		v.rich.semVer = ver
		return err
	case ApkFormat:
		// 根据APK格式填充数据
		ver, err := newApkVersion(v.Raw)
		# 设置APK版本号，并返回可能的错误
		v.rich.apkVer = ver
		return err
	case DebFormat:
		# 创建Deb版本对象，并设置到版本对象中，返回可能的错误
		ver, err := newDebVersion(v.Raw)
		v.rich.debVer = ver
		return err
	case GolangFormat:
		# 创建Golang版本对象，并设置到版本对象中，返回可能的错误
		ver, err := newGolangVersion(v.Raw)
		v.rich.golangVersion = ver
		return err
	case MavenFormat:
		# 创建Maven版本对象，并设置到版本对象中，返回可能的错误
		ver, err := newMavenVersion(v.Raw)
		v.rich.mavenVer = ver
		return err
	case RpmFormat:
		# 创建RPM版本对象，并设置到版本对象中，返回可能的错误
		ver, err := newRpmVersion(v.Raw)
		v.rich.rpmVer = &ver
		return err
	case PythonFormat:
		# 创建Python版本对象，并设置到版本对象中，返回可能的错误
		ver, err := newPep440Version(v.Raw)
		// 将版本信息赋值给 PEP 440 格式的版本对象，并返回可能的错误
		v.rich.pep440version = &ver
		return err
	case KBFormat:
		// 创建一个 KB 版本对象，并赋值给 rich.kbVer，然后返回 nil
		ver := newKBVersion(v.Raw)
		v.rich.kbVer = &ver
		return nil
	case GemFormat:
		// 创建一个 Gemfile 版本对象，并赋值给 rich.semVer，然后返回可能的错误
		ver, err := newGemfileVersion(v.Raw)
		v.rich.semVer = ver
		return err
	case PortageFormat:
		// 创建一个 Portage 版本对象，并赋值给 rich.portVer，然后返回 nil
		ver := newPortageVersion(v.Raw)
		v.rich.portVer = &ver
		return nil
	case UnknownFormat:
		// 使用原始字符串 + 模糊约束
		// 返回 nil，表示未知格式的版本对象不需要进行处理
		return nil
	}

	// 如果没有匹配到任何格式，返回错误信息
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