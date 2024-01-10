# `grype\grype\presenter\sarif\presenter_test.go`

```
package sarif

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "flag"   // 导入 flag 包，用于解析命令行参数
    "fmt"    // 导入 fmt 包，用于格式化输出
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/stretchr/testify/assert"  // 导入 testify/assert 包，用于编写断言语句

    "github.com/anchore/clio"  // 导入 anchore/clio 包
    "github.com/anchore/go-testutils"  // 导入 anchore/go-testutils 包
    "github.com/anchore/grype/grype/pkg"  // 导入 anchore/grype/grype/pkg 包
    "github.com/anchore/grype/grype/presenter/internal"  // 导入 anchore/grype/grype/presenter/internal 包
    "github.com/anchore/grype/grype/presenter/models"  // 导入 anchore/grype/grype/presenter/models 包
    "github.com/anchore/grype/grype/vulnerability"  // 导入 anchore/grype/grype/vulnerability 包
    "github.com/anchore/syft/syft/file"  // 导入 anchore/syft/syft/file 包
    "github.com/anchore/syft/syft/source"  // 导入 anchore/syft/syft/source 包
)

var updateSnapshot = flag.Bool("update-sarif", false, "update .golden files for sarif presenters")  // 定义一个命令行参数，用于更新 .golden 文件

func TestSarifPresenter(t *testing.T) {  // 定义测试函数 TestSarifPresenter
    tests := []struct {  // 定义一个结构体切片 tests
        name   string  // 结构体字段 name，类型为 string
        scheme internal.SyftSource  // 结构体字段 scheme，类型为 internal.SyftSource
    }{
        {
            name:   "directory",  // 第一个结构体的 name 字段值为 "directory"
            scheme: internal.DirectorySource,  // 第一个结构体的 scheme 字段值为 internal.DirectorySource
        },
        {
            name:   "image",  // 第二个结构体的 name 字段值为 "image"
            scheme: internal.ImageSource,  // 第二个结构体的 scheme 字段值为 internal.ImageSource
        },
    }
    # 遍历测试用例切片，每个测试用例为 tc
    for _, tc := range tests {
        # 将 tc 赋值给新的变量 tc，避免闭包中的变量覆盖问题
        tc := tc
        # 使用测试名称运行子测试，每个子测试为 tc.name
        t.Run(tc.name, func(t *testing.T) {
            # 创建一个字节缓冲区
            var buffer bytes.Buffer
            # 调用 internal.GenerateAnalysis 生成分析结果，返回多个值
            _, matches, packages, context, metadataProvider, _, _ := internal.GenerateAnalysis(t, tc.scheme)

            # 创建 PresenterConfig 结构体对象 pb
            pb := models.PresenterConfig{
                ID: clio.Identification{
                    Name: "grype",
                },
                Matches:          matches,
                Packages:         packages,
                Context:          context,
                MetadataProvider: metadataProvider,
            }

            # 使用 PresenterConfig 对象创建 Presenter 对象 pres
            pres := NewPresenter(pb)
            # 调用 Present 方法将结果写入 buffer
            err := pres.Present(&buffer)
            # 如果出现错误，测试失败
            if err != nil {
                t.Fatal(err)
            }

            # 获取 buffer 中的字节内容
            actual := buffer.Bytes()
            # 如果需要更新快照文件
            if *updateSnapshot {
                # 更新测试用例的黄金文件内容
                testutils.UpdateGoldenFileContents(t, actual)
            }

            # 获取黄金文件的内容
            var expected = testutils.GetGoldenFileContents(t)
            # 对实际和期望的内容进行脱敏处理
            actual = internal.Redact(actual)
            expected = internal.Redact(expected)

            # 如果实际内容和期望内容不相等，使用 JSON 格式进行断言
            if !bytes.Equal(expected, actual) {
                assert.JSONEq(t, string(expected), string(actual))
            }
        })
    }
}

// Test_locationPath 函数用于测试 locationPath 函数
func Test_locationPath(t *testing.T) {
    // 定义测试用例
    tests := []struct {
        name     string
        metadata any
        real     string
        virtual  string
        expected string
    }

    // 遍历测试用例
    for _, test := range tests {
        // 在测试中运行子测试
        t.Run(test.name, func(t *testing.T) {
            // 创建目录 Presenter
            pres := createDirPresenter(t)
            // 设置目录 Presenter 的源描述
            pres.src = &source.Description{
                Metadata: test.metadata,
            }

            // 调用 packagePath 函数获取路径
            path := pres.packagePath(pkg.Package{
                Locations: file.NewLocationSet(
                    file.NewVirtualLocation(test.real, test.virtual),
                ),
            })

            // 断言路径是否符合预期
            assert.Equal(t, test.expected, path)
        })
    }
}

// createDirPresenter 函数用于创建目录 Presenter
func createDirPresenter(t *testing.T) *Presenter {
    // 生成分析数据
    _, matches, packages, _, metadataProvider, _, _ := internal.GenerateAnalysis(t, internal.DirectorySource)
    // 创建临时目录
    d := t.TempDir()
    // 从目录创建新的源
    s, err := source.NewFromDirectory(source.DirectoryConfig{Path: d})
    if err != nil {
        t.Fatal(err)
    }

    // 获取源描述
    desc := s.Describe()
    // 创建 Presenter 配置
    pb := models.PresenterConfig{
        Matches:          matches,
        Packages:         packages,
        MetadataProvider: metadataProvider,
        Context: pkg.Context{
            Source: &desc,
        },
    }

    // 创建 Presenter
    pres := NewPresenter(pb)

    return pres
}

// TestToSarifReport 函数用于测试生成 Sarif 报告
func TestToSarifReport(t *testing.T) {
    // 定义测试用例
    tt := []struct {
        name      string
        scheme    internal.SyftSource
        locations map[string]string
    # 定义一个包含两个元素的匿名结构体切片
    }{
        # 第一个元素
        {
            # 元素属性：名称为 "directory"
            name:   "directory",
            # 元素属性：方案为 internal.DirectorySource
            scheme: internal.DirectorySource,
            # 元素属性：位置为包含文件名到路径的映射
            locations: map[string]string{
                "CVE-1999-0001-package-1": "/some/path/somefile-1.txt",
                "CVE-1999-0002-package-2": "/some/path/somefile-2.txt",
            },
        },
        # 第二个元素
        {
            # 元素属性：名称为 "image"
            name:   "image",
            # 元素属性：方案为 internal.ImageSource
            scheme: internal.ImageSource,
            # 元素属性：位置为包含文件名到路径的映射
            locations: map[string]string{
                "CVE-1999-0001-package-1": "image/somefile-1.txt",
                "CVE-1999-0002-package-2": "image/somefile-2.txt",
            },
        },
    }
    # 结束匿名结构体切片的定义
    }
# 定义一个空的元数据提供者结构体
type NilMetadataProvider struct{}

# 实现空的元数据提供者结构体的获取元数据方法
func (m *NilMetadataProvider) GetMetadata(_, _ string) (*vulnerability.Metadata, error) {
    # 返回空的元数据和空的错误
    return nil, nil
}

# 定义一个模拟的元数据提供者结构体
type MockMetadataProvider struct{}

# 实现模拟的元数据提供者结构体的获取元数据方法
func (m *MockMetadataProvider) GetMetadata(id, namespace string) (*vulnerability.Metadata, error) {
    # 定义一个内部函数用于生成包含 CVSS 评分的元数据
    cvss := func(id string, namespace string, scores ...float64) vulnerability.Metadata {
        # 创建一个包含 CVSS 评分的数组
        values := make([]vulnerability.Cvss, len(scores))
        # 遍历评分，将其添加到数组中
        for _, score := range scores {
            values = append(values, vulnerability.Cvss{
                Metrics: vulnerability.CvssMetrics{
                    BaseScore: score,
                },
            })
        }
        # 返回包含评分的元数据
        return vulnerability.Metadata{
            ID:        id,
            Namespace: namespace,
            Cvss:      values,
        }
    }
    # 生成一组模拟的元数据
    values := []vulnerability.Metadata{
        cvss("1", "nvd:cpe", 1),
        cvss("1", "not-nvd", 2),
        cvss("2", "not-nvd", 3, 4),
    }
    # 遍历元数据，查找匹配的 ID 和命名空间
    for _, v := range values {
        if v.ID == id && v.Namespace == namespace {
            return &v, nil
        }
    }
    # 如果未找到匹配的元数据，返回空和错误信息
    return nil, fmt.Errorf("not found")
}

# 测试空元数据的 CVSS 评分
func Test_cvssScoreWithNilMetadata(t *testing.T) {
    # 创建一个 Presenter 结构体实例，使用空的元数据提供者
    pres := Presenter{
        metadataProvider: &NilMetadataProvider{},
    }
    # 调用 cvssScore 方法计算 CVSS 评分
    score := pres.cvssScore(vulnerability.Vulnerability{
        ID:        "id",
        Namespace: "namespace",
    })
    # 断言计算结果为 -1
    assert.Equal(t, float64(-1), score)
}

# 测试计算 CVSS 评分
func Test_cvssScore(t *testing.T) {
    # 定义测试用例
    tests := []struct {
        name          string
        vulnerability vulnerability.Vulnerability
        expected      float64
    # 定义一个包含多个测试用例的列表，每个测试用例包含名称、漏洞信息、期望值等内容
    }{
        # 第一个测试用例，名称为"none"，漏洞信息包含ID为"4"的漏洞，相关漏洞为空，期望值为-1
        {
            name: "none",
            vulnerability: vulnerability.Vulnerability{
                ID: "4",
                RelatedVulnerabilities: []vulnerability.Reference{
                    {
                        ID:        "7",
                        Namespace: "nvd:cpe",
                    },
                },
            },
            expected: -1,
        },
        # 第二个测试用例，名称为"direct"，漏洞信息包含ID为"2"的漏洞，命名空间为"not-nvd"，相关漏洞包含ID为"1"的漏洞，期望值为4
        {
            name: "direct",
            vulnerability: vulnerability.Vulnerability{
                ID:        "2",
                Namespace: "not-nvd",
                RelatedVulnerabilities: []vulnerability.Reference{
                    {
                        ID:        "1",
                        Namespace: "nvd:cpe",
                    },
                },
            },
            expected: 4,
        },
        # 第三个测试用例，名称为"related not nvd"，漏洞信息包含ID为"1"的漏洞，命名空间为"nvd:cpe"，相关漏洞包含ID为"1"和"1"的漏洞，命名空间分别为"nvd:cpe"和"not-nvd"，期望值为2
        {
            name: "related not nvd",
            vulnerability: vulnerability.Vulnerability{
                ID:        "1",
                Namespace: "nvd:cpe",
                RelatedVulnerabilities: []vulnerability.Reference{
                    {
                        ID:        "1",
                        Namespace: "nvd:cpe",
                    },
                    {
                        ID:        "1",
                        Namespace: "not-nvd",
                    },
                },
            },
            expected: 2,
        },
        # 第四个测试用例，名称为"related nvd"，漏洞信息包含ID为"4"的漏洞，命名空间为"not-nvd"，相关漏洞包含ID为"1"和"7"的漏洞，命名空间分别为"nvd:cpe"和"not-nvd"，期望值为1
        {
            name: "related nvd",
            vulnerability: vulnerability.Vulnerability{
                ID:        "4",
                Namespace: "not-nvd",
                RelatedVulnerabilities: []vulnerability.Reference{
                    {
                        ID:        "1",
                        Namespace: "nvd:cpe",
                    },
                    {
                        ID:        "7",
                        Namespace: "not-nvd",
                    },
                },
            },
            expected: 1,
        },
    }
    # 遍历测试用例列表
    for _, test := range tests {
        # 使用测试名称创建子测试，并执行匿名函数
        t.Run(test.name, func(t *testing.T) {
            # 创建 Presenter 结构体实例，其中 metadataProvider 为 MockMetadataProvider 实例
            pres := Presenter{
                metadataProvider: &MockMetadataProvider{},
            }
            # 调用 cvssScore 方法计算漏洞的 CVSS 分数
            score := pres.cvssScore(test.vulnerability)
            # 使用断言检查计算得到的分数是否与预期值相等
            assert.Equal(t, test.expected, score)
        })
    }
# 闭合前面的函数定义
```