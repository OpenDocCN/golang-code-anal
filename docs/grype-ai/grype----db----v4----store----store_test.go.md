# `grype\grype\db\v4\store\store_test.go`

```
package store

import (
    "encoding/json"  // 导入用于 JSON 编解码的包
    "sort"  // 导入用于排序的包
    "testing"  // 导入测试框架包
    "time"  // 导入处理时间的包

    "github.com/go-test/deep"  // 导入用于深度比较的包
    "github.com/stretchr/testify/assert"  // 导入断言包

    v4 "github.com/anchore/grype/grype/db/v4"  // 导入版本 4 的数据库包
    "github.com/anchore/grype/grype/db/v4/store/model"  // 导入数据库模型包
)

func assertIDReader(t *testing.T, reader v4.IDReader, expected v4.ID) {
    t.Helper()  // 标记该函数为测试辅助函数
    if actual, err := reader.GetID(); err != nil {  // 获取 ID 信息
        t.Fatalf("failed to get ID: %+v", err)  // 如果获取失败则输出错误信息
    } else {
        diffs := deep.Equal(&expected, actual)  // 比较预期和实际结果
        if len(diffs) > 0 {  // 如果存在差异
            for _, d := range diffs {  // 遍历差异
                t.Errorf("Diff: %+v", d)  // 输出差异信息
            }
        }
    }
}

func TestStore_GetID_SetID(t *testing.T) {
    dbTempFile := t.TempDir()  // 创建临时目录用于数据库文件

    s, err := New(dbTempFile, true)  // 创建数据库实例
    if err != nil {  // 如果创建失败
        t.Fatalf("could not create store: %+v", err)  // 输出错误信息
    }

    expected := v4.ID{  // 创建预期的 ID 结构
        BuildTimestamp: time.Now().UTC(),  // 设置构建时间戳为当前时间
        SchemaVersion:  2,  // 设置模式版本为 2
    }

    if err = s.SetID(expected); err != nil {  // 设置 ID 信息
        t.Fatalf("failed to set ID: %+v", err)  // 输出错误信息
    }

    assertIDReader(t, s, expected)  // 调用辅助函数进行断言

}

func assertVulnerabilityReader(t *testing.T, reader v4.VulnerabilityStoreReader, namespace, name string, expected []v4.Vulnerability) {
    if actual, err := reader.GetVulnerability(namespace, name); err != nil {  // 获取漏洞信息
        t.Fatalf("failed to get Vulnerability: %+v", err)  // 输出错误信息
    } else {
        if len(actual) != len(expected) {  // 如果实际结果数量与预期不符
            t.Fatalf("unexpected number of vulns: %d", len(actual))  // 输出错误信息
        }
        for idx := range actual {  // 遍历实际结果
            diffs := deep.Equal(expected[idx], actual[idx])  // 比较预期和实际结果
            if len(diffs) > 0 {  // 如果存在差异
                for _, d := range diffs {  // 遍历差异
                    t.Errorf("Diff: %+v", d)  // 输出差异信息
                }
            }
        }
    }
}

func TestStore_GetVulnerability_SetVulnerability(t *testing.T) {
    dbTempFile := t.TempDir()  // 创建临时目录用于数据库文件

    s, err := New(dbTempFile, true)  // 创建数据库实例
    if err != nil {  // 如果创建失败
        t.Fatalf("could not create store: %+v", err)  // 输出错误信息
    }
    # 创建一个包含额外漏洞信息的切片
    extra := []v4.Vulnerability{
        {
            # 漏洞ID
            ID:                "my-cve-33333",
            # 包名
            PackageName:       "package-name-2",
            # 命名空间
            Namespace:         "my-namespace",
            # 版本约束
            VersionConstraint: "< 1.0",
            # 版本格式
            VersionFormat:     "semver",
            # CPEs
            CPEs:              []string{"a-cool-cpe"},
            # 相关漏洞引用
            RelatedVulnerabilities: []v4.VulnerabilityReference{
                {
                    # 相关漏洞ID
                    ID:        "another-cve",
                    # 相关漏洞命名空间
                    Namespace: "nvd",
                },
                {
                    # 相关漏洞ID
                    ID:        "an-other-cve",
                    # 相关漏洞命名空间
                    Namespace: "nvd",
                },
            },
            # 修复信息
            Fix: v4.Fix{
                # 修复版本
                Versions: []string{"2.0.1"},
                # 修复状态
                State:    v4.FixedState,
            },
        },
        {
            # 漏洞ID
            ID:                "my-other-cve-33333",
            # 包名
            PackageName:       "package-name-3",
            # 命名空间
            Namespace:         "my-namespace",
            # 版本约束
            VersionConstraint: "< 509.2.2",
            # 版本格式
            VersionFormat:     "semver",
            # CPEs
            CPEs:              []string{"a-cool-cpe"},
            # 相关漏洞引用
            RelatedVulnerabilities: []v4.VulnerabilityReference{
                {
                    # 相关漏洞ID
                    ID:        "another-cve",
                    # 相关漏洞命名空间
                    Namespace: "nvd",
                },
                {
                    # 相关漏洞ID
                    ID:        "an-other-cve",
                    # 相关漏洞命名空间
                    Namespace: "nvd",
                },
            },
            # 修复信息
            Fix: v4.Fix{
                # 修复状态
                State: v4.NotFixedState,
            },
        },
    }

    # 将额外漏洞信息添加到期望的漏洞信息中
    total := append(expected, extra...)

    # 如果添加漏洞信息的过程中出现错误，则输出错误信息
    if err = s.AddVulnerability(total...); err != nil {
        t.Fatalf("failed to set Vulnerability: %+v", err)
    }

    # 创建一个空的漏洞信息切片
    var allEntries []model.VulnerabilityModel
    # 从数据库中查找所有的漏洞信息
    s.(*store).db.Find(&allEntries)
    # 如果数据库中的漏洞信息数量与期望的数量不一致，则输出错误信息
    if len(allEntries) != len(total) {
        t.Fatalf("unexpected number of entries: %d", len(allEntries))
    }

    # 断言漏洞信息读取的正确性
    assertVulnerabilityReader(t, s, expected[0].Namespace, expected[0].PackageName, expected)
# 定义一个函数，用于断言漏洞元数据读取器的行为是否符合预期
func assertVulnerabilityMetadataReader(t *testing.T, reader v4.VulnerabilityMetadataStoreReader, id, namespace string, expected v4.VulnerabilityMetadata) {
    # 如果通过读取器获取漏洞元数据时发生错误，则输出错误信息并终止测试
    if actual, err := reader.GetVulnerabilityMetadata(id, namespace); err != nil {
        t.Fatalf("failed to get metadata: %+v", err)
    } 
    # 如果获取的实际漏洞元数据为空，则输出相应的错误信息并终止测试
    else if actual == nil {
        t.Fatalf("no metadata returned for id=%q namespace=%q", id, namespace)
    } 
    # 如果获取的实际漏洞元数据不为空，则进行后续的比较和断言
    else {
        # 对实际和期望的 CVSS 数据进行排序
        sortMetadataCvss(actual.Cvss)
        sortMetadataCvss(expected.Cvss)

        # 确保它们都具有相同数量的 CVSS 条目 - 防止后续断言时出现错误
        assert.Len(t, expected.Cvss, len(actual.Cvss))
        # 遍历实际 CVSS 数据，逐一比较其向量、版本、度量值和供应商元数据
        for idx, actualCvss := range actual.Cvss {
            assert.Equal(t, actualCvss.Vector, expected.Cvss[idx].Vector)
            assert.Equal(t, actualCvss.Version, expected.Cvss[idx].Version)
            assert.Equal(t, actualCvss.Metrics, expected.Cvss[idx].Metrics)

            # 将实际和期望的供应商元数据转换为 JSON 字符串，并进行比较
            actualVendor, err := json.Marshal(actualCvss.VendorMetadata)
            if err != nil {
                t.Errorf("unable to marshal vendor metadata: %q", err)
            }
            expectedVendor, err := json.Marshal(expected.Cvss[idx].VendorMetadata)
            if err != nil {
                t.Errorf("unable to marshal vendor metadata: %q", err)
            }
            assert.Equal(t, string(actualVendor), string(expectedVendor))
        }

        # 将期望和实际的 CVSS 字段设置为 nil，因为它是一个接口 - 在这一点上已经进行了 CVSS 的验证
        expected.Cvss = nil
        actual.Cvss = nil
        # 最终比较整个漏洞元数据对象的期望和实际值
        assert.Equal(t, &expected, actual)
    }
}

# 定义一个函数，用于对 CVSS 数据进行排序
func sortMetadataCvss(cvss []v4.Cvss) {
    # 使用sort.Slice函数对cvss切片进行排序，传入比较函数作为参数
    sort.Slice(cvss, func(i, j int) bool {
        # 首先按照Vector字段进行排序
        if cvss[i].Vector > cvss[j].Vector {
            return true
        }
        if cvss[i].Vector < cvss[j].Vector {
            return false
        }
        # 如果Vector相同，则尝试按照BaseScore字段进行排序
        return cvss[i].Metrics.BaseScore < cvss[j].Metrics.BaseScore
    })
// CustomMetadata 是一个空操作，其值并不具有实际意义，主要用于确保任何类型都可以被存储，然后在这些测试用例中进行断言，其中使用了自定义供应商 CVSS 分数
type CustomMetadata struct {
    SuperScore string
    Vendor     string
}

func TestStore_GetVulnerabilityMetadata_SetVulnerabilityMetadata(t *testing.T) {
    // 创建临时文件夹用于数据库
    dbTempFile := t.TempDir()

    // 创建存储对象
    s, err := New(dbTempFile, true)
    if err != nil {
        t.Fatalf("could not create store: %+v", err)
    }

    // 向存储中添加漏洞元数据
    if err = s.AddVulnerabilityMetadata(total...); err != nil {
        t.Fatalf("failed to set metadata: %+v", err)
    }

    // 获取所有漏洞元数据条目
    var allEntries []model.VulnerabilityMetadataModel
    s.(*store).db.Find(&allEntries)
    // 检查条目数量是否符合预期
    if len(allEntries) != len(total) {
        t.Fatalf("unexpected number of entries: %d", len(allEntries))
    }
}

func TestStore_MergeVulnerabilityMetadata(t *testing.T) {
    tests := []struct {
        name     string
        add      []v4.VulnerabilityMetadata
        expected v4.VulnerabilityMetadata
        err      bool
    }
    // 遍历测试用例切片，每个测试用例包含名称和测试函数
    for _, test := range tests {
        // 使用 t.Run 创建子测试，名称为测试用例的名称，执行测试函数
        t.Run(test.name, func(t *testing.T) {
            // 在临时目录创建数据库临时目录
            dbTempDir := t.TempDir()
            // 创建新的存储对象，如果出错则报错
            s, err := New(dbTempDir, true)
            if err != nil {
                t.Fatalf("could not create store: %+v", err)
            }

            // 依次添加每个元数据
            var theErr error
            for _, metadata := range test.add {
                // 添加漏洞元数据，如果出错则记录错误并中断
                err = s.AddVulnerabilityMetadata(metadata)
                if err != nil {
                    theErr = err
                    break
                }
            }

            // 检查是否期望出错但未出错，或者期望不出错但出错
            if test.err && theErr == nil {
                t.Fatalf("expected error but did not get one")
            } else if !test.err && theErr != nil {
                t.Fatalf("expected no error but got one: %+v", theErr)
            } else if test.err && theErr != nil {
                // 如果测试通过，则返回
                return
            }

            // 确保只有一个条目
            var allEntries []model.VulnerabilityMetadataModel
            // 从数据库中查找所有条目
            s.(*store).db.Find(&allEntries)
            if len(allEntries) != 1 {
                t.Fatalf("unexpected number of entries: %d", len(allEntries))
            }

            // 获取预期的元数据对象
            if actual, err := s.GetVulnerabilityMetadata(test.expected.ID, test.expected.Namespace); err != nil {
                t.Fatalf("failed to get metadata: %+v", err)
            } else {
                // 比较预期和实际结果，输出差异
                diffs := deep.Equal(&test.expected, actual)
                if len(diffs) > 0 {
                    for _, d := range diffs {
                        t.Errorf("Diff: %+v", d)
                    }
                }
            }
        })
    }
// 测试函数，用于测试 CVSS 分数是否在元数据中
func TestCvssScoresInMetadata(t *testing.T) {
    // 测试用例
    tests := []struct {
        name     string
        add      []v4.VulnerabilityMetadata
        expected v4.VulnerabilityMetadata
    }
    // 遍历测试用例
    for _, test := range tests {
        t.Run(test.name, func(t *testing.T) {
            // 创建临时目录
            dbTempDir := t.TempDir()

            // 创建新的存储对象
            s, err := New(dbTempDir, true)
            if err != nil {
                t.Fatalf("could not create s: %+v", err)
            }

            // 依次添加每个元数据
            for _, metadata := range test.add {
                err = s.AddVulnerabilityMetadata(metadata)
                if err != nil {
                    t.Fatalf("unable to s vulnerability metadata: %+v", err)
                }
            }

            // 确保只有一个条目
            var allEntries []model.VulnerabilityMetadataModel
            s.(*store).db.Find(&allEntries)
            if len(allEntries) != 1 {
                t.Fatalf("unexpected number of entries: %d", len(allEntries))
            }

            // 断言元数据读取器
            assertVulnerabilityMetadataReader(t, s, test.expected.ID, test.expected.Namespace, test.expected)
        })
    }
}

// 断言漏洞匹配排除读取器
func assertVulnerabilityMatchExclusionReader(t *testing.T, reader v4.VulnerabilityMatchExclusionStoreReader, id string, expected []v4.VulnerabilityMatchExclusion) {
    if actual, err := reader.GetVulnerabilityMatchExclusion(id); err != nil {
        t.Fatalf("failed to get Vulnerability Match Exclusion: %+v", err)
    } else {
        t.Logf("%+v", actual)
        if len(actual) != len(expected) {
            t.Fatalf("unexpected number of vulnerability match exclusions: expected=%d, actual=%d", len(expected), len(actual))
        }
        for idx := range actual {
            diffs := deep.Equal(expected[idx], actual[idx])
            if len(diffs) > 0 {
                for _, d := range diffs {
                    t.Errorf("Diff: %+v", d)
                }
            }
        }
    }
}
# 定义一个测试函数，用于测试获取和设置漏洞匹配排除项
func TestStore_GetVulnerabilityMatchExclusion_SetVulnerabilityMatchExclusion(t *testing.T) {
    # 在临时目录中创建一个数据库临时文件
    dbTempFile := t.TempDir()

    # 创建一个新的存储对象，并检查是否有错误发生
    s, err := New(dbTempFile, true)
    if err != nil {
        # 如果有错误发生，则输出错误信息并终止测试
        t.Fatalf("could not create store: %+v", err)
    }
    # 创建一个包含 VulnerabilityMatchExclusion 结构的切片
    extra := []v4.VulnerabilityMatchExclusion{
        # 创建一个 VulnerabilityMatchExclusion 结构
        {
            # 设置漏洞匹配排除的ID
            ID: "CVE-1234-14567",
            # 设置漏洞匹配排除的约束条件
            Constraints: []v4.VulnerabilityMatchExclusionConstraint{
                # 设置漏洞匹配排除的漏洞约束条件
                {
                    Vulnerability: v4.VulnerabilityExclusionConstraint{
                        Namespace: "extra-namespace:cpe",
                    },
                    # 设置漏洞匹配排除的包约束条件
                    Package: v4.PackageExclusionConstraint{
                        Name:     "abc",
                        Language: "ruby",
                        Version:  "1.2.3",
                    },
                },
                # 设置漏洞匹配排除的漏洞约束条件
                {
                    Vulnerability: v4.VulnerabilityExclusionConstraint{
                        Namespace: "extra-namespace:cpe",
                    },
                    # 设置漏洞匹配排除的包约束条件
                    Package: v4.PackageExclusionConstraint{
                        Name:     "abc",
                        Language: "ruby",
                        Version:  "4.5.6",
                    },
                },
                # 设置漏洞匹配排除的漏洞约束条件
                {
                    Vulnerability: v4.VulnerabilityExclusionConstraint{
                        Namespace: "extra-namespace:cpe",
                    },
                    # 设置漏洞匹配排除的包约束条件
                    Package: v4.PackageExclusionConstraint{
                        Name:     "time-1",
                        Language: "ruby",
                    },
                },
                # 设置漏洞匹配排除的漏洞约束条件
                {
                    Vulnerability: v4.VulnerabilityExclusionConstraint{
                        Namespace: "extra-namespace:cpe",
                    },
                    # 设置漏洞匹配排除的包约束条件
                    Package: v4.PackageExclusionConstraint{
                        Name: "abc.xyz:nothing-of-interest",
                        Type: "java-archive",
                    },
                },
            },
            # 设置漏洞匹配排除的理由
            Justification: "Because I said so.",
        },
        # 创建另一个 VulnerabilityMatchExclusion 结构
        {
            # 设置漏洞匹配排除的ID
            ID:            "CVE-1234-10",
            # 设置漏洞匹配排除的约束条件为空
            Constraints:   nil,
            # 设置漏洞匹配排除的理由
            Justification: "Because I said so.",
        },
    }

    # 将 extra 切片中的元素追加到 expected 切片中，并返回新的切片
    total := append(expected, extra...)
    # 如果将 total 中的所有元素作为参数传递给 AddVulnerabilityMatchExclusion 方法时出现错误
    if err = s.AddVulnerabilityMatchExclusion(total...); err != nil:
        # 输出错误信息并终止测试
        t.Fatalf("failed to set Vulnerability Match Exclusion: %+v", err)

    # 声明一个空的 model.VulnerabilityMatchExclusionModel 切片
    var allEntries []model.VulnerabilityMatchExclusionModel
    # 从数据库中查找所有的 VulnerabilityMatchExclusionModel 记录，并将结果存储到 allEntries 中
    s.(*store).db.Find(&allEntries)
    # 如果 allEntries 的长度不等于 total 的长度
    if len(allEntries) != len(total):
        # 输出错误信息并终止测试
        t.Fatalf("unexpected number of entries: %d", len(allEntries))
    # 调用 assertVulnerabilityMatchExclusionReader 方法，验证 s 对象的行为是否符合预期
    assertVulnerabilityMatchExclusionReader(t, s, expected[0].ID, expected)
func Test_DiffStore(t *testing.T) {
    // 定义测试函数 Test_DiffStore，用于测试 DiffStore 函数
    // GIVEN
    // 创建临时目录作为数据库临时文件
    dbTempFile := t.TempDir()

    // 创建新的存储对象 s1，并检查是否出错
    s1, err := New(dbTempFile, true)
    if err != nil {
        t.Fatalf("could not create store: %+v", err)
    }
    // 重新分配临时目录作为数据库临时文件
    dbTempFile = t.TempDir()

    // 创建新的存储对象 s2，并检查是否出错
    s2, err := New(dbTempFile, true)
    if err != nil {
        t.Fatalf("could not create store: %+v", err)
    }

    // 定义基础漏洞列表 baseVulns
    baseVulns := []v4.Vulnerability{
        {
            Namespace:         "github:language:python",
            ID:                "CVE-123-4567",
            PackageName:       "pypi:requests",
            VersionConstraint: "< 2.0 >= 1.29",
            CPEs:              []string{"cpe:2.3:pypi:requests:*:*:*:*:*:*"},
        },
        {
            Namespace:         "github:language:python",
            ID:                "CVE-123-4567",
            PackageName:       "pypi:requests",
            VersionConstraint: "< 3.0 >= 2.17",
            CPEs:              []string{"cpe:2.3:pypi:requests:*:*:*:*:*:*"},
        },
        {
            Namespace:         "npm",
            ID:                "CVE-123-7654",
            PackageName:       "npm:axios",
            VersionConstraint: "< 3.0 >= 2.17",
            CPEs:              []string{"cpe:2.3:npm:axios:*:*:*:*:*:*"},
            Fix: v4.Fix{
                State: v4.UnknownFixState,
            },
        },
        {
            Namespace:         "nuget",
            ID:                "GHSA-****-******",
            PackageName:       "nuget:net",
            VersionConstraint: "< 3.0 >= 2.17",
            CPEs:              []string{"cpe:2.3:nuget:net:*:*:*:*:*:*"},
            Fix: v4.Fix{
                State: v4.UnknownFixState,
            },
        },
        {
            Namespace:         "hex",
            ID:                "GHSA-^^^^-^^^^^^",
            PackageName:       "hex:esbuild",
            VersionConstraint: "< 3.0 >= 2.17",
            CPEs:              []string{"cpe:2.3:hex:esbuild:*:*:*:*:*:*"},
        },
    }
}
    # 创建基本漏洞元数据列表
    baseMetadata := []v4.VulnerabilityMetadata{
        {
            Namespace:  "nuget",
            ID:         "GHSA-****-******",
            DataSource: "nvd",
        },
    }
    # 创建目标漏洞列表
    targetVulns := []v4.Vulnerability{
        {
            Namespace:         "github:language:python",
            ID:                "CVE-123-4567",
            PackageName:       "pypi:requests",
            VersionConstraint: "< 2.0 >= 1.29",
            CPEs:              []string{"cpe:2.3:pypi:requests:*:*:*:*:*:*"},
        },
        {
            Namespace:         "github:language:go",
            ID:                "GHSA-....-....",
            PackageName:       "hashicorp:nomad",
            VersionConstraint: "< 3.0 >= 2.17",
            CPEs:              []string{"cpe:2.3:golang:hashicorp:nomad:*:*:*:*:*"},
        },
        {
            Namespace:         "github:language:go",
            ID:                "GHSA-....-....",
            PackageName:       "hashicorp:n",
            VersionConstraint: "< 2.0 >= 1.17",
            CPEs:              []string{"cpe:2.3:golang:hashicorp:n:*:*:*:*:*"},
        },
        {
            Namespace:         "npm",
            ID:                "CVE-123-7654",
            PackageName:       "npm:axios",
            VersionConstraint: "< 3.0 >= 2.17",
            CPEs:              []string{"cpe:2.3:npm:axios:*:*:*:*:*:*"},
            # 设置修复状态为不会修复
            Fix: v4.Fix{
                State: v4.WontFixState,
            },
        },
        {
            Namespace:         "nuget",
            ID:                "GHSA-****-******",
            PackageName:       "nuget:net",
            VersionConstraint: "< 3.0 >= 2.17",
            CPEs:              []string{"cpe:2.3:nuget:net:*:*:*:*:*:*"},
            # 设置修复状态为未知
            Fix: v4.Fix{
                State: v4.UnknownFixState,
            },
        },
    }
    // 定义一个期望的漏洞差异列表，包含了不同的漏洞情况
    expectedDiffs := []v4.Diff{
        {
            Reason:    v4.DiffChanged,  // 漏洞原因为已更改
            ID:        "CVE-123-4567",  // 漏洞ID
            Namespace: "github:language:python",  // 漏洞所属的命名空间
            Packages:  []string{"pypi:requests"},  // 受影响的软件包列表
        },
        {
            Reason:    v4.DiffChanged,
            ID:        "CVE-123-7654",
            Namespace: "npm",
            Packages:  []string{"npm:axios"},
        },
        {
            Reason:    v4.DiffRemoved,
            ID:        "GHSA-****-******",
            Namespace: "nuget",
            Packages:  []string{"nuget:net"},
        },
        {
            Reason:    v4.DiffAdded,
            ID:        "GHSA-....-....",
            Namespace: "github:language:go",
            Packages:  []string{"hashicorp:n", "hashicorp:nomad"},
        },
        {
            Reason:    v4.DiffRemoved,
            ID:        "GHSA-^^^^-^^^^^^",
            Namespace: "hex",
            Packages:  []string{"hex:esbuild"},
        },
    }

    // 将基准漏洞列表中的漏洞添加到 s1 中
    for _, vuln := range baseVulns {
        s1.AddVulnerability(vuln)
    }
    // 将目标漏洞列表中的漏洞添加到 s2 中
    for _, vuln := range targetVulns {
        s2.AddVulnerability(vuln)
    }
    // 将基准元数据列表中的元数据添加到 s1 中
    for _, meta := range baseMetadata {
        s1.AddVulnerabilityMetadata(meta)
    }

    // 当执行漏洞差异存储操作时
    result, err := s1.DiffStore(s2)

    // 然后
    // 对结果进行排序，按照漏洞ID进行稳定排序
    sort.SliceStable(*result, func(i, j int) bool {
        return (*result)[i].ID < (*result)[j].ID
    })
    // 对每个结果中的软件包列表进行排序
    for i := range *result {
        sort.Strings((*result)[i].Packages)
    }

    // 断言操作，验证错误是否为空
    assert.NoError(t, err)
    // 断言操作，验证结果是否与期望的漏洞差异列表相等
    assert.Equal(t, expectedDiffs, *result)
# 闭合前面的函数定义
```