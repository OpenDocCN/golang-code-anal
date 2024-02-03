# `grype\grype\pkg\package_test.go`

```go
package pkg

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "strings" // 导入 strings 包，用于处理字符串
    "testing" // 导入 testing 包，用于编写测试函数

    "github.com/stretchr/testify/assert" // 导入 testify 包，用于编写断言

    "github.com/anchore/syft/syft/artifact" // 导入 artifact 包
    "github.com/anchore/syft/syft/cpe" // 导入 cpe 包
    "github.com/anchore/syft/syft/file" // 导入 file 包
    syftFile "github.com/anchore/syft/syft/file" // 导入 file 包，并重命名为 syftFile
    "github.com/anchore/syft/syft/linux" // 导入 linux 包
    syftPkg "github.com/anchore/syft/syft/pkg" // 导入 pkg 包，并重命名为 syftPkg
    "github.com/anchore/syft/syft/sbom" // 导入 sbom 包
    "github.com/anchore/syft/syft/testutil" // 导入 testutil 包
)

func TestNew(t *testing.T) {
    tests := []struct {
        name      string
        syftPkg   syftPkg.Package
        metadata  interface{}
        upstreams []UpstreamPackage
    } // 定义测试用例结构体数组

    // capture each observed metadata type, we should see all of them relate to what syft provides by the end of testing
    tester := testutil.NewPackageMetadataCompletionTester(t) // 创建测试工具

    // run all of our cases
    for _, test := range tests { // 遍历测试用例
        t.Run(test.name, func(t *testing.T) { // 运行测试用例
            tester.Tested(t, test.syftPkg.Metadata) // 测试元数据
            p := New(test.syftPkg) // 创建新的包
            assert.Equal(t, test.metadata, p.Metadata, "unexpected metadata") // 断言元数据相等
            assert.Equal(t, test.upstreams, p.Upstreams, "unexpected upstream") // 断言上游数据相等
        })
    }
}

func TestFromCollection_DoesNotPanic(t *testing.T) {
    collection := syftPkg.NewCollection() // 创建新的包集合

    examplePackage := syftPkg.Package{ // 创建示例包
        Name:    "test",
        Version: "1.2.3",
        Locations: file.NewLocationSet(
            file.NewLocation("/test-path"),
        ),
        Type: syftPkg.NpmPkg,
    }

    collection.Add(examplePackage) // 向包集合中添加包
    // add it again!
    collection.Add(examplePackage) // 再次添加相同的包

    assert.NotPanics(t, func() { // 断言不会发生恐慌
        _ = FromCollection(collection, SynthesisConfig{}) // 从包集合中生成合成配置
    })
}

func TestFromCollection_GeneratesCPEs(t *testing.T) {
    collection := syftPkg.NewCollection() // 创建新的包集合

    collection.Add(syftPkg.Package{ // 向包集合中添加包
        Name:    "first",
        Version: "1",
        CPEs: []cpe.CPE{ // 设置 CPE
            {},
        },
    })
    // 向集合中添加一个包对象，包含名称和版本信息
    collection.Add(syftPkg.Package{
        Name:    "second",
        Version: "2",
    })

    // 当没有标志时，不生成 CPEs
    // 从集合中创建包对象列表，使用默认的综合配置
    pkgs := FromCollection(collection, SynthesisConfig{})
    // 断言第一个包对象的 CPEs 数量为1
    assert.Len(t, pkgs[0].CPEs, 1)
    // 断言第二个包对象的 CPEs 数量为0
    assert.Len(t, pkgs[1].CPEs, 0)

    // 当有标志时，生成 CPEs
    // 从集合中创建包对象列表，使用指定的综合配置
    pkgs = FromCollection(collection, SynthesisConfig{
        GenerateMissingCPEs: true,
    })
    // 断言第一个包对象的 CPEs 数量为1
    assert.Len(t, pkgs[0].CPEs, 1)
    // 断言第二个包对象的 CPEs 数量为1
    assert.Len(t, pkgs[1].CPEs, 1)
# 测试函数，用于测试获取软件包名称和EL版本的函数
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
    }
    # 遍历测试用例
    for _, test := range tests {
        # 运行测试用例
        t.Run(test.name, func(t *testing.T) {
            # 获取实际的软件包名称和EL版本
            actualName, actualVersion := getNameAndELVersion(test.sourceRPM)
            # 断言实际结果与期望结果相等
            assert.Equal(t, test.expectedName, actualName)
            assert.Equal(t, test.expectedVersion, actualVersion)
        })
    }
}

# 返回整数的指针
func intRef(i int) *int {
    return &i
}

# 测试函数，用于测试通过重叠移除软件包
func Test_RemovePackagesByOverlap(t *testing.T) {
    # 定义测试用例
    tests := []struct {
        name             string
        sbom             *sbom.SBOM
        expectedPackages []string
    }
    # 遍历测试用例集合
    for _, test := range tests {
        # 使用测试名称创建子测试，传入测试函数
        t.Run(test.name, func(t *testing.T) {
            # 根据测试用例中的软件构建物对象、关系对象和 Linux 发行版信息，移除重叠的软件包
            catalog := removePackagesByOverlap(test.sbom.Artifacts.Packages, test.sbom.Relationships, test.sbom.Artifacts.LinuxDistribution)
            # 从软件包目录中创建软件包集合
            pkgs := FromCollection(catalog, SynthesisConfig{})
            # 创建空的软件包名称列表
            var pkgNames []string
            # 遍历软件包集合，将软件包类型、名称和版本信息格式化为字符串，添加到软件包名称列表中
            for _, p := range pkgs {
                pkgNames = append(pkgNames, fmt.Sprintf("%s:%s@%s", p.Type, p.Name, p.Version))
            }
            # 断言测试结果是否与期望的软件包名称列表相等
            assert.EqualValues(t, test.expectedPackages, pkgNames)
        })
    }
# 定义一个函数，接收两个参数：packages（包列表）和overlaps（重叠列表），返回一个SBOM对象
func catalogWithOverlaps(packages []string, overlaps []string) *sbom.SBOM {
    # 定义两个空数组，用于存储syftPkg.Package对象和artifact.Relationship对象
    var pkgs []syftPkg.Package
    var relationships []artifact.Relationship

    # 定义一个内部函数，用于将字符串转换为syftPkg.Package对象
    toPkg := func(str string) syftPkg.Package {
        # 定义变量用于存储类型、名称和版本
        var typ, name, version string
        # 使用冒号分割字符串，获取类型和名称/版本
        s := strings.Split(strings.TrimSpace(str), ":")
        if len(s) > 1 {
            typ = s[0]
            str = s[1]
        }
        s = strings.Split(str, "@")
        name = s[0]
        if len(s) > 1 {
            version = s[1]
        }

        # 创建syftPkg.Package对象并设置属性
        p := syftPkg.Package{
            Type:    syftPkg.Type(typ),
            Name:    name,
            Version: version,
        }
        p.SetID()

        return p
    }

    # 遍历包列表，将每个包字符串转换为syftPkg.Package对象并存储到pkgs数组中
    for _, pkg := range packages {
        p := toPkg(pkg)
        pkgs = append(pkgs, p)
    }

    # 遍历重叠列表，将每个重叠字符串转换为artifact.Relationship对象并存储到relationships数组中
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

    # 创建一个包集合对象catalog，包含所有的包对象
    catalog := syftPkg.NewCollection(pkgs...)

    # 返回一个SBOM对象，包含包集合和关系数组
    return &sbom.SBOM{
        Artifacts: sbom.Artifacts{
            Packages: catalog,
        },
        Relationships: relationships,
    }
}

# 定义一个函数，接收两个参数：s（SBOM对象）和id（Linux发行版ID），返回一个SBOM对象
func withDistro(s *sbom.SBOM, id string) *sbom.SBOM {
    # 设置SBOM对象的LinuxDistribution属性为一个Linux发行版对象
    s.Artifacts.LinuxDistribution = &linux.Release{
        ID: id,
    }
    # 返回更新后的SBOM对象
    return s
}
```