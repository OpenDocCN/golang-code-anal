# `grype\test\integration\compare_sbom_input_vs_lib_test.go`

```go
package integration

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "os" // 导入 os 包，用于操作系统功能
    "testing" // 导入 testing 包，用于编写测试函数

    "github.com/scylladb/go-set/strset" // 导入第三方包，用于操作字符串集合
    "github.com/stretchr/testify/assert" // 导入第三方包，用于编写断言

    "github.com/anchore/grype/grype" // 导入 grype 包，用于漏洞扫描
    "github.com/anchore/grype/grype/db" // 导入 grype 数据库包，用于漏洞数据库操作
    "github.com/anchore/grype/internal" // 导入 grype 内部包
    "github.com/anchore/syft/syft/format/spdxjson" // 导入 syftjson 格式化包
    "github.com/anchore/syft/syft/format/spdxtagvalue" // 导入 spdxtagvalue 格式化包
    "github.com/anchore/syft/syft/format/syftjson" // 导入 syftjson 格式化包
    syftPkg "github.com/anchore/syft/syft/pkg" // 导入 syft 包，用于软件包操作
    "github.com/anchore/syft/syft/sbom" // 导入 sbom 包，用于生成软件组织结构
    "github.com/anchore/syft/syft/source" // 导入 source 包，用于源码操作
)

var imagesWithVulnerabilities = []string{ // 定义包含漏洞的镜像列表
    "anchore/test_images:vulnerabilities-alpine",
    "anchore/test_images:gems",
    "anchore/test_images:vulnerabilities-debian",
    "anchore/test_images:vulnerabilities-centos",
    "anchore/test_images:npm",
    "anchore/test_images:java",
    "anchore/test_images:golang-56d52bc",
    "anchore/test_images:arch",
}

func getListingURL() string { // 定义获取更新 URL 的函数
    if value, ok := os.LookupEnv("GRYPE_DB_UPDATE_URL"); ok { // 获取环境变量中的更新 URL
        return value
    }
    return internal.DBUpdateURL // 返回默认的更新 URL
}

func must(e sbom.FormatEncoder, err error) sbom.FormatEncoder { // 定义必须成功的函数
    if err != nil { // 如果有错误发生
        panic(err) // 抛出异常
    }
    return e // 返回结果
}

func TestCompareSBOMInputToLibResults(t *testing.T) { // 定义测试函数
    // get a grype DB
    store, _, closer, err := grype.LoadVulnerabilityDB(db.Config{ // 加载漏洞数据库
        DBRootDir:           "test-fixtures/grype-db", // 指定数据库根目录
        ListingURL:          getListingURL(), // 获取更新 URL
        ValidateByHashOnGet: false, // 设置验证哈希值
    }, true) // 指定为真
    assert.NoError(t, err) // 断言没有错误发生

    if closer != nil { // 如果关闭函数不为空
        defer closer.Close() // 延迟执行关闭函数
    }

    definedPkgTypes := strset.New() // 创建字符串集合
    for _, p := range syftPkg.AllPkgs { // 遍历所有软件包
        definedPkgTypes.Add(string(p)) // 将软件包类型添加到集合中
    }
    // exceptions: rust, php, dart, msrc (kb), etc. are not under test
}
    // 从 definedPkgTypes 中移除指定的软件包类型
    definedPkgTypes.Remove(
        // 由于文件所有权重叠而被移除的软件包类型
        string(syftPkg.BinaryPkg),
        string(syftPkg.RustPkg),
        string(syftPkg.KbPkg),
        string(syftPkg.DartPubPkg),
        string(syftPkg.DotnetPkg),
        string(syftPkg.PhpComposerPkg),
        string(syftPkg.ConanPkg),
        string(syftPkg.HexPkg),
        string(syftPkg.PortagePkg),
        string(syftPkg.CocoapodsPkg),
        string(syftPkg.HackagePkg),
        string(syftPkg.NixPkg),
        string(syftPkg.JenkinsPluginPkg), // 无法为所有格式推断软件包类型
        string(syftPkg.LinuxKernelPkg),
        string(syftPkg.LinuxKernelModulePkg),
        string(syftPkg.Rpkg),
        string(syftPkg.SwiftPkg),
        string(syftPkg.GithubActionPkg),
        string(syftPkg.GithubActionWorkflowPkg),
    )
    // 创建一个新的字符串集合 observedPkgTypes
    observedPkgTypes := strset.New()
    // 创建一个测试用例切片，每个测试用例包含名称、镜像和格式编码器
    testCases := []struct {
        name   string
        image  string
        format sbom.FormatEncoder
    }
    for _, tc := range testCases {
        // 遍历测试用例
        imageArchive := PullThroughImageCache(t, tc.image)
        // 从镜像缓存中获取镜像存档
        imageSource := fmt.Sprintf("docker-archive:%s", imageArchive)
        // 格式化镜像来源字符串

        t.Run(tc.name, func(t *testing.T) {
            // 使用 syft 获取 SBOM，并写入临时文件
            sbomBytes := getSyftSBOM(t, imageSource, tc.format)
            // 创建临时文件
            sbomFile, err := os.CreateTemp("", "")
            assert.NoError(t, err)
            // 在测试结束时删除临时文件
            t.Cleanup(func() {
                assert.NoError(t, os.Remove(sbomFile.Name()))
            })
            // 将 SBOM 数据写入临时文件
            _, err = sbomFile.WriteString(sbomBytes)
            assert.NoError(t, err)
            assert.NoError(t, sbomFile.Close())

            // 获取 SBOM 中的漏洞
            matchesFromSbom, _, pkgsFromSbom, err := grype.FindVulnerabilities(*store, fmt.Sprintf("sbom:%s", sbomFile.Name()), source.SquashedScope, nil)
            assert.NoError(t, err)

            // 获取镜像中的漏洞
            matchesFromImage, _, _, err := grype.FindVulnerabilities(*store, imageSource, source.SquashedScope, nil)
            assert.NoError(t, err)

            // 比较软件包（浅层）
            matchSetFromSbom := getMatchSet(matchesFromSbom)
            matchSetFromImage := getMatchSet(matchesFromImage)

            // 断言 SBOM 中的漏洞是否都在镜像中存在
            assert.Empty(t, strset.Difference(matchSetFromSbom, matchSetFromImage).List(), "vulnerabilities present only in results when using sbom as input")
            // 断言镜像中的漏洞是否都在 SBOM 中存在
            assert.Empty(t, strset.Difference(matchSetFromImage, matchSetFromSbom).List(), "vulnerabilities present only in results when using image as input")

            // 跟踪所有覆盖的软件包类型（用于测试后使用）
            for _, p := range pkgsFromSbom {
                observedPkgTypes.Add(string(p.Type))
            }

        })
    }

    // 确保覆盖了所有软件包类型（-rust，-kb）
    unobservedPackageTypes := strset.Difference(definedPkgTypes, observedPkgTypes)
    assert.Empty(t, unobservedPackageTypes.List(), "not all package type were covered in testing")
# 闭合前面的函数定义
```