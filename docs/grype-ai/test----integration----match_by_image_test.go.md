# `grype\test\integration\match_by_image_test.go`

```
package integration

import (
    "sort" // 导入排序包
    "strings" // 导入字符串处理包
    "testing" // 导入测试包

    "github.com/facebookincubator/nvdtools/wfn" // 导入 Facebook 的 nvdtools 包
    "github.com/google/go-cmp/cmp" // 导入 Google 的 go-cmp 包
    "github.com/google/go-cmp/cmp/cmpopts" // 导入 Google 的 go-cmp 包中的 cmpopts 模块
    "github.com/stretchr/testify/require" // 导入 stretchr 的 testify 包中的 require 模块

    "github.com/anchore/grype/grype" // 导入 anchore 的 grype 包
    "github.com/anchore/grype/grype/db" // 导入 anchore 的 grype 包中的 db 模块
    "github.com/anchore/grype/grype/match" // 导入 anchore 的 grype 包中的 match 模块
    "github.com/anchore/grype/grype/matcher" // 导入 anchore 的 grype 包中的 matcher 模块
    "github.com/anchore/grype/grype/pkg" // 导入 anchore 的 grype 包中的 pkg 模块
    "github.com/anchore/grype/grype/store" // 导入 anchore 的 grype 包中的 store 模块
    "github.com/anchore/grype/grype/vex" // 导入 anchore 的 grype 包中的 vex 模块
    "github.com/anchore/grype/grype/vulnerability" // 导入 anchore 的 grype 包中的 vulnerability 模块
    "github.com/anchore/grype/internal/stringutil" // 导入 anchore 的 grype 包中的 stringutil 模块
    "github.com/anchore/stereoscope/pkg/imagetest" // 导入 anchore 的 stereoscope 包中的 imagetest 模块
    "github.com/anchore/syft/syft" // 导入 anchore 的 syft 包中的 syft 模块
    "github.com/anchore/syft/syft/linux" // 导入 anchore 的 syft 包中的 linux 模块
    syftPkg "github.com/anchore/syft/syft/pkg" // 导入 anchore 的 syft 包中的 pkg 模块
    "github.com/anchore/syft/syft/pkg/cataloger" // 导入 anchore 的 syft 包中的 pkg/cataloger 模块
    "github.com/anchore/syft/syft/source" // 导入 anchore 的 syft 包中的 source 模块
)

func addAlpineMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    packages := catalog.PackagesByPath("/lib/apk/db/installed") // 获取指定路径下的软件包
    if len(packages) != 3 { // 如果软件包数量不等于3
        t.Logf("Alpine Packages: %+v", packages) // 记录 Alpine 软件包信息
        t.Fatalf("problem with upstream syft cataloger (alpine)") // 输出错误信息并终止测试
    }
    thePkg := pkg.New(packages[0]) // 创建新的软件包对象
    theVuln := theStore.backend["alpine:distro:alpine:3.12"][thePkg.Name][0] // 获取指定漏洞信息
    vulnObj, err := vulnerability.NewVulnerability(theVuln) // 创建新的漏洞对象
    require.NoError(t, err) // 断言没有错误发生
    theResult.Add(match.Match{
        // 将匹配结果添加到结果集中
        // 注意：我们主要是在 secdb 记录上进行匹配，而不是 NVD

        Vulnerability: *vulnObj,  // 设置漏洞对象
        Package:       thePkg,    // 设置软件包对象
        Details: []match.Detail{  // 设置匹配详情列表
            {
                // 注意：输入的 pURL 具有上游引用（冗余）
                Type: "exact-indirect-match",  // 设置匹配类型
                SearchedBy: map[string]any{    // 设置搜索条件
                    "distro": map[string]string{  // 设置发行版信息
                        "type":    "alpine",      // 设置发行版类型
                        "version": "3.12.0",      // 设置发行版版本
                    },
                    "namespace": "alpine:distro:alpine:3.12",  // 设置命名空间
                    "package": map[string]string{  // 设置软件包信息
                        "name":    "libvncserver",  // 设置软件包名称
                        "version": "0.9.9",         // 设置软件包版本
                    },
                },
                Found: map[string]any{  // 设置匹配结果
                    "versionConstraint": "< 0.9.10 (unknown)",  // 设置版本约束
                    "vulnerabilityID":   "CVE-alpine-libvncserver",  // 设置漏洞ID
                },
                Matcher:    "apk-matcher",  // 设置匹配器类型
                Confidence: 1,              // 设置匹配置信度
            },
            {
                Type:       match.ExactDirectMatch,  // 设置匹配类型
                Confidence: 1.0,                    // 设置匹配置信度
                SearchedBy: map[string]interface{}{  // 设置搜索条件
                    "distro": map[string]string{  // 设置发行版信息
                        "type":    "alpine",      // 设置发行版类型
                        "version": "3.12.0",      // 设置发行版版本
                    },
                    "namespace": "alpine:distro:alpine:3.12",  // 设置命名空间
                    "package": map[string]string{  // 设置软件包信息
                        "name":    "libvncserver",  // 设置软件包名称
                        "version": "0.9.9",         // 设置软件包版本
                    },
                },
                Found: map[string]interface{}{  // 设置匹配结果
                    "versionConstraint": "< 0.9.10 (unknown)",  // 设置版本约束
                    "vulnerabilityID":   vulnObj.ID,             // 设置漏洞ID
                },
                Matcher: match.ApkMatcher,  // 设置匹配器类型
            },
        },
    })
}
// 添加 JavaScript 匹配项
func addJavascriptMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    // 通过路径查找 JavaScript 包
    packages := catalog.PackagesByPath("/javascript/pkg-json/package.json")
    // 如果找到的包不唯一，则输出日志并报错
    if len(packages) != 1 {
        t.Logf("Javascript Packages: %+v", packages)
        t.Fatalf("problem with upstream syft cataloger (javascript)")
    }
    // 创建包对象
    thePkg := pkg.New(packages[0])
    // 获取漏洞信息
    theVuln := theStore.backend["github:language:javascript"][thePkg.Name][0]
    // 创建漏洞对象
    vulnObj, err := vulnerability.NewVulnerability(theVuln)
    require.NoError(t, err)

    // 添加匹配项
    theResult.Add(match.Match{
        Vulnerability: *vulnObj,
        Package:       thePkg,
        Details: []match.Detail{
            {
                Type:       match.ExactDirectMatch,
                Confidence: 1.0,
                SearchedBy: map[string]interface{}{
                    "language":  "javascript",
                    "namespace": "github:language:javascript",
                    "package": map[string]string{
                        "name":    thePkg.Name,
                        "version": thePkg.Version,
                    },
                },
                Found: map[string]interface{}{
                    "versionConstraint": "> 5, < 7.2.1 (unknown)",
                    "vulnerabilityID":   vulnObj.ID,
                },
                Matcher: match.JavascriptMatcher,
            },
        },
    })
}

// 添加 Python 匹配项
func addPythonMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    // 通过路径查找 Python 包
    packages := catalog.PackagesByPath("/python/dist-info/METADATA")
    // 如果找到的包不唯一，则输出日志并报错
    if len(packages) != 1 {
        for _, p := range packages {
            t.Logf("Python Package: %s %+v", p.ID(), p)
        }

        t.Fatalf("problem with upstream syft cataloger (python)")
    }
    // 创建包对象
    thePkg := pkg.New(packages[0])
    // 获取规范化后的包名
    normalizedName := theStore.normalizedPackageNames["github:language:python"][thePkg.Name]
}
    // 从后端存储中获取指定语言和包名的漏洞信息
    theVuln := theStore.backend["github:language:python"][normalizedName][0]
    // 根据漏洞信息创建漏洞对象
    vulnObj, err := vulnerability.NewVulnerability(theVuln)
    require.NoError(t, err)

    // 将匹配结果添加到结果集中
    theResult.Add(match.Match{
        // 漏洞对象
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
                // 搜索条件
                SearchedBy: map[string]interface{}{
                    "language":  "python",
                    "namespace": "github:language:python",
                    "package": map[string]string{
                        "name":    thePkg.Name,
                        "version": thePkg.Version,
                    },
                },
                // 发现的漏洞信息
                Found: map[string]interface{}{
                    "versionConstraint": "< 2.6.2 (python)",
                    "vulnerabilityID":   vulnObj.ID,
                },
                // 匹配器为Python匹配器
                Matcher: match.PythonMatcher,
            },
        },
    })
# 添加与 dotnet 匹配的漏洞信息到结果集中
func addDotnetMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    # 通过路径获取 dotnet 包
    packages := catalog.PackagesByPath("/dotnet/TestLibrary.deps.json")
    # 如果包的数量不等于 2，则输出包的信息并报错
    if len(packages) != 2 { // TestLibrary + AWSSDK.Core
        for _, p := range packages {
            t.Logf("Dotnet Package: %s %+v", p.ID(), p)
        }
        t.Fatalf("problem with upstream syft cataloger (dotnet)")
    }
    # 创建包对象
    thePkg := pkg.New(packages[1])
    # 获取标准化后的包名
    normalizedName := theStore.normalizedPackageNames["github:language:dotnet"][thePkg.Name]
    # 获取漏洞信息
    theVuln := theStore.backend["github:language:dotnet"][normalizedName][0]
    # 创建漏洞对象
    vulnObj, err := vulnerability.NewVulnerability(theVuln)
    require.NoError(t, err)
    # 添加匹配信息到结果集中
    theResult.Add(match.Match{
        Vulnerability: *vulnObj,
        Package:       thePkg,
        Details: []match.Detail{
            {
                Type:       match.ExactDirectMatch,
                Confidence: 1.0,
                SearchedBy: map[string]interface{}{
                    "language":  "dotnet",
                    "namespace": "github:language:dotnet",
                    "package": map[string]string{
                        "name":    thePkg.Name,
                        "version": thePkg.Version,
                    },
                },
                Found: map[string]interface{}{
                    "versionConstraint": ">= 3.7.0.0, < 3.7.12.0 (unknown)",
                    "vulnerabilityID":   vulnObj.ID,
                },
                Matcher: match.DotnetMatcher,
            },
        },
    })
}

# 添加与 ruby 匹配的漏洞信息到结果集中
func addRubyMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    # 通过路径获取 ruby 包
    packages := catalog.PackagesByPath("/ruby/specifications/bundler.gemspec")
    # 如果包的数量不等于 1，则输出包的信息并报错
    if len(packages) != 1 {
        t.Logf("Ruby Packages: %+v", packages)
        t.Fatalf("problem with upstream syft cataloger (ruby)")
    }
    # 创建包对象
    thePkg := pkg.New(packages[0])
    // 从后端存储中获取指定语言和包名的漏洞信息
    theVuln := theStore.backend["github:language:ruby"][thePkg.Name][0]
    // 创建漏洞对象
    vulnObj, err := vulnerability.NewVulnerability(theVuln)
    require.NoError(t, err)

    // 将匹配结果添加到结果集中
    theResult.Add(match.Match{
        // 漏洞对象
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
                // 搜索条件
                SearchedBy: map[string]interface{}{
                    "language":  "ruby",
                    "namespace": "github:language:ruby",
                    "package": map[string]string{
                        "name":    thePkg.Name,
                        "version": thePkg.Version,
                    },
                },
                // 发现的漏洞信息
                Found: map[string]interface{}{
                    "versionConstraint": "> 2.0.0, <= 2.1.4 (unknown)",
                    "vulnerabilityID":   vulnObj.ID,
                },
                // 匹配器为RubyGemMatcher
                Matcher: match.RubyGemMatcher,
            },
        },
    })
func addGolangMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    // 通过路径查找包含"/golang/go.mod"的包
    modPackages := catalog.PackagesByPath("/golang/go.mod")
    // 如果找到的包不唯一，则输出日志并终止测试
    if len(modPackages) != 1 {
        t.Logf("Golang Mod Packages: %+v", modPackages)
        t.Fatalf("problem with upstream syft cataloger (golang)")
    }

    // 通过路径查找包含"/go-app"的包
    binPackages := catalog.PackagesByPath("/go-app")
    // 如果找到的包不是3个，则输出日志并终止测试
    // 这里包括2个包和一个单一的标准库包
    if len(binPackages) != 3 {
        t.Logf("Golang Bin Packages: %+v", binPackages)
        t.Fatalf("problem with upstream syft cataloger (golang)")
    }

    // 创建一个空的包列表
    var packages []syftPkg.Package
    // 将modPackages和binPackages合并到packages中
    packages = append(packages, modPackages...)
    packages = append(packages, binPackages...)
}
    // 遍历 packages 列表中的每个元素
    for _, p := range packages {
        // 如果包的名称为 "github.com/anchore/coverage"，则跳过当前循环
        if p.Name == "github.com/anchore/coverage" {
            continue
        }

        // 如果包的名称为 "stdlib"，则跳过当前循环
        if p.Name == "stdlib" {
            continue
        }

        // 根据当前包创建一个新的包对象
        thePkg := pkg.New(p)
        
        // 获取当前包对应的漏洞信息
        theVuln := theStore.backend["github:language:go"][thePkg.Name][0]
        
        // 根据漏洞信息创建一个新的漏洞对象
        vulnObj, err := vulnerability.NewVulnerability(theVuln)
        require.NoError(t, err)

        // 将匹配结果添加到结果集合中
        theResult.Add(match.Match{
            Vulnerability: *vulnObj,
            Package:       thePkg,
            Details: []match.Detail{
                {
                    Type:       match.ExactDirectMatch,
                    Confidence: 1.0,
                    SearchedBy: map[string]interface{}{
                        "language":  "go",
                        "namespace": "github:language:go",
                        "package": map[string]string{
                            "name":    thePkg.Name,
                            "version": thePkg.Version,
                        },
                    },
                    Found: map[string]interface{}{
                        "versionConstraint": "< 1.4.0 (unknown)",
                        "vulnerabilityID":   vulnObj.ID,
                    },
                    Matcher: match.GoModuleMatcher,
                },
            },
        })
    }
}
// 向测试结果中添加 Java 包的匹配信息
func addJavaMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    // 创建一个空的包列表
    packages := make([]syftPkg.Package, 0)
    // 遍历 syft 包集合中的 Java 包
    for p := range catalog.Enumerate(syftPkg.JavaPkg) {
        // 将遍历到的包添加到包列表中
        packages = append(packages, p)
    }
    // 如果包列表长度不为 2，输出日志并终止测试
    if len(packages) != 2 { // 2, because there's a nested JAR inside the test fixture JAR
        t.Logf("Java Packages: %+v", packages)
        t.Fatalf("problem with upstream syft cataloger (java)")
    }
    // 获取第一个 Java 包
    theSyftPkg := packages[0]

    // 获取 Java 包的 GroupID
    groupId := theSyftPkg.Metadata.(syftPkg.JavaArchive).PomProperties.GroupID
    // 构建查找键
    lookup := groupId + ":" + theSyftPkg.Name

    // 创建新的包对象
    thePkg := pkg.New(theSyftPkg)

    // 获取漏洞信息
    theVuln := theStore.backend["github:language:java"][lookup][0]
    vulnObj, err := vulnerability.NewVulnerability(theVuln)
    require.NoError(t, err)

    // 向结果中添加匹配信息
    theResult.Add(match.Match{
        Vulnerability: *vulnObj,
        Package:       thePkg,
        Details: []match.Detail{
            {
                Type:       match.ExactDirectMatch,
                Confidence: 1.0,
                SearchedBy: map[string]interface{}{
                    "language":  "java",
                    "namespace": "github:language:java",
                    "package": map[string]string{
                        "name":    thePkg.Name,
                        "version": thePkg.Version,
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

// 向测试结果中添加 Dpkg 包的匹配信息
func addDpkgMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    // 获取指定路径下的 Dpkg 包
    packages := catalog.PackagesByPath("/var/lib/dpkg/status")
    # 如果包的数量不等于1，则记录Dpkg Packages并且以失败的状态终止测试
    if len(packages) != 1 {
        t.Logf("Dpkg Packages: %+v", packages)
        t.Fatalf("problem with upstream syft cataloger (dpkg)")
    }
    # 创建一个新的包对象
    thePkg := pkg.New(packages[0])
    # 注意：这是一个间接匹配，典型的debian风格
    theVuln := theStore.backend["debian:distro:debian:8"][thePkg.Name+"-dev"][0]
    # 创建一个新的漏洞对象
    vulnObj, err := vulnerability.NewVulnerability(theVuln)
    require.NoError(t, err)

    # 将匹配结果添加到theResult中
    theResult.Add(match.Match{
        Vulnerability: *vulnObj,  # 漏洞对象
        Package:       thePkg,     # 包对象
        Details: []match.Detail{   # 匹配的细节
            {
                Type:       match.ExactIndirectMatch,  # 匹配类型
                Confidence: 1.0,                       # 置信度
                SearchedBy: map[string]interface{      # 搜索条件
                    "distro": map[string]string{
                        "type":    "debian",
                        "version": "8",
                    },
                    "namespace": "debian:distro:debian:8",
                    "package": map[string]string{
                        "name":    "apt-dev",
                        "version": "1.8.2",
                    },
                },
                Found: map[string]interface{           # 匹配结果
                    "versionConstraint": "<= 1.8.2 (deb)",
                    "vulnerabilityID":   vulnObj.ID,
                },
                Matcher: match.DpkgMatcher,            # 匹配器类型
            },
        },
    })
// 向结果中添加与 Portage 匹配的漏洞信息
func addPortageMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    // 通过路径获取 Portage 包信息
    packages := catalog.PackagesByPath("/var/db/pkg/app-containers/skopeo-1.5.1/CONTENTS")
    // 如果获取的包数量不为1，则记录日志并报错
    if len(packages) != 1 {
        t.Logf("Portage Packages: %+v", packages)
        t.Fatalf("problem with upstream syft cataloger (portage)")
    }
    // 创建包对象
    thePkg := pkg.New(packages[0])
    // 从存储中获取与包相关的漏洞信息
    theVuln := theStore.backend["gentoo:distro:gentoo:2.8"][thePkg.Name][0]
    // 创建漏洞对象
    vulnObj, err := vulnerability.NewVulnerability(theVuln)
    require.NoError(t, err)

    // 向结果中添加匹配信息
    theResult.Add(match.Match{
        Vulnerability: *vulnObj,
        Package:       thePkg,
        Details: []match.Detail{
            {
                Type:       match.ExactDirectMatch,
                Confidence: 1.0,
                SearchedBy: map[string]interface{}{
                    "distro": map[string]string{
                        "type":    "gentoo",
                        "version": "2.8",
                    },
                    "namespace": "gentoo:distro:gentoo:2.8",
                    "package": map[string]string{
                        "name":    "app-containers/skopeo",
                        "version": "1.5.1",
                    },
                },
                Found: map[string]interface{}{
                    "versionConstraint": "< 1.6.0 (unknown)",
                    "vulnerabilityID":   vulnObj.ID,
                },
                Matcher: match.PortageMatcher,
            },
        },
    })
}

// 向结果中添加与 Rhel 匹配的漏洞信息
func addRhelMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    // 通过路径获取 RPMDB 包信息
    packages := catalog.PackagesByPath("/var/lib/rpm/Packages")
    // 如果获取的包数量不为1，则记录日志并报错
    if len(packages) != 1 {
        t.Logf("RPMDB Packages: %+v", packages)
        t.Fatalf("problem with upstream syft cataloger (RPMDB)")
    }
    // 创建包对象
    thePkg := pkg.New(packages[0])
}
    // 从后端存储中获取指定操作系统和软件包的漏洞信息
    theVuln := theStore.backend["redhat:distro:redhat:8"][thePkg.Name][0]
    // 创建漏洞对象
    vulnObj, err := vulnerability.NewVulnerability(theVuln)
    require.NoError(t, err)

    // 将匹配结果添加到结果集中
    theResult.Add(match.Match{
        // 漏洞对象
        Vulnerability: *vulnObj,
        // 软件包信息
        Package:       thePkg,
        // 匹配的详细信息
        Details: []match.Detail{
            {
                // 匹配类型为精确直接匹配
                Type:       match.ExactDirectMatch,
                // 置信度为1.0
                Confidence: 1.0,
                // 搜索条件
                SearchedBy: map[string]interface{}{
                    "distro": map[string]string{
                        "type":    "centos",
                        "version": "8",
                    },
                    "namespace": "redhat:distro:redhat:8",
                    "package": map[string]string{
                        "name":    "dive",
                        "version": "0:0.9.2-1",
                    },
                },
                // 发现的匹配信息
                Found: map[string]interface{}{
                    "versionConstraint": "<= 1.0.42 (rpm)",
                    "vulnerabilityID":   vulnObj.ID,
                },
                // 使用的匹配器类型为 RPM 匹配器
                Matcher: match.RpmMatcher,
            },
        },
    })
}
// 向结果中添加 SLES 匹配项
func addSlesMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    // 通过路径获取指定包的信息
    packages := catalog.PackagesByPath("/var/lib/rpm/Packages")
    // 如果找到的包数量不等于1，则记录日志并抛出错误
    if len(packages) != 1 {
        t.Logf("Sles Packages: %+v", packages)
        t.Fatalf("problem with upstream syft cataloger (RPMDB)")
    }
    // 创建新的包对象
    thePkg := pkg.New(packages[0])
    // 从存储中获取指定漏洞信息
    theVuln := theStore.backend["redhat:distro:redhat:8"][thePkg.Name][0]
    // 创建新的漏洞对象
    vulnObj, err := vulnerability.NewVulnerability(theVuln)
    require.NoError(t, err)

    // 设置漏洞对象的命名空间
    vulnObj.Namespace = "sles:distro:sles:12.5"
    // 向结果中添加匹配项
    theResult.Add(match.Match{
        Vulnerability: *vulnObj,
        Package:       thePkg,
        Details: []match.Detail{
            {
                Type:       match.ExactDirectMatch,
                Confidence: 1.0,
                SearchedBy: map[string]interface{}{
                    "distro": map[string]string{
                        "type":    "sles",
                        "version": "12.5",
                    },
                    "namespace": "sles:distro:sles:12.5",
                    "package": map[string]string{
                        "name":    "dive",
                        "version": "0:0.9.2-1",
                    },
                },
                Found: map[string]interface{}{
                    "versionConstraint": "<= 1.0.42 (rpm)",
                    "vulnerabilityID":   vulnObj.ID,
                },
                Matcher: match.RpmMatcher,
            },
        },
    })
}

// 向结果中添加 Haskell 匹配项
func addHaskellMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    // 通过路径获取指定包的信息
    packages := catalog.PackagesByPath("/haskell/stack.yaml")
    // 如果找到的包数量小于1，则记录日志并抛出错误
    if len(packages) < 1 {
        t.Logf("Haskel Packages: %+v", packages)
        t.Fatalf("problem with upstream syft cataloger (haskell)")
    }
    // 创建新的包对象
    thePkg := pkg.New(packages[0])
}
    // 从后端存储中获取指定语言和包名的漏洞信息
    theVuln := theStore.backend["github:language:haskell"][strings.ToLower(thePkg.Name)][0]
    // 根据漏洞信息创建漏洞对象
    vulnObj, err := vulnerability.NewVulnerability(theVuln)
    require.NoError(t, err)

    // 将匹配结果添加到结果集中
    theResult.Add(match.Match{
        Vulnerability: *vulnObj, // 漏洞对象
        Package:       thePkg,    // 包对象
        Details: []match.Detail{  // 匹配的详细信息
            {
                Type:       match.ExactDirectMatch, // 匹配类型
                Confidence: 1.0,                    // 匹配的置信度
                SearchedBy: map[string]any{         // 搜索条件
                    "language":  "haskell",                // 语言
                    "namespace": "github:language:haskell", // 命名空间
                    "package": map[string]string{           // 包信息
                        "name":    thePkg.Name,     // 包名
                        "version": thePkg.Version,  // 版本
                    },
                },
                Found: map[string]any{              // 匹配结果
                    "versionConstraint": "< 0.9.0 (unknown)", // 版本约束
                    "vulnerabilityID":   "CVE-haskell-sample", // 漏洞ID
                },
                Matcher: match.StockMatcher, // 匹配器类型
            },
        },
    })
func addRustMatches(t *testing.T, theSource source.Source, catalog *syftPkg.Collection, theStore *mockStore, theResult *match.Matches) {
    // 通过路径获取包
    packages := catalog.PackagesByPath("/hello-auditable")
    // 如果包的数量小于1，则输出日志并终止测试
    if len(packages) < 1 {
        t.Logf("Rust Packages: %+v", packages)
        t.Fatalf("problem with upstream syft cataloger (cargo-auditable-binary-cataloger)")
    }

    // 遍历包列表
    for _, p := range packages {
        // 创建新的包对象
        thePkg := pkg.New(p)
        // 获取漏洞信息
        theVuln := theStore.backend["github:language:rust"][strings.ToLower(thePkg.Name)][0]
        // 创建漏洞对象
        vulnObj, err := vulnerability.NewVulnerability(theVuln)
        require.NoError(t, err)

        // 添加匹配结果
        theResult.Add(match.Match{
            Vulnerability: *vulnObj,
            Package:       thePkg,
            Details: []match.Detail{
                {
                    Type:       match.ExactDirectMatch,
                    Confidence: 1.0,
                    SearchedBy: map[string]any{
                        "language":  "rust",
                        "namespace": "github:language:rust",
                        "package": map[string]string{
                            "name":    thePkg.Name,
                            "version": thePkg.Version,
                        },
                    },
                    Found: map[string]any{
                        "versionConstraint": vulnObj.Constraint.String(),
                        "vulnerabilityID":   vulnObj.ID,
                    },
                    Matcher: match.RustMatcher,
                },
            },
        })
    }
}

func TestMatchByImage(t *testing.T) {
    // 创建观察匹配器集合和定义匹配器集合
    observedMatchers := stringutil.NewStringSet()
    definedMatchers := stringutil.NewStringSet()
    for _, l := range match.AllMatcherTypes {
        definedMatchers.Add(string(l))
    }

    // 定义测试用例
    tests := []struct {
        fixtureImage string
        expectedFn   func(source.Source, *syftPkg.Collection, *mockStore) match.Matches
    }
}
    // 使用循环遍历测试用例，每个测试用例包含 vex 状态和 vex 文档列表
    for n, tc := range map[string]struct {
        vexStatus    vex.Status
        vexDocuments []string
    }{
        // 第一个测试用例，包含 vex 状态为 Affected 和 vex 文档路径
        "openvex-affected":            {vex.StatusAffected, []string{"test-fixtures/vex/openvex/affected.openvex.json"}},
        // 第二个测试用例，包含 vex 状态为 UnderInvestigation 和 vex 文档路径
        "openvex-under_investigation": {vex.StatusUnderInvestigation, []string{"test-fixtures/vex/openvex/under_investigation.openvex.json"}},
    } {
        // 使用 t.Run 创建子测试，并执行匿名函数
        t.Run(n, func(t *testing.T) {
            // 调用 testIgnoredMatches 函数，获取忽略的匹配结果
            ignoredMatches := testIgnoredMatches()
            // 调用 vexMatches 函数，获取 vex 匹配结果
            vexedResults := vexMatches(t, ignoredMatches, tc.vexStatus, tc.vexDocuments)
            // 检查 vex 匹配结果的数量是否为 1，如果不是则输出错误信息
            if len(vexedResults.Sorted()) != 1 {
                t.Errorf("expected one vexed result, got none")
            }

            // 创建预期匹配结果集合
            expectedMatches := match.NewMatches()

            // 获取 vex 匹配结果中的第一个匹配
            result := vexedResults.Sorted()[0]
            // 检查 vex 匹配结果的详情是否符合预期，如果不符合则输出错误信息
            if len(result.Details) != len(ignoredMatches[0].Match.Details)+1 {
                t.Errorf(
                    "Details in VEXed results don't match (expected %d, got %d)",
                    len(ignoredMatches[0].Match.Details)+1, len(result.Details),
                )
            }

            // 移除 vex 匹配结果中的 VEX 匹配器的详情
            result.Details = result.Details[:len(result.Details)-1]
            // 创建实际匹配结果集合，并添加 vex 匹配结果
            actualResults := match.NewMatches()
            actualResults.Add(result)

            // 添加预期匹配结果，并断言预期匹配结果与实际匹配结果是否相等
            expectedMatches.Add(ignoredMatches[0].Match)
            assertMatches(t, expectedMatches.Sorted(), actualResults.Sorted())

            // 遍历 vex 匹配结果，获取每个匹配的详情中的匹配器，并添加到 observedMatchers 集合中
            for _, m := range vexedResults.Sorted() {
                for _, d := range m.Details {
                    observedMatchers.Add(string(d.Matcher))
                }
            }
        })
    }

    // 确保集成测试用例与实现的匹配器保持同步，从 observedMatchers 集合中移除 StockMatcher
    observedMatchers.Remove(string(match.StockMatcher))
    # 从定义的匹配器中移除指定的股票匹配器
    definedMatchers.Remove(string(match.StockMatcher))
    # 从定义的匹配器中移除指定的 Msrc 匹配器
    definedMatchers.Remove(string(match.MsrcMatcher))

    # 如果观察到的匹配器数量与定义的匹配器数量不相等
    if len(observedMatchers) != len(definedMatchers) {
        # 输出错误信息，指出匹配器覆盖不完整
        t.Errorf("matcher coverage incomplete (matchers=%d, coverage=%d)", len(definedMatchers), len(observedMatchers))
        # 将定义的匹配器转换为切片，并按字母顺序排序
        defs := definedMatchers.ToSlice()
        sort.Strings(defs)
        # 将观察到的匹配器转换为切片，并按字母顺序排序
        obs := observedMatchers.ToSlice()
        sort.Strings(obs)
        # 输出定义的匹配器和观察到的匹配器之间的差异
        t.Log(cmp.Diff(defs, obs))
    }
// testIgnoredMatches返回一个被忽略的匹配列表，用于测试vex匹配器
func testIgnoredMatches() []match.IgnoredMatch {
    // 返回一个空的被忽略匹配列表
    return []match.IgnoredMatch{}
}

// vexMatches将匹配列表中的第一个匹配移动到忽略列表，并将VEX“affected”文档应用于它，以将其移回匹配列表
func vexMatches(t *testing.T, ignoredMatches []match.IgnoredMatch, vexStatus vex.Status, vexDocuments []string) match.Matches {
    // 创建一个新的匹配列表
    matches := match.NewMatches()
    // 创建一个新的VEX处理器
    vexMatcher := vex.NewProcessor(vex.ProcessorOptions{
        Documents: vexDocuments,
        IgnoreRules: []match.IgnoreRule{
            {VexStatus: string(vexStatus)},
        },
    })

    // 创建一个包上下文
    pctx := &pkg.Context{
        Source: &source.Description{
            Metadata: source.StereoscopeImageSourceMetadata{
                RepoDigests: []string{
                    "alpine@sha256:ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
                },
            },
        },
        Distro: &linux.Release{},
    }

    // 应用VEX数据到匹配列表和被忽略的匹配列表
    vexedMatches, ignoredMatches, err := vexMatcher.ApplyVEX(pctx, &matches, ignoredMatches)
    if err != nil {
        t.Errorf("applying VEX data: %s", err)
    }

    // 检查被忽略的匹配列表是否为空
    if len(ignoredMatches) != 0 {
        t.Errorf("VEX text fixture %s must affect all ignored matches (%d left)", vexDocuments, len(ignoredMatches))
    }

    // 返回VEX处理后的匹配列表
    return *vexedMatches
}

// assertMatches用于断言预期的匹配和实际的匹配是否相等
func assertMatches(t *testing.T, expected, actual []match.Match) {
    t.Helper()
    // 定义比较选项
    var opts = []cmp.Option{
        cmpopts.IgnoreFields(vulnerability.Vulnerability{}, "Constraint"),
        cmpopts.IgnoreFields(pkg.Package{}, "Locations"),
        cmpopts.SortSlices(func(a, b match.Match) bool {
            return a.Package.ID < b.Package.ID
        }),
    }

    // 比较预期的匹配和实际的匹配
    if diff := cmp.Diff(expected, actual, opts...); diff != "" {
        t.Errorf("mismatch (-want +got):\n%s", diff)
    }
}
```