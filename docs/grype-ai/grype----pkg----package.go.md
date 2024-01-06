# `grype\grype\pkg\package.go`

```
package pkg

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"regexp"  // 导入 regexp 包，用于正则表达式匹配
	"strings"  // 导入 strings 包，用于字符串操作

	"github.com/anchore/grype/internal/log"  // 导入日志包
	"github.com/anchore/grype/internal/stringutil"  // 导入字符串工具包
	"github.com/anchore/syft/syft/artifact"  // 导入 artifact 包
	"github.com/anchore/syft/syft/cpe"  // 导入 cpe 包
	"github.com/anchore/syft/syft/file"  // 导入 file 包
	"github.com/anchore/syft/syft/linux"  // 导入 linux 包
	"github.com/anchore/syft/syft/pkg"  // 导入 pkg 包
	cpes "github.com/anchore/syft/syft/pkg/cataloger/common/cpe"  // 导入 cpe 目录下的包
)

// the source-rpm field has something akin to "util-linux-ng-2.17.2-12.28.el6_9.2.src.rpm"
// in which case the pattern will extract out the following values for the named capture groups:
//
```

// 使用正则表达式定义 RPM 包名的模式，包括名称、版本、发布版本和架构
var rpmPackageNamePattern = regexp.MustCompile(`^(?P<name>.*)-(?P<version>.*)-(?P<release>.*)\.(?P<arch>[a-zA-Z][^.]+)(\.rpm)$`)

// ID 表示每个添加到包集合中的包的唯一值
type ID string

// Package 表示已打包成可分发格式的应用程序或库
type Package struct {
	ID        ID
	Name      string           // 包名
	Version   string           // 包的版本
	Locations file.LocationSet // 导致发现此包的位置集合（注意：这不一定是构成此包的位置）
	Language  pkg.Language     // 此包所属的语言生态系统（例如 JavaScript、Python 等）
	Licenses  []string         // 许可证
	Type      pkg.Type         // 包类型（例如 Npm、Yarn、Python、Rpm、Deb 等）
	CPEs      []cpe.CPE        // 所有可能的通用平台枚举器
	PURL      string           // 包 URL（参见 https://github.com/package-url/purl-spec）
```
	// Upstreams 是一个 UpstreamPackage 类型的切片，用于存储包的上游信息
	Upstreams []UpstreamPackage
	// Metadata 是一个接口类型，用于存储与漏洞匹配相关的选择性数据，而不是完全对应 syft 元数据
	Metadata  interface{} 

}

func New(p pkg.Package) Package {
	// 从包中获取元数据和上游信息
	metadata, upstreams := dataFromPkg(p)

	// 获取包的许可证信息并转换为切片
	licenseObjs := p.Licenses.ToSlice()
	// 注意：这用于下游展示，并且是一个集合，因此应该始终分配空间
	licenses := make([]string, 0, len(licenseObjs))
	for _, l := range licenseObjs {
		licenses = append(licenses, l.Value)
	}
	// 如果许可证为空，则分配一个空切片
	if licenses == nil {
		licenses = []string{}
	}

	// 返回一个 Package 对象
	return Package{
		ID:        ID(p.ID()),
		Name:      p.Name,
		// 设置版本号为 p.Version
		Version:   p.Version,
		// 设置位置信息为 p.Locations
		Locations: p.Locations,
		// 设置许可证信息为 licenses
		Licenses:  licenses,
		// 设置语言信息为 p.Language
		Language:  p.Language,
		// 设置类型信息为 p.Type
		Type:      p.Type,
		// 设置 CPE 信息为 p.CPEs
		CPEs:      p.CPEs,
		// 设置 PURL 信息为 p.PURL
		PURL:      p.PURL,
		// 设置上游信息为 upstreams
		Upstreams: upstreams,
		// 设置元数据信息为 metadata
		Metadata:  metadata,
	}

}

func FromCollection(catalog *pkg.Collection, config SynthesisConfig) []Package {
	// 从包集合中获取已排序的包列表
	return FromPackages(catalog.Sorted(), config)
}

func FromPackages(syftpkgs []pkg.Package, config SynthesisConfig) []Package {
	var pkgs []Package
	var missingCPEs bool
	// 遍历每个包
	for _, p := range syftpkgs {
		if len(p.CPEs) == 0 {
			// 检查包的CPEs列表是否为空
			// 如果为空，根据配置决定是否生成缺失的CPEs
			if config.GenerateMissingCPEs {
				p.CPEs = cpes.Generate(p)
			} else {
				// 如果不生成缺失的CPEs，则记录日志并设置missingCPEs为true
				log.Debugf("no CPEs for package: %s", p)
				missingCPEs = true
			}
		}
		// 将包添加到pkgs列表中
		pkgs = append(pkgs, New(p))
	}
	// 如果有包缺失CPEs，则记录警告日志
	if missingCPEs {
		log.Warnf("some package(s) are missing CPEs. This may result in missing vulnerabilities. You may autogenerate these using: --add-cpes-if-none")
	}
	// 返回包列表
	return pkgs
}

// Stringer to represent a package.
// 用于表示一个包的字符串格式
func (p Package) String() string {
	return fmt.Sprintf("Pkg(type=%s, name=%s, version=%s, upstreams=%d)", p.Type, p.Name, p.Version, len(p.Upstreams))
	// 返回包的类型、名称、版本和上游数量的字符串表示形式
}
// 根据重叠关系移除包
func removePackagesByOverlap(catalog *pkg.Collection, relationships []artifact.Relationship, distro *linux.Release) *pkg.Collection {
	// 创建一个存储重叠关系的映射
	byOverlap := map[artifact.ID]artifact.Relationship{}
	// 遍历关系列表，将重叠关系存储到映射中
	for _, r := range relationships {
		if r.Type == artifact.OwnershipByFileOverlapRelationship {
			byOverlap[r.To.ID()] = r
		}
	}

	// 创建一个新的包集合
	out := pkg.NewCollection()
	// 检查发行版是否是全面的
	comprehensiveDistroFeed := distroFeedIsComprehensive(distro)
	// 遍历目录中的包
	for p := range catalog.Enumerate() {
		// 检查是否存在重叠关系
		r, ok := byOverlap[p.ID()]
		if ok {
			// 如果存在重叠关系，检查是否需要排除该包
			from, ok := r.From.(pkg.Package)
			if ok && excludePackage(comprehensiveDistroFeed, p, from) {
				continue
			}
		}
		out.Add(p)
	}

	return out
}

func excludePackage(comprehensiveDistroFeed bool, p pkg.Package, parent pkg.Package) bool {
	// NOTE: we are not checking the name because we have mismatches like:
	// python      3.9.2      binary
	// python3.9   3.9.2-1    deb

	// 如果版本号不完全相同，则保留两个包
	if !strings.HasPrefix(parent.Version, p.Version) {
		return false
	}

	// 如果父包是操作系统包而子包不是，则在有全面的软件包 feed 的发行版中排除子包。
	// 也就是说，在列出未修复漏洞的发行版中排除子包。否则，子包可能需要用于匹配。
// 如果是综合发行版的软件源，并且父级软件包是操作系统包，而当前软件包不是操作系统包，则返回 true
if comprehensiveDistroFeed && isOSPackage(parent) && !isOSPackage(p) {
    return true
}

// 过滤掉二进制软件包，即使对于非综合发行版也是如此
if p.Type != pkg.BinaryPkg {
    return false
}

return true
}

// distroFeedIsComprehensive 如果发行版的软件源足够全面，我们可以在匹配之前删除发行版软件包的软件包，则返回 true
func distroFeedIsComprehensive(distro *linux.Release) bool {
    // TODO: 一旦 https://github.com/anchore/grype/issues/1426 得到解决，应重新审视这个机制
    if distro == nil {
        return false
	}
	if distro.ID == "amzn" {
		// 如果发行版是 AmazonLinux，则返回 false，因为它虽然显示为 "like rhel"，但不是 rhel 的克隆，也没有详尽的漏洞信息。
		return false
	}
	// 遍历 comprehensiveDistros 列表
	for _, d := range comprehensiveDistros {
		// 如果 distro.ID 与 comprehensiveDistros 中的某个元素相等，则返回 true
		if strings.EqualFold(d, distro.ID) {
			return true
		}
		// 遍历 distro.IDLike 列表
		for _, n := range distro.IDLike {
			// 如果 distro.IDLike 中的元素与 comprehensiveDistros 中的某个元素相等，则返回 true
			if strings.EqualFold(d, n) {
				return true
			}
		}
	}
	// 如果以上条件都不满足，则返回 false
	return false
}

// computed by:
// 从数据库中查询漏洞信息，筛选出修复状态为"不修复"或"未修复"的漏洞所属的命名空间，并按命名空间排序
// 然后去除'github'，将'redhat'替换为'rhel'
var comprehensiveDistros = []string{
    "debian",
    "mariner",
    "rhel",
    "ubuntu",
}

// 判断包是否为操作系统的软件包
func isOSPackage(p pkg.Package) bool {
    switch p.Type {
    case pkg.DebPkg, pkg.RpmPkg, pkg.PortagePkg, pkg.AlpmPkg, pkg.ApkPkg:
        return true
    default:
        return false
    }
}

// 从软件包中获取元数据和上游软件包列表
func dataFromPkg(p pkg.Package) (interface{}, []UpstreamPackage) {
    var metadata interface{}
# 定义一个空的 UpstreamPackage 切片
var upstreams []UpstreamPackage

# 根据 p.Metadata 的类型进行不同的处理
switch p.Metadata.(type) {
    # 如果是 GolangModuleEntry 或者 GolangBinaryBuildinfoEntry 类型，则调用 golangMetadataFromPkg 函数处理
    case pkg.GolangModuleEntry, pkg.GolangBinaryBuildinfoEntry:
        metadata = golangMetadataFromPkg(p)
    
    # 如果是 DpkgDBEntry 类型，则调用 dpkgDataFromPkg 函数处理，并将结果赋值给 upstreams
    case pkg.DpkgDBEntry:
        upstreams = dpkgDataFromPkg(p)
    
    # 如果是 RpmArchive 或者 RpmDBEntry 类型，则调用 rpmDataFromPkg 函数处理，并将结果赋值给 metadata 和 upstreams
    case pkg.RpmArchive, pkg.RpmDBEntry:
        m, u := rpmDataFromPkg(p)
        upstreams = u
        if m != nil {
            metadata = *m
        }
    
    # 如果是 JavaArchive 类型，则调用 javaDataFromPkg 函数处理，并将结果赋值给 metadata
    case pkg.JavaArchive:
        if m := javaDataFromPkg(p); m != nil {
            metadata = *m
        }
    
    # 如果是 ApkDBEntry 类型，则先调用 apkMetadataFromPkg 函数处理，并将结果赋值给 metadata，然后调用 apkDataFromPkg 函数处理，并将结果赋值给 upstreams
    case pkg.ApkDBEntry:
        metadata = apkMetadataFromPkg(p)
        upstreams = apkDataFromPkg(p)
}
# 从包对象中提取 APK 元数据
func apkMetadataFromPkg(p pkg.Package) interface{} {
    # 检查包对象的元数据是否为 ApkDBEntry 类型
    if m, ok := p.Metadata.(pkg.ApkDBEntry); ok {
        # 创建 ApkMetadata 结构体
        metadata := ApkMetadata{}

        # 创建一个空的 ApkFileRecord 切片，预留足够的空间
        fileRecords := make([]ApkFileRecord, 0, len(m.Files))
        
        # 遍历包对象中的文件记录，将其转换为 ApkFileRecord 结构体并添加到切片中
        for _, record := range m.Files {
            r := ApkFileRecord{Path: record.Path}
            fileRecords = append(fileRecords, r)
        }

        # 将文件记录切片赋值给 ApkMetadata 结构体的 Files 字段
        metadata.Files = fileRecords

        # 返回 ApkMetadata 结构体
        return metadata
    }

    # 如果包对象的元数据不是 ApkDBEntry 类型，则返回空值
    return nil
}
// 从给定的包中提取 Golang 元数据
func golangMetadataFromPkg(p pkg.Package) interface{} {
    // 根据包的元数据类型进行不同的处理
    switch value := p.Metadata.(type) {
    // 如果是 Golang 二进制构建信息条目
    case pkg.GolangBinaryBuildinfoEntry:
        // 创建 Golang 二进制元数据对象
        metadata := GolangBinMetadata{}
        // 如果构建设置不为空，则赋值给元数据对象
        if value.BuildSettings != nil {
            metadata.BuildSettings = value.BuildSettings
        }
        // 赋值其他属性
        metadata.GoCompiledVersion = value.GoCompiledVersion
        metadata.Architecture = value.Architecture
        metadata.H1Digest = value.H1Digest
        metadata.MainModule = value.MainModule
        // 返回 Golang 二进制元数据对象
        return metadata
    // 如果是 Golang 模块条目
    case pkg.GolangModuleEntry:
        // 创建 Golang 模块元数据对象
        metadata := GolangModMetadata{}
        // 赋值摘要属性
        metadata.H1Digest = value.H1Digest
        // 返回 Golang 模块元数据对象
        return metadata
    }
    // 默认情况下返回空
    return nil
}
}

// 从包中提取 dpkg 数据并返回上游包列表
func dpkgDataFromPkg(p pkg.Package) (upstreams []UpstreamPackage) {
	// 检查包的元数据是否为 dpkg.DpkgDBEntry 类型
	if value, ok := p.Metadata.(pkg.DpkgDBEntry); ok {
		// 如果包的源不为空，则将其添加到上游包列表中
		if value.Source != "" {
			upstreams = append(upstreams, UpstreamPackage{
				Name:    value.Source,
				Version: value.SourceVersion,
			})
		}
	} else {
		// 如果无法提取 dpkg 元数据，则记录警告信息
		log.Warnf("unable to extract DPKG metadata for %s", p)
	}
	// 返回上游包列表
	return upstreams
}

// 从包中提取 rpm 数据并返回 rpm 元数据和上游包列表
func rpmDataFromPkg(p pkg.Package) (metadata *RpmMetadata, upstreams []UpstreamPackage) {
	// 根据包的元数据类型进行不同的处理
	switch m := p.Metadata.(type) {
	case pkg.RpmDBEntry:
		// 如果包的源 RPM 不为空，则将其添加到上游包列表中
		if m.SourceRpm != "" {
# 根据包类型处理源码包和 RPM 包
switch p.Type {
case pkg.RpmPackage:
    # 如果 RPM 包的源码包不为空，处理源码包
    if m.SourceRpm != "" {
        upstreams = handleSourceRPM(p.Name, m.SourceRpm)
    }
    # 创建 RPM 包的元数据对象
    metadata = &RpmMetadata{
        Epoch:           m.Epoch,
        ModularityLabel: m.ModularityLabel,
    }
case pkg.RpmArchive:
    # 如果 RPM 存档的源码包不为空，处理源码包
    if m.SourceRpm != "" {
        upstreams = handleSourceRPM(p.Name, m.SourceRpm)
    }
    # 创建 RPM 存档的元数据对象
    metadata = &RpmMetadata{
        Epoch:           m.Epoch,
        ModularityLabel: m.ModularityLabel,
    }
}
# 返回元数据对象和处理后的源码包
return metadata, upstreams
# 处理源码包的函数，接受包名和源码包路径作为参数，返回一个UpstreamPackage类型的切片
func handleSourceRPM(pkgName, sourceRpm string) []UpstreamPackage {
    var upstreams []UpstreamPackage  # 声明一个UpstreamPackage类型的切片
    name, version := getNameAndELVersion(sourceRpm)  # 调用getNameAndELVersion函数获取源码包的名称和版本
    if name == "" && version == "" {  # 如果无法从源码包中提取名称和版本
        log.Warnf("unable to extract name and version from SourceRPM=%q ", sourceRpm)  # 记录警告日志
    } else if name != pkgName {  # 如果提取到的名称不等于包名
        # 如果提取到的名称和版本都不为空
        if name != "" && version != "" {
            upstreams = append(upstreams,  # 将提取到的名称和版本添加到upstreams切片中
                UpstreamPackage{
                    Name:    name,
                    Version: version,
                },
            )
        }
    }
    return upstreams  # 返回upstreams切片
}

# 获取源码包的名称和版本
func getNameAndELVersion(sourceRpm string) (string, string) {
// 从源 RPM 包名中匹配捕获组
groupMatches := stringutil.MatchCaptureGroups(rpmPackageNamePattern, sourceRpm)
// 将版本和发布信息组合成一个字符串
version := groupMatches["version"] + "-" + groupMatches["release"]
// 返回软件包名和版本信息
return groupMatches["name"], version
}

// 从软件包中提取 Java 元数据
func javaDataFromPkg(p pkg.Package) (metadata *JavaMetadata) {
	// 判断软件包的元数据是否为 Java 归档
	if value, ok := p.Metadata.(pkg.JavaArchive); ok {
		var artifactID, groupID, name string
		// 如果存在 POM 属性，则提取 artifactID 和 groupID
		if value.PomProperties != nil {
			artifactID = value.PomProperties.ArtifactID
			groupID = value.PomProperties.GroupID
		}
		// 如果存在清单文件，则提取名称
		if value.Manifest != nil {
			if n, ok := value.Manifest.Main["Name"]; ok {
				name = n
			}
		}

		// 如果存在归档摘要，则提取摘要信息
		var archiveDigests []Digest
		if len(value.ArchiveDigests) > 0 {
# 遍历 value.ArchiveDigests 切片中的元素
for _, d := range value.ArchiveDigests {
    # 将 d.Algorithm 和 d.Value 组成 Digest 结构体，并添加到 archiveDigests 切片中
    archiveDigests = append(archiveDigests, Digest{
        Algorithm: d.Algorithm,
        Value:     d.Value,
    })
}

# 如果条件成立，创建 JavaMetadata 结构体并赋值给 metadata
metadata = &JavaMetadata{
    VirtualPath:    value.VirtualPath,
    PomArtifactID:  artifactID,
    PomGroupID:     groupID,
    ManifestName:   name,
    ArchiveDigests: archiveDigests,
}
# 如果条件不成立，记录警告日志
else {
    log.Warnf("unable to extract Java metadata for %s", p)
}
# 返回 metadata
return metadata
# 从包对象中提取 APK 数据，并返回上游包列表
func apkDataFromPkg(p pkg.Package) (upstreams []UpstreamPackage) {
    # 检查包的元数据是否为 ApkDBEntry 类型
    if value, ok := p.Metadata.(pkg.ApkDBEntry); ok {
        # 如果原始包名不为空，则将其添加到上游包列表中
        if value.OriginPackage != "" {
            upstreams = append(upstreams, UpstreamPackage{
                Name: value.OriginPackage,
            })
        }
    } else {
        # 如果无法提取 APK 元数据，则记录警告信息
        log.Warnf("unable to extract APK metadata for %s", p)
    }
    # 返回上游包列表
    return upstreams
}

# 根据 ID 查找包对象
func ByID(id ID, pkgs []Package) *Package {
    # 遍历包列表，查找与给定 ID 匹配的包对象
    for _, p := range pkgs {
        if p.ID == id {
            # 返回匹配的包对象的指针
            return &p
        }
    }
}
# 返回空值，表示没有返回任何有效的数据。
```