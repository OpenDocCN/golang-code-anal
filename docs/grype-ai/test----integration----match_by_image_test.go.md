# `grype\test\integration\match_by_image_test.go`

```
package integration
// 导入所需的包

import (
	"sort"
	"strings"
	"testing"

	"github.com/facebookincubator/nvdtools/wfn"
	"github.com/google/go-cmp/cmp"
	"github.com/google/go-cmp/cmp/cmpopts"
	"github.com/stretchr/testify/require"

	"github.com/anchore/grype/grype"
	"github.com/anchore/grype/grype/db"
	"github.com/anchore/grype/grype/match"
	"github.com/anchore/grype/grype/matcher"
	"github.com/anchore/grype/grype/pkg"
	"github.com/anchore/grype/grype/store"
	"github.com/anchore/grype/grype/vex"
	"github.com/anchore/grype/grype/vulnerability"
)
// 导入必要的包
"github.com/anchore/grype/internal/stringutil"
"github.com/anchore/stereoscope/pkg/imagetest"
"github.com/anchore/syft/syft"
"github.com/anchore/syft/syft/linux"
syftPkg "github.com/anchore/syft/syft/pkg"
"github.com/anchore/syft/syft/pkg/cataloger"
"github.com/anchore/syft/syft/source"

// 定义函数 addAlpineMatches，接受测试对象 t、源、包目录、存储和结果对象作为参数
func addAlpineMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    // 通过路径获取包
    packages := catalog.PackagesByPath("/lib/apk/db/installed")
    // 如果包的数量不等于 3，则输出日志并终止测试
    if len(packages) != 3 {
        t.Logf("Alpine Packages: %+v", packages)
        t.Fatalf("problem with upstream syft cataloger (alpine)")
    }
    // 创建包对象
    thePkg := pkg.New(packages[0])
    // 从存储中获取漏洞信息
    theVuln := theStore.backend["alpine:distro:alpine:3.12"][thePkg.Name][0]
    // 创建漏洞对象
    vulnObj, err := vulnerability.NewVulnerability(theVuln)
    // 如果创建漏洞对象时出错，则输出错误信息
    require.NoError(t, err)
}
		theResult.Add(match.Match{
		// 添加匹配结果到结果集合中

		Vulnerability: *vulnObj,
		// 设置匹配的漏洞对象

		Package:       thePkg,
		// 设置匹配的软件包对象

		Details: []match.Detail{
			// 添加匹配的细节信息到结果集合中
			{
				// 添加细节信息对象

				Type: "exact-indirect-match",
				// 设置匹配类型为精确间接匹配

				SearchedBy: map[string]any{
					// 设置搜索条件为一个包含不同类型键值对的映射
					"distro": map[string]string{
						"type":    "alpine",
						"version": "3.12.0",
					},
					// 设置操作系统类型和版本信息

					"namespace": "alpine:distro:alpine:3.12",
					// 设置命名空间信息

					"package": map[string]string{
						"name":    "libvncserver",
						"version": "0.9.9",
					},
					// 设置软件包名称和版本信息
				},
# 匹配到的漏洞信息，包括版本约束和漏洞ID
Found: map[string]any{
    "versionConstraint": "< 0.9.10 (unknown)",
    "vulnerabilityID":   "CVE-alpine-libvncserver",
},
# 匹配器类型
Matcher:    "apk-matcher",
# 匹配的置信度
Confidence: 1,
},
{
    # 精确直接匹配类型
    Type:       match.ExactDirectMatch,
    # 置信度为1.0
    Confidence: 1.0,
    # 搜索条件，包括发行版、命名空间和软件包信息
    SearchedBy: map[string]interface{}{
        "distro": map[string]string{
            "type":    "alpine",
            "version": "3.12.0",
        },
        "namespace": "alpine:distro:alpine:3.12",
        "package": map[string]string{
            "name":    "libvncserver",
            "version": "0.9.9",
        },
				},
				Found: map[string]interface{}{
					"versionConstraint": "< 0.9.10 (unknown)",
					"vulnerabilityID":   vulnObj.ID,
				},
				Matcher: match.ApkMatcher,
			},
		},
	})
}

func addJavascriptMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
	// 通过路径查找 JavaScript 包的 package.json 文件
	packages := catalog.PackagesByPath("/javascript/pkg-json/package.json")
	// 如果找到的包不唯一，输出日志并终止测试
	if len(packages) != 1 {
		t.Logf("Javascript Packages: %+v", packages)
		t.Fatalf("problem with upstream syft cataloger (javascript)")
	}
	// 创建包对象
	thePkg := pkg.New(packages[0])
	// 从存储中获取 JavaScript 包的漏洞信息
	theVuln := theStore.backend["github:language:javascript"][thePkg.Name][0]
	// 创建漏洞对象
	vulnObj, err := vulnerability.NewVulnerability(theVuln)
	// 确保没有错误发生
	require.NoError(t, err)

	// 将匹配结果添加到结果集中
	theResult.Add(match.Match{
		Vulnerability: *vulnObj,  // 添加漏洞对象
		Package:       thePkg,     // 添加包对象
		Details: []match.Detail{   // 添加匹配的细节信息
			{
				Type:       match.ExactDirectMatch,  // 设置匹配类型为精确直接匹配
				Confidence: 1.0,                     // 设置匹配的置信度为1.0
				SearchedBy: map[string]interface{   // 设置搜索条件
					"language":  "javascript",                // 搜索语言为 JavaScript
					"namespace": "github:language:javascript", // 搜索命名空间为 github:language:javascript
					"package": map[string]string{             // 设置包信息
						"name":    thePkg.Name,      // 设置包名
						"version": thePkg.Version,   // 设置包版本
					},
				},
				Found: map[string]interface{   // 设置匹配结果
					"versionConstraint": "> 5, < 7.2.1 (unknown)",  // 设置版本约束
					"vulnerabilityID":   vulnObj.ID,                 // 设置漏洞ID
// 定义一个函数，用于向测试中添加 Python 匹配项
func addPythonMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    // 通过路径查找 Python 包
    packages := catalog.PackagesByPath("/python/dist-info/METADATA")
    // 如果找到的包不唯一，则输出每个包的信息并报错
    if len(packages) != 1 {
        for _, p := range packages {
            t.Logf("Python Package: %s %+v", p.ID(), p)
        }
        t.Fatalf("problem with upstream syft cataloger (python)")
    }
    // 创建一个新的包对象
    thePkg := pkg.New(packages[0])
    // 获取标准化后的包名
    normalizedName := theStore.normalizedPackageNames["github:language:python"][thePkg.Name]
    // 获取包的漏洞信息
    theVuln := theStore.backend["github:language:python"][normalizedName][0]
    // 创建漏洞对象
    vulnObj, err := vulnerability.NewVulnerability(theVuln)
```

	// 检查是否有错误发生，如果有则标记测试失败
	require.NoError(t, err)

	// 将匹配结果添加到结果集中
	theResult.Add(match.Match{
		// 设置漏洞对象
		Vulnerability: *vulnObj,
		// 设置包信息
		Package:       thePkg,
		// 设置匹配的细节
		Details: []match.Detail{
			{
				// 设置匹配类型为精确直接匹配
				Type:       match.ExactDirectMatch,
				// 设置匹配的置信度
				Confidence: 1.0,
				// 设置搜索条件
				SearchedBy: map[string]interface{}{
					"language":  "python",
					"namespace": "github:language:python",
					"package": map[string]string{
						"name":    thePkg.Name,
						"version": thePkg.Version,
					},
				},
				// 设置找到的匹配信息
				Found: map[string]interface{}{
					"versionConstraint": "< 2.6.2 (python)",
// 添加与Python匹配的漏洞匹配项
func addPythonMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
	// 通过路径查找包含"/python/requirements.txt"的包
	packages := catalog.PackagesByPath("/python/requirements.txt")
	// 如果找到的包数量不等于1，则输出包的信息并报错
	if len(packages) != 1 {
		for _, p := range packages {
			t.Logf("Python Package: %s %+v", p.ID(), p)
		}
		t.Fatalf("problem with upstream syft cataloger (python)")
	}
	// 创建新的包对象
	thePkg := pkg.New(packages[0])
	// 获取包的标准化名称
	normalizedName := theStore.normalizedPackageNames["github:language:python"][thePkg.Name]
	// 获取存储中与包相关的漏洞
	theVuln := theStore.backend["github:language:python"][normalizedName][0]
}
// 创建一个新的漏洞对象，并检查是否有错误发生
vulnObj, err := vulnerability.NewVulnerability(theVuln)
require.NoError(t, err)

// 向结果对象中添加匹配信息
theResult.Add(match.Match{
    Vulnerability: *vulnObj, // 设置匹配的漏洞对象
    Package:       thePkg,   // 设置匹配的包对象
    Details: []match.Detail{ // 设置匹配的详细信息列表
        {
            Type:       match.ExactDirectMatch, // 设置匹配类型为精确直接匹配
            Confidence: 1.0, // 设置匹配的置信度为1.0
            SearchedBy: map[string]interface{ // 设置搜索条件
                "language":  "dotnet", // 设置搜索的语言为dotnet
                "namespace": "github:language:dotnet", // 设置搜索的命名空间
                "package": map[string]string{ // 设置搜索的包信息
                    "name":    thePkg.Name,    // 设置包的名称
                    "version": thePkg.Version, // 设置包的版本
                },
            },
            Found: map[string]interface{ // 设置匹配结果
                // 这里应该继续添加匹配结果的具体信息
            },
        },
        // 这里可以继续添加其他匹配的详细信息
    },
});
		// 设置版本约束为大于等于 3.7.0.0，小于 3.7.12.0
		"versionConstraint": ">= 3.7.0.0, < 3.7.12.0 (unknown)",
		// 设置漏洞 ID 为 vulnObj.ID
		"vulnerabilityID":   vulnObj.ID,
	},
	// 设置匹配器为 DotnetMatcher
	Matcher: match.DotnetMatcher,
},
},
})
}

func addRubyMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
	// 通过路径获取指定的 Ruby 包
	packages := catalog.PackagesByPath("/ruby/specifications/bundler.gemspec")
	// 如果找到的包数量不为 1，则输出日志并报错
	if len(packages) != 1 {
		t.Logf("Ruby Packages: %+v", packages)
		t.Fatalf("problem with upstream syft cataloger (ruby)")
	}
	// 创建新的包对象
	thePkg := pkg.New(packages[0])
	// 从存储中获取指定语言和包名的漏洞信息
	theVuln := theStore.backend["github:language:ruby"][thePkg.Name][0]
	// 创建漏洞对象
	vulnObj, err := vulnerability.NewVulnerability(theVuln)
	// 如果创建漏洞对象时出错，则输出错误信息
	require.NoError(t, err)
	// 将匹配结果添加到结果集中
	theResult.Add(match.Match{
		// 设置漏洞信息
		Vulnerability: *vulnObj,
		// 设置包信息
		Package:       thePkg,
		// 设置匹配的详细信息
		Details: []match.Detail{
			{
				// 设置匹配类型为精确直接匹配
				Type:       match.ExactDirectMatch,
				// 设置匹配的置信度为1.0
				Confidence: 1.0,
				// 设置搜索条件
				SearchedBy: map[string]interface{}{
					"language":  "ruby",
					"namespace": "github:language:ruby",
					"package": map[string]string{
						"name":    thePkg.Name,
						"version": thePkg.Version,
					},
				},
				// 设置匹配结果
				Found: map[string]interface{}{
					"versionConstraint": "> 2.0.0, <= 2.1.4 (unknown)",
					"vulnerabilityID":   vulnObj.ID,
				},
// 定义一个函数，用于向测试中添加 Golang 包的匹配结果
func addGolangMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    // 从 catalog 中获取指定路径下的 Golang 包
    modPackages := catalog.PackagesByPath("/golang/go.mod")
    // 如果获取到的包不止一个，输出日志并终止测试
    if len(modPackages) != 1 {
        t.Logf("Golang Mod Packages: %+v", modPackages)
        t.Fatalf("problem with upstream syft cataloger (golang)")
    }

    // 从 catalog 中获取指定路径下的二进制 Golang 包
    binPackages := catalog.PackagesByPath("/go-app")
    // 如果获取到的包不止三个，输出日志并终止测试
    // 这里包括了两个自定义包和一个标准库包
    if len(binPackages) != 3 {
        t.Logf("Golang Bin Packages: %+v", binPackages)
        t.Fatalf("problem with upstream syft cataloger (golang)")
    }
}
# 定义一个空的包列表
var packages []syftPkg.Package
# 将 modPackages 列表中的包添加到 packages 列表中
packages = append(packages, modPackages...)
# 将 binPackages 列表中的包添加到 packages 列表中
packages = append(packages, binPackages...)

# 遍历 packages 列表中的包
for _, p := range packages:
    # 如果包的名称为 "github.com/anchore/coverage"，则跳过当前循环，继续下一个包
    if p.Name == "github.com/anchore/coverage" {
        continue
    }
    # 如果包的名称为 "stdlib"，则跳过当前循环，继续下一个包
    if p.Name == "stdlib" {
        continue
    }

    # 根据包创建一个新的包对象
    thePkg := pkg.New(p)
    # 从存储中获取与包相关的漏洞信息
    theVuln := theStore.backend["github:language:go"][thePkg.Name][0]
    # 根据漏洞信息创建一个新的漏洞对象
    vulnObj, err := vulnerability.NewVulnerability(theVuln)
    # 断言没有错误发生
    require.NoError(t, err)

    # 将匹配结果添加到结果对象中
    theResult.Add(match.Match{
# 定义漏洞对象
Vulnerability: *vulnObj,
# 定义包对象
Package:       thePkg,
# 定义匹配的详细信息
Details: []match.Detail{
    # 定义匹配的类型为精确直接匹配
    {
        Type:       match.ExactDirectMatch,
        # 设置匹配的置信度为1.0
        Confidence: 1.0,
        # 设置搜索条件，包括语言、命名空间和包名
        SearchedBy: map[string]interface{}{
            "language":  "go",
            "namespace": "github:language:go",
            "package": map[string]string{
                "name":    thePkg.Name,
                "version": thePkg.Version,
            },
        },
        # 设置找到的漏洞信息，包括版本约束和漏洞ID
        Found: map[string]interface{}{
            "versionConstraint": "< 1.4.0 (unknown)",
            "vulnerabilityID":   vulnObj.ID,
        },
        # 设置匹配器为Go模块匹配器
        Matcher: match.GoModuleMatcher,
    },
		},
	})
}

// addJavaMatches function adds Java package matches to the result
func addJavaMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
	// Initialize an empty slice to store Java packages
	packages := make([]syftPkg.Package, 0)
	// Enumerate through the catalog to find Java packages and add them to the slice
	for p := range catalog.Enumerate(syftPkg.JavaPkg) {
		packages = append(packages, p)
	}
	// Check if the number of Java packages is not equal to 2
	if len(packages) != 2 { // 2, because there's a nested JAR inside the test fixture JAR
		// Log the Java packages found
		t.Logf("Java Packages: %+v", packages)
		// Fail the test with a message if there is a problem with the upstream syft cataloger (java)
		t.Fatalf("problem with upstream syft cataloger (java)")
	}
	// Get the first Java package from the slice
	theSyftPkg := packages[0]

	// Get the GroupID from the metadata of the Java package
	groupId := theSyftPkg.Metadata.(syftPkg.JavaArchive).PomProperties.GroupID
	// Create a lookup key using GroupID and the package name
	lookup := groupId + ":" + theSyftPkg.Name
// 使用给定的包创建一个新的包对象
thePkg := pkg.New(theSyftPkg)

// 从存储后端中获取指定语言和查找条件下的第一个漏洞
theVuln := theStore.backend["github:language:java"][lookup][0]

// 使用漏洞对象创建一个新的漏洞对象
vulnObj, err := vulnerability.NewVulnerability(theVuln)
require.NoError(t, err)

// 将匹配结果添加到结果对象中
theResult.Add(match.Match{
	Vulnerability: *vulnObj, // 漏洞对象
	Package:       thePkg,    // 包对象
	Details: []match.Detail{  // 匹配的细节
		{
			Type:       match.ExactDirectMatch, // 匹配类型
			Confidence: 1.0,                    // 置信度
			SearchedBy: map[string]interface{   // 搜索条件
				"language":  "java",                    // 语言
				"namespace": "github:language:java",    // 命名空间
				"package": map[string]string{           // 包信息
					"name":    thePkg.Name,      // 包名
					"version": thePkg.Version,   // 包版本
				},
				},
				Found: map[string]interface{}{
					"versionConstraint": ">= 0.0.1, < 1.2.0 (unknown)",
					"vulnerabilityID":   vulnObj.ID,
				},
				Matcher: match.JavaMatcher,
			},
		},
	})
}

func addDpkgMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
	// 通过路径获取 dpkg 系统包信息
	packages := catalog.PackagesByPath("/var/lib/dpkg/status")
	// 如果获取到的包数量不为 1，则输出日志并终止测试
	if len(packages) != 1 {
		t.Logf("Dpkg Packages: %+v", packages)
		t.Fatalf("problem with upstream syft cataloger (dpkg)")
	}
	// 创建一个新的包对象
	thePkg := pkg.New(packages[0])
	// 注意：这是一个间接匹配，典型的 Debian 风格
	// 从存储中获取与包名相关的漏洞信息
	theVuln := theStore.backend["debian:distro:debian:8"][thePkg.Name+"-dev"][0]
// 创建一个新的漏洞对象，并检查是否有错误发生
vulnObj, err := vulnerability.NewVulnerability(theVuln)
require.NoError(t, err)

// 向结果对象中添加匹配信息
theResult.Add(match.Match{
    Vulnerability: *vulnObj, // 设置匹配的漏洞对象
    Package:       thePkg,   // 设置匹配的软件包
    Details: []match.Detail{ // 设置匹配的细节信息
        {
            Type:       match.ExactIndirectMatch, // 设置匹配类型为精确间接匹配
            Confidence: 1.0, // 设置匹配的置信度为1.0
            SearchedBy: map[string]interface{ // 设置搜索条件
                "distro": map[string]string{ // 设置操作系统信息
                    "type":    "debian", // 设置操作系统类型为debian
                    "version": "8",      // 设置操作系统版本为8
                },
                "namespace": "debian:distro:debian:8", // 设置命名空间
                "package": map[string]string{ // 设置软件包信息
                    "name":    "apt-dev", // 设置软件包名称为apt-dev
                    "version": "1.8.2",   // 设置软件包版本为1.8.2
                },
// 定义一个函数，用于添加 Portage 匹配项到结果中
func addPortageMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    // 通过路径获取 Portage 包
    packages := catalog.PackagesByPath("/var/db/pkg/app-containers/skopeo-1.5.1/CONTENTS")
    // 如果获取到的包数量不为1，则输出日志并终止测试
    if len(packages) != 1 {
        t.Logf("Portage Packages: %+v", packages)
        t.Fatalf("problem with upstream syft cataloger (portage)")
    }
    // 创建一个新的包对象
    thePkg := pkg.New(packages[0])
    // 从存储中获取与包相关的漏洞信息
    theVuln := theStore.backend["gentoo:distro:gentoo:2.8"][thePkg.Name][0]
}
// 创建一个新的漏洞对象，并检查是否有错误发生
vulnObj, err := vulnerability.NewVulnerability(theVuln)
require.NoError(t, err)

// 将匹配结果添加到结果对象中
theResult.Add(match.Match{
    Vulnerability: *vulnObj, // 设置漏洞对象
    Package:       thePkg,    // 设置包对象
    Details: []match.Detail{  // 设置匹配的详细信息列表
        {
            Type:       match.ExactDirectMatch, // 设置匹配类型为精确直接匹配
            Confidence: 1.0,                   // 设置匹配的置信度为1.0
            SearchedBy: map[string]interface{  // 设置搜索条件
                "distro": map[string]string{   // 设置发行版信息
                    "type":    "gentoo",       // 设置发行版类型
                    "version": "2.8",          // 设置发行版版本
                },
                "namespace": "gentoo:distro:gentoo:2.8",  // 设置命名空间
                "package": map[string]string{              // 设置包信息
                    "name":    "app-containers/skopeo",   // 设置包名称
                    "version": "1.5.1",                   // 设置包版本
                },
		},
		Found: map[string]interface{}{
			"versionConstraint": "< 1.6.0 (unknown)", // 设置版本约束为小于 1.6.0（未知）
			"vulnerabilityID":   vulnObj.ID, // 设置漏洞 ID
		},
		Matcher: match.PortageMatcher, // 设置匹配器为 PortageMatcher
	},
},
})
}

func addRhelMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
	// 通过路径获取 RPMDB 包
	packages := catalog.PackagesByPath("/var/lib/rpm/Packages")
	// 如果包的数量不为 1，则输出日志并终止测试
	if len(packages) != 1 {
		t.Logf("RPMDB Packages: %+v", packages)
		t.Fatalf("problem with upstream syft cataloger (RPMDB)")
	}
	// 创建包对象
	thePkg := pkg.New(packages[0])
	// 获取 RedHat 漏洞信息
	theVuln := theStore.backend["redhat:distro:redhat:8"][thePkg.Name][0]
	// 创建漏洞对象
	vulnObj, err := vulnerability.NewVulnerability(theVuln)
	// 检查是否有错误发生，如果有则测试失败
	require.NoError(t, err)

	// 向结果中添加匹配对象
	theResult.Add(match.Match{
		// 漏洞信息
		Vulnerability: *vulnObj,
		// 包信息
		Package:       thePkg,
		// 匹配的详细信息
		Details: []match.Detail{
			{
				// 匹配类型为精确直接匹配
				Type:       match.ExactDirectMatch,
				// 置信度为1.0
				Confidence: 1.0,
				// 通过哪些信息进行搜索
				SearchedBy: map[string]interface{}{
					// 操作系统信息
					"distro": map[string]string{
						"type":    "centos",
						"version": "8",
					},
					// 命名空间信息
					"namespace": "redhat:distro:redhat:8",
					// 包信息
					"package": map[string]string{
						"name":    "dive",
						"version": "0:0.9.2-1",
					},
				},
				Found: map[string]interface{}{
					"versionConstraint": "<= 1.0.42 (rpm)", // 设置版本约束条件
					"vulnerabilityID":   vulnObj.ID, // 设置漏洞ID
				},
				Matcher: match.RpmMatcher, // 设置匹配器为RPM匹配器
			},
		},
	})
}

func addSlesMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
	packages := catalog.PackagesByPath("/var/lib/rpm/Packages") // 通过路径获取包信息
	if len(packages) != 1 { // 如果包的数量不等于1
		t.Logf("Sles Packages: %+v", packages) // 记录Sles包信息
		t.Fatalf("problem with upstream syft cataloger (RPMDB)") // 报告上游syft目录器（RPMDB）存在问题
	}
	thePkg := pkg.New(packages[0]) // 创建新的包对象
	theVuln := theStore.backend["redhat:distro:redhat:8"][thePkg.Name][0] // 获取漏洞信息
	vulnObj, err := vulnerability.NewVulnerability(theVuln) // 创建新的漏洞对象
	// 检查错误是否为空，如果不为空则触发测试失败
	require.NoError(t, err)

	// 设置漏洞对象的命名空间
	vulnObj.Namespace = "sles:distro:sles:12.5"
	
	// 将匹配结果添加到结果集中
	theResult.Add(match.Match{
		Vulnerability: *vulnObj, // 设置漏洞对象
		Package:       thePkg,    // 设置包对象

		// 设置匹配的详细信息
		Details: []match.Detail{
			{
				Type:       match.ExactDirectMatch, // 设置匹配类型为精确直接匹配
				Confidence: 1.0,                    // 设置匹配的置信度为1.0
				SearchedBy: map[string]interface{}{  // 设置搜索条件
					"distro": map[string]string{     // 设置操作系统信息
						"type":    "sles",           // 设置操作系统类型
						"version": "12.5",           // 设置操作系统版本
					},
					"namespace": "sles:distro:sles:12.5", // 设置命名空间
					"package": map[string]string{         // 设置包信息
						"name":    "dive",              // 设置包名
						"version": "0:0.9.2-1",         // 设置包版本
					},
				},
				Found: map[string]interface{}{
					"versionConstraint": "<= 1.0.42 (rpm)", // 设置版本约束条件
					"vulnerabilityID":   vulnObj.ID, // 设置漏洞ID
				},
				Matcher: match.RpmMatcher, // 设置匹配器为RPM匹配器
			},
		},
	})
}

func addHaskellMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
	packages := catalog.PackagesByPath("/haskell/stack.yaml") // 通过路径获取Haskell包
	if len(packages) < 1 {
		t.Logf("Haskel Packages: %+v", packages) // 记录Haskell包信息
		t.Fatalf("problem with upstream syft cataloger (haskell)") // 报告上游syft目录器（Haskell）的问题
	}
	thePkg := pkg.New(packages[0]) // 创建新的包对象
	theVuln := theStore.backend["github:language:haskell"][strings.ToLower(thePkg.Name)][0] // 获取Haskell语言的漏洞信息
	vulnObj, err := vulnerability.NewVulnerability(theVuln) // 创建新的漏洞对象
	// 检查是否有错误发生，如果有则测试失败
	require.NoError(t, err)

	// 将匹配结果添加到结果集中
	theResult.Add(match.Match{
		Vulnerability: *vulnObj,  // 添加漏洞对象到匹配结果中
		Package:       thePkg,     // 添加包信息到匹配结果中
		Details: []match.Detail{   // 添加匹配的详细信息到匹配结果中
			{
				Type:       match.ExactDirectMatch,  // 设置匹配类型为精确直接匹配
				Confidence: 1.0,                     // 设置匹配的置信度为1.0
				SearchedBy: map[string]any{          // 设置搜索条件
					"language":  "haskell",                  // 设置搜索条件中的语言为Haskell
					"namespace": "github:language:haskell",  // 设置搜索条件中的命名空间为github:language:haskell
					"package": map[string]string{            // 设置搜索条件中的包信息
						"name":    thePkg.Name,      // 设置包名
						"version": thePkg.Version,   // 设置包版本
					},
				},
				Found: map[string]any{  // 设置匹配结果
					"versionConstraint": "< 0.9.0 (unknown)",  // 设置版本约束
					"vulnerabilityID":   "CVE-haskell-sample",  // 设置漏洞ID
// addRustMatches 函数用于向测试结果中添加 Rust 包的匹配信息
func addRustMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    // 通过包路径获取包信息
    packages := catalog.PackagesByPath("/hello-auditable")
    // 如果获取的包信息为空，则输出日志并终止测试
    if len(packages) < 1 {
        t.Logf("Rust Packages: %+v", packages)
        t.Fatalf("problem with upstream syft cataloger (cargo-auditable-binary-cataloger)")
    }

    // 遍历获取的包信息
    for _, p := range packages {
        // 创建包对象
        thePkg := pkg.New(p)
        // 从存储中获取与包相关的漏洞信息
        theVuln := theStore.backend["github:language:rust"][strings.ToLower(thePkg.Name)][0]
        // 创建漏洞对象
        vulnObj, err := vulnerability.NewVulnerability(theVuln)
        // 如果创建漏洞对象时出错，则输出错误信息
        require.NoError(t, err)
		# 将匹配结果添加到结果集中
		theResult.Add(match.Match{
			# 匹配的漏洞对象
			Vulnerability: *vulnObj,
			# 包信息
			Package:       thePkg,
			# 匹配的详细信息
			Details: []match.Detail{
				# 具体匹配类型为精确直接匹配
				{
					Type:       match.ExactDirectMatch,
					# 置信度为1.0
					Confidence: 1.0,
					# 根据语言和命名空间进行搜索
					SearchedBy: map[string]any{
						"language":  "rust",
						"namespace": "github:language:rust",
						"package": map[string]string{
							"name":    thePkg.Name,
							"version": thePkg.Version,
						},
					},
					# 找到的具体信息
					Found: map[string]any{
						"versionConstraint": vulnObj.Constraint.String(),
						"vulnerabilityID":   vulnObj.ID,
					},
					# 匹配器为 RustMatcher
					Matcher: match.RustMatcher,
// 定义测试函数，用于测试根据图片进行匹配
func TestMatchByImage(t *testing.T) {
	// 创建观察到的匹配器集合和已定义的匹配器集合
	observedMatchers := stringutil.NewStringSet()
	definedMatchers := stringutil.NewStringSet()
	// 遍历所有的匹配器类型，并添加到已定义的匹配器集合中
	for _, l := range match.AllMatcherTypes {
		definedMatchers.Add(string(l))
	}

	// 定义测试用例
	tests := []struct {
		fixtureImage string
		expectedFn   func(source.Source, *syftPkg.Collection, *mockStore) match.Matches
	}{
		{
			fixtureImage: "image-debian-match-coverage",
			expectedFn: func(theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore) match.Matches {
# 创建一个新的匹配对象
expectedMatches := match.NewMatches()
# 调用不同语言的匹配函数，将匹配结果添加到expectedMatches中
addPythonMatches(t, theSource, catalog, theStore, &expectedMatches)
addRubyMatches(t, theSource, catalog, theStore, &expectedMatches)
addJavaMatches(t, theSource, catalog, theStore, &expectedMatches)
addDpkgMatches(t, theSource, catalog, theStore, &expectedMatches)
addJavascriptMatches(t, theSource, catalog, theStore, &expectedMatches)
addDotnetMatches(t, theSource, catalog, theStore, &expectedMatches)
addGolangMatches(t, theSource, catalog, theStore, &expectedMatches)
addHaskellMatches(t, theSource, catalog, theStore, &expectedMatches)
# 返回匹配结果
return expectedMatches
# 创建一个新的匹配对象
expectedMatches := match.NewMatches()
# 调用特定操作系统的匹配函数，将匹配结果添加到expectedMatches中
addRhelMatches(t, theSource, catalog, theStore, &expectedMatches)
# 返回匹配结果
return expectedMatches
		{
			// 定义 fixtureImage 属性为 "image-alpine-match-coverage"，expectedFn 属性为一个函数，返回匹配结果
			fixtureImage: "image-alpine-match-coverage",
			expectedFn: func(theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore) match.Matches {
				// 创建一个空的匹配结果集合
				expectedMatches := match.NewMatches()
				// 调用 addAlpineMatches 函数，将匹配结果添加到 expectedMatches 中
				addAlpineMatches(t, theSource, catalog, theStore, &expectedMatches)
				// 返回匹配结果集合
				return expectedMatches
			},
		},
		{
			// 定义 fixtureImage 属性为 "image-sles-match-coverage"，expectedFn 属性为一个函数，返回匹配结果
			fixtureImage: "image-sles-match-coverage",
			expectedFn: func(theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore) match.Matches {
				// 创建一个空的匹配结果集合
				expectedMatches := match.NewMatches()
				// 调用 addSlesMatches 函数，将匹配结果添加到 expectedMatches 中
				addSlesMatches(t, theSource, catalog, theStore, &expectedMatches)
				// 返回匹配结果集合
				return expectedMatches
			},
		},
		{
			// 定义 fixtureImage 属性为 "image-portage-match-coverage"，expectedFn 属性为一个函数，返回匹配结果
			fixtureImage: "image-portage-match-coverage",
			expectedFn: func(theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore) match.Matches {
				// 创建一个空的匹配结果集合
				expectedMatches := match.NewMatches()
# 调用 addPortageMatches 函数，传入参数 t, theSource, catalog, theStore, &expectedMatches，并将返回值赋给 expectedMatches
addPortageMatches(t, theSource, catalog, theStore, &expectedMatches)
# 返回 expectedMatches
return expectedMatches
# 定义一个匿名函数，接收 theSource, catalog, theStore 三个参数，返回 match.Matches 类型的值
expectedFn: func(theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore) match.Matches {
    # 创建一个新的空的 Matches 对象
    expectedMatches := match.NewMatches()
    # 调用 addRustMatches 函数，传入参数 t, theSource, catalog, theStore, &expectedMatches，并将返回值赋给 expectedMatches
    addRustMatches(t, theSource, catalog, theStore, &expectedMatches)
    # 返回 expectedMatches
    return expectedMatches
}

# 遍历 tests 列表
for _, test := range tests {
    # 对每个测试用例执行以下操作
    t.Run(test.fixtureImage, func(t *testing.T) {
        # 创建一个新的 mockDbStore 对象
        theStore := newMockDbStore()
        # 获取指定名称的 fixtureImage，并将其解压到指定路径
        imagetest.GetFixtureImage(t, "docker-archive", test.fixtureImage)
        # 获取指定名称的 fixtureImage 的 tar 文件路径
        tarPath := imagetest.GetFixtureImageTarPath(t, test.fixtureImage)
// 拼接用户图像的路径，创建 docker-archive 类型的镜像
userImage := "docker-archive:" + tarPath

// 使用 Detect 方法检测用户图像，返回检测结果和可能的错误
detection, err := source.Detect(userImage, source.DetectConfig{})
require.NoError(t, err)

// 为了设置模拟数据，创建检测结果的数据源
theSource, err := detection.NewSource(source.DetectionSourceConfig{})
require.NoError(t, err)
t.Cleanup(func() {
    require.NoError(t, theSource.Close())
})

// 设置默认配置，并将搜索范围设置为 SquashedScope
// TODO: 目前未验证关系
config := cataloger.DefaultConfig()
config.Search.Scope = source.SquashedScope

// 启用所有的 catalogers 来覆盖非默认情况
config.Catalogers = []string{"all"}
// 从给定的源和配置中获取软件包集合和发行版信息，同时检查是否有错误发生
collection, _, theDistro, err := syft.CatalogPackages(theSource, config)
require.NoError(t, err)

// 创建默认的匹配器集合
matchers := matcher.NewDefaultMatchers(matcher.Config{})

// 创建漏洞提供者和元数据提供者
vp, err := db.NewVulnerabilityProvider(theStore)
require.NoError(t, err)
mp := db.NewVulnerabilityMetadataProvider(theStore)
ep := db.NewMatchExclusionProvider(theStore)
str := store.Store{
    Provider:          vp,
    MetadataProvider:  mp,
    ExclusionProvider: ep,
}

// 使用提供的信息查找软件包的漏洞
actualResults := grype.FindVulnerabilitiesForPackage(str, theDistro, matchers, pkg.FromCollection(collection, pkg.SynthesisConfig{}))
for _, m := range actualResults.Sorted() {
    // 遍历实际结果中的漏洞详情，将匹配器添加到观察到的匹配器集合中
    for _, d := range m.Details {
        observedMatchers.Add(string(d.Matcher))
    }
}
		}

		// 从目录中发现的内容构建预期匹配项
		expectedMatches := test.expectedFn(theSource, collection, theStore)

		// 断言预期匹配项与实际结果的匹配情况
		assertMatches(t, expectedMatches.Sorted(), actualResults.Sorted())
	})

	// 测试当使用具有“affected”状态的文档时，VEX匹配器是否能够产生匹配项。
	for n, tc := range map[string]struct {
		vexStatus    vex.Status
		vexDocuments []string
	}{
		"openvex-affected":            {vex.StatusAffected, []string{"test-fixtures/vex/openvex/affected.openvex.json"}},
		"openvex-under_investigation": {vex.StatusUnderInvestigation, []string{"test-fixtures/vex/openvex/under_investigation.openvex.json"}},
	} {
		t.Run(n, func(t *testing.T) {
			// 获取忽略的匹配项
			ignoredMatches := testIgnoredMatches()
// 使用给定的参数调用 vexMatches 函数，返回 vexedResults
vexedResults := vexMatches(t, ignoredMatches, tc.vexStatus, tc.vexDocuments)
// 如果 vexedResults 中的匹配结果数量不等于 1，则输出错误信息
if len(vexedResults.Sorted()) != 1 {
    t.Errorf("expected one vexed result, got none")
}

// 创建一个新的空匹配结果集合
expectedMatches := match.NewMatches()

// 从 vexedResults 中获取第一个匹配结果
result := vexedResults.Sorted()[0]
// 如果 result 中的详情数量不等于 ignoredMatches[0].Match.Details 的数量加 1，则输出错误信息
if len(result.Details) != len(ignoredMatches[0].Match.Details)+1 {
    t.Errorf(
        "Details in VEXed results don't match (expected %d, got %d)",
        len(ignoredMatches[0].Match.Details)+1, len(result.Details),
    )
}

// 从 result 的详情中移除最后一个元素
result.Details = result.Details[:len(result.Details)-1]
// 创建一个新的匹配结果集合，并将 result 添加进去
actualResults := match.NewMatches()
actualResults.Add(result)
		// 将忽略的匹配项添加到预期匹配集合中
		expectedMatches.Add(ignoredMatches[0].Match)
		// 断言预期匹配集合和实际结果集合是否相等
		assertMatches(t, expectedMatches.Sorted(), actualResults.Sorted())

		// 遍历vexedResults集合中的匹配项
		for _, m := range vexedResults.Sorted() {
			// 遍历每个匹配项的细节
			for _, d := range m.Details {
				// 将观察到的匹配器添加到观察到的匹配器集合中
				observedMatchers.Add(string(d.Matcher))
			}
		}
	})

	// 确保集成测试用例与实现的匹配器保持同步
	// 从观察到的匹配器集合中移除StockMatcher
	observedMatchers.Remove(string(match.StockMatcher))
	// 从定义的匹配器集合中移除StockMatcher
	definedMatchers.Remove(string(match.StockMatcher))
	// 从定义的匹配器集合中移除MsrcMatcher
	definedMatchers.Remove(string(match.MsrcMatcher))

	// 如果观察到的匹配器数量不等于定义的匹配器数量
	if len(observedMatchers) != len(definedMatchers) {
		// 输出匹配器覆盖不完整的错误信息
		t.Errorf("matcher coverage incomplete (matchers=%d, coverage=%d)", len(definedMatchers), len(observedMatchers))
		// 将定义的匹配器转换为切片
		defs := definedMatchers.ToSlice()
// 对字符串数组进行排序
sort.Strings(defs)
// 将观察到的匹配项转换为切片，并对切片进行排序
obs := observedMatchers.ToSlice()
sort.Strings(obs)

// 记录比较结果的差异
t.Log(cmp.Diff(defs, obs))
}

}

// testIgnoredMatches 返回一个被忽略的匹配项列表，用于测试匹配器
func testIgnoredMatches() []match.IgnoredMatch {
	return []match.IgnoredMatch{
		{
			Match: match.Match{
				Vulnerability: vulnerability.Vulnerability{
					ID:        "CVE-alpine-libvncserver",
					Namespace: "alpine:distro:alpine:3.12",
				},
				Package: pkg.Package{
```

# 定义一个结构体或对象，包含了libvncserver软件包的相关信息
ID:       "44fa3691ae360cac",  # 软件包的唯一标识符
Name:     "libvncserver",      # 软件包的名称
Version:  "0.9.9",             # 软件包的版本号
Licenses: []string{"GPL-2.0-or-later"},  # 软件包的许可证信息
Type:     "apk",               # 软件包的类型
CPEs: []wfn.Attributes{       # 软件包的CPE（通用平台标识符）信息
    {
        Part:    "a",          # CPE的部分信息
        Vendor:  "libvncserver",  # CPE的供应商信息
        Product: "libvncserver",  # CPE的产品信息
        Version: "0.9.9",         # CPE的版本信息
    },
},
PURL:      "pkg:apk/alpine/libvncserver@0.9.9?arch=x86_64&distro=alpine-3.12.0",  # 软件包的PURL（可持续资源定位符）信息
Upstreams: []pkg.UpstreamPackage{{Name: "libvncserver"}},  # 软件包的上游包信息
},
Details: []match.Detail{  # 匹配的详细信息列表
    {
        Type: "exact-indirect-match",  # 匹配类型为精确间接匹配
        SearchedBy: map[string]any{  # 搜索的详细信息
            # 在这里添加具体的搜索信息
        }
    }
}
# 定义一个包含软件信息的结构体，包括发行版、命名空间、软件包等信息
"distro": map[string]string{
    "type":    "alpine",  # 发行版类型为alpine
    "version": "3.12.0",  # 发行版版本为3.12.0
},
"namespace": "alpine:distro:alpine:3.12",  # 命名空间为alpine:distro:alpine:3.12
"package": map[string]string{
    "name":    "libvncserver",  # 软件包名称为libvncserver
    "version": "0.9.9",  # 软件包版本为0.9.9
},
# 包含发现的漏洞信息，包括版本约束和漏洞ID
Found: map[string]any{
    "versionConstraint": "< 0.9.10 (unknown)",  # 版本约束为小于0.9.10（未知）
    "vulnerabilityID":   "CVE-alpine-libvncserver",  # 漏洞ID为CVE-alpine-libvncserver
},
Matcher:    "apk-matcher",  # 匹配器为apk-matcher
Confidence: 1,  # 置信度为1
```
		},
	}
}

// vexMatches函数将匹配列表中的第一个匹配移动到忽略列表，并将VEX“affected”文档应用于它，以将其移回匹配列表。
func vexMatches(t *testing.T, ignoredMatches []match.IgnoredMatch, vexStatus vex.Status, vexDocuments []string) match.Matches {
	// 创建一个新的匹配列表
	matches := match.NewMatches()
	// 创建一个VEX处理器
	vexMatcher := vex.NewProcessor(vex.ProcessorOptions{
		Documents: vexDocuments, // 设置VEX处理器的文档
		IgnoreRules: []match.IgnoreRule{ // 设置VEX处理器的忽略规则
			{VexStatus: string(vexStatus)}, // 根据VEX状态设置忽略规则
		},
	})

	// 创建一个包上下文
	pctx := &pkg.Context{
		Source: &source.Description{
			Metadata: source.StereoscopeImageSourceMetadata{
				RepoDigests: []string{
					"alpine@sha256:ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff", // 设置镜像的RepoDigests
		},
	},
},
Distro: &linux.Release{},
```
- 创建一个名为Distro的结构体，并将其指针赋值为linux.Release类型的实例。

```
vexedMatches, ignoredMatches, err := vexMatcher.ApplyVEX(pctx, &matches, ignoredMatches)
```
- 调用vexMatcher的ApplyVEX方法，传入参数pctx、matches的指针和ignoredMatches，并将返回的vexedMatches、ignoredMatches和err分别赋值给对应的变量。

```
if err != nil {
	t.Errorf("applying VEX data: %s", err)
}
```
- 如果err不为空，则输出错误信息"applying VEX data: "和err的值。

```
if len(ignoredMatches) != 0 {
	t.Errorf("VEX text fixture %s must affect all ignored matches (%d left)", vexDocuments, len(ignoredMatches))
}
```
- 如果ignoredMatches的长度不为0，则输出错误信息"VEX text fixture %s must affect all ignored matches (%d left)"，并将vexDocuments和ignoredMatches的长度作为参数传入。

```
return *vexedMatches
```
- 返回vexedMatches的值。

```
func assertMatches(t *testing.T, expected, actual []match.Match) {
	t.Helper()
```
- 定义一个名为assertMatches的函数，接受参数t、expected和actual，其中t是*testing.T类型的变量。
# 创建一个空的选项列表，用于配置比较器的行为
var opts = []cmp.Option{
    # 忽略vulnerability.Vulnerability结构体中的"Constraint"字段
    cmpopts.IgnoreFields(vulnerability.Vulnerability{}, "Constraint"),
    # 忽略pkg.Package结构体中的"Locations"字段
    cmpopts.IgnoreFields(pkg.Package{}, "Locations"),
    # 对比match.Match结构体切片时，按照Package.ID进行排序
    cmpopts.SortSlices(func(a, b match.Match) bool {
        return a.Package.ID < b.Package.ID
    }),
}

# 使用配置好的选项进行比较，如果有差异则输出错误信息
if diff := cmp.Diff(expected, actual, opts...); diff != "" {
    t.Errorf("mismatch (-want +got):\n%s", diff)
}
```