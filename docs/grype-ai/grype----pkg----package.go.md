# `grype\grype\pkg\package.go`

```
// 声明包名为 pkg，引入所需的包
package pkg

// 引入所需的包
import (
    "fmt" // 格式化输入输出
    "regexp" // 正则表达式
    "strings" // 字符串操作

    "github.com/anchore/grype/internal/log" // 引入日志包
    "github.com/anchore/grype/internal/stringutil" // 引入字符串工具包
    "github.com/anchore/syft/syft/artifact" // 引入 artifact 包
    "github.com/anchore/syft/syft/cpe" // 引入 cpe 包
    "github.com/anchore/syft/syft/file" // 引入 file 包
    "github.com/anchore/syft/syft/linux" // 引入 linux 包
    "github.com/anchore/syft/syft/pkg" // 引入 pkg 包
    cpes "github.com/anchore/syft/syft/pkg/cataloger/common/cpe" // 引入 cpes 包
)

// 定义正则表达式，用于提取 RPM 包名的各个部分
var rpmPackageNamePattern = regexp.MustCompile(`^(?P<name>.*)-(?P<version>.*)-(?P<release>.*)\.(?P<arch>[a-zA-Z][^.]+)(\.rpm)$`)

// 定义 ID 类型，表示包集合中每个包的唯一值
type ID string

// 定义 Package 结构体，表示一个被打包成可分发格式的应用程序或库
type Package struct {
    ID        ID
    Name      string           // 包名
    Version   string           // 包版本
    Locations file.LocationSet // 导致发现此包的位置集合（注意：这不一定是构成此包的位置）
    Language  pkg.Language     // 此包所属的语言生态系统（例如 JavaScript、Python 等）
    Licenses  []string         // 许可证
    Type      pkg.Type         // 包类型（例如 Npm、Yarn、Python、Rpm、Deb 等）
    CPEs      []cpe.CPE        // 所有可能的通用平台枚举器
    PURL      string           // 包 URL（参见 https://github.com/package-url/purl-spec）
    Upstreams []UpstreamPackage
    Metadata  interface{}      // 这不是与 syft 元数据一一对应的！只包含用于匹配漏洞的选择数据
}

// New 函数，用于创建 Package 结构体
func New(p pkg.Package) Package {
    // 从包中获取元数据和上游信息
    metadata, upstreams := dataFromPkg(p)

    // 将许可证对象转换为切片
    licenseObjs := p.Licenses.ToSlice()
    // 注意：这用于向下游展示，并且是一个集合，因此应始终分配空间
    licenses := make([]string, 0, len(licenseObjs))
    // 遍历许可证对象，将值添加到许可证切片中
    for _, l := range licenseObjs {
        licenses = append(licenses, l.Value)
    }
    // 如果许可证为空，则分配一个空切片
    if licenses == nil {
        licenses = []string{}
    }

    // 返回一个包对象
    return Package{
        ID:        ID(p.ID()),
        Name:      p.Name,
        Version:   p.Version,
        Locations: p.Locations,
        Licenses:  licenses,
        Language:  p.Language,
        Type:      p.Type,
        CPEs:      p.CPEs,
        PURL:      p.PURL,
        Upstreams: upstreams,
        Metadata:  metadata,
    }
// 从给定的 pkg.Collection 和 SynthesisConfig 中生成 Package 数组
func FromCollection(catalog *pkg.Collection, config SynthesisConfig) []Package {
    // 调用 FromPackages 函数，传入已排序的包列表和配置信息，返回 Package 数组
    return FromPackages(catalog.Sorted(), config)
}

// 从给定的 pkg.Package 数组和 SynthesisConfig 中生成 Package 数组
func FromPackages(syftpkgs []pkg.Package, config SynthesisConfig) []Package {
    // 初始化 Package 数组
    var pkgs []Package
    // 初始化标记缺失 CPEs 的布尔值
    var missingCPEs bool
    // 遍历给定的包列表
    for _, p := range syftpkgs {
        // 如果包的 CPEs 为空
        if len(p.CPEs) == 0 {
            // 如果配置允许生成缺失的 CPEs
            if config.GenerateMissingCPEs {
                // 生成缺失的 CPEs
                p.CPEs = cpes.Generate(p)
            } else {
                // 记录缺失 CPEs 的包信息，并设置标记为 true
                log.Debugf("no CPEs for package: %s", p)
                missingCPEs = true
            }
        }
        // 将包转换为 Package 类型并添加到 pkgs 数组中
        pkgs = append(pkgs, New(p))
    }
    // 如果存在缺失 CPEs 的包，输出警告信息
    if missingCPEs {
        log.Warnf("some package(s) are missing CPEs. This may result in missing vulnerabilities. You may autogenerate these using: --add-cpes-if-none")
    }
    // 返回生成的 Package 数组
    return pkgs
}

// 用于表示一个包的 Stringer
func (p Package) String() string {
    // 返回包的字符串表示形式
    return fmt.Sprintf("Pkg(type=%s, name=%s, version=%s, upstreams=%d)", p.Type, p.Name, p.Version, len(p.Upstreams))
}

// 通过重叠关系移除包
func removePackagesByOverlap(catalog *pkg.Collection, relationships []artifact.Relationship, distro *linux.Release) *pkg.Collection {
    // 创建一个映射，用于存储重叠关系
    byOverlap := map[artifact.ID]artifact.Relationship{}
    // 遍历关系列表
    for _, r := range relationships {
        // 如果关系类型为 OwnershipByFileOverlapRelationship
        if r.Type == artifact.OwnershipByFileOverlapRelationship {
            // 将关系添加到映射中
            byOverlap[r.To.ID()] = r
        }
    }

    // 创建一个新的包集合
    out := pkg.NewCollection()
    // 检查 distroFeed 是否全面
    comprehensiveDistroFeed := distroFeedIsComprehensive(distro)
    // 遍历包集合
    for p := range catalog.Enumerate() {
        // 从映射中查找包的重叠关系
        r, ok := byOverlap[p.ID()]
        // 如果存在重叠关系
        if ok {
            // 尝试将关系的来源转换为 pkg.Package 类型
            from, ok := r.From.(pkg.Package)
            // 如果成功转换，并且需要排除该包
            if ok && excludePackage(comprehensiveDistroFeed, p, from) {
                // 继续下一次循环
                continue
            }
        }
        // 将包添加到新的包集合中
        out.Add(p)
    }

    // 返回移除重叠包后的包集合
    return out
}

// 排除包的函数
func excludePackage(comprehensiveDistroFeed bool, p pkg.Package, parent pkg.Package) bool {
    // 注意：这里没有检查包的名称，因为存在不匹配的情况
    // 返回是否需要排除该包
    return comprehensiveDistroFeed
}
    // 检查版本是否有效，如果不是，则保留两个版本
    if !strings.HasPrefix(parent.Version, p.Version) {
        return false
    }

    // 如果父包是操作系统包而子包不是，则在具有全面反馈的发行版中排除子包
    // 也就是说，列出未修复的漏洞的发行版。否则，子包可能需要用于匹配
    if comprehensiveDistroFeed && isOSPackage(parent) && !isOSPackage(p) {
        return true
    }

    // 过滤掉二进制包，即使对于非全面反馈的发行版也是如此
    if p.Type != pkg.BinaryPkg {
        return false
    }

    return true
// distroFeedIsComprehensive函数用于检查发行版的feed是否足够全面，以便在匹配之前丢弃发行版软件包的所有权软件包
func distroFeedIsComprehensive(distro *linux.Release) bool {
    // TODO: 一旦https://github.com/anchore/grype/issues/1426得到解决，应重新审视这个机制
    if distro == nil {
        return false
    }
    if distro.ID == "amzn" {
        // AmazonLinux显示"like rhel"，但不是rhel克隆，也没有详尽的漏洞feed
        return false
    }
    for _, d := range comprehensiveDistros {
        if strings.EqualFold(d, distro.ID) {
            return true
        }
        for _, n := range distro.IDLike {
            if strings.EqualFold(d, n) {
                return true
            }
        }
    }
    return false
}

// 通过以下方式计算：
// sqlite3 vulnerability.db 'select distinct namespace from vulnerability where fix_state in ("wont-fix", "not-fixed") order by namespace;' | cut -d ':' -f 1 | sort | uniq
// 然后删除'github'，将'redhat'替换为'rhel'
var comprehensiveDistros = []string{
    "debian",
    "mariner",
    "rhel",
    "ubuntu",
}

// isOSPackage函数用于检查软件包是否是操作系统软件包
func isOSPackage(p pkg.Package) bool {
    switch p.Type {
    case pkg.DebPkg, pkg.RpmPkg, pkg.PortagePkg, pkg.AlpmPkg, pkg.ApkPkg:
        return true
    default:
        return false
    }
}

// dataFromPkg函数用于从软件包中获取数据
func dataFromPkg(p pkg.Package) (interface{}, []UpstreamPackage) {
    var metadata interface{}
    var upstreams []UpstreamPackage

    switch p.Metadata.(type) {
    case pkg.GolangModuleEntry, pkg.GolangBinaryBuildinfoEntry:
        metadata = golangMetadataFromPkg(p)
    case pkg.DpkgDBEntry:
        upstreams = dpkgDataFromPkg(p)
    case pkg.RpmArchive, pkg.RpmDBEntry:
        m, u := rpmDataFromPkg(p)
        upstreams = u
        if m != nil {
            metadata = *m
        }
    # 如果包类型是Java存档
    case pkg.JavaArchive:
        # 如果从包中获取到Java数据，则将其赋值给metadata
        if m := javaDataFromPkg(p); m != nil:
            metadata = *m
    # 如果包类型是Apk数据库条目
    case pkg.ApkDBEntry:
        # 从包中获取Apk元数据并赋值给metadata
        metadata = apkMetadataFromPkg(p)
        # 从包中获取Apk数据并赋值给upstreams
        upstreams = apkDataFromPkg(p)
    # 返回metadata和upstreams
    }
    return metadata, upstreams
}
# 闭合函数的大括号

func apkMetadataFromPkg(p pkg.Package) interface{} {
    # 检查包的元数据是否为 ApkDBEntry 类型
    if m, ok := p.Metadata.(pkg.ApkDBEntry); ok {
        # 创建 ApkMetadata 结构体
        metadata := ApkMetadata{}

        # 创建 ApkFileRecord 切片
        fileRecords := make([]ApkFileRecord, 0, len(m.Files))
        # 遍历包的文件记录，创建 ApkFileRecord 结构体并添加到切片中
        for _, record := range m.Files {
            r := ApkFileRecord{Path: record.Path}
            fileRecords = append(fileRecords, r)
        }

        # 将文件记录切片赋值给 metadata 的 Files 字段
        metadata.Files = fileRecords

        # 返回 metadata
        return metadata
    }

    # 如果包的元数据不是 ApkDBEntry 类型，则返回 nil
    return nil
}

func golangMetadataFromPkg(p pkg.Package) interface{} {
    # 根据包的元数据类型进行不同的处理
    switch value := p.Metadata.(type) {
    case pkg.GolangBinaryBuildinfoEntry:
        # 创建 GolangBinMetadata 结构体
        metadata := GolangBinMetadata{}
        # 如果 BuildSettings 不为 nil，则赋值给 metadata 的 BuildSettings 字段
        if value.BuildSettings != nil {
            metadata.BuildSettings = value.BuildSettings
        }
        # 赋值其他字段
        metadata.GoCompiledVersion = value.GoCompiledVersion
        metadata.Architecture = value.Architecture
        metadata.H1Digest = value.H1Digest
        metadata.MainModule = value.MainModule
        # 返回 metadata
        return metadata
    case pkg.GolangModuleEntry:
        # 创建 GolangModMetadata 结构体
        metadata := GolangModMetadata{}
        # 赋值字段
        metadata.H1Digest = value.H1Digest
        # 返回 metadata
        return metadata
    }
    # 如果包的元数据类型不匹配，则返回 nil
    return nil
}

func dpkgDataFromPkg(p pkg.Package) (upstreams []UpstreamPackage) {
    # 检查包的元数据是否为 DpkgDBEntry 类型
    if value, ok := p.Metadata.(pkg.DpkgDBEntry); ok {
        # 如果元数据中的 Source 字段不为空，则创建 UpstreamPackage 结构体并添加到 upstreams 切片中
        if value.Source != "" {
            upstreams = append(upstreams, UpstreamPackage{
                Name:    value.Source,
                Version: value.SourceVersion,
            })
        }
    } else {
        # 如果包的元数据类型不匹配，则记录警告日志
        log.Warnf("unable to extract DPKG metadata for %s", p)
    }
    # 返回 upstreams
    return upstreams
}

func rpmDataFromPkg(p pkg.Package) (metadata *RpmMetadata, upstreams []UpstreamPackage) {
    # 根据包的元数据类型进行不同的处理
    switch m := p.Metadata.(type) {
    case pkg.RpmDBEntry:
        # 如果元数据中的 SourceRpm 字段不为空，则调用 handleSourceRPM 函数处理，并赋值给 upstreams
        if m.SourceRpm != "" {
            upstreams = handleSourceRPM(p.Name, m.SourceRpm)
        }

        # 创建 RpmMetadata 结构体指针，并赋值字段
        metadata = &RpmMetadata{
            Epoch:           m.Epoch,
            ModularityLabel: m.ModularityLabel,
        }
    # 如果包类型是RPM存档
    case pkg.RpmArchive:
        # 如果源RPM不为空，则处理源RPM并返回上游信息
        if m.SourceRpm != "":
            upstreams = handleSourceRPM(p.Name, m.SourceRpm)

        # 创建RPM元数据对象并赋值
        metadata = &RpmMetadata{
            Epoch:           m.Epoch,
            ModularityLabel: m.ModularityLabel,
        }
    }
    # 返回元数据和上游信息
    return metadata, upstreams
}

// 根据源 RPM 包名和源 RPM 文件路径处理并返回上游包列表
func handleSourceRPM(pkgName, sourceRpm string) []UpstreamPackage {
    var upstreams []UpstreamPackage
    // 从源 RPM 文件路径中提取包名和版本
    name, version := getNameAndELVersion(sourceRpm)
    // 如果无法提取包名和版本，则记录警告日志
    if name == "" && version == "" {
        log.Warnf("unable to extract name and version from SourceRPM=%q ", sourceRpm)
    } else if name != pkgName {
        // 如果包名不匹配当前包名，则将包名和版本添加到上游包列表中
        if name != "" && version != "" {
            upstreams = append(upstreams,
                UpstreamPackage{
                    Name:    name,
                    Version: version,
                },
            )
        }
    }
    return upstreams
}

// 从源 RPM 文件路径中提取包名和 EL 版本
func getNameAndELVersion(sourceRpm string) (string, string) {
    groupMatches := stringutil.MatchCaptureGroups(rpmPackageNamePattern, sourceRpm)
    version := groupMatches["version"] + "-" + groupMatches["release"]
    return groupMatches["name"], version
}

// 从包对象中提取 Java 元数据
func javaDataFromPkg(p pkg.Package) (metadata *JavaMetadata) {
    if value, ok := p.Metadata.(pkg.JavaArchive); ok {
        var artifactID, groupID, name string
        // 如果存在 POM 属性，则提取 artifactID 和 groupID
        if value.PomProperties != nil {
            artifactID = value.PomProperties.ArtifactID
            groupID = value.PomProperties.GroupID
        }
        // 如果存在 MANIFEST，则提取 Name
        if value.Manifest != nil {
            if n, ok := value.Manifest.Main["Name"]; ok {
                name = n
            }
        }

        var archiveDigests []Digest
        // 如果存在归档摘要，则提取算法和值
        if len(value.ArchiveDigests) > 0 {
            for _, d := range value.ArchiveDigests {
                archiveDigests = append(archiveDigests, Digest{
                    Algorithm: d.Algorithm,
                    Value:     d.Value,
                })
            }
        }

        // 构建 Java 元数据对象
        metadata = &JavaMetadata{
            VirtualPath:    value.VirtualPath,
            PomArtifactID:  artifactID,
            PomGroupID:     groupID,
            ManifestName:   name,
            ArchiveDigests: archiveDigests,
        }
    } else {
        # 如果无法提取Java元数据，则记录警告信息
        log.Warnf("unable to extract Java metadata for %s", p)
    }
    # 返回元数据
    return metadata
# 从给定的包对象中提取 APK 数据，并返回上游包的列表
func apkDataFromPkg(p pkg.Package) (upstreams []UpstreamPackage) {
    # 检查包的元数据是否为 ApkDBEntry 类型
    if value, ok := p.Metadata.(pkg.ApkDBEntry); ok {
        # 如果包的原始包名不为空，则将其添加到上游包列表中
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

# 根据 ID 查找包对象，并返回指向该包对象的指针
func ByID(id ID, pkgs []Package) *Package {
    # 遍历包列表
    for _, p := range pkgs {
        # 如果找到与给定 ID 匹配的包，则返回指向该包的指针
        if p.ID == id {
            return &p
        }
    }
    # 如果未找到匹配的包，则返回空指针
    return nil
}
```