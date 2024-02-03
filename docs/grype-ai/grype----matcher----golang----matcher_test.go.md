# `grype\grype\matcher\golang\matcher_test.go`

```go
package golang

import (
    "testing"  // 导入测试包

    "github.com/google/uuid"  // 导入 UUID 生成包
    "github.com/scylladb/go-set/strset"  // 导入字符串集合包
    "github.com/stretchr/testify/assert"  // 导入断言包

    "github.com/anchore/grype/grype/distro"  // 导入发行版包
    "github.com/anchore/grype/grype/pkg"  // 导入包信息包
    "github.com/anchore/grype/grype/version"  // 导入版本信息包
    "github.com/anchore/grype/grype/vulnerability"  // 导入漏洞信息包
    "github.com/anchore/syft/syft/cpe"  // 导入 CPE 包
    syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syft 包

)

func TestMatcher_DropMainPackage(t *testing.T) {

    mainModuleMetadata := pkg.GolangBinMetadata{  // 定义主模块元数据
        MainModule: "istio.io/istio",
    }

    subjectWithoutMainModule := pkg.Package{  // 定义没有主模块的包
        ID:       pkg.ID(uuid.NewString()),  // 使用 UUID 生成唯一标识
        Name:     "istio.io/istio",  // 包名
        Version:  "v0.0.0-20220606222826-f59ce19ec6b6",  // 版本号
        Type:     syftPkg.GoModulePkg,  // 包类型
        Language: syftPkg.Go,  // 包语言
        Metadata: pkg.GolangBinMetadata{},  // 包元数据
    }

    subjectWithMainModule := subjectWithoutMainModule  // 复制没有主模块的包
    subjectWithMainModule.Metadata = mainModuleMetadata  // 设置包的元数据为主模块元数据

    subjectWithMainModuleAsDevel := subjectWithMainModule  // 复制包含主模块的包
    subjectWithMainModuleAsDevel.Version = "(devel)"  // 设置版本为开发版本

    matcher := NewGolangMatcher(MatcherConfig{})  // 创建 Golang 匹配器
    store := newMockProvider()  // 创建模拟提供者

    preTest, _ := matcher.Match(store, nil, subjectWithoutMainModule)  // 进行匹配测试
    assert.Len(t, preTest, 1, "should have matched the package when there is not a main module")  // 断言匹配结果数量为1，当没有主模块时应该匹配成功

    actual, _ := matcher.Match(store, nil, subjectWithMainModule)  // 进行匹配测试
    assert.Len(t, actual, 0, "unexpected match count; should not match main module")  // 断言匹配结果数量为0，不应该匹配主模块

    actual, _ = matcher.Match(store, nil, subjectWithMainModuleAsDevel)  // 进行匹配测试
    assert.Len(t, actual, 0, "unexpected match count; should not match main module (devel)")  // 断言匹配结果数量为0，不应该匹配主模块（开发版本）
}

func TestMatcher_SearchForStdlib(t *testing.T) {

    // values derived from:
    //   $ go version -m $(which grype)
    //  /opt/homebrew/bin/grype: go1.21.1
}
    # 创建一个名为 subject 的 pkg.Package 结构体对象
    subject := pkg.Package{
        # 为 ID 属性赋值一个新生成的 UUID 字符串
        ID:       pkg.ID(uuid.NewString()),
        # 设置 Name 属性为 "stdlib"
        Name:     "stdlib",
        # 设置 Version 属性为 "go1.18.3"
        Version:  "go1.18.3",
        # 设置 Type 属性为 syftPkg.GoModulePkg
        Type:     syftPkg.GoModulePkg,
        # 设置 Language 属性为 syftPkg.Go
        Language: syftPkg.Go,
        # 设置 CPEs 属性为包含一个 CPE 对象的数组
        CPEs: []cpe.CPE{
            cpe.Must("cpe:2.3:a:golang:go:1.18.3:-:*:*:*:*:*:*"),
        },
        # 设置 Metadata 属性为 pkg.GolangBinMetadata 结构体对象
        Metadata: pkg.GolangBinMetadata{},
    }

    # 创建一个名为 cases 的结构体数组
    cases := []struct {
        name         string
        cfg          MatcherConfig
        subject      pkg.Package
        expectedCVEs []string
    }

    # 创建一个名为 store 的 mockProvider 对象
    store := newMockProvider()

    # 遍历 cases 数组
    for _, c := range cases {
        # 使用测试名称和匿名函数运行测试
        t.Run(c.name, func(t *testing.T) {
            # 创建一个名为 matcher 的 GolangMatcher 对象
            matcher := NewGolangMatcher(c.cfg)

            # 调用 matcher 的 Match 方法，获取匹配结果和错误信息
            actual, _ := matcher.Match(store, nil, c.subject)
            # 创建一个名为 actualCVEs 的字符串集合
            actualCVEs := strset.New()
            # 遍历匹配结果，将漏洞 ID 添加到 actualCVEs 中
            for _, m := range actual {
                actualCVEs.Add(m.Vulnerability.ID)
            }

            # 创建一个名为 expectedCVEs 的字符串集合，包含预期的漏洞 ID
            expectedCVEs := strset.New(c.expectedCVEs...)

            # 使用 assert.ElementsMatch 方法断言预期的漏洞 ID 和实际的漏洞 ID 是否匹配
            assert.ElementsMatch(t, expectedCVEs.List(), actualCVEs.List())

        })
    }
// 创建一个新的模拟提供者对象
func newMockProvider() *mockProvider {
    // 初始化一个包含语言和漏洞数据的映射
    mp := mockProvider{
        data: make(map[syftPkg.Language]map[string][]vulnerability.Vulnerability),
    }

    // 填充模拟数据
    mp.populateData()

    // 返回模拟提供者对象的指针
    return &mp
}

// 定义模拟提供者结构
type mockProvider struct {
    data map[syftPkg.Language]map[string][]vulnerability.Vulnerability
}

// 根据 ID 和命名空间获取漏洞数据
func (mp *mockProvider) Get(id, namespace string) ([]vulnerability.Vulnerability, error) {
    // 抛出错误，提示需要实现该方法
    panic("implement me")
}

// 填充模拟数据
func (mp *mockProvider) populateData() {
    // 填充 Go 语言的漏洞数据
    mp.data[syftPkg.Go] = map[string][]vulnerability.Vulnerability{
        // 用于测试 TestMatcher_DropMainPackage
        "istio.io/istio": {
            {
                Constraint: version.MustGetConstraint("< 5.0.7", version.UnknownFormat),
                ID:         "CVE-2013-fake-BAD",
            },
        },
    }

    // 填充 NVD:CPE 的漏洞数据
    mp.data["nvd:cpe"] = map[string][]vulnerability.Vulnerability{
        // 用于测试 TestMatcher_SearchForStdlib
        "cpe:2.3:a:golang:go:1.18.3:-:*:*:*:*:*:*": {
            {
                Constraint: version.MustGetConstraint("< 1.18.6 || = 1.19.0", version.UnknownFormat),
                ID:         "CVE-2022-27664",
            },
        },
    }
}

// 根据 CPE 获取漏洞数据
func (mp *mockProvider) GetByCPE(p cpe.CPE) ([]vulnerability.Vulnerability, error) {
    return mp.data["nvd:cpe"][p.BindToFmtString()], nil
}

// 根据发行版和包获取漏洞数据
func (mp *mockProvider) GetByDistro(d *distro.Distro, p pkg.Package) ([]vulnerability.Vulnerability, error) {
    return []vulnerability.Vulnerability{}, nil
}

// 根据语言和包获取漏洞数据
func (mp *mockProvider) GetByLanguage(l syftPkg.Language, p pkg.Package) ([]vulnerability.Vulnerability, error) {
    return mp.data[l][p.Name], nil
}
```