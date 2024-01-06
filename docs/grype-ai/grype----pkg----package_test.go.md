# `grype\grype\pkg\package_test.go`

```
package pkg
// 声明包名为pkg，用于组织代码

import (
	"fmt"
	"strings"
	"testing"

	"github.com/stretchr/testify/assert"
	// 导入第三方测试断言库

	"github.com/anchore/syft/syft/artifact"
	"github.com/anchore/syft/syft/cpe"
	"github.com/anchore/syft/syft/file"
	syftFile "github.com/anchore/syft/syft/file"
	"github.com/anchore/syft/syft/linux"
	syftPkg "github.com/anchore/syft/syft/pkg"
	"github.com/anchore/syft/syft/sbom"
	"github.com/anchore/syft/syft/testutil"
	// 导入其他自定义包和第三方包
)

func TestNew(t *testing.T) {
// 定义测试函数TestNew，用于测试New函数
// 定义一个测试用例切片，每个测试用例包含名称、syftPkg.Package对象、元数据和上游包列表
tests := []struct {
    name      string
    syftPkg   syftPkg.Package
    metadata  interface{}
    upstreams []UpstreamPackage
}{
    // 第一个测试用例
    {
        name: "alpm package with source info",
        // 设置syftPkg.Package对象的元数据
        syftPkg: syftPkg.Package{
            Metadata: syftPkg.AlpmDBEntry{
                BasePackage:  "base-pkg-info",
                Package:      "pkg-info",
                Version:      "version-info",
                Architecture: "arch-info",
                Files: []syftPkg.AlpmFileRecord{{
                    Path: "/this/path/exists",
                }},
            },
        },
    },
    // 其他测试用例...
}
		{
			# 定义一个名为 "dpkg with source info" 的包
			name: "dpkg with source info",
			# 设置包的元数据
			syftPkg: syftPkg.Package{
				# 设置 dpkg 包的元数据
				Metadata: syftPkg.DpkgDBEntry{
					# 包名为 "pkg-info"
					Package:       "pkg-info",
					# 包的源信息为 "src-info"
					Source:        "src-info",
					# 包的版本信息为 "version-info"
					Version:       "version-info",
					# 包的源版本信息为 "src-version-info"
					SourceVersion: "src-version-info",
					# 包的架构信息为 "arch-info"
					Architecture:  "arch-info",
					# 包的维护者信息为 "maintainer-info"
					Maintainer:    "maintainer-info",
					# 包的安装大小为 10
					InstalledSize: 10,
					# 包含文件信息的列表
					Files: []syftPkg.DpkgFileRecord{
						{
							# 文件路径为 "path-info"
							Path: "path-info",
							# 文件的摘要信息
							Digest: &file.Digest{
								# 摘要算法为 "algo-info"
								Algorithm: "algo-info",
								# 摘要值为 "digest-info"
								Value:     "digest-info",
							},
							# 文件是否为配置文件
							IsConfigFile: true,
						},
# 定义一个包含 RPM 包信息的结构体
{
    name: "rpm archive with source info",  # RPM 包的名称
    syftPkg: syftPkg.Package{  # 使用 syftPkg 包的 Package 结构体
        Metadata: syftPkg.RpmArchive{  # 使用 syftPkg 包的 RpmArchive 结构体
            Name:      "name-info",  # RPM 包的名称信息
            Version:   "version-info",  # RPM 包的版本信息
            Epoch:     intRef(30),  # RPM 包的时代信息
            Arch:      "arch-info",  # RPM 包的架构信息
            Release:   "release-info",  # RPM 包的发布信息
            SourceRpm: "sqlite-3.26.0-6.el8.src.rpm",  # RPM 包的源码 RPM 信息
# 设置包的大小为40
Size:      40,
# 设置包的供应商为"vendor-info"
Vendor:    "vendor-info",
# 设置包的文件信息
Files: []syftPkg.RpmFileRecord{
    # 设置文件路径为"path-info"
    {
        Path: "path-info",
        # 设置文件的权限为20
        Mode: 20,
        # 设置文件的大小为10
        Size: 10,
        # 设置文件的摘要信息
        Digest: file.Digest{
            # 设置摘要算法为"algo-info"
            Algorithm: "algo-info",
            # 设置摘要值为"digest-info"
            Value:     "digest-info",
        },
        # 设置文件的所有者用户名为"user-info"
        UserName:  "user-info",
        # 设置文件的所有者组名为"group-info"
        GroupName: "group-info",
        # 设置文件的标志为"flag-info"
        Flags:     "flag-info",
    },
},
# 设置包的元数据
metadata: RpmMetadata{
    # 设置包的时代为30
    Epoch: intRef(30),
		},
		// 定义一个空的 upstreams 切片
		upstreams: []UpstreamPackage{
			// 添加一个 UpstreamPackage 结构体实例到 upstreams 切片中
			{
				// 设置包的名称为 "sqlite"
				Name:    "sqlite",
				// 设置包的版本为 "3.26.0-6.el8"
				Version: "3.26.0-6.el8",
			},
		},
	},
	// 定义一个新的 Package 结构体实例
	{
		// 设置包的名称为 "rpm db entry with source info"
		name: "rpm db entry with source info",
		// 设置包的 syftPkg 字段为 Package 结构体实例
		syftPkg: syftPkg.Package{
			// 设置 Metadata 字段为 RpmDBEntry 结构体实例
			Metadata: syftPkg.RpmDBEntry{
				// 设置包的名称为 "name-info"
				Name:      "name-info",
				// 设置包的版本为 "version-info"
				Version:   "version-info",
				// 设置包的 Epoch 为 30
				Epoch:     intRef(30),
				// 设置包的架构为 "arch-info"
				Arch:      "arch-info",
				// 设置包的发布版本为 "release-info"
				Release:   "release-info",
				// 设置包的源 RPM 为 "sqlite-3.26.0-6.el8.src.rpm"
				SourceRpm: "sqlite-3.26.0-6.el8.src.rpm",
				// 设置包的大小为 40
				Size:      40,
				// 设置包的供应商为 "vendor-info"
				Vendor:    "vendor-info",
# 定义一个名为Files的列表，包含syftPkg.RpmFileRecord对象
Files: []syftPkg.RpmFileRecord{
    # 定义一个syftPkg.RpmFileRecord对象，包含文件路径、模式、大小、摘要、用户名、组名和标志
    {
        Path: "path-info",
        Mode: 20,
        Size: 10,
        Digest: file.Digest{
            Algorithm: "algo-info",
            Value: "digest-info",
        },
        UserName: "user-info",
        GroupName: "group-info",
        Flags: "flag-info",
    },
},
# 定义metadata对象，包含Epoch字段
metadata: RpmMetadata{
    Epoch: intRef(30),
},
# 定义upstreams列表，包含UpstreamPackage对象
upstreams: []UpstreamPackage{
# 创建一个包含名称和版本信息的字典
{
    Name:    "sqlite",
    Version: "3.26.0-6.el8",
},
# 创建一个包含名称和源信息的 RPM 存档元数据对象
{
    name: "rpm archive with source info that matches the package info",
    syftPkg: syftPkg.Package{
        Name: "sqlite",
        Metadata: syftPkg.RpmArchive{
            SourceRpm: "sqlite-3.26.0-6.el8.src.rpm",
        },
    },
    metadata: RpmMetadata{},
},
# 创建一个包含名称的 RPM 存档元数据对象
{
    name: "rpm archive with modularity label",
    syftPkg: syftPkg.Package{
        Name: "sqlite",
# 创建一个名为 Metadata 的结构体，其中包含了 RPM 包的元数据信息
Metadata: syftPkg.RpmArchive{
    # 设置 RPM 包的源 RPM
    SourceRpm:       "sqlite-3.26.0-6.el8.src.rpm",
    # 设置 RPM 包的模块化标签
    ModularityLabel: "abc:2",
},
# 创建一个名为 metadata 的结构体，其中包含了 RPM 包的元数据信息，包括模块化标签
metadata: RpmMetadata{ModularityLabel: "abc:2"},
# 创建一个名为 syftPkg 的结构体，其中包含了 Java 包的信息
syftPkg: syftPkg.Package{
    # 创建一个名为 Metadata 的结构体，其中包含了 Java 包的元数据信息
    Metadata: syftPkg.JavaArchive{
        # 设置 Java 包的虚拟路径信息
        VirtualPath: "virtual-path-info",
        # 创建一个名为 Manifest 的结构体，其中包含了 Java 包的清单信息
        Manifest: &syftPkg.JavaManifest{
            # 设置 Java 包的主要信息
            Main: map[string]string{
                "Name": "main-section-name-info",
            },
            # 创建一个名为 NamedSections 的结构体，其中包含了 Java 包的命名部分信息
            NamedSections: map[string]map[string]string{
                "named-section": {
                    "named-section-key": "named-section-value",
                },
					},
					},
					PomProperties: &syftPkg.JavaPomProperties{
						Path:       "pom-path-info", // 设置 POM 文件的路径信息
						Name:       "pom-name-info", // 设置 POM 文件的名称信息
						GroupID:    "pom-group-ID-info", // 设置 POM 文件的 GroupID 信息
						ArtifactID: "pom-artifact-ID-info", // 设置 POM 文件的 ArtifactID 信息
						Version:    "pom-version-info", // 设置 POM 文件的版本信息
						Extra: map[string]string{ // 设置额外的信息
							"extra-key": "extra-value", // 设置额外信息的键值对
						},
					},
					ArchiveDigests: []syftFile.Digest{{
						Algorithm: "sha1", // 设置归档文件的摘要算法
						Value:     "236e3bfdbdc6c86629237a74f0f11414adb4e211", // 设置归档文件的摘要数值
					}},
				},
			},
			metadata: JavaMetadata{
				VirtualPath:   "virtual-path-info", // 设置 Java 元数据的虚拟路径信息
# 设置PomArtifactID字段的值为"pom-artifact-ID-info"
PomArtifactID: "pom-artifact-ID-info",
# 设置PomGroupID字段的值为"pom-group-ID-info"
PomGroupID:    "pom-group-ID-info",
# 设置ManifestName字段的值为"main-section-name-info"
ManifestName:  "main-section-name-info",
# 设置ArchiveDigests字段的值为包含一个Digest对象的空列表
ArchiveDigests: []Digest{{
    # 设置Digest对象的Algorithm字段的值为"sha1"
    Algorithm: "sha1",
    # 设置Digest对象的Value字段的值为"236e3bfdbdc6c86629237a74f0f11414adb4e211"
    Value:     "236e3bfdbdc6c86629237a74f0f11414adb4e211",
}},
# 设置name字段的值为"apk with source info"
name: "apk with source info",
# 设置syftPkg字段的值为一个Package对象
syftPkg: syftPkg.Package{
    # 设置Metadata字段的值为一个ApkDBEntry对象
    Metadata: syftPkg.ApkDBEntry{
        # 设置Package字段的值为"libcurl-tools"
        Package:       "libcurl-tools",
        # 设置OriginPackage字段的值为"libcurl"
        OriginPackage: "libcurl",
        # 设置Maintainer字段的值为"somone"
        Maintainer:    "somone",
        # 设置Version字段的值为"1.2.3"
        Version:       "1.2.3",
        # 设置Architecture字段的值为"a"
        Architecture:  "a",
        # 设置URL字段的值为"a"
        URL:           "a",
        # 设置Description字段的值为"a"
        Description:   "a",
# 定义一个包含 Size 和 InstalledSize 属性的对象
{
    Size:          1,
    InstalledSize: 1,
},
# 定义一个包含 Name 属性的 UpstreamPackage 对象
upstreams: []UpstreamPackage{
    {
        Name: "libcurl",
    },
},
# 定义一个包含 Files 属性的 ApkMetadata 对象
metadata: ApkMetadata{Files: []ApkFileRecord{}},
},
# 以下的包是那些没有元数据或上游信息需要解析的包
{
    # 定义一个名为 "npm-metadata" 的包
    name: "npm-metadata",
    # 定义一个包含 Metadata 属性的 syftPkg.Package 对象
    syftPkg: syftPkg.Package{
        # 定义一个包含 Author、Homepage、Description 和 URL 属性的 syftPkg.NpmPackage 对象
        Metadata: syftPkg.NpmPackage{
            Author:      "a",
            Homepage:    "a",
            Description: "a",
            URL:         "a",
		},
		},
		{
			# 定义包名为 "python-metadata" 的包
			name: "python-metadata",
			# 设置 syftPkg 属性为 PythonPackage 类型的对象
			syftPkg: syftPkg.Package{
				# 设置 Metadata 属性为 PythonPackage 类型的对象
				Metadata: syftPkg.PythonPackage{
					# 设置包的名称为 "a"
					Name:                 "a",
					# 设置包的版本为 "a"
					Version:              "a",
					# 设置包的作者为 "a"
					Author:               "a",
					# 设置包的作者邮箱为 "a"
					AuthorEmail:          "a",
					# 设置包的平台为 "a"
					Platform:             "a",
					# 设置包的安装路径为 "a"
					SitePackagesRootPath: "a",
				},
			},
		},
		{
			# 定义包名为 "gem-metadata" 的包
			name: "gem-metadata",
			# 设置 syftPkg 属性为 Package 类型的对象
			syftPkg: syftPkg.Package{
				# 设置 Metadata 属性为 RubyGemspec 类型的对象
				Metadata: syftPkg.RubyGemspec{
# 创建一个名为 "a" 的包
name: "a",
# 设置包的版本为 "a"
Version:  "a",
# 设置包的主页为 "a"
Homepage: "a",
# 创建一个名为 "kb-metadata" 的包
name: "kb-metadata",
# 设置包的元数据为 MicrosoftKbPatch 类型，包含产品ID和KB号
Metadata: syftPkg.MicrosoftKbPatch{
    ProductID: "a",
    Kb:        "a",
},
# 创建一个名为 "rust-metadata" 的包
name: "rust-metadata",
# 设置包的元数据为 RustCargoLockEntry 类型，包含名称为 "a" 的条目
Metadata: syftPkg.RustCargoLockEntry{
    Name:     "a",
		{
			# 定义包名为"golang-metadata"的元数据
			name: "golang-metadata",
			# 设置syftPkg包的元数据
			syftPkg: syftPkg.Package{
				# 设置元数据的构建信息
				Metadata: syftPkg.GolangBinaryBuildinfoEntry{
					# 设置构建设置为空的字符串映射
					BuildSettings:     map[string]string{},
					# 设置Go编译版本为"1.0.0"
					GoCompiledVersion: "1.0.0",
					# 设置H1摘要为"a"
					H1Digest:          "a",
					# 设置主模块为"myMainModule"
					MainModule:        "myMainModule",
				},
			},
			# 设置Golang二进制元数据
			metadata: GolangBinMetadata{
				# 设置构建设置为空的字符串映射
				BuildSettings:     map[string]string{},
				# 设置Go编译版本为"1.0.0"
				GoCompiledVersion: "1.0.0",
				# 设置H1摘要为"a"
				H1Digest:          "a",
# 定义一个名为 "myMainModule" 的主模块
MainModule: "myMainModule",
# 定义一个名为 "golang-mod-metadata" 的模块，并设置其元数据
name: "golang-mod-metadata",
syftPkg: syftPkg.Package{
    # 设置 Golang 模块的元数据，包括 H1Digest
    Metadata: syftPkg.GolangModuleEntry{
        H1Digest: "h1:as234NweNNTNWEtt13nwNENTt",
    },
},
# 设置 Golang 模块的元数据，包括 H1Digest
metadata: GolangModMetadata{
    H1Digest: "h1:as234NweNNTNWEtt13nwNENTt",
},
# 定义一个名为 "php-composer-lock-metadata" 的模块，并设置其元数据
name: "php-composer-lock-metadata",
syftPkg: syftPkg.Package{
    # 设置 PHP Composer 锁的元数据，包括名称和版本
    Metadata: syftPkg.PhpComposerLockEntry{
        Name: "a",
        Version: "a",
		},
	},
	{
		// 定义包名为 "php-composer-installed-metadata" 的包
		name: "php-composer-installed-metadata",
		// 设置包的元数据，包括名称和版本
		syftPkg: syftPkg.Package{
			Metadata: syftPkg.PhpComposerInstalledEntry{
				Name:    "a",
				Version: "a",
			},
		},
	},
	{
		// 定义包名为 "dart-pub-metadata" 的包
		name: "dart-pub-metadata",
		// 设置包的元数据，包括名称和版本
		syftPkg: syftPkg.Package{
			Metadata: syftPkg.DartPubspecLockEntry{
				Name:    "a",
				Version: "a",
			},
		},
		},
		{
			// 定义包名为 "dotnet-metadata" 的包
			name: "dotnet-metadata",
			// 设置包的元数据
			syftPkg: syftPkg.Package{
				// 设置元数据为 .NET 依赖项条目
				Metadata: syftPkg.DotnetDepsEntry{
					Name:     "a",
					Version:  "a",
					Path:     "a",
					Sha512:   "a",
					HashPath: "a",
				},
			},
		},
		{
			// 定义包名为 "cpp conan-metadata" 的包
			name: "cpp conan-metadata",
			// 设置包的元数据
			syftPkg: syftPkg.Package{
				// 设置元数据为 Conanfile 条目
				Metadata: syftPkg.ConanfileEntry{
					Ref: "catch2/2.13.8",
				},
			},
		},
		{
			// 定义一个名为 "cpp conan lock metadata" 的包，包含一些元数据信息
			name: "cpp conan lock metadata",
			// 定义一个名为 "syftPkg" 的包对象，包含元数据信息
			syftPkg: syftPkg.Package{
				// 元数据信息为ConanLockEntry类型
				Metadata: syftPkg.ConanLockEntry{
					// 引用版本为 "zlib/1.2.12"
					Ref: "zlib/1.2.12",
					// 选项为一个键值对，包含 "fPIC" 和 "shared" 两个选项
					Options: map[string]string{
						"fPIC":   "True",
						"shared": "False",
					},
					// 路径为 "all/conanfile.py"
					Path:    "all/conanfile.py",
					// 上下文为 "host"
					Context: "host",
				},
			},
		},
		{
			// 定义一个名为 "cocoapods cocoapods-metadata" 的包，包含一些元数据信息
			name: "cocoapods cocoapods-metadata",
			// 定义一个名为 "syftPkg" 的包对象，包含元数据信息
			syftPkg: syftPkg.Package{
				// 元数据信息为CocoaPodfileLockEntry类型
				Metadata: syftPkg.CocoaPodfileLockEntry{
					// 校验和为 "123eere234"
					Checksum: "123eere234",
		},
	},
},
{
	// 定义包名为 "portage-metadata" 的软件包
	name: "portage-metadata",
	// 设置软件包的元数据
	syftPkg: syftPkg.Package{
		// 设置 PortageEntry 类型的元数据
		Metadata: syftPkg.PortageEntry{
			// 设置已安装大小为 1
			InstalledSize: 1,
			// 设置文件列表为空
			Files:         []syftPkg.PortageFileRecord{},
		},
	},
},
{
	// 定义包名为 "hackage-stack-lock-metadata" 的软件包
	name: "hackage-stack-lock-metadata",
	// 设置软件包的元数据
	syftPkg: syftPkg.Package{
		// 设置 HackageStackYamlLockEntry 类型的元数据
		Metadata: syftPkg.HackageStackYamlLockEntry{
			// 设置包哈希值为 "some-hash"
			PkgHash: "some-hash",
		},
	},
},
		{
			# 定义包名为"hackage-stack-metadata"，并设置其对应的syftPkg.Package对象
			name: "hackage-stack-metadata",
			syftPkg: syftPkg.Package{
				# 设置Metadata为HackageStackYamlEntry对象，其中包含PkgHash属性为"some-hash"
				Metadata: syftPkg.HackageStackYamlEntry{
					PkgHash: "some-hash",
				},
			},
		},
		{
			# 定义包名为"rebar-metadata"，并设置其对应的syftPkg.Package对象
			name: "rebar-metadata",
			syftPkg: syftPkg.Package{
				# 设置Metadata为ErlangRebarLockEntry对象，其中包含Name属性为"rebar"和Version属性为"v0.1.1"
				Metadata: syftPkg.ErlangRebarLockEntry{
					Name:    "rebar",
					Version: "v0.1.1",
				},
			},
		},
		{
			# 定义包名为"npm-package-lock-metadata"，并设置其对应的syftPkg.Package对象
			...
		// 创建一个名为 syftPkg 的包对象，包含 NpmPackageLockEntry 类型的元数据
		{
			name: "npm-package-lock-metadata",
			syftPkg: syftPkg.Package{
				// 设置 NpmPackageLockEntry 的 Resolved 和 Integrity 属性
				Metadata: syftPkg.NpmPackageLockEntry{
					Resolved:  "resolved",
					Integrity: "sha1:ab7d8979989b7a98d97",
				},
			},
		},
		// 创建一个名为 syftPkg 的包对象，包含 ElixirMixLockEntry 类型的元数据
		{
			name: "mix-lock-metadata",
			syftPkg: syftPkg.Package{
				// 设置 ElixirMixLockEntry 的 Name 和 Version 属性
				Metadata: syftPkg.ElixirMixLockEntry{
					Name:    "mix-lock",
					Version: "v0.1.2",
				},
			},
		},
		// 创建一个名为 syftPkg 的包对象，包含 PythonPipfileLockEntry 类型的元数据
		{
			name: "pipfile-lock-metadata",
			syftPkg: syftPkg.Package{
				// 设置 PythonPipfileLockEntry 的 Hashes 属性
				Metadata: syftPkg.PythonPipfileLockEntry{
					Hashes: []string{
# 定义一个名为 "sha1:ab8v88a8b88d8d8c88b8s765s47" 的索引项
{
    "sha1": "ab8v88a8b88d8d8c88b8s765s47",
},
# 设置索引为 "1"
Index: "1",
# 定义一个名为 "python-requirements-metadata" 的包
{
    name: "python-requirements-metadata",
    # 设置包的元数据
    syftPkg: syftPkg.Package{
        Metadata: syftPkg.PythonRequirementsEntry{
            Name: "a",
            Extras: []string{"a"},
            VersionConstraint: "a",
            URL: "a",
            Markers: "a",
        },
    },
},
# 定义一个名为 "binary-metadata" 的包
# 创建一个名为syftPkg的Package结构体
syftPkg: syftPkg.Package{
    # 在Package结构体中创建一个名为Metadata的BinarySignature结构体
    Metadata: syftPkg.BinarySignature{
        # 在BinarySignature结构体中创建一个名为Matches的ClassifierMatch切片
        Matches: []syftPkg.ClassifierMatch{
            # 在ClassifierMatch切片中创建一个Classifier字段并赋值为"node"
            {
                Classifier: "node",
            },
        },
    },
},
# 创建另一个名为syftPkg的Package结构体
{
    # 在Package结构体中创建一个名为name的字段并赋值为"nix-store-metadata"
    name: "nix-store-metadata",
    # 在Package结构体中创建一个名为syftPkg的Package结构体
    syftPkg: syftPkg.Package{
        # 在Package结构体中创建一个名为Metadata的NixStoreEntry结构体
        Metadata: syftPkg.NixStoreEntry{
            # 在NixStoreEntry结构体中创建一个名为OutputHash的字段并赋值为"a"
            OutputHash: "a",
            # 在NixStoreEntry结构体中创建一个名为Output的字段并赋值为"a"
            Output: "a",
            # 在NixStoreEntry结构体中创建一个名为Files的字符串切片并赋值为["a"]
            Files: []string{
                "a",
            },
        },
    },
}
		},
		},
		{
			# 定义名称为 "linux-kernel-metadata" 的包
			name: "linux-kernel-metadata",
			# 设置 syftPkg 包的元数据
			syftPkg: syftPkg.Package{
				# 设置元数据为 LinuxKernel 类型
				Metadata: syftPkg.LinuxKernel{
					# 设置 Linux 内核的名称
					Name:            "a",
					# 设置 Linux 内核的架构
					Architecture:    "a",
					# 设置 Linux 内核的版本
					Version:         "a",
					# 设置 Linux 内核的扩展版本
					ExtendedVersion: "a",
					# 设置 Linux 内核的构建时间
					BuildTime:       "a",
					# 设置 Linux 内核的作者
					Author:          "a",
					# 设置 Linux 内核的格式
					Format:          "a",
					# 设置 Linux 内核的可读写根文件系统
					RWRootFS:        true,
					# 设置 Linux 内核的交换设备
					SwapDevice:      10,
					# 设置 Linux 内核的根设备
					RootDevice:      11,
					# 设置 Linux 内核的视频模式
					VideoMode:       "a",
				},
			},
		},
{
    # 定义一个名为 "linux-kernel-module-metadata" 的对象
    name: "linux-kernel-module-metadata",
    # 定义一个名为 "syftPkg" 的包对象
    syftPkg: syftPkg.Package{
        # 定义一个名为 "Metadata" 的 Linux 内核模块对象
        Metadata: syftPkg.LinuxKernelModule{
            # 设置 Linux 内核模块的名称为 "a"
            Name:          "a",
            # 设置 Linux 内核模块的版本为 "a"
            Version:       "a",
            # 设置 Linux 内核模块的源版本为 "a"
            SourceVersion: "a",
            # 设置 Linux 内核模块的路径为 "a"
            Path:          "a",
            # 设置 Linux 内核模块的描述为 "a"
            Description:   "a",
            # 设置 Linux 内核模块的作者为 "a"
            Author:        "a",
            # 设置 Linux 内核模块的许可证为 "a"
            License:       "a",
            # 设置 Linux 内核模块的内核版本为 "a"
            KernelVersion: "a",
            # 设置 Linux 内核模块的版本魔术为 "a"
            VersionMagic:  "a",
            # 定义一个名为 "Parameters" 的参数对象，包含一个名为 "a" 的参数
            Parameters: map[string]syftPkg.LinuxKernelModuleParameter{
                "a": {
                    # 设置参数类型为 "a"
                    Type:        "a",
                    # 设置参数描述为 "a"
                    Description: "a",
                },
            },
        },
    },
		},
		},
		{
			# 定义包的名称为 "r-description-file-metadata"
			name: "r-description-file-metadata",
			# 定义包的属性
			syftPkg: syftPkg.Package{
				# 定义包的元数据
				Metadata: syftPkg.RDescription{
					# 定义包的标题为 "a"
					Title:            "a",
					# 定义包的描述为 "a"
					Description:      "a",
					# 定义包的作者为 "a"
					Author:           "a",
					# 定义包的维护者为 "a"
					Maintainer:       "a",
					# 定义包的 URL 为 ["a"]
					URL:              []string{"a"},
					# 定义包的仓库为 "a"
					Repository:       "a",
					# 定义包的构建信息为 "a"
					Built:            "a",
					# 定义包是否需要编译为 true
					NeedsCompilation: true,
					# 定义包的导入项为 ["a"]
					Imports:          []string{"a"},
					# 定义包的依赖项为 ["a"]
					Depends:          []string{"a"},
					# 定义包的建议项为 ["a"]
					Suggests:         []string{"a"},
				},
			},
		},
```
		{
			// 定义包名为 "dotnet-portable-executable-metadata"，并设置其元数据
			name: "dotnet-portable-executable-metadata",
			// 设置 syftPkg.Package 结构体的 Metadata 字段为 syftPkg.DotnetPortableExecutableEntry 结构体
			syftPkg: syftPkg.Package{
				Metadata: syftPkg.DotnetPortableExecutableEntry{
					// 设置 DotnetPortableExecutableEntry 结构体的各个字段的值
					AssemblyVersion: "a",
					LegalCopyright:  "a",
					Comments:        "a",
					InternalName:    "a",
					CompanyName:     "a",
					ProductName:     "a",
					ProductVersion:  "a",
				},
			},
		},
		{
			// 定义包名为 "swift-package-manager-metadata"，并设置其元数据
			name: "swift-package-manager-metadata",
			// 设置 syftPkg.Package 结构体的 Metadata 字段为 syftPkg.SwiftPackageManagerResolvedEntry 结构体
			syftPkg: syftPkg.Package{
				Metadata: syftPkg.SwiftPackageManagerResolvedEntry{
					// 设置 SwiftPackageManagerResolvedEntry 结构体的 Revision 字段的值
					Revision: "a",
				},
			},
		},
		},
		{
			# 定义包的名称为 "conaninfo-entry"
			name: "conaninfo-entry",
			# 定义包的信息，包括元数据
			syftPkg: syftPkg.Package{
				# 定义元数据为 ConaninfoEntry 类型
				Metadata: syftPkg.ConaninfoEntry{
					# 定义引用为 "a"
					Ref:       "a",
					# 定义包ID为 "a"
					PackageID: "a",
				},
			},
		},
		{
			# 定义包的名称为 "rust-binary-audit-entry"
			name: "rust-binary-audit-entry",
			# 定义包的信息，包括元数据
			syftPkg: syftPkg.Package{
				# 定义元数据为 RustBinaryAuditEntry 类型
				Metadata: syftPkg.RustBinaryAuditEntry{
					# 定义名称为 "a"
					Name:    "a",
					# 定义版本为 "a"
					Version: "a",
					# 定义来源为 "a"
					Source:  "a",
				},
			},
		},
	}

	// 创建一个用于测试包元数据完成的测试器
	tester := testutil.NewPackageMetadataCompletionTester(t)

	// 运行所有测试用例
	for _, test := range tests {
		// 对每个测试用例运行测试
		t.Run(test.name, func(t *testing.T) {
			// 使用测试器测试 syftPkg 的元数据
			tester.Tested(t, test.syftPkg.Metadata)
			// 创建一个新的 Package 对象
			p := New(test.syftPkg)
			// 检查元数据是否符合预期
			assert.Equal(t, test.metadata, p.Metadata, "unexpected metadata")
			// 检查上游是否符合预期
			assert.Equal(t, test.upstreams, p.Upstreams, "unexpected upstream")
		})
	}
}

func TestFromCollection_DoesNotPanic(t *testing.T) {
	// 创建一个新的包集合
	collection := syftPkg.NewCollection()
```

注释：以上代码是一个测试函数，用于测试包的元数据。首先创建一个测试器，然后运行测试用例。每个测试用例都会对包的元数据进行测试，并创建一个新的 Package 对象进行比较。最后还有一个测试函数用于测试从包集合创建对象时是否会出现异常。
// 创建一个名为examplePackage的Package对象，包括名称、版本、位置、类型等信息
examplePackage := syftPkg.Package{
    Name:    "test",
    Version: "1.2.3",
    Locations: file.NewLocationSet(
        file.NewLocation("/test-path"),
    ),
    Type: syftPkg.NpmPkg,
}

// 将examplePackage添加到集合中
collection.Add(examplePackage)
// 再次将examplePackage添加到集合中

// 使用assert库的NotPanics函数来测试匿名函数，确保不会发生panic
assert.NotPanics(t, func() {
    // 调用FromCollection函数，生成合成配置的CPEs
    _ = FromCollection(collection, SynthesisConfig{})
})
}

// 定义测试函数TestFromCollection_GeneratesCPEs
func TestFromCollection_GeneratesCPEs(t *testing.T) {
    // 创建一个新的包集合
    collection := syftPkg.NewCollection()
}
// 向集合中添加一个名为"first"，版本为"1"的软件包，且不包含CPEs
collection.Add(syftPkg.Package{
    Name:    "first",
    Version: "1",
    CPEs: []cpe.CPE{
        {},
    },
})

// 向集合中添加一个名为"second"，版本为"2"的软件包，且不包含CPEs
collection.Add(syftPkg.Package{
    Name:    "second",
    Version: "2",
})

// 当没有标志时，不生成CPEs
pkgs := FromCollection(collection, SynthesisConfig{})
// 断言第一个软件包的CPEs长度为1
assert.Len(t, pkgs[0].CPEs, 1)
// 断言第二个软件包的CPEs长度为0
assert.Len(t, pkgs[1].CPEs, 0)

// 当有标志时，生成CPEs
# 从集合中创建包，并使用合成配置
pkgs = FromCollection(collection, SynthesisConfig{
    GenerateMissingCPEs: true,
})
# 断言第一个包的CPEs长度为1
assert.Len(t, pkgs[0].CPEs, 1)
# 断言第二个包的CPEs长度为1
assert.Len(t, pkgs[1].CPEs, 1)
}

# 测试获取名称和EL版本
func Test_getNameAndELVersion(t *testing.T) {
    # 定义测试用例
    tests := []struct {
        name            string
        sourceRPM       string
        expectedName    string
        expectedVersion string
    }{
        {
            name:            "sqlite-3.26.0-6.el8.src.rpm",
            sourceRPM:       "sqlite-3.26.0-6.el8.src.rpm",
            expectedName:    "sqlite",
            expectedVersion: "3.26.0-6.el8",
        },
# 创建一个包含测试数据的列表
tests = [
    {
        name:            "util-linux-ng-2.17.2-12.28.el6_9.src.rpm",
        sourceRPM:       "util-linux-ng-2.17.2-12.28.el6_9.src.rpm",
        expectedName:    "util-linux-ng",
        expectedVersion: "2.17.2-12.28.el6_9",
    },
    {
        name:            "util-linux-ng-2.17.2-12.28.el6_9.2.src.rpm",
        sourceRPM:       "util-linux-ng-2.17.2-12.28.el6_9.2.src.rpm",
        expectedName:    "util-linux-ng",
        expectedVersion: "2.17.2-12.28.el6_9.2",
    },
]

# 遍历测试数据列表
for _, test := range tests:
    # 对每个测试数据运行测试
    t.Run(test.name, func(t *testing.T):
        # 获取实际的名称和版本号
        actualName, actualVersion := getNameAndELVersion(test.sourceRPM)
        # 断言实际的名称和版本号与期望的相等
        assert.Equal(t, test.expectedName, actualName)
        assert.Equal(t, test.expectedVersion, actualVersion)
    )
}

func intRef(i int) *int {
	return &i
}

func Test_RemovePackagesByOverlap(t *testing.T) {
	tests := []struct {
		name             string  // 测试用例名称
		sbom             *sbom.SBOM  // 软件构建物料清单对象指针
		expectedPackages []string  // 期望的包列表
	}{
		{
			name: "includes all packages without overlap",  // 测试用例描述
			sbom: catalogWithOverlaps(  // 使用catalogWithOverlaps函数创建软件构建物料清单对象
				[]string{":go@1.18", "apk:node@19.2-r1", "binary:python@3.9"},  // 包含的软件包列表
				[]string{}),  // 不包含的软件包列表
			expectedPackages: []string{":go@1.18", "apk:node@19.2-r1", "binary:python@3.9"},  // 期望的包列表
		},
		{
# 测试用例：通过重叠排除单个软件包
name: "excludes single package by overlap",
# 创建包含重叠的软件目录
sbom: catalogWithOverlaps(
    # 初始软件包列表
    []string{"apk:go@1.18", "apk:node@19.2-r1", "binary:node@19.2"},
    # 要排除的软件包列表
    []string{"apk:node@19.2-r1 -> binary:node@19.2"}),
# 期望的软件包列表
expectedPackages: []string{"apk:go@1.18", "apk:node@19.2-r1"},
},
{
# 测试用例：如果操作系统软件包拥有操作系统软件包，则不排除
name: "does not exclude if OS package owns OS package",
# 创建包含重叠的软件目录
sbom: catalogWithOverlaps(
    # 初始软件包列表
    []string{"rpm:perl@5.3-r1", "rpm:libperl@5.3"},
    # 要排除的软件包列表
    []string{"rpm:perl@5.3-r1 -> rpm:libperl@5.3"}),
# 期望的软件包列表
expectedPackages: []string{"rpm:libperl@5.3", "rpm:perl@5.3-r1"},
},
{
# 测试用例：如果拥有软件包是非操作系统软件包，则不排除
name: "does not exclude if owning package is non-OS",
# 创建包含重叠的软件目录
sbom: catalogWithOverlaps(
    # 初始软件包列表
    []string{"python:urllib3@1.2.3", "python:otherlib@1.2.3"},
    # 要排除的软件包列表
    []string{"python:urllib3@1.2.3 -> python:otherlib@1.2.3"}),
# 期望的软件包列表
expectedPackages: []string{"python:otherlib@1.2.3", "python:urllib3@1.2.3"},
},
		{
			// 测试用例名称：排除多个重叠包
			name: "excludes multiple package by overlap",
			// 创建包含重叠的软件目录
			sbom: catalogWithOverlaps(
				// 输入的软件包列表
				[]string{"apk:go@1.18", "apk:node@19.2-r1", "binary:node@19.2", "apk:python@3.9-r9", "binary:python@3.9"},
				// 重叠关系列表
				[]string{"apk:node@19.2-r1 -> binary:node@19.2", "apk:python@3.9-r9 -> binary:python@3.9"}),
			// 期望的软件包列表
			expectedPackages: []string{"apk:go@1.18", "apk:node@19.2-r1", "apk:python@3.9-r9"},
		},
		{
			// 测试用例名称：不排除不同类型的软件包
			name: "does not exclude with different types",
			// 创建包含重叠的软件目录
			sbom: catalogWithOverlaps(
				// 输入的软件包列表
				[]string{"rpm:node@19.2-r1", "apk:node@19.2"},
				// 重叠关系列表
				[]string{"rpm:node@19.2-r1 -> apk:node@19.2"}),
			// 期望的软件包列表
			expectedPackages: []string{"apk:node@19.2", "rpm:node@19.2-r1"},
		},
		{
			// 测试用例名称：如果操作系统软件包拥有操作系统软件包，则不排除
			name: "does not exclude if OS package owns OS package",
			// 创建包含重叠的软件目录
			sbom: catalogWithOverlaps(
				// 输入的软件包列表
				[]string{"rpm:perl@5.3-r1", "rpm:libperl@5.3"},
				// 重叠关系列表
				[]string{"rpm:perl@5.3-r1 -> rpm:libperl@5.3"}),
			// 期望的软件包列表
			expectedPackages: []string{"rpm:libperl@5.3", "rpm:perl@5.3-r1"},
		},
		{
			// 测试用例名称：如果拥有的软件包不是操作系统，则不排除
			name: "does not exclude if owning package is non-OS",
			// 创建重叠的软件目录
			sbom: catalogWithOverlaps(
				// 定义软件包列表
				[]string{"python:urllib3@1.2.3", "python:otherlib@1.2.3"},
				// 定义重叠关系
				[]string{"python:urllib3@1.2.3 -> python:otherlib@1.2.3"}),
			// 期望的软件包列表
			expectedPackages: []string{"python:otherlib@1.2.3", "python:urllib3@1.2.3"},
		},
		{
			// 测试用例名称：系统 RPM 安装的 Python 绑定
			name: "python bindings for system RPM install",
			// 创建带有发行版信息的重叠软件目录
			sbom: withDistro(catalogWithOverlaps(
				// 定义软件包列表
				[]string{"rpm:python3-rpm@4.14.3-26.el8", "python:rpm@4.14.3"},
				// 定义重叠关系
				[]string{"rpm:python3-rpm@4.14.3-26.el8 -> python:rpm@4.14.3"}), "rhel"),
			// 期望的软件包列表
			expectedPackages: []string{"rpm:python3-rpm@4.14.3-26.el8"},
		},
		{
			// 测试用例名称：amzn Linux 不会以这种方式移除软件包
			name: "amzn linux doesn't remove packages in this way",
			// 创建带有发行版信息的重叠软件目录
			sbom: withDistro(catalogWithOverlaps(
				// 定义软件包列表
				[]string{"rpm:python3-rpm@4.14.3-26.el8", "python:rpm@4.14.3"},
				// 定义重叠关系
				[]string{"rpm:python3-rpm@4.14.3-26.el8 -> python:rpm@4.14.3"}), "amzn"),
// 定义一个测试用例结构体数组，包含了预期的软件包列表
expectedPackages: []string{"rpm:python3-rpm@4.14.3-26.el8", "python:rpm@4.14.3"},
}

// 遍历测试用例数组
for _, test := range tests {
    // 对每个测试用例运行子测试
    t.Run(test.name, func(t *testing.T) {
        // 从给定的软件包、关系和 Linux 发行版构建目录，移除重叠的软件包
        catalog := removePackagesByOverlap(test.sbom.Artifacts.Packages, test.sbom.Relationships, test.sbom.Artifacts.LinuxDistribution)
        // 从目录中创建软件包集合
        pkgs := FromCollection(catalog, SynthesisConfig{})
        var pkgNames []string
        // 遍历软件包集合，将软件包类型、名称和版本信息格式化为字符串并添加到 pkgNames 中
        for _, p := range pkgs {
            pkgNames = append(pkgNames, fmt.Sprintf("%s:%s@%s", p.Type, p.Name, p.Version))
        }
        // 断言预期的软件包列表和实际的软件包列表是否相等
        assert.EqualValues(t, test.expectedPackages, pkgNames)
    })
}

// 定义一个函数，返回包含重叠的软件包和关系的 SBOM 结构体指针
func catalogWithOverlaps(packages []string, overlaps []string) *sbom.SBOM {
    var pkgs []syftPkg.Package
    var relationships []artifact.Relationship
# 定义一个匿名函数，参数为一个字符串，返回一个 syftPkg.Package 对象
toPkg := func(str string) syftPkg.Package {
    # 定义变量用于存储类型、名称、版本信息
    var typ, name, version string
    # 去除字符串两端的空白字符，并按冒号分割字符串
    s := strings.Split(strings.TrimSpace(str), ":")
    # 如果分割后的数组长度大于1，将第一个元素赋值给类型，第二个元素赋值给字符串
    if len(s) > 1 {
        typ = s[0]
        str = s[1]
    }
    # 再次按 @ 分割字符串
    s = strings.Split(str, "@")
    # 将第一个元素赋值给名称
    name = s[0]
    # 如果分割后的数组长度大于1，将第二个元素赋值给版本
    if len(s) > 1 {
        version = s[1]
    }

    # 创建一个 syftPkg.Package 对象，设置类型、名称、版本信息
    p := syftPkg.Package{
        Type:    syftPkg.Type(typ),
        Name:    name,
        Version: version,
    }
    # 为包设置一个唯一的 ID
    p.SetID()
		// 返回 p
		return p
	}

	// 遍历包列表，将每个包转换为对应的结构体并添加到 pkgs 列表中
	for _, pkg := range packages {
		p := toPkg(pkg)
		pkgs = append(pkgs, p)
	}

	// 遍历重叠列表，将重叠关系拆分为起始和目标包，并添加到 relationships 列表中
	for _, overlap := range overlaps {
		parts := strings.Split(overlap, "->")
		if len(parts) < 2 {
			panic("invalid overlap, use -> to specify, e.g.: pkg1->pkg2")
		}
		from := toPkg(parts[0])
		to := toPkg(parts[1])

		relationships = append(relationships, artifact.Relationship{
			From: from,
			To:   to,
			Type: artifact.OwnershipByFileOverlapRelationship,
		})
	}
// 创建一个新的软件包集合
catalog := syftPkg.NewCollection(pkgs...)

// 创建一个SBOM对象，包含软件包集合和关系
return &sbom.SBOM{
    Artifacts: sbom.Artifacts{
        Packages: catalog,
    },
    Relationships: relationships,
}

// 为SBOM对象添加Linux发行版信息
func withDistro(s *sbom.SBOM, id string) *sbom.SBOM {
    s.Artifacts.LinuxDistribution = &linux.Release{
        ID: id,
    }
    return s
}
```